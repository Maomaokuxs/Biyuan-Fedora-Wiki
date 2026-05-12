# 说明

可以在 fedora 上使用 nix 包管理器来安装更多的软件供使用。

## 配置环境

```bash
# 安装 Fedora 仓库中的 Nix 及其守护进程
sudo dnf install nix nix-daemon

# 启动并设置开机自启
sudo systemctl enable --now nix-daemon

# 将自己加入 nixbld 组（可能需要重新登录生效）
sudo usermod -aG nixbld $USER
```

## 测试环境

```bash
nix shell nixpkgs#hello -c hello
```

## 常用命令

| 功能    | 命令                           | 备注                  |
|-------|------------------------------|---------------------|
| 搜索软件  | nix search nixpkgs <关键词>     | 在本地索引中查找包名          |
| 安装软件  | nix profile add nixpkgs#<包名> | 类似于 dnf install     |
| 查看已安装 | nix profile list             | 列出所有通过 profile 安装的包 |
| 卸载软件  | nix profile remove <包名或索引>   | 索引号可以通过 list 查看     |
| 升级所有包 | nix profile upgrade '.*'     | 升级 profile 中的所有软件包  |

- 参考：

  [Changes/Nix package tool](https://fedoraproject.org/wiki/Changes/Nix_package_tool)