# 说明

本篇文章旨在提供 snapper 使用的指南。

## 1. 安装snapper，snapper-gui，btrfs-assistant等软件包

### 1.启用corp仓库

```bash
sudo dnf copr enable gasinvein/snapper-edge
```

### 2.安装 snapper，btrfs-progs 和 btrfs-assistant

1. 安装 snapper 和 btrfs-assistant

    ```bash
    sudo dnf install snapper btrfs-progs btrfs-assistant
    ```

2. (可选)安装依赖，并克隆仓库文件编译安装snapper-gui

    ```bash
    sudo dnf install python3-devel python3-setuptools gtksourceview3
     
    git clone https://github.com/ricardo-vieira/snapper-gui/
     
    cd snapper-gui/
     
    sudo python3 setup.py install
    ```

3. 测试是否能启动软件

    ```bash
    snapper-gui
    ```

4. 安装完之后可以删除克隆的仓库文件，工作文件夹在主目录，默认文件在$HOME/

### 3.(可选)安装 grub-btrfs 和 dnf5-autosnapper

⚠️注意：这两个软件包需要将snapper配置完成后再使用。

1. 安装并启用grub-btrfs

    - 启用第三方仓库

    ```bash
    sudo dnf copr enable kylegospo/grub-btrfs  
    ```

    - 安装 grub-btrfs 软件包

    ```bash
    sudo dnf install grub-btrfs
    ```

    - 启用相关服务

    ```bash
    sudo systemctl enable --now grub-btrfs.service
    ```

    - 确认状态

    ```bash
    sudo systemctl status grub-btrfs.service
    ```

    - 生成 grub2 配置文件

    ```bash
    sudo grub2-mkconfig -o /boot/grub2/grub.cfg
    ```

2. 安装 dnf5-autosnapper

    - 启用第三方仓库

    ```bash
    sudo dnf copr enable douglascdev/dnf5-autosnapper 
    ```

    - 安装软件包

    ```bash
    sudo dnf install dnf5-autosnapper
    ```

参考文档：

- [snapper-gui](https://github.com/ricardomv/snapper-gui)

---

## 2. 显示当前挂载的btrfs子卷

``` bash
# 确定能实现快照的子卷
sudo btrfs subvolume list /
```

---

## 3. 为根目录创建配置文件

### 1. 创建配置文件

```bash
sudo snapper -c root create-config /
```

### 2. 显示当前配置文件

```bash
sudo snapper list-configs
```

### 3. 检查root配置文件

```bash
sudo snapper -c root get-config
```

参考文档：

- [snapper-archwiki](https://wiki.archlinuxcn.org/wiki/Snapper)

---

## 4. 优化配置文件(结合自身需求，不必与下面相同)

示例：

```text
$ sudo snapper -c home get-config

键                       │ 值
─────────────────────────┼──────
ALLOW_GROUPS             │ wheel # 允许执行此配置命令的用户组。
ALLOW_USERS              │       # 允许执行此配置命令的特定用户，默认root/wheel用户组。
BACKGROUND_COMPARISON    │ yes   # 是否在后台预计算快照之间的差异。
EMPTY_PRE_POST_CLEANUP   │ yes   # 是否自动清理内容完全一致的 Pre/Post 快照。
EMPTY_PRE_POST_MIN_AGE   │ 3600  # 空 Pre/Post 快照被清理前的最小存活时间。
FREE_LIMIT               │ 0.2   # 磁盘剩余空间临界值，0.2 表示当磁盘空间不足 20% 时，开始积极清理。
FSTYPE                   │ btrfs # 文件系统类型。
NUMBER_CLEANUP           │ yes   # 是否启用数量清理，针对非定时生成的（如 DNF 插件生成的）快照。
NUMBER_LIMIT             │ 50    # 普通快照保留的数量上限。
NUMBER_LIMIT_IMPORTANT   │ 10    # 重要快照保留的数量上限。
NUMBER_MIN_AGE           │ 3600  # 数量快照被清理前的最小存活时间，保证新生成的快照不会瞬间被清理掉。
QGROUP                   │       # 如果为空，Snapper 无法准确计算快照实际占用的物理空间
SPACE_LIMIT              │ 0.15  # 快照占用总空间上限，0.15 表示快照占用超过 15% 磁盘空间时触发清理。
SUBVOLUME                │ /home # 该配置管理的子卷路径。
SYNC_ACL                 │ yes   # 是否将 ACL（访问控制列表）同步到快照中，确保恢复后权限一致。
TIMELINE_CLEANUP         │ yes   # 是否启用“时间线”定期清理机制。
TIMELINE_CREATE          │ yes   # 是否每小时自动创建一条快照。
TIMELINE_LIMIT_DAILY     │ 1     # 每日快照保留个数。
TIMELINE_LIMIT_HOURLY    │ 5     # 每小时快照保留个数。
TIMELINE_LIMIT_MONTHLY   │ 1     # 每月快照保留个数。
TIMELINE_LIMIT_QUARTERLY │ 3     # 每季度快照保留个数。
TIMELINE_LIMIT_WEEKLY    │ 1     # 每周快照保留个数。
TIMELINE_LIMIT_YEARLY    │ 1     # 每年快照保留个数。
TIMELINE_MIN_AGE         │ 3600  # 自动生成的定时快照最小存活时间。
```

### 1. 查看当前配置

```bash
sudo snapper -c root get-config
```

### 2. 自定义 root配置

```bash
sudo snapper -c root set-config ALLOW_GROUPS="wheel"
sudo snapper -c root set-config NUMBER_LIMIT="20"
sudo snapper -c root set-config NUMBER_LIMIT_IMPORTANT="5"
sudo snapper -c root set-config SPACE_LIMIT="0.15"
sudo snapper -c root set-config SYNC_ACL="yes"
sudo snapper -c root set-config TIMELINE_LIMIT_DAILY="1"
sudo snapper -c root set-config TIMELINE_LIMIT_HOURLY="5"
sudo snapper -c root set-config TIMELINE_LIMIT_MONTHLY="1"
sudo snapper -c root set-config TIMELINE_LIMIT_QUARTERLY="3"
sudo snapper -c root set-config TIMELINE_LIMIT_WEEKLY="1"
sudo snapper -c root set-config TIMELINE_LIMIT_YEARLY="1"
```

### 3. 自定义 home 配置

```bash
sudo snapper -c home set-config ALLOW_GROUPS="wheel"
sudo snapper -c home set-config NUMBER_LIMIT="20"
sudo snapper -c home set-config NUMBER_LIMIT_IMPORTANT="5"
sudo snapper -c home set-config SPACE_LIMIT="0.15"
sudo snapper -c home set-config SYNC_ACL="yes"
sudo snapper -c home set-config TIMELINE_LIMIT_DAILY="1"
sudo snapper -c home set-config TIMELINE_LIMIT_HOURLY="5"
sudo snapper -c home set-config TIMELINE_LIMIT_MONTHLY="1"
sudo snapper -c home set-config TIMELINE_LIMIT_QUARTERLY="3"
sudo snapper -c home set-config TIMELINE_LIMIT_WEEKLY="1"
sudo snapper -c home set-config TIMELINE_LIMIT_YEARLY="1"
```

### 4. 单独配置 QGROUP 参数

1. QGROUP 是 Quota Group（配额组）的缩写。它是 Btrfs 文件系统中的一种机制，专门用来统计和限制子卷（Subvolumes）及其快照（Snapshots）所占用的磁盘空间。

    ```bash
    sudo btrfs quota enable <挂载点> （开启内核支持）
    
    sudo snapper -c <配置名> setup-quota （建立 Snapper 关联）
    
    sudo snapper -c <配置名> get-config | grep QGROUP （确认握手成功）
    ```

2. 例如 root 分区，配置文件名为 root

    ```bash
    sudo btrfs quota enable /
    
    sudo snapper -c root setup-quota
    
    sudo snapper -c root get-config | grep QGROUP
    ```

### 5. 查看最终配置

```bash
sudo snapper -c root get-config
    
sudo snapper -c home get-config
```

---

## 5. 启用自动服务

### 1. 启用定时服务

1. 启用并启动服务

    ```bash
    sudo systemctl enable --now snapper-timeline.timer
    sudo systemctl enable --now snapper-cleanup.timer
    ```

2. 检查服务状态

    ```bash
    sudo systemctl status snapper-timeline.timer
    sudo systemctl status snapper-cleanup.timer
    ```

3. 查看定时计划

    ```bash
    sudo systemctl list-timers --all | grep snapper
    ```

### 2. 立即运行一次快照创建

1. 手动触发时间线快照

    ```bash
    sudo systemctl start snapper-timeline.service
    ```

2. 查看日志

    ```bash
    sudo journalctl -u snapper-timeline.service -n 10
    ```

3. 使用dnf命令后自动创建快照

    - 安装dnf5-autosnapper

        ```bash
        sudo dnf copr enable douglascdev/dnf5-autosnapper && sudo dnf install dnf5-autosnapper
        ```

    - 使用dnf命令并检查快照是否正在生成。

参考文档：

- [dnf5-autosnapper](https://github.com/douglascdev/dnf5-autosnapper)

---

## 6. 快照操作指南

### 1.快速预览

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

# 7.创建副本为只读子卷
sudo btrfs subvolume snapshot -r <源子卷> <快照路径>

# 8.创建副本为可读写子卷
sudo btrfs subvolume snapshot <源子卷> <快照路径>
```

---

### 2.手动创建快照

``` bash
# 1.创建手动快照

# 2.创建 root 快照
sudo snapper -c root create --description "系统更新前"

# 3.创建 home 快照
sudo snapper -c home create --description "重要文件备份"
```

### 3.查看和管理快照

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

### 4.从快照恢复文件

```bash
# 查看快照内容
sudo ls /.snapshots/SNAPSHOT_NUMBER/snapshot/

# 恢复单个文件
sudo cp /.snapshots/SNAPSHOT_NUMBER/snapshot/path/to/file /path/to/restore

# 比较并恢复
sudo snapper -c root undochange PREVIOUS..CURRENT
```

## 7. 使用 btrfs-assistant 工具管理快照

### 7.1 管理快照

- 点击标签栏进入 snapper > new/delete

- select config 可以选择配置文件

- new 为新建

- delete 为删除

- refresh 为刷新

### 7.2 回滚

- 点击标签栏进入 snapper > browse/restore

- select config 可以选择配置文件

- 选择目标快照

- 点击 restore

### 7.3 修改部分 snapper 配置文件

- 点击标签栏进入 snapper settings

- select config 可以选择配置文件

- 选择对应的条目进行修改

## 8. 帮助

### 0. 注意事项

1. 遇到报错为：列出配置失败 (reading sysconfig-file failed)，可以尝试下面的操作。

    - 备份现有文件（如果有）

    ```bash
    sudo cp /etc/sysconfig/snapper /etc/sysconfig/snapper.backup 2>/dev/null
    ```

    - 创建正确的文件

    ```bash
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
    ```

    - 设置权限

    ````bash
    sudo chmod 644 /etc/sysconfig/snapper
    sudo chown root:root /etc/sysconfig/snapper
    ````

    - 修复 SELinux 上下文

    不执行这一步可能会遇到这种报错：

    Relabeled /etc/sysconfig/snapper from unconfined_u:object_r:etc_t:s0 to unconfined_u:object_r:snapperd_conf_t:s0

    ```bash
    sudo restorecon -v /etc/sysconfig/snapper
    ```

    - 验证文件

    ```bash
    cat /etc/sysconfig/snapper
    ```

    - 现在创建配置

    ```bash
    sudo snapper -c root create-config /
    ```

2. 遇到报错为： 创建配置失败 (creating btrfs subvolume .snapshots failed since it already exists)。

    - 删除相对应的子卷，比如根目录下

    ```bash
    # 列出所有快照
    sudo btrfs subvolume list /
    # 删除根目录下创造的子卷
    sudo btrfs subvolume delete /.snapshots
    # 删除主目录下创造的子卷
    sudo btrfs subvolume delete /home/.snapshots
    ```

    - 验证子卷是否删除
    sudo btrfs subvolume list /

3. 遇到报错为：创建配置失败 (config already exists)。

    说明已经创建了对应子卷的快照配置文件，如果要删除配置文件请执行下面命令，将root改为相对应的配置文件ID，请勿直接删除该/etc/snapper/configs/目录下的文件，因为在/etc/sysconfig/snapper中SNAPPER_CONFIGS参数记录着已经创建了的配置文件ID。

    ```bash
    sudo snapper -c root delete-config 2>/dev/null
    # ⚠️注意：配置文件删除后对应的快照也会被删除，这将自动执行下列操作。
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
