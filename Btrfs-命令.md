# 说明

总结一些管理brtfs文件系统的常用命令。

## 基础信息查询

- 查看整体使用情况：

```bash
sudo btrfs filesystem usage /
```

比传统的 df -h 更准确，能显示真正分配的物理空间和未分配空间。

- 查看磁盘列表：

```bash
sudo btrfs filesystem show
```

- 按项目列出空间占用：

```bash
sudo btrfs filesystem du -s /path/to/dir
```

---

## 子卷（Subvolumes）管理

- 创建子卷：

```bash
sudo btrfs subvolume create /path/to/subvol
```

- 列出所有子卷：

```bash
sudo btrfs subvolume list /
```

- 删除子卷：

```bash
sudo btrfs subvolume delete /path/to/subvol
```

- 查看默认启动子卷：

```bash
sudo btrfs subvolume get-default /
```

---

## 快照（Snapshots）操作

- 创建只读快照：

```bash
sudo btrfs subvolume snapshot -r / /snapshots/root_backup
```

- 创建可写快照：

```bash
sudo btrfs subvolume snapshot / /snapshots/root_editable
```

---

## 数据一致性与维护

- Scrub（数据洗刷）：

```bash
sudo btrfs scrub start /
```

读取所有数据并验证校验和，如果发现静默数据损坏且有冗余，会自动修复。

- 查看 Scrub 进度：

```bash
sudo btrfs scrub status /
```

- Balance（负载均衡）：

```bash
sudo btrfs balance start /
```

在多硬盘或频繁删除文件后重新排列数据块，回收空闲的 Block Groups。

---

## 多设备管理（RAID）

Btrfs 可以在线添加或移除磁盘，无需卸载文件系统。

- 添加新硬盘：

```bash
sudo btrfs device add /dev/sdb /mnt/btrfs
```

- 在线移除硬盘：

```bash
sudo btrfs device remove /dev/sdb /mnt/btrfs
```

系统会自动将数据迁移到其他盘后再断开。
