---
title: Archlinux Install Notes
id: 85
categories:
  - 技术记录
date: 2013-03-27 00:57:22
tags:
---

分区格式化
cfdisk
mkfs.ext /dev/sda*
mkswap /dev/sdaX
swapon /dev/sdaX

挂载当前分区
检查当前磁盘的标识符和布局
lsblk /dev/sda
mount /dev/sda1 /mnt
mkdir /mnt/home
如果是home独立分区
mount /dev/sda2 /mnt/home

编辑 /etc/pacman.d/mirrorlist
更新 pacman -Syy

使用pacstrap脚本安装基本系统

# pacstrap /mnt base base-devel

如果 pacman 报告安装时遇到错误的签名(error: failed to commit transaction (invalid or corrupted package))，请运行如下命令：

# pacman-key --init &amp;&amp; pacman-key --populate archlinux

base
来自 [core] 软件源的最小基本系统。
base-devel
来自 [core] 的附加工具例如make和 automake。大部分新手都应该安装它，后续扩展系统会用到它，安装AUR中软件包时，base-devel也是必须的

生成 fstab

用下面命令生成 fstab。如果想使用 UUIDs，使用 -U 选项；如果想使用标签，用 -L 选项，完成后最好检查一下生成的/etc/fstab。
Note: 后面如果出现问题，请不要再次运行genfstab，如果需要，手动编辑/etc/fstab。

# genfstab -U -p /mnt  | sed 's/rw,relatime,data=ordered/defaults,relatime/' &gt;&gt; /mnt/etc/fstab

# nano /mnt/etc/fstab

Chroot 到新系统

# arch-chroot /mnt

Locale
需要编辑两个文件：locale.gen 和 locale.conf.

默认情况下 /etc/locale.gen 是一个仅包含注释文档的空文件。选定你需要的本地化类型(移除前面的＃即可), 比如中文系统可以使用:

en_US.UTF-8 UTF-8
zh_CN.GB18030 GB18030
zh_CN.GBK GBK
zh_CN.UTF-8 UTF-8
zh_CN GB2312

然后运行：

# locale-gen

locale.conf 文件默认不存在，一般设置LANG就行了，它是其它设置的默认值。
/etc/locale.conf

LANG=en_US.UTF-8
LC_TIME=en_GB.UTF-8

Hostname

# echo myhostname &gt; /etc/hostname

网络查看接口     #ip link
动态 IP
If you only use a single fixed wired network connection, you do not need a network management service and
can simply enable the dhcpcd service, Where &lt;interface&gt; is your wired interface:

# systemctl enable dhcpcd@&lt;interface&gt;.service

Alternatively, you can use netcfg's net-auto-wired, which gracefully handles dynamic connections to new networks:

Install ifplugd, which is required for net-auto-wired:

# pacman -S ifplugd

Set up the dhcp profile and enable the net-auto-wired service:

# cd /etc/network.d

# ln -s examples/ethernet-dhcp .

# systemctl enable net-auto-wired.service

在 Arch x86_64 上运行 32 位应用程序，请在 /etc/pacman.conf 中加入如下内容以启用 [multilib] 源
[multilib]
SigLevel = PackageRequired
Include = /etc/pacman.d/mirrorlist

修改完成后需要用pacman 的 -Sy 参数更新服务器信息，否则会出现 "warning: database file for 'multilib' does not exist" 错误。

设置 Root 密码并创建一般用户
用 passwd 设置一个root密码：

# passwd

警告: Linux 是个多用户环境。请不要使用 root 登录进行日常工作。这不仅仅是坏习惯，还非常危险。Root 账户仅是用来做管理任务的。

因此，下面将创建用户archie，交互方式请使用adduser。

# useradd -m -g users -G wheel -s /bin/bash archie

# passwd archie

新非 root 用户创建完成，同时还建立了用户主目录和登录密码。

如果你弄错了账户设置，或者你想删除一个账户，或者你想要换个账户名，或者任何其他什么原因，使用 /usr/sbin/userdel ：

# userdel -r archie

Grub
Install the grub-bios package and then run grub-install:

# pacman -S grub-bios

# grub-install --recheck /dev/sda

# cp /usr/share/locale/en\@quot/LC_MESSAGES/grub.mo /boot/grub/locale/en.mo

虽然手动配置grub.cfg完全可以工作，建议用户自动生成这个文件。

要搜索硬盘上安装的其它操作系统，请先用 # pacman -S os-prober 安装 os-prober。

# grub-mkconfig -o /boot/grub/grub.cfg

卸载分区并重启系统
如果还在 chroot 环境，先用 exit 命令退出系统：

# exit

卸载/mnt中挂载的系统：

# umount /mnt/{boot,home,}

要退出到/mnt目录外面才能卸载
重启：

# reboot

图形用户界面
安装 X
X 窗口管理系统(X11或者X) 是基于网络的显示协议，提供了窗口功能，包含建立图形用户界面(GUI)的标准工具和协议。

安装基础的 Xorg 包：

# pacman -S xorg-server xorg-xinit xorg-utils xorg-server-utils

安装 mesa 以获得 3D 支持:

# pacman -S mesa

安装显卡驱动
注意: 如果是从 Virtualbox 虚拟机安装，则不需要安装显卡驱动，参见 此文，然后跳到下面的配置部分。

Linux 内核包含了开源的视频驱动，支持硬件加速的 framebuffers。OpenGL 和 X11 的 2D 加速需要用户空间工具。

如果不知道显卡类型，请执行如下命令进行查询：

$ lspci | grep VGA

输入下面命令，查看所有开源驱动:

$ pacman -Ss xf86-video | less

vesa是一个支持大部分显卡的通用驱动，不提供任何 2D 和 3D 加速功能。如果无法找到显卡芯片组的对应驱动或载入失败，Xorg 会使用vesa：要安装：

# pacman -S xf86-video-vesa

Intel     开源     xf86-video-intel     lib32-intel-dri
Nvidia
开源
xf86-video-nouveau     lib32-nouveau-dri
86-video-nv         –
闭源
nvidia             lib32-nvidia-utils
nvidia-304xx     lib32-nvidia-304xx-utils

笔记本(或触摸屏)用户需要 synaptics 软件包以支持触摸板/触摸屏：

# pacman -S xf86-input-synaptics

最小安装kde

如果你想最小安装KDE SC，安装 kdebase， phonon-vlc 或 phonon-gstreamer 以及，可选的语言包 kde-l10n-yourlanguagehere （对于简体中文语言数据，它是kde-l10n-zh_cn）。
注意: 各种后端需要一个 ttf-* 字体软件包。 phonon-vlc 已经依赖于 ttf-freefont，但你使用 phonon-gstreamer 时，你还应该安装 ttf-dejavu 或者别的字体。更多信息可以浏览 FS#26012。

使用 xinitrc
xinitrc的意义和用途在这里有详细描述。

安装 kdebase-workspace 编辑 ~/.xinitrc。然后取消注释：

exec startkde

启动不了X
copy file
I just looked in /etc/skel/.xinitrc and copied it to my home directory.