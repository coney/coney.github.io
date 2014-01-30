---
author: coney
comments: true
date: 2013-03-23 09:13:30+00:00
layout: post
slug: c%e5%ae%9e%e7%8e%b0%e7%9a%84%e6%8e%a5%e5%8f%a3%e6%9c%aa%e6%8f%90%e4%be%9b%e8%99%9a%e6%9e%90%e6%9e%84%e7%9a%84%e5%90%8e%e6%9e%9c
title: c++声明接口未提供虚析构函数的后果
wordpress_id: 55
categories:
- C/C++
tags:
- Interface 虚析构
---

这是一个比较低级的错误, 用C++模拟C#提供接口, 声明了纯虚函数, 看起来像这样:

``` cpp
class ITestable
{
public:
    virtual void Test() = 0;
};
```
<!-- more -->

这个接口看起来OK, 用起来也OK, 但是问题是拿到实例化后的对象通过接口形式的指针释放的时候, 没有调用到实际对象的析构函数, 像这样:

``` cpp
ITestable *test = new TestCase();
delete test;
```

执行上面的语句, TestCase对象的析构函数未被调用到. 问题原因很简单, 就是c++虚析构的问题, 只不过声明接口的时候, 为了美观忘了接口也是个类, 没有添加虚析构函数. 修复后的代码如下:

```
class ITestable
{
public:
    virtual ~ITestable() {}
    virtual void Test() = 0;
};
```
