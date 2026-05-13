# 说明

安装 fedora 的篇外故事。

## 联接网络

- 有线网默认直接链接

- 无线网使用 nmcli 工具打开 TUI 界面链接

## 启用 root 账户

```bash
sudo -i 
```

如果需要使用另一台电脑连接安装参考以下方式。

- 配置 root 密码

```bash
passwd
```

- 获取安装机器的网络ip地址

```bash
ip addr
```

- 通过ssh连接机器

```bash
ssh root@<ip地址>
```

## 创建分区

```bash
cfdisk /dev/nvme1n1
```

## 配置分区

我使用的是 btrfs 文件系统。

```bash
# nix-shell -p btrfs-progs
# mkfs.fat -F 32 /dev/nvme1n1p7
# mkfs.btrfs /dev/nvme1n1p8
# mkdir -p /mnt
# mount /dev/nvme1n1p8 /mnt
# btrfs subvolume create /mnt/root
# btrfs subvolume create /mnt/home
# btrfs subvolume create /mnt/nix
# umount /mnt
```

## 挂载分区和子卷

```bash
# mount -o compress=zstd,subvol=root /dev/nvme1n1p8 /mnt
# mkdir /mnt/{home,nix}
# mount -o compress=zstd,subvol=home /dev/nvme1n1p8 /mnt/home
# mount -o compress=zstd,noatime,subvol=nix /dev/nvme1n1p8 /mnt/nix

# mkdir /mnt/boot
# mount /dev/nvme1n1p7 /mnt/boot
```

## 安装 NixOS

```bash
# nixos-generate-config --root /mnt
# nano /mnt/etc/nixos/configuration.nix # manually add mount options (see Compression below for an example)
# nixos-install
```

## NixOS 系统配置

NixOS 使用声明式配置系统，允许用户管理整个操作系统设置，包括已安装的软件包、系统服务、用户帐户、硬件设置和更详细的配置文件。此页面概述了如何使用和管理 NixOS 系统配置。

有关声明式配置的介绍，请参阅 NixOS Linux 发行版概述#声明式配置 和 NixOS 官方手册。

用法
安装 NixOS 时，默认系统配置模板由 nixos-generate-config 工具生成。 这会创建一个基本的 configuration.nix 文件以及相应的 hardware-configuration.nix 文件，后者捕获检测到的硬件设置和文件系统定义。 在更改 configuration.nix 后，可以使用 nixos-rebuild 应用它们

## nixos-rebuild switch

要查找 NixOS 模块选项，请参阅 https://search.nixos.org/options 。

## NixOS 安装

```bash
# cd /mnt
# nixos-install
```
