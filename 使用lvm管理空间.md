# 说明

这一部分主要是一些使用 LVM 常用命令

## 1. 查看当前的LVM结构

``` bash
# 1.查看物理卷（PV）情况
sudo pvdisplay

# 2.查看卷组（VG）情况
sudo vgdisplay vgroup0

# 3.查看逻辑卷（LV）情况
sudo lvdisplay
```
