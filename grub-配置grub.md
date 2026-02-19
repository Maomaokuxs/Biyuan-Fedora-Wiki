# 说明

- fedora grub 默认配置中将不显示内核调试输出以及控制台日志，下面提供一些命令来显示这些内容。

- 我的修改前 grub 配置文件内容，配置文件路径：/etc/default/grub

```txt
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="rd.luks.uuid=luks-aea5a643-311b-4a15-b4e9-08940ff9577b rhgb quiet rd.driver.blacklist=nouveau,nova_core modprobe.blacklist=nouveau,nova_core"
GRUB_DISABLE_RECOVERY="true"
GRUB_ENABLE_BLSCFG=true

#GRUB_DISABLE_OS_PROBER=false
```

## 1. (可选) 显示内核调试输出以及控制台日志

### 1.1 使用 root 权限编辑 /etc/default/grub

```bash
sudo vim /etc/default/grub
```

找到 GRUB_CMDLINE_LINUX 行，将其中的 rhgb 和 quiet 删除。修改后的行可能类似于：

```text
GRUB_CMDLINE_LINUX="rd.luks.uuid=luks-aea5a643-311b-4a15-b4e9-08940ff9577b rd.driver.blacklist=nouveau,nova_core modprobe.blacklist=nouveau,nova_core"
```

`rhgb (Red Hat Graphical Boot)`：启用图形化启动界面，显示 Fedora 的徽标和进度条，隐藏内核和服务的详细输出。

`quiet`：抑制大部分内核消息，只显示严重错误，使启动界面更简洁。

### 1.2 (可选) 如果需要更详细的调试输出，还可以添加以下参数

`loglevel=5`：设置控制台日志级别为5。

`debug`：启用内核调试输出。

例如：

```text
GRUB_CMDLINE_LINUX="rd.luks.uuid=... loglevel=5 debug"
```

### 1.3 重新生成 GRUB 配置文件

```bash
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

## 2.搜索其他操作系统

### 2.0 安装 os-prober 包

```bash
sudo dnf install os-prober
```

### 2.1 编辑配置文件

```bash
sudo vim /etc/default/grub
```

### 2.2 在文件底部添加

```text
GRUB_DISABLE_OS_PROBER=false
```

### 3.重新生成 GRUB 配置文件

```bash
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```
