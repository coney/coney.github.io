---
author: coney
comments: true
date: 2013-03-27 05:24:43+00:00
layout: post
slug: '%e7%9b%b4%e6%8e%a5%e6%94%b9%e5%86%99stdstring%e5%86%85%e5%ad%98%e5%af%bc%e8%87%b4%e7%9a%84%e9%97%ae%e9%a2%98'
title: 直接改写std::string内存导致的问题
wordpress_id: 58
categories:
- C/C++
tags:
- string
---

今天帮一个同事查问题: 他写了一个全局的string变量, 在程序运行过程中可能会对该变量进行解析操作, 但是在持续运行过程中第二次访问到这个全局变量的时候, string的内容发生了改变. 他代码大体逻辑类似下面这样:

``` cpp
    void process(const string &rawstr)
    {
        string delim = ",";
        string newstr = rawstr;
        // 认为这里会复制文本内容
        printf("rawstr:%p newstr:%p\n", rawstr.c_str(), newstr.c_str());

        char *ptoken = strtok(const_cast<char *>(newstr.c_str()), delim.c_str());
        while (ptoken)
        {
            // process ptoken
            ptoken = strtok(NULL, delim.c_str());
        }
    }

    int main(int argc, char **argv)
    {
        string str = "1,2,3,4";
        printf("before:'%s'\n", str.c_str());
        process(str);
        printf("after:'%s'\n", str.c_str());
        return 0;
    }
```

<!-- more -->

按照代码的意图, 从main传入的str应该是不会因为process处理发生变化的, 因为process处理的时候从原数据复制了一份newstr进行处理. 在gcc 4.1.2下编译, 输出结果如下:

``` cpp
before:'1,2,3,4'
rawstr:0x804a014 newstr:0x804a014
after:'1
```

通过运行结果我们可以看到process完后, str的内容的确发生了变化, 根据我们输出的string地址, 发现复制前后的字符串数据指针的地址是相同的. 正是由于string提供的这种copy-on-write机制, 导致strtok处理数据时修改了原始数据内容.
虽然看起来这个问题是 string的实现中提供了cow的特性导致的, 但是根本原因是没有按照函数的参数规范进行编码, 强制将const char*转为char*进行操作, 导致原始数据内容被改写. 对于我们调用的接口(尤其标准库的接口), 如果发现未带有const限定符, 应仔细考虑是否有对输入数据改写的可能, 然后在决定如何传递参数.
