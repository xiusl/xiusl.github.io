---
title: my-ios-day5
date: 2016-09-01 16:00:39
categories: iOS
tags: 
- ios
- gitbook
---

这个是16年重新复习的iOS笔记，当时是放在了[gitbook](https://sleenxiu.gitbooks.io/my-learning-ios/content/)，现在转移过来！
<!-- more -->

# 回顾
- 封装广告轮播
-
---

# day 05
## 自动布局
#### Autoresizing
- storyboard
- 代码
```objectivec
UIViewAutoresizingFlexibleLeftMargin // 距离父控件的左边是可以伸缩的
UIViewAutoresizingFlexibleBottomMargin // 距离父控件的底部是可以伸缩的
UIViewAutoresizingFlexibleRightMargin // 距离父控件的右边是可以伸缩的
UIViewAutoresizingFlexibleTopMargin // 距离父控件的顶部是可以伸缩的
UIViewAutoresizingFlexibleHeight // 高度会跟随父控件的高度进行伸缩
UIViewAutoresizingFlexibleWidth // 宽度会跟随父控件的宽度进行伸缩
// 设置autoresizingMask属性
redView.autoresizingMask = UIViewAutoresizingFlexibleLeftMargin | UIViewAutoresizingFlexibleTopMargin;
```
- Autoresizing仅能解决父子控件之间的相对关系，比较有局限性

#### Autolayout
- 简介
- 约束：通过给控件添加约束，来决定控件的位置和尺寸
- 参照：在添加约束的时候需要选择参照物，父控件或兄弟控件
- storyboard
- 代码
- 使用NSLayoutConstraint类创建具体的约束对象
- 添加约束到相应的View上
```objectivec
- (void)addConstraint:(NSLayoutConstraint *)constraint; // 添加一个
- (void)addConstraints:(NSArray*)constraints; // 添加多个
```
- 注意点
- 需要先禁止autoresizing功能，view.translatesAutoresizingMaskIntoConstraint = NO;
- 添加约束的时候一定要确定控件已经添加到了父控件上
- 不需要再设置frame值
- 核心计算公式
- obj1.property1 = (obj2.proterty2 * multiplier) + constant value
- 添加约束规则
- 两个同层view(亲兄弟型)之间的约束关系，添加到他们的父view上
- 对于两个不同层view(堂兄弟型)之间的约束关系，添加到他们最近的共同父view上
- 对于有层次关系的view(父子型)之间的约束关系，添加到层次较高的view上

#### VFL语言
- Visual Format Language，可视化格式语言，苹果味监护Autolayout代码而推出的抽象语言
- 简单示例
```
H:[cancelBtn(70)-12-[acceptBtn(50)]] cancelBtn宽70，acceptBtn宽50，之间间距12
V:[redView][yellowView(==redView)] 竖直方向上，先有一个redView，下面有一个和redView等高的yellowView
```
- 使用VFL创建约束数组
+ (NSArray *)constraintsWithVisualFormat:(NSString *)format options:(NSLayoutFormatOptions)opts metrics:(NSDictionary *)metrics views:(NSDictionary *)views;
format：VFL语句，opts：约束类型，metrics：VFL语句中用到的具体数值，views：VFL语句中用到的控件
- 小示例
```objectivec
NSString *vvfl = @"V:[blueView(50)]-space-|";
NSArray *vlcs = [NSLayoutConstraint constraintsWithVisualFormat:vvfl options:kNilOptions metrics:metrics views:views];
[self.view addConstraints:vlcs];
```

#### Masonry框架


---

## 自动布局小例子













