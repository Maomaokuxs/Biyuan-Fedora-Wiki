# 说明

这篇内容只是粗略的说明了一下 Fedora，用于使用 Fedora 时挑选不同的版本，以及提及一下官方的镜像写入工具。

Fedora 是一个由社区维护基于自由开源软件的 `Linux`发行版，是Redhat的上游。提供有多种发行版，大多采用图形化的安装器Anaconda，并提供官方写入工具[Fedora Media Writer](https://docs.fedoraproject.org/en-US/fedora/latest/preparing-boot-media/#fedora_media_writer)

参考文档：

- [Fedora’s Mission and Foundations](https://docs.fedoraproject.org/en-US/project/)

## fedora 各种版本的区别

- 大多数衍生版本都基于某个特定版本（通常是 Fedora 工作站版），并使用相应的安装流程。大体分为 Editions，Spins，Atomic Desktops 和 Labs 四个类别,每一类包含数个发行版，详细说明请跳转至[[fedora-各种发行版的详细说明]]。

- Editions

- 该类别包含各种不同服务场景的版本，为个人电脑提供的 Fedora 工作站版本，使用 gnome 为桌面环境，其定制版是使用 kde 为桌面环境的 Fedora KDE Plasma 桌面；适用于服务器的 Fedora 服务器版；用于“物联网”服务器的 Fedora IOT 版；适用于容器化的Fedora CoreOS 版，专注于云网络上的 Fedora Cloud版。

- Spins

- 该类别提供多种除了 gnome 和 ked 除外的其他桌面环境，通常适用于个人电脑。

- Atomic Desktops

- 该类别的特点是整个系统一次性更新，如果出现任何问题，更新都不会生效，可以回滚到之前的版本，图形应用程序通过 Flatpak 安装，实现容器化，并且提供多种桌面环境。

- Labs

- 该类别为为不同用户需求提供的发行版，例如游戏，天文学，科学等。

参考文档：

- [Fedora 用户文档](https://docs.fedoraproject.org/en-US/fedora/latest/)
