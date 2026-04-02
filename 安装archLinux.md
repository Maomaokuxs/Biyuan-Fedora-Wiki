# 说明

从 Arch Linux 镜像到安装基本的系统结束，大部分内容基于[Arch Wiki 安装指南](https://wiki.archlinuxcn.org/wiki/%E5%AE%89%E8%A3%85%E6%8C%87%E5%8D%97)。

## 一、准备工作

### 1.下载系统镜像文件

[国内清华源](https://mirrors.tuna.tsinghua.edu.cn/archlinux/)

进入下载页面，按路径 `Index of /archlinux/iso/2026.04.01/` 下载安装镜像，注意路径中的时间每个月更新一次，下载最近版本的即可。

`archlinux-2026.04.01-x86_64.iso`文件为安装镜像，如果需要使用种子下载可以使用`archlinux-2026.04.01-x86_64.iso.torrent`文件

`sha256sums.txt`文件中包含校验值，下载完成可以校验镜像是否完整，在命令提示符中使用以下命令计算哈希值 。

```shell
certutil -hashfile "文件路径" SHA256
```

例如：

```shell
C:\Users\Biyuan>certutil -hashfile "F:\IDM\常规\archlinux-2026.04.01-x86_64.iso" SHA256
SHA256 的 F:\IDM\常规\archlinux-2026.04.01-x86_64.iso 哈希:
f14bf46afbe782d28835aed99bfa2fe447903872cb9f4b21153196d6ed1d48ae
CertUtil: -hashfile 命令成功完成。
```

然后使用瞪眼法比较上下有什么不同:

```shell
C:\Users\Biyuan>f14bf46afbe782d28835aed99bfa2fe447903872cb9f4b21153196d6ed1d48ae

C:\Users\Biyuan>f14bf46afbe782d28835aed99bfa2fe447903872cb9f4b21153196d6ed1d48ae
```

### 2.准备安装介质

Arch Linux 的ISO文件可以被制作成多种类型安装介质，如 U 盘、光盘和带有 PXE 的网络安装映像。我建议使用 U盘，第一次安装这个最为方便。
我建议使用 [Ventoy](https://www.ventoy.net/cn/) ，如果是在 Windows 使用，在官网下载 .exe 后缀的文件安装后，将 U 盘插上电脑,打开图像化界面后使用默认的配置，选择将要使用的 U 盘，注意这将会格式化 U 盘，先将里面的重要数据备份，接着直接将下载好的 ArchLinux 镜像复制进 U 盘，重启系统后进入 Bios 中将首个启动项设置为 U 盘，建议在 Windows 中关闭快速启动。

### 3.预留磁盘空间

如果直接全盘安装的话并不需要这一步。

在 Windows 系统磁盘管理中，右键空间充足的分区，点击压缩卷，至少预留 20 GB的空间。想要极限安装，根据 Arch Wiki 指南中可以只预留 2 GB 的空间。

### 4.启动到 live 环境

Arch Linux 安装镜像不支持 UEFI 安全启动（Secure Boot）功能。如果要引导安装介质，需要禁用安全启动。如果需要，可在完成安装后重新配置。这需要在 Bios 中进行设置，怎么进入 Bios 各家主板不一，拿华硕 TUF 系列主板来说，开机时多次按下 `F2` 或者 `Del` 即可，进入后需要将 U 盘作为启动项即可接着将 root 身份登录进入一个虚拟控制台，默认的 Shell 是 Zsh。

如果在启动后界面不符合屏幕的分辨率或者需要调整刷新率，可以在进入 GRUB 时点击 e 键 进入编辑模式，在 Linux 这行的末尾添加 `video=1920x1080@120`，@前为分辨率，@后为刷新率。

如果字体太小看不请可以执行下面的命令

```shell
setfont ter-132b
```

---

## 二、安装系统

### 1.网络配置

建议使用有线网络，最为方便，如果需要使用无线网络也有办法。

  1.1 确保系统已经列出并启用了网络接口

  ```shell
  ip link
  ```
  
  1.2 有线网络
  
  连接网线就可以了。

  1.3 无线网络
  
  列出网络设备名称

  ```shell
  iwctl device list
  ```

  连接网络
  
  ```shell
  iwctl station wlan0 connect SSID --passphrase passwd
  ```
  
  `wlan0`  为接口名。
  `SSID`   为 WIFI 名称。
  `passwd` 为 WIFI 密码。

  1.4 测试网络连接
  
  ```shell
  ping -c 3 ping.archlinux.org  
  ```

### 2.更新系统时间

为确保软件包签名校验成功以及防止 TLS 证书错误，Live 系统需要准确的时间，为此 systemd-timesyncd 默认启用，也就是说当系统已经创建互联网连接后，系统时间将自动同步。

```shell
timedatectl
```

### 3.创建硬盘分区并格式化

  3.1 列出硬盘
  
  ```shell
  lsblk
  ```
  
  3.2 创建硬盘分区
  
  3.2.1 使用 fdisk 命令,下面拿 /dev/nvme0n1 来举例 ，如果时机械硬盘或者 U盘 会分配到的路径 /dev/sda1 。

  注意，这一步需要非常的谨慎，如果不明白分区这个概念，十分危险，尽量不要在安装了 Windows 系统盘上进行操作，如果需要至少可以在 Windows 中压缩出空闲空间，不要对已有的分区进行任何操作。如果是全盘安装，直接把所有分区删除即可。

  ```shell
  fdisk /dev/nvme0n1
  # 进入交互界面，界面左下角会出现 Command (m for help): 

  Command (m for help): m
  # 打印帮助手册

  Command (m for help): n
  # 创建新分区表

  Partition number (1-128，default 1)：
  # 通常回车即可，设置分区编号，默认按已有的编号按顺序排列，如果分区表中缺少一个数字，那么会见缝插针。
  
  First sector (2048-1000215182,default 2048):
  # 通常回车即可，这是设置初始扇区

  Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-1000215182): +1G
  # 为当前分区配置 1G 硬盘空间。

  # 以此类推创建所需要的分区 

  Command (m for help): p 
  # 打印当前创建的所有分区，在这一步并没有将分区写入硬盘
  # 这一步必须检查清楚是否和自己预想的一致

  Command (m for help): w
  # 写入当前的分区方案

  ```
  
  3.2.2 使用 cfdisk 命令
  
  这个工具会比 fdisk 更见简单，但是不能通过文本完全的描述，所以此处大概说明。

  ```shell
  cfdisk /dev/nvme0n1
  ```

  使用该命令之后可以使用上下左右控制选项，按会回车确认，选择 `删除` 可以删除分区，选择 `创建` 可以在空闲空间中创建新分区，接着使用 `类型` 为分区加上标签，最后确认无误之后使用 `写入` 将分区表写入硬盘。

  3.3 检查分区情况

  ```shell
  lsblk
  ```

  3.4 我使用的分区方案
  
  - 方案一：方便使用btrfs快照功能

  | 目录 | 分区 | 文件系统 | 子卷 |
  | --------- | --------- | ------ | ------ |
  | **/efi** | /dev/nvme0n1p1 | fat32 | |
  | **/** | /dev/nvme0n1p2 | btrfs | @ |
  | **/home** | /dev/nvme0n1p2 | btrfs | @home |

  - 方案二：使用 LVM 管理 / 和 /home 方便调整分区大小以及跨磁盘容

  | 目录 | 分区 | 文件系统 |
  | --------- | --------- | ------ |
  | **/boot/efi** | /dev/nvme0n1p1 | fat32 |
  | **/boot** | /dev/nvme0n1p2 | ext4 |
  | **/** | /dev/mapper/vgroup0-lvol0 | ext4 |
  | **/home** | /dev/mapper/vgroup0-lvol1 | ext4 |

  3.5 格式化硬盘分区

  这一步要跟据当前的分区方案去调整，因为有两个方案，所以我在这一部分分开叙述。
  
  3.5.1 方案一
  
  ```shell
  mkfs.fat -F 32 /dev/nvme0n1p1
  # 格式胡
  ```


