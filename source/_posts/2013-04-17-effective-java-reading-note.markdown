---
author: coney
comments: true
date: 2013-04-17 08:30:16+00:00
layout: post
slug: effective-java%e9%98%85%e8%af%bb%e8%ae%b0%e5%bd%95
title: Effective Java阅读记录
wordpress_id: 107
categories:
- Java
---

# 1. 考虑用静态工厂方法代替构造函数
  * 静态工厂方法具有名字, 可以了解被创建对象的特点
  * 静态工厂方法调用后不一定产生新的对象, 例如实现单例或者复用
  * 静态工厂方法可以返回一个子类的对象示例
  <!-- more -->

# 2. 使用私有构造函数强化singleton属性
  * 不直接暴露类成员供访问
  * 通过静态成员函数返回实例提供一定灵活性, 例如修改为per-thread的单例等

# 3. 通过私有构造函数强化不可能实例化的能力
  * 用于实现一些不想实例化的静态类, 例如工具类(java.lang.Math)等.

# 4. 避免创建重复的对象
  * 不要创建重复功能的对象, 而是复用现有功能相同的对象
  * 可以使用静态工厂方法实现一些对象重用的功能, 例如有限个状态的对象(Boolean状态等)

# 5. 消除过期对象的引用
  * Java提供了GC, 但是一些隐式的对象引用可能导致内存泄漏, 例如放在数组中的对象应用, 或者是缓存中对对象的引用
  * 清除对象引用的方法是直接赋值为null

# 6. 避免使用终结函数(finalize)
  * finalizer调用时机不可预测, 且不同JVM调用的时机可能不同
  * 显式的通过try-finally来释放资源, 或者显式提供清理函数
  * 如果一定要override finalize, 一定要记得调用super.finalize()

#  7. override equals是需要遵守通用约定
  * 自反性, 对称性, 传递性, 一致性
  * 非空性, 需要处理传入的null并返回false, 不能抛出NullPointerException
  * 不要替换equals的Object参数类型, 否则将导致overload equals而不是override

# 8. override equals时总是要同时override hashCode
  * 相等的对象必须要有相等的hashCode
  * 如果没有override hashCode会导致该类型无法在HashMap, HashSet等容器中正常工作

# 9. 总是要override toString
  * 提供一种反馈对象包含信息的方法, 用于调试等场景

# 10. 谨慎的改写clone
  * **待详细阅读**

# 11. 考虑实现Comparable接口
  * 使对象可以适用于一些依赖于Comparable的容器或者算法

# 12. 使类和成员的可访问能力最小化
