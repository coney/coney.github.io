---
author: coney
comments: true
date: 2013-06-28 17:32:12+00:00
layout: post
slug: mac%e4%b8%8bgem-install-mysql%e5%92%8clibv8%e6%97%b6%e7%bc%96%e8%af%91%e9%94%99%e8%af%af%e7%9a%84%e5%a4%84%e7%90%86
title: mac下gem install mysql和libv8时编译错误的处理
wordpress_id: 127
categories:
- C/C++
- Ruby
---

今天通过gem安装mysql和libv8时, 均遇到了编译错误, 尤其是mysql的比较晦涩:
    coney:cp-agentadmin$gem install mysql
    Building native extensions.  This could take a while...
    ERROR:  Error installing mysql:
    	ERROR: Failed to build gem native extension.
    
            /Users/coney/.rvm/rubies/ree-1.8.7-2012.02/bin/ruby extconf.rb
    checking for mysql_ssl_set()... no
    checking for rb_str_set_len()... no
    checking for rb_thread_start_timer()... no
    checking for mysql.h... no
    checking for mysql/mysql.h... no
    *** extconf.rb failed ***
    
    ...
    
    Results logged to /Users/coney/.rvm/gems/ree-1.8.7-2012.02@customer-platform/gems/mysql-2.9.1/ext/mysql_api/gem_make.out
起初认为是mysql安装路径有问题, 但是无论往/usr/include或者/usr/local/include中放头文件, 或者通过--with-mysql-include等指定路径, 错误依旧.  打开下面提示的gem_make.out, 发现内容跟上面显示的差不多, 没有任何帮助. 后来在gem_make.out相同目录下发现一个名为mkmf.log的日志文件, 打开后发现真正的问题在这里:
    have_header: checking for mysql.h... -------------------- no
    
    "/usr/local/bin/gcc-4.2 -E -I/opt/local/include -I. -I/Users/coney/.rvm/rubies/ree-1.8.7-2012.02/lib/ruby/1.8/i686-darwin12.4.0 -I.  -D_XOPEN_SOURCE -D_DARWIN_C_SOURCE   -I/usr/local/Cellar/mysql/5.6.12/include  -Wno-null-conversion -Os -g -fno-strict-aliasing -g -O2 -I/usr/local/opt/libyaml/include -I/usr/local/opt/readline/include -I/usr/local/opt/libxml2/include -I/usr/local/opt/libxslt/include -I/usr/local/opt/libksba/include -I/usr/local/opt/openssl098/include  -fno-common -pipe -fno-common    conftest.c -o conftest.i"
    cc1: error: unrecognized command line option "-Wno-null-conversion"
    checked program was:
    /* begin */
    1: #include <mysql.h>
    /* end */
在configure的时候, 它通过gcc尝试编译一个只有#include <mysql.h>的文件来判断是否包含mysql.h, 如果编译失败返回非0就认为头文件不存在(理论上来说这么简单的程序也不可能出现什么问题), 但是恰好mac上的这个gcc-4.2不支持-Wno-null-conversion这个开关, 导致cc失败, 所以外面打出的提示就是找不到mysql.h, 真是坑爹.
找到问题后就要想办法解决掉, google了一把, 找到这个编译选项的位置, 存放在mysql安装路径的bin/mysql_config中, 修改如下两行:
    cflags="-I$pkgincludedir  -Wall -Wno-null-conversion -Wno-unused-private-field -Os -g -fno-strict-aliasing -DDBUG_OFF " #note: end space!
    cxxflags="-I$pkgincludedir  -Wall -Wno-null-conversion -Wno-unused-private-field -Os -g -fno-strict-aliasing -DDBUG_OFF " #note: end space!
去掉其中的-Wno-null-conversion -Wno-unused-private-field, 然后重新执行gem install mysql就可以了.
当然, 还有另一种方法就是通过brew安装一个较新的gcc, 不用mac提供的那个古老的gcc, 目前我这边使用的gcc-4.9编译是没有什么问题的.

libv8的问题比较直接, 安装时提示信息如下:

    
    coney:temp$gem install libv8
    Building native extensions.  This could take a while...
    ERROR:  Error installing libv8:
    	ERROR: Failed to build gem native extension.
    
        /System/Library/Frameworks/Ruby.framework/Versions/1.8/usr/bin/ruby extconf.rb
    creating Makefile
    Compiling v8 for x64
    Unable to find a compiler officially supported by v8.
    It is recommended to use GCC v4.4 or higher
    Unable to find a compiler officially supported by v8.
    It is recommended to use GCC v4.4 or higher
    cc1plus: warnings being treated as errors
    In file included from ../src/atomicops.h:51,
                     from ../src/atomicops_internals_x86_gcc.cc:33:
    ../src/../include/v8.h:2632: warning: unused parameter ‘name’
    ../src/../include/v8.h:3093: warning: unused parameter ‘string’
    ../src/../include/v8.h:3104: warning: unused parameter ‘value’
    ../src/../include/v8.h:3104: warning: unused parameter ‘class_id’
    ../src/../include/v8.h:4092: warning: unused parameter ‘data’
    ../src/../include/v8.h:4092: warning: unused parameter ‘count’
    ../src/../include/v8.h:4302: warning: unused parameter ‘o’
    ../src/../include/v8.h:4303: warning: unused parameter ‘o’
    ../src/../include/v8.h:4304: warning: unused parameter ‘o’
    ../src/../include/v8.h:4305: warning: unused parameter ‘o’
    ../src/../include/v8.h:4306: warning: unused parameter ‘o’
    ../src/../include/v8.h:4307: warning: unused parameter ‘o’
    ../src/../include/v8.h:4308: warning: unused parameter ‘o’
    make[1]: *** [/Users/coney/.rvm/gems/ruby-2.0.0-p195/gems/libv8-3.16.14.1/vendor/v8/out/x64.release/obj.target/preparser_lib/src/atomicops_internals_x86_gcc.o] Error 1
    make: *** [x64.release] Error 2


看样子是libv8的头文件没处理好, 有一些未使用到的变量, 触发了警告, 同时不知道在哪里设置了-Werror, 所有警告都被当做错误处理导致安装失败.

正常的解决方法是修复v8.h中的warnings, 简单点的方案可以去掉-Werror或-Wall, 或者添加-Wunused-variable来忽略这种warning. 我因为没有找到-Werror在哪设置的, 所以直接将环境变量CXXFLAGS中的-Wall临时去掉进行编译. `
`
