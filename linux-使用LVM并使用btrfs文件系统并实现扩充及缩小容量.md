# 说明

可以使用 LVM 管理，也可以使用 btrfs 自带的功能。

## 0. 当前分区情况

[[images/my-photo.png]]

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

# 1.3 扩容后分区挂载情况

![[Pasted image 20260118153146.png]]

# 2.缩小容量

## 2.1 缩小btrfs文件系统

```bash
# 缩小100GB
sudo btrfs filesystem resize -100G /
```
## 2.2 缩小逻辑卷

```bash
# 1. 缩小100GB
sudo lvreduce -L -100G --resizefs /dev/vgroup0/lvol0

# 2. 查看逻辑卷大小
sudo lvdisplay /dev/vgroup0/lvol0 | grep "LV Size"

# 3. 查看物理卷状态
sudo pvs
```

## 2.3  (可选)迁移数据

```bash 
# (可选)迁移需要移除物理卷的大小
sudo pvmove /dev/nvme1n1p7
```
## 2.4 (可选)移除物理卷

```bash
# 1. 从卷组中移除物理卷
sudo vgreduce vgroup0 /dev/nvme1n1p7

# 2. 删除物理卷的LVM标记
sudo pvremove /dev/nvme1n1p7
```

## 2.5. 验证状态

```bash
# 验证移除结果
sudo pvs
sudo vgdisplay vgroup0
```

## 2.6 缩小容量后

![[Pasted image 20260119112503.png]]