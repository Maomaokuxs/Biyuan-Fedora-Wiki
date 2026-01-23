# 1. 查看当前的 LVM 结构

``` bash
# 查看物理卷（PV）情况
sudo pvdisplay

# 查看卷组（VG）情况
sudo vgdisplay vgroup0

# 查看逻辑卷（LV）情况
sudo lvdisplay
```

- 报错示例：

``` 
# 可以看到 vgroup0 丢失 物理卷 PV EOAf56-s2OK-i9Qf-qRee-5AXY-zeyw-VIAFpt 其写入到 设备/dev/nvme1n1p9

WARNING: Couldn't find device with uuid EOAf56-s2OK-i9Qf-qRee-5AXY-zeyw-VIAFpt.
  WARNING: VG vgroup0 is missing PV EOAf56-s2OK-i9Qf-qRee-5AXY-zeyw-VIAFpt (last written to /dev/nvme1n1p9).
  WARNING: Couldn't find all devices for LV vgroup0/lvol0 while checking used and assumed devices.
  --- Physical volume ---
  PV Name               /dev/sda3
  VG Name               vgroup0
  PV Size               <93.85 GiB / not usable 0   
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              24025
  Free PE               0
  Allocated PE          24025
  PV UUID               XQslOS-necU-YGlf-0WGr-EA2H-XQ0O-z1in0q
   
  --- Physical volume ---
  PV Name               [unknown]
  VG Name               vgroup0
  PV Size               111.38 GiB / not usable 0   
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              28514
  Free PE               0
  Allocated PE          28514
  PV UUID               EOAf56-s2OK-i9Qf-qRee-5AXY-zeyw-VIAFpt
```
# 2. 尝试查找丢失的 PV

```bash
# 1.查看所有存储设备
lsblk -f

# 在下面的图中可以看到设备 /dev/nvme1n1p5 中包含有逻辑卷组，且 UUID 可以对应上丢失的设备 /dev/nvme1n1p9
```
![[Pasted image 20260118142629.png]]

```bash
# （可选）2.扫描所有设备上的物理卷
sudo pvscan

# （可选）3.查看是否有未激活的 PV
sudo vgdisplay -v vgroup0（修改为对应的逻辑卷组）
```

# 3. 尝试重新激活 PV

```bash
# 1.重新激活物理卷
sudo pvchange -ay /dev/nvme1n1p5
# 2.检查是否成功
sudo pvdisplay /dev/nvme1n1p5
```

- 报错示例

![[Pasted image 20260118144156.png]]

- 报错原因

```markdown
# 1. 什么是 LVM Devices File？

这是 LVM 2.03+ 引入的安全特性：

- 文件位置：/etc/lvm/devices/system.devices
    
- 目的：只允许 LVM 使用白名单中的设备
    
- 安全作用：防止意外使用外部/USB设备
    

# 2. 为什么会出现这个错误？

当您运行 sudo pvdisplay /dev/nvme1n1p5 时：

1. LVM 检查设备文件 /etc/lvm/devices/system.devices
    
2. 如果设备不在白名单中，拒绝访问
    
3. 显示错误：device is not in devices file
```

# 4. 直接更新设备数据库

``` bash
# 1. 清除旧的缓存（在删除前可以1进行备份）
sudo rm -f /etc/lvm/cache/.cache

# 2. 手动添加设备到设备数据库
# 这里的设备文件要根据实际情况进行修改
sudo lvmdevices --adddev /dev/nvme1n1p5
sudo lvmdevices --adddev /dev/sda3

# 3. 更新设备数据库
# 这里逻辑卷名称要根据实际情况修改
sudo vgimportdevices vgroup0

# 4. 重新扫描
sudo pvscan --cache -aay

# 5. 查看当前使用的设备
sudo cat /etc/lvm/devices/system.devices  

# 示例：

# LVM uses devices listed in this file.  
# Created by LVM command vgimportdevices pid 6424 at Sun Jan 18 13:55:53 2026  
# HASH=3984349505  
PRODUCT_UUID=5e2a1c66-d554-5c95-8eb9-bcfce74db9d0  
VERSION=1.1.29  
IDTYPE=sys_wwid IDNAME=naa.5000c500faed0560 DEVNAME=/dev/sda3 PVID=XQslOSnecUYGlf0WGrEA2HXQ0Oz1in0q PART=3  
IDTYPE=sys_wwid IDNAME=eui.00000000000000008ce38e03009dcb67 DEVNAME=/dev/nvme1n1p5 PVID=EOAf56s2OKi9QfqRee5AXYzeywVIAFpt PART=5
```

# 5. 验证修复

```bash
# 1. 检查物理卷状态
sudo pvdisplay

# 2. 检查卷组状态
sudo vgdisplay vgroup0

# 3. 检查逻辑卷状态
sudo lvdisplay /dev/vgroup0/lvol0
```
# 6. 可能需要进行的操作，上面恢复正常的话不用执行

```
# 1. 备份当前 LVM 配置
sudo cp /etc/lvm/backup/vgroup0 /etc/lvm/backup/vgroup0.backup.$(date +%Y%m%d)

# 2. 编辑配置文件，将 nvme1n1p9 改为 nvme1n1p5
sudo sed -i 's|/dev/nvme1n1p9|/dev/nvme1n1p5|g' /etc/lvm/backup/vgroup0

# 3. 恢复配置
sudo vgcfgrestore vgroup0

# 4. 激活
sudo vgchange -ay vgroup0
```