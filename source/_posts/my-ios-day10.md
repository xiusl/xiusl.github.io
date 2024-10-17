---
title: my-ios-day10
date: 2016-09-01 16:11:49
categories: iOS
tags: 
- ios
- gitbook
---

这个是16年重新复习的iOS笔记，当时是放在了[gitbook](https://sleenxiu.gitbooks.io/my-learning-ios/content/)，现在转移过来！
<!-- more -->

# day 10
## 多控制器管理（一）
- 项目中常见文件
- UIApplication
- 程序启动原理
- UIWindow
- 加载控制器
---
#### 常见文件
- 系统框架及LaunchScreen
- Xcode5，框架是苹果事先导入进去的，在项目栏中就会看见导入的框架，使用image.xcassets中的`LaunchScreen`文件
- Xcode6，会自动导入一些常用框架，不会看到自动导入的框架，使用`LaunchScreen.xib`文件
- Xcode7，会自动导入一些常用框架，不会看到自动导入的框架，使用`LaunchScreen.storyboard`文件
- 新版本中，依旧通过更改项目的General的`Launch Screen File`配置来使用`Assets.xcassets`中的`LaunchScreen`文件
- PCH文件
- 一般与项目名相同，新版本不会自动添加
- 添加过程
- 新建一个`项目名.pch`文件
- 在Build Setting中查找**perfix**，找到`Precomplie prefix Header`设置为 YES
- 设置`prefix Header`为pch文件的完整工程路径，`项目名/项目名.pch`
- 只要在pch文件中定义的东西，就会被整个应用程序共享
- 是一个预编译文件，系统提前去编译，完成一些配置
- 主要用途
- 存放一些常用的宏，屏幕、版本...
- 存放公用的头文件，系统分类的头文件
- 自定义NSLog，实现部署不打印
- 注意
- 问题：pch会把当中的所有内容导入到工程中的所有文件中，当工程中存在C语言文件时，就会产生错误
- 解决：每一个OC文件都会定义一个`__OBJC__`宏，判断文件是否有这个宏，就可以判断文件类型

---
#### UIApplication
- UIApplication简介
- UIApplication对象是一个应用程序的象征，系统为我们创建，是一个单例对象，一个应用程序启动后就是一个UIApplication对象，可以通过`[UIApplication sharedApplication]`获得这个对象
- 应用UIApplication对象可以完成一些应用级别的操作
- 设置应用程序右上角的红色提醒数字
- 设置联网指示器的可见性
- 设置应用程序的状态栏
- 完成应用之间的跳转
- UIApplication功能
- 提醒数字
```objc
// 获取UIApplication对象
UIApplication *app = [UIApplication sharedApplication];
// 在设置之前，要注册⼀个通知，从ios8之后，都要先注册一个通知对象，才能够接收到提醒
UIUserNotificationSettings *notice = [UIUserNotificationSettings settingsForTypes:UIUserNotificationTypeBadge categories:nil];
// 注册通知对象
[app registerUserNotificationSettings:notice];
// 设置提醒数字
app.applicationIconBadgeNumber = 10;
```
- 设置联网状态
```objc
app.networkActivityIndicatorVisible = YES;
```
- 设置状态栏
- 状态栏默认是交个控制器管理的
```objc
// 重写控制器的方法，设置显示样式
-(UIStatusBarStyle)preferredStatusBarStyle {
return UIStatusBarStyleLightContent;
}
// 是否隐藏状态栏，YES--显示，NO--隐藏
-(BOOL)prefersStatusBarHidden {
return NO;
}
```
- 开发中一般都是应用程序来统一管理，在`info.plist`文件中添加一个Key值，`View controller-based status bar appearance`设置为 NO，就交给应用程序管理了
```objc
// 获取UIApplication
UIApplication *app = [UIApplication sharedApplication];
// 设置状态栏样式.
app.statusBarStyle = UIStatusBarStyleLightContent;
// 设置状态的隐藏
app.statusBarHidden = YES;
```
- 跳转网页
```objc
UIApplication *app = [UIApplication sharedApplication];
// URL--协议头://域名
// 应用程序通过协议头的类型，去打开相应的软件
NSURL *url =[NSURL URLWithString:@"http://www.baidu.com"];
[app openURL:url];
```
- 打电话发短信
```objc
// 打电话
[app openURL:[NSURL URLWithString:@"tel://10086"]];
// 发短信
[app openURL:[NSURL URLWithString:@"sms://10086"]];
```
- UIApplication代理
- 所有的移动应用有一个致命的缺点：**app容易受到打扰**，如来电话，锁屏等会导致app进入后台甚至会被终止
- iOS中当app受到干扰的时候会产生一些系统事件，这时UIApplication会通知它的代理对象，来做一些相应的处理
- AppDelegate类就是在程序启动创建的一个遵守UIApplicationDelegate的类，在这个类中定义很多方法来处理事件
- 主要方法
```objc
#parmark 应⽤程序的⽣生命周期
// 应用程序启动完成的时候调⽤
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions: (NSDictionary *)launchOptions {
NSLog(@"%s",__func__);
return YES;
}
// 当我们应用程序即将失去焦点的时候调⽤
- (void)applicationWillResignActive:(UIApplication *)application {
NSLog(@"%s",__func__);
}
// 当我们应⽤程序完全进⼊后台的时候调⽤
- (void)applicationDidEnterBackground:(UIApplication *)application{
NSLog(@"%s",__func__);
}
// 当我们应用程序即将进⼊前台的时候调⽤
- (void)applicationWillEnterForeground:(UIApplication *)application {
NSLog(@"%s",__func__);
}
// 当我们应⽤程序完全获取焦点的时候调⽤
// 只有当一个应⽤程序完全获取到焦点,才能与用户交互.
- (void)applicationDidBecomeActive:(UIApplication *)application {
NSLog(@"%s",__func__);
}
// 当我们应用程序即将关闭的时候调⽤
- (void)applicationWillTerminate:(UIApplication *)application {
NSLog(@"%s",__func__);
}
```
---
#### 应用程序启动原理
- 程序启动时，依旧最先执行main函数
```objc
int main(int argc, char * argv[]) {
@autoreleasepool {
// 第三个参数:UIApplication类名或者⼦类的名称 nil == @"UIApplication"
// 第四个参数:UIApplication的代理的代理名称
// NSStringFromClass:把类名转化字符串， NSStringFromClass好处:1.有提示功能 2.避免输入错误
return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
}
}
```
- 程序启动流程
1. 执行`main`函数
2. 执行`UIApplicationMain`函数
3. 创建`UIApplication`对象，设置`UIApplication`对象的代理
4. 开启一个主循环，保证应用程序不会退出
5. 加载`info.plist`配置文件，判断文件中的`Main storyboard file base name`有没有指定`storyboard`文件，若果有指定就去加载文件
---
#### UIWindow
- UIWindow简介
- UIWindow是一个特殊的UIView，通常一个app中至少有一个UIWindow，一个iOS程序之所以可以显示到屏幕上，就是靠UIWindow
- iOS启动完毕后，创建的第一个视图控件就是UIWindow，接着创建控制器的View，在将控制器的View添加到UIWindow上
- UIWindow创建
- 加载info.plist文件后，如果存在main就会加载storyboard
- 创建一个窗口
- 加载main.storyboard，初始化一个控制器
- 把初始化的控制器设为窗口的根控制器，显示到屏幕上
- 如果没有指定main这个时候就需要在程序加载完毕后手动创建UIWindow
```objc
// 应用程序启动完成的时候调⽤
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions: (NSDictionary *)launchOptions {
// 创建窗口
self.window = [[UIWindow alloc] initWithFrame:[UIScreen mainScreen].bounds];
// 创建控制器
UIViewController *Vc = [[UIViewController alloc] init];
Vc.view.backgroundColor = [UIColor redColor];
// 设置根控制器
self.window.rooViewController = Vc;
// 显示窗口
[self.window makeKeyAndVisiable];
return YES;
}
```
- `makeKeyAndVisible`的底层实现
1. 让窗⼝变为显示状态
```objc
self.window.hidden = NO;
```
2. 把根控制器的View添加到窗口上
```objc
[self.window addSubView:rootVC.view];
```
3. 把当前窗口设置成应用程序的主窗⼝
application.keyWindow获得应用程序的主窗⼝
在程序中，状态栏和键盘都是窗口
self.window.windowLevel = UIWindowLevelNormal
层级关系
UIWindowLevelNormal < UIWindowLevelStatusBar < UIWindowLevelAlert
---
#### 加载控制器
- storyboard
- 过程
```objc
// 创建窗口
self.window = [[UIWindow alloc] inihFrame:[UIScreen mainScreen].bounds];
// 加载控制器
// 指定storyboard
UIStoryboard *storyboard =[UIStoryboard storyboardWithName:@"Main" bundle:nil];
// 加载箭头指向的Vc
UIViewController *Vc = [storyboard instantiateInitialViewController];
// 设置根控制器
self.window.rooViewController = Vc;
// 显示窗口
[self.window makeKeyAndVisiable];
```
- 加载控制器的两种方式
```objc
// 指定storyboard
UIStoryboard *storyboard =[UIStoryboard storyboardWithName:@"Main" bundle:nil];
// 加载箭头指向的Vc
UIViewController *Vc = [storyboard instantiateInitialViewController];
// 加载指定标识的控制器.
UIViewController *Vc = [storyBoard instantiateViewControllerWithIdentifier:@"VCStoryBoardID"];
```
- xib
```objc
// 创建窗⼝
self.window = [[UIWindow alloc] initWithFrame:[UIScreen nScreen].bounds];
// 从XIB当中加载控制器.
MyViewController *Vc = [[MyViewController alloc] tWithNibName:@"VC" bundle:nil];
// 设置根控制器
self.window.rootViewController = Vc;
// 显⽰示窗⼝
[self.window makeKeyAndVisible];
```




















