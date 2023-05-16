---
title: my-ios-day17
date: 2016-09-01 16:12:09
categories: 
- technology
- Objective-C 
tags: 
- ios
- gitbook
---

这个是16年重新复习的iOS笔记，当时是放在了[gitbook](https://sleenxiu.gitbooks.io/my-learning-ios/content/)，现在转移过来！
<!-- more -->

# day 17
## 多线程
- 进程与线程
- pthread
- NSThread
- GCD
---
#### 进程与线程
- 进程
- 在系统中正在运行的一个应用程序。每个进程间是独立的，每个进程运行在专有且受保护的内存空间
- 是系统分配资源的基本单位
- 线程
- 一个进程要想执行任务，必须要有一个线程，线程执行任务是串行的，一个线程只能在执行一个任务
- 是系统程序调度的基本单位
- 多线程
- 一个进程可以开启多个线程，多条线程可以并行执行，同时执行
- 单核CPU的这种并行也是只有一条线程工作，只是CPU调度快，线程间不断切换
- 优点
1. 能适当提高程序的执行效率
2. 能适当提高资源（CPU、内存）利用率
- 缺点
1. 开启线程会占用一定内存，开启过多线程会大量占用空间
2. 线程越多，CPU调度线程的开销就越大
3. 程序设计会过于复杂（进程间通信、资源共享）
- 是一种以空间换取时间的技术
---
#### pthread
- 简介
- 一套跨平台的多线程API（Unix、Linux、Windows等）
- 采用C语言，使用难度较大，很少使用
- 使用
```objc
// 创建线程对象
pthread_t thread1;
// 创建线程
/*
pthread_create(pthread_t *restrict, const pthread_attr_t *restrict, void *(*)(void *), void *restrict)
第一个参数：线程对象、传递地址
第二个参数：线程属性、NULL
第三个参数：指向函数的指针
第四个参数：函数需要接收的参数
*/
pthread_create(&thread1, NULL, test, NULL);
```
---
#### pthread
- 简介
- 三种创建方法
```objc
// 通过init方法创建线程，需要手动开启
/*
initWithTarget:(nonnull id) selector:(nonnull SEL) object:(nullable id)
第一个参数：目标对象、self
第二个参数：方法选择器、调用的方法
第三个参数：前面方法需要的参数、nil
*/
NSThread *thread1 = [[NSThread alloc] initWithTarget:self selector:@selector(test1) object:nil];
// 开启线程
[thread1 start];
// 通过类方法创建，分离出的子线程
/*
detachNewThreadSelector:(nonnull SEL) toTarget:(nonnull id) withObject:(nullable id)
第一个参数：方法选择器、调用的方法
第二个参数：目标对象、self
第三个参数：前面方法需要的参数、nil
*/
NSThread *thread2 = [NSThread detachNewThreadSelector:@selector(test2) toTarget:self withObject:nil];
// 开启后台线程
/*
performSelectorInBackground:(nonnull SEL) withObject:(nullable id)
第一个参数：方法选择器、调用的方法
第二个参数：前面方法需要的参数、nil
*/
[self performSelectorInBackground:@selector(test3) withObject:nil];
```
- 属性
```objc
// 设置名字
thread1.name = @"线程1";
// 线程优先级
thread1.threadPriority = 1;
```
- 线程状态
- 新建状态
- 就绪状态
- 运行状态
- 阻塞状态
```objc
// 第一种：阻塞两秒
[NSThread sleepForTimeInterval:2.0];
// 第二种：阻塞三秒
[NSThread sleepUntilDate:[NSDate dateWithTimeInterval:3.0]];
```
- 死亡状态
```objc
// 退出线程，退出后不能再重启线程
[NSThread exit];
```
- 线程安全
- 一块资源被多个线程访问，会引发数据混乱
- 加锁
```objc
// 锁是全局唯一的
@synchronized(self) {
NSInteger count = self.totalCount;
if (count > 0) {
for (int i = 0; i < 10000; i++) {
}
self.totalCount -= 1;
}else {
NSLog(@"没有了");
break;
}
}
```
- 注意：
1. 加锁的位置
2. 加锁的前提条件（多线程共享同一资源）
3. 加锁是有代价的，需要耗费性能的
4. 加锁的结果是：线程同步
- 原子和非原子
- nonatomic和atomic
1. atomic：线程安全，需要耗费大量资源，加锁
2. nonatomic：非线程安全，适合小内存移动设备
- iOS注意
1. 所有属性都声明为nonatomic
2. 尽量避免多线程抢夺同一资源
3. 将加锁、资源抢夺交给服务器，减小客户端压力
- 线程间通信
- 具体体现
1. 一个线程传递数据给另一个线程
2. 在一个特定线程执行完成过后，转到另一个线程继续执行任务
- 常用方法
```objc
/*
第一个参数：回到线程的什么方法
第二个参数：回到那个线程
第三个参数：传递的参数
第四个参数：是否需要等待
*/
- (void)performSelector:(SEL)aSelector onThread:(NSThread *)thr withObject:(id)arg waitUntilDone:(BOOL)wait;
/*
第一个参数：回到主线程的什么方法
第二个参数：传递的参数
第三个参数：是否需要等待
*/
- (void)performSelectorOnMainThread:(SEL)aSelector withObject:(id)arg waitUntilDone:(BOOL)wait;
```
- 图片下载
- 添加UIImageView
- 开启线程，下载图片
- 回到主线程刷新UI
- 代码
```objc
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
[NSThread detachNewThreadSelector:@selector(downLoadImage) toTarget:self withObject:nil];
}
- (void)downLoadImage {
NSLog(@"%@", [NSThread currentThread]);
CFTimeInterval startD = CFAbsoluteTimeGetCurrent();
// 确定url
NSURL *url = [NSURL URLWithString:@"http://www.onegreen.net/maps/Upload_maps/201412/2014120107143496.jpg"];
// 根据url下载图片到本地（二进制数据）
NSData *imageData = [NSData dataWithContentsOfURL:url];
// 转换数据
UIImage *image = [UIImage imageWithData:imageData];
// 回到主线程显示到imageView
[self performSelectorOnMainThread:@selector(showImage:) withObject:image waitUntilDone:YES];
CFTimeInterval endD = CFAbsoluteTimeGetCurrent();
NSLog(@"%f", endD - startD);
}
- (void)showImage:(UIImage *)image {
NSLog(@"%@", [NSThread currentThread]);
self.imageView.image = image;
}
```
- 可以通过imageView调用回到主线程的方法
```objc
[self.imageView performSelectorOnMainThread:@selector(setImage:) withObject:image waitUntilDone:YES];
```
---
#### GCD
- 简介
- Grand Centra Dispatch，纯C语言，拥有非常强大的函数
- 充分利用CPU多核，发挥设备性能
- 会自动管理线程的生命周期（创建、调度、销毁）
- 只需告诉GCD要执行的代码，不编写线程管理代码
- 两个核心概念
- 任务：需要执行的操作
- 队列：用来存放任务
- 两个步骤
- 定制任务，确定要执行的操作
- 添加到队列
1. GCD会自动取出队列中的任务，放到相应的线程执行
2. 任务的取出遵守FIFO（先进先出）原则
- 两个函数
- 同步函数
```objc
/*
queue：队列
block：任务
*/
dispatch_sync(dispatch_queue_t queue, ^(void)block)
```
- 异步函数
```objc
/*
queue：队列
block：任务
*/
dispatch_async(dispatch_queue_t queue, ^(void)block)
```
- 区别
- 同步：只能在当前线程中执行任务，不具备开启新线程的能力
- 异步：可以在新线程中执行任务，具备开启新线程的能力，但不一定开启新线程
- 两种队列
- 并发队列
- 可以让多个任务并发（同时）执行，自动开启多个线程同时执行任务
- 只有在异步函数dispatch_async中在有效（开线程）
- 串行队列
- 让任务一个接着一个的执行（一个任务执行完再执行下一个任务）
- 四种组合
- 异步函数+并发队列
```objc
/**
* 异步函数+并发队列
* 会开启多条线程，任务异步执行（没有顺序，一起执行）
* 并不是有多少个任务就会开多少个线程
*/
- (void)asyncConcurrent {
// 创建队列
/**
* label：C语言字符串，标签
* queue：队列类型
* DISPATCH_QUEUE_CONCURRENT：并发
* DISPATCH_QUEUE_SERIAL：串行
*/
dispatch_queue_t queue = dispatch_queue_create("xyz.Sleen", DISPATCH_QUEUE_CONCURRENT);
// 封装任务，添加到队列
dispatch_async(queue, ^{
NSLog(@"downLoad - image1--%@", [NSThread currentThread]);
});
dispatch_async(queue, ^{
NSLog(@"downLoad - image2--%@", [NSThread currentThread]);
});
dispatch_async(queue, ^{
NSLog(@"downLoad - image3--%@", [NSThread currentThread]);
});
/* 打印结果
downLoad - image1--<NSThread: 0x7ff4a370ce40>{number = 2, name = (null)}
downLoad - image3--<NSThread: 0x7ff4a3614ff0>{number = 4, name = (null)}
downLoad - image2--<NSThread: 0x7ff4a347d260>{number = 3, name = (null)}
*/
}
```
- 异步函数+串行队列
```objc
/**
* 异步函数+串行队列
* 会开启一条新线程，任务串行执行（一个接着一执行）
*/
- (void)asyncSerial {
// 创建队列
dispatch_queue_t queue = dispatch_queue_create("xyz.Sleen", DISPATCH_QUEUE_SERIAL);
// 封装任务，添加到队列
dispatch_async(queue, ^{
NSLog(@"downLoad - image1--%@", [NSThread currentThread]);
});
dispatch_async(queue, ^{
NSLog(@"downLoad - image2--%@", [NSThread currentThread]);
});
dispatch_async(queue, ^{
NSLog(@"downLoad - image3--%@", [NSThread currentThread]);
});
/* 打印结果
downLoad - image1--<NSThread: 0x7fe481e0ba00>{number = 2, name = (null)}
downLoad - image2--<NSThread: 0x7fe481e0ba00>{number = 2, name = (null)}
downLoad - image3--<NSThread: 0x7fe481e0ba00>{number = 2, name = (null)}
*/
}
```
- 同步函数+并发队列
```objc
/**
* 同步函数+并发队列
* 不会开线程，任务串行执行（一个接着一执行）
*/
- (void)syncConcurrent {
// 创建队列
dispatch_queue_t queue = dispatch_queue_create("xyz.Sleen", DISPATCH_QUEUE_CONCURRENT);
// 封装任务，添加到队列
dispatch_sync(queue, ^{
NSLog(@"downLoad - image1--%@", [NSThread currentThread]);
});
dispatch_sync(queue, ^{
NSLog(@"downLoad - image2--%@", [NSThread currentThread]);
});
dispatch_sync(queue, ^{
NSLog(@"downLoad - image3--%@", [NSThread currentThread]);
});
/* 打印结果：
downLoad - image1--<NSThread: 0x7fd3b2f00f50>{number = 1, name = main}
downLoad - image2--<NSThread: 0x7fd3b2f00f50>{number = 1, name = main}
downLoad - image3--<NSThread: 0x7fd3b2f00f50>{number = 1, name = main}
*/
}
```
- 同步函数+串行队列
```objc
/**
* 同步函数+串行队列
* 不会开线程，任务串行执行（一个接着一执行）
*/
- (void)syncSerial {
// 创建队列
dispatch_queue_t queue = dispatch_queue_create("xyz.Sleen", DISPATCH_QUEUE_SERIAL);
// 封装任务，添加到队列
dispatch_sync(queue, ^{
NSLog(@"downLoad - image1--%@", [NSThread currentThread]);
});
dispatch_sync(queue, ^{
NSLog(@"downLoad - image2--%@", [NSThread currentThread]);
});
dispatch_sync(queue, ^{
NSLog(@"downLoad - image3--%@", [NSThread currentThread]);
});
/* 打印结果：
downLoad - image1--<NSThread: 0x7fe333f04dd0>{number = 1, name = main}
downLoad - image2--<NSThread: 0x7fe333f04dd0>{number = 1, name = main}
downLoad - image3--<NSThread: 0x7fe333f04dd0>{number = 1, name = main}
*/
}
```
- 涉及到同步函数的都不会创建新线程，任务都是串行执行
- GCD提供一个全局并发队列
```objc
/**
* 获得全局并发队列
* 第一个参数：优先级
* 第二个参数：“未来使用”--0
*/
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
```
- GCD提供一个主队列（串行队列）
- 异步函数+主队列
```objc
/**
* 异步函数+主队列
* 不会开线程，都在主线程中执行
*/
- (void)asycnMain {
// 获取主线程
dispatch_queue_t queue = dispatch_get_main_queue();
// 封装任务，添加到队列
dispatch_async(queue, ^{
NSLog(@"downLoad - image1--%@", [NSThread currentThread]);
});
dispatch_async(queue, ^{
NSLog(@"downLoad - image2--%@", [NSThread currentThread]);
});
dispatch_async(queue, ^{
NSLog(@"downLoad - image3--%@", [NSThread currentThread]);
});
/* 打印结果
start---
end---
downLoad - image1--<NSThread: 0x7fb533f01aa0>{number = 1, name = main}
downLoad - image2--<NSThread: 0x7fb533f01aa0>{number = 1, name = main}
downLoad - image3--<NSThread: 0x7fb533f01aa0>{number = 1, name = main}
*/
}
```
- 同步函数+主队列
```objc
/**
* 同步函数+主队列
* 死锁
*/
- (void)sycnMain {
// 获取主线程
dispatch_queue_t queue = dispatch_get_main_queue();
NSLog(@"start---");
// 封装任务，添加到队列
dispatch_sync(queue, ^{
NSLog(@"downLoad - image1--%@", [NSThread currentThread]);
});
dispatch_sync(queue, ^{
NSLog(@"downLoad - image2--%@", [NSThread currentThread]);
});
dispatch_sync(queue, ^{
NSLog(@"downLoad - image3--%@", [NSThread currentThread]);
});
NSLog(@"end---");
}
```
- 死锁分析
- 主队列特点：如果主队列发现当前主线程正在执行任务，那么主队列会暂停调用队列中的任务，知道主线程空闲为止
- 同步函数+主队列，主线程正在执行`sycnMain`，因为接下来是同步函数，主线程不执行完`sycnMain`就不会继续执行，主队列就不会调用队列的任务，死锁
- 异步函数+主队列，主线程正在执行`asycnMain`，因为接下来是异步函数，可以等下回来在执行，主线程先执行完`sycnMain`，然后主队列再调用队列的任务，不会死锁
- GCD线程间通信
```objc
dispatch_async(dispatch_get_global_queue(0, 0), ^{
NSLog(@"%@", [NSThread currentThread]);
// 确定url
NSURL *url = [NSURL URLWitString:@"http://www.onegreen.net/maps/Upload_maps/2014122014120107143496.jpg"];
// 根据url下载图片到本地（二进制数据）
NSData *imageData = [NSData dataWihContentsOfURL:url];
// 转换数据
UIImage *image = [UIImage imageWthData:imageData];
// 回到主线程显示到imageView
dispatch_async(dispatch_get_main_queue(), ^{
NSLog(@"%@", [NSThread currentThread]);
self.imageView.image = image;
});
});
/* 打印结果
<NSThread: 0x7ffa8861e9e0>{number = 2, name = (null)}
<NSThread: 0x7ffa88404630>{number = 1, name = main}
*/
```
- GCD其他方法
- 延迟执行
```objc
// 第一种：两秒执行self的test方法
[self performSelector:@selector(test) withObject:self afterDelay:2];
// 第二种：三秒后执行self的test方法，不重复
[NSTimer timerWithTimeInterval:3 target:self selector:@selector(test) userInfo:nil repeats:NO];
// 第三种：delayInSeconds秒后再主队列执行操作
/*
第一个参数：DISPATCH_TIME_NOW，从现在开始计算
第二个参数：延迟的时间，GCD时间单位纳秒
第三个参数：队列（主队列、串行队列、并发队列）
block：操作
*/
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(delayInSeconds * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
// code to be executed after a specified delay
});
```
- 一次性代码
```objc
// 整个程序的运行过程中只会调用一次
static dispatch_once_t onceToken;
dispatch_once(&onceToken, ^{
// code to be executed once
NSLog(@"--once--");
});
```
- 应用场景 ---- 单例模式
- 栅栏函数
- 控制多线程异步执行顺序
```objc
dispatch_queue_t queue = dispatch_queue_create("sxy.Sleen", DISPATCH_QUEUE_CONCURRENT);
dispatch_async(queue, ^{
NSLog(@"downLoad--1--%@", [NSThread currentThread]);
});
dispatch_async(queue, ^{
NSLog(@"downLoad--2--%@", [NSThread currentThread]);
});
// 栅栏函数
// 不能使用全局并发队列
dispatch_barrier_async(queue, ^{
NSLog(@"-------栅栏函数");
});
dispatch_async(queue, ^{
NSLog(@"downLoad--3--%@", [NSThread currentThread]);
});
/*
downLoad--1--<NSThread: 0x7fad93c08760>{number = 3, name = (null)}
downLoad--2--<NSThread: 0x7fad93e2b890>{number = 2, name = (null)}
-------栅栏函数
downLoad--3--<NSThread: 0x7fad93e2b890>{number = 2, name = (null)}
*/
```
- 快速迭代
- 通过子线程和主线程并发执行
```objc
/**
* 第一个参数：遍历的次数
* 第二个参数：队列（只能是并发队列）
* 第三个参数：索引
*/
dispatch_apply(5, dispatch_get_global_queue(0, 0), ^(size_t index) {
NSLog(@"%zu---%@", index, [NSThread currentThread]);
});
/*
0---<NSThread: 0x7fe83b629bb0>{number = 2, name = (null)}
1---<NSThread: 0x7fe83b706490>{number = 1, name = main}
2---<NSThread: 0x7fe83b517710>{number = 3, name = (null)}
3---<NSThread: 0x7fe83b4115b0>{number = 4, name = (null)}
4---<NSThread: 0x7fe83b629bb0>{number = 2, name = (null)}
*/
```
- 队列组
- 用来管理队列，
```objc
- (void)group {
// 创建队列
dispatch_queue_t queue = dispatch_get_global_queue(0, 0);
// 创建队列组
dispatch_group_t group = dispatch_group_create();
dispatch_group_async(group, queue, ^{
NSLog(@"downLoad--1--%@", [NSThread currentThread]);
});
dispatch_group_async(group, queue, ^{
NSLog(@"downLoad--2--%@", [NSThread currentThread]);
});
dispatch_group_async(group, queue, ^{
NSLog(@"downLoad--3--%@", [NSThread currentThread]);
});
// 拦截通知，队列组中的任务执行完毕后调用
dispatch_group_notify(group, queue, ^{
NSLog(@"ALL--downLoad--");
});
// 等待，直到队列组所有任务完成后才向下执行
dispatch_group_wait(group, DISPATCH_TIME_FOREVER);
// 区别
/* 异步函数
* 1. 封装任务
* 2. 添加任务到队列
dispatch_async(queue, ^{
NSLog(@"downLoad--1--%@", [NSThread currentThread]);
});
* 队列组
* 1. 封装任务
* 2. 添加任务到队列
* 3. 监听任务的执行情况，通知group
dispatch_group_async(group, queue, ^{
NSLog(@"downLoad--2--%@", [NSThread currentThread]);
});
*/
}
```















