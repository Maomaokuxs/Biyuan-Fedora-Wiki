# 说明

Wine 是 Linux 上运行 Windows 程序的兼容层。以下记录了在 Fedora 上安装和配置 Wine 的基本流程。

## 1. 安装 Wine

```bash
sudo dnf install -y wine winetricks
```

Wine Staging 版本包含更多的实验性补丁，对游戏兼容性更好。

## 2. 创建 Wine 容器（prefix）

Wine 容器是一个独立的 Windows 环境，每个容器有自己的注册表、C 盘和配置。

```bash
# 默认 64 位容器
WINEPREFIX=~/.wine-名称 wineboot -u
```

> Wine 11.0 起不再支持 `WINEARCH=win32`（WoW64 模式不需要指定）32 位程序在 64 位容器中也能运行。

## 3. 安装字体

Windows 程序运行时如果缺少中文字体或日文字体会显示乱码。

```bash
# 安装 IPA 日文字体（哥特体 + 明朝体）
WINEPREFIX=~/.wine-名称 winetricks fakejapanese_ipamona

# 或者安装 CJK 全字体包（中文 + 日文 + 韩文）
WINEPREFIX=~/.wine-名称 winetricks cjkfonts
```

已安装的字体：

```bash
ls ~/.wine-名称/drive_c/windows/Fonts/
```

## 4. 设置区域

```bash
# 中文区域（适用于汉化版游戏）
WINEPREFIX=~/.wine-名称 wine reg add "HKEY_CURRENT_USER\Software\Wine" /v Locale /d zh_CN /f

# 日文区域（适用于日文原版游戏）
WINEPREFIX=~/.wine-名称 wine reg add "HKEY_CURRENT_USER\Software\Wine" /v Locale /d ja_JP /f
```

## 5. 解压 Windows 游戏/程序

Windows 游戏常以自解压 exe、RAR 或 ISO 格式分发。

```bash
# RAR 自解压文件（常见于视觉小说、日式游戏）
sudo dnf install unrar
unrar x 游戏名.part1.exe ~/Games/游戏名/

# 普通 RAR 分卷
unrar x 游戏名.part1.rar ~/Games/游戏名/

# 7z 压缩包
7z x 游戏名.7z -o~/Games/游戏名/

# ISO 光盘镜像（需要挂载或直接解压）
7z x 游戏名.iso -o~/Games/游戏名/
```

解压后的目录需要手动确认可执行文件路径（通常为 `游戏名.exe` 或 `游戏名/游戏名.exe`）。

## 6. 运行程序

```bash
cd /path/to/game
WINEPREFIX=~/.wine-名称 wine 程序名.exe
```

## 7. 使用 Lutris 管理游戏

Lutris 是一个游戏管理器，可以自动管理 Wine 容器、版本和配置。

```bash
sudo dnf install lutris
```

添加游戏：

1. 打开 Lutris
2. 点击 **+** → **添加本地游戏**
3. 填写信息

  | 字段 | 说明 |
  | ------ | ------ |
  | 名称 | 游戏名 |
  | 运行器 | Wine |
  | 可执行文件 | 指向游戏 EXE 的完整路径 |
  | 工作目录 | 游戏所在目录 |
  | Wine 容器 | 指向已配置好的 Wine prefix 路径 |

## 8. 游戏手柄配置

Wine 对部分国产手柄兼容性不佳。最佳方案是使用 AntiMicroX 将手柄按键映射为键盘操作：

```bash
sudo dnf install antimicrox
antimicrox
```

对于视觉小说游戏，建议按键映射：

| 手柄按键 | 键盘映射 | 功能 |
| --------- | --------- | ------ |
| A 键 | Enter | 推进对话 |
| B 键 | Esc | 退出/菜单 |
| 方向键 | ↑↓←→ | 选项 |
| LB / RB | Ctrl | 快进 |
| Start | Space | 隐藏文本框 |

## 9. 常见问题

- **`WINEARCH=win32` 报错**：Wine 11.0 不支持此参数，直接去掉即可
- **字体乱码**：运行 `winetricks fakejapanese_ipamona` 或 `cjkfonts`
- **GPU 启动失败**：可能需要 `--no-sandbox` 或 `--disable-gpu` 参数
- **手柄不被识别**：用 AntiMicroX 映射为键盘操作
