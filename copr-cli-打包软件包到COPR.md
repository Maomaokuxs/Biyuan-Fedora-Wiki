# 说明

COPR（Cool Other Package Repo）是 Fedora 社区的第三方软件包构建服务。你可以将自己的软件包源码提交到 COPR，它会自动在云端构建 RPM，生成可供他人使用的 yum/dnf 仓库。

## 1. 准备工作

- GitHub 账号（用于登录 COPR）
- 完整的 RPM spec 文件
- 源码包或 git 仓库地址

## 2. 配置 copr-cli

- **安装 copr-cli**

  ```bash
  sudo dnf install copr-cli
  ```

- **获取 API Token**

  浏览器打开 `https://copr.fedorainfracloud.org/api/`，用 GitHub 登录，复制生成的 Token。

- **配置 Token**

  ```bash
  mkdir -p ~/.config
  cat > ~/.config/copr << 'EOF'
  [copr-cli]
  login = 你的login
  username = 你的用户名
  token = 你的token
  copr_url = https://copr.fedorainfracloud.org
  EOF
  ```

- **验证**

  ```bash
  copr-cli list
  ```

## 3. 创建 COPR 项目

```bash
copr-cli create 项目名 \
  --description "项目描述" \
  --chroot fedora-44-x86_64
```

`--chroot` 指定构建目标，可以多次使用来支持多个 Fedora 版本：

```bash
copr-cli create 项目名 \
  --chroot fedora-43-x86_64 \
  --chroot fedora-44-x86_64
```

## 4. 从本地 SRPM 构建

- **准备 spec 文件**

  spec 文件需要包含完整的构建信息，至少需要：

  ```spec
  Name:           软件包名
  Version:        版本号
  Release:        1%{?dist}
  Summary:        简短描述
  License:        许可证
  URL:            项目主页
  Source0:        %{name}-%{version}.tar.gz
  ```

  如果源码需要从 GitHub 下载，Source0 写完整 URL：

  ```spec
  Source0: https://github.com/用户名/仓库名/archive/refs/tags/%{version}.tar.gz
  ```

- **使用 tito 构建 SRPM**

  如果项目已有 `.tito` 配置，可以用 tito 快速打包：

  ```bash
  # 安装 tito
  sudo dnf install tito

  # 创建 tag（格式：软件包名-版本-释出号）
  git tag 软件包名-0.6-1

  # 构建 SRPM（--offline 跳过远程 tag 检查）
  tito build --srpm --offline
  ```

  SRPM 生成在 `/tmp/tito/` 目录下。

- **提交到 COPR**

  ```bash
  copr-cli build 项目名 /tmp/tito/软件包名-0.6-1.fc44.src.rpm
  ```

## 5. 从 GitHub 直接构建

也可以跳过本地构建，让 COPR 直接从 GitHub 拉取源码：

```bash
copr-cli buildscm 项目名 \
  --clone-url https://github.com/用户名/仓库名 \
  --method rpkg \
  --commit main
```

参数说明：

| 参数 | 说明 |
| ------ | ------ |
| `--clone-url` | git 仓库地址 |
| `--method` | 构建方法，`rpkg` 或 `tito` |
| `--commit` | 分支名、tag 名或提交哈希 |
| `--spec` | spec 文件路径（默认自动查找） |

## 6. 查看构建状态

```bash
# 查看所有构建
copr-cli list-builds 项目名

# 监视正在进行的构建
copr-cli watch-build 构建ID
```

构建成功后在 `https://copr.fedorainfracloud.org/coprs/用户名/项目名/` 可以看到仓库详情。

## 7. 使用构建好的软件包

启用仓库后即可安装：

```bash
sudo dnf copr enable 用户名/项目名
sudo dnf install 软件包名
```

## 8. 常见问题

- **Source0 无法下载**

  原因：spec 中的 Source0 没有完整的 URL。COPR 构建环境不会自动从 git 获取源码，必须给出可下载的完整地址。

  ```spec
  # 错误写法（COPR 找不到文件）
  Source0: %{name}-%{version}.tar.gz

  # 正确写法（需要完整 URL）
  Source0: https://github.com/用户名/仓库名/archive/refs/tags/v%{version}.tar.gz
  ```

- **chroots 错误**

  创建项目时须指定 `--chroot`：

  ```bash
  copr-cli create 项目名
  # Error: chroots: '[]' is not a valid choice

  copr-cli create 项目名 --chroot fedora-44-x86_64
  # 成功
  ```

- **Tag 不存在**

  tito 需要正确的 git tag 格式。默认格式为 `软件包名-版本-释出号`：

  ```bash
  git tag dnf5-autosnapper-0.6-1
  tito build --srpm --offline
  ```

- **配置文件中缺少节标题**

  ```bash
  # 错误配置（缺少 [copr-cli]）
  login = xxx
  username = xxx

  # 正确配置
  [copr-cli]
  login = xxx
  username = xxx
  token = xxx
  copr_url = https://copr.fedorainfracloud.org
  ```
