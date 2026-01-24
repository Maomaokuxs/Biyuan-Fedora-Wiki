# 说明

因为 hyprland 并没有提供图形化提权工具，所以需要申请权限的软件无法正常运行，下面是使用 ked 的提权工具实现功能。

## 1. 安装软件包

```bash
sudo dnf install polkit-kde
```

## 2. 确认正确的路径

```bash
find /usr -name "*polkit*kde*" -type f 2>/dev/null
```

- 找不到可以使用下面命令来查看桌面快捷防止指向的路径

```bash
cat /usr/share/applications/org.kde.polkit-kde-authentication-agent-1.desktop
```

## 3. 在hyprland配置文件中设置自启动

```bash
exec-once = /usr/libexec/polkit-kde-authentication-agent-1
```
