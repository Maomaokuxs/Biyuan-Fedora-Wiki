# 说明

本文总结将应用程序打包为 RPM 并上传到 COPR 仓库的完整流程。内容来源于实际打包 kazumi（Flutter）、hmcl（Java）、splayer/bilibili（Electron）、rime-ice（Rime 输入法方案）等应用的经验。

## COPR 是什么

COPR 是 Fedora 官方的个人/第三方软件仓库平台，任何人都可以上传 SRPM 或通过 CLI 触发构建，用户通过 `dnf copr enable` 启用仓库后直接用 DNF 安装。

## 准备工作

```bash
# 安装必要工具
sudo dnf install rpm-build rpmdevtools copr-cli mock

# 创建 RPM 构建目录结构
rpmdev-setuptree

# 登录 COPR（浏览器打开链接完成认证）
copr-cli login
```

## 创建 COPR 仓库

```bash
# 创建仓库（名称建议简短，如 apps）
copr-cli create apps --chroot fedora-44-x86_64 --description "个人应用仓库" --repo "https://copr.fedorainfracloud.org/coprs/你的用户名/apps/"
```

## 编写 spec 文件

### 通用模板（适用于 tar.gz 二进制重打包）

```spec
Name:       myapp
Version:    1.0.0
Release:    1%{?dist}
Summary:    应用简介
License:    GPL-3.0
URL:        https://github.com/owner/myapp

# 指向 GitHub Release 的 tar.gz
Source0:    https://github.com/owner/myapp/releases/download/v%{version}/myapp_linux_%{version}_x86_64.tar.gz
Source1:    %{name}.desktop
Source2:    %{name}.png

ExclusiveArch: x86_64

# 运行时依赖
Requires:   gtk3

%description
应用的详细描述。

%prep
%setup -q -n %{name}

%install
# 创建目标目录
mkdir -p %{buildroot}%{_libdir}/%{name}
mkdir -p %{buildroot}%{_bindir}
mkdir -p %{buildroot}%{_datadir}/applications
mkdir -p %{buildroot}%{_datadir}/icons/hicolor/1024x1024/apps

# 复制二进制与库文件
cp -r * %{buildroot}%{_libdir}/%{name}/

# 创建符号链接到 /usr/bin
ln -s %{_libdir}/%{name}/%{name} %{buildroot}%{_bindir}/%{name}

# 安装桌面文件和图标
install -m 644 %{SOURCE1} %{buildroot}%{_datadir}/applications/
install -m 644 %{SOURCE2} %{buildroot}%{_datadir}/icons/hicolor/1024x1024/apps/%{name}.png

%files
%{_bindir}/%{name}
%{_libdir}/%{name}/
%{_datadir}/applications/%{name}.desktop
%{_datadir}/icons/hicolor/1024x1024/apps/%{name}.png

%changelog
* %{date} %{?packager} - %{version}-%{release}
- 初始打包
```

### Java 应用（jar 类型）

```spec
Name:       hmcl
Version:    3.15.2
Release:    1%{?dist}
Summary:    Hello Minecraft! Launcher
License:    GPL-3.0
URL:        https://github.com/HMCL-dev/HMCL
Source0:    https://github.com/HMCL-dev/HMCL/releases/download/v%{version}/HMCL-%{version}.jar
Source1:    %{name}.desktop
Source2:    %{name}.png

ExclusiveArch: x86_64
Requires:   java

%description
HMCL 是一个 Minecraft 启动器。

%prep
%setup -q -c -T
cp %{SOURCE0} %{name}.jar

%install
mkdir -p %{buildroot}%{_libdir}/%{name}
mkdir -p %{buildroot}%{_bindir}
mkdir -p %{buildroot}%{_datadir}/applications
mkdir -p %{buildroot}%{_datadir}/icons/hicolor/1024x1024/apps

cp %{name}.jar %{buildroot}%{_libdir}/%{name}/

# 生成启动脚本
cat > %{buildroot}%{_bindir}/%{name} << 'SCRIPT'
#!/bin/sh
exec java -Dglass.gtk.uiScale=1.5 -jar /usr/lib64/hmcl/hmcl.jar
SCRIPT
chmod +x %{buildroot}%{_bindir}/%{name}

install -m 644 %{SOURCE1} %{buildroot}%{_datadir}/applications/
install -m 644 %{SOURCE2} %{buildroot}%{_datadir}/icons/hicolor/1024x1024/apps/%{name}.png

%files
%{_bindir}/%{name}
%{_libdir}/%{name}/
%{_datadir}/applications/%{name}.desktop
%{_datadir}/icons/hicolor/1024x1024/apps/%{name}.png
```

### 纯数据包（noarch，如输入法方案）

```spec
Name:       rime-ice
Version:    2025.05.21
Release:    1%{?dist}
Summary:    雾凇拼音 - Rime 简体中文输入方案
License:    GPL-3.0
URL:        https://github.com/iDvel/rime-ice
Source0:    https://github.com/iDvel/rime-ice/archive/refs/tags/%{version}.tar.gz

BuildArch:  noarch
Requires:   rime

%description
Rime 输入法配置方案。

%prep
%autosetup -n rime-ice-%{version}

%install
mkdir -p %{buildroot}%{_datadir}/rime-data/%{name}
cp -r * %{buildroot}%{_datadir}/rime-data/%{name}/

%post
# 安装后自动部署到当前用户的 Rime 目录
if [ -n "$SUDO_USER" ]; then
    USER_HOME=$(getent passwd "$SUDO_USER" | cut -d: -f6)
    RIME_DIR="$USER_HOME/.local/share/fcitx5/rime"
    if [ -d "$RIME_DIR" ]; then
        ln -sf %{_datadir}/rime-data/%{name}/* "$RIME_DIR/" 2>/dev/null || true
    fi
fi

%files
%{_datadir}/rime-data/%{name}/

%changelog
* %{date} %{?packager} - %{version}-%{release}
- 初始打包
```

## 构建并上传

```bash
# 1. 将 spec 和相关资源放入正确位置
cp myapp.spec ~/rpmbuild/SPECS/
cp myapp.desktop logo.png ~/rpmbuild/SOURCES/

# 2. 构建源码包（生成 .src.rpm）
rpmbuild -bs ~/rpmbuild/SPECS/myapp.spec

# 3. 上传到 COPR
copr-cli build apps ~/rpmbuild/SRPMS/myapp-*.src.rpm

# 4. 等待 COPR 构建完成（可在网页查看）
# https://copr.fedorainfracloud.org/coprs/你的用户名/apps/
```

## 桌面文件和元数据

### desktop 文件

```desktop
[Desktop Entry]
Name=MyApp
Comment=应用简介
Exec=myapp
Icon=myapp
Terminal=false
Type=Application
Categories=Utility;
```

### AppStream 元数据（可选）

```xml
<?xml version="1.0" encoding="utf-8"?>
<component type="desktop-application">
  <id>myapp</id>
  <name>MyApp</name>
  <summary>应用简介</summary>
  <metadata_license>GPL-3.0</metadata_license>
  <project_license>GPL-3.0</project_license>
</component>
```

## 更新版本

```bash
# 1. 修改 spec 中的 Version 和 Release
vim ~/rpmbuild/SPECS/myapp.spec

# 2. 下载新版 tar.gz 到 SOURCES
curl -sL "https://github.com/owner/myapp/releases/download/v新版/...tar.gz" \
  -o ~/rpmbuild/SOURCES/myapp_新版.tar.gz

# 3. 构建并上传
rpmbuild -bs ~/rpmbuild/SPECS/myapp.spec
copr-cli build apps ~/rpmbuild/SRPMS/myapp-*-1.fc44.src.rpm
```

## 用户安装方式

```bash
# 启用仓库
sudo dnf copr enable 用户名/apps

# 安装软件
sudo dnf install myapp

# 更新
sudo dnf upgrade myapp
```

## 不同类型应用的打包策略

| 类型 | 打包策略 | COPR 是否合适 |
| ------ | --------- | ------------- |
| tar.gz 二进制（如 kazumi、splayer） | 解压到 `/usr/lib64/%{name}/`，symlink 到 `/usr/bin/` | ✅ 合适 |
| jar（如 hmcl） | 复制 jar + 生成 shell wrapper | ✅ 合适 |
| noarch 数据包（如 rime-ice） | 复制到 `/usr/share/`，%post 脚本部署 | ✅ 合适 |
| 源码编译（如 Rust/C 项目） | `%build` 阶段编译 | ✅ 合适但耗时较长 |
| AppImage | 不适合 COPR（构建环境无 FUSE，大文件易截断） | ❌ 建议 opt-mgr 本地管理 |
| 已有 RPM | 本身就是可安装包，用 DNF 或 opt-mgr 管理 | ❌ 无需二次打包 |

## 注意事项

- **依赖解析**：RPM 的 `find-requires` 会自动从 ELF 二进制扫描 soname 依赖，无需手动指定，但自带的 `.so` 需要排除（`%global _requires_exceptions lib*.so`）
- **RPATH 清理**：Flutter 等框架打包的二进制可能嵌入 CI 机器的 RPATH，需用 `chrpath -d` 清理并设置 `$ORIGIN`
- **COPR 构建环境**：是隔离的 mock chroot，无网络（只能访问 Source URL）、无 FUSE、无 GPU
- **图标 URL**：GitHub 图标需用 raw 链接（`raw.githubusercontent.com` 而非 `github.com` 的 blob 页面）
