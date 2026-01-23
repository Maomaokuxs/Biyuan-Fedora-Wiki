
- 将中文名称改为相对应的英文

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

- 自动修改

- 安装软件包

```
sudo dnf install xdg-user-dirs-gtk-update
```

- 执行命令

```bash
xdg-user-dirs-update
```

- 手动安装

- 编辑`~/.config/user-dirs.locale`

```bash
vim ~/.config/user-dirs.locale
```

- 修改为

``` 
XDG_DESKTOP_DIR="$HOME/Desktop"
XDG_DOWNLOAD_DIR="$HOME/Downloads"
XDG_DOCUMENTS_DIR="$HOME/Documents"
XDG_MUSIC_DIR="$HOME/Music"
XDG_PICTURES_DIR="$HOME/Pictures"
XDG_VIDEOS_DIR="$HOME/Videos"
XDG_PUBLICSHARE_DIR="$HOME/Public"
XDG_TEMPLATES_DIR="$HOME/Templates"
```