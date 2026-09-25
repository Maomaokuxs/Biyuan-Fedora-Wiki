# Kinoite配置指南

- 本文记录 Fedora Kinoite（rpm-ostree 不可变系统）从零到可用的完整配置过程。与普通 Fedora（dnf）通用的背景知识（akmod 原理、安全启动密钥、硬件编解码）见《安装Nvidia显卡驱动并开启硬件编解码》，不重复。
- 核心概念：Kinoite 的 `/usr` 只读，没有 dnf；系统更新靠切换部署（升级→重启生效→旧部署保留可回滚）。所有 `rpm-ostree` 操作都只改写同一个待定部署，可攒多条、重启一次统一生效。
- 实测环境：Fedora Kinoite 44，台式机（华硕 TUF B550M，Ryzen 5 5600，RTX 4060），经 SSH 从另一台 Fedora 管理。

## 1. SSH 共享通道（ControlMaster）

远程只允许密码登录时，在管理机另开终端建共享主连接，之后免密操作；远程重启后通道会断，重建一次即可：

```bash
# 管理机执行，输一次密码后挂后台
ssh -M -S ~/.ssh/fedora-5.sock -fN -o ControlPersist=1h fedora-test
# 复用执行命令 / 传文件
ssh -S ~/.ssh/fedora-5.sock fedora-test '命令'
ssh -S ~/.ssh/fedora-5.sock fedora-test 'cat > /tmp/文件名' < 本地文件
```

## 2. 更新系统

```bash
systemctl reboot          # 先应用已 staged 的更新
sudo rpm-ostree upgrade   # 进新系统后再检查
systemctl reboot          # 有更新则重启；rpm-ostree status 确认版本
sudo flatpak update       # 顺手更新 flatpak
```

## 3. 软件安装三板斧（优先级从高到低）

1. **Flatpak**（图形软件首选）：`flatpak install flathub 包名`，或用 Discover 图形安装。Kinoite 默认只有 `fedora` 源，建议加 Flathub：

   ```bash
   flatpak remote-add --user --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
   ```

2. **Toolbox**（命令行/开发环境）：见第 9 节，COPR、编译环境都在里面玩。
3. **Layer**（宿主机集成）：`sudo rpm-ostree install 包名` + 重启，只留给驱动、输入法、shell 这类必须进系统的东西。代价：升级变慢占空间、跨大版本可能卡住（惯例先 `rpm-ostree reset` 再升）；`rpm-ostree status` 查看，`uninstall` 可卸。

## 4. 换国内镜像（出境带宽慢时）

`/etc` 下的改动持久有效，换一次长期受益（以下中科大为例，清华同理）：

```bash
# Fedora 主源（立即生效，不用重启）
sudo sed -i 's|^metalink=|#metalink=|; s|^#baseurl=http://download.example/pub/fedora/linux|baseurl=https://mirrors.ustc.edu.cn/fedora|' /etc/yum.repos.d/fedora.repo /etc/yum.repos.d/fedora-updates.repo
# RPM Fusion 源（装好源包后执行）
sudo sed -i 's|^mirrorlist=|#mirrorlist=|; s|^#baseurl=http://download1.rpmfusion.org|baseurl=https://mirrors.ustc.edu.cn/rpmfusion|' /etc/yum.repos.d/rpmfusion-*.repo
sudo rpm-ostree cleanup -m
```

## 5. NVIDIA 驱动（官方 OSTree 节）

依据 [Howto/NVIDIA - OSTree](https://rpmfusion.org/Howto/NVIDIA#OSTree_.28Silverblue.2FKinoite.2Fetc.29)，Secure Boot 关闭时跳过密钥章节：

```bash
# 5.1 装 RPM Fusion 源（配置页面），重启
sudo rpm-ostree install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
systemctl reboot
```

踩坑：URL 形式可能下载成功却报 `error: Packages not found:`（rpm-ostree 解析 URL 包的老毛病）。绕法是先下载到本地再从本地文件安装（别处下载好传过来同样有效，md5 对上即可）：

```bash
curl -LO https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-44.noarch.rpm
curl -LO https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-44.noarch.rpm
sudo rpm-ostree install /tmp/rpmfusion-free-release-44.noarch.rpm /tmp/rpmfusion-nonfree-release-44.noarch.rpm
```

```bash
# 5.2 源包转 layered（防跨大版本升级卡住），重启
sudo rpm-ostree update \
  --uninstall rpmfusion-free-release --uninstall rpmfusion-nonfree-release \
  --install rpmfusion-free-release --install rpmfusion-nonfree-release
systemctl reboot
```

```bash
# 5.3 装驱动（文档原文；cuda 为 nvidia-smi 验证所需，建议装）
sudo rpm-ostree install akmod-nvidia xorg-x11-drv-nvidia
sudo rpm-ostree install akmod-nvidia xorg-x11-drv-nvidia-cuda
# 5.4 kargs 屏蔽开源驱动（文档原文；ostree 限制，RPM 包改不了内核参数）
sudo rpm-ostree kargs --append=rd.driver.blacklist=nouveau,nova_core --append=modprobe.blacklist=nouveau,nova_core
systemctl reboot
```

首次开机 akmods 后台编译内核模块，多等几分钟，进系统验证：`nvidia-smi`、`lspci -k -d 10de:`（应为 `nvidia`）。

## 6. 多媒体硬解码

前提：闭源驱动先生效。音频编解码 base 自带（MP3/AAC/FLAC/Opus），缺的是视频非自由编解码，一次事务搞定（单行，避免换行误执行）：

```bash
sudo rpm-ostree override remove ffmpeg-free libavcodec-free libavdevice-free libavfilter-free libavformat-free libavutil-free libswresample-free libswscale-free --install ffmpeg --install libva-nvidia-driver --install libva-utils
sudo rpm-ostree install gstreamer1-plugins-bad-freeworld gstreamer1-plugins-ugly
systemctl reboot
```

验证：`vainfo | grep "Driver version"`（VA-API NVDEC driver）、`ffmpeg -hide_banner -hwaccels`（应有 cuda/vaapi）。

说明：`override remove + --install` 是原子系统 swap 包的标准写法；报错 `conflicting requests` 时把冲突的 `-free` 包一次卸全即可。

## 7. 中文输入法（fcitx5）

输入法要注入所有程序，必须 layer（flatpak/toolbox 做不到）；base 自带 ibus 框架（只缺中文引擎），但 KDE/Wayland 下 fcitx5 体验更好，为标准做法：

```bash
sudo rpm-ostree install fcitx5 fcitx5-chinese-addons fcitx5-configtool
```

环境变量（家目录，无需 sudo，Plasma 登录自动加载）：

```bash
mkdir -p ~/.config/plasma-workspace/env
cat > ~/.config/plasma-workspace/env/im.sh <<'EOF'
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS=@im=fcitx
export SDL_IM_MODULE=fcitx
EOF
```

重启进桌面后：`fcitx5-configtool` 添加 Pinyin；系统设置→键盘→虚拟键盘确认 Fcitx 5；`Ctrl+空格` 切换验证。

## 8. 音频

PipeWire + WirePlumber 开箱即用，一般不用配。唯一常调的是输出设备二选一：

- 图形：系统设置→音频→输出设备设默认（主板模拟口需先把配置文件从 `数字立体声 (IEC958)` 切成`模拟立体声输出`）。
- 命令行：`wpctl status | grep -A8 Sinks:` 看 id，`wpctl set-default <id>` 切换。

## 9. Toolbox 与 Distrobox

```bash
toolbox create && toolbox enter   # 进去是完整可写 Fedora，sudo 免密，dnf/copr 随便用
toolbox run 命令                   # 不进去直接跑里面的命令
exit / toolbox list / toolbox rm -f 名字
```

- 家目录与宿主机共享；里面装的软件隔离保存，宿主机升级零负担。
- COPR 的命令行包、编译环境都在容器里玩：`toolbox enter` → `sudo dnf copr enable 用户/项目` → `sudo dnf install 包名`。
- 容器与宿主机共享内核，无虚拟化开销，同时开几个随便造；删容器即释放磁盘（`podman system df` 查看）。
- 普通 Fedora 上用法完全相同（`dnf install toolbox`）。
- 要跑 Arch/Ubuntu 等其他发行版容器：`sudo rpm-ostree install distrobox`，重启后 `distrobox create -i archlinux:latest -n arch && distrobox enter arch`。原生 toolbox 只面向 Fedora/RHEL 系。

## 10. COPR 装到宿主机

```bash
sudo curl -o /etc/yum.repos.d/xxx.repo https://copr.fedorainfracloud.org/coprs/用户/项目/repo/fedora-44/用户-项目-fedora-44.repo
sudo rpm-ostree install 包名
```

COPR 是个人构建，跨大版本升级比 RPM Fusion 更易卡住；命令行工具优先放 toolbox，不用时连 repo 文件一起删。

## 11. 实测验证记录（2026-09-25，RTX 4060）

- `nvidia-smi`：驱动 615.71.09，认卡正常
- `vainfo`：Driver version: VA-API NVDEC driver
- `ffmpeg -hwaccels`：cuda / vaapi / vdpau 均可用
- 同一部署另含：fcitx5、distrobox、fastfetch、gstreamer 插件；旧部署保留可回滚

## 参考文档

- [Howto/NVIDIA - RPM Fusion](https://rpmfusion.org/Howto/NVIDIA#OSTree_.28Silverblue.2FKinoite.2Fetc.29)
- [Howto/OSTree - RPM Fusion](https://rpmfusion.org/Howto/OSTree)
- [Using NVIDIA drivers - Fedora Atomic troubleshooting](https://fedora.gitlab.io/ostree/docs/fedora-atomic-desktops/troubleshooting/)
