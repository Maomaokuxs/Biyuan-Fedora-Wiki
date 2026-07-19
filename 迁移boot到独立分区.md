# 说明

当 Btrfs 根文件系统存在多设备时，GRUB 读取 `/boot` 下的文件可能出现 `sparse file not allow` 等报错。将 `/boot` 迁移到独立的 ext4 分区可以彻底解决此问题。本文记录了完整操作流程及避坑指南。

## 0. 前提

- Btrfs 多设备存储池
- `/boot` 在 Btrfs 子卷上
- 需要一块空闲空间用于创建 ext4 分区

## 1. 创建 ext4 分区

从 Btrfs 池成员末尾压缩空间：

```bash
# 查看当前磁盘布局
sudo parted /dev/nvme0n1 unit s print

# 缩小 Btrfs 文件系统（释放设备 1 末尾 2G）
sudo btrfs filesystem resize 1:-2G /

# 缩小分区（用计算出的新结束扇区替换）
sudo parted /dev/nvme0n1 resizepart 8 996020223s

# 在末尾创建 2G ext4 分区
sudo parted /dev/nvme0n1 mkpart primary ext4 996020224s 100%

# 格式化
sudo mkfs.ext4 /dev/nvme0n1p10
```

## 2. 迁移 /boot 内容

```bash
sudo mount /dev/nvme0n1p10 /mnt
sudo cp -a /boot/* /mnt/
sudo umount /mnt
```

## 3. 挂载新 /boot 并更新 EFI

> ⚠️ 这一步必须先将 ext4 挂载到 `/boot`，否则重装 GRUB 包检测到的还是旧的 Btrfs 路径。

```bash
# 将 ext4 挂载到 /boot（覆盖当前挂载）
sudo mount /dev/nvme0n1p10 /boot

# 重装 GRUB 包（自动检测当前 /boot 分区，生成正确的 EFI grub.cfg）
sudo dnf reinstall shim-x64 grub2-efi-x64 -y
```

重装后 EFI grub.cfg 会自动指向 ext4 UUID 和 `/grub2` 前缀。无需手动 `sed`。

## 4. 修改 fstab

```bash
# 备份
sudo cp /etc/fstab /etc/fstab.bak

# 将 /boot 行改为 ext4（用实际 UUID 替换）
sudo sed -i 's|^UUID=.* /boot btrfs.*|UUID=你的UUID /boot ext4 defaults 0 2|' /etc/fstab

# 验证
grep " /boot " /etc/fstab
```

## 5. 重建 GRUB

```bash
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

## 6. 重启验证

```bash
sudo systemctl reboot
```

重启后验证：

```bash
cat /proc/cmdline | grep subvol    # 应为 subvol=@
mount | grep " / "                  # 应为 /dev/nvme0n1p8
mount | grep " /boot "              # 应为 /dev/nvme0n1p10 ext4
df -h /boot                         # 应为 2G 分区
```

## 正确操作顺序（避坑必看）

这是最关键的总结，顺序错了会陷入死循环：

```text
① 创建 ext4 分区
② 复制 /boot 内容到 ext4
③ 挂载 ext4 到 /boot
④ 重装 shim-x64 + grub2-efi-x64（自动检测当前 ext4，生成正确的 EFI grub.cfg）
⑤ 修改 fstab 为 ext4
⑥ 重建 GRUB
⑦ 重启
```

**关键点：** 步骤 ③ 到 ⑤ 必须在同一次会话中一次性完成。顺序不能错：

- 先挂载 ext4 → 重装 GRUB 检测到 ext4 → 生成正确的 EFI 配置
- 再改 fstab → 下次重启时 systemd 将 ext4 挂到 /boot
- 先改 fstab 再重启 → systemd 挂 ext4 覆盖当前 /boot → 可能失败 → 进快照 → 死循环

**错误操作（我踩过的坑）：**

```text
① 创建 ext4 分区 ✓
② 复制 /boot 内容 ✓
③ 挂载 ext4 到 /boot ✓
④ 重建 GRUB ✓
⑤ 没更新 EFI grub.cfg (x)
⑥ 重启
   ↓
   fstab 指向 ext4 → systemd 挂载 ext4
   /boot 成功切换到 ext4 ✅
   ↓
   但 EFI 仍指向 Btrfs UUID
   BLS 内核路径仍是 /@boot/vmlinuz-...（从 Btrfs 复制过来的）
   ext4 上没有 @boot 子卷 → 内核路径不对
   ↓
   后续操作导致第二次重启失败
   被迫选 Snapshot → kernel-install add 写死快照路径 → 死循环
```

**根因：** `/boot` 切换到 ext4 后，EFI grub.cfg 没同步更新。同时 BLS 条目中的内核路径还带着 `/@boot/` 前缀（ext4 上没有这个子卷），导致从 ext4 引导时找不到内核。

**关键教训：** 迁完 `/boot` 后，EFI 指向、BLS 内核路径、fstab 三者必须同步更新，缺一不可。

## 常见问题

### GRUB 报错 `sparse file not allow`

原因：Btrfs 启用压缩后，GRUB 无法读取稀疏文件。
解决：将 `/boot` 迁移到 ext4（本文完整流程）。

### GRUB 报错 `invalid environment block`

原因：grubenv 中的 `env_block` 变量指向磁盘扇区，GRUB 更新后位置变化。
解决：

```bash
sudo grub2-editenv /boot/grub2/grubenv unset env_block
```

### 修改 BLS 条目后重启被还原

原因：修改的是 Btrfs 快照中的文件（只读），不是 ext4 上的文件。
解决：先确认 `/boot` 挂载的是哪个设备。

```bash
df /boot        # 显示 ext4 还是 Btrfs？
mount | grep /boot  # 确认设备
```

### EFI grub.cfg 被手动 sed 弄坏

解决：

```bash
sudo dnf reinstall shim-x64 grub2-efi-x64 -y
```

### 在快照中无法运行 dnf

原因：Snapper 快照的 RPM 数据库是只读的。
解决：重启进入真实 `@` 子卷后再执行。

### BLS 条目指向了快照子卷

原因：在快照中运行了 `kernel-install add`，写入了当前快照路径。
解决：

```bash
sudo sed -i 's|subvol=@/.snapshots/[0-9]*/snapshot|subvol=@|g' /boot/loader/entries/*.conf
```
