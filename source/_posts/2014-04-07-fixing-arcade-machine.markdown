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
获取交叉编译工具的方法很多, 可以通过包管理器安装, 或是从树莓派的github上进行[下载](https://github.com/raspberrypi/tools). 如果你使用mac或cygwin的话, 还可以通过crosstool-ng来生成自己的交叉编译工具链. 这里我们选择最快捷方式, 直接从树莓派的github上下载.
交叉编译工具准备好后, 下一步就是获取AdvanceMAME的源码进行编译, 代码可以在此处下载: http://advancemame.sourceforge.net/download.html
将下载的tar包解压后, 进入

# 编写摇杆驱动 #
# 联机调试 #

