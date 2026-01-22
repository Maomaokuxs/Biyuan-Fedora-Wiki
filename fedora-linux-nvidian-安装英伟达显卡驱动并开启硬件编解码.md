# 0.注意当前作者设备环境

- fedora版本：Fedora Linux 43 (KDE Plasma Desktop Edition) x86_64

- GPU: NVIDIA GeForce RTX 4060

- WM: KWin (Wayland)

# 1. 启用自由和非自由软件仓库

```bash
sudo dnf install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
```

```bash
*在 Fedora 41 及更高版本上：*
sudo dnf config-manager setopt fedora-cisco-openh264.enabled=1
*在 Fedora 40 及更早版本上，命令如下：*
sudo dnf config-manager --enable fedora-cisco-openh264
```

# 2. 检查GPU型号

```bash
/sbin/lspci | grep -e VGA
```

可以看到输出的NVIDIA显卡信息；

如果设备包含核显则会显示核显的信息；

如果NVIDIA GPU 没有用作显示输出，比如笔记本或者使用核显输出独显计算的情况，运行这个命令

```bash
/sbin/lspci | grep -e 3D
```

根据 GPU 型号查询可用驱动的最新版本号。

参考文档：

- [Configuration - RPM Fusion](https://rpmfusion.org/Configuration#Installing_Free_and_Nonfree_Repositories)
- [Fedora 安装 NVIDIA 驱动的方法 （Fedora 42、43）](https://zhuanlan.zhihu.com/p/1904281445544989652)

# 3.检测安全启动并安装密钥

## 3.1 检测安全启动（Secure Boot）并安装密钥

通过此命令检查是否启用 Secure Boot

```bash
# 检查是否使用UEFI启动，如果输出为 UEFI mode 才有可能启用了安全启动
[ -d /sys/firmware/efi ] && echo "UEFI mode" || echo "Legacy BIOS mode"

# 安装依赖
sudo dnf install mokutil

# 检测
mokutil --sb-state
```

如果显示 `SecureBoot enabled` 则说明系统启用了 安全启动，需要进行下面的签名步骤，如果不想了解安全启动直接跳至3.4。

## 3.2 安全启动简介及说明

- 安全启动是一项自 Fedora 18 及更高版本引入的功能，旨在保护 EFI 固件下的启动阶段，并被 Windows 10 及以上系统所要求。

- 从 Fedora 36 开始，akmods 软件包支持使用自生成的密钥自动为本地构建的内核模块 (kmod) 进行签名。此密钥必须导入到 EFI 固件中（您需要拥有访问 EFI 固件的权限）。

- 无需禁用安全启动（甚至不需要切换到 BIOS 兼容模式）。

## 3.3 保护密钥

- 由于安全启动密钥存储在本地的计算机上（默认位于 /etc/pki/akmods 目录），您可能需要考虑对根文件系统进行加密，以保护该密钥。请务必将此视为一项强制性要求，或者考虑将密钥转移到外部（且安全）的位置，甚至可以使用硬件令牌。

## 3. 4 导入密钥

- 安装以下工具：

```bash
sudo dnf install kmodtool akmods mokutil openssl
```

步骤如下所述。更多信息请参考 /usr/share/doc/akmods/README.secureboot。

- 使用默认值生成密钥：

```bash
sudo kmodgenca -a
```

- 现在您需要在 MOK 中注册公钥，使用以下命令注册带有证书的新密钥对：

```bash
sudo mokutil --import /etc/pki/akmods/certs/public_key.der
```

Mokutil 会要求生成一个密码来注册公钥。您很快将需要这个密码。

需要重启系统以便 MOK 注册新的公钥。

```bash
systemctl reboot
```

- 在下一次启动时，系统会启动 MOK 管理程序，您需要选择“Enroll MOK”。
选择“Continue”以注册密钥，或选择“View key 0”查看已注册的密钥。

选择“Yes”确认注册。

- 系统会提示您输入之前生成的密码。

警告：此时键盘布局被映射为 QWERTY！

- 新密钥注册成功后，系统会提示您重启。

- 更新 BIOS/EFI

- 当更新 BIOS / EFI 时，请注意您可能需要重新导入安全启动密钥。您可以使用导入命令来完成：

```bash
sudo mokutil --import /etc/pki/akmods/certs/public_key.der
```

参考文章：

[Secure Boot](https://rpmfusion.org/Howto/Secure%20Boot)

# 4. 安装驱动

Fedora 的 RPM Fusion 仓库提供 akmod-nvidia 驱动，akmod 指的是 Automatic Kernel Module，基于 DKMS (Dynamic Kernal Module Support) ，能在系统内核升级时自动触发Nvidia内核模块重新编译，自动保证驱动与内核兼容，免去每次更新内核都要手动编译Nvidia内核模块的麻烦。另外 akmod-nvidia 自动处理相关依赖，不会与系统配置冲突，使用官方脚本安装需要提前手动安装依赖 gcc 、kernel-devel 等，因此 Fedora 推荐使用 RPM Fusion 的驱动。 

- 安装前更新系统

```bash
sudo dnf update -y 
```

- 如果内核更新还需要重启系统应用内核更新

```bash
systemctl reboot

sudo dnf install akmod-nvidia

# 如果需要使用 cuda/nvdec/nvenc (比如torch/tensorflow)
sudo dnf install xorg-x11-drv-nvidia-cuda 

# 把驱动标记为用户安装，避免 dnf autoremove 意外删除驱动
sudo dnf mark user akmod-nvidia
```

请注意，这里安装时需要根据显卡型号选择正确的驱动版本号（一般来说十年内的卡都能用最新驱动），上面的命令安装的是最新驱动，如果需要安装旧版驱动，请搜索对应的包名 比如 xorg-x11-drv-nvidia-470xx akmod-nvidia-470xx xorg-x11-drv-nvidia-470xx-cuda ，请注意旧版需要安装的是三个包。搜索命令可用例如 dnf search akmod-nvidia。

- 完成后运行命令检测内核模块是否安装

```bash
modinfo -F version nvidia
# 输出版本号比如 
# 570.144
```

如果输出错误信息或者没有输出，请等待（官方文档的描述是最长5分钟）模块编译完成再尝试。

可以使用这个命令查看是否编译完成 `systemctl list-jobs` ，如果有akmod.service说明还在编译中，等待任务结束消失驱动才可用。（试了几次都没输出，如果是不使用安全启动的话直接启动没什么问题，其实驱动程序已经正常编译和安装了。需要开启安全启动，确保万无一失不建议直接重启，执行下面出现错误后的操作，在完全确定后我会考虑删除上面这段说明。）

- （使用了安全启动后，不建议直接重启）重启系统

```bash
systemctl reboot
```

- 如果重启进入桌面直接黑屏了,cpu带核显的话还是能进桌面，但是会出现模块未加载的提示，解决办法参考以下操作，注意操作顺序

```bash
# 1. 验证驱动是否安装成功，有输出版本号即成功，报错即失败，失败则进行下面操作
nvidia-smi

# 2. 更新软件包
sudo dnf upgrade -y

# 3. 重装相关软件包
sudo dnf reinstall akmod-nvidia

# 4.检查内核模块是否加载
lsmod | grep nvidia

# 5.检查驱动安装情况
rpm -qa | grep nvidia
sudo dnf list installed | grep nvidia

# 6. (可选) 停止的显示管理器
#gnome 桌面环境
sudo systemctl stop gdm

# kde桌面环境(未来可能会更换)
sudo systemctl stop sddm

# 7.强制重建 NVIDIA 内核模块
sudo akmods --rebuild --force
sudo dracut --force 

# 8.检查 Secure Boot 状态
mokutil --sb-state

# 9.如果启用了 Secure Boot
sudo kmodgenca -a --force
sudo mokutil --import /etc/pki/akmods/certs/public_key.der
# 在 /usr/share/doc/akmods/README.secureboot 文件中有英文说明。

# 10.重启
sudo reboot

# 11.重新查看显卡驱动状况
nvidia-smi
```

- 如果输出正常，能看到驱动和显卡信息则说明安装完成。

# 5. 其他依赖

- **NVENC / NVDEC**

`ffmpeg` 硬件加速解编码会用到

```bash
sudo dnf install xorg-x11-drv-nvidia-cuda-libs
```

- **VDPAU / VAAPI**

视频播放器的硬件解码会用到

```bash
sudo dnf install nvidia-vaapi-driver libva-utils vdpauinfo
```

- 处理不能正确安装`nvidia-vaapi-driver`问题

```bash
# 0.错误示例
sudo dnf install nvidia-vaapi-driver
仓库更新和加载中:
仓库加载完成。
Failed to resolve the transaction:
No match for argument: nvidia-vaapi-driver
# 也就是说不能找到相关软件包

# 1. 重新执行第一节启用自由和非自由软件仓库中的命令

# 2. 更新缓存
sudo dnf makecache

# 3. 再次安装
sudo dnf install nvidia-vaapi-driver
# 如果再次出现上面的错误那就说明仓库中并不包含相关软件包
# 使用下面命令验证
sudo dnf search nvidia-vaapi-driver
# 发现 RPM Fusion 尚未为 Fedora 43 正式打包 nvidia-vaapi-driver 该包通常滞后于新 Fedora 版本几周甚至几个月。
# 由此，可以自己先手动编译，等 RPM Fusion 适配打包后再删除自己手动编译的驱动，安装统一打包维护的 nvidia-vaapi-driver 驱动。

# 4.手动编译安装
# 项目仓库地址：[github.com/elFarto/nvidia-vaapi-driver](https://github.com/elFarto/nvidia-vaapi-driver)

# 4.1 首先安装编译依赖：
sudo dnf install git gcc make cmake pkg-config libva-devel vulkan-devel

# 4.2 拉取 nvidia-vaapi-driver 源码并进入对应目录：
git clone https://github.com/elFarto/nvidia-vaapi-driver.git
cd nvidia-vaapi-driver

# 4.3 安装编译所需依赖：
sudo dnf install meson libva-devel gstreamer1-plugins-bad-freeworld nv-codec-headers libdrm-devel gstreamer1-plugins-bad-free-devel

# 4.4 使用 Meson 初始化构建环境：
meson setup build
# Meson 自动检测到所有依赖（包括 ffnvcodec, libva, libdrm 等）。
# 示例输出信息：

Run-time dependency ffnvcodec found: YES 13.0.19.0
Run-time dependency libva found: YES 1.22.0
...
Build targets in project: 1
# 主要看每一行后面是否有yes,如果不明白问AI少了什么依赖。

# 错误示例
$ meson setup build

Directory already configured. Just run your build command (e.g. ninja) and Meson will regenerate as necessary. Run "meson setup --reconfigure" to force Meson to regenerate. If build failures persist, run "meson setup --wipe" to rebuild from scratch using the same options as passed when configuring the build.

# 如果配置已存在但缺少依赖需要重新构建目录
meson setup build --reconfigure

# 4.5 进行编译安装驱动：
sudo meson install -C build

# 5.验证是否安装成功
# 5.1 使用ls命令
ls -a /usr/lib64/dri | grep "nvidia_drv_video.so"
# 得到输出为：vidia_drv_video.so 说明驱动成功安装。

# 5.2 使用vainfo命令
vainfo
# 输出示例：
Trying display: wayland  
libva info: VA-API version 1.22.0  
libva info: Trying to open /usr/lib64/dri-nonfree/nvidia_drv_video.so  
libva info: Trying to open /usr/lib64/dri-freeworld/nvidia_drv_video.so  
libva info: Trying to open /usr/lib64/dri/nvidia_drv_video.so  
libva info: Found init function __vaDriverInit_1_0  
libva info: va_openDriver() returns 0  
vainfo: VA-API version: 1.22 (libva 2.22.0)  
vainfo: Driver version: VA-API NVDEC driver [direct backend]  
vainfo: Supported profile and entrypoints

出现错误 vaInitialize failed with error code -1 (unknown libva error) 并且 VA-API 尝试加载多个路径下的 nvidia_drv_video.so 都失败了，说明 nvidia-vaapi-driver 仍然无法安装。

# 6.(可选) 如果解码 H.264 还是卡顿，那么需要执行以下命令将 fedora 官方仓库提供的只包含自由/开源编解码器的 FFmpeg 更改为 RPM Fusion 仓库中包含完整编解码器（包括非自由如 H.264）的 FFmpeg
# 在执行之前确保开启了 RPM Fusion 仓库的软件源
sudo dnf install ffmpeg ffmpeg-libs --allowerasing
```

# 6. 卸载驱动

```bash
sudo dnf remove xorg-x11-drv-nvidia\*
```

参考文章：

- [fedora43 安装 nvidia 驱动以及开启视频编解码硬件加速 - 张火火isgudi - 博客园](https://www.cnblogs.com/zbyisgudi/p/19418112)
- [Howto/NVIDIA - RPM Fusion](https://rpmfusion.org/Howto/NVIDIA#About_this_Howto)
