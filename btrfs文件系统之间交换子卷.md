# 说明

在需要备份btrfs子卷的场景中有一些帮助，因为 home 目录不能被占用，环境是tty，以下的命令都是针对 root 账户。

## 0.登录 root 账户

```bash
# 1.开机后，不要登陆普通用户进入tty
ctrl + alt + F2 ～ F6
# 如果不计划替换 home 目录可以直接登陆图形化界面使用普通用户

# 3.输入用户名root和密码登陆root账户
```

## 1.创建挂载点

```bash
mkdir /mnt/old
mkdir /mnt/new
```

## 2.卸载 home

```bash
umount /home
```

## 3.挂载分区

```bash
mount /dev/nvme0n1p3 /mnt/old/
mount /dev/nvme0n1p4 /mnt/new/
```

## 4.发送与接受子卷

```bash
sudo btrfs send /mnt/old/snapper/ | sudo btrfs receive /mnt/new/
```

## 5.替换 /home 下的子卷

```bash
# 1.备份当前的 home 子卷
mv /mnt/new/@home/ /mnt/new/@home_backup

# 2. 基于旧子卷创建一个可读写的子卷
btrfs subvolume snapshot /mnt/new/snapper/ /mnt/new/@home
```

## 6.尝试挂载并检查当前的挂载情况

```bash

# 1.尝试挂载
mount -a

# 2.检查挂载情况
df -h
```

## 7.登出 root 账户，登录普通账户

```bash
# 1.卸载分区
umount -R /mnt

# 2.登出 root 账户 
exit

# 3.输入用户名和密码登录普通账户
```

## 8.重启

```bash
reboot
```
