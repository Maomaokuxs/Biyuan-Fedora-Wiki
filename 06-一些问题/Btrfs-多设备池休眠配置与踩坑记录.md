# 说明

根分区为 Btrfs 文件系统，原为单设备（`nvme0n1p6`）。为了扩容使用 `btrfs device add` 将第二个设备加入 Btrfs 存储池，此后休眠全面失效。本文记录了从单设备休眠正常工作到加入设备后休眠崩溃再到最终修复的全过程。

环境：Fedora 44 KDE、Btrfs 多设备、UEFI + GRUB

## 0. 单设备阶段（休眠正常）

- **配置方案（swapfile）**

```bash
# 创建独立子卷 @swap
# btrfs filesystem mkswapfile --size 4G /swap/swapfile
# 配置 resume=UUID=... resume_offset=...
# /proc/cmdline
resume=UUID=5434cf7c-7871-40ce-91d6-92a03d4ad7a0 resume_offset=8686766
```

## 1. 加入第二块设备后休眠崩溃

```bash
sudo btrfs device add /dev/nvme0n1p7 /
```

休眠立即失败，报错 `Call to Hibernate failed: Specified resume device is missing or is not an active swap device`

## 2. 故障排查

1. **UUID 歧义**

    - **症状**：内核解析 `resume=UUID=5434cf7c-...` 时可能指向 `nvme0n1p7`（新设备）而非 `nvme0n1p6`，swapfile 的物理 extent 只在 `nvme0n1p6` 上，在 `nvme0n1p7` 上找不到 swap 签名
    - **验证**：两个设备共享同一个 Btrfs UUID，`/sys/power/resume` 可能指向错误设备
    - **修复**：使用 PARTUUID 替代 UUID，唯一标识 `nvme0n1p6`

    UUID 是文件系统级别的标识，加入 Btrfs 池的多个设备共享同一 UUID。PARTUUID 是 GPT 分区表级别的标识，每个分区唯一，不受文件系统影响。

2. **systemd 预检拒绝**

    - **症状**：`sudo systemctl hibernate` 失败，但 `echo disk | sudo tee /sys/power/state` 正常工作
    - **原因**：systemd 在执行休眠前做容量预检，检查 swap 分区是否足够容纳已用内存。swapfile 只有 4G，而已用内存约 4.2G，systemd 直接拒绝

    | 方式 | 行为 | 结果 |
    | ------ | ------ | ------ |
    | `systemctl hibernate` | systemd 预检 → 拒绝 | ❌ |
    | `echo disk > /sys/power/state` | 直接调用内核，跳过预检 | ✅ |

    - **解决方案**：扩大 swapfile 或改用独立 swap 分区

3. **swapfile 跨设备分配（致命）**

    - **症状**：`swapon` 失败，`btrfs inspect-internal map-swapfile -r` 返回 `ERROR: file stored on multiple devices`
    - **原因**：Linux 内核不支持跨多个 Btrfs 设备的 swapfile。Btrfs 池中有多个设备时，mkswapfile 创建的文件可能被分配到不同设备上
    - **结论**：只要 Btrfs 池有多设备，swapfile 就不可靠，必须改用独立 swap 分区

4. **无法从池中移除设备**

    - **症状**：`btrfs device remove` 报错 `No space left on device`
    - **原因**：数据分布到两个设备后，要把数据全部搬回 p6，但 p6 容量小于已用数据，无法回流

5. **SELinux（假阳性）**

    日志中出现 SELinux AVC 拒绝，但关闭 SELinux 后休眠仍然失败，确认 SELinux 不是根因。

## 3. 最终解决方案：独立 Swap 分区

放弃 swapfile + Btrfs 多设备方案。swap 分区不依赖文件系统，无跨设备问题，不需要 `resume_offset`，使用 PARTUUID 定位无 UUID 歧义。

```bash
# 1. 创建 swap 分区（16G）
sudo cfdisk /dev/nvme0n1

# 2. 格式化并启用
sudo mkswap /dev/nvme0n1p5
sudo swapon /dev/nvme0n1p5

# 3. 获取 PARTUUID
ls -la /dev/disk/by-partuuid/ | grep nvme0n1p5

# 4. 写入 fstab
echo 'PARTUUID=你的PARTUUID none swap defaults 0 0' | sudo tee -a /etc/fstab

# 5. 配置内核参数
sudo grubby --update-kernel=ALL --args="resume=PARTUUID=你的PARTUUID"

# 6. 配置 dracut
echo 'add_dracutmodules+=" resume "' | sudo tee /etc/dracut.conf.d/resume.conf
sudo dracut --force
sudo grub2-mkconfig -o /boot/grub2/grub.cfg

# 7. 重启测试
sudo systemctl hibernate
```

独立 swap 分区不需要 `resume_offset`。

## 4. 关键区别总结

| 项目 | 单设备 Btrfs（swapfile） | 多设备 Btrfs（swapfile） | 独立 swap 分区 |
| ------ | ------------------------ | ------------------------ | ---------------- |
| 可用 | ✅ | ❌ 跨设备限制 | ✅ |
| resume 参数 | UUID + offset | UUID/PARTUUID + offset | PARTUUID（无 offset） |
| 休眠稳定性 | 高 | 极低 | 最高 |
| 扩容灵活性 | 需重建 swapfile | 需处理跨设备问题 | 独立分区，不受影响 |

## 5. 常见错误与修复速查

| 错误信息 | 原因 | 修复 |
| --------- | ------ | ------ |
| `resume device is missing or is not an active swap device` | UUID 歧义 / swap 太小 / 跨设备 swapfile | 改用 PARTUUID + 独立 swap 分区 |
| `file stored on multiple devices` | swapfile 在 Btrfs 多设备池中 | 删除 swapfile，改用独立 swap 分区 |
| `invalid environment block` | grubenv 损坏 | `sudo rm -f /boot/grub2/grubenv && sudo grub2-editenv /boot/grub2/grubenv create` |
| `Cannot find module directory` | 救援内核 initramfs 版本不匹配 | `sudo dracut --force /boot/initramfs-0-rescue-*.img $(uname -r)` |

## 6. 最终分区布局参考

```text
nvme0n1p1   100M  vfat     EFI
nvme0n1p2    16M           BIOS
nvme0n1p3  339.2G ntfs     Windows
nvme0n1p4    16G  swap     [SWAP]    ← 独立 swap 分区用于休眠
nvme0n1p6    40G  btrfs    Btrfs 池成员
nvme0n1p7   488M  vfat     /boot/efi
nvme0n1p8  80.3G  btrfs    Btrfs 池成员（/)
```
