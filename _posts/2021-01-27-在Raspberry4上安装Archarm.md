---
layout:     post
title:      "树莓派4上安装 Archarm"
subtitle:   "Install Archarm on Raspberry Pi 4"
date:       2021-01-28 12:06:00
author:     "CellNet"
header-img: "img/post-bg-raspberry-pi-4.png"
catalog: true
tags:
    - Archarm
    - Raspberry Pi
---

**注意：** Raspberry Pi 4 有着比 Raspberry Pi 3 更高的电源要求。官方推荐使用 3A 的电源。使用不足的电源将导致随机、莫名其妙的错误和文件系统损坏。

Arch Linux 不再支持 ARM 体系结构（由 Raspberry Pi 等设备使用）。但是，还有一个名为 Arch Linux 的单独项目，该项目将 Arch Linux 移植到 ARM 设备上。它具有 32 位和 64 位格式。

请准备本教程所需的以下内容：
- 至少 8GB 的 microSD 卡。
- 一个带有读卡插槽（或者读卡器）的 Linux 系统，用于准备 mircoSD 卡以进行 Arch 安装。
- 互联网连接。
- 一个 Raspberry Pi 及其相关配件 :)

所有的安装过程完全基于终端，因此**你应当具备 Linux 命令行的相关知识**。


## 烧录 Arch Linux 镜像

### 找到你的块设备
将 microSD 卡插入 Linux 电脑中，打开终端，获取 **root/sudo** 权限来列出电脑上的块设备。**安装过程如果没有特殊说明均使用 root 权限**
```shell
fdisk -l
```
我的块设备是 sdb，**你的设备不一定和我一样**。以下请用你的设备 id 代替 **sdX**

### 格式化和创建分区
把 SD 卡分区使用 `fdisk` 命令，注意用你的 sd 卡名字替换 **sdX**
```shell
fdisk /dev/sdX
```
在`fdisk`提示符下，必须删除现有分区，然后创建一个新分区。**再次提醒不要弄错设备名啦！**

1. 键入 `o`。这步将会清除驱动器上的所有分区。
2. 键入 `p` 以列出分区,检查是否仍然存在任何分区。
3. **创建 boot 分区**： 键入 `n`，然后键入 `p` 使用主驱动器，键入 `1` 使用驱动器上的第一个分区，按回车键接收默认的第一扇区，然后键入 `+200M` 作为最后一个扇区。
4. 键入 `t`，然后键入 `c` 将第一个分区设置为 W95 FAT32 (LBA)。
5. **创建 root 分区**：键入 `n`，然后键入 `p` 使用主驱动器，键入 `2` 使用驱动器上的第二部分，然后按**两次**回车键接受第一扇区和最后一个扇区的默认设置。
6. 键入 `w` 退出并写入分区表。

### 创建并挂载 FAT & ext4 文件系统
在这部分，我将使用`mkfs`命令为 boot 和 root 分区创建文件系统然后挂载他们。注意用你的设备名替换**sdX**，如果你不记得设备 id，可以使用 `fdisk -l` 再看一下。

```shell
mkfs.vfat /dev/sdX1
mkdir boot
mount /dev/sdX1 boot
mkfs.ext4 /dev/sdX2
mkdir root
mount /dev/sdX2 root
```

### 下载和解压 Arch Linux for Raspberry Pi 4
确保你有 root 权限（否则某些操作将会失败），执行以下命令（使用 sudo，如果你不是 root）。
```shell
wget http://os.archlinuxarm.org/os/ArchLinuxARM-rpi-aarch64-latest.tar.gz
bsdtar -xpf ArchLinuxARM-rpi-aarch64-latest.tar.gz -C root
sync
```

现在移动 boot 文件到刚刚创建的 boot 分区中去。

```shell
mv root/boot/* boot
sed -i 's/mmcblk0/mmcblk1/g' root/etc/fstab
```

### 关闭 HDMI 检测
在默认情况下 Raspberry Pi 4 会检测 HDMI 接口的状态，如果没有连接显示器那么 Raspberry Pi 4 将不会开机直到连接上显示器。但是 Raspberry Pi 4 一般是作为服务器使用的，不会接显示器。所以我们要把 HDMI 检测关闭。
```shell
nano boot/config.txt
```
在文件末尾添加如下语句
```vim
hdmi_force_hotplug=1
```
键入 `ctrl+o` 再键入 `ENTER` 保存，`ctrl+x`离开。

### 收尾
现在 Arch Linux 镜像已经烧录 sd 卡上了，弹出 sd 卡，再把 sd 卡插入到 Raspberry Pi 4 上，加电！enjoy it！
```shell
unmout boot root
```

## Arch Linux ARM 常规设置

**注意：**Raspberry Pi 4 装了 Arch Linux ARM 后无法使用键盘。暂时不知道怎么解决，解决后会更新本教程。

你可以按照本教程的其余部分进行操作，或者直接在 Raspberry Pi 上通过连接显示器和键盘设置，也可以通过 SSH 远程连接到 Raspberry Pi （如果没有备用显示器，则需要通过以太网连接到您的本地网络）。

### 连接 WiFi
如果无法使用以太网连接，则可以使用以下命令以 **root** 用户执行以下命令。
**注意：**你需要一个键盘和显示器才能初始化连接 WiFi。
```shell
wifi-menu
```

### SSH 连接 Raspberry Pi
一旦你找到了 Raspberry Pi 的 IP 地址，你可以使用以下命令在你的电脑上操作 Raspberry Pi。
```shell
ssh alarm@raspberry_pi_ip_address
```
**注意：**默认用户名是 alarm ，密码是 alarm。默认的 root 密码是 root。

### pacman 初始化
替换 pacman 源适应国内网络环境
```shell
nano /etc/pacman.d/mirrorlist
```
在文件最顶端添加
```vim
Server = https://mirrors.ustc.edu.cn/archlinuxarm/$arch/$repo
```

为了完成安装过程，你需要初始化 pacman 密钥并填充 Arch Linux ARM 软件包签名密钥。
```shell
pacman-key --init
pacman-key --populate archlinuxarm
```

至此，安装过程已经完成，你可以使用与 x86 体系结构机器相同的 pacman 命令以 root 用户身份升级系统软件包。
```shell
pacman -Syu
```
如果要在系统升级后重新启动 Raspberry pi，只需在终端中键入 reboot 并通过 SSH 重新连接。

### 为你的用户授予 sudo 特权
为了能够给用户sudo特权，需要先安装sudo软件包。
```shell
pacman -S sudo
```
sudo的配置文件是`/etc/sudoers`。应该始终使用 `visudo` 命令对其进行编辑。
```shell
EDITOR=nano visudo
```
打开配置文件后，以与我相似的方式添加用户名，最好在root用户下。然后保存文件并退出。
```vim
alarm ALL=(ALL) ALL
```

### 安装 AUR 助手
对于大型 Arch 用户存储库，许多用户更喜欢 Arch Linux 或基于 Arch Linux 的发行版。你可以在 ARM 指令集计算机上使用 AUR 软件包，但并非所有软件包都与此体系结构兼容。
首先，请确保您已安装 git 软件包和 base-devel 组。
```shell
sudo pacman -S git base-devel
```
现在，你可以按照自己喜欢的方式从 AUR 中安装任何软件包，也可以通过 AUR Helper 安装 AUR 中的软件包。
```shell
git clone https://aur.archlinux.org/yay.git 
cd yay
makepkg -si
```


## References
- <http://itsfoss.com/install-arch-raspberry-pi>
- <https://archlinuxarm.org/platforms/armv8/broadcom/raspberry-pi-4#installation>
- <https://unix.stackexchange.com/questions/259193/what-is-a-block-device>
- <https://blog.csdn.net/qq_35475292/article/details/109034765>
- <https://mirrors.ustc.edu.cn/help/archlinuxarm.html>