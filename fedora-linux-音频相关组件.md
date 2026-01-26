# 说明

在 Fedora 43 中，音频系统默认基于 **PipeWire** 构建，它集成了低延迟音频处理和多媒体流管理。如果你需要确保音频系统的完整性，或是在最小化安装的基础上构建音频环境，可以按照以下步骤操作。

---

## 1. 核心音频驱动与服务

除了`alsa-firmware`没有预装，其他软件包在`fedora 43`上都安装了。

除非是真的没有声音输出，一般不需要这一步。

Fedora 43 使用 `PipeWire` 代替了传统的 PulseAudio 和 Jack。确保这些核心包已安装：

- **PipeWire**: 核心服务。
- **WirePlumber**: 现代化的会话管理器。
- **ALSA/Pulse 兼容层**: 确保老旧应用也能正常发声。

```bash
# 安装/重新安装核心音频组件
sudo dnf install pipewire pipewire-pulseaudio pipewire-alsa pipewire-jack-audio-connection-kit wireplumber alsa-firmware
```

## 2. 增强多媒体解码器 (Codecs)

因为`fedora`是百分百开源的系统所以一些专利格式都不受支持。

Fedora 官方库不包含某些受限格式的解码器（如 AAC, MP3, H.264 等）。需要启用 **RPM Fusion** 仓库。

### 2.1 启用 RPM Fusion

安装了显卡驱动那一节就没必要再执行这一步了

```bash
sudo dnf install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm \
                 https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm

```

### 2.2 安装多媒体增强包

```bash
# 1. 安装多媒体组包
sudo dnf group install multimedia

# 使用下面命令可以查看 multimedia 包组中的软件包
dnf group info multimedia

# 2. 安装额外的音频插件（GStreamer 等）
sudo dnf install gstreamer1-plugins-bad-free-extras gstreamer1-plugins-ugly gstreamer1-plugins-bad-freeworld
```

## 3. (可选) 音频管理与可视化工具

如果你需要更精细地控制音量、路由或查看频谱，可以安装以下实用工具：

| 工具名称 | 作用 | 安装命令 |
| --- | --- | --- |
| **pavucontrol** | 经典的音量控制面板（PulseAudio/PipeWire 通用） | `sudo dnf install pavucontrol` |
| **qpwgraph** | 图形化 PipeWire 音频路由工具（非常直观） | `sudo dnf install qpwgraph` |
| **helvum** | 一个 PipeWire 节点连接管理工具 | `sudo dnf install helvum` |
| **alsamixer** | 终端界面的底层驱动增益调节工具 | `sudo dnf install alsa-utils` |

## 4. 检查服务状态

```bash
# 查看 PipeWire 和 WirePlumber 状态
systemctl --user status pipewire pipewire-pulse wireplumber
```

> **注意**：如果在安装后发现没声音，一般重启系统（`sudo reboot`）或重启用户服务（`sudo systemctl --user restart pipewire`）即可解决。
