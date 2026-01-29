# 说明

提供一些模板，方便手动创建一些桌面快捷方式。应该有一些更加方便的应用程序，后续应该会更新。

- 创建桌面启动器：

```bash
sudo vim /usr/share/applications/idea.desktop
```

- 添加以下内容：

```text
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

- 登出刷新图标
