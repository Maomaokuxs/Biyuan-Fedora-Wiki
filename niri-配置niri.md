# 说明

实验性质的文章，从入门开始摸索。

## 安装 niri

```bash
# 1.启用 copr 仓库
sudo dnf copr enable alternateved/niri 

# 2.安装 niri
sudo dnf install niri
```

---

## 使用 KDE 桌面环境时可能会遇到启动后有一个黑色的窗口

### 1. 原因

可能是 xwaylandvideobridge 这个程序。

```text
biyuan@fedora:~$ niri msg windows 2>/dev/null || echo "niri msg not available"
Window ID 2:
  Title: "Wayland 到 X 录像桥接程序 — Xwayland 视频桥接程序"
  App ID: "xwaylandvideobridge"
  Is floating: no
  PID: 28701
  Workspace ID: 1
  Layout:
    Tile size: 936 x 1006
    Scrolling position: column 1, tile 1
    Window size: 936 x 1006
    Window offset in tile: 0 x 0
```

xwaylandvideobridge 是一个用于在 Wayland 和 XWayland 之间传输视频数据的桥接程序，通常由某些应用（如屏幕录制、截图工具、远程桌面等）自动启动。但在 Niri 中，它可能无法正常工作，导致显示一个黑色窗口。

### 2.解决办法

阻止这个程序自启动，执行以下命令：

```bash
systemctl --user mask app-org.kde.xwaylandvideobridge@autostart.service
```

如果想要恢复执行：

```bash
systemctl --user unmask app-org.kde.xwaylandvideobridge@autostart.service
```

---


