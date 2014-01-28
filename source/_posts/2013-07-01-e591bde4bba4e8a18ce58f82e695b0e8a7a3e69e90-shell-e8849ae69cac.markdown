---
author: coney
comments: true
date: 2013-07-01 05:05:46+00:00
layout: post
slug: '%e5%91%bd%e4%bb%a4%e8%a1%8c%e5%8f%82%e6%95%b0%e8%a7%a3%e6%9e%90-shell-%e8%84%9a%e6%9c%ac'
title: 命令行参数解析 - Shell 脚本
wordpress_id: 142
categories:
- Linux/Network
tags:
- getopt
- shell
---

平时多少会用shell写点小工具, 而这些小工具运行后的第一件事就是解析参数, 这里总结了下shell脚本几种处理命令行参数的方法.

比较常见的做法就是解析bash内置的几个特殊变量, 例如直接遍历$*或者$@:

    
    #/bin/sh
    echo 'args from \$*'
    for arg in $*; do
        echo $arg
    done
    
    echo 'args from \$@'
    for arg in $@; do
        echo $arg
    done
    
    echo 'args from "\$*"'
    for arg in "$*"; do
        echo $arg
    done
    
    echo 'args from "\$@"'
    for arg in "$@"; do
        echo $arg
    done


上面总共列出了4种遍历的格式, 分别对$@和$*进行遍历, 这两个变量的区别是$*将所有的参数合为一个字符串, 而$@将输入的参数分别保存不同的字符串, 例如执行get-args.sh a "b c" d的结果如下:

    
    coney:temp$./get-args.sh a "b c" d
    args from $*
    a
    b
    c
    d
    args from $@
    a
    b
    c
    d
    args from "$*"
    a b c d
    args from "$@"
    a
    b c
    d


还有一种方法就是通过shift方法来将处理过的参数移除, 从而完成遍历:

    
    #!/bin/sh
    while [ $# != 0 ]; do
        echo $1
        shift
    done


其中$1代表第一个参数, 类似的$2, $3...分别代表第二个 第三个参数. $#是参数的总个数, 每次调用shift后, 会将所有的参数左移, 并将第一个参数移除, 所以在脚本里面我们只要一直访问$1即可, 这种方式比较适合解析类似-p 80这种类型的参数, 可以在解析到-p后, shift一下, 然后再解析该参数附带的内容. 执行shift-args.sh a "b c" d的结果类似遍历"$@":

    
    coney:temp$./shift-args.sh a "b c" d
    a
    b c
    d


以上是使用shell内建变量处理的方式, 虽然遍历较为简单, 但是解析参数的时候很复杂, 尤其是要对参数的合法性做校验时, 需要大量逻辑.

所以我们有更好的选择: getopt. getopt命令可以将根据格式化字符串将输入参数重新整理排序, 方便我们进行处理, 并且对输入的参数格式进行校验, 判断参数格式是否正确以及符合规范. 下面是一个getopt的使用示例:

    
    #!/bin/sh
    args=`getopt abc: "$@"`
    
    # check arguments, exit on fail
    if [ $? != 0 ]; then
        exit $?
    fi
    
    # reorder arguments
    echo before reorder, args : $*
    set -- $args
    echo after reorder, args : $*
    
    # parse arguments
    while :; do
        opt=$1; shift
        case $opt in
            -a|-b)
                echo $opt is set;;
            -c) 
                echo found $opt with $1; shift;;
            --) 
                break;;
        esac
    done
    
    # print rest arguments
    if [ -n "$*" ]; then
        echo arguments : $*
    fi


脚本一开始将格式化字符串和脚本接受的所有参数传递给getopt, getopt执行后将重新排序后的参数存储在args中. 当然getopt会对参数进行校验, 如果校验失败会直接在stderr上输出错误信息, 并且将返回码设为非0, 这样我们就能够及时终止脚本执行.



我们传入的格式化字符串"abc:"的含义是, 接收-a -b -c三种开关参数, 并且-c开关有附加参数, 例如"-c file".

在拿到getopt重新排序的参数后, 我们要通过"set -- $args"替换脚本接收的参数, 这样$* $@ $1 $2等变量就可以使用排序后的参数, 先看下这个命令的运行效果:

    
    coney@UServer:~/temp$ ./getopt.sh file1 -ab -c "haha" file2
    before reorder, args : file1 -ab -c haha file2
    after reorder, args : -a -b -c haha -- file1 file2
    -a is set
    -b is set
    found -c with haha
    arguments : file1 file2


getopt将参数通过--分隔为两部分, 前面是控制开关, 后面是其他传入的参数内容, 并且支持"-ab"这种格式的参数, 并自动分解为"-a -b". 这样我们就能顺序遍历所有的开关, 然后再处理其他参数. 如果是"-c haha"这种类型的参数, 我们只需要在遍历的过程中多shift一次, 取出附加的参数即可. 在遍历到"--"后, 剩下的参数便都是非开关的参数了.

再来看下getopt对参数校验的效果:

    
    coney@UServer:~/temp$ ./getopt.sh -d -a -b
    getopt: invalid option -- 'd'
    coney@UServer:~/temp$ ./getopt.sh -a -b -c
    getopt: option requires an argument -- 'c'
    coney@UServer:~/temp$ ./getopt.sh -a --b
    getopt: unrecognized option '--b'


所以通过getopt进行参数, 不但极大的提升开发效率, 还能够完美的支持标准的参数格式, 以及获得健壮的错误处理¤
