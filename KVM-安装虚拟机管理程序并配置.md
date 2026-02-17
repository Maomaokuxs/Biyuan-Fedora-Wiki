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
sudo dnf -y install bridge-utils libvirt virt-install qemu-kvm

# 2.安装后，确认内核模块已加载
$ lsmod | grep kvm

# 3.还要安装有用的虚拟机管理工具：
sudo dnf install libvirt-devel virt-top libguestfs-tools guestfs-tools
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

## 5.创建一个测试实例

首先，创建一个连接到实例上的桥接网络，下面的指南会有帮助。
在Linux中创建和配置KVM桥接网络
一旦桥接接口准备好，使用CLI或虚拟机管理器创建一个测试实例。

### 5.0 停用占用网卡

```bash
# 停用占用网卡的有线连接
sudo nmcli connection down "有线连接 1"
# 有线连接 1 是在安装完系统之后默认创建的，不建议删除，直接停用就可以了。
```

### 5.1 创建一个桥接连接

```bash
sudo nmcli connection add type bridge con-name br0 ifname br0
```

### 5.2 设置桥接的 IP 方法为自动(DHCP)

```bash
sudo nmcli connection modify br0 ipv4.method auto
```

### 5.3 将物理网卡(例如 eno1 )作为从设备添加到桥接中

```bash
# 创建一个类型为 ethernet 的从连接，并将其主设备设置为 br0
sudo nmcli connection add type ethernet con-name bridge-slave-eno1 ifname eno1 master br0
```

### 5.4 激活桥接连接

```bash
sudo nmcli connection up bridge-slave-eno1
sudo nmcli connection up br0
```

### 5.5 (可选) 手动重置桥接接口

```bash
sudo ip Link set br0 down
sudo ip link set bro up
```

- 参考文档：
[如何在Fedora 43/42/41/40上安装KVM](https://computingforgeeks.com/how-to-install-kvm-on-fedora/)
