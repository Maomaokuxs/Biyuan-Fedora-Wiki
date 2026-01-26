# 说明

本篇文章旨在提供 snapper 使用的一些基础方式。

## 1. 安装snapper，snapper-gui

```bash
# 1.启用corp仓库
sudo dnf copr enable gasinvein/snapper-edge

# 2.安装 snapper 和 btrfs-progs
sudo dnf install snapper btrfs-progs

# 3.安装依赖，并克隆仓库文件编译安装
sudo dnf install python3-devel python3-setuptools gtksourceview3
 
git clone https://github.com/ricardo-vieira/snapper-gui/
 
cd snapper-gui/
 
sudo python3 setup.py install
 
 # 4.测试是否能启动软件
 snapper-gui
 
 # 5.安装完之后可以删除克隆的仓库文件，工作文件夹在主目录，默认文件在$HOME/
```

参考文档：

- [snapper-gui](https://github.com/ricardomv/snapper-gui)

## 2. 显示当前挂载的btrfs子卷

``` bash
# 确定能实现快照的子卷
sudo btrfs subvolume list /
```

## 3. 为根目录创建配置文件

``` bash
# 1.创建配置文件
sudo snapper -c root create-config /

# 2.显示当前配置文件
sudo snapper list-configs

# 3.检查root配置文件
sudo snapper -c root get-config

# 注意事项：
# 3.0 遇到报错为：列出配置失败 (reading sysconfig-file failed)，可以尝试下面的操作。
# 3.1 备份现有文件（如果有）
sudo cp /etc/sysconfig/snapper /etc/sysconfig/snapper.backup 2>/dev/null

# 3.2 创建正确的文件
sudo tee /etc/sysconfig/snapper << 'EOF'
# System config for snapper
# See snapper(8) for details

## Path:Systems/Snapper
## Description:System configuration for Snapper
## Type:text
## Default:""
## ServiceRestart:

# Enable/disable cron jobs.
# Disabling the cron jobs is especially useful when
# snapper-timeline.service and snapper-cleanup.service are started by a
# systemd timer or some other external program.
# Possible values: "yes", "no"
SNAPPER_CRON_JOB_TIMELINE=""
SNAPPER_CRON_JOB_CLEANUP=""

# Log level. Possible values 0-3. Higher means more verbose.
SNAPPER_LOGLEVEL="3"

# Enable D-Bus ACL (access control list).
# Disable this if you want to set the ACL yourself.
# Possible values: "yes", "no"
SNAPPER_DBUS_ACL=""

# Parameters for timeline cron job.
SNAPPER_TIMELINE_PARAMS=""

# Parameters for cleanup cron job.
SNAPPER_CLEANUP_PARAMS=""

# Email address for notifications. Empty string means no email is sent.
SNAPPER_EMAIL_FROM=""
SNAPPER_EMAIL_TO=""

# Parameters for email notifications.
SNAPPER_EMAIL_PARAMS=""
EOF

# 3.3 设置权限
sudo chmod 644 /etc/sysconfig/snapper
sudo chown root:root /etc/sysconfig/snapper

# 3.4 修复 SELinux 上下文
# 不执行这一步可能会遇到这种报错：
# Relabeled /etc/sysconfig/snapper from unconfined_u:object_r:etc_t:s0 to unconfined_u:object_r:snapperd_conf_t:s0
sudo restorecon -v /etc/sysconfig/snapper

# 3.5 验证文件
cat /etc/sysconfig/snapper

# 3.6 现在创建配置
sudo snapper -c root create-config /

# 4.0 遇到报错为： 创建配置失败 (creating btrfs subvolume .snapshots failed since it already exists)。
# 4.1 删除相对应的子卷，比如根目录下
# 列出所有快照
sudo btrfs subvolume list /
# 删除根目录下创造的子卷
sudo btrfs subvolume delete /.snapshots
# 删除主目录下创造的子卷
sudo btrfs subvolume delete /home/.snapshots

# 4.2 验证子卷是否删除
sudo btrfs subvolume list /

# 5.0 遇到报错为：创建配置失败 (config already exists)。
# 说明已经创建了对应子卷的快照配置文件，如果要删除配置文件请执行下面命令，将root改为相对应的配置文件ID，请勿直接删除该/etc/snapper/configs/目录下的文件，因为在/etc/sysconfig/snapper中SNAPPER_CONFIGS参数记录着已经创建了的配置文件ID。
sudo snapper -c root delete-config 2>/dev/null
# ⚠️注意：配置文件删除后对应的快照也会被删除。
```

- 根据 `/usr/share/snapper/config-templates/default` 处的默认配置模板创建一个配置文件 `/etc/snapper/configs/root`。
- 在 `/subvolume/.snapshots` 处创建一个子卷，用于存储未来该配置文件产生的子卷。子卷的路径将会是 `/subvolume/.snapshots/#/snapshot`，`#` 是子卷序号。

```text
顶级子卷 (ID 5, 路径 /)
├── @ (ID 259)                    ← 这是你的 "subvolume"
│   ├── (所有根文件系统文件)      ← 你日常使用的文件
│   └── .snapshots (ID 262)       ← Snapper 创建的子卷
│       ├── 1/                    ← 快照 #1 目录
│       │   ├── info.xml          ← 元数据
│       │   └── snapshot/         ← 实际的快照子卷 (ID 263)
│       ├── 2/                    ← 快照 #2
│       │   └── snapshot/         ← 快照子卷 (ID 264)
│       └── ...
```

- 将 `config` 加入到 `/etc/conf.d/snapper` 的 `SNAPPER_CONFIGS` 中。

参考文档：

- [snapper-archwiki](https://wiki.archlinuxcn.org/wiki/Snapper)

## 4. 优化配置文件(结合自身需求，不必与下面相同)

```bash
# 1.查看当前配置
sudo snapper -c root get-config

# 2.优化root配置
sudo snapper -c root set-config ALLOW_GROUPS="wheel"
sudo snapper -c root set-config SYNC_ACL="yes"
sudo snapper -c root set-config SPACE_LIMIT="0.25"
sudo snapper -c root set-config FREE_LIMIT="0.3"
sudo snapper -c root set-config NUMBER_LIMIT="20"
sudo snapper -c root set-config NUMBER_LIMIT_IMPORTANT="5"
sudo snapper -c root set-config TIMELINE_LIMIT_HOURLY="6"
sudo snapper -c root set-config TIMELINE_LIMIT_DAILY="7"
sudo snapper -c root set-config TIMELINE_LIMIT_WEEKLY="2"
sudo snapper -c root set-config TIMELINE_LIMIT_MONTHLY="1"
sudo snapper -c root set-config TIMELINE_LIMIT_YEARLY="0"

# 3.专用 home 配置（更保守）
sudo snapper -c home set-config SPACE_LIMIT="0.15"
sudo snapper -c home set-config TIMELINE_CREATE="no"
sudo snapper -c home set-config NUMBER_LIMIT="10"
sudo snapper -c home set-config BACKGROUND_COMPARISON="no"

# 4查看最终配置
echo "=== 根配置 ==="
sudo snapper -c root get-config | grep -E "(SPACE_LIMIT|NUMBER_LIMIT|TIMELINE_LIMIT)"

echo ""
echo "=== Home 配置 ==="
sudo snapper -c home get-config | grep -E "(SPACE_LIMIT|TIMELINE_CREATE|NUMBER_LIMIT)"
```

## 5. 启用自动服务

``` bash
# 1.1启用定时服务

# 1.2启用并启动服务
sudo systemctl enable --now snapper-timeline.timer
sudo systemctl enable --now snapper-cleanup.timer

# 1.3检查服务状态
sudo systemctl status snapper-timeline.timer
sudo systemctl status snapper-cleanup.timer

# 1.4查看定时计划
sudo systemctl list-timers --all | grep snapper

# 2.1立即运行一次快照创建

# 2.2手动触发时间线快照
sudo systemctl start snapper-timeline.service

# 2.3查看日志
sudo journalctl -u snapper-timeline.service -n 10

# 3.使用dnf命令后自动创建快照

# 3.1安装dnf5-autosnapper
sudo dnf copr enable douglascdev/dnf5-autosnapper && sudo dnf install dnf5-autosnapper
#使用dnf命令并检查快照是否正在生成。
```

参考文档：

- [dnf5-autosnapper](https://github.com/douglascdev/dnf5-autosnapper)

## 6. 快照操作指南

### 6.0 快速预览

```bash
# 1.创建快照
sudo snapper -c root create --description "描述"

# 2.列出快照
sudo snapper -c root list

# 3.删除快照
sudo snapper -c root delete 编号

# 4.比较快照
sudo snapper -c root status 前一个..后一个

# 5.修改配置
sudo snapper -c root set-config 参数=值

# 6.清理空间
sudo snapper -c root cleanup timeline
sudo snapper -c root cleanup number
```

### 6.1 手动创建快照

``` bash
# 1.创建手动快照

# 2.创建 root 快照
sudo snapper -c root create --description "系统更新前"

# 3.创建 home 快照
sudo snapper -c home create --description "重要文件备份"
```

### 6.2  查看和管理快照

```bash
# 1.列出所有快照
sudo snapper -c root list
sudo snapper -c home list

# 2.查看快照详细信息
sudo btrfs subvolume list /.snapshots/

# 3.比较快照差异
sudo snapper -c root status PREVIOUS（前一个快照）..CURRENT（当前快照）

# 4.删除快照
sudo snapper -c root delete SNAPSHOT_NUMBER（快照ID）
```

### 6.3 从快照恢复文件

```bash
# 查看快照内容
sudo ls /.snapshots/SNAPSHOT_NUMBER/snapshot/

# 恢复单个文件
sudo cp /.snapshots/SNAPSHOT_NUMBER/snapshot/path/to/file /path/to/restore

# 比较并恢复
sudo snapper -c root undochange PREVIOUS..CURRENT
```
