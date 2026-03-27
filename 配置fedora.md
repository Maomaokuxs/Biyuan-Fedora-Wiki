# 说明

这是一篇系统配置板块的整合文档，只提供部分内容，具体的内容部分是在其他章节中，我使用的是 KDE 桌面环境，美化通常没有。

## 配置时间

```bash
# 1.设置时区为上海
sudo timedatectl set-timezone Asia/Shanghai

# 2. (可选) 将 RTC 改为 UTC 模式
sudo timedatectl set-local-rtc 0
# Fedora43 默认使用模式的与 Windows 一致，双系统不建议修改。
```

- 更多说明可以看：
  - [[设置时区]]
  - [[解决与windows系统共存的问题]]

## (可选) 将默认 Home 为中文文件夹修改为英文

解决在终端中输入中文路径的痛点。

## 1.重命名

将中文目录重命名和启动的文件移动至新文件夹

```bash
cd
mv 公共 Public
mv 文档 Documents
mv 模板 Templates
mv 音乐 Music
mv 图片 Pictures
mv 视频 Video
mv 下载 Downloads
mv 桌面 Desktop
```

## 2.自动修改

```bash
# 1.安装软件包
sudo dnf install xdg-user-dirs-gtk-update

# 2.临时将语言设置为英文
export LC_ALL=en_US.UTF-8
# 在重启或者注销后失效

# 3.更新目录配置
xdg-user-dirs-update --force 

# 4.如果使用的是 ked 桌面环境需要在文件管理器中重新配置标签页对应的路径
```

- 更多说明可以看：
  - [[将home目录中的默认中文文件夹改为英文]]

## 配置软件源并更新软件包

```bash
# 1. 备份官方软件源
sudo cp /etc/yum.repos.d/fedora.repo /etc/yum.repos.d/fedora.repo.bak
sudo cp /etc/yum.repos.d/fedora-updates.repo /etc/yum.repos.d/fedora-updates.repo.bak

# 2. 更新本地缓存
sudo dnf makecache

# 3. 替换软件源
sudo sed -e 's|^metalink=|#metalink=|g' \
    -e 's|^#baseurl=http://download.example/pub/fedora/linux|baseurl=http://mirrors.tuna.tsinghua.edu.cn/fedora|g' \
    -i.bak \
    /etc/yum.repos.d/fedora.repo \
    /etc/yum.repos.d/fedora-updates.repo

# 4. 启用自由和非自由 RPM 软件仓库
sudo dnf install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm

# 5. 替换掉 Fedora 自带的 flathub 仓库

可以解决 Kde 桌面环境带的图形化软件商店的使用。

# 5.1 移除受限的 Fedora 仓库
sudo flatpak remote-delete fedora

# 5.2 添加 Flathub 完整仓库
sudo flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo

# 6. (可选) Appimage 包格式支持

# 6.1 安装 Fuse
sudo dnf install -y fuse fuse-libs
# 6.2  安装管理工具
sudo flatpak install -y flathub it.mijorus.gearlever

# 7. 更新软件包
sudo dnf upgrade
# 更新完成后重启系统
```

- 更多说明建议看：
  - [[dnf-常用命令]]
  - [[更新软件包和系统]]

- 参考文档：
  - [Fedora-Noble-Setup](https://github.com/wz790/Fedora-Noble-Setup?tab=readme-ov-file#first-things)

## 配置主机名

```bash
sudo hostnamectl set-hostname biyuan
# biyuan 改成需要的名字
```

## (可选) 配置系统级快照

[[snapper-实现快照功能]]

注：在自由的系统中，需要自己确定每次执行的命令是什么功能，如果不确定使用系统快照是明智的选择。

## 安装显卡驱动

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

## 添加更多的音频相关组件

[[音频相关组件]]

## 文本编辑器 Vim 和 Git

### 1.安装 Vim 和 Git

```bash
sudo dnf install vim git 
```

### 2.基本配置 Git

```bash
# 1.设置用户名和邮箱（提交代码时会用到）
git config --global user.name "名字"
git config --global user.email "邮箱@example.com"

# 2.配置代理
git config --global http.proxy http://代理地址:端口
git config --global https.proxy https://代理地址:端口
```

### 3. (可选) 安装 vscodium 或 vscode

```bash
# 1.1 添加 包含 vscodium 的软件仓库
sudo tee -a /etc/yum.repos.d/vscodium.repo << 'EOF'
[gitlab.com_paulcarroty_vscodium_repo]
name=gitlab.com_paulcarroty_vscodium_repo
baseurl=https://paulcarroty.gitlab.io/vscodium-deb-rpm-repo/rpms/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://gitlab.com/paulcarroty/vscodium-deb-rpm-repo/raw/master/pub.gpg
metadata_expire=1h
EOF

# 1.2 安装 vscodium
sudo dnf install codium

# 2.1 下载 vscode 软件包
https://code.visualstudio.com/
# 进入官网下载 .rpm 后缀软件包

# 2.2 安装 vscode 
sudo dnf instasll 软件包路径
```

## 安装输入法

### 1.安装 Fcitx5 软件包及工具

```bash
sudo dnf install fcitx5 fcitx5-chinese-addons fcitx5-configtool
```

- `fcitx5`：输入法主程序；
- `fcitx5-chinese-addons`：额外中文包，提供拼音五笔等输入；
- `fcitx5-configtool`：输入法配置工具。

### 2.在 KDE 设置中启用 Fcitx5 输入法

```text
设置 -> 键盘 -> 虚拟键盘 -> Fcitx5 Wayland 启动器
```

## 配置 firefox 浏览器

```text
右上角三条横线 -> 设置 -> 主页
```

### 1.(可选) 移除 Fedora 官方创建的标签页

[[/images/fedora/Fedora-configs/firefox-config-1.png]]

### 2.更改搜索引擎为 Bing

[[/images/fedora/Fedora-configs/firefox-config-2.png]]

## 配置 KDE

```text
设置 -> 显示器和监视器
```

### 1 修改显示和监视器设置

[[images/fedora/Fedora-configs/kde-config-2.png]]

### 2 关闭将鼠标移动至左上角屏幕边缘开启窗口平铺展示

[[images/fedora/Fedora-configs/kde-config-1.png]]
