- 创建桌面启动器：

```bash
sudo vim /usr/share/applications/idea.desktop
```

- 添加以下内容：

```
[Desktop Entry]
Version=1.0
Type=Application
Name=IntelliJ IDEA
Exec=/opt/idea/bin/idea.sh
Icon=/opt/idea/bin/idea.png
Comment=Java IDE
Categories=Development;IDE;
Terminal=false
StartupWMClass=jetbrains-idea
```

- 刷新图标
	
	登出
