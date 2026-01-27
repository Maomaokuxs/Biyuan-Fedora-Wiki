# 说明

不建议使用grub2-install命令,Fedora 使用 Shim 和 GRUB 的组合来支持 UEFI Secure Boot，直接使用 grub2-install 会绕过 Secure Boot 安全机制。

不需要删除原有 /boot/efi/EFI/fedora/ 目录下的文件，直接重新安装相关的软件包即可，EFI 分区里有一个 grub.cfg，它并不是真正的配置文件，而是一个重定向脚本，配置文件都统一存放在 /boot/grub2/grub.cfg，使用`grub2-mkconfig`命令就可以重新生成配置文件，如果删除 /boot/efi/EFI/fedora/ 目录下的文件，我并不确定能否通过重装`grub2-efi-x64`软件包来恢复重定向脚本。

## 1\. 挂载分区（根据你的实际情况调整设备名）

- 一般情况

```plaintext
mount /dev/sda2 /mnt  # 根分区
mount /dev/sda1 /mnt/boot/efi  # EFI 分区
```

- 对于使用了 btrfs 文件系统并创建了子卷

```plaintext
mount -o subvol=/@ /dev/sda2/ /mnt  # 根分区
mount -o subvol=/@home /dev/sda2 /mnt/home
mount /dev/sda1 /mnt/boot/efi  # EFI 分区
```

- 挂载系统 API 虚拟文件系统：

```plaintext
for i in /dev /dev/pts /proc /sys /run /sys/firmware/efi/efivars; do sudo mount -B $i /mnt$i; done
```

## 2\. 然后 chroot

```plaintext
 sudo chroot /mnt
```

## 3\. 重新安装所有引导组件

```plaintext
dnf reinstall shim-x64 grub2-efi-x64 grub2-common
```

这会确保 `/boot/efi/EFI/fedora/shimx64.efi` 和 `grubx64.efi` 都是最新的且带有正确签名。

## 4\. 重新生成 initramfs

```plaintext
dracut --force
```

## 5\. 重新生成 GRUB 配置

```plaintext
grub2-mkconfig -o /boot/grub2/grub.cfg
```

注意：即使是 UEFI 系统，在 Fedora 中推荐做法也是更新 `/boot/grub2/grub.cfg`，因为它会自动链接到 EFI 分区，没必要使用`grub2-install` 命令。

## 6\. 验证文件

```plaintext
# 1. 检查 EFI 文件
ls -la /boot/efi/EFI/fedora/
# 应该看到：shimx64.efi, grubx64.efi, mmx64.efi

# 2. 检查 GRUB 配置
ls -la /boot/grub2/grub.cfg

# 3. 检查内核文件
ls -la /boot/vmlinuz-* /boot/initramfs-*

# 4. 检查 UEFI 启动项（在 Live 环境中）
efibootmgr -v
```
