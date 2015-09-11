---
layout: post
title: "gmock with const function"
date: 2015-09-11 12:32:00 +0800
comments: true
categories:
---

``` cpp
class SomeClass {
public:
    virtual ~SomeClass() throw() {}
    virtual int get() = 0;
    virtual int get() const = 0;
};

class SomeClassMock : public SomeClass {
public:
    virtual ~SomeClassMock() throw() {}
    MOCK_METHOD0(get, int());
    MOCK_CONST_METHOD0(get, int());
};

TEST(ConstMock, ShouldCallConstFunction) {
    SomeClassMock mock;

    EXPECT_CALL(mock, get())
        .WillRepeatedly(Return(1));
    EXPECT_CALL(const_cast<const SomeClassMock&>(mock), get())
        .WillRepeatedly(Return(2));

    ASSERT_EQ(1, mock.get());
    ASSERT_EQ(2, const_cast<const SomeClassMock&>(mock).get());
```
