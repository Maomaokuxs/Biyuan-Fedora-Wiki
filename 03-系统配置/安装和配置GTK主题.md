# 说明

以 Orchis 主题为例，通过下载 GitHub Releases 预编译包安装 GTK 主题，更新 GTK 2/3/4 配置，无需从源码编译。

- 环境：

  - Fedora 版本：Fedora Linux 44 x86_64
  - 桌面: niri

---

## 下载预编译主题

从主题项目的 GitHub Releases 页面下载预编译 tar 包：

```bash
curl -LO https://github.com/vinceliuice/Orchis-theme/releases/latest/download/Orchis.tar.xz
```

---

## 解压到主题目录

```bash
mkdir -p ~/.themes
tar -xf Orchis.tar.xz -C ~/.themes/
```

解压后 `~/.themes/` 下会出现 `Orchis-Dark/`、`Orchis-Light/`、`Orchis-Dark-Compact/` 等目录。

---

## 更换主题变体

Release 包通常包含所有颜色和尺寸变体。如需不同风格，直接修改配置文件中的主题名即可：

| 主题目录名 | 说明 |
| ----------- | ------ |
| `Orchis` | 默认浅色 |
| `Orchis-Dark` | 深色 |
| `Orchis-Light` | 浅色 |
| `Orchis-Dark-Compact` | 深色紧凑 |
| `Orchis-Light-Compact` | 浅色紧凑 |

---

## 更新 GTK 3 配置

`~/.config/gtk-3.0/settings.ini`：

```ini
gtk-theme-name=Orchis-Dark
```

---

## 更新 GTK 4 配置

`~/.config/gtk-4.0/settings.ini`：

```ini
gtk-theme-name=Orchis-Dark
```

---

## 更新 GTK 2 配置

`~/.gtkrc-2.0`：

```ini
gtk-theme-name="Orchis-Dark"
```

---

## 设置环境变量

在 `~/.bashrc` 中写入：

```bash
export GTK_THEME=Orchis-Dark
```

---

## 使配置生效

```bash
source ~/.bashrc
```

重新登录后主题将完全生效。

---

## 验证配置

```bash
cat ~/.config/gtk-3.0/settings.ini | grep theme-name
cat ~/.config/gtk-4.0/settings.ini | grep theme-name
cat ~/.gtkrc-2.0 | grep theme-name
```

---

## 自定义主题参数

预编译包的主题样式已固定，无法直接调整圆角、透明度等参数。如需自定义，需要从源码编译：

1. 克隆源码仓库并运行 `install.sh`：

    ```bash
    git clone https://github.com/vinceliuice/Orchis-theme.git ~/Orchis-theme
    cd ~/Orchis-theme
    ```

2. 通过 CLI 参数调整：

    | 参数 | 说明 |
    | ------ | ------ |
    | `-c dark` | 深色变体 |
    | `-c light` | 浅色变体 |
    | `-t purple\|pink\|red\|green\|teal\|grey` | 强调色 |
    | `-i fedora` | Fedora 图标 |
    | `--round 5px` | 自定义圆角半径 |
    | `-s compact` | 紧凑布局 |
    | `--tweaks solid` | 无透明面板 |
    | `--tweaks black` | 纯黑变体 |
    | `--tweaks macos` | macOS 风格窗口按钮 |

3. 编译需要 `sassc` 或 `dart-sass`。无 sudo 时可通过 npm 安装并创建包装脚本。

---

## 卸载主题

```bash
rm -rf ~/.themes/Orchis*
```

---

- 参考文档：

  - [Orchis-theme](https://github.com/vinceliuice/Orchis-theme)
