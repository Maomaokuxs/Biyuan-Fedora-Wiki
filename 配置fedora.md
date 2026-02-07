# 说明

这是一篇系统配置板块的整合文档，通常是提供一个思路，具体的内容是在目录的其他文件中，我使用的是 kde 桌面环境，美化通常没有，一般美化的尽头是默认，也是因为实力有限。

## 1.更新系统

```bash
sudo dnf upgrade
# 更新完成后重启系统
```

## 2.安装文本编辑器vim和git

```bash
sudo dnf install vim git 
```

## 3.安装输入法

```bash
sudo dnf install fcitx5 fcitx5-chinese-addons fcitx5-configtool
```

`fcitx5`：输入法主程序；
`fcitx5-chinese-addons`：额外中文包，提供拼音五笔等输入；
`fcitx5-configtool`：输入法配置工具。
