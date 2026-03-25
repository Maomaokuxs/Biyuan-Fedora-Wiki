# 说明

这是一篇系统配置板块的整合文档，只提供部分内容，具体的内容部分是在其他章节中，我使用的是 KDE 桌面环境，美化通常没有，也是因为实力有限。

## 0. (可选) 配置系统级快照

[[snapper-实现快照功能]]

注：在自由的系统中，需要自己确定每次执行的命令是什么功能，如果不确定使用系统快照是明智的选择。

## 1. 安装英伟达显卡驱动

[[安装英伟达显卡驱动并开启硬件编解码]]

注：暂时只写了安装英伟达显卡驱动，我手头只有英伟达的显卡，其他显卡没法测试。

## 2.配置软件源并更新软件包

```bash
# 1.备份官方软件源
sudo cp /etc/yum.repos.d/fedora.repo /etc/yum.repos.d/fedora.repo.bak
sudo cp /etc/yum.repos.d/fedora-updates.repo /etc/yum.repos.d/fedora-updates.repo.bak

# 2.更新本地缓存
sudo dnf makecache

# 3.替换软件源
sudo sed -e 's|^metalink=|#metalink=|g' \
    -e 's|^#baseurl=http://download.example/pub/fedora/linux|baseurl=http://mirrors.tuna.tsinghua.edu.cn/fedora|g' \
    -i.bak \
    /etc/yum.repos.d/fedora.repo \
    /etc/yum.repos.d/fedora-updates.repo

# 4.更新软件包
sudo dnf upgrade
# 更新完成后重启系统
```

## 3.添加更多的音频相关组件

[[音频相关组件]]

## 4.文本编辑器 Vim 和 Git

### 4.1 安装 Vim 和 Git

```bash
sudo dnf install vim git 
```

### 4.2 基本配置 Git

```bash
# 1.设置用户名和邮箱（提交代码时会用到）
git config --global user.name "名字"
git config --global user.email "邮箱@example.com"

# 2.配置代理
git config --global http.proxy http://代理地址:端口
git config --global https.proxy https://代理地址:端口
```

### 4.3 (可选) 安装 vscodium 或 vscode

```bash
# 1.1 添加 包含 vscodium 的软件仓库
sudo tee -a /etc/yum.repos.d/vscodium.repo << 'EOF'
[gitlab.com_paulcarroty_vscodium_repo]
name=gitlab.com_paulcarroty_vscodium_repo
baseurl=https://paulcarroty.gitlab.io/vscodium-deb-rpm-repo/rpms/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://gitlab.com/paulcarroty/vscodium-deb-rpm-repo/raw/master/pub.gpg
metadata_expire=1h
EOF

# 1.2 安装 vscodium
sudo dnf install codium

# 2.1 下载 vscode 软件包
https://code.visualstudio.com/
# 进入官网下载 .rpm 后缀软件包

# 2.2 安装 vscode 
sudo dnf instasll 软件包路径
```

## 5.安装输入法

### 5.1 安装 Fcitx5 软件包及工具

```bash
sudo dnf install fcitx5 fcitx5-chinese-addons fcitx5-configtool
```

`fcitx5`：输入法主程序；
`fcitx5-chinese-addons`：额外中文包，提供拼音五笔等输入；
`fcitx5-configtool`：输入法配置工具。

### 5.2 在 KDE 设置中启用 Fcitx5 输入法

```text
设置 -> 键盘 -> 虚拟键盘 -> Fcitx5 Wayland 启动器
```

## 6. 配置 firefox 浏览器

```text
右上角三条横线 -> 设置 -> 主页
```

### 6.1 (可选) 移除 Fedora 官方创建的标签页

[[/images/fedora/Fedora-configs/firefox-config-1.png]]

### 6.2 更改搜索引擎为 Bing

[[/images/fedora/Fedora-configs/firefox-config-2.png]]

## 7. 配置 KDE

```text
设置 -> 显示器和监视器
```

### 7.1 修改显示和监视器设置

[[images/fedora/Fedora-configs/kde-config-2.png]]

### 7.2 关闭将鼠标移动至左上角屏幕边缘开启窗口平铺展示

[[images/fedora/Fedora-configs/kde-config-1.png]]
