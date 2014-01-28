---
author: coney
comments: true
date: 2013-03-17 16:39:38+00:00
layout: post
slug: '%e8%ae%be%e8%ae%a1%e6%a8%a1%e5%bc%8f%e4%b8%80%e5%8f%a5%e8%af%9d%e6%80%bb%e7%bb%93'
title: 设计模式一句话总结
wordpress_id: 44
categories:
- OOP
---

可能理解有偏差, 望指正, 勿参考



	
  1. Factory: 将创建者和调用者分开(创建者, 调用者)

	
  2. Factory Method: 创建者的创建行为交由子类完成

	
  3. Abstract Method: 将有共性的类型的创建行为抽象出来

	
  4. Builder: 将复杂对象的组装过程与调用者分离(创建者, 组装者, 调用者)

	
  5. Prototype: 基于现有对象复制修改

	
  6. Singleton: 全局唯一实例

	
  7. Facade: 提供对外调用接口

	
  8. Adapter: 兼用不同的接口

	
  9. Proxy: 接口调用增加一个代理层, 不改变原有逻辑的情况下加入额外调用

	
  10. Decorator: 动态增加原有接口的功能, 并能自由组合(使用组合扩展而不是继承)

	
  11. Bridge: 将抽象和实现分开(作为参数传入?)

	
  12. Composite: 类的组合关系(包含)

	
  13. Flyweight: 复用已有的重复对象(线程池, 连接池)

	
  14. Template: 定义框架, 将行为延迟到子类中实现

	
  15. Observer: 将一个对象的行为通知到多个对象(回调注册)

	
  16. State: 根据不同的状态实现不同的行为

	
  17. Strategy: 抽象行为, 根据场景使用(ca)

	
  18. Chain of Responsibility: 使多个对象都有机会处理请求(按需处理)

	
  19. Command: 将操作抽象, 独立于调用者和接收者

	
  20. Visitor: 不改变元素和类的情况下提供对元素的新操作

	
  21. Mediator: 将各个对象解耦, 将多对象间的行为关联

	
  22. Memento: 备份恢复类对象(有啥用? 相对于Prototype)

	
  23. Iterator: 不暴露内部接口的情况下提供对内部元素的访问方法(对比Visitor)

	
  24. Interpreter: 对语言进行解析, 抽象语法树(毛用?)



