# 说明

这篇文档旨在快速的查阅一些dnf命令，如果你擅长英文的话可以使用`man dnf`直接查看帮助手册。

## 1. 应用程序管理命令

```bash
# 安装程序
dnf install <程序名>

# 卸载程序
dnf remove <程序名>

# 重装程序
dnf reinstall <程序名>

# 安装历史
dnf histoty list
```

## 2. 仓库管理命令

```bash
# 显示已启用软件仓库
dnf repo list

# 显示所有仓库
dnf repo list --all

#启用fedora官方仓库及RPM FUDION仓库
sudo dnf config-manager setopt <仓库ID>.enabled=1

#禁用仓库
sudo dnf config-manager setopt <仓库ID>.enabled=0

#启用fedora copr仓库
sudo dnf copr enable 维护者/仓库名称
#例如
sudo dnf copr enable vaniiiiii/kaczynski-ted 
```

|    特性      |  COPR        | Fedora 官方仓库 | RPM Fusion     |
| :-------:    | :----:       | :---------:     | :--------:     |
| **维护者**   | 个人/社区    | Fedora 官方团队 |    社区组织    |
|  **审核**    | 基本自动检查 |   严格审核流程  |    中等审核    |
| **稳定性**   |  参差不齐    |    高稳定性     |   一般较稳定   |
| **支持级别** | 无官方支持   |    完全支持     |    社区支持    |

### 2.1 COPR（第三方）

- **全称**：Cool Other Package Repositories

- **性质**：**Fedora 官方提供的平台**，但承载的是**第三方内容**

- **类比**：类似于 Ubuntu 的 PPA、Arch 的 AUR

- **维护者**：任何 Fedora 用户都可以创建和维护 COPR 仓库

### 官方支持但非官方内容

- **平台是官方的**：COPR 服务由 Fedora 基础设施团队维护

- **内容是第三方的**：仓库中的软件包由个人或社区维护

- **质量不一**：软件包质量取决于维护者，未经过官方全面测试

### 2.2 Fedora 官方仓库

- **性质**：Fedora 项目官方维护

- **内容**：只包含完全自由和开源软件

- **政策**：严格遵守 Fedora 的 FOSS 准则

### 2.3 RPM Fusion（第三方）

- **性质**：由社区维护的非官方仓库

- **内容**：包含 Fedora 因法律、专利或政策原因不能直接分发的软件

  - 多媒体编解码器（如 MP3、H.264、AAC）

  - NVIDIA/AMD 闭源显卡驱动

  - 某些受限软件

- **授权**：提供免费和开源软件，也包含一些闭源软件。
