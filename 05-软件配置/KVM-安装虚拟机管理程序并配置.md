# 说明

这篇文档旨在安装KVM虚拟机管理程序并进行一些基本上的配置。

## 1.检查虚拟化扩展

第一步是确认你的CPU是否支持 Intel VT 或 AMD-V 虚拟化扩展。在某些系统中，BIOS中关闭了这个功能，可能需要启用。

```bash
cat /proc/cpuinfo | egrep "vmx|svm"
```

## 2.在 Fedora 上安装 KVM / QEMU

```bash
# 1.安装应用程序
# Fedora 40-43:
# sudo dnf -y install bridge-utils libvirt virt-install qemu-kvm

# Fedora 44+ (bridge-utils 已弃用, 跳过):
sudo dnf -y install libvirt virt-install qemu-kvm

# 2.安装后，确认内核模块已加载
$ lsmod | grep kvm

# 3.安装有用的虚拟机管理工具：
sudo dnf install libvirt-devel virt-top libguestfs-tools guestfs-tools

# 4.将当前用户加入 libvirt 组 (免 sudo 管理虚拟机)
sudo usermod -aG libvirt $USER
# ⚠️ 注意：需要退出重新登录后组权限才生效
```

## 3.启动并启用 KVM 守护进程

```bash
# 1.默认情况下，KVM 守护进程未启动，请使用以下命令启动服务：
sudo systemctl start libvirtd

# 2.然后启用启动时开始服务
sudo systemctl enable libvirtd
```

## 4.安装虚拟机管理器图形界面

```bash
sudo dnf -y install virt-manager
# 提供了一个图形界面来管理虚拟机
```

## 5.配置默认 NAT 网络

libvirtd 安装后会自动定义默认的 NAT 网络 `default`，但需要手动启动：

```bash
# 1.启动默认 NAT 网络
sudo virsh net-start default

# 2.设为开机自启
sudo virsh net-autostart default

# 3.验证 (确认 default 显示为"活跃")
virsh net-list --all
```

默认 NAT 网络 (`virbr0`, 192.168.122.1/24) 开箱即用，**WiFi 和有线都适用**。

## 6.创建测试实例 (NAT 网络)

对于大部分桌面用户（尤其是 WiFi 环境），使用 NAT 网络是最简单的方式。

- **安装 virt-viewer**

  ```bash
  sudo dnf install -y virt-viewer
  ```

- **准备安装镜像**

  将 ISO 文件复制到 QEMU 可访问的目录（`/var/lib/libvirt/images/`），避免权限问题：

  ```bash
  sudo cp ~/Downloads/Fedora-Everything-netinst-x86_64-44-1.7.iso /var/lib/libvirt/images/
  ```

- **创建虚拟机**

  ```bash
  virt-install \
    --connect qemu:///system \
    --name fedora-vm \
    --ram 4096 \
    --vcpus 4 \
    --disk size=30,bus=virtio \
    --cdrom /var/lib/libvirt/images/Fedora-Everything-netinst-x86_64-44-1.7.iso \
    --os-variant detect=on,name=fedora-unknown \
    --network network=default,model=virtio \
    --graphics spice,listen=none \
    --video virtio \
    --sound none \
    --boot uefi \
    --check disk_size=off
  ```

  参数说明：

  | 参数 | 说明 |
  | ------ | ------ |
  | `--connect qemu:///system` | 连接到系统 libvirtd，使用 system 会话（默认是 /session） |
  | `--disk size=30` | 30G qcow2 磁盘，稀疏分配，不立即占满 |
  | `--cdrom` | 指定安装 ISO，必须放在 QEMU 可读的路径下 |
  | `--network network=default` | 使用默认 NAT 网络 |
  | `--graphics spice,listen=none` | 启用 SPICE 图形，通过 virt-viewer 连接 |
  | `--boot uefi` | 使用 UEFI 引导（OVMF） |
  | `--check disk_size=off` | 跳过磁盘空间检查（稀疏分配时宿主机可能显示空间不足） |

- **连接显示**

  ```bash
  virt-viewer --connect qemu:///system fedora-vm
  ```

- **常见问题**

  | 问题 | 原因 | 修复 |
  | ------ | ------ | ------ |
  | `Permission denied` 读取 ISO | ISO 在用户目录下，QEMU 无权访问 | 复制到 /var/lib/libvirt/images/ |
  | `network 'default' is not active` | 使用了 /session 而非 /system | 加 `--connect qemu:///system` |
  | 连接不到图形显示 | listen=none 导致无监听端口 | 改为 listen=127.0.0.1 或安装 virt-viewer |
  | 磁盘空间警告 | 池中剩余空间不足 | 加 `--check disk_size=off`（稀疏分配不会立即占用） |

## 7.创建测试实例 (桥接网络 - 仅有线)

> ⚠️ **WiFi 用户注意**：WiFi 接口通常不支持桥接，桥接后虚拟机无法上网。
> 如果你用的是 WiFi，直接使用第 5 步的默认 NAT 网络即可，跳过本节。

以下步骤适用于有线网卡：

- **停用占用网卡**
  
  ```bash
  sudo nmcli connection down "有线连接 1"
  # 有线连接 1 是在安装完系统之后默认创建的，不建议删除，直接停用即可
  ```

- **创建桥接连接**
  
  ```bash
  sudo nmcli connection add type bridge con-name br0 ifname br0
  ```

- **设置桥接的 IP 方法为自动(DHCP)**
  
  ```bash
  sudo nmcli connection modify br0 ipv4.method auto
  ```

- **将物理网卡作为从设备添加到桥接中**
  
  ```bash
  sudo nmcli connection add type ethernet con-name bridge-slave-eno1 ifname eno1 master br0
  ```

- **激活桥接连接**
  
  ```bash
  sudo nmcli connection up bridge-slave-eno1
  sudo nmcli connection up br0
  ```

- **(可选) 手动重置桥接接口**
  
  ```bash
  sudo ip Link set br0 down
  sudo ip link set bro up
  ```

- 参考文档：
[如何在Fedora 43/42/41/40上安装KVM](https://computingforgeeks.com/how-to-install-kvm-on-fedora/)
