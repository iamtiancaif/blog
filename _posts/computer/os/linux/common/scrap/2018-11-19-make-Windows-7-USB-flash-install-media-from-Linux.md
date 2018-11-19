---
layout: post
title:  "make Windows 7 USB flash install media from Linux"
categories: computer
tags:  computer linux
author: web
source: "https://serverfault.com/questions/6714/how-to-make-windows-7-usb-flash-install-media-from-linux"
---

* content
{:toc}


OK, after unsuccessfully trying all methods mentioned here, I finally got it working. Basically, the missing step was to write a proper boot sector to the USB stick, which can be done from Linux with `ms-sys` or `lilo -M`. This works with the Windows 7 retail version.

Here is the complete rundown again:

Install ms-sys - if it is not in your repositories, get it [here](http://ms-sys.sourceforge.net/). Or alternatively, make sure lilo is installed (but **do not** run the liloconfig step on your local box if e.g. Grub is installed there!)

Check what device your USB media is assigned - here we will assume it is `/dev/sdb`. Delete all partitions, create a new one taking up all the space, set type to NTFS (7), and remember to set it bootable:

`# cfdisk /dev/sdb`   _or_   `fdisk /dev/sdb` (partition **type 7**, and **bootable** flag)

Create an NTFS filesystem:

`# mkfs.ntfs -f /dev/sdb1`

Write Windows 7 [MBR](http://en.wikipedia.org/wiki/Master_boot_record) on the USB stick (also works for windows 8), multiple options here:

1.  `# ms-sys -7 /dev/sdb`
2.  or (e.g. on newer Ubuntu installs) `sudo lilo -M /dev/sdb mbr` ([info](http://ubuntuforums.org/showthread.php?t=622828))
3.  or (if syslinux is installed), you can run `sudo dd if=/usr/lib/syslinux/mbr/mbr.bin of=/dev/sdb`

Mount ISO and USB media:

\# mount -o loop win7.iso /mnt/iso
# mount /dev/sdb1 /mnt/usb

Copy over all files:

`# cp -r /mnt/iso/* /mnt/usb/`   _...or use the standard GUI file-browser of your system_

Call `sync` to make sure all files are written.

Open gparted, select the USB drive, right-click on the file system, then click on "Manage Flags". Check the boot checkbox, then close.

...and you're done.

After all that, you probably want to back up your USB media for further installations and get rid of the ISO file... Just use dd: `# dd if=/dev/sdb of=win7.img`

Note, this copies the whole device! — which is usually (much) bigger than the files copied to it. So instead I propose

    # dd count=[(size of the ISO file in MB plus some extra MB for boot block) divided by default dd blocksize] if=/dev/sdb of=win7.img
    

Thus for example with 8 M extra bytes:

    # dd count=$(((`stat -c '%s' win7.iso` + 8*1024*1024) / 512)) if=/dev/sdb of=win7.img status=progress
    

As always, double check the device names very carefully when working with `dd`.

The method creating a bootable USB presented above works also with Win10 installer iso. I tried it running Ubuntu 16.04 copying Win10\_1703\_SingleLang\_English\_x64.iso (size 4,241,291,264 bytes) onto an 8 GB USB-stick — in non-UEFI \[non-secure\] boot only. After execution dd reports: 8300156+0 records in 8300156+0 records out 4249679872 bytes (4.2 GB, 4.0 GiB) copied, 412.807 s, 10.3 MB/s

Reverse if/of next time you want to put the Windows 7 installer onto USB.


Install ms-sys on Debian
-------------------------

http://notepad2.blogspot.com/2014/09/install-ms-sys-on-debian.html

1.  Download ms-sys from [http://ms-sys.sourceforge.net/#Download](http://prdownloads.sourceforge.net/ms-sys/ms-sys-2.4.0.tar.gz?download).
2.  Install required packages:
    
    sudo apt-get install build-essential gettext
    
3.  Extract the source, and compile, then install:
    
    cd /tmp; tar zxvf ~/Downloads/ms-sys-2.4.0.tar.gz; 
    cd /tmp/ms-sys-2.4.0
    make
    sudo make install
    

### NOTE:

*   You need to install **build-essential** package to install the compilers.
*   You need to install **gettext** package to fix the error `msgfmt: command not found`



linux系统创建windows启动盘
-------------------------

http://www.cnblogs.com/mingxiazhichan/p/6923304.html


平时工作中用到linux的操作命令较多，因此为了方便，就给电脑装了双系统，一般工作的时候，都选择进入linux系统。但是今天有件工作之外的事情需要解决下：创建一个windows启动盘。如果按照往常来说，我会启动windows，然后用xxx工作制作u盘启动工具，傻瓜式的创建启动盘。但是今天不想再重启系统进入windows做u盘的启动盘了。想在想在linux系统中做u盘启动盘。

    之前也接触到在linux中做u盘启动盘的一些博客或其他信息，全部是说用dd命令就可以搞定，之前也试过，dd命令刻录windows的iso文件有问题。系统无法识别刻录出来的启动盘。之前没有深究什么原因造成的，今天继续这个问题，上网查找解决方案。找了好长时间终于找到一篇可以正常做启动盘的文章。下面是文章的网址，有兴趣的大家可以自行查看：

    url:http://blog.csdn.net/mike8825/article/details/51138575?locationNum=9 感谢 “天外之客”分享。

    博客中也说了，linux的iso都自带mbr，因此dd刻录linux的iso时没有问题，会将mbr一同刻录进u盘中。但是windows的iso文件不自带mbr，所以在linux系统做windows的启动盘的时候，首先需要向u盘中写入mbr信息。之后在将windows的iso内容拷贝到u盘中。这样bios就能正确识别mbr，进而安装windows系统。






