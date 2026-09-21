# 说明

对小企鹅输入法对简单配置。

## 安装 Fcitx5 软件包及工具

```bash
sudo dnf install fcitx5 fcitx5-chinese-addons fcitx5-configtool
```

- `fcitx5`：Fcitx5 输入法主程序；
- `fcitx5-chinese-addons`：Fcitx5 额外中文包，提供拼音五笔等输入；
- `fcitx5-configtool`：Fcitx5 输入法配置工具；
- （可选）`kcm-fcitx5` : 将 Fcitx5 输入法配置集成到 KDE 桌面环境中的设置中。

---

## 使用中州韵 + 雾凇拼音方案

1. 安装中州韵引擎

    ```bash
    sudo dnf install fcitx5-rime
    ```

2. 获取雾凇拼音配置文件压缩包

- 下载压缩包

  ```bash
  wget https://github.com/iDvel/rime-ice/archive/refs/heads/main.zip -O /tmp/rime-ice.zip
  ```

- 解压（如果没安装 unzip 请 sudo dnf install unzip）

  ```bash
  unzip /tmp/rime-ice.zip -d /tmp/
  ```

- 移动文件到 rime 目录

  ```bash
  cp -rv /tmp/rime-ice-main/* ~/.local/share/fcitx5/rime/
  ```

- 创建自定义配置（防止你的个性化设置被后续更新覆盖）：

  ```Bash
  touch ~/.local/share/fcitx5/rime/default.custom.yaml

  并写入：

  patch:
    schema_list:
      - schema: rime_ice
  ```

- 强制重启 fcitx5

  ```bash
  fcitx5 -r -d
  ```

- 在 fcitx5 输入法中选择 Rime

  ```bash
  fcitx5-config
  ```

- 等待配置生效

```text
1.切换输入法到 Rime (默认 Ctrl + Space)。

2.你会发现可能打不出字，或者只有一个 Rime 图标。

3.耐心等待
```

- 清理临时文件

```bash
rm /tmp/rime-ice.zip
rm -rf /tmp/rime-ice-main
```

- 引用：

  - [中州韵](https://rime.im/)
  - [雾凇拼音](https://github.com/iDvel/rime-ice)
