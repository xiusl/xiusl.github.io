---
title: my-ios-day2
date: 2016-09-01 16:00:30
categories: iOS
tags: 
- ios
- gitbook
---

这个是16年重新复习的iOS笔记，当时是放在了[gitbook](https://sleenxiu.gitbooks.io/my-learning-ios/content/)，现在转移过来！
<!-- more -->

##回顾
##加法计算器
- 控件
- 3个UILabel
- 2个UITextField
- 1个UIButton
- 简单逻辑
- 取得用户在textField中的输入，判断输入是否满足需求（数字和非空）
- 计算结果并显示
---
# day 02
## 基本控件
- UILabel 显示文本
- UIImageView 显示图片
- UIButton 按钮
- UITextField 文本输入框
- UITextView 带滚动条的文本输入框
- UIProgressView 进度条
- UISlider 滑块
- UIActivityIndicator 圈圈
- UIAlertView 中间弹框
- UIActionSheet 底部弹框
- UIScrollView 滚动的view
- UIPageControl 分页控件
- UITableView 表格
- UICollectionView 九宫格
- UIWebView 显示网页
- UISwitch 开关
- UISegmentControl 选项卡
- UIPickView 选择器
- UIDatePicker 日期选择器
- UIToolbar 工具条
---
## 控件学习
#### UILabel
- 经常使用的控件，显示文字
- 常见属性
```objectivec
@property(nullable, nonatomic,copy) NSString *text; // 显示的文字
@property(null_resettable, nonatomic,strong) UIFont *font; // 字体
@property(null_resettable, nonatomic,strong) UIColor *textColor; // 文字的颜色
@property(nullable, nonatomic,strong) UIColor *shadowColor; // 文字的阴影颜色
@property(nonatomic) CGSize shadowOffset; // 文字的阴影偏移
@property(nonatomic) NSTextAlignment textAlignment; // 文字对方式
@property(nonatomic) NSLineBreakMode lineBreakMode; // 换行模式
@property(nonatomic) NSInteger numberOfLines; // 文字行数，0表示换行
```
- UIColor类
- 表示颜色
- 常用方法
```objectivec
+ (UIColor *)blackColor; // 0.0 white
+ (UIColor *)whiteColor; // 1.0 white
+ (UIColor *)grayColor; // 0.5 white
+ (UIColor *)redColor; // 1.0, 0.0, 0.0 RGB
+ (UIColor *)greenColor; // 0.0, 1.0, 0.0 RGB
+ (UIColor *)blueColor; // 0.0, 0.0, 1.0 RGB
+ (UIColor *)clearColor; // 0.0 white, 0.0 alpha
- (UIColor *)initWithRed:(CGFloat)red green:(CGFloat)green blue:(CGFloat)blue alpha:(CGFloat)alpha; // 使用aRGB初始化颜色
- (UIColor *)initWithPatternImage:(UIImage*)image; // 图片中获取均色
```
- UIFont类
- 表示字体
- 常用方法
```objectivec
+ (UIFont *)systemFontOfSize:(CGFloat)fontSize; // 系统默认字体
+ (UIFont *)boldSystemFontOfSize:(CGFloat)fontSize; // 粗体
+ (UIFont *)italicSystemFontOfSize:(CGFloat)fontSize; // 斜体
```
- shadowOffset
- 阴影CGSizeMake(x, y);
- x>0 向右偏 y>0 向下偏
- textAlignment
```
- NSTextAlignmentLeft 左对齐
- NSTextAlignmentCenter 居中
- NSTextAlignmentRight 右对齐
```
- lineBreakMode
```
- NSLineBreakByWordWrapping = 0, 按单词换行，默认的
- NSLineBreakByCharWrapping, 按字母换行
- NSLineBreakByTruncatingHead, 开始省略"...wxyz"
- NSLineBreakByTruncatingTail, 结尾省略 "abcd..."
- NSLineBreakByTruncatingMiddle, 中间省略 "ab...yz"
```
---
#### UIImageView
- 常用控件，显示图片，播放帧动画
- 常见属性
```objectivec
@property(nonatomic,retain) UIImage *image; // 显示的图片
@property(nonatomic,copy) NSArray *animationImages; // 显示的动画图片
@property(nonatomic) NSTimeInterval animationDuration; // 动画图片的持续时间
@property(nonatomic) NSInteger animationRepeatCount; // 动画的播放次数（默认是0，代表无限播放）
```
- 常见方法
```objectivec
- (void)startAnimating; // 开始动画
- (void)stopAnimating; // 停止动画
- (BOOL)isAnimating; // 是否正在执行动画
```
- UIImage类
- 常用创建方法
```objectivec
+ (UIImage *)imageNamed:(NSString *)name; // 图片名
+ (UIImage *)imageWithContentsOfFile:(NSString *)path; // 图片全路径
```
- 图片拉伸代码
```objectivec
- (UIImage *)resizableImageWithCapInsets:(UIEdgeInsets)capInsets
- (UIImage *)resizableImageWithCapInsets:(UIEdgeInsets)capInsets resizingMode:(UIImageResizingMode)resizingMode
```
---
#### UIButton
- 可以和用户交互，显示图片和文字
- 状态
- normal 普通状态，默认状态Default， UIControlStateNormal
- highlighted 高亮状态，按钮被按下的状态，手没有抬起， UIControlStateHighlighted
- disable 失效状态，不能点击的状态， UIControlStateDisabed
- 设置
```objectivec
- (void)setTitle:(NSString *)title forState:(UIControlState)state; // 设置按钮的文字
- (void)setTitleColor:(UIColor *)color forState:(UIControlState)state; // 设置按钮的文字颜色
- (void)setImage:(UIImage *)image forState:(UIControlState)state; // 设置按钮内部的小图片
- (void)setBackgroundImage:(UIImage *)image forState:(UIControlState)state; // 设置按钮的背景图片
```
- 监听事件
- 拖线<br/>
![](http://ccc)
- 代码
```objc
// 按钮点击后就会调用self的btnClick:方法，并将按钮本身作为参数传递过去
[button addTarget:self action:@selector(btnClick:) forControlEvents:UIControlEventTouchUpInside];
```
- 内部子控件
- 一个UILabel和一个UIImageView
- 不可以直接改变button内部子控件的位置
- 自定义的时候，需要重写两个方法
```objectivec
- (CGRect)titleRectForContentRect:(CGRect)contentRect{}
- (CGRect)imageRectForContentRect:(CGRect)contentRect{}
- (void)layoutSubviews {
[super layoutSubviews];
}
```
- 内边距属性
- contentEdgeInsets 所有内容的内边距
- titleEdgeInsets 文字的内边距
- imageEdgeInsets 图片的内边距
---
##拳皇小例子
![](https://thumbnail0.baidupcs.com/thumbnail/05250442ffa49b13b8df3a97a3716711?fid=3191038685-250528-1084723479568916&time=1472371200&rt=sh&sign=FDTAER-DCb740ccc5511e5e8fedcff06b081203-IZMnTrQSboZz5ElN9SL0fbR7zh4%3D&expires=8h&chkv=0&chkbd=0&chkpc=&dp-logid=5582483404209842225&dp-callid=0&size=c710_u400&quality=100)
- 帧动画
```objectivec
self.imageView.animationImages = images; // 动画需要图片数组
self.imageView.animationRepeatCount = repeatCount; // 动画播放次数
self.imageView.animationDuration = duration; // 每次的播放时间
[self.imageView startAnimating]; // 开始动画
```



