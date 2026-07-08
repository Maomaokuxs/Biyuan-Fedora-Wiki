# 说明

在 UEFI 系统中，引导加载器存储在 EFI 系统分区（ESP，通常是 `/boot/efi`）上。如果主引导项损坏或被覆盖，系统将无法启动。通过创建一个备份 UEFI 启动项，可以在主引导项损坏时从备份项引导进入系统。

## 1. 查看当前 UEFI 启动项

```bash
efibootmgr -v
```

输出示例：

```text
BootCurrent: 0000
BootOrder: 0000,0003,2001,2002,2003
Boot0000* Fedora          HD(7,GPT,...)/\EFI\fedora\shim.efi
Boot0003* Windows Boot Manager   HD(1,GPT,...)/\EFI\Microsoft\Boot\bootmgfw.efi
```

- `BootCurrent`：当前从哪个条目启动
- `BootOrder`：启动顺序
- `Boot0000* Fedora`：Fedora 启动项，加载 `\EFI\fedora\shim.efi`

## 2. 查看 EFI 分区内容

```bash
sudo ls /boot/efi/EFI/
```

输出示例：

```text
fedora  Microsoft
```

`fedora` 目录下包含引导文件：

```bash
sudo ls /boot/efi/EFI/fedora/
```

```text
BOOTIA32.CSV  BOOTX64.CSV  gcdia32.efi  gcdx64.efi  grub.cfg
grubia32.efi  grubx64.efi  mmia32.efi  mmx64.efi
shim.efi      shimia32.efi  shimx64.efi
```

## 3. 创建备份引导项

```bash
# 1. 复制引导文件到独立目录
sudo cp -a /boot/efi/EFI/fedora /boot/efi/EFI/fedora-backup

# 2. 创建新的 UEFI 启动项，指向备份目录
sudo efibootmgr -c -d /dev/nvme0n1 -p 7 \
  -L "Fedora (Backup)" \
  -l '\EFI\fedora-backup\shim.efi'
```

参数说明：

| 参数 | 说明 |
|------|------|
| `-c` | 创建新启动项 |
| `-d /dev/nvme0n1` | 磁盘设备 |
| `-p 7` | EFI 分区号 |
| `-L "Fedora (Backup)"` | 启动菜单中显示的名称 |
| `-l '\EFI\fedora-backup\shim.efi'` | 引导文件路径（相对于 ESP 分区） |

## 4. 验证

```bash
efibootmgr -v | grep Fedora
```

输出示例：

```text
Boot0000* Fedora           HD(7,...)/\EFI\fedora\shim.efi
Boot0004* Fedora (Backup)  HD(7,...)/\EFI\fedora-backup\shim.efi
```

两个条目指向同一磁盘同一 EFI 分区，但引导文件路径不同——备份使用的是 `fedora-backup` 目录下的独立副本。

## 5. 调整启动顺序

```bash
# 设置为主 Fedora 优先，备份第二
sudo efibootmgr -o 0000,0004,0003,2001,2002,2003
```

## 6. 维护说明

UEFI 备份的是**引导加载器**（shim + grub），不复制内核和 initramfs。内核心和 initramfs 在 `/boot` 子卷上，两个启动项共享。

- **日常更新**：不需要同步，备份项自动共享最新内核
- **shim/grub 包大版本升级后**：需要手动同步

```bash
  sudo cp -a /boot/efi/EFI/fedora /boot/efi/EFI/fedora-backup
  ```

## 7. 常见问题

- **备份项不可用**

  可能原因：ESP 分区本身损坏。备份项在同一 ESP 上，无法抵御分区级别的故障。真正的冗余需要多个磁盘。

- **启动菜单中出现两个同名项**

  `efibootmgr -c` 时 `-L` 参数可以随意命名，同名也不会报错：

  ```bash
  sudo efibootmgr -c -d /dev/nvme0n1 -p 7 \
    -L "Fedora" \
    -l '\EFI\fedora-backup\shim.efi'
  ```

- **备份项不需要更新**

  引导加载器很少更新。shim 和 grub 包的大版本更新间隔很长，手动同步一次即可。
