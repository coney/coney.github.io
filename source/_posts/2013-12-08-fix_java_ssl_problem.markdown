---
author: coney
comments: true
date: 2013-12-08 09:21:09+00:00
layout: post
slug: fix_java_ssl_problem
title: Java下"Failure initializing default system SSL context"的修复
wordpress_id: 241
categories:
- Java
tags:
- gradle java ssl
---

今天准备用java+gradle写个hello world, 结果在download jar包时卡了很久. 错误的信息就像这个样子:

    
    > Could not resolve org.eclipse.jetty:jetty-server:9.1.0.v20131115.
    Required by:
    :blog-search:1.0
    > Failure initializing default system SSL context


从网上搜了很多内容, 大部分都没有什么帮助, 后来发现不是gradle的问题, 貌似是使用了java的https并且证书有问题导致的. 根据网上的一些建议应该是证书的问题, 于是参考这篇文章准备修改默认的keystore:[http://publib.boulder.ibm.com/infocenter/javasdk/v6r0/index.jsp?topic=%2Fcom.ibm.java.security.component.60.doc%2Fsecurity-component%2Fjsse2Docs%2Fcustomizingstores.html](http://publib.boulder.ibm.com/infocenter/javasdk/v6r0/index.jsp?topic=%2Fcom.ibm.java.security.component.60.doc%2Fsecurity-component%2Fjsse2Docs%2Fcustomizingstores.html)
<!-- more -->

当我准备打开 <java-home>/lib/security/cacerts的时候, 发现这个文件是一个软连接指向另一个目录下:

    
    lrwxr-xr-x  1 root  wheel    81B Mar 18  2013 cacerts -> /System/Library/Java/Support/CoreDeploy.bundle/Contents/Home/lib/security/cacerts


然后指向的/System/Library/Java/Support/CoreDeploy.bundle/Contents/下面根本没有Home目录. 不知道是安装的问题导致的还是什么, 正因为缺少这个东西而导致java https产生异常.

修复的方法是从同事的机器拷来相同的文件放到对应目录. 若大家遇到相同的问题. 可以先从这里入手检查下证书是否正确.




