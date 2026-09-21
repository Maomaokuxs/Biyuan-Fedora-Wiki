# 说明

因为 kde plasma 在 6.6 版本起将窗口管理器由原来 sddm 的更改为 plasmalogin，但是 fedora 安装最新版的桌面环境并不会主动替换，需要手动安装相关软件包并配置。

## 1.安装 plasmalogin

首先，确保你的系统已通过 Discover 或 DNF 完整更新到 KDE Plasma 6.6,可以在设置关于中查看当前 kde 版本也可以使用下面的命令。

```bash
plasmashell --version
```

```bash
sudo dnf install plasmalogin
```

## 2.禁用当前的 SDDM 服务

```bash
sudo systemctl disable sddm
```

## 3.启用新的 plasmalogin 服务

```bash
sudo systemctl enable plasmalogin
```

## 4.重启系统

重启电脑，你应该会看到新的登录管理器界面。

```bash
reboot
```

## 5. (可选)删除 sddm 软件包

- **确认 plasmalogin 窗口管理器已启用**

```bash
sudo systemctl status display-manager
```

确保输出为

```text
biyuan@fedoralinux:~$ sudo systemctl status display-manager
● plasmalogin.service - Plasma Login Manager
     Loaded: loaded (/usr/lib/systemd/system/plasmalogin.service; enabled; preset: disabled)
    Drop-In: /usr/lib/systemd/system/service.d
             └─10-timeout-abort.conf
     Active: active (running) since Thu 2026-02-19 14:05:35 CST; 1h 3min ago
```

- **删除 sddm 软件包**

```bash
sudo dnf remove sddm
```
