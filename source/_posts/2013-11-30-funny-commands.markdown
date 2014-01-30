---
author: coney
comments: true
date: 2013-11-30 10:30:32+00:00
layout: post
slug: funny-commands
title: Funny Commands
wordpress_id: 225
---




最近总是能够从其他同事那里学来一些有趣的命令, 害怕时间长忘记了, 所以开个帖子记录下.




# ipcalc:
这个命令是用来计算IP或者子网的, 能够以漂亮的色彩分析ip或者子网的范围, 掩码等信息. 我比较喜欢用这个东西来查看类似192.168.0.0/22这种子网的范围.
{% img /images/2013/11/QQ20131130-2.png '' 'QQ20131130-2' %}
<!-- more -->

# watch
在发现watch命令前, 我一直是使用while+sleep的方法反复执行某个命令, watch除了提供重复执行命令外 还有一个比较好玩的功能, 就是显示两次命令间的差异.
{% img /images/2013/11/QQ20131130-3.png '' 'QQ20131130-3' %}
例如执行watch -d -n1 date, 每隔一秒执行一次date命令, 并显示结果, 并且高亮两次命令结果的差异, 执行效果:
{% img /images/2013/11/QQ20131130-4.png '' 'QQ20131130-4' %}
