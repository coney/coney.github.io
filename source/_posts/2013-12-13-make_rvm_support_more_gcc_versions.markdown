---
author: coney
comments: true
date: 2013-12-13 11:50:40+00:00
layout: post
slug: make_rvm_support_more_gcc_versions
title: 让mac下rvm支持更多的编译器
wordpress_id: 245
categories:
- Ruby
---

通过rvm安装ruby-1.9.3-p484时, 发现rvm在尝试通过brew 安装gcc46, 并且在编译gcc46依赖的ppl011时产生错误, 导致整个安装流程失败. 但是电脑上已经通过brew安装了更高版本的gcc49, rvm看起来还不够聪明自动使用较新的编译器.

从网上搜索解决方案时看到这个commit:

{% img /images/2013/12/QQ20131210-1.png '' 'QQ20131210-1' %}

[https://github.com/wayneeseguin/rvm/commit/2b899c9c446a7e5391720f70208c0e2f71629ff0](https://github.com/wayneeseguin/rvm/commit/2b899c9c446a7e5391720f70208c0e2f71629ff0)

rvm通过这个脚本来维护支持的编译器列表, 所以依葫芦画瓢, 自己改动了下home/.rvm/scripts/functions/requirements/osx_brew文件, 添加了对gcc49的支持:

{% img /images/2013/12/QQ20131210-2.png '' 'QQ20131210-2' %}
