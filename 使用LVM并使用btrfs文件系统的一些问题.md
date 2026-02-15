# 说明

- 可以使用 LVM 管理，也可以使用 btrfs 自带的功能,这一部分后面应该会加上。
- 我十分不建议你这样部署分区用于个人使用，LVM 和 btrfs 部分功能重叠，还加入一层实在是不方便。
- 如果不考虑快照的话，LVM 可以和 ext4 搭配使用。
- 如果有重要文件可以参考第三节的内容转换为btrfs。

## 0. 当前分区情况

[[images/fedora/LVM-btrfs/disk-status-1.png]]

## 1. 扩大容量

### 1.1 创建一个未格式化的分区

```txt
在/dev/nvme1n1设备上创建分区/dev/nvme1n1p7
```

## 1.2  将新分区加入LVM

```bash
# 1. 创建物理卷
sudo pvcreate /dev/nvme1n1p7

# 2. 扩展卷组
sudo vgextend vgroup0 /dev/nvme1n1p7

# 3. 扩展逻辑卷（使用全部空闲空间）
sudo lvextend -l +100%FREE /dev/vgroup0/lvol0

# 4. 扩展btrfs文件系统
sudo btrfs filesystem resize max /
# 或指定大小：sudo btrfs filesystem resize +50G /
```

### 1.3 扩容后分区挂载情况

[[images/fedora/LVM-btrfs/disk-status-2.png]]

## 2.缩小容量

### 2.1 缩小btrfs文件系统

```bash
# 缩小100GB
sudo btrfs filesystem resize -100G /
```

### 2.2 缩小逻辑卷

```bash
# 1. 缩小100GB
sudo lvreduce -L -100G --resizefs /dev/vgroup0/lvol0

# 2. 查看逻辑卷大小
sudo lvdisplay /dev/vgroup0/lvol0 | grep "LV Size"

# 3. 查看物理卷状态
sudo pvs
```

### 2.3  (可选)迁移数据

```bash
# (可选)迁移需要移除物理卷的大小
sudo pvmove /dev/nvme1n1p7
```

### 2.4 (可选)移除物理卷

```bash
# 1. 从卷组中移除物理卷
sudo vgreduce vgroup0 /dev/nvme1n1p7

# 2. 删除物理卷的LVM标记
sudo pvremove /dev/nvme1n1p7
```

### 2.5. 验证状态

```bash
# 验证移除结果
sudo pvs
sudo vgdisplay vgroup0
```

### 2.6 缩小容量后

[[images/fedora/LVM-btrfs/disk-status-3.png]]

## 3. (可选) 将 LVM 管理下 btrfs 子卷发送至新的系统，或将其中的文件备份

如果当前系统还能够使用可以先将一些配置文件备份，并将 home 目录整个备份带入新系统，再者就是备份旧系统中安装了什么软件包。

我并不认为直接将 btrfs 子卷直接发送至新系统是一个很好的决定，但是下面的内容会表述相关内容，写这一部分内容时我使用 LVM 创建逻辑卷并将文件系统格式化为 btrfs，这意义并不是很大，我需要将 btrfs 子卷中的内容备份到我的新系统中，以下内同均在新系统中进行。

### 3.1 定位 LVM 卷

首先，需要确保 LVM 卷已激活，并找到它的设备路径。

- 扫描并激活卷组

```bash
sudo vgchange -ay
```

- 查看逻辑卷路径

```bash
sudo lvdisplay 或 lsblk
# 下文中假设逻辑卷组路径是 /dev/vgroup0/lvol0
```

### 3.2 查看 Btrfs 子卷信息并挂载顶级子卷

```bash
# 1.创建挂载点
sudo mkdir -p /mnt/old

# 2.临时挂载逻辑卷到 /mnt/old
sudo mount -o subvolid=5 /dev/vgroup0/lvol0 /mnt/old

# 3.查看所有子卷及其 ID
sudo btrfs subvolume list /mnt/old
# 下文假设操作的子卷信息为：
# ID 601 gen 12509 top level 457 path @home/.snapshots/40/snapshot
# @home :子卷名称

# 4.验证子卷是否为只读
sudo btrfs subvolume show /mnt/old/@home/.snapshots/40/snapshot | grep -i ro
# 如果显示 ro: true 则直接可用；若为 false，请先将其设为只读
sudo btrfs property set -fst /mnt/old/@home/.snapshots/40/snapshot ro true

```

### 3.3 （可选）恢复 snapper 创建的某个快照

- 备份当前（坏掉的）home：

```Bash
sudo mv /mnt/@home /mnt/@home_bad_backup
```

- 从快照创建一个新的可写子卷：
快照通常是只读的，所以我们需要基于快照创建一个新的可写副本。

```Bash
# 假设你选定 ID 601 的快照作为恢复点
sudo btrfs subvolume snapshot /mnt/@home_bad_backup/.snapshots/40/snapshot /mnt/@home
```

### 3.4 将 LVM 管理下的btrfs文件系统的 @home 子卷发送至新系统

#### 3.4.1 在当前系统中创建一个存放位置

```bash
sudo mkdir -p ~/recovered_files/
```

#### 3.4.2 执行传送：将旧快照的数据流导入到新系统中

```bash
sudo btrfs send /mnt/old/@home/.snapshots/40/snapshot | sudo btrfs receive /receive_files/
```

#### (可选) 3.4.3 创建可写子卷作为新的 home

建议在 root 控制台（tty模式）下以root用户或者在Live系统中执行下面的操作。

```bash
# 1.卸载当前的 /home
sudo umount /home

# 2.备份当前的home目录
cd /
sudo mv /@home /@home_backup

# 3.将接收的只读快照创建可写子卷并重命名为@home
sudo btrfs subvolume snapshot /receive_files/snapshot /@home

# 4.挂载新的 @home 到 /home
sudo mount -o subvol=@home /dev/sda2 /home

# 5.验证挂载是否成功
ls -l /home

# 6.更新 /etc/fstab 以实现永久挂载
UUID=xxxx-xxxx  /home  btrfs  subvol=/@home,defaults  0  0

# 7.重启验证
# 在重启之前一定要检查清除上述设置完成。

sudo reboot
```
