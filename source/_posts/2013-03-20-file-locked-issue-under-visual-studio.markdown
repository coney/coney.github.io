---
author: coney
comments: true
date: 2013-03-20 16:36:22+00:00
layout: post
slug: vs%e7%bc%96%e8%af%91%e8%bf%90%e8%a1%8c%e5%90%8e%e5%8f%af%e6%89%a7%e8%a1%8c%e6%96%87%e4%bb%b6%e4%b8%8d%e8%83%bd%e5%86%8d%e6%ac%a1%e5%86%99%e5%85%a5%e7%9a%84%e9%97%ae%e9%a2%98
title: VS编译运行后可执行文件不能再次写入的问题
wordpress_id: 48
categories:
- WinDev
---

在公司和家里的电脑上都遇到这个问题, 编译运行程序后, 无论clean或者rebuild, 生成的exe都不能删除掉, 通过processhacker查看是system占用, 怀疑是杀软引用了exe的文件句柄. 可惜卸载杀软后问题同样出现. 提示信息如下:
    LINK : fatal error LNK1104: cannot open file 'D:\Coney\Desktop\Solution\Bin\ProjectApp.exe'
后来google下, 发现已经有人碰到过类似问题了,  原因是没有开启Application Experience服务, 大体原因是该服务维护应用程序缓存, 当删除文件时, 应用程序标记为待删除, 待所有打开的进程句柄关闭后进行删除. 但是因为Application Experince服务没有启动, 导致无法获取这个应用程序状态, 所以要等待一段时间应用程序才能被删除掉.

ref: 

[http://stackoverflow.com/questions/3906404/link-fatal-error-lnk1104-cannot-open-file-d-myproj-exe?1363796963](http://stackoverflow.com/questions/3906404/link-fatal-error-lnk1104-cannot-open-file-d-myproj-exe?1363796963)

[http://www.retrocopy.com/blog/28/cant-delete-exe-files-in-vista--windows-7-solved.aspx](http://www.retrocopy.com/blog/28/cant-delete-exe-files-in-vista--windows-7-solved.aspx)
