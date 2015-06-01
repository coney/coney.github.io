---
layout: post
title: "禁止Visual Assist自动添加Override关键字"
date: 2015-06-01 14:32:21 +0800
comments: true
categories: c++ visual assist
---
## 问题描述
Visual Assist是一款历史悠久的Visual Studio插件, 能够极大的提升C++开发效率. 但是某些时候智能过了头, 反倒成了开发效率的绊脚石, 例如在实现虚方法(Implement Virtual Methods)时自动添加的`override`关键字.

``` cpp
class SomeInterface {
public:
	virtual void doSomething() = 0
};
```

使用Implement Virtual Methods后, Visual Assist帮我们生成如下代码:
``` cpp
class SomeImplementation : public SomeInterface {
public:
    virtual void doSomething() override
    {
        throw std::logic_error("The method or operation is not implemented.");
    }
};
```

Visual Assist在`virtual void doSomething()`后面帮我们添加了一个`override`关键字, 这个关键是符合c++11标准的. 但是一些老旧的编译器并不支持`override`, 因此导致编译错误. 这使我们不得不手动删除所有生成方法后的`override`关键字.

<!--more-->

## 解决方案
Visual Assist没有直接提供选项来禁用`override`关键字, 但是在其官方论坛咨询后得到如下链接:

[Add the override identifier to Implement Interface](http://docs.wholetomato.com/default.asp?W346)

文章提到在2036之后的版本可以通过注册表来禁用`override`关键字:

``` text
In Visual Assist build 2036 and older, add the "override" identifier in C++ commands by editing the following registry binary value:

HKCU\Software\Whole Tomato\Visual Assist X\<IDE spec>\UseOverrideKeywordInImplementInterface = 01

Set the value to 00 to return to the default behavior.
```

将`UseOverrideKeywordInImplementInterface`的值改为0, 便可以禁用自动添加`override`的功能, 注册表路径中的`<IDE spec>`表示的是要应用的Visual Studio版本,  例如在Visual Studio 2013中, 对应的字段是`VANet12`.

修改注册表前一定要记得首先关闭Visual Studio.

如果你怕麻烦, 也可以下载并运行这个文件来修改注册表(适用于Visual Studio 2013):

[disable-override-keyword-for-vs2013.reg](/assets/resources/disable-override-keyword-for-vs2013.reg)
