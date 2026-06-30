# 说明

这篇内容只是粗略的说明了一下 Fedora，用于使用 Fedora 时挑选不同的版本，以及提及一下官方的镜像写入工具，因为官方文档采取的是按发行版本来更新文档，旧文档已经归档，下面的内容是基于 Fedora 43 版本而言，具体看参考文档，其文档按照版本归档，内容在 Fedora 新版本发布时内容非常的少，即使是下一个版本发布完了内容也没很大的参考价值，反倒论坛或者使用 AI 工具更能获得帮助。如果你不知道从哪里开始就选择 [Fedora Workstation](https://fedoraproject.org/zh-Hans/workstation/) 或者 [Fedora KDE Plasma](https://fedoraproject.org/zh-Hans/kde/)。如果想要构建最小化 Fedora 镜像，选择 [Fedora Everything](https://fedoraproject.org/zh-Hans/misc/#everything)，在安装时选择需要的工具及桌面环境。

---

## 介绍

Fedora 是一个由社区维护基于自由开源软件的 `Linux` 发行版，是 Redhat 的上游。提供有多种发行版，大多采用图形化的安装器 Anaconda，并提供官方写入工具[Fedora Media Writer](https://docs.fedoraproject.org/en-US/fedora/latest/preparing-boot-media/#fedora_media_writer)，不是所有版本都支持，有些版本依旧用着老版本的安装器，但是换汤不换药，想要下载新版安装镜像统一进入[发布页](https://fedoraproject.org/zh-Hans/)进行选择下载。

参考文档：

- [Fedora’s Mission and Foundations](https://docs.fedoraproject.org/en-US/project/)

---

## Fedora 各种版本的区别

大多数衍生版本都基于某个特定版本（通常是 Fedora 工作站版），并使用相应的安装流程。大体分为 Editions，Spins，Atomic Desktops 和 Labs 四个类别，每一类包含数个发行版，详细说明请跳转至 [[Fedora-各种发行版的详细说明]]。

- **Editions**
  
  该类别包含各种不同服务场景的版本，为个人电脑提供的 [Fedora Workstation](https://fedoraproject.org/zh-Hans/workstation/) 版本，使用 Gnome 为桌面环境，其定制版是使用 KDE 为桌面环境的 [Fedora KDE Plasma](https://fedoraproject.org/zh-Hans/kde/) 桌面；适用于服务器的 Fedora 服务器版；用于"物联网"服务器的 [Fedora IOT](https://fedoraproject.org/zh-Hans/iot/) 版；适用于容器化的 [Fedora CoreOS](https://fedoraproject.org/zh-Hans/coreos/) 版，专注于云网络上的 [Fedora Cloud](https://fedoraproject.org/zh-Hans/cloud/) 版。

- **Spins**
  
  该类别提供多种除 Gnome 和 KDE 之外的其他桌面环境，通常适用于个人电脑。
  
  - [Fedora Xfce Desktop](https://fedoraproject.org/spins/xfce)
  - [Fedora Cinnamon Desktop](https://fedoraproject.org/spins/cinnamon)
  - [Fedora Mate + Compiz Desktop](https://fedoraproject.org/spins/mate)
  - [Fedora i3 Tiling WM](https://fedoraproject.org/spins/i3)
  - [Fedora LXQt Desktop](https://fedoraproject.org/spins/lxqt)
  - [Fedora LXDE Desktop](https://fedoraproject.org/spins/lxde)
  - [Fedora SOAS Desktop](https://fedoraproject.org/spins/soas)
  - [Fedora Sway Tiling WM](https://fedoraproject.org/spins/sway)
  - [Fedora Budgie Desktop](https://fedoraproject.org/spins/budgie)
  - [Fedora Miracle Desktop](https://fedoraproject.org/spins/miraclewm)
  - [Fedora KDE Mobile Desktop](https://fedoraproject.org/spins/kde-mobile)
  - [Fedora COSMIC Desktop](https://fedoraproject.org/spins/cosmic)

- **Atomic Desktops**
  
  该类别的特点是整个系统一次性更新，如果出现任何问题，更新都不会生效，可以回滚到之前的版本，图形应用程序通过 Flatpak 安装，实现容器化，并且提供多种桌面环境。
  
  - [Fedora Silverblue](https://fedoraproject.org/zh-Hans/atomic-desktops/silverblue/)(Gnome 桌面)
  - [Fedora Kinoite](https://fedoraproject.org/zh-Hans/atomic-desktops/kinoite/)(KDE 桌面)
  - [Fedora Sway Atomic](https://fedoraproject.org/zh-Hans/atomic-desktops/sway/)(Sway 桌面)
  - [Fedora Budgie Atomic](https://fedoraproject.org/zh-Hans/atomic-desktops/budgie/)(Budgie 桌面)
  - [Fedora Cosmic Atomic](https://fedoraproject.org/zh-Hans/atomic-desktops/cosmic/)(COSMIC 桌面)

- **Labs**
  
  该类别为不同用户需求提供的发行版，例如游戏，天文学，科学等。

参考文档：

- [Fedora 用户文档](https://docs.fedoraproject.org/en-US/fedora/latest/)

## 杂项下载

- [通过种子下载 Fedora linux](https://fedoraproject.org/zh-Hans/misc/#torrents)
- [Fedora 最小化安装](https://fedoraproject.org/zh-Hans/misc/#minimal)
- [Fedora Everything](https://fedoraproject.org/zh-Hans/misc/#everything)
- [Fedora容器基础 44](https://fedoraproject.org/zh-Hans/misc/#container_minimal_base)
- [Fedora 容器最小化基础镜像](https://fedoraproject.org/zh-Hans/misc/#container_minimal_base)
- [Fedora 容器工具](https://fedoraproject.org/zh-Hans/misc/#container_toolbox)
- [Fedora WSL](https://fedoraproject.org/zh-Hans/misc/#wsl_base)
- [Fedora 测试镜像](https://fedoraproject.org/zh-Hans/misc/#testing_images)
- [Fedora Rawhide](https://fedoraproject.org/zh-Hans/misc/#rawhide)
