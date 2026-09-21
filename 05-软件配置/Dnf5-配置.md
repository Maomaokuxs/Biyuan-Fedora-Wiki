# 说明

这一节用于对包管理器进行一些微调，用于提升下载速度及使用体验。

## 编辑配置文件

```bash
sudo vim /etc/dnf/dnf.conf
```

## 完整配置

```bash
[main]

# 开启 RPM 包的 GPG 签名验证,防止软件包在传输过程中被篡改或源被劫持
gpgcheck=True

# 限制系统同时保留的内核（Kernel）最大数量
installonly_limit=3

# 卸载软件时，自动清理因其而被引入、但现在已无用的依赖包（孤立包）
clean_requirements_on_remove=True

# 开启最高并行下载数（默认是 3，最大 20）
max_parallel_downloads=10

# 自动选择最快镜像源
fastestmirror=True

# 终端输出使用彩色字体
color=always

# 防止卡镜像
minrate=10k      # 低于10kB/s就判为卡死
timeout=30       # 卡30秒就切镜像
retries=2        # 单个镜像重试2次就换
```
