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

今日准备用java + gradle体验下hello world, 结果在download jar包时卡了很久. 错误信息像这个样子:

``` text
> Could not resolve org.eclipse.jetty:jetty-server:9.1.0.v20131115.
Required by:
:blog-search:1.0
> Failure initializing default system SSL context
```

从网上查了很多内容, 大部分都没有什么帮助, 后来发现并不是gradle的问题, 而是在使用java的https时证书有问题导致. 参考网上的资料, 要对keystore进行一些处理:

[http://publib.boulder.ibm.com/infocenter/javasdk/v6r0/index.jsp?topic=%2Fcom.ibm.java.security.component.60.doc%2Fsecurity-component%2Fjsse2Docs%2Fcustomizingstores.html](http://publib.boulder.ibm.com/infocenter/javasdk/v6r0/index.jsp?topic=%2Fcom.ibm.java.security.component.60.doc%2Fsecurity-component%2Fjsse2Docs%2Fcustomizingstores.html)
<!-- more -->

当我准备对keystore动手时, 突然发现`<java-home>/lib/security/cacerts`是一个软链接并且无法访问:

``` text
lrwxr-xr-x  1 root  wheel    81B Mar 18  2013 cacerts -> /System/Library/Java/Support/CoreDeploy.bundle/Contents/Home/lib/security/cacerts
```

`/System/Library/Java/Support/CoreDeploy.bundle/Contents/`下面根本没有Home目录. 不清楚为什么这里东西会消失, 但正因为缺少这个目录的内容而导致使用java https时产生异常.

修复的方法是从某同事的机器拷来对应目录的所有文件放到本机. 若大家遇到类似SSL的问题, 不防检查下证书目录是否有问题.




