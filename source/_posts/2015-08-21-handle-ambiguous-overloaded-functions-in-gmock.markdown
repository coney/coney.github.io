---
layout: post
title: "handle ambiguous overloaded functions in gmock"
date: 2015-08-21 15:44:24 +0800
comments: true
categories:
---

``` c++
#include "gmock/gmock.h"

class Inf {
public:
    virtual ~Inf() throw() {}
    virtual void f(const ::std::string &name, ::std::string &value) = 0;
    virtual void f(const ::std::string &name, long &value) = 0;
};

class InfMock {
public:
    virtual ~InfMock() throw() {}
    MOCK_CONST_METHOD2(f, void(const ::std::string &name, ::std::string &value));
    MOCK_CONST_METHOD2(f, void(const ::std::string &name, long &value));
};


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
