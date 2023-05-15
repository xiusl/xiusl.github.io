---
title: my-ios-day14
date: 2016-09-01 16:11:59
categories: 
- 技术
- Objective-C 
tags: 
- ios
- gitbook
---

这个是16年重新复习的iOS笔记，当时是放在了[gitbook](https://sleenxiu.gitbooks.io/my-learning-ios/content/)，现在转移过来！
<!-- more -->

# day 14
## 核心动画
- CALayer
- 核心动画
---
#### CALayer
- 简介
CALayer有称它为“层”
每一个UIView内部都有一个layer这样的属性
UIView之所以可以显示，就是因为它里面有这样一个层，才具有显示功能
我们通过操作CALayer对象，可以方便的调整UIView的外观属性
可以给UIView设置阴影、圆角、边框等
- 操作layer
- 设置阴影
```objc
// 默认图层是有阴影的，只不过是透明的
_redView.layer.shadowOPacity = 1;
// 设置阴影的圆角
_redView.layer.shadowRadius = 10;
// 设置阴影的颜色
_redView.layer.shadowColor = [UIColor blueColor].CGColor;
```
- 设置边框
```objc
// 设置图层边框，在图层中使用CoreGraphics的CGColoerRef
_redView.layer.borderColor = [UIColor whiteColor].CGColor;
_redView.layer.borderWidth = 2;
```
- 设置圆角
```objc
// 图层的圆角半径，若为宽度一般就是一个圆
_redView.layer.cornerRadius = 50;
```
- layer改变`UIImageView`
- 设置边框
```objc
_imageView.layer.borderWidth = 2;
_imageView.layer.borderColor = [UIColor whiteColor].CGColor;
```
- 图片圆角
```objc
// 半径
_imageView.layer.cornerRadius = 50;
// 裁剪
_imageView.layer.masksToBounds = YES;
```
注意：UIImageView当中Image并不是直接添加在图层上，而是在layer的contents里。设置的属性只是作用在层上，如果不裁剪的话是不能看见图片的圆角的
- layer的`CATransform3D`属性
- 只有旋转的时候在会看到3D效果
```objc
// x, y, z分别代表x, y, z轴
// 旋转
CATransform3DMakeRotation(M_PI, 1, 0, 0);
// 平移
CATransform3DMakeTranslation(x, y, z);
// 缩放
CATransform3DMakeScale(x, y, z);
```
- 可以通过KVC的方式进行设置属性
- CATransform3DMake*的值是一个结构体，要把结构体转换成对象
```objc
NSValue *value = [NSValue valueWithCATransform3D:CATransform3DMakeRotation(M_PI, 1, 0, 0)];
[_imageView.layer setValue:value forKeyPath:@"transform.scale"];
```
- 何时使用
当需要进行快速的缩放、平移、旋转的时候使用KVC
如[_imageView.layer setValue:@0.5 forKeyPath:@"transform.scale"];
后面的forKeyPath不是乱写的，苹果文档有给出
- 自定义UILayer
- 与在定义UIView相似
```objc
CALayer *layer = [CALayer layer];
layer.frame = CGRectMake(50, 50, 100, 100);
layer.backgroundColor = [UIColor redColor].CGColor;
// 给Layer添加图片
layer.contents = (id)[UIImage imageNamed:@"xixi"].CGImage;
```
- 关于CALayer
为什么要使用CGImageRef，CGColorRef？
保证了可移植性，QuartzCore不能使用UIImage、UIColor，只能用CGImageRef，CGColorRef
UIView和CALayer都能够显示东西，该怎样选择?
对比CALayer，UIView多了一个事件处理的功能。也就是说，CALayer不能处理用户的触摸事件，而UIView可以
如果显示出来的东西需要跟用户进行交互的话，用UIView；
如果不需要跟用户进行交互，用UIView或者CALayer都可以，CALayer的性能会高一些，因为它少了事件处理的功能，更加轻量级
- position和anchorPoint属性
- 是Layer的两个属性
- 可以通过这两个属性修改控件的位置，他们两个是配合使用的
posrtion：
它是用来设置当前控件在父控件中的位置的
它的坐标原点是父控件的左上角为(0, 0)点
anchorPoint：
它是决定CALayer身上哪一个点会在postion属性
所指的位置，它以当前图层左上角为原点(0, 0)
取值范围0~1，默认在中间位置(0.5, 0.5)位置
又称锚点，就是把锚点定在position的位置
- 两者结合使用
- 设置它的position点
- layer身上的anchorPoint会自动定在position所在的位置
- 隐式动画
- 两个概念
- 根层：UIView内部关联的哪个layer称为根层
- 非根层：自己手动创建的layer就是非根层
- 在对非根层修改时，会自动产生一些动画效果，为隐式动画
- 取消隐式动画
- 事务：很多操作绑定在一起，这些操作按顺序执行，
- 动画底层就是包装成事务进行的
```objc
// 开启事务
[CATransaction begin];
// 设置事务没有动画
[CATransaction setDisableActions:YES];
// 设置动画执行的时长
[CATransaction setAnimationDuration:2];
// 提交事务
[CATransaction commit];
```
---
#### 核心动画
- 简介
- Core Animation，非常强大的动画处理API。通过很少的代码完成功能很炫的效果
- 同时可以应用于Mac OS X和iOS
- 所有动画都是在后台执行，不会阻塞主线程
- 直接作用于CALayer层，非UIView
- 使用步骤
1. 首先要有一个layer
2. 初始化一个CAAnimation对象，并设置相应的属性
3. 通过调用CALayer的`addAnimation: forKey:`方法，添加CAAnimation对象到CALayer中，
4. 调用CALayer的`removeAnimation: forKey:`方法，停止CALayer中的动画
- 属性说明
```
duration：动画的持续时间
repeatCount：重复次数，无限循环可以设置HUGE_VALF或者MAXFLOAT
repeatDuration：重复时间
removedOnCompletion：默认为YES，代表动画执行完毕后就从图层上 移除，图形会恢复到动画执行前的状态。如果想让图层保持显示动画 执行后的状态，那就设置为NO，不过还要设置fillMode为kCAFillMod eForwards
fillMode：决定当前对象在非active时间段的行为。比如动画开始之 前或者动画结束之后
beginTime：可以用来设置动画延迟执行时间，若想延迟2s，就设置为 CACurrentMediaTime()+2，CACurrentMediaTime()为图层的当前时间
timingFunction：速度控制函数，控制动画运行的节奏
delegate：动画代理
```
- 动画填充模式
- fillMode属性值（要想fillMode有效，最好设置`removedOnCompletion = NO`）
```
kCAFillModeRemoved 这个是默认值，也就是说当动画开始前和动画结束后，动画对layer都没有影响，动画结束后，layer会恢复到之前的状态
kCAFillModeForwards 当动画结束后，layer会一直保持着动画最后的状态
kCAFillModeBackwards 在动画开始前，只需要将动画加入了一个layer，layer便立即进入动画的初始状态并等待动画开始。
kCAFillModeBoth 这个其实就是上面两个的合成.动画加入后开始之前，layer便处于动画初始状态，动画结束后layer保持动画最后的状态
```
- 速度控制函数
- 速度控制函数`CAMediaTimingFunction`
```
kCAMediaTimingFunctionLinear（线性）：匀速，给你一个相对静态的感觉
kCAMediaTimingFunctionEaseIn（渐进）：动画缓慢进入，然后加速离开
kCAMediaTimingFunctionEaseOut（渐出）：动画全速进入，然后减速的到达目的地
kCAMediaTimingFunctionEaseInEaseOut（渐进渐出）：动画缓慢的进入，中间加速，然后减速的到达目的地。这个是默认的动画行为。
```
- 动画的代理
- CAAnimation在分类中定义了代理方法
```objc
@interface NSObject (CAAnimationDelegate)
// 动画开始的时候调用
- (void)animationDidStart:(CAAnimation *)anim;
// 动画完成，移除时调用
- (void)animationDidStop:(CAAnimation *)anim finished:(BOOL)flag;
@end
```
- 动画的暂停和恢复
- 暂停
```objc
- (void)pauseLayer:(CALayer *)layer {
CFTimeInterval pausedTime = [layer convertTime:CACurrentMediaTime() fromLayer:nil];
// 让CALayer的时间停止走动
layer.speed = 0.0;
// 让CALayer的时间停留在pausedTime这个时刻
layer.timeOffset = pausedTime;
}
```
- 恢复
```objc
- (void)resumeLayer:(CALayer *)layer {
CFTimeInterval pausedTime = layer.timeOffset;
// 1. 让CALayer的时间继续行走
layer.speed = 1.0;
// 2. 取消上次记录的停留时刻
layer.timeOffset = 0.0;
// 3. 取消上次设置的时间
layer.beginTime = 0.0;
// 4. 计算暂停的时间(这里也可以用CACurrentMediaTime()-pausedTime)
CFTimeInterval timeSincePause = [layer convertTime:CACurrentMediaTime() fromLayer:nil] - pausedTime;
// 5. 设置相对于父坐标系的开始时间(往后退timeSincePause)
layer.beginTime = timeSincePause;
}
```
- 基础动画`CABasicAnimation`
- `CAPropertyAnimation`的子类
- 属性说明
```
fromValue：keyPath相应属性的初始值
toValue：keyPath相应属性的结束值
```
- 过程
- 随着动画的进行，在长度为duration的持续时间内，keyPath相应属性的值从fromValue渐渐地变为toValue
- keyPath内容是CALayer的可动画Animatable属性
- 如果fillMode=kCAFillModeForwards同时removedOnComletion=NO，那么在动画执行完毕后，图层会保持显示动画执行后的状态。但在实质上，图层的属性值还是动画执行前的初始值，并没有真正被改变。
- 关键帧动画`CAKeyframeAnimation`
- `CAPropertyAnimation`的子类
- 区别：
- `CABasicAnimation`只能从一个数值（fromValue）变到另一个数值（toValue）
- `CAKeyframeAnimation`会使用一个NSArray保存这些数值
- 属性说明
```objc
values：上述的NSArray对象。里面的元素称为“关键帧”(keyframe)。动画对象会在指定的时间（duration）内，依次显示values数组中的每一个关键帧
path：可以设置一个CGPathRef、CGMutablePathRef，让图层按照路径轨迹移动。path只对CALayer的anchorPoint和position起作用。如果设置了path，那么values将被忽略
keyTimes：可以为对应的关键帧指定对应的时间点，其取值范围为0到1.0，keyTimes中的每一个时间值都对应values中的每一帧。如果没有设置keyTimes，各个关键帧的时间是平分的
```
- 组动画`CAAnimationGroup`
- `CAAnimation`的子类
- 可以保存一组动画对象，将`CAAnimationGroup`对象加入层后，组中所有动画对象可以同时并发运行

- 属性说明
```objc
animations：用来保存一组动画对象的NSArray
默认情况下，一组动画对象是同时运行的，也可以通过设置动画对象的beginTime属性来更改动画的开始时间
```
- 转场动画`CATransition`
- `CAAnimation`的子类
- 能够为层提供移出屏幕和移入屏幕的动画效果。iOS比Mac OS X的转场动画效果少一点
- 属性说明
```objc
type：动画过渡类型
subtype：动画过渡方向
startProgress：动画起点(在整体动画的百分比)
endProgress：动画终点(在整体动画的百分比)
```
- 应用
```objc
/**
* UIView单视图转场动画函数
* duration：动画的持续时间
* view：需要进行转场动画的视图
* options：转场动画的类型
* animations：将改变视图属性的代码放在这个block中
* completion：动画结束后，会自动调用这个block
*/
+ (void)transitionWithView:(UIView *)view duration:(NSTimeInterval)duration options:(UIViewAnimationOptions)options animations:(void (^)(void))animations completion:(void (^)(BOOL finished))completion;
/**
* UIView双视图转场动画函数
* duration：动画的持续时间
* options：转场动画的类型
* animations：将改变视图属性的代码放在这个block中
* completion：动画结束后，会自动调用这个block
*/
+ (void)transitionFromView:(UIView *)fromView toView:(UIView *)toView duration:(NSTimeInterval)duration options:(UIViewAnimationOptions)options completion:(void (^)(BOOL finished))completion;
```
---
#### 小实例
- 钟表效果
- 分析
界面上时针、分针、秒针不需要与用户进行交互，所以都可以使用layer方式来做
表针是根据当前的时间，绕着表盘的中心点进行旋转
无论是旋转，缩放它都是绕着锚点进行的
要想让时针、分针、称针显示的中间，还要绕着中心点进行旋转，那就要设置它的position和anchorPoint两个属性.
- 实现
- 秒针
```objc
// 创建秒针
CALayer *layer = [CALayer layer];
_secLayer = layer;
layer.bounds = CGRectMake(0, 0, 1, 80);
layer.anchorPoint = CGPointMake(0.5, 1);
layer.position = CGPake(_clockView.bounds.size.width * 0.5, _clew.bounds.size.height * 0.5);
layer.backgroundColor = [UIColor redColor].CGColor;
[_clockView.layer addSublayer:layer];
```
```objc
让秒针旋转，所以要计算当前的旋转度是多少?
当前的旋转角度为：当前的时间 * 每秒旋转多少度
计算每一秒旋转多少度
60 秒转一圈360度
360 除以60就是每一秒转多少度，每秒转6度
获取当前的时间
创建日历类
NSCalendar *calendar = [NSCalendar currentCalendar];
把日历类转换成一个日期组件
日期组件（年、月、日、时、分、秒）
component:日期组件有哪些东西组成,他是一个枚举,里面有年月日时分秒
fromDate:当前的日期
NSDateComponents *cmp = [calendar components:NSCalendarUnitSecond fromDate:[NSDate date]];
我们的秒就是保存在日期组件里面，它里面提供了很多get方法。
NSInteger second = cmp.second;
那么当前秒针旋转的角度就是
当前的秒数乘以每秒转多少度
econd * perSecA
还得要把角度转换成弧度
因为下面分针，时针也得要用到，就把它抽出一个速参数的宏。
#define angle2Rad(angle) ((angle) / 180.0 * M_PI)
```
```objc
// 让它每隔一秒旋转一次.所以添加一个定时器.
// 每个一秒就调用,旋转秒针
- (void)timeChange {
// 获取当前的秒数
// 创建日历类
NSCalendar *calendar = [NSCalendar currentCalendar];
// 把日历类转换成一个日期组件
// 日期组件(年、月、日、时、分、秒)
// component:日期组件有哪些东西组成，他是一个枚举，里面有年月日时分秒
// fromDate:当前的日期
NSDateComponents *cmp = [calendar components:NSCalendarUnitSecond fromDate:[NSDate date]];
// 我们的秒就是保存在日期组件里面，它里面提供了很多get方法
NSInteger second = cmp.second;
// 秒针旋转多少度
CGFloat angel = angle2Rad(second * perSecA);
// 旋转秒针
self.secondL.transform = CATransform3DMakeRotation(angel, 0, 0, 1);
}
// 运行发现他会一下就跳到某一个时间才开始旋转
// 一开始的时候就要来到这个方法，获取当前的秒数把它定位好
// 要在添加定时器之后就调用一次timeChange方法
```
- 分针
```objc
代码基本相同，添加一个分针成员属性
修改宽度，修改颜色，让它旋转
要算出每分钟转多少度
转60分钟刚好是一圈，所以每一分钟也是转6度
获取当前多少分?
同样是在日期组件里面获得
里面有左移符号,右移符号.他就可以用一个并运算
现在同时让他支持秒数和分 后面直接加上一个 |
NSDateComponents *cmp = [calendar components:NSCalendarUnitSecond | NSCalendarUnitMinute fromDate:[NSDate date]];
CGFloat minueteAngel = angle2Rad(minute * perMinuteA);
self.minueL.transform = CATransform3DMakeRotation(minueteAngel, 0, 0, 1);
```
- 时针
```objc
小时转多少度
当前是多少小时，再计算先每一小时转多少度
12个小时转一圈，360除以12，每小时转30度
时针旋转多少度
CGFloat hourAngel = angle2Rad(hour * perHourA);
旋转时针
self.hourL.transform = CATransform3DMakeRotation(hourAngel, 0, 0, 1);
```
直接这样写会有问题
就是每转一分钟，小时也会移动一点点
接下来要算出，每一分钟，小时要转多少度
60分钟一小时，一小时转30度
30 除以60，就是每一分钟，时针转多少度0.5
```objc
// 时针旋转多少度
CGFloat hourAngel = angle2Rad(hour * perHourA + minute * perMinuteHourA);
// 旋转时针
self.hourL.transform = CATransform3DMakeRotation(hourAngel, 0, 0, 1);
```
- 心跳效果
- 思路：就是让一张图片做一个放大缩小的持续动画
- 代码实现
```objc
CABasicAnimation *anim =[CABasicAnimation animation];
// 设置缩放属性
anim.keyPath = @"transform.scale";
// 缩放到最小
anim.toValue = @0;
// 设置动画执行的次数
anim.repeatCount = MAXFLOAT;
// 设置动画执行的时长
anim.duration = 0.25;
// 设置动画自动反转(怎么去, 怎么回)
anim.autoreverses = YES;
// 添加动画
[self.heartView.layer addAnimation:anim forKey:nil];
```
- 图片抖动
- 思路：其实就是做一个左右旋转的动画。先让它往左边旋转-5，再往右边旋转5度，再从5度旋转到-5度，就会有左右摇摆的效果了。
- 实现
```objc
// 创建帧动画
CAKeyframeAnimation *anim = [CAKeyframeAnimation animation];
// 设置动画属性为旋转
anim.keyPath = @"transform.rotation";
// 设置属性值为多个属性
anim.values = @[@(angle2radio(-5)), @(angle2radio(5)), @(angle2radio(-5))];
// 设置动画执行次数
anim.repeatCount = MAXFLOAT;
// 添加动画
[_imageView.layer addAnimation:anim forKey:nil];
```
- 转盘
- 界面
把转盘View给封装起来，界面是固定不变的，可以使用一个Xib展示界面
外界使用时直接来一个类方法直接调用
- 旋转
内部提供开始和结束旋转，供外界调用
- 按钮
按钮是倾斜的，排列在一周，先将所有按钮添加到最上方，让按钮绕着圆盘中心进行旋转
锚点设置为按钮底部中心(0.5, 1)，按钮的position为圆盘中心
添加按钮时，让每一个按钮进行旋转，计算出每一个按钮旋转的角度：总共有12个按钮,正好占圆的一圈，那每一个按钮占30
一个按钮在上一个按钮的基础上加上30，即可算出每一个按钮旋转的角度
- 点击事件
- 在添加按钮时，给每个按钮加选中状态（默认UIImageView不能交互，手动设置）
- 点击后按钮称为选中，上一个按钮取消选中状态
- 按钮图片
- 美工图片为一整张图片，需要自己裁剪，确定裁剪的宽高
- 使用下面方法裁剪
```objc
// image:为要裁剪的图片，既原始图片。它的类型为CGImageRef类型，所以要转成CGImage
// rect:为裁剪的范围.这时需要确定每一个X的位置
// 每一个x为它当前的角标 * 要裁剪的宽度
CGImageCreateWithImageInRect(image, rect);
// 这个方法它会返回一个图片.图片类型为CGImageRef imgR.
// 所要我们在设置背景图片的时候，要把这张图片给转成UIImage，转的方法为:
UIImage *image = [UIImage imageWithCGImage:imgR];
```
- 问题
```
CGImageCreateWithImageInRect方法为C语言方法，
裁剪的时候是按像素，而iOS是使用点，需要乘以像素比例
[UIScreen mainScreen].scale
```
- 中间按钮
- 逻辑：快速的让圆盘旋转几圈，动画完成时，让选择的按钮指上最上方，使用核心动画
- 在动画结束后获得旋转角度
```objc
CGFloat angel = atan2(self.selectBtn.transform.b, self.selectBtn.transform.a);
- (void)animationDidStop:(nonnull CAAnimation *)anim finished:(BOOL)flag {
CGAffineTransform transform = self.selectBtn.transform;
CGFloat angel = atan2(self.selectBtn.transform.b, self.selectBtn.transform.a);
self.innerCircle.transform = CGAffineTransformMakeRotation(-angel);
}
```













