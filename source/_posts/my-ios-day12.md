---
title: my-ios-day12
date: 2016-09-01 16:11:54
categories: iOS
tags: 
- ios
- gitbook
---

这个是16年重新复习的iOS笔记，当时是放在了[gitbook](https://sleenxiu.gitbooks.io/my-learning-ios/content/)，现在转移过来！
<!-- more -->

# day 12
## 事件处理
- 简介
- UIView的触摸事件
- 最合适的View
- 事件响应
- 手势识别
---
#### 事件处理
- iOS中常用的事件分为三种
- 触摸事件
- 加速计事件
- 远程控制事件
- 响应者对象
- 继承自`UIResponder` 的对象称为响应者
- `UIApplication`、`UIViewController`、`UIView`等，他们都是响应者对象，能够接收和处理事件
- `UIResponder`类
- 触摸事件
```objc
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event;
- (void)touchesMoved:(NSSet *)touches withEvent:(UIEvent *)event;
- (void)touchesEnded:(NSSet *)touches withEvent:(UIEvent *)event;
- (void)touchesCancelled:(NSSet *)touches withEvent:(UIEvent *)event;
```
- 加速计事件
```objc
- (void)motionBegan:(UIEventSubtype)motion withEvent:(UIEvent *)event;
- (void)motionEnded:(UIEventSubtype)motion withEvent:(UIEvent *)event;
- (void)motionCancelled:(UIEventSubtype)motion withEvent:(UIEvent *)event;
```
- 远程控制事件
```objc
- (void)remoteControlReceivedWithEvent:(UIEvent *)event;
```
- 事件的产生和传递
- 产生
当发生一个触摸事件后，系统会将该事件加入到一个由UIApplication管理的事件队列中
UIApplication会从事件队列中取出最前面的事件，并将事件分发下去以便处理
- 传递
主窗口会在视图层次结构中找到一个最合适的视图来处理触摸事件
触摸事件的传递是从父控件传递到子控件的
如果一个父控件不能接收事件，那么它里面的子控件也不能够接收事件
- 不能接受事件的情况
1. 不接收用户交互时不能够处理事件
userInteractionEnabled = NO
2. 当一个控件隐藏的时候不能够接收事件
hidden = YES
3. 当一个控件为透明白时候也不能够接收事件
alpha = 0
· 注意`UIImageView`d的`userInteractionEnabled`默认为NO，因此`UIImageView`和它的子控件默认都不能接收事件
---
#### UIView的触摸事件
- 要监听`UIView`的触摸事件，要使用自定义的view并实现`UIResponder`中的相应方法
- 主要事件
```objc
// 一根或者多根手指开始触摸view，系统会自动调用view的下面方法.
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
// 一根或者多根手指在view上移动时，系统会自动调用view的下面方法（随着手指的移动，会持续调用该方法，也就是说这个方法会调用很多次）
- (void)touchesMoved:(NSSet *)touches withEvent:(UIEvent *)event
// 一根或者多根手指离开view，系统会自动调用view的下面方法
- (void)touchesEnded:(NSSet *)touches withEvent:(UIEvent *)event
```
- 参数说明
```objc
touches：
touches中存放的都是UITouch对象，是一个NSSet集合
UITouch对象是用来保存手指相关联的信息，位置、时间、阶段等信息
每一个手指对应一个UITouch对象，这个对象是系统自动为我们创建的，当手指移动时，系统会更新同一个UITouch对象，使它一直保存该手指的信息
通过获取UITouch属性，我们可以获得触摸产生时所处的窗口，触摸的View，时间，点击的次数等
还可以通过UITouch提供的方法获取当前手指所在的点,以及上一个手指所在的点
- (CGPoint)locationInView:(UIView *)view;
- (CGPoint)previousLocationInView:(UIView *)view;
event:
每产生一个事件，就会产生一个UIEvent对象，记录事件产生的时刻和类型
```
- 触摸过程
- 触摸开始
```objc
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event;
```
- 触摸移动
```objc
- (void)touchesMoved:(NSSet *)touches withEvent:(UIEvent *)event;
```
- 触摸结束
```objc
- (void)touchesEnded:(NSSet *)touches withEvent:(UIEvent *)event;
```
- 触摸取消（可能会经历）
```objc
- (void)touchesCancelled:(NSSet *)touches withEvent:(UIEvent *)event;
```
- 注意
一次完整的触摸过程中，只会产生一个事件对象，4个触摸方法都是同一个event参数
如果两根手指同时触摸一个view，那么view只会调用一次touchesBegan:withEvent:方法，touches参数中装着2个UITouch对象
如果这两根手指一前一后分开触摸同一个view，那么view会分别调用2次touchesBegan:withEvent:方法，并且每次调用时的touches参数中只包含一个UITouch对象
---
####最合适的view
- 寻找最合适的view
1. 先判断自己是否可以接收触摸事件，如果可以进行下一步
2. 判断触摸的点是否在自己上
3. 如果在自己上，从后往前遍历子控件，子控件重复上述两步
4. 如果没有符合条件的子控件，那么它自己就是最合适的view
- `hitTest`方法与`pointInside`方法
- 使用
```objc
/**
* 寻找最合适的view
* point：当前手指所在的点
* event：产生的事件
* 返回值：最合适的view
* 调用：只要一个事件传递给一个控件，就会调用这个控件的hitTest方法
*/
- (UIView *)hitTest:(CGPoint) withEvent:(UIEvent *)event;
/**
* 判断point在不在方法的调用者上
* point：必须是调用者的坐标系
* 调用：hitTest底层会调用
*/
- (BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event;
```
- 底层实现
1. 判断当前能否接收事件
```objc
if (self.userInteractionEnabled == NO || self.hidden == YES || self.alpha <= 0.01)
return nil;
```
2. 判断触摸点在不在当前控件上
```objc
if (![self pointInside:point withEvent:event])
return nil;
```
3. 从后往前遍历子控件
```objc
int count = (int)self.subviews.count;
for (int i = count - 1; i >= 0; i--) {
UIView *childView = self.subViews[i];
// 转换坐标系
CGPoint childPoint = [self convertPoint:point toView:childView];
// 判断是否最合适
UIView *fitView = [child hitTest:childPoint withEvent:event];
// 如果最适合，就直接返回
if (fitView) {
return fitView;
}
}
```
4. 自己就是最合适的View
```objc
return self;
```
---
#### 事件响应
- 响应者链条
- 是由多个响应者对象连接起来的链条
- 响应者对象
- 继承了`UIResponder`的对象称之为响应者对象，用来处理事件的对象
- 事件传递
- 在产生一个事件时，系统会将该事件加入到一个由`UIApplication`管理的事件队列中
- `UIApplication`会从事件队列中取出最前面的事件，将它传递给先发送事件给应用程序的主窗口
- 主窗口会调用`hitTest`方法寻找最适合的视图控件，找到后就会调用视图控件的touches方法来做具体的事情
- 当调用touches方法，它的默认做法，就会将事件顺着响应者链条往上传递，传递给上一个响应者，接着就会调用上一个响应者的touches方法
- 上一个响应者
- 如果当前的View是控制器的View，那么控制器就是上一个响应者
- 如果当前的View不是控制器的View，那么它的父控件就是上一个响应者
- 在视图层次结构的最顶级视图，如果也不能处理收到的事件或消息，则其将事件或消息传递给window对象进行处理
- 如果window对象也不处理，则其将事件或消息传递给UIApplication对象
- 如果UIApplication也不能处理该事件或消息，则将其丢弃
---
#### 手势识别
- 手势识别器
- `UIGestureRecognizer`苹果官方推出的手势识别功能
- 可以轻松识别用户在某个View上的常见手势
- 是一个抽象类，定义了所有手势的基本行为，使用其子类就可以处理具体的手势
- 几种常见手势
```objc
UITapGestureRecognizer // 敲击
UIPinchGestureRecognizer // 捏合，用于缩放
UIPanGestureRecognizer // 拖拽
UISwipeGestureRecognizer // 轻扫
UIRotationGestureRecognizer // 旋转
UILongPressGestureRecognizer // 长按
```
- 手势的使用
- 点按手势
```objc
// Target：当哪对象要坚听手势
// action：手势发生时调用的方法
UITapGestureRecognizer *tap = [[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(tap)];
// 手势也可以设置代理
tap.delegate = self;
// 添加手势
[self.imageV addGestureRecognizer:tap];
// 手指开始点击时调用
- (void)tap {
NSLog(@"%s", __func__);
}
// 手势代理方法
- (BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer shouldReceiveTouch:(UITouch *)touch {
// 获取当前手指所在的点
CGPoint curP = [touch locationInView:self.imageV];
}
```
- 长按手势
```objc
UILongPressGestureRecognizer *longP = [[UILongPressGestureRecognizer alloc] initWithTarget:self action:@selector(longPress:)];
// 添加手势
[self.imageV addGestureRecognizer:longP];
// 当手指长按时调用
// 注意，长按手势会调用多次，当开始长按时会调用，当长按松开时会调用，当长按移动时，也会调用
// 一般我们都是在长按刚开始时做事情，所以要判断它的状
// 这个状态是保存的当前的手势当中 所以要把当前的长按手势传进来，来判断当前手势的状态
- (void)longPress:(UILongPressGestureRecognizer *)longP {
// 手势的状态是一个枚举UIGestureRecognizerState，可以进入头文件当中查看
if (longP.state == UIGestureRecognizerStateBegan) {
NSLog(@"开始长按时调用");
}else if (longP.state == UIGestureRecognizerStateChanged) {
// 会持续调用
NSLog(@"当长按拖动时调用");
}else if (longP.state == UIGestureRecognizerStateEnded) {
NSLog(@"当长按松手指松开进调用");
}
}
```
- 轻扫手势
```objc
UISwipeGestureRecognizer *swipe = [[UISwipeGestureRecognizer alloc] initWithTarget:self action:@selector(swipe:)];
// 注意：轻扫手势默认轻扫的方向是往右轻扫，可以去手动修改轻扫的方向
// 一个手势只能对象一个方向，想要支持多个方向的轻扫，要添加多个轻扫手势
swipe.direction = UISwipeGestureRecognizerDirectionLeft;
// 添加手势
[self.imageV addGestureRecognizer:swipe];
```
- 平移手势
```objc
UIPanGestureRecognizer *pan = [[UIPanGestureRecognizer alloc] initWithTarget:self action:@selector(pan:)];
// 添加手势
[self.imageV addGestureRecognizer:pan];
// 实现手势方法
// 手指在屏幕上移动进调用
- (void)pan:(UIPanGestureRecognizer *)pan{
// 获取当前手指移动的偏移量
CGPoint transP = [pan translationInView:self.imageV];
NSLog(@"%@", NSStringFromCGPoint(transP));
// Make它会清空上一次的形变
self.imageV.transform = CGAffineTransformMakeTranslation(transP.x, transP.y);
self.imageV.transform = CGAffineTransformTranslate(self.imageV.transform, transP.x, transP.y);
// 复位，相对于上一次
[pan setTranslation:CGPointZero inView:self.imageV];
}
```
- 旋转手势
```objc
// 添加旋转手势
UIRotationGestureRecognizer *rotation = [[UIRotationGestureRecognizer alloc] initWithTarget:self action:@selector(rotation:)];
// 设置代理，设置代理的目的就让它能够同时支持旋转跟缩放
rotation.delegate = self;
// 添加手势
[self.imageV addGestureRecognizer:rotation];
// 当旋转时调用
- (void)rotation:(UIRotationGestureRecognizer *)rotation {
// 旋转也是相对于上一次
self.imageV.transform = CGAffineTransformRotate(self.imageV.transform, rotation.rotation);
// 设置代理,设置代理的目的就让它能够同时支持旋转跟缩放
rotation.delegate = self;
// 也要做复位操作
rotation.rotation = 0;
}
```
- 缩放手势
```objc
// 添加缩放手势
UIPinchGestureRecognizer *pinch = [[UIPinchGestureRecognizer alloc] initWithTarget:self action:@selector(pinch:)];
// 添加手势
[self.imageV addGestureRecognizer:pinch];
// 缩放手势时调用
- (void)pinch:(UIPinchGestureRecognizer *)pinch {
// 平移也是相对于上一次
self.imageV.transform = CGAffineTransformScale(self.imageV.transform, pinch.scale, pinch.scale);
// 复位
pinch.scale = 1;
}
```

















