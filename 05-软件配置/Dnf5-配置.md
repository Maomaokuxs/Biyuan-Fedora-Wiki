# 说明

这一节用于对包管理器进行一些微调，用于提升下载速度及使用体验。

## 编辑配置文件

```bash
sudo vim /etc/dnf/dnf.conf
```

## 完整配置

```bash
[main]
# RPM 包 GPG 签名验证（Fedora 默认 True）
gpgcheck=True
# 同时保留的内核数量（Fedora 默认 3）
installonly_limit=3
# 卸载时自动清理孤立依赖（Fedora 默认 True）
clean_requirements_on_remove=True
# 并行下载数
max_parallel_downloads=10
# 自动选择最快镜像
fastestmirror=True
# 终端彩色输出
color=always
# 防止卡镜像：速度低于 10kB/s 持续 30 秒则切换
minrate=10k
timeout=30
retries=2
```
