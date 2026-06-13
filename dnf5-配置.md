# 说明

这一节用于对包管理器进行一些微调，用于提升下载速度及使用体验。

## 编辑配置文件

```bash
sudo vim /etc/dnf/dnf.conf
```

## 完整配置

```bash

[main]
gpgcheck=True
installonly_limit=2
clean_requirements_on_remove=True
# 开启最高并行下载数（默认是 3，最大 20）
max_parallel_downloads=10
# 自动选择最快镜像源
fastestmirror=True

```
