---
layout: post
title: "GMOCK下对const函数重载的处理"
date: 2015-09-11 12:32:00 +0800
comments: true
categories:
---

在C++中我们可能会遇到函数名, 函数签名, 返回值都相同的重载, 唯一的区别是const和non-const, 例如:

``` cpp
class SomeClass {
public:
    virtual ~SomeClass() throw() {}
    virtual int get() = 0;
    virtual int get() const = 0;
};
```

这种类我们在使用时编译器会根据我们的传入对象是否带有const属性自动调用对应的版本, 但是通过GMOCK的编写Mock调用时, 很多人对如何设置const函数的EXPECT_ALL和调用产生了疑问.  
其实处理方式也比较简单, 在使用EXPECT_CALL或者调用mock对象的方法时, 我们可以显示通过const_cast将mock对象的转为const版本.  
例如对于上面的类, 我们可以这么去写Mock, 设置期望和调用:

<!--more-->

``` cpp
class SomeClassMock : public SomeClass {
public:
    virtual ~SomeClassMock() throw() {}
    MOCK_METHOD0(get, int());
    MOCK_CONST_METHOD0(get, int());
};
```


``` cpp

TEST(ConstMock, ShouldCallConstFunction) {
    SomeClassMock mock;

    EXPECT_CALL(mock, get())
        .WillRepeatedly(Return(1));
    EXPECT_CALL(const_cast<const SomeClassMock&>(mock), get())
        .WillRepeatedly(Return(2));

    ASSERT_EQ(1, mock.get());
    ASSERT_EQ(2, const_cast<const SomeClassMock&>(mock).get());
```
