---
title: my-ios-day13
date: 2016-09-01 16:11:57
categories: iOS
tags: 
- ios
- gitbook
---

这个是16年重新复习的iOS笔记，当时是放在了[gitbook](https://sleenxiu.gitbooks.io/my-learning-ios/content/)，现在转移过来！
<!-- more -->

# day 13
## Quartz2D
- 简介
- 基本线条绘制
- 小实例
- 应用
---
#### 简介
- Quartz2D
- 一个同时支持Mac和iOS的二维绘图引擎
- 画基本线条、绘制文字、图片、截图、自定义`UIView`
- 当所需控件样式及其复杂时，可以将控件内部的结构绘制出来，就是自定义控件
- 图形上下文
- 用来保存用户绘制的内容状态，并决定绘制到哪里
- 用户会把需要绘制的内容保存到图像上下文，根据选择的图形上下文，显示到不同的地方，即输出的目标不同
- 常见上下文
Bitmap Graphics Context 位图上下文
PDF Graphics Context PDF上下文
Window Graphics Context 窗口上下文
Layer Graphics Context 图层上下文
Printer Graphics Context 打印上下文
- 自定义`UIView`
1. 创建一个继承`UIView`的`SLView`
2. 实现`DrawRect`方法
3. 在`DrawRect`方法中取得与View相关的上下文
4. 绘制路径
5. 把描述好的路径保存到上下文中
6. 把上下文中的内容渲染到View上
---
#### 基本线条绘制
- `DrawRect`方法
- 在这给方法中完成绘图操作，只有这个方法才能拿到与View关联的上下文
- 当View显示时，系统自动调用
- 画直线
- 基本过程
```objc
// 获取与view关联的上下文
CGContextRef ctx = UIGraphicsGetCurrentContext();
// 绘制路径
UIBezierPath *path = [UIBeizerPath beizerPath];
// 设置起点
[path moveToPoint:CGPointMake(50, 50)];
// 添加一根线到一个点
[path addLineToPoint:CGPointMake(200, 100)];
// 把路径添加到上下文
CGContextAddPath(ctx, path.CGPath);
// 在view上渲染
CGContextStrokePath(ctx);
```
- 再添加一条线
- 第一种：重新设置起点，添加一根线到一个点，一个`UIBezierPath`路径可以有多条线
- 第二种：直接在原来的基础上，把上一条线的终点作为起点，直接`addLineToPoint:`
- 线的属性<br/>
设置这个属性，就是修改图形上下文的状态
设置线宽：CGContextSetLineWidth(ctx, 20);
设置线段的连接样式：CGContextSetLineJoin(ctx, kCGLineJoinRound);
添加顶角样式：CGContextSetLineCap(ctx, kCGLineCapRound);
设置线的颜色：[[UIColor redColor] setStroke];
- 画曲线
- 画曲线方法比较特殊，需要一个控制点来决定曲线的弯曲程度
```objc
// 先设置一个曲线的起点
[path moveToPoint:CGPointMake(30, 125)];
// 再添加到个点到曲线的终点，同时还须要一个controlPoint(控制点决定曲线弯曲)
[path addQuadCurveToPoint:CGPointMake(200, 120) controlPoint:CGPointMake(120, 40)];
```
- 画矩形（圆角）
- 画矩形直接利用UIBezierPath给我们封装好的路径方法
```objc
// (x, y)点决定了矩形左上角的点在哪个位置
// (width, height)是矩形的宽度高度
bezierPathWithOvalInRect:CGRectMake(x, y, width, height)
```
- 圆角矩形的画法多了一个参数`cornerRadius`
```objc
// cornerRadius它是矩形的圆角半径.
// 通过圆角矩形可以画一个圆，当矩形是正方形的时候，把圆角半径设为宽度的一半，就是一个圆
bezierPathWithRoundedRect:CGRectMake(10, 100, 50, 50) cornerRadius:10
```
- 画椭圆（圆）
- 方法
```objc
// 前两个参数分别代码圆的圆心，后面两个参数分别代表圆的宽度与高度
// 宽高都相等时，画的是一个正圆，不相等时画的是一个椭圆
bezierPathWithOvalInRect:CGRectMake(10, 100, 50, 50)
```
- 画圆弧
- 首先要确定圆才能确定圆弧，圆孤就是圆的一部分
```objc
// center：圆心
// radius：圆的半径
// startAngle：起始角度
// endAngle：终点角度
// clockwise：Yes顺时针，No逆时针
// 注意：startAngle角度的位置是从圆的最右侧为0度
UIBezierPath *path = [UIBezierPath bezierPathWithArcCenter:CGPointMake(125, 125) radius:100 startAngle:0 endAngle:M_PI * 2 clockwise:YES];
```
- UIKit的封装
- `[path stroke]`方法直接使用
- 底层：获取上下文，拼接路径，添加路径，渲染
---
#### 小实例
- 下载进度条
- 搭建界面
- 拖动滑块时，圆弧的大小也改变，进度数字也实时变（显示%需要转义%%）
- 相关代码
```objc
// 从最上面，按顺时针画，它的起始角度是-90度，结束角度也是-90度
// 起始角度-90度，看你下载进度是多少，假如说你下载进度是100，就是1 * 360度
CGContextRef ctx = UIGraphicsGetCurrentContext();
CGPoint center = CGPointMake(50, 50);
CGFloat radius = rect.size.width * 0.5;
CGFloat startA = -M_PI_2;
CGFloat endA = -M_PI_2 + M_PI * 2 * progress;
UIBezierPath *path = [UIBezierPath bezierPathWithArcCenter:center radius:radius startAngle:startA endAngle:endA clockwise:YES];
要获得Progress的值，拖线拿到它的该变量
要在值改变的时候就要传进来，并且每次值改变的时候就要去绘制圆弧
重写`setProgress:`方法
- (void)setProgress:(CGFloat)progress{
_progress = progress;
// 手动调用drawRect方法，让它重新绘制
[self drawRect:self.bounds];
}
```
- 运行后不绘制
- 原因：DrawRect方法是不能手动调用，因为在DrawRect方法中需要获取跟View相关联的上下文。系统在调用DrawRect方法时，会自动创建一个跟View相关联的上下文，并且传递给它。自己调用，没有给DrawRect方法传递上下文，所以在DrawRect方法中没有上下文
- 解决：想要重绘，调用`[self setNeedsDisplay]`，告诉系统重新绘制View，系统就会自动调用DrawRect方法，这样就可以获得上下文进行绘制
- 饼状图
- 方法
```objc
// 获取上下文
CGContextRef ctx = UIGraphicsGetCurrentContext();
// 基本数据
CGPoint center = CGPointMake(125, 125);
CGFloat radius = 100;
CGFloat startA = 0;
CGFloat endA = 0;
CGFloat angle = 25 / 100.0 * M_PI * 2;
endA = startA + angle;
// 拼接路经
UIBezierPath *path = [UIBezierPath bezierPathWithArcCenter:center radius:radius startAngle:startA endAngle:endA clockwise:YES];
// 添加一条线到圆心
[path addLineToPoint:center];
// 把路径添加到上下文
CGContextAddPath(ctx, path.CGPath);
// 把上下文渲染到View
CGContextFillPath(ctx);
// 第二个扇形
startA = endA;
angle = 25 / 100.0 * M_PI * 2;
endA = startA + angle;
path = [UIBezierPath bezierPathWithArcCenter:center radius:radius startAngle:startA endAngle:endA clockwise:YES];
[path addLineToPoint:center];
// 把二个路径添加到上下文
CGContextAddPath(ctx, path.CGPath);
// 把上下文渲染到View
CGContextFillPath(ctx);
// 添加第二个扇形之后，发现它们的颜色都一样，想要修改它的颜色，在下面再写一个
[[UIColor greenColor] set];
```
- 第三个扇形的基本方法与上面基本相同，发现代码重复较多，进行抽取
- 抽取后
```objc
// 假设给一组数据
NSArray datas = @[@25, @25, @50];
CGPoint center = CGPointMake(125, 125);
CGFloat radius = 100;
CGFloat startA = 0;
CGFloat angle = 0;
CGFloat endA = 0;
for (NSNumber *number in datas) {
startA = endA;
angle = number.intValue / 100.0 * M_PI * 2;
endA = startA + angle;
// 描述路径
UIBezierPath *path = [UIBezierPath bezierPathWithArcCenter:center radius:radius startAngle:startA endAngle:endA clockwise:YES];
// 添加直线
[path addLineToPoint:center];
// 随机颜色
[[self randomColor] set];
// 闭合
[path fill];
}
// 随机返回颜色，开发时可定义为宏
- (UIColor *)randomColor {
CGFloat r = arc4random_uniform(256)/ 255.0;
CGFloat g = arc4random_uniform(256)/ 255.0;
CGFloat b = arc4random_uniform(256)/ 255.0;
return [UIColor colorWithRed:r green:g blue:b alpha:1];
}
```
- UIKit绘图
- 绘制文字
```objc
// 先创建好要画的文字
NSString *str = @"Sleen";
// drawAtPoint：要画到哪个位置
// withAttributes：文本的样式.
[str drawAtPoint:CGPointZero withAttributes:nil];
```
- 属性设置
```objc
// 通过绘制方法的最后一个属性withAttributes来设置文字属性
// 创建一个可变的字典,设置key,value
NSMutableDictionary *dict = [NSMueDictionary dictionary];
// 字体
dict[NSFontAttributeName] = [UIFont sys tOfSize:50];
// 颜色
dict[NSForegroundColorAttributeName] = [UIColor red ;
// 设置边框颜色
dict[NSStrokeColorAttributeName] = [UIColor redCo;
dict[NSStrokeWidthAttributeName] = @1;
// 阴影
NSShadow *shadow = [[NSShadow alloc] init];
shadow.shadowOffset = CGSizeMake(10, 10);
shadow.shadowColor = [UIColor greenColor];
shadow.shadowBlurRadius = 3;
dict[NSShadowAttributeName] = shadow;
```
- 绘制图片
- 绘制图片同样开始要先把图片素材导入
- AtPoint：参数说明图片要绘制到哪个位置
- 通过调用UIKit的方法`drawAtPoint:CGPointZero`方法进行绘制
- 两个方法
- 文字
1. `drawAtPoint:` 不能换行
2. `drawInRect:` 能够换行
- 图片
1. `drawAtPoint:` 按照图片尺寸绘制
2. `drawInRect:` 图片尺寸与Rect的尺寸一样
- 平铺图片
```objc
[image drawAsPatternInRect:rect];
```
- 快速矩形
```objc
UIRectFill(rect);
```
- 裁剪
```objc
// 这个方法必须要设置好裁剪区域，才能有裁剪效果
UIRectClip(CGRectMake(0, 0, 50, 50));
```
- 仿UIImageView
- 思路
- 分析系统UIImageView的功能
1. 创建UIImageView对象，设置frame，设置image，添加到view上
2. 创建时就设置image，frame与图片尺寸相同
- 实现
1. 新建一个UIView，如`SLImageView`
2. 给`SLImageView`添加一个UIImage属性
3. 在`DrawRect`方法中绘制图片到view上
4. 重写UIImage的set方法，实现切换图片
5. 提供`- (instancetype)initWithImage:(UIImage *)image;`方法
- 代码
```objc
- (instancetype)initWithImage:(UIImage *)image {
if (self = [super init]) {
self.frame = CGRectMake(0, 0, image.size.width, image.size.height);
_image = image;
}
return self;
}
- (void)setImage:(UIImage *)image {
_image = image;
[self setNeedsDisplay];
}
- (void)drawRect:(CGRect)rect {
[_image drawInRect:rect];
}
```
- 雪花
- 思路
1. 首先绘制一个雪花
2. 在View加载完毕添加一个定时器
3. 在定时器中调用重绘方法
4. 在重绘方法中不断修改雪花的y值
5. 当y值超过屏幕，就重设为0
- 定时器
1. NSTime和CADisplayLink
2. `setNeedsDisplay`方法底层会调用`DrawRect`，但是它不是立马就进行重绘，它仅仅是设置了一个重绘标志，等到下一次屏幕刷新的时候才会调用`DrawRect`方法
3. NSTime的时间可能和屏幕的刷新时间不匹配，就会产生延迟，造成界面卡顿，CADisplayLink的时间正好与屏幕刷新时间相同
4. 使用CADisplayLink添加定时器
```objc
// target：哪个对象要监听方法
// selector：监听的方法名称
CADisplayLink *link = [CADisplayLink displayLinkWithTarget:self selector:@selector(setNeedsDisplay)];
// 想要让CADisplayLink工作，必须得要把它添加到主运行循环
// 只要添加到主运行循环，跟模式没有关系
[link addToRunLoop:[NSRunLoop mainRunLoop] forMode:NSDefaultRunLoopMode];
```
- 代码
```objc
- (void)awakeFromNib {
CADisplayLink *link = [CADisplayLink displayLinkWithTarget:self selector:@selector(setNeedsDisplay)];
[link addToRunLoop:[NSRunLoop mainRunLoop] forMode:NSDefaultRunLoopMode];
}
- (void)drawRect:(CGRect)rect {
if (_snowY > rect.size.height) {
_snowY = 0;
}
UIImage *image = [UIImage imageNamed:@"雪花"];
[image drawAtPoint:CGPointMake(0, _snowY)];
_snowY += 10;
}

```
- 水印
- 用处
- 标明图片来源（微博）
- 防止盗图（淘宝）
- 思路
1. 开启一个和原始图片一样的图片上下文
2. 把原始图片绘制到上下文
3. 再把要添加的水印（文字，logo）等绘制到图片上下文
4. 最后从上下文中取出一张图片
5. 关闭上下文
- 主要代码
```objc
// 开启一个图片上下文?
// size：开启多大的上文
// opaque：不透明度
// scale：缩放上下文.
UIGraphicsBeginImageContextWithOptions(image.size, YES, 0);
// 从图片上下文当中生成一张图片
UIImage *newImage = UIGraphicsGetImageFromCurrentImageContext();
// 关闭上下文
UIGraphicsEndImageContext();
```
- 圆形头像（裁剪）
- 思路
1. 开启一个图片上下文
2. 上下文的大小和原始图片保持一样，以免图片被拉伸缩放
3. 在上下文的上面添加一个圆形裁剪区域，圆形裁剪区域的半径大小和图片的宽度一样大
4. 把要裁剪的图片绘制到图片上下文当中
5. 从上下文当中取出图片
6. 关闭上下文
- 主要代码
```objc
// 设置圆形路径
UIBezierPath *path = [UIBezierPath bezierPathWithOvalInRect:CGRectMake(0, 0, image.size.width, image.size.width)];
// 把一个路径设为裁剪区域
[path addClip];
```
- 截屏
- 思路
- 将UIView上的东西保存到图片上下文中，生成一张新图片
- UIView神的东西是不能直接保存的，需要的是view内部的layer层
- 调用layer的`renderInContext:`方法
- 代码
```objc
// 开启一个图片上
UIGraphicsBeginImageContextWithOptions(self.view.bounds.size, NO, 0);
// 获取当前的上下文
CGContextRef ctx = UIGraphicsGetCurrentContext();
// 把控制器View的内容绘制上下文当中
[self.view.layer renderInContext:ctx];
// 从上下文当中取出图片
UIImage *newImage = UIGraptImageFromCurrentImageContext();
// 关闭上下文
UIGraphicsEndImageContext();
```
- 图片擦除
- 思路
1. 两张不同的图片，上面一张，下面一张
2. 添加手势，手指移动擦除图片
3. 确定擦除区域大小和位置
4. 生成新的图片（擦除区域透明），就可以看到后面的图片
- 代码
```objc
// 确定擦除的范围
CGFloat rectWH = 30;
// 获取手指的当前点.curP
CGPoint curP = [pan locationInView:pan.view];
CGFloat x = curP.x - rectWH * 0.5;
CGFloat y = curP.y - rectWH * 0.5;
CGRect rect = CGRectMake(x, y,rectWH, rectWH);
// 先把图片绘制到上下文.
UIGraphicsBeginImageContextWithOptions(self.imageView.bounds.size, NO, 0);
// 获取当前的上下文
CGContextRef ctx = UIGraphicsGetCurrentContext();
// 把上面一张图片绘制到上下文.
[self.imageView.layer renderInContext:ctx];
// 再绘上下文当中图片进行擦除.
CGContextClearRect(ctx, rect);
// 生成一张新图片
UIImage *newImage = UIGraphicsGetImageFromCurrentImageContext();
// 再把新的图片给重新负值
self.imageView.image = newImage;
// 关闭上下文.
UIGraphicsEndImageContext();
```
- 图片截屏
- 思路
1. 手指在屏幕上移动的时候，添加一个半透明的view
2. 开启一个上下文把UIView的farme设置为裁剪区域
3. 把view的layer显示的内容绘制到上下文当中
4. 生成一张新的图片在赋值给UIImageView
- 具体实现
1. 给图片添加一个手势，监听手指在图片上的拖动（注意UIImageView默认不能接收事件）
2. 监听手指移动，手指移动时添加一个UIView，x,y就是起始点，手指开始的点，width是x轴的偏移量，height是y轴偏移量
3. 计算代码
```objc
CGFloat offSetX = curP.x - self.beginP.x;
CGFloat offsetY = curP.y - self.beginP.y;
CGRect rect = CGRectMake(self.beginP.x, self.beginP.y, offSetX, offsetY);
```
4. UIView只需要添加一次，给UIView设置成懒加载，保证只有一个UIView，在移动时修改frame就好
5. 开启一个图片上下文，图片上下文的大小为位原始图片尺寸，是整个图片可以截取
```objc
// 利用UIBezierPath设置一个矩形的裁剪区域
// 然后把这个路径设置为裁剪区域
// 把路径设为裁剪区域的方法为
[path addClip];
```
6. 把图片绘制到上下文中
```objc
// 由于是一个UIImageView上面的图片，所以也得需要渲染到上下文当中
// 要先获取当前的上下文
// 把UIImageView的layer渲染到当前的上下文当中
CGContextRef ctx = UIGraphicsGetCurrentContext();
[self.imageV.layer renderInContext:ctx];
```
7. 取出新的图片，重新赋值图片
```objc
UIImage *newImage = UIGraphicsGetImageFromCurrentImageContext();
self.imageV.image = newImage;
```
8. 关闭上下文,移除上面半透明的UIView
```objc
UIGraphicsEndImageContext();
[self.coverView removeFromSuperview];
```
- 手势解锁
- 分析界面
- 当手指在上面移动时，移动到按钮的一定范围中，会把按钮变成选中状态
- 并把第一个选中的按钮中心作为一条线的起点，手指移到下一个按钮就会添加一条线
- 手指松开后就会取消选中，清空所有线
- 思路
1. 先判断当前手指在不在当前按钮上，如果在按钮上，就把按钮设置成选中状态
2. 把当前选中的按钮添加到一个数组当中，如果当前按钮已经是选中状态，就不需要再添加到数组中了
3. 每次移动都让它重绘，在绘图中，遍历所有选中按钮
4. 判断数组当中的第一个无素，如果是第一个，那么就把它设为路径的起点，其它都在添加一根线到按钮的圆心
5. 如果当前点不在按钮上，那么就记录住当前手指所在的点，直接从起点添加一根线到当前手指所在的点
- 具体实现
1. 搭建界面
界面是一个九宫格的布局.九宫格实现思路.
先确定有多少列 cloum = 3;
计算出每列之间的距离
计算为: CGFloat margin = (当前View的宽度 - 列数 * 按钮的宽度) / 总列数 + 1
每一列的X的值与它当前所在的行有关
当前所在的列为:curColum = i % cloum
每一行的Y的值与它当前所在的行有关.
当前所在的行为:curRow = i / cloum
每一个按钮的X值为, margin + 当前所在的列 * (按钮的宽度 + 每个按钮之间的间距)
每一个按钮的Y值为 当前所在的行 * (按钮的宽度 + 每个按钮之间的距离)
具体代码
```objc
// 总列数
int colum = 3;
// 每个按钮的宽高
CGFloat btnWH = 74;
// 每个按钮之间的距离
CGFloat margin = (self.bounds.size.width - colum * btnWH) (colum + 1);
for(int i = 0; i < self.subviews.count; i++ ){
// 当前所在的列
int curColum = i % colum;
// 当前所在的行
int curRow = i / colum;
CGFloat x = margin + (btnWH + margin) * curColum;
CGFloat y = (btnWH + margin) * curRow;
// 取出所有的子控件
UIButton *btn = self.subviews[i];
btn.frame = CGRectMake(x, y, btnWH, btnWH);
}
```
2. 监听手指在上面的操作
在手指点击屏幕时，如果当前手指所在的点在按钮上，那就让按钮变成选中状态
判断一个点在不在一个区域，在返回YES，不在NO
CGRectContainsPoint(btn.frame, point)
当手指点击屏幕的时候，需要做的事情：
(1) 获取当前手指所在的点
UITouch *touch = [touches anyObject];
GPoint curP = [touch locationInView:self];
(2) 判断当前点在不在按钮上
for (UIButton *btn in self.subviews) {
if (CGRectContainsPoint(btn.frame, point)) {
return btn;
}
}
(3) 如果当前点在按钮上，并且当前按钮不是选中的状态
那么把当前的按钮成为选中状态
并且把当前的按钮添加到数组当中
3. 在绘图方法中
创建路径
遍历出有的选中按钮，如果是第一个按钮，把第一个按钮的中心点当做是路径的起点
其它按钮都直接添加一条线，到该按钮的中心
遍历完所有的选中按钮后
最后添加一条线到当前手指所在的点
- 画板
- 分析
顶部是一个工具栏。有清屏、撤销、橡皮擦、照片功能，最右部是一个保存按钮
中间部分为画板区域
最下部拖动滑块能够改变画笔的粗线，可以选颜色
- 界面搭建
最上部为一个ToolBar，往ToolBar拖些item，使用ToolBar的好处，里面按钮的位置不需要我们再去管理
给最上部的工具栏做自动布局，离父控件左、上、右都为0，保存工具条的高度不变
拖一个UIView当前下部的View，在下部的View当中拖累三个按钮，设置每一个按钮的背景颜色，点击每一按钮时办到设置画笔的颜色
其中三个按钮只间的间距始终保存等，每一个按钮的宽度和高度都相等，通过自动布局的方式办到
先把这个UIView的自动布局设好，让其左、右、下都是0，高度固定
自动布局设置为：第一个按钮高度固定，与左、右、下都保持20间距
第二个按钮与第一个按钮，高度、宽度、centerY都相等，与右边有20间距
第三个按钮也是第一个按钮的高度、宽度、centerY都相等，与右边有20间距，最右边也保持20间距
最后是中间的画板区域，画板区域只需要距离上、下、左、右都为0即可
- 画板功能
1. 监听手指在屏幕上的状态，在开始点击屏幕的时候，创建一个路径，并把手指当前的点设为路径的起点
2. 设置一个成员属性记录当前绘制的路径，并把当前的路径添加到路径数组当中
3. 当手指在移动的时候，用当前的路径添加一根线到当前手指所在的点，然后进行重绘
4. 在绘图方法当中，取出所有的路径，把所有的路径给绘制出来
- 小工具
- 清屏
删除所有路径，进行重绘
- 撤销
删除最后一条路径，进行重绘
- 线宽
由于每一条线宽度都不样，所以要在开始创建路径的时，就要添加一个成员属性，设置一个默认值
在把当前路径添加到路径数组之前，设置好线的宽度，然后重写线宽属性方法
下一次再创建路径时，线的宽度就是当前设置的宽度
- 颜色
每一条线的颜色也不一样，也需要一个属性记录每一条路径的颜色
UIBezierPath没有给我们直接提供设置颜色的属性，需要自定义一个UIBezierPath，创建一个MyBezierPath类，继承UIBezierPath，在该类中添加一个颜色的属性
在创建路径的时候，直接使用自己定义的路径，设置路径默认的一个颜色，方法给设置线宽一样
在绘图过程中，取出来的都是MyBezierPath，把MyBezierPath的颜色设置为路径的颜色
- 橡皮擦功能
路径的颜色设为白色
- 保存功能
1. 开启一个跟View相同大小的图片上下文
2. 把View的layer上面内容渲染到上下文当中
3. 生成一张图片，把图片保存到上下文
```objc
// 第一个参数：要写入到相册的图片
// 第二个参数：哪个对象坚听写入完成时的状态
// 第三个参数：图片保存完成时调用的方法
UIImageWriteToSavedPhotosAlbum(newImage, self, @selector(image:didFinishSavingWithError: contextInfo:), nil);
// 注意:图片保存完成时调用的方法必须得是image:didFinishSavingWithError: contextInfo:
```
- 选择图片
1. 弹出系统相册
```objc
// 使用UIImagePickerController控件器Modal出它来.
UIImagePickerController *pick = [[UIImagePickerController alloc] init];
// 设置照片的来源
pick.sourceType = UIImagePickerControllerSourceTypeSavedPhotosAlbum;
// 设置代码，监听选择图片，UIImagePickerController比较特殊，它需要遵守两个协议<UINavigationControllerDelegate, UIImagePickerControllerDelegate>
pick.delegate = self;
// modal出控件器
[self presentViewController:pick animated:YES completion:nil];
// 注意没有实现代码方法时，选择一张照片会自动的dismiss掉相册控制器，但是设置代码后，就得要自己去dismiss了
```
2. 实现代理，取得照片
```objc
// 选择的照片就在这个方法第二个参数当中, 它是一个字典
- (void)imagePickerController:(UIImagePickerController *)picker didFinishPickingMediaWithInfo:(nonnull NSDictionary<NSString *,id> *)info {
// 获取当前选中的图片,通过UIImagePickerControllerOriginalImage就能获取
UIImage *image = info[UIImagePickerControllerOriginalImage];
}
```
3. 得到照片的布局
获取完图片后，图片是能够缩放、平移的，因此获取完图片后，是在画板板View上面添加了一个UIView，只有UIView才能做平移、缩放、旋转等操作
因此做法为，在图片选择完毕后，添加一个和画板View相同大小的UIView，这个UIView内部有一个UIImageView
对这个UIImageView进行一些手势操作，操作完成时，长按图片，把View的内容截屏，生成一张图片，把新生成的图片绘制到画板上面
- 绘制图片
在画板View当中提供一个UImage属性，供外界传递，重写属性的set方法，每次传递图片时进行重绘
画图片也有有序的，所以要把图片也添加到路径数组当中
在绘图片过过程当中，如果发现取出来的是一个图片类型，那就直接图片绘制到上下文当中
具体代码
```objc
- (void)drawRect:(CGRect)rect{
for (DrawPath *path in self.pathArray) {
if ([path isKindOfClass:[UIImage class]]) {
UIImage *image = (UIImage *)path;
[image drawInRect:rect];
}else {
[path.lineColor set];
[path stroke];
}
}
}
```



















