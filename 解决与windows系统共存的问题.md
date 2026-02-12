# 说明

部分用户有双系统的需求，对于我来说 fedora 在当前来说是我的日常使用系统，但往往有些游戏并不支持启动，当然有一些解决方案，但是我还是希望有些事情简单点，所以下文用于解决一些共存方面的问题。

## 0.建议

- 即使是写了这篇文档，单我个人并不建议一台电脑使用双系统。
- 不要与 windows 公用一个启动分区，有 linux 的启动分区被windows删除的情况。
- 建议两个系统放在独立的磁盘中，因为在配置linux时，有时候需要配置分区，以防操作失误致使 windows 分区被格式化了。

## 1.双系统时间不一致问题

### 1.1 现象

对于北京时间（UTC+8），linux 与 windows 之间会相差八个小时，而且 windows 在启动系统时并不会自行同步时间，需要在设置里手动同步时间，在下一次进入 linux 之后进入 windows 时间又会恢复原状。

### 1.2 原因

因为两个系统对硬件时钟的解释不同，所以导致时间不同步。

- Linux（如 Fedora）： 默认将硬件时钟看作 UTC（协调世界时），然后根据时区（比如北京时间 UTC+8）在系统内进行转换。

- Windows： 默认将硬件时钟直接看作 Local Time（本地时间）。

### 1.3 解决办法

- 修改 Windows 使用 UTC，右键 windows 的徽标打开命令提示符（管理员），或者直接搜索并打开。

- 输入以下命令并回车

这将会修改注册表，告诉 Windows 硬件时钟是 UTC。

```text
reg add HKLM\SYSTEM\CurrentControlSet\Control\TimeZoneInformation /v RealTimeIsUniversal /t REG_DWORD /d 1
```

- 手动在“设置 -> 时间和语言 -> 日期和时间”中手动点击“立即同步”。

- 重启系统并进入 bios 查看当前时间，对于北京时间（UTC+8）是不是与现在相差八个小时，是就说明修改成功了。

- (可选) 修改启动项，将 fedora 设置为第一位，fedora 默认使用的 GRUB 2 这不仅可以启动 windows 也可以启动 linux 。

参考文档

- [debian-cookbook](https://github.com/smgdream/debian-cookbook/blob/main/improve/deb+win.md)

## 2.安全启动导致无法加载启动 U 盘 或者想要在 fedora linux 系统上实现安全启动

### 2.1 现象

在制作好 fedora 的启动 U 盘 后并设置了启动项，但是依旧不能加载。

### 2.2 原因

部分主板是默认开启安全启动，在开启后，主板只会引导经过微软签名的 shim 程序，fedora 是支持安全启动的，但可能因为制作启动 U 盘时可能出现了问题，导致无法启动。

### 2.3 解决办法

- 最简单的就是直接将安全启动关闭，如果有需要可以安装完之后看 [[fedora-linux-secure-boot-实现安全启动]]来实现相关功能。

- 如果你就是要不关闭安全启动去安装系统，那么使用用官方的 U 盘制作工具[Fedora Media Writer](https://docs.fedoraproject.org/en-US/fedora/latest/preparing-boot-media/#fedora_media_writer)。

## 3.开启了 windows 快速启动 linux 无法挂载  ntfs 文件系统的分区

### 3.1 原因

如果 Windows 启用了“快速启动 (Fast Startup)”或处于休眠状态，NTFS 分区会被锁定为只读。

### 3.2 解决办法

进入 windows 系统关闭安全启动。

## 4.linux 与 windows共享文件

### 4.1 两个系统独立

#### 4.1.1 共享磁盘

1.Windows 分享目录，Linux 访问

- Windows 端设置

右键点击你想分享的文件夹 -> 属性 -> 共享 -> 高级共享。

勾选“共享此文件夹”，点击“权限”，确保你的用户有“读取”或“更改”权限。

获取 Windows 的 IP 地址

```DOS
ipconfig
# 查看的是 IPv4 地址 . . . . . . . . . . . . :
```

注意：请记住 Windows 的 IP 地址 和 用户名（如果是微软账户，通常是邮箱或设置里的显示名），网络共享路径的格式是：\\计算机名\分享名 或 \\IP地址\分享名。

- 确定共享磁盘或目录

按下 Win + R，输入 cmd 并回车。

输入以下命令：

```DOS
net share
```

注意：像 C$、ADMIN$ 这种带 $ 符号的是系统内置管理共享，通常不需要去动它们

- 在linux中挂载磁盘

```bash
# 0.安装软件包
sudo dnf install cifs-utils
# 默认已经安装

# 1.创建挂载点
sudo mkdir -p /mnt/win_f

# 2.执行挂载命令 假设你的 Windows IP 是 192.168.x.x：
sudo mount -t cifs //192.168.x.x/F /mnt/win_f -o username=用户名,uid=$(id -u),gid=$(id -g),iocharset=utf8

# //192.168.x.x/F: 这里的 F 就是你 net share 列表里的共享名。

# uid=$(id -u),gid=$(id -g): 自动获取你当前 Fedora 用户的 ID，确保你对挂载后的目录有完全控制权。

# username=: 你的 Windows 用户名。

# 3.在回车之后会要求输入密码

# Password for 用户名@//192.168.x.x/F:

# 密码是你的 Windows 账户登录密码，如果你是用 PIN 码（4位或6位数字）登录 Windows 的，这里的密码通常不是 PIN 码。

# 4.设置自动挂载

# 4.1 创建一个文件，存放 window 的账户信息
vim ~/.smbcredentials
# 输入以下内容
username=用户名
password=你的 Windows 密码

# 4.2 设置文件权限，确保只有属主可以看
chmod 600 ~/.smbcredentials

# 4.3 创建挂载点
sudo mkdir -p /mnt/win

# 4.4 备份fstab并编辑
sudo cp /etc/fstab /etc/fstab.bak
sudo nano /etc/fstab
#在文件末尾添加以下一行内容

//192.168.5.5/F /mnt/win cifs credentials=/home/biyuan/.smbcredentials,uid=1000,gid=1000,vers=3.0,x-systemd.automount,x-systemd.idle-timeout=60,actimeo=3,0 0

# //192.168.x.x/F：Windows 共享路径，IP 是按实际设置，建议将 Windows 设置为静态 IP。
# /mnt/win：本地挂载点，所有 Windows 共享文件将出现在此目录。
# cifs：文件系统类型，CIFS 是 SMB 协议在 Linux 内核中的实现模块。
# credentials=/home/biyuan/.smbcredentials，指定凭据文件路径，避免密码明文暴露在 /etc/fstab 中。
# uid=1000,gid=1000指定挂载后所有文件和目录的属主、属组为 Linux 用户 UID 1000 / GID 1000（通常是第一个普通用户）。
# vers=3.0，强制使用 SMB 3.0 协议通信。
# x-systemd.automount，按需挂载，系统启动时不立即挂载该共享，只有首次访问 /mnt/win 时才触发挂载，网络不可用时不影响开机。
# x-systemd.idle-timeout=60，空闲 60 秒后自动卸载该共享。
# actimeo=3，目录项/文件属性缓存超时时间，单位秒，避免频繁查询服务器，属性缓存保持有效 3 秒。3 秒内再次 ls 直接读缓存，不发网络请求。
# 0 0
#第一个 0：不被 dump 备份（现代系统几乎不用）。
#第二个 0：开机时不执行 fsck 检查（网络文件系统无需 fsck）。
```

注意：如果你的 Windows 账号没有设置密码，SMB 默认是不允许连接的，建议给 Windows 账号设个密码。

2.Linux 分享目录，Windows 访问

- 安装并配置 Samba

```Bash
sudo apt update
sudo apt install samba
```

- 编辑配置文件

编辑 /etc/samba/smb.conf，在文件末尾添加：

```text
[LinuxShare]
   path = /home/username/shared
   available = yes
   browseable = yes
   public = yes
   writable = yes
```

- 设置访问密码并重启

Samba 需要独立的密码库：

```Bash
sudo smbpasswd -a username  # 设置你的 Linux 用户名和 Samba 专用密码
sudo systemctl restart smbd
```

- Windows 端访问

```bash
# 查看当前ip
ip addr show
```

在 Windows 文件资源管理器的地址栏输入： \\192.168.x.x\LinuxShare

#### 4.1.2 共享文件

1. 使用 localsend 软件

- 安装软件

github仓库地址：<https://github.com/localsend/localsend>

- 下载安装包

在 windows 上下载 .exe 后缀的包，在 linux 上下载 .rpm 的包或者使用 flathub 提供的包。
