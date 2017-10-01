---
layout: post
title: "handle ambiguous overloaded functions in gmock"
date: 2015-08-21 15:44:24 +0800
comments: true
categories:
---

在声明的接口中, 可能会存在返回值及函数参数个数一直的情况, 例如下面`Inf::f()`方法:

``` c++
#include "gmock/gmock.h"

class Inf {
public:
    virtual ~Inf() throw() {}
    virtual void f(const ::std::string &name, ::std::string &value) = 0;
    virtual void f(const ::std::string &name, long &value) = 0;
};
```

调用时传入对应类型的参数可以匹配到对应签名的方法, 对其编写Mock也没有任何问题:

``` cpp
class InfMock {
public:
    virtual ~InfMock() throw() {}
    MOCK_CONST_METHOD2(f, void(const ::std::string &name, ::std::string &value));
    MOCK_CONST_METHOD2(f, void(const ::std::string &name, long &value));
};
```

但是实际调用时, 向第二个有歧义的参数设置值时, EXPECT_CALL接受Mock方法时通过Any是没有办法区分出期望调用的是哪个函数, 此时可以通过`::testing::An`来传入具体类型以匹配对应签名的函数:

``` cpp
TEST(GMock, ShouldAbleToIdentifyOverloadedFunctions) {
    InfMock infmock;
    EXPECT_CALL(infmock, f("hello", ::testing::An<::std::string &>()))
        .WillOnce(::testing::SetArgReferee<1>("9527"));
    EXPECT_CALL(infmock, f("world", ::testing::An<long &>()))
        .WillOnce(::testing::SetArgReferee<1>(134));

    ::std::string strVal;
    infmock.f("hello", strVal);
    ASSERT_EQ("9527", strVal);

    long longVal;
    infmock.f("world", longVal);
    ASSERT_EQ(134, longVal);
}
```
