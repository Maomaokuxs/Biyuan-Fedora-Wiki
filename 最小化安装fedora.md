# 说明

## 配置终端字体

### 1.临时更改

```bash
sudo setfont latarcyrheb-sun32
```

### 2.永久更改（重启依然有效）

编辑 /etc/vconsole.conf 文件：

```bash
sudo vi /etc/vconsole.conf
将 FONT 这一行修改为：
Plaintext
FONT="latarcyrheb-sun32"
保存退出。这样下次开机时，终端字体就会很大。
```

---

## 配置GRUB

### 1.编辑 GRUB 配置

```bash
sudo vi /etc/default/grub

找到 GRUB_CMDLINE_LINUX 这一行，在末尾添加（注意在引号内）：
video=1920x1080@60
# 分辨率和刷新率根据实际需求设置，如果不添加通常没有问题。

# 记忆上一次所选启动项
GRUB_SAVEDEFAULT="true"
```

### 2.更新 GRUB

```bash
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

---

## 更新系统

```bash
sudo dnf upgrade
```

---

## 配置snapper

[[snapper-实现快照功能]]

注：在自由的系统中，需要自己确定每次执行的命令是什么功能，如果不确定使用系统快照是明智的选择。

---

## 配置显卡驱动

### 1.NVIDIA显卡驱动

[[安装英伟达显卡驱动并开启硬件编解码]]

### 2.AMD 和 Intel 显卡驱动

通常情况下这些可以即插即用。

- 安装内核头文件和开发工具

```bash
sudo dnf install kernel-devel kernel-headers gcc make dkms acpid libglvnd-glx libglvnd-opengl libglvnd-devel pkgconfig
```

- 核心软件包（AMD 与 Intel）

Vulkan 及基础加速支持：

```bash
sudo dnf install mesa-vulkan-drivers vulkan-loader mesa-libGLU libva-utils
```

- AMD（视频加速）

替换默认驱动为 freeworld 版本以获得完整编解码器支持（H.264/HEVC）：

```bash
sudo dnf swap mesa-va-drivers mesa-va-drivers-freeworld mesa-vdpau-drivers mesa-vdpau-drivers-freeworld
```

- Intel（视频加速）

  - 英特尔较新 GPU（第 11 代及更高版本）

    ```bash

    # 适用于较新英特尔 GPU 的视频加速
    sudo dnf install intel-media-driver
    ```

  - 英特尔旧款 GPU（第 10 代及更早版本）

    ```bash
    # 适用于旧款英特尔 GPU 的视频加速
    sudo dnf install libva-intel-driver
    ```

- 参考文档：

  - [Fedora-Noble-Setup](https://github.com/wz790/Fedora-Noble-Setup?tab=readme-ov-file#first-things)

---

## 配置fedora默认开机动画

### 1.安装 Plymouth 核心组件

安装启动动画程序以及 Fedora 的默认主题包：

```Bash
sudo dnf install plymouth plymouth-scripts
```

### 2.搜索主题包并安装

```bash
dnf search plymouth-theme

sudo dnf install fedora-logos plymouth-theme-spinner
```

### 3.设置为 bgrt 主题

```bash
sudo plymouth-set-default-theme bgrt -R
```

### 4.打包模块

```bash
sudo dracut --force
# 需要将 Plymouth 模块直接打包进初始内存盘，这样它才能在根文件系统挂载前启动。
```

### 5.修改 GRUB 配置

```Bash
sudo vi /etc/default/grub

# 找到 GRUB_CMDLINE_LINUX 这一行，在引号内添加 rhgb（Red Hat Graphical Boot）和 quiet：
GRUB_CMDLINE_LINUX="... rhgb quiet"
# rhgb: 开启图形启动界面。
# quiet: 隐藏那些刷屏的内核检测日志。

```

### 6.更新 GRUB 配置文件

```bash
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

### 7.配置显卡驱动加载顺序

配置系统在 initramfs 阶段就强制加载 NVIDIA 驱动：

- 创建或编辑 dracut 配置文件：

```Bash
sudo vi /etc/dracut.conf.d/nvidia.conf
```

- 添加以下内容：

```txt
force_drivers+=" nvidia nvidia_modeset nvidia_uvm nvidia_drm "
```

## 安装niri

```bash
# 核心：窗口管理器、终端、状态栏、壁纸、通知、应用启动器
sudo dnf install niri kitty waybar swaybg mako rofi-wayland
# 基础：文件管理器、图片查看器
sudo dnf install nautilus loupe 
# 门户支持：确保 OBS 录屏和屏幕共享正常工作
sudo dnf install xdg-desktop-portal-gnome xdg-desktop-portal-wlr
```

## 配置 Greetd + Tuigreet 作为窗口管理器

这是目前 Wayland 用户最主流的选择。它运行在 TTY 模式下，外观极简，却能完美支持 Wayland 会话启动。

### 1.安装软件包

```Bash
sudo dnf install greetd tuigreet
```

### 2.配置 Greetd

- 编辑 /etc/greetd/config.toml，将默认启动项改为 tuigreet：

```Bash
sudo vi /etc/greetd/config.toml
```

在 greetd/tuigreet 中启动 Niri 出现黑屏，通常是因为显卡驱动环境变量没有传递给 Niri，或者是 Niri 在启动时无法获取 座席（Session）控制权。

我使用的是 NVIDIA 显卡，其他厂商的显卡可能有区别

1. 修改 tuigreet 启动指令（解决环境传递问题）

    编辑 /etc/greetd/config.toml，修改 command 行：

    ```Ini, TOML
    [default_session]
    # 使用 bash -l (login shell) 来启动，确保加载 /etc/profile 和 ~/.bash_profile 中的驱动变量
    command = "tuigreet --time --remember --cmd /usr/bin/niri"

    user = "greetd"
    ```

2. 检查 NVIDIA 权限与内核参数

    如果 D-Bus 没问题但依然黑屏，通常是显卡没“醒”。

    - 确认驱动加载：在 TTY 执行 lsmod | grep nvidia。如果没有输出，说明驱动没加载。

    - 强制设置内核参数：
    确保 /etc/default/grub 中有以下三项：

    ```text
    nvidia-drm.modeset=1 nvidia_drm.fbdev=1 ibt=off
    fbdev=1: 解决部分 NVIDIA 卡在 Wayland 下的黑屏/闪烁问题。

    ibt=off: 某些较新的内核在 NVIDIA 上需要此参数才能正常启动图形环境。
    ```

    - 更新 GRUB：

    ```bash
    sudo grub2-mkconfig -o /boot/grub2/grub.cfg。
    ```

3. 给 greeter 用户增加显卡权限

    greetd 的默认用户是 greetd，如果这个用户没有权限访问视频设备，也会导致启动失败。

    - 执行以下命令：将其加入 video 和 render 组（为了 NVIDIA 驱动权限）

    ```bash
    sudo usermod -aG video,render greeter

    sudo systemctl start greetd
    ```

    - 查看当前默认目标

    ```bash
    systemctl get-default
    ```

    - 如果显示 multi-user.target，将其修改为 graphical.target

    ```bash
    sudo systemctl set-default graphical.target，我配置了这个就可以进入greeter了，
    ```

## 修改主机名

```bash
sudo hostnamectl set-hostname fedora
# fedora 改成需要的名字
```

## （可选）将旧 @home 子卷复制到新系统

### 将 root 和 var 子卷重命名为 @root @var

需要注意的是要重新生成 grub 配置文件，不然不能进入系统，下面是临时的解决办法。

1. 手动修改 GRUB 临时进入系统

    - 重启电脑，在 GRUB 菜单界面（就是你之前调大字体的那个）按 e 进入编辑模式。

    - 找到以 linux 开头的那一行（末尾通常有 rhgb quiet）。

    - 找到 rootflags=subvol=root（或你之前的名字），将其改为 rootflags=subvol=@root。

## 安装必要的字体

- 在浏览器中下载字体解压后复制到 /usr/share/fonts 目录下

```text
Iosevka nerd font
```

- 刷新字体缓存

```bash
fc-cache -fv
```

## 安装必要的软件

```bash
fastfetch
swww
waypaper
rofi
clash-verge
waypaper
hyprlock
hypridle
starship
firfox
nvim
codium
hellwal
PackageKit-Qt6
dolphin
mako
splayer
steam
bilibili
plasma-discover
```
