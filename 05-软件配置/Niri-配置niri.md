# 说明

实验性质的文章，从入门开始摸索。

## 安装 niri

1. 启用 copr 仓库

    ```bash
    sudo dnf copr enable alternateved/niri 
    ```

2. 安装 niri

    ```bash
    sudo dnf install niri
    ```

---

## 修改niri配置文件

1. 打开配置文件

    ```bash
    sudo vim ~/.config/niri/config.kdl
    ```

2. 修改终端为 kitty

    ```text
    Mod+T hotkey-overlay-title="Open a Terminal: kitty" { spawn "kitty"; }
    ```

3. 卸载 Alacritty 终端

    ```bash
    sudo dnf remove alacritty
    ```

4. 修改应用启动器为 Rofi

    ```text
    Mod+D hotkey-overlay-title="Run an Application: rofi" { spawn "rofi" "-show" "drun"; }

5. 添加一些快捷键

    ```text
    // 自定义快捷键
    // 刷新waybar
    Ctrl+Alt+R hotkey-overlay-title="Refresh Waybar"  { spawn-sh "pkill waybar || true && waybar"; }
    Ctrl+Alt+C { spawn "~/.config/niri/scripts/toggle-theme.sh"; }
    ```

6. 添加自启动

    ```text
    //自启动
      // fcitx5
      spawn-at-startup "fcitx5" "-d"
      // kde polkit代理（ 提权工具）
      spawn-at-startup "/usr/libexec/kf6/polkit-kde-authentication-agent-1"
    ```

7. 添加窗口规则

    ```text
    // 窗口规则
    window-rule {
    // 窗口矩形绘制设置为无背景
    draw-border-with-background false
    // 设置背景圆角
    geometry-corner-radius 5
    // 剪裁应用边缘适用圆角
    clip-to-geometry true
    }
    ```

8. 修改壁纸

    ```bash
    sudo dnf install swaybg
    ```

    ```text
    // waypaper
      spawn-at-startup "swaybg" "-i" "/home/biyuan/Pictures/paper.png" "-m" "fill"

    ```

---

## 配置终端美化

- 参考文档：
  - [Starship](https://starship.rs/zh-CN/guide/)

1. 美化终端

    - 安装 starship

        ```bash
        sudo dnf copr enable atim/starship
        sudo dnf install starship
        ```

    - 启用 starship

        在 ~/.bashrc 的最后，添加以下内容：

        ```text
        eval "$(starship init bash)"
        ```

    - 配置 starship

        ```shell
        starship preset pastel-powerline -o ~/.config/starship.toml
        # 使用仓库中的配置文件
        ```

2. 配置 kitty

    - 设置字体
  
        ```bash
        # 设置字体
        font_family      Adwaita Mono Nerd Font
        bold_font        auto
        italic_font      auto
        bold_italic_font auto
        # 设置字体大小
        font_size 14
        ```

    - 设置光标

        ```text
        # 设置光标
        #未聚焦光标形状
        cursor_shape_unfocused block
        #未聚焦光标停止闪烁时间，0为永不闪烁
        cursor_stop_blinking_after 0

        #光标追踪动画,0为禁用，大于0的任何值将启用
        cursor_trail 1
        #设置启用光标路径的距离阈值
        cursor_trail_start_threshold 2
        #设置光标追踪的颜色
        cursor_trail_color none
        ```
  
    - 背景设置

        ```text
        # 背景设置
        foreground #dddddd
        background #000000
        #背景透明度,介于0至1之间
        background_opacity 0.6
        #背景模糊，正值启用
        background_blur 0
        #背景图片
        background_image none
        ```

    - 窗口设置

        ```text
        # 窗口
        #隐藏标题栏
        hide_window_decorations yes
        window_padding_width 10
        ```

---

## 配置通知

1. 安装 mako

    ```bash
    sudo dnf install mako
    ```

2. 修改 niri 配置文件

    ```bash
    vim ~/.config/niri/config.kdl
    ```

    ```shell
    # 添加以下内容
    spawn-at-startup "mako"
    ```

3. 美化

---

## 配置应用启动器

1. 安装 rofi

    ```bash
    vim ~/.config/niri/config.kdl
    ```

2. 修改 niri 配置文件

    ```bash
    vim ~/.config/niri/config.kdl
    ```

    ```shell
    # 添加以下内容
    Mod+D hotkey-overlay-title="Run an Application: rofi" { spawn "rofi" "-show" "drun"; }
    ```

3. 美化

---

## 配置输入法

1. 安装 fcitx5

    ```bash
    sudo dnf install fcitx5 fcitx5-chinese-addons fcitx5-configtool
    ```

    - `fcitx5`：Fcitx5 输入法主程序；
    - `fcitx5-chinese-addons`：Fcitx5 额外中文包，提供拼音五笔等输入；
    - `fcitx5-configtool`：Fcitx5 输入法配置工具；

2. 修改niri配置文件

    ```bash
    # 添加以下内容
    spawn-at-startup "fcitx5" "-d"
    ```

---

## 配置 rofi waybar niri 三者随壁纸同步颜色

尽量精简了结构，主要是：

```txt
取色与分发脚本 -> 软件的配色文件 -> 软件配置文件引用配色文件
```

1. 文件结构体系

    ```txet
    ~ (Home Directory)
    ├── .config/
    │   ├── niri/
    │   │   ├── colors.kdl               # 动态生成Niri 的色彩变量文件
    │   │   └── scripts/
    │   │       ├── theme-sync.sh        # 全系统取色与分发脚本
    │   │       └── wallpaper-picker.sh  # Rofi 唤起的壁纸选择器，调用了取色与分发脚本用于切换全局配色
    │   │
    │   ├── waybar/
    │   │   ├── style.css                # Waybar 样式表 (顶部引入动态 CSS 变量)
    │   │   └── scripts/
    │   │       └── cava.sh              # 自适应 PipeWire/Pulse 的频谱脚本
    │   │
    │   └── rofi/
    │
    └── .cache/
        └── hellwal/                       # 存放脚本生成的配色文件
            ├── colors-waybar.css          # 取色脚本生成供 Waybar 引入的 @define-color 变量
            └── colors-rofi.rasi           # 取色脚本生成供 Rofi 引入的 * { ... } 变量
    ```

---

## 帮助

## 使用 KDE 桌面环境时可能会遇到启动后有一个黑色的窗口

1. 原因

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

2. 解决办法

    阻止这个程序自启动，执行以下命令：

    ```bash
    systemctl --user mask app-org.kde.xwaylandvideobridge@autostart.service
    ```

3. 如果想要恢复执行：

    ```bash
    systemctl --user unmask app-org.kde.xwaylandvideobridge@autostart.service
    ```

---
