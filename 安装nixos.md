# 说明

安装 fedora 的篇外故事。

## 联接网络

- 有线网默认直接链接

- 无线网使用 nmcli 工具打开 TUI 界面链接

## 启用 root 账户

```bash
sudo -i 
```

如果需要使用另一台电脑连接安装参考以下方式。

- 配置 root 密码

```bash
passwd
```

- 获取安装机器的网络ip地址

```bash
ip addr
```

- 通过ssh连接机器

```bash
ssh root@<ip地址>
```

## 创建分区

```bash
cfdisk /dev/nvme1n1
```

## 配置分区

我使用的是 btrfs 文件系统。

```bash
# nix-shell -p btrfs-progs
# mkfs.fat -F 32 /dev/nvme1n1p7
# mkfs.btrfs /dev/nvme1n1p8
# mkdir -p /mnt
# mount /dev/nvme1n1p8 /mnt
# btrfs subvolume create /mnt/root
# btrfs subvolume create /mnt/home
# btrfs subvolume create /mnt/nix
# umount /mnt
```

## 挂载分区和子卷

```bash
# mount -o compress=zstd,subvol=root /dev/nvme1n1p8 /mnt
# mkdir /mnt/{home,nix}
# mount -o compress=zstd,subvol=home /dev/nvme1n1p8 /mnt/home
# mount -o compress=zstd,noatime,subvol=nix /dev/nvme1n1p8 /mnt/nix

# mkdir /mnt/boot
# mount /dev/nvme1n1p7 /mnt/boot
```

## 生成 NixOS 配置文件

```bash
# nixos-generate-config --root /mnt
# vim /mnt/etc/nixos/configuration.nix # manually add mount options (see Compression below for an example)
```

## NixOS 系统配置

NixOS 使用声明式配置系统，允许用户管理整个操作系统设置，包括已安装的软件包、系统服务、用户帐户、硬件设置和更详细的配置文件。此页面概述了如何使用和管理 NixOS 系统配置。

有关声明式配置的介绍，请参阅 NixOS Linux 发行版概述#声明式配置 和 NixOS 官方手册。

安装 NixOS 时，默认系统配置模板由 nixos-generate-config 工具生成。 这会创建一个基本的 configuration.nix 文件以及相应的 hardware-configuration.nix 文件，后者捕获检测到的硬件设置和文件系统定义。 在更改 configuration.nix 后，可以使用 nixos-rebuild 应用它们

- 启动引导器

    默认为systemd-boot，如果要改成 GRUB 按下面的配置。

    ```bash
      # 1. 禁用原有的 systemd-boot
      boot.loader.systemd-boot.enable = false;
    
      # 2. 启用并配置 GRUB (适配 UEFI 启动模式)
      boot.loader.grub = {
        enable = true;
        efiSupport = true;
        
        # 在 UEFI 启动模式下，device 必须严格设置为 "nodev"
        device = "nodev"; 
        
        # 开启系统探测。这会自动扫描并探测出你的 Windows 引导，并将其完美添加到 GRUB 的启动菜单中，省去手动同步时间的麻烦。
        useOSProber = true; 
      };
    
      # 3. 允许系统修改 EFI 变量（如果这行原来就有，保留即可）
      boot.loader.efi.canTouchEfiVariables = true;
    ```

- 内核

    默认为最新的内核，不改变即可。

    ```text
    # Use latest kernel.
    boot.kernelPackages = pkgs.linuxPackages_latest;
    ```

- 主机名

    自定义要取消注释并修改。

    ```text
    # networking.hostName = "nixos"; # Define your hostname.
    ```

- 设置时区

    改成亚洲上海，需要取消注释。

    ```text
    # Set your time zone.                                 
    # time.timeZone = "Europe/Amsterdam"; 
    ```

- 代理

    按需配置，需要取消注释。

    ```text
    # Configure network proxy if necessary                 
    # networking.proxy.default = "http://user:password@proxy:port/";
    # networking.proxy.noProxy = "127.0.0.1,localhost,internal.domain";
    ```

- 设置本地化及终端字体

    需要取消注释。

    ```text
      # Select internationalisation properties.              
      # i18n.defaultLocale = "en_US.UTF-8";
      # console = {                                          
      #   font = "Lat2-Terminus16";                          
      #   keyMap = "us";                                     
      #   useXkbConfig = true; # use xkb.options in tty.     
      # };
    ```

- 是否启用 X11

    需要就取消注释。

    ```text
      # Configure keymap in X11
      # services.xserver.xkb.layout = "us";
      # services.xserver.xkb.options = "eurosign:e,caps:escape";
    ```

- 设置 进入 X11 后的键盘布局及键盘按键重映射

    ```text
      # Configure keymap in X11
      # services.xserver.xkb.layout = "us";
      # services.xserver.xkb.options = "eurosign:e,caps:escape";
    ```

- 管理和支持实体打印机

    ```text
      # Enable CUPS to print documents.                      
      # services.printing.enable = true;
    ```

- 配置音频服务

    ```text
      # Enable sound.                                        
      # services.pulseaudio.enable = true;                   
      # OR                                                   
      # services.pipewire = {                                
      #   enable = true;                                     
      #   pulse.enable = true;                               
      # };
    ```

- 是否启用触摸板

    ```text
      # Enable touchpad support (enabled default in most desktopManager).
      # services.libinput.enable = true; 
    ```

- 设置用户名及用户软件包

    ```text
      # Define a user account. Don't forget to set a password with ‘passwd’. 
      # users.users.alice = {                                
      #   isNormalUser = true;                               
      #   extraGroups = [ "wheel" ]; # Enable ‘sudo’ for the user. 
      #   packages = with pkgs; [                            
      #     tree                                             
      #   ];                                                 
      # };
    ```

- 是否需要火狐浏览器

    ```bash
      # programs.firefox.enable = true;
    ```

- 配置软件包

    ```text
    
      # List packages installed in system profile. 
      # You can use https://search.nixos.org/ to find more packages (and options). 
       environment.systemPackages = with pkgs; [ 
         nvim # Do not forget to add an editor to edit configuration.nix! The Nano editor is also installed by default. 
         wget 
       ];
    ```

- SUID wrappers

    配置那些在 Linux 系统中有着特殊权限要求（SUID），或者需要在你的桌面后台常驻运行（守护进程/Agent）的程序

    ```text
      # programs.mtr.enable = true;                          
      # programs.gnupg.agent = {                             
      #   enable = true;
      #   enableSSHSupport = true;                           
      # }; 
    ```

- SSH

    ```text
    
      # List services that you want to enable:               
                                                             
      # Enable the OpenSSH daemon.                           
      # services.openssh.enable = true; 
    
    ```

- 防火墙

    ```text
      # Open ports in the firewall.
      # networking.firewall.allowedTCPPorts = [ ... ];
      # networking.firewall.allowedUDPPorts = [ ... ];       
      # Or disable the firewall altogether. 
      # networking.firewall.enable = false;  
    ```

- 备份配置文件

    ```text
      # Copy the NixOS configuration file and link it from the resulting system
      # (/run/current-system/configuration.nix). This is useful in case you
      # accidentally delete configuration.nix.               
      # system.copySystemConfiguration = true; 
    ```

- 配置显卡驱动

    ```txet
    # 允许安装非自由软件
    nixpkgs.config.allowUnfree = true;
    
    # 加载驱动
    services.xserver.videoDrivers = [ "nvidia" ];
    
    hardware.nvidia = {
      modesetting.enable = true;
      # 4060 属于新架构，建议开启开源内核模块
      open = true;
      # 选择稳定版驱动
      package = config.boot.kernelPackages.nvidiaPackages.stable;
    };
    
    # 启用硬件加速
    hardware.graphics.enable = true;
    ```

## nixos-rebuild switch

要查找 NixOS 模块选项，请[参阅](https://search.nixos.org/options) 。

## NixOS 安装

```bash
# cd /mnt
# nixos-install
```
