---
author: coney
layout: post
title: 防止Ubuntu启动时自动进入GUI
date: 2014-02-06 21:56:15 +0800
comments: true
categories:
- Linux/Network
---

为了实验几个需要X的小程序, 很不情愿的在Ubuntu Server上安装了Ubuntu-Desktop, 但是Ubuntu每次引导后自动进入Gnome实在是烦.

一般情况下想默认进入CLI的话只需要修改下runlevel, 可是我安装的Ubuntu(Server 12.04 LTS)默认的runlevel是2, Gnome也跑在level 2上. 貌似不能像往常一样通过修改`inittab`来禁用X.

<!-- more -->
网上的得到的方法是修改内核启动参数, 修改`/etc/default/grub`(需要root):
``` text Update /etc/default/grub
    # 将GRUB_CMDLINE_LINUX_DEFAULT修改成text, 注意大小写
    GRUB_CMDLINE_LINUX_DEFAULT="text"
    # 更新grub.conf
    update-grub
```
修改完成后重启, 下次启动完成后遍不会直接进入GUI. 看了下`update-grub`对`grub.conf`的修改, 貌似是在引导参数后加了一项`text`:
``` text /boot/grub/grub.cfg
menuentry 'Ubuntu, with Linux 3.2.0-59-generic' --class ubuntu --class gnu-linux --class gnu --class os {
    recordfail
    gfxmode $linux_gfx_mode
    insmod gzio
    insmod part_msdos
    insmod ext2
    set root='(hd0,msdos1)'
    search --no-floppy --fs-uuid --set=root ed929838-1e80-43fb-ab11-8827e0c3b900
    # 注意最后的text
    linux	/boot/vmlinuz-3.2.0-59-generic root=UUID=ed929838-1e80-43fb-ab11-8827e0c3b900 ro text
    initrd	/boot/initrd.img-3.2.0-59-generic
}
```

如果想在CLI下重新进入Desktop, 可以执行

`service lightdm start`

或者

`startx`
