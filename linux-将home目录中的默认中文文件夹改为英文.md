# 说明

如果自动更改的方式不行直接使用手动更改，优先选择自动，无效再选择手动。

## 1.重命名

- 将中文目录重命名和启动的文件移动至新文件夹

```bash
cd
mv 公共 Public
mv 文档 Documents
mv 模板 Templates
mv 音乐 Music
mv 图片 Pictures
mv 视频 Video
mv 下载 Downloads
mv 桌面 Desktop
```

## 2.自动修改

```bash
# 1.安装软件包
sudo dnf install xdg-user-dirs-gtk-update

# 2.临时将语言设置为英文
export LC_ALL=en_US.UTF-8
# 在重启或者注销后失效

# 3.更新目录配置
xdg-user-dirs-update --force 

# 4.如果使用的是 ked 桌面环境需要在文件管理器中重新配置标签页对应的路径
```

## 3.（可选）手动修改

```bash
# 1.编辑`~/.config/user-dirs.locale`
vim ~/.config/user-dirs.locale

# 2.修改为
XDG_DESKTOP_DIR="$HOME/Desktop"
XDG_DOWNLOAD_DIR="$HOME/Downloads"
XDG_DOCUMENTS_DIR="$HOME/Documents"
XDG_MUSIC_DIR="$HOME/Music"
XDG_PICTURES_DIR="$HOME/Pictures"
XDG_VIDEOS_DIR="$HOME/Videos"
XDG_PUBLICSHARE_DIR="$HOME/Public"
XDG_TEMPLATES_DIR="$HOME/Templates"

# 3.如果使用的是 ked 桌面环境需要在文件管理器中重新配置标签页对应的路径
```
