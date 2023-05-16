---
title: my-ios-day18
date: 2016-09-01 16:12:12
categories: 
- technology
- Objective-C 
tags: 
- ios
- gitbook
---

这个是16年重新复习的iOS笔记，当时是放在了[gitbook](https://sleenxiu.gitbooks.io/my-learning-ios/content/)，现在转移过来！
<!-- more -->

# Day 18
## 多线程
- 单例模式
- NSOperation
- 多图下载
- SDWebImage框架
- NSCache
- 位移枚举
- RunLoop
---
#### 单例模式
- 简介
- 整个程序运行中，一个类只有一个实例，且易于外界访问
- 方便控制实例个数，节约资源
- 使用场景
- 整个应用共享一份资源，只需要初始化一次
- 实现
- 新建一个类`SLPerson`
```objc
@interface SLPerson : NSObject
/**
* 返回person单例
*/
+ (instancetype)sharePerson;
@end
@implementation SLPerson
static SLPerson *_person;
+ (instancetype)allocWithZone:(struct _NSZone *)zone {

// 懒加载
// @synchronized(self) {
// if (_person == nil) {
// _person = [super allocWithZone:zone];
// }
// }
// 一次性代码
static dispatch_once_t onceToken;
dispatch_once(&onceToken, ^{
_person = [super allocWithZone:zone];
});
return _person;
}

+ (instancetype)sharePerson {
return [[SLPerson alloc] init];
}

- (id)copyWithZone:(NSZone *)zone {
return _person;
}
- (id)mutableCopyWithZone:(NSZone *)zone {
return _person;
}
@end
```
---
#### NSOperation
- 简介
- 基于GCD
- 两个概念：队列和操作
- 使用
- NSOperation是一个抽象类，要使用其子类
- 可以使用的三个子类：NSBlockOperation、NSInvocationOperation和自定义继承自NSOperation的类
- NSOperation和NSOperationQueue需要结合使用
- 代码
- `NSInvocationOperation`
```objc
/*
第一个参数：目标对象
第二个参数：该操作要调用的方法
第三个参数：调用方法传递的参数
*/
NSInvocationOperation *operation = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(run) object:nil];
// 启动操作
[operation start];
```
- `NSBlockOperation`
```objc
NSBlockOperation *ope1 = [NSBlockOperation blockOperationWithBlock:^{
NSLog(@"downLoad--1--%@", [NSThread currentThread]);
}];
NSBlockOperation *ope2 = [NSBlockOperation blockOperationWithBlock:^{
NSLog(@"downLoad--2--%@", [NSThread currentThread]);
}];
// 启动操作
[ope1 start];
[ope2 start];
```
- `NSOperationQueue`的使用
- 两种队列
1. 主队列，通过mainQueue获得，放在主队列的操作都在主线程中执行
2. 非主队列，使用alloc init得到的队列，具有并发和串行的功能，通过设置最大并发数属性来控制操作是并发还是串行
- 代码
```objc
/*
invocation和非主队列
会开子线程，实现并发操作
*/
- (void)invocationWithQueue {
// 创建操作
NSInvocationOperation *op1 = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(run1) object:nil];
NSInvocationOperation *op2 = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(run2) object:nil];
NSInvocationOperation *op3 = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(run3) object:nil];
// 创建队列
NSOperationQueue *queue = [[NSOperationQueue alloc] init];
// 添加操作到队列
// 内部会调用[op start];
[queue addOperation:op1];
[queue addOperation:op2];
[queue addOperation:op3];
}
/*
block和非主队列
会开子线程，操作并发
*/
- (void)blockWithQueue {
NSBlockOperation *op1 = [NSBlockOperation blockOperationWithBlock:^{
NSLog(@"downLoad--1--%@", [NSThread currentThread]);
}];
NSBlockOperation *op2 = [NSBlockOperation blockOperationWithBlock:^{
NSLog(@"downLoad--2--%@", [NSThread currentThread]);
}];
NSBlockOperation *op3 = [NSBlockOperation blockOperationWithBlock:^{
NSLog(@"downLoad--3--%@", [NSThread currentThread]);
}];
// 创建队列
NSOperationQueue *queue = [[NSOperationQueue alloc] init];
// 添加操作到队列
// 内部会调用[op start];
[queue addOperation:op1];
[queue addOperation:op2];
[queue addOperation:op3];
// 有快速的创建方法
// [queue addOperationWithBlock^{}];
}
```
- 自定义`SLOperation` : `NSOperation`
- 代码
```objc
- (void)customWithQueue {
// 创建操作
SLOperation *op1 = [[SLOperation alloc] init];
SLOperation *op2 = [[SLOperation alloc] init];
// 创建队列
NSOperationQueue *queue = [[NSOperationQueue alloc] init];
// 添加操作到队列
[queue addOperation:op1];
[queue addOperation:op2];
}
// SLOperation.m
@implemetation SLOperation
// 重写mian方法，设置操作
- (void)main {
NSLog(@"downLoad---");
}
@end
```
- 使用
- 有利于代码的隐蔽性
- 提高代码的复用性
- `NSOperation`的一些功能
- 最大并发数
```objc
// 创建队列
NSOperationQueue *queue = [[NSOperationQueue alloc] init];
// 设置最大并发数：同一时间最多有多少个操作执行，默认不受限制的
queue.maxConcurrentOperationCount = 5;
// 可以将默认的并发设置为串行
// 串行并不是只开一条线程，而是线程同步
// queue.maxConcurrentOperationCount = 1;
// 创建操作
SLOperation *op1 = [[SLOperation alloc] init];
SLOperation *op2 = [[SLOperation alloc] init];
// 添加操作到队列
[queue addOperation:op1];
[queue addOperation:op2];
```
- 暂停恢复和取消
- 暂停恢复
```objc
// 设置队列的暂停 YES、恢复 NO
@property (getter=isSuspended) BOOL suspended;
```
- 取消
```objc
// 取消执行队列中的所有任务
- (void)cancelAllOperations;
// 在自定义的Operation中
// 苹果官方建议，每当执行完一次耗时操作之后，就查看一下当前操作是否为取消状态，如果是，那么就直接退出，可以用户体验
if (self.isCancelled) {
return;
}
```
- 操作依赖和监听
- 依赖
```objc
NSBlockOperation *op1 = [NSBlockOperation blockOperationWithBlock:^{
NSLog(@"downLoad--1--%@", [NSThread currentThread]);
}];
NSBlockOperation *op2 = [NSBlockOperation blockOperationWithBlock:^{
NSLog(@"downLoad--2--%@", [NSThread currentThread]);
}];
NSBlockOperation *op3 = [NSBlockOperation blockOperationWithBlock:^{
NSLog(@"downLoad--3--%@", [NSThread currentThread]);
}];
// 创建队列
NSOperationQueue *queue = [[NSOperationQueue alloc] init];
// 添加依赖
// op1依赖于op2，只有op2完成才能执行op1
[op1 addDependency:op2];
// 添加操作到队列
[queue addOperation:op1];
[queue addOperation:op2];
[queue addOperation:op3];
```
- 监听
```objc
NSBlockOperation *op1 = [NSBlockOperation blockOperationWithBlock:^{
NSLog(@"downLoad--1--%@", [NSThread currentThread]);
}];
NSBlockOperation *op2 = [NSBlockOperation blockOperationWithBlock:^{
NSLog(@"downLoad--2--%@", [NSThread currentThread]);
}];
NSBlockOperation *op3 = [NSBlockOperation blockOperationWithBlock:^{
NSLog(@"downLoad--3--%@", [NSThread currentThread]);
}];
// 监听操作
op3.completionBlock = ^{
NSLog(@"op3 完成");
};
// 创建队列
NSOperationQueue *queue = [[NSOperationQueue alloc] init];
// 添加操作到队列
[queue addOperation:op1];
[queue addOperation:op2];
[queue addOperation:op3];
```
- `NSOperation`线程间通信
```objc
// 创建队列
NSOperationQueue *queue = [[NSOperationQueue alloc]init];
// 使用简便方法封装操作并添加到队列中
[queue addOperationWithBlock:^{
// 在该block中下载图片
NSURL *url = [NSURL URLWithString:@"http://news.51sheyuan.com/uploads/allimg/111001/133442IB-2.jpg"];
NSData *data = [NSData dataWithContentsOfURL:url];
UIImage *image = [UIImage imageWithData:data];
NSLog(@"下载图片操作--%@",[NSThread currentThread]);
// 回到主线程刷新UI
[[NSOperationQueue mainQueue] addOperationWithBlock:^{
self.imageView.image = image;
NSLog(@"刷新UI操作---%@",[NSThread currentThread]);
}];
}];
```
---
#### 多图下载
- 简要代码
```objc
SLApp *app = self.apps[indexPath.row];
cell.textLabel.text = app.name;
cell.detailTextLabel.text = app.download;

NSData *data = [NSData dataWithContentsOfURL:[NSURL URLWithString:app.icon]];
UIImage *image = [UIImage imageWithData:data];
cell.imageView.image = image;
return cell;
```
- 涉及问题
- UI卡顿
- 图片多次下载
- 解决
- 图片多次下载
```objc
// 使用内存缓存
// 去内存缓存找图片
UIImage *image = [self.images objectForKey:app.name];
if (image == nil) {
// 如果没有就去下载
NSData *data = [NSData dataWithContentsOfURL:[NSURL URLWithString:app.icon]];
UIImage *image = [UIImage imageWithData:data];
cell.imageView.image = image;
// 添加图片到内存缓存
[self.images setObject:image forKey:app.name];
}else {
cell.imageView.image = image;
}
// 使用磁盘缓存
/*
Documents：会备份，
Libray：
Preferences：偏好设置，账号属性
Caches：缓存文件
tmp：临时文件，随时会被删除
*/
// 去内存缓存找图片
UIImage *image = [self.images objectForKey:app.name];
if (image == nil) {
// 获取缓存路径
NSString *path = [NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) lastObject];
// 拼接图片全路径
NSString *fileName = [app.icon lastPathComponent];
NSString *imagePath = [path stringByAppendingPathComponent:fileName];
// 内存中没有
// 1. 先检查磁盘缓存
NSData *imageData = [NSData dataWithContentsOfFile:imagePath];
if (imageData == nil) {
// 2. 如果没有就去下载，保存到内存和沙盒
NSData *data = [NSData dataWithContentsOfURL:[NSURL URLWithString:app.icon]];
UIImage *image = [UIImage imageWithData:data];
cell.imageView.image = image;
// 添加图片到内存缓存
[self.images setObject:image forKey:app.name];
// 添加图片到磁盘缓存
// 写数据打沙盒
[data writeToFile:imagePath atomically:YES];
}
UIImage *image = [UIImage imageWithData:imageData];
cell.imageView.image = image;
}else {
cell.imageView.image = image;
}
return cell;
```
- UI的流畅性
- 开线程下载图片
```objc
if (imageData == nil) {
// 2. 如果没有就去下载，保存到内存和沙盒
[self.queue addOperationWithBlock:^{
NSData *data = [NSData dataWithContentsOfURL:[NSURL URLWithString:app.icon]];
UIImage *image = [UIImage imageWithData:data];
// 添加图片到内存缓存
[self.images setObject:image forKey:app.name];
// 添加图片到磁盘缓存
[data writeToFile:imagePath atomically:YES];
// 回到主线程刷新UI
[[NSOperationQueue mainQueue] addOperationWithBlock:^{
cell.imageView.image = image;
}];
}];
}
```
- 图片不会刷新
- 图片已经下载，但是应为frame问题不会显示，主动刷新
```objc
// 刷新tableView的一行
[[NSOperationQueue mainQueue] addOperationWithBlock:^{
[self.tableView reloadRowsAtIndexPaths:@[indexPath] withRowAnimation:UITableViewRowAnimationAutomatic];
// cell.imageView.image = image;
}];
```
- 滑动过快，图片下载需要时间，图片下载操作反复添加
- 操作缓存
```objc
[self.operation setObject:[[self.queue operations] lastObject] forKey:app.icon];
```
- 总结
- 主要关注点
1. 缓存的处理，NSDictionary使用
2. 存储数据到沙盒
3. 占位图片的处理
4. 程序并发时，差错控制
5. cell重用的数据错乱处理
6. NSOperation的使用
- 主要逻辑
1. 在本地磁盘查找缓存，如果存在直接显示
2. 磁盘中没有查看内存缓存，如有直接显示，并保存到磁盘
3. 如果内存没有，创建NSOperation下载操作，添加到NSOperationQueue中
4. 下载操作中，图片下载完毕，需要保存到内存和磁盘缓存
5. 对url进行容错处理，下载不成功要重新下载
6. 添加操作到操作缓存，避免操作多次添加，图片多次下载
7. 图片下载完成回到主线程，要立即刷新当前行
8. 在最开始为imageView设置占位图片
---
#### SDWebImage
- 简介
- 使用
- 方法
```objc
/*
第一个参数：下载图片的url
第二个参数：占位图片
第三个参数：枚举值，下载图片的策略
回调参数
receivedSize：已经下载完成的数据大小
expectedSize：该文件的总大小
image：下载得到的图片数据
error：下载的错误信息
SDImageCacheType：图片的缓存策略
imageURL：下载图片的url
*/
[cell.imageView sd_setImageWithURL:(NSURL *) placeholderImage:(UIImage *) options:(SDWebImageOptions) progress:^(NSInteger receivedSize, NSInteger expectedSize) {
NSLog(@"%f", 1.0 * receivedSize / expectedSize);
} completed:^(UIImage *image, NSError *error, SDImageCacheType cacheType, NSURL *imageURL) {
NSLog(@"%@", image);
}];
```
- 内存
- 一些细节
- 开子线程的最大并发数
```objc
// 默认是6
_downloadQueue.maxConcurrentOperationCount = 6;
- (NSInteger)maxConcurrentDownloads {
return _downloadQueue.maxConcurrentOperationCount;
}
```
- 缓存文件的路径和名称
```objc
名称是URL的MD5加密
Cache-->default-->*ache-->图片
```
- 内部内存管理
```objc
// 通过监听通知的方式，来清理的
[[NSNotificationCenter defaultCenter] addObserver:self
selector:@selector(clearMemory)
name:UIApplicationDidReceiveMemoryWarningNotification
object:nil];

[[NSNotificationCenter defaultCenter] addObserver:self
selector:@selector(cleanDisk)
name:UIApplicationWillTerminateNotification
object:nil];

[[NSNotificationCenter defaultCenter] addObserver:self
selector:@selector(backgroundCleanDisk)
name:UIApplicationDidEnterBackgroundNotification
object:nil];
```
- 内部缓存方法
```objc
类似于NSMutableDictionary的 ---> NSCache
```
- 图片类别的判断
```objc
// 内部主要根据图片十六进制的第一个字节
+ (NSString *)sd_contentTypeForImageData:(NSData *)data;
```
- 图片下载方法
```objc
NSURLConnection ----> 建立网络请求连接
建立连接后通过代理来下载图片
```
- 内部逻辑结构
```objc
// UIButton+WebCache
- (void)sd_setBackgroundImageWithURL:(NSURL *)url forState:(UIControlState)state placeholderImage:(UIImage *)placeholder options:(SDWebImageOptions)options completed:(SDWebImageCompletionBlock)completedBlock;
// SDWebImageManager
- (id <SDWebImageOperation>)downloadImageWithURL:(NSURL *)url options:(SDWebImageOptions)options progress:(SDWebImageDownloaderProgressBlock)progressBlock completed:(SDWebImageCompletionWithFinishedBlock)completedBlock;
// SDWebImageDownloader
- (id <SDWebImageOperation>)downloadImageWithURL:(NSURL *)url options:(SDWebImageDownloaderOptions)options progress:(SDWebImageDownloaderProgressBlock)progressBlock completed:(SDWebImageDownloaderCompletedBlock)completedBlock
// SDWebImageDownloaderOperation
- (id)initWithRequest:(NSURLRequest *)request options:(SDWebImageDownloaderOptions)options progress:(SDWebImageDownloaderProgressBlock)progressBlock completed:(SDWebImageDownloaderCompletedBlock)completedBlock cancelled:(SDWebImageNoParamsBlock)cancelBlock;
```
---
#### NSCache
- 简介
- 苹果官方的缓存处理类，与字典类似
- 内存很低会自动释放对象
- 线程安全的，不需要加锁
- 对象的强引用，不是拷贝
- 属性
- name：名称
- delegate：设置代理
- totalCostLimit：缓存空间的总成本，超出会回收，默认0没限制
- evictsObjectsWithDiscardedContent：标识缓存是否回收废弃内容
- 使用
```objc
// 添加缓存
- (void)setObject:(ObjectType)obj forKey:(KeyType)key; // 0 cost：成本
- (void)setObject:(ObjectType)obj forKey:(KeyType)key cost:(NSUInteger)g;
// 拿到缓存
- (nullable ObjectType)objectForKey:(KeyType)key;
// 删除缓存
- (void)removeObjectForKey:(KeyType)key;
- (void)removeAllObjects;
```
---
#### 位移枚举
- 枚举
- 第一种
```objc
typedef enum
{
SLTypeOne,
SLTypeTwo
}SLType;
```
- 第二种，可以定义类型
```objc
typedef NS_ENUM(NSInteger, SLType)
{
SLTypeOne,
SLTypeTwo
};
```
- 第三种，位移枚举
```objc
typedef NS_OPTIONS(NSInteger, SLNewType)
{
SLTypeOne = 1 << 0, // 1
SLTypeTwo = 1 << 1, // 2
SLTypeThree = 1 << 2, // 4
SLTypeFour = 1 << 3 // 8
}
```
- 用途：多个量传递
```objc
// 按位或 |
// objc.type = 0011
obj.type = SLTypeOne | SLTypeTwo;
// 按位与 &
// 0011 & 0001 = 1
if(obj.type & SLTypeOne) {
NSLog(@"SLTypeOne");
}
// 0011 & 0010 = 1
if(obj.type & SLTypeTwo) {
NSLog(@"SLTypeTwo");
}
// 0011 & 0100 = 0
if(obj.type & SLTypeThree) {
NSLog(@"SLTypeThree");
}
// 0011 & 1000 = 0
if(obj.type & SLTypeFour) {
NSLog(@"SLTypeFour");
}
```
---
#### RunLoop
- 简介
- 运行循环，保持程序的持续运行
- 处理App中的各种事件（触摸、定时器、Selector）
- 节省CPU资源，提高性能，有事就做，没事休息
- 两套API访问RunLoop
- Foundation-->NSRunLoop
- Core Foundation-->CFRunLoopRef
- RunLoop与线程
- 每条线程都有唯一的一个RunLoop对象
- 主线程的RunLoop已经自动创建好了，子线程的RunLoop需要主动创建
- RunLoop在第一次获取是创建，在线程结束时销毁
- 获得RunLoop对象
- Foundation
```objc
[NSRunLoop currentRunLoop]; // 获得当前线程的RunLoop对象
[NSRunLoop mainRunLoop]; // 获得主线程的RunLoop对象
```
- Core Foundation
```objc
CFRunLoopGetCurrent(); // 获得当前线程的RunLoop对象
CFRunLoopGetMain(); // 获得主线程的RunLoop对象
```
- Core Foundation 中的五个类
- CFRunLoopRef
- CFrunLoopModeRef 运行模式
- CFRunLoopSourceRef 事件源输入源
- CFRunLoopTimerRef 定时器事件
- CFRunLoopObserveRef 监听运行
- RunLoop运行
- RunLoop有多个运行模式，但它只能在一种模式运行
- Mode中至少要一个Source或Timer，Observe可选
- 每次RunLoop启动，只能指定一个Mode（currentMode）
- 如果需要切换Mode，只能退出Loop，在指定新的Mode
- 系统默认五种运行模式
- kCFRunLoopDefaultMode：app的默认模式，通常主线程在这个模式下运行
- UITrackingRunLoopMode：界面跟踪Mode，用于ScrollView追踪滑动触摸，保证界面滑动不受其他Mode影响
- UIInitializationRunLoopMode：在刚启动App时进入的第一个Mode，启动完就不在使用
- GSEventReceiveRunLoopMode：接收系统事件的内部Mode，通常不用
- kCFRunLoopCommonModes：这是一个占位用的Mode，不是真的Mode
- 事件源（输入源）分类
- 官方文档
1. Port-Based Sources
2. Custom Input Sources
3. Cocoa Perform Selector Sources
- 函数调用栈
1. Source0：非基于Port的
2. Source1：基于Port的
- 监听者Observer
- 监听RunLoop的状态改变
- 步骤
```objc
// 创建一个监听者对象
CFRunLoopObserverRef observer = CFRunLoopObserverCreateWithHandler(CFAllocatorGetDefault(),kCFRunLoopAllActivities, YES, 0, ^(CFRunLoopObserverRef observer, CFRunLoopActivity activity) {
NSLog(@"监听runloop状态改变---%zd",activity);
});
// 添加监听者
CFRunLoopAddObserver(CFRunLoopGetCurrent(), observer, kCFRunLoopDefaultMode);
// 移除监听者
CFRelease(observer);
```
- 几个RunLoop状态
```objc
typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
kCFRunLoopEntry = (1UL << 0), //即将进入Runloop
kCFRunLoopBeforeTimers = (1UL << 1), //即将处理NSTimer
kCFRunLoopBeforeSources = (1UL << 2), //即将处理Sources
kCFRunLoopBeforeWaiting = (1UL << 5), //即将进入休眠
kCFRunLoopAfterWaiting = (1UL << 6), //刚从休眠中唤醒
kCFRunLoopExit = (1UL << 7), //即将退出runloop
kCFRunLoopAllActivities = 0x0FFFFFFFU //所有状态改变
};
```
- RunLoop的运行逻辑
- 事件队列
- 每次运行RunLoop，会自动处理之前未处理的消息，并通知相关的监听者
- 具体过程
1. 通知监听者RunLoop已经启动
2. 通知监听者任何即将开始的定时器
3. 通知监听者任何即将启动的非基于端口的源（Source）
4. 启动任何准备好的非基于端口的源
5. 如果基于端口的源准备好并处于等待状态，立即启动，处理未处理的事件
6. 通知监听者线程进入休眠
7. 直到以下事件发生，线程将被唤醒
某一事件到达基于端口的源
定时器启动
RunLoop设置的时间已经超时
RunLoop被显式唤醒
8. 通知监听者线程将被唤醒
9. 处理未处理事件
10. 通知观察者RunLoop结束
11. 未处理事件
定时器启动，处理定时器事件并重启RunLoop
输入源启动，传递相应消息
被显式唤醒且没有超时，重启RunLoop
- 应用
- NSTimer
- ImageView显示
- performSelector
- 常驻线程：子线程中开启一个runloop
- 自动释放池








