---
layout: post
title: "C++ 源码阅读项目推荐"
date: 2015-08-06 10:25:47 +0800
comments: true
categories:
---
### Chromium
- 流行浏览器的Chrome的开发版本
- 源码符合Google C++ 编程风格规范[Google C++ Style Guide](http://google.github.io/styleguide/cppguide.html)
- 较多人进行阅读, 有一些现有阅读笔记[Chrome源码剖析](http://www.cnblogs.com/duguguiyu/archive/2008/10/02/1303095.html)
- 能够学习到比较全面各种子系统的设计知识, 如网络, 多线程, IO, 内存管理等
- 其项目中包含了若干Google的子项目, 也很值得阅读, 例如: libv8, gtest, gflags
- 源码地址: https://code.google.com/p/chromium/

### STL
- C++标准模板库, 日常开发必备
- 通过源码能够学习到模板编程基础, 相对于Boost来说可读性很高, 上手容易
- 通过学习STL实现能够更好的掌握STL的开发范式, 例如通过迭代器编程而不是传递容器, 使用或扩展STL中的算法
- 通过了解STL常用容器的底层实现也能更好的让我们再不同场景下选择合适的容器, 避免因误用造成时间空间的浪费
- STL实现版本较多, 内部接口的编程风格不是很好, 建议通过SGI STL的源码进行阅读
- 源码地址: https://www.sgi.com/tech/stl/


### Poco
- 大而全的实用类库, 提供类似于JDK或者.net frameworks的功能, 是stl/boost的强大补充
- 通过Poco能够找到大量可以直接应用于项目的组件和库, 即使我们不打算直接使用, 也可以借鉴其架构设计
- 设计和编码风格良好, 代码规范偏Java, 同样与M2000的编程风格较为接近
- 源码地址: http://pocoproject.org/

### MFC & Qt
- 传统老牌的大型C++框架, 提供UI能力
- 类似Poco, 通过阅读源码能够对我们未来开发自己的模块和框架提供参考和指导
- 忽略掉MFC中类的C前缀和Qt中类的Q前缀, 其编码规范还是较为良好
- 完整的框架结构和继承体系, 有利于我们了解大型SDK的组织结构
- Qt源码地址: https://wiki.qt.io/Get_the_Source
- MFC源码安装完Visual Studio即可获得, 源码使用方法: https://msdn.microsoft.com/en-us/library/bs046sh0.aspx

### ACE
- 高性能分布式框架
- 从中可以学习到一些网络和IO设计模型, 其设计思想即使对于了解其他语言的类似Java的BIO, NIO, AIO也很有帮助
- 代码阅读难度稍大, 工程也较为复杂, 上手难度大
- 源码地址: https://github.com/DOCGroup/ATCD
