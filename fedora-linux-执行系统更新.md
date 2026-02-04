
# 说明

## 清理 Hyprland 及其依赖（针对 Problem 1, 2, 3, 6, 7）

先卸载冲突的 Hyprland 组件：

```Bash
Failed to resolve the transaction:
Package "rpmfusion-nonfree-release-43-1.noarch" is already installed.
Package "rpmfusion-free-release-43-1.noarch" is already installed.
Packages for argument 'nvidia-gpu-firmware' available, but not installed.
Packages for argument 'kmahjongg' available, but not installed.
Packages for argument 'kmines' available, but not installed.
Packages for argument 'kpat' available, but not installed.
Packages for argument 'kde-spin-initial-setup' installed, but not available.
Package "gstreamer1-plugins-bad-freeworld-1:1.26.10-1.fc43.x86_64" is already installed.
Problem 1: installed package aquamarine-0.10.0-5.fc43.x86_64 requires libhyprutils.so.10()(64bit), but none of the providers can be installed
  - hyprutils-0.11.0-2.fc43.x86_64 does not belong to a distupgrade repository
  - problem with installed package
 Problem 2: installed package hyprland-0.53.3-1.fc43.x86_64 requires hyprcursor(x86-64) >= 0.1.13, but none of the providers can be installed
  - hyprcursor-0.1.13-2.fc43.x86_64 does not belong to a distupgrade repository
  - problem with installed package
 Problem 3: installed package hyprlock-0.9.2-7.fc43.x86_64 requires libhyprgraphics.so.4()(64bit), but none of the providers can be installed
  - hyprgraphics-0.5.0-1.fc43.x86_64 does not belong to a distupgrade repository
  - problem with installed package
 Problem 4: installed package telegram-desktop-6.3.10-1.fc43.x86_64 requires libavutil.so.59()(64bit), but none of the providers can be installed
  - installed package telegram-desktop-6.3.10-1.fc43.x86_64 requires libavutil.so.59(LIBAVUTIL_59)(64bit), but none of the providers can be installed
  - installed package telegram-desktop-6.3.10-1.fc43.x86_64 requires libavcodec.so.61()(64bit), but none of the providers can be installed
  - installed package telegram-desktop-6.3.10-1.fc43.x86_64 requires libavcodec.so.61(LIBAVCODEC_61)(64bit), but none of the providers can be installed
  - installed package telegram-desktop-6.3.10-1.fc43.x86_64 requires libavformat.so.61()(64bit), but none of the providers can be installed
  - installed package telegram-desktop-6.3.10-1.fc43.x86_64 requires libavformat.so.61(LIBAVFORMAT_61)(64bit), but none of the providers can be installed
  - installed package telegram-desktop-6.3.10-1.fc43.x86_64 requires libswresample.so.5()(64bit), but none of the providers can be installed
  - installed package telegram-desktop-6.3.10-1.fc43.x86_64 requires libswresample.so.5(LIBSWRESAMPLE_5)(64bit), but none of the providers can be installed
  - installed package telegram-desktop-6.3.10-1.fc43.x86_64 requires libswscale.so.8()(64bit), but none of the providers can be installed
  - installed package telegram-desktop-6.3.10-1.fc43.x86_64 requires libswscale.so.8(LIBSWSCALE_8)(64bit), but none of the providers can be installed
  - installed package telegram-desktop-6.3.10-1.fc43.x86_64 requires libavfilter.so.10()(64bit), but none of the providers can be installed
  - installed package telegram-desktop-6.3.10-1.fc43.x86_64 requires libavfilter.so.10(LIBAVFILTER_10)(64bit), but none of the providers can be installed
  - ffmpeg-libs-7.1.2-7.fc43.x86_64 does not belong to a distupgrade repository
  - problem with installed package
 Problem 5: problem with installed package
  - conflicting requests
  - libheif-1.20.2-6.fc43.x86_64 does not belong to a distupgrade repository
  - nothing provides libheif(x86-64) = 1.21.1 needed by libheif-freeworld-1.21.1-2.fc44.x86_64 from rpmfusion-free-updates
  - nothing provides libheif(x86-64) = 1.21.1 needed by libheif-freeworld-1.21.1-2.fc44.x86_64 from rpmfusion-free
  - nothing provides libheif(x86-64) = 1.21.1 needed by libheif-freeworld-1.21.1-2.fc44.x86_64 from rpmfusion-free-updates-testing
 Problem 6: problem with installed package
  - package xdg-desktop-portal-hyprland-1.3.6-2.fc42.x86_64 from fedora requires libsdbus-c++.so.1()(64bit), but none of the providers can be installed
  - package xdg-desktop-portal-hyprland-1.3.6-2.fc42.x86_64 from updates requires libsdbus-c++.so.1()(64bit), but none of the providers can be installed
  - package xdg-desktop-portal-hyprland-1.3.6-2.fc42.x86_64 from updates-testing requires libsdbus-c++.so.1()(64bit), but none of the providers can be installed
  - xdg-desktop-portal-hyprland-1:1.3.11-3.fc43.x86_64 does not belong to a distupgrade repository
  - sdbus-cpp-2.1.0-3.fc43.x86_64 does not belong to a distupgrade repository
 Problem 7: problem with installed package
  - package hyprlang-0.6.4-4.fc44.x86_64 from fedora requires libhyprutils.so.6()(64bit), but none of the providers can be installed
  - package hyprlang-0.6.4-4.fc44.x86_64 from updates requires libhyprutils.so.6()(64bit), but none of the providers can be installed
  - package hyprlang-0.6.4-4.fc44.x86_64 from updates-testing requires libhyprutils.so.6()(64bit), but none of the providers can be installed
  - cannot install both hyprutils-0.7.1-4.fc43.x86_64 from fedora and hyprutils-0.11.0-2.fc43.x86_64 from @System
  - cannot install both hyprutils-0.7.1-4.fc43.x86_64 from updates and hyprutils-0.11.0-2.fc43.x86_64 from @System
  - cannot install both hyprutils-0.7.1-4.fc43.x86_64 from updates-testing and hyprutils-0.11.0-2.fc43.x86_64 from @System
  - installed package hyprpicker-0.4.5-6.fc43.x86_64 requires libhyprutils.so.10()(64bit), but none of the providers can be installed
  - hyprlang-0.6.8-1.fc43.x86_64 does not belong to a distupgrade repository
  - problem with installed package
在使用fedoralinux时sudo dnf system-upgrade download --releasever=44

执行该命令发生的报错
sudo dnf remove hyprland hyprutils hyprlang hyprlock hyprpicker xdg-desktop-portal-hyprland
```

## 强制卸载引起冲突的软件包

```bash
Package "rpmfusion-nonfree-release-43-1.noarch" is already installed.
Package "rpmfusion-free-release-43-1.noarch" is already installed.
Packages for argument 'nvidia-gpu-firmware' available, but not installed.
Packages for argument 'kmahjongg' available, but not installed.
Packages for argument 'kmines' available, but not installed.
Packages for argument 'kpat' available, but not installed.
Packages for argument 'kde-spin-initial-setup' installed, but not available.
Package "gstreamer1-plugins-bad-freeworld-1:1.26.10-1.fc43.x86_64" is already installed.
Problem 1: installed package telegram-desktop-6.3.10-1.fc43.x86_64 requires libavutil.so.59()(64bit), but none of the providers can be installed
  - installed package telegram-desktop-6.3.10-1.fc43.x86_64 requires libavutil.so.59(LIBAVUTIL_59)(64bit), but none of the providers can be installed
  - installed package telegram-desktop-6.3.10-1.fc43.x86_64 requires libavcodec.so.61()(64bit), but none of the providers can be installed
  - installed package telegram-desktop-6.3.10-1.fc43.x86_64 requires libavcodec.so.61(LIBAVCODEC_61)(64bit), but none of the providers can be installed
  - installed package telegram-desktop-6.3.10-1.fc43.x86_64 requires libavformat.so.61()(64bit), but none of the providers can be installed
  - installed package telegram-desktop-6.3.10-1.fc43.x86_64 requires libavformat.so.61(LIBAVFORMAT_61)(64bit), but none of the providers can be installed
  - installed package telegram-desktop-6.3.10-1.fc43.x86_64 requires libswresample.so.5()(64bit), but none of the providers can be installed
  - installed package telegram-desktop-6.3.10-1.fc43.x86_64 requires libswresample.so.5(LIBSWRESAMPLE_5)(64bit), but none of the providers can be installed
  - installed package telegram-desktop-6.3.10-1.fc43.x86_64 requires libswscale.so.8()(64bit), but none of the providers can be installed
  - installed package telegram-desktop-6.3.10-1.fc43.x86_64 requires libswscale.so.8(LIBSWSCALE_8)(64bit), but none of the providers can be installed
  - installed package telegram-desktop-6.3.10-1.fc43.x86_64 requires libavfilter.so.10()(64bit), but none of the providers can be installed
  - installed package telegram-desktop-6.3.10-1.fc43.x86_64 requires libavfilter.so.10(LIBAVFILTER_10)(64bit), but none of the providers can be installed
  - ffmpeg-libs-7.1.2-7.fc43.x86_64 does not belong to a distupgrade repository
  - problem with installed package
 Problem 2: problem with installed package
  - conflicting requests
  - libheif-1.20.2-6.fc43.x86_64 does not belong to a distupgrade repository
  - nothing provides libheif(x86-64) = 1.21.1 needed by libheif-freeworld-1.21.1-2.fc44.x86_64 from rpmfusion-free-updates-testing
  - nothing provides libheif(x86-64) = 1.21.1 needed by libheif-freeworld-1.21.1-2.fc44.x86_64 from rpmfusion-free
  - nothing provides libheif(x86-64) = 1.21.1 needed by libheif-freeworld-1.21.1-2.fc44.x86_64 from rpmfusion-free-updates

绕过 RPM Fusion（最推荐，成功率高）
目前大多数冲突（Problem 4, 5 以及 libav* 相关）都源于 RPM Fusion 提供的多媒体库与 Fedora 44 官方库不兼容。我们可以先升级系统核心，等进入 F44 后再解决第三方库。
sudo dnf system-upgrade download --releasever=44 --allowerasing --disablerepo=rpmfusion-free* --disablerepo=rpmfusion-nonfree*

```
