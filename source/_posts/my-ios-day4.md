---
title: my-ios-day4
date: 2016-09-01 16:00:36
categories: iOS
tags: 
- ios
- gitbook
---

这个是16年重新复习的iOS笔记，当时是放在了[gitbook](https://sleenxiu.gitbooks.io/my-learning-ios/content/)，现在转移过来！
<!-- more -->

# day04
##控件学习
####UIScrollView
- 基本使用
- 移动设备的屏幕大小是及其有限的，需要展示的内容较多，需要用户使用滚动手势查看屏幕以外的内容
- 将要展示的内容添加到UIScrollView中，设置UIScrollView的contentSize属性
- 常见属性
```objectivec
// 表示scrollView的滚动位置，内容左上角与scrollV左上角的间距值
@property(nonatomic) CGPoint contentOffSet；
// 表示scrollView中内容的位置，滚动范围
@property(nonatomic) CGSize contentSize;
// 在scrollView的四周添加额外的滚动区域
@property(nonatomic) UIEdgeInsets contentInset
// 设置scrollView是否需要弹簧效果
@property(nonatomic) BOOL bounces;
// 是否显示垂直滚动条
@property(nonatomic) BOOL alwaysBounceVertical;
// 是否显示水平滚动条
@property(nonatomic) BOOL alwaysBounceHorizontal;
```
- 代理
- 需要监听UIScrollView的滚动过程，需要使用UIScrollView的代理
```objectivec
@property(nullable, nonatomic, weak) id<UIScrollViewDelegate> delegate;
```
- 代理实现缩放
```objectivec
// 设置minimumZoomScale(缩放最小比例)和maximumZoomScale(缩放最大比例)
// 返回需要缩放的控件
- (nullable UIView *)viewForZoomingInScrollView:(UIScrollView *)scrollView;
// 即将开始缩放的时候调用
- (void)scrollViewWillBeginZooming:(UIScrollView *)scrollView withView:(nullable UIView *)view
// 正在缩放的时候调用
- (void)scrollViewDidZoom:(UIScrollView *)scrollView;
```
- 代理监听滚动
```objectivec
// 当scrollView正在滚动的时候就会自动调用这个方法
- (void)scrollViewDidScroll:(UIScrollView *)scrollView;
// 用户即将开始拖拽scrollView时就会调用这个方法
- (void)scrollViewWillBeginDragging:(UIScrollView *)scrollView;
// 用户即将停止拖拽scrollView时就会调用这个方法
- (void)scrollViewWillEndDragging:(UIScrollView *)scrollView withVelocity:(CGPoint)velocity targetContentOffset:(inout CGPoint *)targetContentOffset;
// scrollView减速完毕会调用,停止滚动
- (void)scrollViewDidEndDecelerating:(UIScrollView *)scrollView
```
- 分页功能
- 设置UIScrollView的pagingEnabled = YES开启分页功能，以scrollView的尺寸为一页
- UIPageControl增强分页效果
```objectivec
@property(nonatomic) NSInteger numberOfPages; // 一共有多少页
@property(nonatomic) NSInteger currentPage; // 当前显示页码
@property(nonatomic) BOOL hidesForSinglePage; // 分一页时是否隐藏页码指示器
@property(nonatomic, reatain) UIColor *pageIndicatorTintColor; // 其他页码指示器的颜色
@property(nonatomic, reatain) UIColor *currentPageIndicatorTintColor; // 当前页码指示器的颜色
```
- NSTimer自动滚动
```objectivec
// 每隔2秒调用self的next方法
NSTimer *timer = [NSTimer timerWithTimeInterval:2 target:self selector:@selector(next) userInfo:nil repeats:YES];
// 解决定时器在主线程不工作的问题
[[NSRunLoop mainRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
```
- KVC
- 简单赋值
```objectivec
- (void)setValue:(nullable id)value forKey:(NSString *)key;
- (void)setValue:(nullable id)value forKeyPath:(NSString *)keyPath;
```
forKey和forKeyPath
1. forKeyPath 包含了所有 forKey 的功能
2. forKeyPath 进行内部的点语法,层层访问内部的属性
3. 注意: key值一定要在属性中找到
- 修改私有成员变量
```objectivec
SLPerson *person = [[SLperson alloc] init];
[person setValue:@"18" forKeyPath:@"_age"];
```
age 是SLPerson的私有成员，可以通过kvc修改它的值
- 取值
```objectivec
- (nullable id)valueForKeyPath:(NSString *)keyPath;
```
- KVO
- Key Value Observing (键值监听)，当某个对象的属性值发生改变的时候(用KVO监听)
- 添加监听
```objectivec
- (void)addObserver:(NSObject *)observer forKeyPath:(NSString *)keyPath options:(NSKeyValueObservingOptions)options context:(nullable void *)context;
```
- 移除监听
```objectivec
- (void)removeObserver:(NSObject *)observer forKeyPath:(NSString *)keyPath;
```
- 监听事件
```objectivec
/**
* 当监听的属性值发生改变
*
* @param keyPath 要改变的属性
* @param object 要改变的属性所属的对象
* @param change 改变的内容
* @param context 上下文
*/
- (void)observeValueForKeyPath:(nullable NSString *)keyPath ofObject:(nullable id)object change:(nullable NSDictionary<NSString*, id> *)change context:(nullable void *)context;
```


## 广告轮播器的实现








