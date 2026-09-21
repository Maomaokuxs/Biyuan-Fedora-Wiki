# 说明

通常情况下这些可以即插即用。

## 启用 RPM Fusion 仓库

```bash
sudo dnf install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
```

- 核心软件包

Vulkan 及基础加速支持：

```bash
sudo dnf install mesa-vulkan-drivers vulkan-loader  libva-utils
```

- mesa-vulkan-drivers，Mesa 的 Vulkan 驱动，包含 AMD 的 RADV
- vulkan-loader，Vulkan 加载器，连接应用和驱动
- libva-utils，提供 vainfo 等命令行工具，检测 VA-API 状态
  
## AMD（视频加速）

替换默认驱动为 freeworld 版本以获得完整编解码器支持（H.264/HEVC）：

```bash
sudo dnf install mesa-va-drivers mesa-va-drivers-freeworld  --allowerasing
```

- mesa-va-drivers， Fedora 官方仓库的 Mesa VA-API 驱动，因专利原因不含 H.264/H.265
- mesa-va-drivers-freeworld ，RPM Fusion 提供的增强版，包含 H.264/H.265 硬解+硬编

## Intel（视频加速）

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