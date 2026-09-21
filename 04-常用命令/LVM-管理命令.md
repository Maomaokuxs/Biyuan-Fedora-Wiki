# 说明

- 这一部分主要是一些使用 LVM 常用命令。

- LVM（Logical Volume Manager，逻辑卷管理器）通过将物理硬盘抽象为统一的存储池，提供了极高的灵活性。

---

## 1. 物理卷 (PV) 操作

- 初始化物理卷（可以是分区或整块盘）

```bash
pvcreate /dev/nvme0n1p3        

```

- 查看物理卷简要信息

```bash
pvs                            
```

- 查看物理卷详细状态

```bash
pvdisplay                     

```

- 将数据从一个PV迁移到另一个PV（常用于换盘）

```bash
pvmove /dev/sda1 /dev/sdb1     

```

- 移除物理卷标签

```bash
pvremove /dev/sda1             

```

---

## 2. 卷组 (VG) 操作

- 创建卷组并关联物理卷

```bash
vgcreate my_vg /dev/nvme0n1p3  

```

- 查看卷组简要信息

```bash
vgs                            

```

- 查看卷组详细信息

```bash
vgdisplay                      

```

- 向卷组中添加新的物理卷进行扩容

```bash
vgextend my_vg /dev/sda1       

```

- 从卷组中移除一个物理卷

```bash
vgreduce my_vg /dev/sda1       

```

- 删除整个卷组

```bash
vgremove my_vg                 

```

---

## 3. 逻辑卷 (LV) 操作

- 在 my_vg 中创建 50G 的逻辑卷

```bash
lvcreate -L 50G -n my_lv my_vg             

```

- 使用所有剩余空间创建逻辑卷

```bash
lvcreate -l 100%FREE -n my_lv my_vg        

```

- 查看逻辑卷简要信息

```bash
lvs                                       

```

- 查看逻辑卷详细信息

```bash
lvdisplay                                  

```

- 扩容 10G 并同步调整文件系统（推荐）

```bash
lvextend -L +10G /dev/my_vg/my_lv -r       

```

- 缩减 5G 并同步调整文件系统（有风险，慎用）

```bash
lvreduce -L -5G /dev/my_vg/my_lv -r        

```

- 删除逻辑卷

```bash
lvremove /dev/my_vg/my_lv                  

```

---

## 4. 文件系统手动调整 (未加 -r 参数时)

- 针对 ext4 文件系统的在线扩容

```bash
resize2fs /dev/my_vg/my_lv     

```

- 针对 XFS 文件系统的扩容（参数为挂载点）

```bash
xfs_growfs /                      

```

---

## 5. 辅助与扫描命令

- 扫描系统中存在的卷组

```bash
vgscan                         

```

- 扫描并列出所有激活的逻辑卷路径

```bash
lvscan                         

```

- 备份卷组元数据到指定文件

```bash
vgcfgbackup -f /tmp/vg_bak     

```
