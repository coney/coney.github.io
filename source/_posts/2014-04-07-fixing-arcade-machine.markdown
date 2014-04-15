---
author: Coney
layout: post
title: 西安Office游戏机修复之路
date: 2014-04-07 16:33:24 +0800
comments: true
categories:
- Linux/Device Driver MAME
---

加入Thoughtworks前参观西安Office时就已经注意到餐厅的游戏机, 但入职后才知道游戏机已经坏了有些年月. 机缘巧合, 加入硬件小组后了解到有修复游戏机的计划, 所以便有了以下的经历.

#方案选择#
开搞之前, 新宇和高亮已经把游戏机肚里的东西摸得一清二楚. 游戏机主板寄给某个淘宝店家后已杳无音讯, 屏幕则是一块接受RGBS信号的25寸显像管, 摇杆和按钮是最直接的按键开关, 闭合后会跟GND连通. 有了这些基本输入输出信息, 下一步就是选择合适的软硬件来重建主板.

回想曾经的街机经历, 无非也就两种选择: 一是到游戏厅玩真机, 第二种便是通过PC上的模拟器. 重买一个街机主板放进去肯定不是我们想要的方案, 所以我们基本确定思路就是通过PC主机+模拟器的方式. 比较常见的街机模拟器有winkawaks, nebula和MAME等, 相信不少人都有玩过:

![MAME](/images/2014/04/MAME.jpg)
<!-- more -->
MAME for windows

但是这些主流模拟器大多都是基于Windows, 而往往支持Windows平台的设备成本都比较高. 幸好新宇已经提前做过不少预研, 推荐我们使用一款叫做[AdvancedMAME](advancemame.sourceforge.net)的跨平台街机模拟器. 这样我们就能将模拟器部署在较低成本的Linux设备上.

硬件方面, 市面上大多数两三百元左右的嵌入式开发板都能够流畅运行Linux并且带有VGA或HDMI等显示输出. 而手边正好有树莓派和CubieBoard两种板子, 相比而言树莓派更加成熟稳定, 资料也相对丰富, 所以这次选择树莓派来扮演游戏机主板的角色.
树莓派本身带有十多个GPIO接口, 可以直接用来接入游戏机摇杆信号. 而图像处理要稍微麻烦一些, 因为树莓派提供了HDMI及同轴模拟视频接口, 不能直接兼容游戏机的RGBS信号, 需要额外的设备将HDMI输入转换为RGBS输出. 幸运的是这个问题可以通过万能的淘宝来解决, 我们购买新开发板时同时购入了一套HDMI转VGA再转RGBS的装置. 整套装置的连接示意图如下:

![HD layout](/images/2014/04/hd-layout.png)

# 交叉编译模拟器 #
方案就绪, 下一步就是先将游戏在树莓派上跑起来. AdvanceMAME是基于C的跨平台软件, API兼容而ABI不兼容, 所以如果想要在arm架构的树莓派上跑起来, 需要对模拟器进行交叉编译.
获取交叉编译工具的方法很多, 可以通过包管理器安装, 或是从树莓派的github上进行[下载](https://github.com/raspberrypi/tools). 如果你使用mac或cygwin的话, 还可以通过crosstool-ng来生成自己的交叉编译工具链. 这里我们选择最简单的方式, 从树莓派的github上clone一份:

`git clone https://github.com/raspberrypi/tools`

clone完成后, 我们可以尝试编译一个简单的小程序, 验证交叉编译环境是否OK, 国际惯例, hello world:
``` c hello.c
#include <stdio.h>

int main(int argc, char **argv)
{
	printf("hello world\n");
	return 0;
}
```
将代码保存至hello.c, 之后调用gcc进行编译
``` text
coney@UServer: ~ $ ./raspi/tools/arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian/bin/arm-linux-gnueabihf-gcc  hello.c -o hello
coney@UServer: ~ $ file hello
hello: ELF 32-bit LSB executable, ARM, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.26, BuildID[sha1]=0x1a4b10f8034567384fbc51490dc09d6646262a58, not stripped
```
`raspi/tools`这个目录是刚刚clone下来的工具链目录. 编译完成后通过`file`命令可以查看生成的文件是否是32位的arm应用程序. 接下来将`hello`程序复制到树莓派上执行, 验证我们的程序是否真的能hello world:
```
coney@coney-pi: ~ $ ./hello
hello world
```
交叉编译工具验证没有问题后, 下一步就是获取AdvanceMAME的源码进行编译, 源码可以在此处下载:

http://advancemame.sourceforge.net/download.html

下载后解开tar包, 进入源码目录, 首先要做的是调用`configure`生成Makefile. 因为这次是交叉编译, 所以在./configure之前需要将交叉编译工具的路径加入PATH中:

`export PATH=~/raspi/tools/arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian/bin/:$PATH`

运行`configure`时, 指定目标平台为arm-linux-gnueabihf, 另外为了方便获取编译后的程序, 显式指定`--prefix`来更改安装路径:

`./configure --prefix=/home/coney/mame-arm/ --host=arm-linux-gnueabihf`

接着执行`make`开始编译, 并将编译后的内容安装到`--prefix`指定的目录中:

`make all install`

编译完成后, 将`--prefix`指定目录中的内容全部复制到树莓派上, 给树莓派连接键盘显示器, 在树莓派上体验下刚刚编译的模拟器.
``` text
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
`/opt/mame-arm`是编译后的模拟器目录, 第一次运行AdvanceMAME时, 会在当前用户目录下创建`.advance`目录. 默认情况下, AdvanceMAME所有的配置, 游戏Rom, 截图等都放在这里. 我们将下载的游戏Rom拷入`.advance/rom`中(可能存在版权问题, 请自行查找rom), 再次启动AdvanceMAME并传入游戏名. 呵呵, 还是没有启动起来. 根据提示, 我们没有配置显示器的刷新率, 根据网上现有的经验, 使用树莓派HDMI接口输出的话, 可以编辑`.advance/advmame.rc`并加入这行配置:

`device_video_clock 5 - 50 / 15.62 / 50 ; 5 - 50 / 15.73 / 60`

再次运行`advmame snowbros`启动游戏, 不出意外的话你就能看到久违的游戏画面:

图片

p.s. AdvanceMAME可以直接操纵FrameBuffer, 不需要启动X, 但是需要root权限.

# 编写摇杆驱动 #
到此游戏已经能够在树莓派上运行, 通过视频转换接入游戏机后便能够通过游戏机的显像管输出. 但是现在还有一个重要的问题没有解决, 那就是游戏的操控只能通过键盘而不是游戏机摇杆.
关于如何接入游戏机摇杆, 我们曾考虑通过GPIO读入信号, 然后模拟键盘事件. 但是通过AdvanceMAME附带的几个测试程序, 很神奇的发现Linux居然有专门的摇杆设备标准, 真是工作娱乐两不误的好系统. 虽然网上关于linux摇杆驱动的内容并不多, 但幸运的是linux内核是开源的, 并带有大量第三方摇杆驱动源码可以借鉴(山寨).
摇杆驱动在linux中属于Input Driver的一种. Input Driver分为Input Device Driver及Input Event Driver两类, 大致关系如下图:

![Input Subsystem](/images/2014/04/input-subsystem.jpg)

其中event driver创建了我们最终向应用层提供输出的设备文件`/dev/input/jsX`, 但重新编写我们自己的event driver并且实现所有摇杆相关的ioctl无疑工作量巨大. 所以内核向我们提供了一套input相关的标准事件. 我们只需编写一个device driver, 并且装成一个摇杆的样子, 向event driver上报一些上下左右或是按钮按下之类的事件, 剩下处理就可以委托给现有的event driver去处理.
这里忽略驱动框架部分的内容, 主要介绍摇杆事件相关的处理. 首先通过`input_allocate_device`创建一个input device:
``` c Allocate Joystick Device
struct input_dev *js_input_dev = input_allocate_device();
```
接着我们向input子系统声明将要上报摇杆专属事件, 这样上层的event driver会将这个device识别成一个摇杆, 并在`/dev/input/`下创建对应的摇杆设备文件:
``` c Report Events
// report joystick button and direction
set_bit(EV_KEY, js_input_dev->evbit);
set_bit(EV_ABS, js_input_dev->evbit);

// direction x and y range from -1 to 1
input_set_abs_params(js_input_dev, ABS_X, -1, 1, 0, 0);
input_set_abs_params(js_input_dev, ABS_Y, -1, 1, 0, 0);

// report key ABCXYZ
set_bit(BTN_A, js_input_dev->keybit);
set_bit(BTN_B, js_input_dev->keybit);
set_bit(BTN_C, js_input_dev->keybit);
set_bit(BTN_X, js_input_dev->keybit);
set_bit(BTN_Y, js_input_dev->keybit);
set_bit(BTN_Z, js_input_dev->keybit);

// register this device to input subsystem
input_register_device(js_input_dev)
```
最后, 我们需要采集gpio数据并且上报input事件, 目前的实现使用中断来获取gpio变化, 触发定时器并在定时器回调中判断GPIO的最终状态, 上报相关的input事件. 定时器的作用主要是为了消除按键抖动(貌似抖得不是很厉害):
``` c Read Events From GPIO

typedef struct gpio_config
{
    unsigned int button;
    unsigned int gpio;
    unsigned int irq;
    struct timer_list timer;
    char irq_name[16];
} gpio_config_t;

static int setup_gpio_irq(gpio_config_t *config, unsigned int gpio,
                          unsigned int button)
{
    // register associated joystick button
    config->button = button;
    config->gpio = gpio;
    config->irq = gpio_to_irq(gpio);
    snprintf(config->irq_name, sizeof(config->irq_name),
        "gpio%02u", gpio);

    // report input events in this timer callback
    init_timer(&config->timer);
    config->timer.data = (unsigned long)config;
    config->timer.function = gpio_timer_callback;

    // register interrupt handler for gpio edge events
    return request_irq(config->irq, gpio_irq_handler,
        IRQF_TRIGGER_RISING | IRQF_TRIGGER_FALLING,
        config->irq_name, config);
}

static irqreturn_t gpio_irq_handler(int irq, void *data)
{
    gpio_config_t *config = (gpio_config_t *)data;
    // delay 5 milliseconds to make sure that the signal is stabilized
    mod_timer(&config->timer, jiffies + 5);
    return IRQ_HANDLED;
}

static void gpio_timer_callback(unsigned long data)
{
    gpio_config_t *config = (gpio_config_t *)data;
    int value = !gpio_get_value(config->gpio);

    // call input_report_key or input_report_abs to report an input event
    // with associated joystick button, and then call input_sync to flush the event
    js_device_process(0, config->button, value);
}
```
以上是摇杆驱动的核心功能部分代码, 完整的代码请猛击这里:

https://github.com/coney/tw-joystick.git

驱动代码同样需要进行交叉编译, 驱动的交叉编译会稍微麻烦些, 首先要获取对应版本的内核源码. 内核源码可以从树莓派的官方github上得到, 需要注意的是clone的branch要和树莓派的内核版本对应, 树莓派的内核版本可以通过`uname -r`进行获取. 例如我的树莓派内核版本是`3.10.28+`, 我要获取的branch就是`rpi-3.10.y`:

`git clone https://github.com/raspberrypi/linux.git -b rpi-3.10.y`

获取到内核源码后, 接下来要拿到当前树莓派使用的内核配置, 只有配置相同才能保证编译出来驱动与内核兼容. 树莓派当前的内核配置可以很方便的通过`/proc/config.gz`进行获取. 将这个文件复制并解压到内核源码的目录, 接着导出一些环境变量, 让内核和编译器知道我们将要进行交叉编译:

``` bash
export RASPI_BASE=$(pwd)
# kernel source path, will be used in the Makefile of joystick driver
export KERNEL_SRC=$RASPI_BASE/linux
export ARCH=arm
# cross compiler path and prefix
export CCPREFIX=$RASPI_BASE/tools/arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian/bin/arm-linux-gnueabihf-
export CROSS_COMPILE=$CCPREFIX
```

这些环境变量准备妥当后, 需要先进入内核源码目录`make`一次, 虽然我们不会去更新树莓派的更新内核, 但是这次`make`能够对内核源码进行一些配置(例如最大CPU数量, 特性开关, 内核符号依赖等). 这次编译可能会比较漫长, 但只要今后内核版本及配置文件没有更改, 可以一直使用这个源码来辅助编译驱动.
 
内核编译完成后, 进入摇杆驱动目录, 执行`make`, 编译成功后, 将生成的`tw_joystick.ko`复制到树莓派, 并以root身份执行以下命令:

`insmod tw_joystick.ko`

顺利的话, 通过`dmesg`能够看到一些驱动输出的调试信息. 因为游戏机摇杆按键闭合后会跟GND联通, 所以在驱动正常工作前要将GPIO设置为上拉模式, 这样GPIO在开关闭合前是稳定的高电平, 闭合后变为低电平. 通过WiringPI中提供的gpio工具可以方便的更改GPIO状态. 安装WiringPI请参考这个链接:

https://projects.drogon.net/raspberry-pi/wiringpi/download-and-install/

配置好WiringPI后, 在shell下运行这段命令将所有GPIO设为pull up模式:

`for ((i=0;i<=20;i++)); do gpio mode $i up; done`

在接入游戏前, 通过AdvanceMAME中的`advj`工具或者`joystick`包中的`jstest`工具可以更方便直观的测试摇杆驱动, 以`jstest`为例:

`jstest --event /dev/input/js0`

启动命令后, `jstest`会一直监听摇杆事件. 此时从树莓派的GND引出一根线, 分别碰触驱动中定义的GPIO引脚(千万不要碰到VCC), 观察`jstest`是否有正确的摇杆事件输出, 例如:

``` text
root@coney-pi: /home/coney # jstest --event /dev/input/js0
Driver version is 2.1.0.Joystick (Thoughtworks Fake Joystick) has 2 axes (X, Y)
and 8 buttons (BtnX, BtnY, BtnZ, BtnTL, BtnTR, BtnTL2, BtnTR2, BtnSelect).
Testing ... (interrupt to exit)
Event: type 129, time 347000, number 0, value 0
Event: type 129, time 347000, number 1, value 0
Event: type 129, time 347000, number 2, value 0
Event: type 129, time 347000, number 3, value 0
Event: type 129, time 347000, number 4, value 0
Event: type 129, time 347000, number 5, value 0
...
```

# 联机调试 #
万事具备, 只欠东风. 输入输出都搞定后就可以上机测试啦. 这里不得不赞一下游戏机的金手指接口定义, 对开发者相当友好, 每一个引脚都有对应的中文描述, 省去了分析引脚功能的烦恼.
因为我们没有专业的金手指插片, 所以临时使用一张硬纸片分离开插槽两边的弹簧片以达到绝缘效果. 同时通过新宇精湛的手艺, 所有摇杆按钮和电源相关的引脚都焊接了一根面包线, 用来连接树莓派的GPIO, 完成的效果图如下:

`图片`

图像显示则通过两个视频转换设备, 将HDMI转为VGA再转至RGBS, 并将最终输出的4根信号线接在金手指插槽上(标注红绿蓝信号的几个引脚).

`图片`

上电开机, 此时我们还是需要一些键盘的辅助, 首先是通过`advj`或`jstest`测试连接的摇杆按键是否灵敏. 确定摇杆功能正常后, 启动MAME加载游戏, 在游戏中按`TAB`键呼出MAME功能菜单, 进入摇杆控制设置, 校对各个功能的按键配置, 并将多出来的两个按键设置为投币和开始游戏. 接下来便可以开始这台山寨街机初体验:

`图片`

至此游戏机基本功能已经恢复, 但是每次都要接着键盘启动游戏着实不爽, 下面要做的就是让游戏机能够自行启动并加载游戏.
最初尝试的做法是通过在`rcX.d`中加入一个脚本来启动游戏, 但是这样启动的AdvanveMAME有些问题(可能是因为没有tty的原因), 最终采取的方式是让linux启动后自动以root用户登陆一个tty, 并bash profile中判断如果是从这个tty登录, 插入驱动并启动游戏.
修改系统的`/etc/inittab`, 将tty6改为`mingetty`并自动以root登陆:
``` text /etc/inittab
...
1:2345:respawn:/sbin/getty --noclear 38400 tty1
#2:23:respawn:/sbin/getty 38400 tty2
#3:23:respawn:/sbin/getty 38400 tty3
#4:23:respawn:/sbin/getty 38400 tty4
#5:23:respawn:/sbin/getty 38400 tty5
6:23:respawn:/sbin/mingetty --autologin root tty6
...
```
修改`/root/.profile`, 新增如下几行, 在通过tty6登陆后加载驱动并启动游戏:
``` bash /root/.profile
...
GAME=snowbrow
if [[ $TTY == "/dev/tty6" ]]; then
    insmod tw_joystick.ko
    advmame $GAME
fi
```
重启游戏机, 系统引导完成后, 游戏应该会自动启动, enjoy yourself!
