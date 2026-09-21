# 说明

在需要备份btrfs子卷的场景中有一些帮助，因为 home 目录不能被占用，环境是tty，以下的命令都是针对 root 账户，如果你需要交换 root子卷，可以是用普通用户登录tty，如果home子卷替换失败只不过没有了用户文件，但是要替换根分区我并不建议这样做，系统坏了要么使用 snapper + btrfs-assistant 执行回滚，要么直接重装。

## 0.登录 root 账户

```bash
# 1.开机后，不要登陆普通用户进入tty
ctrl + alt + F2 ～ F6
# 如果不计划替换 home 目录可以直接登陆图形化界面使用普通用户

# 2.输入用户名root和密码登录root账户
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

## 4.创建只读子卷

```bash
sudo btrfs subvolume snapshot -r /mnt/old/snapper_rw /mnt/old/snapper
```

## 5.发送与接收子卷

```bash
# 1.本地发送
sudo btrfs send /mnt/old/snapper/ | sudo btrfs receive /mnt/new/

# 2.局域网发送

# 2.1 临时启用 SSH 服务
sudo systemctl start sshd

# 2.2 确定接收端的局域网 IP
ip a 

# 例如：
inet 192.168.5.7/24 brd 192.168.5.255 scope global noprefixroute eno1
IP 就是 192.168.5.7

# 2.4 测试连接
# 在发送端测试
ssh biyuan@192.168.5.7

# 2.5 在接收端解决权限问题
  # 方式一：更改文件夹权限
sudo chown biyuan:biyuan /mnt/new

  # 方式二：临时配置 sudoers
  # 编辑配置文件（修改前建议备份）
visudo

  # 取消下面一行的注释
  %wheel ALL=(ALL)       NOPASSWD: ALL
  # 保存并退出
  # 发送完成后可以重新加上注释

# 3. 通过 SSH 管道发送
sudo btrfs send /mnt/old/snapper | ssh biyuan@192.168.5.7 "sudo btrfs receive /mnt/new/"
# 发送完成后输出：BTRFS_IOC_SEND returned 0
```

## 6.替换 /home 下的子卷

```bash
# 1.备份当前的 home 子卷
mv /mnt/new/@home/ /mnt/new/@home_backup

# 2. 基于旧子卷创建一个可读写的子卷
btrfs subvolume snapshot /mnt/new/snapper/ /mnt/new/@home
```

## 7.尝试挂载并检查当前的挂载情况

```bash
# 1.尝试挂载
sudo systemctl daemon-reload
mount -a

# 2.检查挂载情况
df -h
```

## 8.登出 root 账户，登录普通账户

```bash
# 1.卸载分区
umount -R /mnt

# 2.登出 root 账户 
exit

# 3.输入用户名和密码登录普通账户
```

## 9.重启

```bash
reboot
```
