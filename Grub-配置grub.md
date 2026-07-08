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

# 记忆所选启动项
GRUB_SAVEDEFAULT="true"
#GRUB_DISABLE_OS_PROBER=false
```

- 一步或多步骤修改都要重新生成 GRUB 配置文件

---

## (可选) 显示内核调试输出以及控制台日志

### 1. 使用 root 权限编辑 /etc/default/grub

```bash
sudo vim /etc/default/grub
```

找到 GRUB_CMDLINE_LINUX 行，将其中的 rhgb 和 quiet 删除。修改后的行可能类似于：

```text
GRUB_CMDLINE_LINUX="rd.luks.uuid=luks-aea5a643-311b-4a15-b4e9-08940ff9577b rd.driver.blacklist=nouveau,nova_core modprobe.blacklist=nouveau,nova_core"
```

`rhgb (Red Hat Graphical Boot)`：启用图形化启动界面，显示 Fedora 的徽标和进度条，隐藏内核和服务的详细输出。

`quiet`：抑制大部分内核消息，只显示严重错误，使启动界面更简洁。

### 2. (可选) 如果需要更详细的调试输出，还可以添加以下参数

`loglevel=5`：设置控制台日志级别为5。

`debug`：启用内核调试输出。

例如：

```text
GRUB_CMDLINE_LINUX="rd.luks.uuid=... loglevel=5 debug"
```

### 3. 重新生成 GRUB 配置文件

```bash
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

---

## 搜索其他操作系统

### 1. 安装 os-prober 包

```bash
sudo dnf install os-prober
```

### 2. 编辑配置文件

```bash
sudo vim /etc/default/grub
```

### 3. 在文件底部添加

```text
GRUB_DISABLE_OS_PROBER=false
```

---

## 记忆上次启动所选启动项

### 1.编辑 GRUB 配置

```bash
sudo vi /etc/default/grub

# 分辨率和刷新率根据实际需求设置，如果不添加没有问题。
找到 GRUB_CMDLINE_LINUX 这一行，在末尾添加（注意在引号内）：
video=1920x1080@60


# 记忆上一次所选启动项
GRUB_SAVEDEFAULT="true"

# 用于配合扫描其他操作系统
GRUB_DISABLE_OS_PROBER=false
```

---

## 美化GRUB

### 1.下载并解压

### 2. 将 Tribbie 复制到 GRUB 主题目录

   ```bash
   sudo cp -r Tribbie /usr/share/grub/themes/
   ```

### 3.编辑 GRUB 配置文件

```bash
sudo vim /etc/default/grub
```

### 4.在文本文件末尾添加主题配置

```bash
GRUB_THEME="/usr/share/grub/themes/Tribbie/theme.txt"
```

---

## grubenv 损坏修复

启动时显示 `invalid environment block` 错误时，说明 `/boot/grub2/grubenv` 文件损坏。

```bash
# 删除损坏的 grubenv
sudo rm -f /boot/grub2/grubenv

# 重建空的 grubenv
sudo grub2-editenv /boot/grub2/grubenv create

# 重新生成 GRUB 配置文件
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

---

## 重新生成 GRUB 配置文件

```bash
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

---
