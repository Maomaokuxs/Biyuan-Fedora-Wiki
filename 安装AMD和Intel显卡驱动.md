# 说明

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