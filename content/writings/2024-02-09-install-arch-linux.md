+++
title = 'Arch Linux 安装与配置记录'
date = 2024-02-09T12:37:00+08:00
article_author = "Hao Wu [ Ertzizart ]"
preface = ""
preface_author = ""
license = "CC BY-NC-SA 4.0"
license_url = "https://creativecommons.org/licenses/by-nc-sa/4.0/"
novel = false
draft = false
+++

第一次年少无知误入Ubuntu至今八九年的时光里算是接触过不少Linux发行版。兜兜转转之下也只有Fedora、Arch Linux、Debian用的久一点，Fedora是2020年那会儿听说Linus Torvalds配了台搭载“线程撕裂者”的新电脑，使用的正是Fedora，不过后来看到红帽系（包括德国的openSUSE）均注明其遵守美国出台的《出口管理条例》（EAR），虽然实际没什么影响，但看着就是膈应，遂换成Arch Linux直至今日；Debian则是一直作为轻量云服务器的系统在使用。

## 硬件信息

整理出这篇文章是因为淘到一台Dell OptiPlex 3060MFF，搭配的是英特尔i5 8500T散片，内存是从退休的老笔记本上拆下来的2根8G内存条，固态硬盘也是老笔记本上拆下来的256G三星PM981A，可惜这个机型只能装一条固态，倒是有2.5寸盘位但用不着，还有一条致钛PC005只能晾在一边。这种机器多用于商务，后缀带T的低功耗芯片让整机待机功耗特别低，利用闲置的配件装一套拿来当软路由、服务器之类绰绰有余。有些品牌的迷你PC甚至留了一条PCIE4.0x8的插槽，双网口、独显、NAS玩法多样。

## 安装系统

Windows用户可以前往Arch Linux官网选择合适的分流服务器下载[最新系统镜像](https://archlinux.org/download/#http-downloads)。下载完成后可以使用[Rufus](https://rufus.ie/)刻录到U盘中。进BIOS设定为U盘启动即可进入Arch Linux临时系统，其中内置了很多工具可以在必要的时候用来急救，虽然目前Arch Linux滚挂的概率很低，但还是建议始终保留这块安装盘，最好每个月1号更新一次镜像。

### 连接网络

安装系统的过程中需要连接网络来下载安装一些软件包，这里只列出两种我常用的方法。

第一种方法是使用有线连接，比如网线、手机USB共享网络或者其他任何能用来联网的东西，插上网线或数据线后直接在终端里执行`dhcpcd`命令即可联网，一键操作十分简单。

第二种方法则是使用`iwd`工具通过无线网卡连接无线网，使用逻辑和平时用电脑或手机连WIFI别无二致，只是把图形操作变成了命令行操作，下面是`iwd`工具的简单使用。

首先敲`iwctl`命令并回车，进入`iwd`工具的环境：

```bash
iwctl
```

为了保险起见，先使用`device list`指令查看网卡设备的名字，列出的无线网卡的名字不出意外多半是`wlan0`，后续为了方便就以`wlan0`来作为样板来操作，这个名字并非绝对，也有可能是其他名字，按实际情况随机应变。

```bash
device list
```

得知网卡的名字后就可以像平时连WiFi一样，如下代码块中所示三条指令，对应三个步骤：

- 第一步扫描附近的WiFi
- 第二步拿到包含WIFI名称（SSID）的列表
- 第三步用WIFI名称（SSID）连接并回车输入密码再回车确认连接

当然前两个步骤不是必需操作，在知道系统里无线网卡的名字和无线网名称时完全可以直接执行第三步。这里建议WIFI名称和密码只用中英文及常见符号，临时系统里默认没有配置中文输入法和noto-fonts-cjk字库，中日韩之类的语言会显示方块乱码。

```bash
station wlan0 scan
station wlan0 get-networks
station wlan0 connect TP-LINK-1234
```

成功连接WIFI后使用`quit`指令即可退出`iwd`工具环境，然后通过`ping`工具随便找一个幸运网站测试一下网络连通性即可，类似`ping www.bing.com`。

确定网络连接成功后就需要启用网络时间同步服务并设置时区，避免后续下载安装出现问题。

```bash
timedatectl set-ntp true
timedatectl set-timezone Asia/Shanghai
```

### 硬盘分区

再次确保能够连上网络后就可开始进行硬盘分区，这里先使用`lsblk`命令列出所有硬盘及其分区，硬盘名字多半也是`/dev/sda`、`/dev/sdb`、`/dev/nvme0n1`之类。由于这台设备仅有一块NVME固态硬盘，所以这里使用`/dev/nvme0n1`来进行操作，具体如下所示，用`fdisk`工具选定`/dev/nvme0n1`硬盘来进行分区操作。

```bash
fdisk /dev/nvme0n1
```

回车后便会进入`fdisk`的环境，第一步输入`g`并回车将硬盘分区表格式化成GPT（GUID分区表）格式;第二步输入`n`并回车创建新分区，其中分区序号、分区起点不用管，回车两次到要求输入分区终点时停下，可以输入`+512M`划出512MB空间、`+1G`为划出1G空间、`+8G`划出8G空间、什么都不写直接回车则是划出剩余所有空间，分区完成后一定要输入`w`保存分区改动再退出，否则就得从头再来。下表是我的分区方案，仅供参考（若无需休眠功能则swap分区可以略过不去划分设定）：

|硬盘分区|挂载路径|空间|
|-----|-----|-----|
|/dev/nvme0n1p1|/boot|1GB|
|/dev/nvme0n1p2|/swap|8GB|
|/dev/nvme0n1p3|/|硬盘剩余所有空间|

swap分区的大小根据实际负载来定，如果物理内存32G，设定swap分区8G，但日常打开的软件共计占用内存超过8G时直接休眠就会出问题。

分区结束后即可对这三个分区分别格式化：

```bash
mkfs.fat -F32 /dev/nvme0n1p1
mkswap /dev/nvme0n1p2
mkfs.btrfs -f /dev/nvme0n1p3
```

因为主分区使用的是Btrfs文件系统，所以格式化完成后要将刚在硬盘里划出的`/dev/nvme0n1p3`分区挂载到临时系统中的`/mnt`目录作为新系统的根目录，创建并初始化几个Btrfs子卷，创建完成后再将该分区卸载：

```bash
mount /dev/nvme0n1p3 /mnt
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home
btrfs subvolume create /mnt/@var
chattr +C /mnt/@var
umount /mnt
```

卸载完成后再按顺序重新挂载`/dev/nvme0n1p3`分区到临时系统的`/mnt`目录并附带压缩功能，然后创建boot目录并将`/dev/nvme0n1p1`分区挂在于其上，再将刚才创建的Btrfs子卷挂载到各自的目录，最后为`/dev/nvme0n1p2`分区启动swap功能：

```bash
mount /dev/nvme0n1p3 /mnt -o subvol=@,compress=zstd
mkdir /mnt/boot
mkdir /mnt/home
mkdir /mnt/var
mount /dev/nvme0n1p1 /mnt/boot
mount /dev/nvme0n1p3 /mnt/home -o subvol=@home,compress=zstd,nosuid,nodev
mount /dev/nvme0n1p3 /mnt/var -o subvol=@var
swapon /dev/nvme0n1p2
```

### 安装基础系统和软件包

这里作为演示仅按前文提到的设备硬件来选择软件包，其余情况会略微补充：

```bash
pacstrap /mnt base base-devel linux linux-headers linux-firmware grub btrfs-progs intel-ucode efibootmgr bash zsh dhcpcd iwd nano vim openssh
```

- `base` 系统基础组件
- `base-devel` 包含一些编译组件
- `linux` 内核，也可选zen版、lts版等其他内核
- `linux-headers` 配套上面的内核
- `linux-firmware` 内核中未包含的驱动固件包
- `grub` 启动系统时需要用到
- `btrfs-progs` 处理Btrfs文件系统所需
- `intel-ucode` 英特尔微码
- `efibootmgr` UEFI引导所需
- `bash`和`zsh` 都是shell工具
- `dhcpcd`与`iwd` 皆为网络工具
- `nano` 文本编辑器，比vi/vim简单很多
- `vim` 文本编辑器，有些地方会用到
- `openssh` 远程连接需要

> [2025年6月22日更新] `linux-firmware`包已拆分，可以按需安装所需固件：
>
> - linux-firmware-amdgpu AMD
> - linux-firmware-atheros 高通
> - linux-firmware-broadcom 博通
> - linux-firmware-cirrus Cirrus Logic
> - linux-firmware-intel 英特尔
> - linux-firmware-mediatek 联发科
> - linux-firmware-nvidia 英伟达
> - linux-firmware-other 其他
> - linux-firmware-radeon AMD
> - linux-firmware-realtek 瑞昱

若想使用图形界面建议安装以下图形驱动：

- `mesa` OpenGL驱动
- `vulkan-intel` 英特尔核显的Vulkan驱动

若使用的是AMD处理器则可将`intel-ucode`更换为`amd-ucode`，Vulkan驱动需要由`vulkan-intel`更换为`vulkan-radeon`。

AMD无论核显还是独显在Linux平台都有得天独厚的条件，崩溃概率也比Windows平台低非常多，也不用刻意折腾。

全部安装完成后便可生成分区表`fstab`：

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

至此，基本系统安装完成。

## 配置系统

使用`arch-chroot`命令前往新系统进行更进一步配置：

```bash
arch-chroot /mnt
```

### 时区、区域与主机设置

进入新系统后与前面操作类似，首先设置时区，然后将硬件时间调整为当前的系统时间。

```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

hwclock --systohc
```

时间设置完成之后可以使用`timedatectl`命令查看当前时间，然后使用nano或者vim编辑`/etc/locale.gen`，往下翻找到并取消注释`en_US.UTF-8 UTF-8`或`zh_CN.UTF-8 UTF-8`，保存退出并使用`locale-gen`命令生成区域设置。：

```bash
nano /etc/locale.gen
locale-gen
```

设置语言：

```bash
echo 'LANG=en_US.UTF-8' > /etc/locale.conf
```
或
```bash
echo 'LANG=zh_CN.UTF-8' > /etc/locale.conf
```

想要显示中文还需要安装中文字体：
```bash
pacman -Sy noto-fonts-cjk
```

设置主机名，这里我用arch，可按喜好自定义：

```bash
echo 'arch' > /etc/hostname
```

编辑`hosts`文件：

```bash
nano /etc/hosts
```

进入`hosts`文件后输入以下内容：

```bash
127.0.0.1	localhost
::1	    	localhost
127.0.0.1	arch.localdomain	arch
```

若自定义了别的主机名请务必将上面的arch改为自定义的主机名。

### 文件系统、启动与休眠配置

由于使用了Btrfs文件系统，需要配置一些initramfs参数：

```bash
nano /etc/mkinitcpio.conf
```

进入`mkinitcpio.conf`文件后找到`MODULES`那一行，在括号里的最后添加一个```btrfs```，括号里可能会有其他东西，请**不要随意删减**。

```bash
MODULES = ( btrfs )
```

然后顺便在`HOOKS`那一行的括号里的最后添加一个`resume`，用于休眠，如果前面没有配置swap则可略过。

```bash
HOOKS = ( resume )
```

编辑完成后重新生成initramfs：

```bash
mkinitcpio -P
```

initramfs生成后即可生成引导程序，这里使用前面安装的`grub`进行配置：

```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=ArchLinux
grub-mkconfig -o /boot/grub/grub.cfg
```

不出意外即可顺利完成配置，至此也已经可以重启拔U盘进入新系统了，但还有一些小细节需要调整。如果前面没有配置swap，则可以直接跳到下一节进行收尾工作。

如果配置了swap，则需要在此使用nano编辑`/etc/default/grub`并找到`GRUB_CMDLINE_LINUX_DEFAULT`行并在引号内最后添加`resume=UUID=`字段：

```bash
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet resume=UUID=2af5c358-dde7-4fcd-895e-8193ca4cace8"
```

`resume=UUID=`后面的UUID就是`/dev/nvme0n1p2`即swap分区的UUID，可以通过`blkid`命令获得。

配置完成后记得重新生成grub配置文件：

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

后面进入新系统后即可通过`hibernate`命令启动休眠，将当前内存中的工作状态转移至swap分区，按下电源键便会恢复：

```bash
systemctl hibernate
```

### 安装完成之前的收尾

最后还需一些细微配置，首先将两个联网工具设置成开机自启，成功连接网络后重启即可自动联网：

```bash
systemctl enable iwd
systemctl enable dhcpcd
```

还有ssh功能也不要忘了设置开机自启，使用服务器90%的时间都得用ssh服务：

```bash
systemctl enable sshd
```

目前新系统中仅有一个root用户（就是超级管理员），所以需要添加一个普通用户，并将这个普通用户加入wheel组，为了能方便设置sudo命令，这里作为演示用name，实际可自定义其他用户名：

```bash
useradd -mG wheel -s /bin/zsh xiaowang
```

为刚创建的用户设置密码：

```bash
passwd xiaowang
```

然后使用`visudo`命令为用户设置root权限，进入`visudo`后找到并取消注释`%wheel ALL=(ALL) ALL`，这里的wheel就是刚刚创建用户附带加入的组名。

全部设置完成后千万不要忘了设置root用户的超级管理员密码：

```bash
passwd
```

到这里安装基本已经完成，可以退出、卸载、重启、拔U盘进入新系统了：

```bash
exit
umount -R /mnt
reboot now
```

启动之后输入用户名和密码即可正常使用乌漆麻黑的TTY界面了。

如果是作为服务器使用到此已经可以结束了。

如果前面已经选择了安装图形驱动，下方展示的是平铺窗口管理器Sway的安装命令和相关服务的设置：

```bash
pacman -S sway kitty swaybg wayland rofi-wayland wlroots seatd
systemctl enable seatd
```

当然，也不要忘记让普通用户xiaowang加入seatd组，否则是没有权限启动Sway的：

```bash
usermod -aG seatd xiaowang
```

设定完成重启并使用`sway`命令即可进入平铺桌面。

重启并正常登录到系统后即可输入`sway`命令进入桌面，原始状态比较简陋，需要稍微配置一下才能达到比较好用的状态。