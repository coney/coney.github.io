---
author: Coney
layout: post
title: 西安Office游戏机修复之路
date: 2014-04-07 16:33:24 +0800
comments: true
categories:
- Linux/Device Driver
---

加入Thoughtworks前参观西安Office时就已经注意到餐厅的游戏机, 但是入职后才知道游戏机已经坏了有些年月. 机缘巧合, 加入硬件小组后了解到有修复游戏机的计划, 所以便有了以下的经历.

#方案选择#
开搞之前, 新宇和高亮已经把游戏机肚里的东西摸得一清二楚. 游戏机主板寄给某个淘宝店家后已杳无音讯, 屏幕则是一块接受RGBS信号的25寸显像管, 摇杆和按钮是最直接的按键开关, 闭合后会跟GND连通. 有了这些基本输入输出信息, 下一步就是选择合适的软硬件来重建主板.

回想曾经的街机经历, 无非也就两种选择: 一是到游戏厅玩真机, 第二种便是通过PC上的模拟器. 重买一个街机主板放进去肯定不是我们想要的方案, 所以我们确定的基本思路就是通过PC主机+模拟器的方式.
比较常见的街机模拟器有winkawaks, nebula和MAME等, 相信不少人都有玩过:

{% img /images/2014/04/MAME.jpg '' 'MAME' %}
<!-- more -->
MAME for windows
但是这些流行的模拟器大多数都是基于Windows平台的, 而往往支持Windows平台的设备成本都比较高. 幸好新宇已经提前做了不少预研, 推荐我们使用一款叫做[AdvancedMAME](advancemame.sourceforge.net)的跨平台街机模拟器. 这样我们就能将模拟器部署在较低成本的Linux设备上.

硬件方面, 市面上大多数两三百元左右的嵌入式开发板都能够运行Linux并且带有VGA或HDMI等显示输出接口. 而手边正好有树莓派和CubieBoard两个板子, 相比而言树莓派更成熟稳定, 资料也相对丰富, 所以这次选择树莓派来扮演游戏机主板. 
树莓派本身带有十多个GPIO接口, 可以直接用来接入游戏机摇杆信号. 而图像处理要稍微麻烦一些, 因为树莓派提供了HDMI及同轴模拟视频接口, 不能直接兼容游戏机的RGBS信号, 需要额外的设备将HDMI输入转换为RGBS输出. 幸运的是这个问题可以通过万能的淘宝来解决, 我们购买开发板时同时购入了一套HDMI转VGA再转RGBS的装置. 
整套硬件的连接方式如下:
{% img /images/2014/04/hd-layout.png '' 'HD layout' %}

# 交叉编译模拟器 #
方案就绪, 下一步就是先将游戏在树莓派上跑起来. AdvanceMAME是基于C的跨平台软件, API兼容ABI不兼容, 所以如果想要在arm架构的树莓派上跑起来的话, 需要交叉编译模拟器.
获取交叉编译工具的方法很多, 可以通过包管理器安装, 或是从树莓派的github上进行[下载](https://github.com/raspberrypi/tools). 如果你使用mac或cygwin的话, 还可以通过crosstool-ng来生成自己的交叉编译工具链. 这里我们选择最快捷方式, 直接从树莓派的github上下载:
```git clone https://github.com/raspberrypi/tools```
下载完成后, 我们可以先编译一个简单的小程序, 验证下交叉编译环境是否有问题, 国际惯例, 先来个hello world:
``` c++
#include <stdio.h>

int main(int argc, char **argv)
{
	printf("hello world\n");
	return 0;
}
```
将代码保存至hello.c中, 之后调用gcc进行编译
```
coney@UServer: ~ $ ./raspi/tools/arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian/bin/arm-linux-gnueabihf-gcc  hello.c -o hello
coney@UServer: ~ $ file hello
hello: ELF 32-bit LSB executable, ARM, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.26, BuildID[sha1]=0x1a4b10f8034567384fbc51490dc09d6646262a58, not stripped
```
`raspi/tools`这个目录是刚刚clone下来的工具链目录. 编译完成后通过`file`命令可以看到生成的文件是32位的arm应用程序. 接下来将`hello`程序复制到树莓派上进行执行, 验证我们的程序是否真的能hello world:
```
coney@coney-pi: ~ $ ./hello
hello world
```
交叉编译工具准备好后, 下一步就是获取AdvanceMAME的源码进行编译, 代码可以在此处下载: http://advancemame.sourceforge.net/download.html
将下载的tar包解压, 进入解压后的目录, 首先要通过configure生成makefile. 因为这次是交叉编译, 所以在./configure前需要将交叉编译工具的路径加入PATH中:
`export PATH=~/raspi/tools/arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian/bin/:$PATH`
然后运行configure, 指定目标平台为arm-linux-gnueabihf, 另外可以修改prefix, 以方便编译完成后获取生成的程序.
`./configure --prefix=/home/coney/mame-arm/ --host=arm-linux-gnueabihf`
configure完成后, 运行make编译模拟器, 并将编译后的内容安装到刚才通过prefix指定的目录:
`make all install`
编译完成后, 将prefix指定目录中的内容全部复制到树莓派上, 接下来接上树莓派的键盘显示器, 在树莓派上体验下刚刚编译的模拟器.
```
root@coney-pi: /opt/mame-arm # ./bin/advmame
Creating a standard configuration file...
Configuration file `/root/.advance/advmame.rc' created with all the default options.
root@coney-pi: /opt/mame-arm # cp ~/snowbros.zip ~/.advance/rom
root@coney-pi: /opt/mame-arm # ./bin/advmame snowbros
AdvanceMAME - Copyright (C) 1999-2003 by Andrea Mazzoleni
MAME - Copyright (C) 1997-2003 by Nicola Salmoria and the MAME Team
No monitor clocks specification `device_video_p/h/vclock'.
video_init failed
```
`/opt/mame-arm`是编译后的模拟器目录, 第一次运行advmame时, 会在当前用户目录下创建`.advance`目录, 默认情况下, advanceMAME所有的配置, 游戏Rom, 截图等都放在这里. 之后我们将下载的游戏Rom拷入`.advance/rom`中(可能存在版权问题, 请自行查找rom), 再次启动`advmame`并传入游戏名称. 呵呵, 还是没有启动起来. 根据提示, 我们没有配置显示器的刷新率, 根据网上现有的经验, 如果使用树莓派HDMI接口输出的话, 可以编辑`.advance/advmame.rc`并加入这行配置:
`device_video_clock 5 - 50 / 15.62 / 50 ; 5 - 50 / 15.73 / 60`
再次运行`advmame snowbros`启动游戏, 不出意外的话你就能看到久违的游戏画面:
图片
p.s. advmame可以直接操纵framebuffer, 不需要启动X, 但是需要root权限.

# 编写摇杆驱动 #
到此游戏已经能够成功的在树莓派上运行, 通过视频转换接入游戏机后便能够通过游戏机的显像管输出. 但是现在还有一个另一个重要的问题没有解决, 那就是游戏操控只能通过键盘而不是游戏机上的摇杆.
关于如何接入游戏机的摇杆, 我们曾考虑过读入GPIO信号, 然后模拟键盘事件. 但是通过advMAME附带的几个测试程序, 很神奇的发现Linux居然有专门的摇杆设备标准, 真是工作娱乐两不误的好系统. 网上关于linux摇杆驱动的内容并不多, 但是幸运的是linux内核是开源的, 并且带有大量第三方摇杆驱动源码可以借鉴.
摇杆驱动在linux中属于Input Driver的一种. Input Driver分为Input Device Driver及Input Event Driver两类:
{% img /images/2014/04/input-subsystem.jpg '' 'Input Subsystem' %}
event driver创建了我们最终向应用层提供输出的设备文件`/dev/input/jsX`, 但是从头去编写我们自己的event driver并且实现所有摇杆相关的ioctl无疑工作量巨大. 所以内核向我们提供了一套input的标准事件. 我们只需编写一个device driver, 并且装成一个摇杆的样子, 向event driver上报一些上下左右或是按钮按下之类的事件, 剩下处理委托给现有的event driver去处理.
 


# 联机调试 #
