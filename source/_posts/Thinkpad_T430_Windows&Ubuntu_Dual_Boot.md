---
title: Thinkpad T430 Windows&Ubuntu Dual Boot
id: 301
categories:
  - 技术记录
date: 2015-12-11 10:03:56
tags:
---

在 Thinkpad T430 带有原装操作系统（其实是原声的 EFI 分区）的情形下，UEFI 模式下安装 Ubuntu 并成功启动进入 Ubuntu 系统。主要难度是原装电脑启动时会首先加载一个 “LenovoBT.efi” 的文件，然后 Lenovo 直接会去找 Windows 系统相关的 efi 文件，然后忽略其他的系统。[T440s 预装win8.1 GPT+EFI ubuntu 双系统 图文过程](http://tieba.baidu.com/p/2805772637)中提到这样的启动处理：

1.  加载 /EFI/Boot/LenovoBT.EFI
2.  /EFI/Boot/bootx64.efi，加载windows的boot loader
3.  /boot/efi/EFI/Microsoft/Boot/bootmgfw.efi，启动windows

因此这个文档记录了如何把这个流程给截断，使得 Ubuntu 的启动引导文件可以被看到。

首先说明没有起作用的教程：

1.  [Windows 10 Pro + Ubuntu 14.04.3 LTS 雙系統安裝](https://blog.birkhoff.me/windows-10-and-ubuntu-14_04_3-lts-dual-boot/) 中提到最后在 Windows 的命令行中使用如下命令 `bcdedit /set {bootmgr} path \EFI\ubuntu\grubx64.efi` 并没有生效
2.  百度贴吧一个教程 - [T440s 预装win8.1 GPT+EFI ubuntu 双系统 图文过程](http://tieba.baidu.com/p/2805772637) 中提到使用 Ubuntu 的引导文件 “grubx64.efi” 替换 “EFI/Boot” 里面的 “bootx64.efi”，没有效果，可能是 “LenovoBT.efi” 没有走前文提到的处理流程。

* * *

有效的教程 [Windows 8 removes Grub as default boot manager](http://askubuntu.com/questions/235567/windows-8-removes-grub-as-default-boot-manager)：

I can make no promises, but try this from a Windows Command Prompt window launched with Administrator privileges:

`bcdedit /set {bootmgr} path \EFI\ubuntu\grubx64.efi`

Note that {bootmgr} should be typed exactly; that's not a variable. If that doesn't work, you could try this in Linux:

1.  Back up the entire contents of /boot/efi (your EFI System Partition, or ESP).
2.  Type `sudo mv /boot/efi/EFI/Microsoft/Boot/bootmgfw.efi /boot/efi/EFI/Microsoft`.
3.  Type `cp /boot/efi/EFI/ubuntu/grubx64.efi /boot/efi/EFI/Microsoft/Boot/bootmgfw.efi`.
4.  Create a new /etc/grub.d/40_custom file entry that refers to EFI/Microsoft/bootmgfw.efi. Model it after the existing entry in /boot/grub/grub.cfg that refers to EFI/Microsoft/Boot/bootmgfw.efi; just remove Boot from the boot path and give the entry a new name.
5.  Type `sudo update-grub` to install the new GRUB entry.

* * *

上面的流程进行完第三步就足够了。

关键技术点：

1.  关闭 Windows 的“安全启动”
2.  CSM Support 这个可以开启也可以关闭
3.  UEFI 模式下安装好 Ubuntu 后，再次使用 Live USB 进入 UBuntu 系统，将 Windows 的 EFI 分区挂载，进行文件替换操作
4.  虽然不推荐，但是可以在 `sudo update-grub` 后直接修改 /boot/grub/grub.cfg ，将其中的 Windows EFI 文件地址修正为真正的 EFI 文件所在位置（我的是 /boot/efi/EFI/Microsoft/bootmgfw.efi）。这样做不太好，因为这个文件是用 /etc/grub.d/ 下的文件自动生成出来的，正确的做法是去那个文件夹下面增加或者修改一个 Entry 。