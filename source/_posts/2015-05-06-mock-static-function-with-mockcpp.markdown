---
layout: post
title: "使用Mockcpp Mock C++静态函数"
date: 2015-05-06 15:22:10 +0800
comments: true
categories: c++ mock mockcpp gmock
---

传统测试中的Mock, 都是基于多态实现的, 也就是Mock面向接口的虚函数. 但是在C++的代码中, 经常会混入大量的C函数或是静态成员函数.
例如工厂函数, 单例函数, 或是C库中的函数甚至STL的算法等.

对于这些静态函数, 比较传统的做法是创建一个Wrapper, 用虚方法对这些静态函数进行包裹. 在测试的时候对Wrapper进行Mock便可控制被包裹的静态函数的行为:

``` cpp
int add(int x, int y);
```
可以通过Wrapper包裹为:

``` cpp
class Calc {
public:
  virtual int add(int x, int y) {
    return ::add(x, y);
  }
}
```

但是对于存量代码, 这种重构并不现实(工作量及流程问题). 正当我们束手无策时, 我们发现mockcpp可以帮助我们解决一部分静态方法Mock的需求.

<!--more-->

### Mock静态函数

虽然mockcpp主要提供面向虚方法的Mock, 但是mockcpp同时通过inline hooking提供了对静态方法的Mock, 更为强大的是它的inline hooking支持32位和64位的Windows及Linux环境.

抛开实现, 我们来看两个mockcpp的应用实例:

``` cpp
// Mock C Function
static int add(int x, int y) {
    return x + y;
}

TEST(Mockcpp, ShouldAbleToMockCFunction) {
    MOCKER(add)
        .expects(once())
        .with(eq(1), eq(2))
        .will(returnValue(0));
    ASSERT_EQ(0, add(1, 2));
}

```

``` cpp
// Mock Static Member Function
class Calc {
  class Calc {
public:
    static int add(int x, int y) {
        return x + y;
    }
};

TEST(Mockcpp, ShouldAbleToMockStaticFunction) {
    MOCKER(&Calc::add)
        .expects(once())
        .with(eq(1), eq(2))
        .will(returnValue(0));
    ASSERT_EQ(0, Calc::add(1, 2));
}
```

mockcpp通过`MOCKER`宏改变了C函数`add`及静态成员函数`Calc::add`的行为. 并且对这两个函数的调用设置了期望.
对于mockcpp的详细用法可以查看这个文档: [mockcpp使用方法简明指导](https://code.google.com/p/mockcpp/wiki/SimpleUserInstruction_zh)

### Mockcpp对静态函数的副作用

前面提到, mockcpp对静态函数的Mock是通过inline hooking实现的. 我们来简单看下这个inline hook的过程.

首先是`int add(int, int)`在x64下生成的汇编指令:

``` objdump
Dump of assembler code for function _ZL3addii:
   0x00000000004060a4 <+0>:     push   %rbp
   0x00000000004060a5 <+1>:     mov    %rsp,%rbp
   0x00000000004060a8 <+4>:     mov    %edi,-0x4(%rbp)
   0x00000000004060ab <+7>:     mov    %esi,-0x8(%rbp)
   0x00000000004060ae <+10>:    mov    -0x8(%rbp),%eax
   0x00000000004060b1 <+13>:    mov    -0x4(%rbp),%edx
   0x00000000004060b4 <+16>:    add    %edx,%eax
   0x00000000004060b6 <+18>:    pop    %rbp
   0x00000000004060b7 <+19>:    retq
End of assembler dump.
```
通过mockcpp Mock之后, 开头的一段指令让逻辑跳转到了`<_ZN38Mockcpp_ShouldAbleToMockCFunction_Test8TestBodyEv+53>`:

``` objdump
Dump of assembler code for function _ZL3addii:
   0x00000000004060a4 <+0>:     jmpq   *0x0(%rip)        # 0x4060aa <_ZL3addii+6>
   0x00000000004060aa <+6>:     sahf
   0x00000000004060ab <+7>:     js     0x4060ed <_ZN38Mockcpp_ShouldAbleToMockCFunction_Test8TestBodyEv+53>
   0x00000000004060ad <+9>:     add    %al,(%rax)
   0x00000000004060af <+11>:    add    %al,(%rax)
   0x00000000004060b1 <+13>:    add    %dl,-0x4(%rbp)
   0x00000000004060b4 <+16>:    add    %edx,%eax
   0x00000000004060b6 <+18>:    pop    %rbp
   0x00000000004060b7 <+19>:    retq
End of assembler dump.
```
而跳转到的`<_ZN38Mockcpp_ShouldAbleToMockCFunction_Test8TestBodyEv+53>`正是我们通过mockcpp设置的expects:

``` objdump
Dump of assembler code for function _ZN38Mockcpp_ShouldAbleToMockCFunction_Test8TestBodyEv:
   ...
   0x00000000004060ed <+53>:    lea    -0x60(%rbp),%rax
   0x00000000004060f1 <+57>:    mov    %rax,%rdi
   0x00000000004060f4 <+60>:    callq  0x449396 <_ZN7mockcpp11returnValueERKNS_3AnyE>
   ...
End of assembler dump.
```
然后问题来了, mockcpp的inline hooking只会对原函数做一次, 如果发现原来的函数已经被hook过, 就会忽略后续的MOCKER.
这会导致:
* 被Mock的函数在脱离MOCKER作用域后依旧生效
* 被Mock的函数后续不能继续通过MOCKER更改行为

对于第一个问题, 暂时没有发现什么比较好的方法, mockcpp在做inline hooking的时候已经破坏了函数入口的汇编逻辑.

而对于第二种情况, 我么则可以通过一种proxy的方法, 通过mockcpp将静态函数转发到某个被Mock的成员函数上, 然后再通过控制这个成员函数改变原来静态函数的行为.
具体的实现可以参考下面一节的示例.

### 使用Google Mock语法Mock静态函数

mockcpp虽然能够对静态函数进行Mock, 但mockcpp出现的年代比较久远, 限于当时的编译器能力, 语法不是很友好.

在目前的环境中, Google Mock无疑是一个更好的选择(编译器允许的话也可以尝试FakeIt). 但是Google Mock并不具备对静态方法进行Mock的能力.

通过mockcpp做为proxy, 我们可以在不学习新的mockcpp语法的情况下对静态函数进行Mock(同时解决了mockcpp不能多次Mock同一个静态函数的问题).

我们先来看一个示例:

``` cpp

class Calc {
public:
    static int add(int x, int y) {
        return x + y;
    }
};

class CalcMock {
public:
    static CalcMock &getInstance() {
        static CalcMock calcMock;
        return calcMock;
    }

    MOCK_METHOD2(add, int(int, int));
    static int addProxy(int x, int y) {
        return getInstance().add(x, y);
    }
};

TEST(Mockcpp, ShouldAbleToMockStaticFunctionWithGoogleMock) {
    MOCKER(Calc::add)
        .defaults()
        .with(any(), any())
        .will(invoke(CalcMock::addProxy));

    EXPECT_CALL(CalcMock::getInstance(), add(1, 2))
        .WillRepeatedly(testing::Return(0));
    ASSERT_EQ(0, Calc::add(1, 2));


    EXPECT_CALL(CalcMock::getInstance(), add(1, 2))
        .WillRepeatedly(testing::Return(10));
    ASSERT_EQ(10, Calc::add(1, 2));
}
```

我们在Mock `Calc::add`方法时, 首先使用Mockcpp将所有`Calc:add`的调用转发到`CalcMock::addProxy`上.
之后在`CalcMock::addProxy`里则会调用`CalcMock`单例的`CalcMock::add`方法, 而这个`CalcMock::add`方法则是通过Google Mock声明的.
可以使用`EXPECT_CALL`及其他Google Mock支持的语法对Mock的`Calc::add`设置期望.

此时我们已经可以脱离Mockcpp, 使用更加友好的Google Mock来控制静态函数.

在这个示例中, 我们同时解决了上一节提到的Mockcpp不能多次对同一静态函数Mock的问题.
我们使用Mockcpp一劳永逸的将静态函数转发给了Google Mock, 而Google Mock则可以对Mock对象的单例任意设置期望.
