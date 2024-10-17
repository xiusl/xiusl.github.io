---
title: block & delegate & notification 特点和区别
date: 2019-04-02 14:22:15
categories:
- technology
tags:
- iOS 
- 模拟器
---

iOS 中三种回调的特点、区别和使用场景的总结。

<!-- more -->

#### 特点和区别

- block 简单易用，可以直接访问上下文的变量，不需要额外存储临时变量，代码阅读连续。
- delegate 比较复杂，需要声明和实现接口，而且两者不在同一文件中，需要跳转文件，还可能需要存储临时变量。
- notification 较为复杂，需要定义通知类型，同样通知的发送和接收不在一个文件中。

#### 问题

- block 虽然简单，但会存在循环引用的问题。
- delegate 需要对方法进行检验 `respondsToSelector:`。
- notification 需要注意他的多对多特性，通知的移除（iOS 9 之后不需要？）。
- delegate 的运行成本低，block 的运行成本高，block 在出栈的时候需要将使用的数据从栈区拷贝到堆区，而 delegate 只是保存一个指针而已。


#### 场景

- 当只有1、2个回调时使用 block，3个以上的话使用 delegate。
- 避免循环引用的时候使用 delegate。
- 根据回调的性质来选择
  - 回调不定期触发，或者会多次触发，delegate 更加合适。
  - 回调是一次性的，单线性（一对一这种）的关系，block 更加合适。
  - 回调是广播性质的，需要多个类同时收到，notification 更加合适。

#### 总结

- 通过不同执行线（一对一，一对多）、不同的执行次数、不同的执行数量来判断使用哪一种回调。

#### 参考

> https://www.zhihu.com/question/29023547
> https://blog.csdn.net/u010670946/article/details/71340711
