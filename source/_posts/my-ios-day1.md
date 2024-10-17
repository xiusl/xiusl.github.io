---
title: my-ios-day1
date: 2016-09-01 16:00:27
categories: iOS
tags: 
- ios
- gitbook
---

这个是16年重新复习的iOS笔记，当时是放在了[gitbook](https://sleenxiu.gitbooks.io/my-learning-ios/content/)，现在转移过来！
<!-- more -->

#day01
## iOS简单路线
* UI基础与进阶
* 网络和多线程
* 简单项目开发
* 实用技术
* h5跨平台
* Swift学习

---
## UIView
### 简单介绍
* 一般在屏幕上看到的都是UIView，可以把它看做控件或视图
* 控件包含一些共同的属性，他们都在UIView中定义
* 尺寸
* 位置
* 背景颜色
* 所有控件的父控件都是UIView
* 在开发中，我们也会将一些共同的属性抽取到一个父类中
* 苹果就将控件的一些共同属性抽取在UIView类中
---
### 常见属性
```objc
@property(nonatomic,readonly) UIView *superview; // 获得自己的父控件对象
@property(nonatomic,readonly,copy) NSArray *subviews; // 获得自己的所有子控件对象
@property(nonatomic) NSInteger tag; // 控件的ID(标识)，父控件可以通过tag来找到对应的子控件
@property(nonatomic) CGAffineTransform transform; // 控件的形变属性(可以设置旋转角度、比例缩放、平移等属性)
```
### 常见方法
```objectivec
- (void)addSubview:(UIView *)view; // 添加一个子控件view
- (void)removeFromSuperview; // 将自己从父控件中移除
- (UIView *)viewWithTag:(NSInteger)tag; // 根据一个tag标识找出对应的控件（一般都是子控件）
```
### 关于尺寸
```objc
// 控件矩形框在父控件中的位置和尺寸(以父控件的左上角为坐标原点)
@property(nonatomic) CGRect frame;
// 控件矩形框的位置和尺寸(以自己左上角为坐标原点，所以bounds的x、y一般为0)
@property(nonatomic) CGRect bounds;
// 控件中点的位置(以父控件的左上角为坐标原点)
@property(nonatomic) CGPoint center;
```
##加法计算器



