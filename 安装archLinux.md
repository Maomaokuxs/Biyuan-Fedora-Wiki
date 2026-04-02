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

### 3.创建硬盘分区、格式化和挂载

  3.1 列出硬盘
  
  ```shell
  lsblk
  ```
  
  3.2 创建硬盘分区
  
  3.2.1 使用 fdisk 命令,下面拿 /dev/nvme0n1 来举例 ，如果时机械硬盘或者 U盘 会分配到的路径 /dev/sda1 。

  注意，这一步需要非常的谨慎，如果不明白分区这个概念，十分危险，尽量不要在安装了 Windows 系统盘上进行操作，如果需要至少可以在 Windows 中压缩出空闲空间，不要对已有的分区进行任何操作。如果是全盘安装，直接把所有分区删除即可。

  如果要使用我的分区方案，只需要分两个区，大小再下面的表格中。
  
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
  
  3.4.1 方案一：方便使用btrfs快照功能

  | 目录 | 分区 | 文件系统 | 子卷 | 大小 |
  | --------- | --------- | ------ | ------ | ------ |
  | **/efi** | /dev/nvme0n1p1 | fat32 | | 100MB |
  | **/** | /dev/nvme0n1p2 | btrfs | @ | 不限 |
  | **/home** | /dev/nvme0n1p2 | btrfs | @home | 不限 |

  3.4.2 方案二：使用 LVM 管理 / 和 /home 方便调整分区大小以及跨磁盘容

  | 目录 | 分区 | 文件系统 | 大小 |
  | --------- | --------- | ------ | ----- |
  | **/boot** | /dev/nvme0n1p1 | f32 | 1-2GB |
  | **/** | /dev/mapper/vgroup0-lvol0 | ext4 | 至少60GB |
  | **/home** | /dev/mapper/vgroup0-lvol1 | ext4 | 至少40GB |

  3.5 格式化硬盘分区

  这一步要跟据当前的分区方案去调整，因为有两个方案，所以我在这一部分分开叙述。
  
  3.5.1 方案一
  
  ```shell
  mkfs.fat -F 32 /dev/nvme0n1p1
  # 格式化分区1为fat32
  
  mkfs.btrfs /dev/nvme0n1p2 
  # 格式化分区2为btrfs
  
  lsblk -f 
  # 查看当前分区及格式化情况
  
  mount -t btrfs -t btrfs /dev/nvme0n1p2 /mnt
  # 临时挂载

  btrfs subvolume create /mnt/@ 
  btrfs subvolume create /mnt/@home
  # 创建 @ 和 @home 两个btrfs子卷
  
  umount /mnt 
  # 取消挂载

  mount -t btrfs -o subvol=/@,compress=zstd /dev/nvme0n1p2 /mnt
  mount --mkdir -t btrfs -o subvol=/@home,compress=zstd /dev/nvme0n1p2 /mnt/home
  mount --mkdir /dev/nvme0n1p1 /mnt/efi 
  # 挂载分区到指定挂载点
  ```

  3.5.2 方案二
  
  ```shell
  mkfs.fat -F 32 /dev/nvme0n1p1
  # 格式化分区1为fat32
  
  pvcreate /dev/nvme0n1p2
  # 创建物理卷
  
  vgcreate vg0 /dev/nvme0n1p2
  # 创建逻辑卷组

  lvcreate -L 60G -n root vg0
  lvcreate -l 100%FREE -n home vg0
  # 创建逻辑卷root 以及 home ，root 大小为 60GB，剩余分配给
  
  mkfs.ext4 /dev/vg0/root 
  mkfs.ext4 /dev/vg0/home
  
  lsblk -f 
  # 检查分区及格式化情况
 
  mount /dev/vg0/root /mnt
  # 挂载根分区

  mkdir -p /mnt/{boot，home}
  # 创建挂载点
  
  mount /dev/nvme0n1p1 /mnt/boot
  # 挂载引导分区

  mount /dev/vg0/home /mnt/home
  # 挂载家目录
  ```

### 4.正式安装系统
  
  4.1 安装系统及必要软件包
  
  ```shell
  pacstrap -K /mnt/ base base-devel linux linux-firmware btrfs-progs networkmanager vim sudo amd-ucode
  # 如果是 Intel 将 amd-ucode 改成 intal-ucode
  ```
  
  4.2 自动生成fstab
  
  ```shell
  genfstab -U /mnt > /mnt/etc/fstab
  
  cat /mnt/etc/fstab
  # 打印生成的fstab内容到终端，检查挂载项是否有问题
  ```
  
### 5.基本配置系统
  
  5.1 进入新安装的系统

  ```shell
  arch-chroot /mnt
  ```
  
  5.2 设置时区
  
  ```shell
  ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
  hwclock --systohc
  ```

  5.3 本地化
  
  ```shell
  vim /etc/locale.gen
  # 可以使用点击/，加上en_US.UTF-8 搜索
  # 点击 i 进入编辑模式
  # 方向键可以控制光标
  # 点击 esc 推出编辑模式
  # 取消 en_US.UTF-8 及 zh_CN.UTF-8 前的注释
  # 输入 :wq 保存并推出

  locale-gen
  # 生成本地化文件

  vim /etc/locale.conf
  # 设置本地化
  # 添加 LANG=en_US.UTF-8 
  ```

  5.4 配置主机名
  
  ```shell
   vim /etc/hostname
  # 输入需要的主机名
  ```

  5.5 配置 root 密码

  ```shell
  passwd root  
  # 连续输入两次密码
  ```
  
  5.6 安装系统引导器

  ```shell
  pacman -S grub efibootmgr
  grub-install --target=x86.64-efi --efi-directory=/efi --boot-directary=/efi --botloaser-id=arch
  ```
  
  5.7 将 /efi/grub 链接至 /boot/efi
  
  ```shell
  ln -s /efi/grub /boot/grub
  ```
  
  5.8 配置双系统

  ```shell
  pacman -S os-prober exfat-utils
  
  vim /etc/default/grub 
  # 将 GRUB_DISABLE_OS_PROBER=false取消注释 
  ```

  5.9 生成启动项及启动流程

  ```shell
  grub-mkconfig -o /boot/grub/grub.cfg 
  ```
  
  5.10 配置 zram 内存压缩和交换空间

  ```shell
  pacman -S zram-generator
  # 自动化管理zram工具
  ```

  ```shell
  vim /etc/systemd/zram-generator.conf
  # 写入配置文件 
  [zram0]
  zram-size = ram
  compression-algorithm = zstd
  ```

  ```shell
  vim /etc/default/grub
  # 在 GRUB_CMDLINE_LINUX_DEFAULT=""中添加 zswap.enabled=0
  grub-mkconfig -o /boot/grub/grub.cfg
  ```

  5.11 重启系统

```shell
exit
# 退出 chroot

reboot
# 重启系统
```

---

## 三、启动系统

进入选择 arch linux 的启动项，部分主板会自动切换不需要设置。

使用 root 名加配置的密码进入系统。

### 1.配置 networkmanager 自启动

```shell
systemctl enable --now NetworkManager
```

### 2.连接网络

有线网络将自动连接，无线使用下面命令。

```shell
nmtui
```

### 3.更新软件包

```shell
pamcan -Syu
```

### 4.添加普通用户

```shell
useradd -G wheel -m biyuan
# biyuan是用户名

passwd biyuan
# 配置用户 biyuan 的密码

visudo 
# 取消 %wheel ALL=(ALL:ALL) ALL注释 
```

### 4.注销 root 账户

```shell
exit
```

### 5.登录普通用户

输入刚刚配置的普通用户的信息。

### 4.安装 fastfetch

```shell
sudo pamcan -S fastfetch
# 安装软件包

fastfetch
# 输出当前系统信息
```
