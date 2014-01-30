---
author: coney
comments: true
date: 2013-11-09 10:34:49+00:00
layout: post
slug: '%e5%91%bd%e4%bb%a4%e8%a1%8c%e7%bc%96%e8%af%91arduino%e6%ba%90%e7%a0%81%e5%8f%8a%e4%b8%8a%e4%bc%a0%e6%89%a7%e8%a1%8c'
title: 命令行编译arduino源码及上传执行
wordpress_id: 218
categories:
- Raspi &amp; Arduino
---

Arduino官方提供了一个IDE, 集成了编辑, 编译, 上传和串口监视等功能, 但是这个基于java的IDE的编辑功能略弱. 所以想是否能使用其他的c++ IDE外加类似Makefile的命令行方式编译执行. 这样也能够在类似树莓派之类的设备上直接编译和烧录程序至arduino.

首先尝试带参启动arduino的IDE, 没有发现类似devenv.exe带参直接编译工程的功能. 之后在arduino主站搜索一番, 倒是发现了一个通过makefile的方法([http://playground.arduino.cc/Learning/CommandLine](http://playground.arduino.cc/Learning/CommandLine)), 不过提供的库略坑爹. 折腾了半天后发现提供的uart.h里面没有mega2560的定义. 从IDE中提取的库也找不到这个定义. 只能搞一些较老的芯片, 遂放弃.

继续google, 发现另外一套基于python的命令行工具ino, 试用了下, 还是比较稳定的. ino的介绍可以看这里:[http://inotool.org/](http://inotool.org/) 网站上提供了多种安装方式, 因为我的树莓派是debian的系统,  所以相关的东西基本都是通过apt安装的.
<!-- more -->

因为ino可以通过pip安装, 首先我们要安装一个pip:

    
    coney@UServer:~$ sudo apt-get install python-pip -y


然后通过pip安装ino:

    
    coney@UServer:~$ sudo pip install ino


ino只是一个构建工具, 编译时依赖于arduino的库, 有了apt真的很方便:

    
    coney@UServer:~$ sudo apt-get install arduino -y




安装好之后我们来看看ino都有什么功能:

    
    usage: ino [-h] {build,clean,init,list-models,preproc,serial,upload} ...
    
    Ino is a command-line toolkit for working with Arduino hardware.
    
    It is intended to replace Arduino IDE UI for those who prefer to work in
    terminal or want to integrate Arduino development in a 3rd party IDE.
    
    Ino can build sketches, libraries, upload firmwares, establish
    serial-communication. For this it is split in a bunch of subcommands, like git
    or mercurial do. The full list is provided below. You may run any of them with
    --help to get further help. E.g.:
    
        ino build --help
    
    positional arguments:
      {build,clean,init,list-models,preproc,serial,upload}
        build               Build firmware from the current directory project
        clean               Remove intermediate compilation files completely
        init                Setup a new project in the current directory
        list-models         List supported Arduino board models
        preproc             Transform a sketch file into valid C++ source
        serial              Open a serial monitor
        upload              Upload built firmware to the device
    
    optional arguments:
      -h, --help            show this help message and exit


命令行很简单, 但是包含了build, upload, serial monitor等功能, 已经基本覆盖了IDE提供的所有功能.

我们通过一个简单的blink工程来测试下整个流程:

    
    coney@UServer:~$ mkdir blink && cd blink
    
    coney@UServer:~/blink$ ino init -t blink
    
    coney@UServer:~/blink$ tree
    .
    ├── lib
    └── src
        └── sketch.ino
    
    coney@UServer:~/blink$ cat src/sketch.ino
    
    #define LED_PIN 13
    
    void setup()
    {
        pinMode(LED_PIN, OUTPUT);
    }
    
    void loop()
    {
        digitalWrite(LED_PIN, HIGH);
        delay(100);
        digitalWrite(LED_PIN, LOW);
        delay(900);
    }


首先是通过ino init创建一个arudino的工程, -t可以指定模板(实际除了默认的empty就提供了一个blink模板). 创建后可以看到工程的目录结构包含src和lib, 分别存放工程源码ino文件和将来要引入的库.

从blink模板创建的源码包含一个简单的闪烁led的示例. 程序将反复将13口(arduino板子上13口接着一个led)在高低电平见切换.

接下来我们要把刚才的程序编译并且上传到arduino上执行:

    
    coney@UServer:~/blink$ ino build
    Searching for Board description file (boards.txt) ... /usr/share/arduino/hardware/arduino/boards.txt
    Searching for Arduino lib version file (version.txt) ... /usr/share/arduino/lib/version.txt
    Detecting Arduino software version ...  1.0.0 (1.0)
    Searching for Arduino core library ... /usr/share/arduino/hardware/arduino/cores/arduino
    Searching for Arduino standard libraries ... /usr/share/arduino/libraries
    Searching for Arduino variants directory ... /usr/share/arduino/hardware/arduino/variants
    Searching for make ... /usr/bin/make
    Searching for avr-gcc ... /usr/bin/avr-gcc
    Searching for avr-g++ ... /usr/bin/avr-g++
    Searching for avr-ar ... /usr/bin/avr-ar
    Searching for avr-objcopy ... /usr/bin/avr-objcopy
    src/sketch.ino
    Searching for Arduino lib version file (version.txt) ... /usr/share/arduino/lib/version.txt
    Detecting Arduino software version ...  1.0.0 (1.0)
    Scanning dependencies of src
    Scanning dependencies of arduino
    src/sketch.cpp
    arduino/WInterrupts.c
    arduino/wiring_digital.c
    arduino/wiring_pulse.c
    arduino/wiring.c
    arduino/wiring_shift.c
    arduino/wiring_analog.c
    arduino/new.cpp
    arduino/IPAddress.cpp
    arduino/HardwareSerial.cpp
    arduino/Print.cpp
    arduino/Stream.cpp
    arduino/HID.cpp
    arduino/WMath.cpp
    arduino/WString.cpp
    arduino/Tone.cpp
    arduino/main.cpp
    arduino/USBCore.cpp
    arduino/CDC.cpp
    Linking libarduino.a
    Linking firmware.elf
    Converting to firmware.hex


编译生成的文件都放在.build中, 所以通过ls看不到的时候千万别紧张~ 接着我们要把程序uplaod上去:

    
    [~/arduino/blink]$ ino upload
    Searching for stty ... /bin/stty
    Searching for avrdude ... /usr/share/arduino/hardware/tools/avrdude
    Searching for avrdude.conf ... /usr/share/arduino/hardware/tools/avrdude.conf
    Guessing serial port ... /dev/ttyACM0
    avrdude: stk500_recv(): programmer is not responding
    
    avrdude done.  Thank you.


ino比较智能, 会探测串口对应的tty, 但是上传好像失败了? 查看下upload的帮助:

    
    [~/arduino/blink]$ ino upload --help
    usage: ino upload [-h] [-p PORT] [-m MODEL] [-d PATH]
    
    Upload built firmware to the device.
    
    The firmware must be already explicitly built with `ino build'. If current
    device firmare reads/writes serial port extensively, upload may fail. In
    that case try to retry few times or upload just after pushing Reset button
    on Arduino board.
    
    optional arguments:
      -h, --help            show this help message and exit
      -p PORT, --serial-port PORT
                            Serial port to upload firmware to
                            Try to guess if not specified
      -m MODEL, --board-model MODEL
                            Arduino board model (default: uno)
                            For a full list of supported models run:
                            `ino list-models'
      -d PATH, --arduino-dist PATH
                            Path to Arduino distribution, e.g.
                            ~/Downloads/arduino-0022.
                            Try to guess if not specified


原来默认的model是uno, 而我的是mega2560. 重新执行build和upload, 指定model:(如果探测不到正确的tty, 可以通过-p指定串口)

    
    [~/arduino/blink]$ ino build -m mega2560
       ...
    [~/arduino/blink]$ ino upload -m mega2560
    Searching for stty ... /bin/stty
    Searching for avrdude ... /usr/share/arduino/hardware/tools/avrdude
    Searching for avrdude.conf ... /usr/share/arduino/hardware/tools/avrdude.conf
    Guessing serial port ... /dev/ttyACM0
    
    avrdude: AVR device initialized and ready to accept instructions
    
    Reading | ################################################## | 100% 0.02s
    
    avrdude: Device signature = 0x1e9801
    avrdude: reading input file ".build/mega2560/firmware.hex"
    avrdude: writing flash (1556 bytes):
    
    Writing | ################################################## | 100% 0.25s
    
    avrdude: 1556 bytes of flash written
    avrdude: verifying flash memory against .build/mega2560/firmware.hex:
    avrdude: load data flash data from input file .build/mega2560/firmware.hex:
    avrdude: input file .build/mega2560/firmware.hex contains 1556 bytes
    avrdude: reading on-chip flash data:
    
    Reading | ################################################## | 100% 0.19s
    
    avrdude: verifying ...
    avrdude: 1556 bytes of flash verified
    
    avrdude: safemode: Fuses OK
    
    avrdude done.  Thank you.


一切就绪, 可以看到板子上led在闪烁.

如果觉得每次都要-m指定model或者-p指定tty的话, 可以通过ino.ini来配置model和port:

    
    [~/arduino/blink]$ cat ino.ini
    [build]
    board-model = mega2560
    
    [upload]
    board-model = mega2560
    serial-port = /dev/ttyACM0
    
    [serial]
    serial-port = /dev/ttyACM0


ino.ini可以放在工程目录下对当前工程生效, 也可以放在/etc/下, 作为全局配置. 设置好ino.ini后, 我们就不用每次build或者upload的时候都显式指定model.

最后是ino的serial monitor功能, 这个实际是依赖于picocom, 这个功能其实不需要ino也能用. 有兴趣可以看看官方的示例.

另外, 在Arduino 1.5版本中, 好像也提供了更官方的命令行支持:[https://github.com/arduino/Arduino/wiki/Arduino-IDE-1.5-from-command-line](https://github.com/arduino/Arduino/wiki/Arduino-IDE-1.5-from-command-line) , 因为目前还在beta, 尚未体验过.
