# 说明

部分用户有双系统的需求，对于我来说 fedora 在当前来说是我的日常使用系统，但往往有些游戏并不支持启动，当然有一些解决方案，但是我还是希望有些事情简单点，所以下文用于解决一些共存方面的问题。

## 1.双系统时间不一致问题

### 1.1 现象

对于北京时间（UTC+8），linux 与 windows 之间会相差八个小时，而且 windows 在启动系统时并不会自行同步时间，需要在设置里手动同步时间，在下一次进入 linux 之后进入 windows 时间又会恢复原状。

### 1.2 原因

因为两个系统对硬件时钟的解释不同，所以导致时间不同步。

- Linux（如 Fedora）： 默认将硬件时钟看作 UTC（协调世界时），然后根据时区（比如北京时间 UTC+8）在系统内进行转换。

- Windows： 默认将硬件时钟直接看作 Local Time（本地时间）。

### 1.3 解决办法

- 修改 Windows 使用 UTC，右键 windows 的徽标打开命令提示符（管理员），或者直接搜索并打开。

- 输入以下命令并回车

这将会修改注册表，告诉 Windows 硬件时钟是 UTC。

```text
reg add HKLM\SYSTEM\CurrentControlSet\Control\TimeZoneInformation /v RealTimeIsUniversal /t REG_DWORD /d 1
```

- 手动在“设置 -> 时间和语言 -> 日期和时间”中手动点击“立即同步”。

- 重启系统并进入 bios 查看当前时间，对于北京时间（UTC+8）是不是与现在相差八个小时，是就说明修改成功了。

- (可选) 修改启动项，将 fedora 设置为第一位，fedora 默认使用的 GRUB 2 这不仅可以启动 windows 也可以启动 linux 。

## 2.安全启动导致无法加载启动 U 盘 或者想要在 fedora linux 系统上实现安全启动

### 2.1 现象

在制作好 fedora 的启动 U 盘 后并设置了启动项，但是依旧不能加载。

### 2.2 原因

部分主板是默认开启安全启动，在开启后，主板只会引导经过微软签名的 shim 程序，fedora 是支持安全启动的，但可能因为制作启动 U 盘时可能出现了问题，导致无法启动。

### 2.3 解决办法

- 最简单的就是直接将安全启动关闭，如果有需要可以安装完之后看 [[fedora-linux-secure-boot-实现安全启动]]来实现相关功能。

- 如果你就是要不关闭安全启动去安装系统，那么使用用官方的 U 盘制作工具[Fedora Media Writer](https://docs.fedoraproject.org/en-US/fedora/latest/preparing-boot-media/#fedora_media_writer)。

## 3.开启了 windows 快速启动 linux 无法挂载  ntfs 文件系统的分区

### 3.1 原因

如果 Windows 启用了“快速启动 (Fast Startup)”或处于休眠状态，NTFS 分区会被锁定为只读。

### 3.2 解决办法

进入 windows 系统关闭安全启动。

参考文档

- [debian-cookbook](https://github.com/smgdream/debian-cookbook/blob/main/improve/deb+win.md)