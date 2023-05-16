---
title: my-ios-day11
date: 2016-09-01 16:11:51
categories: 
- technology
- Objective-C 
tags: 
- ios
- gitbook
---

这个是16年重新复习的iOS笔记，当时是放在了[gitbook](https://sleenxiu.gitbooks.io/my-learning-ios/content/)，现在转移过来！
<!-- more -->

# day 11
## 多控制器管理（二）
- UINavigationController
- 通讯录简例
- 数据存储
- UITabBarController
---
#### UINavigationController
- 使用步骤
- 初始化`UINavigationController`
- 设置`UIWindow`的`rootViewController`为`UINavigationController`
- 根据具体情况，通过push方法添加子控制器
- 子控制器
- `UINavigationController`以栈的形式保存子控制器
```objc
@property(nonatomic,copy) NSArray *viewControllers;
@property(nonatomic,readonly) NSArray *childViewControllers;
```
- 使用push方法将子控制器入栈
```objc
- (void)pushViewController:(UIViewController *)viewController animated:(BOOL)animated;
```
- 使用pop方法移除控制器
```objc
// 将栈顶的控制器移除
- (UIViewController *)popViewControllerAnimated:(BOOL)animated;
// 回到指定的子控制器
- (NSArray *)popToViewController:(UIViewController *)viewController animated:(BOOL)animated;
// 回到根控制器（栈底控制器）
- (NSArray *)popToRootViewControllerAnimated:(BOOL)animated;
```
- navigationItem
- 导航条的内容由navigationItem决定
- 常用属性
```objc
// 左上角的返回按钮
@property(nonatomic, retain) UIBarButtonItem *backBarButtonItem;
// 中间的标题视图
@property(nonatomic, retain) UIView *titleView;
// 中间的标题文字
@property(nonatomic, copy) NSString *title;
// 左上角的视图
@property(nonatomic, retain) UIBarButtonItem *leftBarButtonItem;
// 右上角的视图
@property(nonatomic, retain) UIBarButtonItem *rightBarButtonItem;
```
- segue
- storyboard上的每一条控制器之间的线都是一个UIStoryboardSegue对象
- 属性
```objc
// 唯一标识
@property (nonatomic, readonly) NSString *identifier;
// 来源控制器
@property (nonatomic, readonly) id sourceViewController;
// 目标控制器
@property (nonatomic, readonly) id destinationViewController;
```
- 类型
- 手动型：需要通过代码手动执行segue，才能完成跳转
```objc
// 需要为segue绑定一个唯一标识
// segue必须由来源控制器来执行，也就是说，这个perform方法必须由来源控制器来调用
- [self performSegueWithIdentifier:@"login2contacts" sender:nil];
```
- 自动型：点击某个控件，自动自行segue，自动跳转
- `performSegueWithIdentifier:sender:`执行过程
- 根据`identifier`在`storyboard`中找到对应的线，新建`UIStoryboardSegue`对象
- 设置segue对象的`sourceViewControl`
- 新建并设置segue对象的`destinationViewController`
- 调用`sourceViewController`的`prepareForSegue: sender:`方法，做一些跳转前的准备并传入创建好的segue
- 调用segue的`perform`方法，开始执行界面跳转操作

- 控制器间的数据传递
- 顺传
- 数据方向：sourceViewcontroller->destinationViewController
- 传递方式：在源控制器`sourceViewcontroller`的`prepareForSegue:sender:`方法中根据segue参数取得`destinationViewController`, 也就是目标控制器，直接给控制器赋值
- 逆传
- 数据方向：destinationViewController->sourceViewcontroller
- 传递方式：让源控制器`sourceViewcontroller`成为目标控制器`destinationViewController`的代理，在目标控制器`destinationViewController`中调用源控制器`sourceViewcontroller`的代理方法，通过代理方法的参数传递数据给源控制器
---
####通讯录
- 结构分析
- 登录界面
1. 登录按钮只有文本框都有文字才能点击
2. 自动登录开关和记住密码开关的协调
3. 文本框占位符，提示输入信息
4. 密码文本框是暗文
5. 文本框输入文字后会有清除按钮
6. 点击登录会判断账号和密码是否正确，只有正确才跳转，提示用户正在登录，模拟网络延迟
- 联系人列表界面
1. 联系人列表导航条标题跟账号有关系，控制器之间传值
2. 注销按钮回到登录界面
3. 添加按钮，进入联系人编辑界面
- 添加联系人界面
1. 默认弹出姓名的文本框，不需要用户点击文本框弹出键盘
2. 添加按钮，只有文本框都有文字才能点击
3. 点击添加后，回到联系人界面，并把数据显示到联系人列表
- 编辑联系人界面
1. 点击联系人界面的cell，进入编辑界面
2. 默认保存按钮是隐藏的，默认文本框不能交互，是查看联系人
3. 点击编辑按钮的时候，文本框才会允许交互，并且保存按钮默认不能点击
4. 默认弹出电话文本框，一般修改联系人都是修改电话
5. 编辑状态，点击取消按钮，恢复文本框数据回到查看联系人
6. 编辑状态，点击保存，提示用户保存，并显示到联系人界面
- 界面较少且固定，采用storyboard开发，使用导航控制器
- 具体代码在 **07-通讯录**

---
#### 数据存储
- plist存储
- 存数据
- 数据存储是保存在手机里面，保存到应用的沙盒中
- plist文件一般存储的都是字典和数组，直接写成plist文件，保存到沙盒中
- 每个应用在手机中都有一个文件夹，通过下面的方法获取当前应用的手机里安装的路径
```objc
NSString *homeDir = NSHomeDirectory();
```
```objc
/**
* 在某个范围内搜索文件夹的路径
* 方法会得到一个数组（可能会有多个路径）
* directory：获取哪个文件夹
* domainMask：在哪个路径下搜索
* expandTilde：是否展开路径
*/
NSArray *arr = NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserdomainMask, YES);
// 在这里我们搜索的是Cache目录，结果只有一个
NSString *cachePath = arr[0];
// 拼接文件路径
NSString *filePathName = [cachePath stringByAppendingPathComponent:@"agePlist.plist"];
// 保存字典
NSDictionary *dict = @{@"age" : @18,@"name" : @"gaowei"};
[dict writeToFile:filePathName atomically:YES];
// 保存数组
NSArray *dataArray = @[@56,@"asdfa"];
[dataArray writeToFile:filePathName atomically:YES];
```
- 读取文件
- 与写基本相同，都是通过路径完成
```objc
// 同样搜索路径
NSArray *array = NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUs erDomainMask, YES);
// 拿到路径
NSString *cachePath = array[0];
// 拼接⽂件路径
NSString *filePathName = [cachePath stri ngByAppendingPathComponent:@"agePlist.plist"];
// 从⽂件当中读取字典
NSDictionary *dict = [NSDictionary dict ionaryWithContentsOfFile:filePathName];
// 从⽂件当中读取数组
NSArray *dataArray = [NSArray arrayWithContentsOfFile:filePathName];
NSLog(@"%@",dataArray);
```
- 偏好设置
- NSUserDefaults
- 写数据
```objc
// 默认在目录下生成一个字典，并以plist文件的方式储存
// 不用关心文件名，快速进行键值对存储
NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];
[defautls setObject:@"sleen" forKey:@"name"];
[defautls setBool:YES forKey:@"isBool"];
[defautls setInteger:5 forKey:@"num"];
//同步，⽴即写⼊文件.
[defautls synchronize];
```
- 读数据
```objc
// 存的时候用什么key存的, 读的时候就要用什么key值取
// 存的时候用什么类型,取的时候也要⽤什么类型.
NSString *str = [[NSUserDefaults standardUserDefaults] objectForKey:@"name"];
BOOL isBool =[[NSUserDefaultsstandardUserDefaults]
boolForKey:@"isBool"];
NSInteger num = [[NSUserDefaults standardUserDefaults]
integerForKey:@"num"];
NSLog(@"name =%@-isBool=%d-num=%ld",str,isBool,num);
```
- 归档
- 归档一般都是保存自定义对象的时候，plist文件不能完成需求，如果一个字典中保存有自定义对象，如果要把字典写到文件中，是不会写成plist文件的
- 保存数据
```objc
// 自定义的对象
SLPerson *person = [[SLPerson alloc] init];
person.name = @"sleen";
person.age = 18;
// 获取沙盒临时目录
NSString *tempPath = NSTemporaryDirectory();
NSString *filePath = [tempPath stringByAppendingPathComponent:@"person.data"];
// archiveRootObject这个⽅法底层会去调用保存对象的encodeWithCoder⽅法，去
询问要保存这个对象的哪些属性
[NSKeyedArchiver archiveRootObject:person toFile:filePath];
```
- 读取数据
```objc
// 获取沙盒临时目录
NSString *tempPath = NSTemporaryDirectory();
NSString *filePath = [tempPath stringByAppendingPathComponent:@"persion.data"];
// NSKeyedUnarchiver会调用initWithCoder这个方法，来让你告诉它去获取这个对
象的哪些属性
Person *person = [NSKeyedUnarchiver unarchiveObjectWithFile:filePath];
NSLog(@"name=%@---age=%d",person.name,person.age);
```
- 在自定义的对象中，要遵守`NSCoding`协议
```objc
// SLPerson.h
#import <Foundation/Foundation.h>
@interface Person : NSObject <NSCoding>
@property (nonatomic, strong) NSString *name;
@property (nonatomic, assign) int age;
@end
// SLPerson.m
#import "SLPerson.h"
@implementation SLPerson
- (void)encodeWithCoder:(NSCoder *)encode {
[encode encodeObject:self.name forKey:@"name"];
[encode encodeInt32:self.age forKey:@"age"];
}
- (instancetype)initWithCoder:(NSCoder *)decoder {
if (self = [super init]) {
self.age = [decoder decodeInt32ForKey:@"age"];
self.name = [decoder decodeObjectForKey:@"name"];
}
return self;
}
@end
```
---
#### UITabBarController
- 基本使用
- UITabBarController与UINavigationController类似，管理多个控制器，在底部有一个导航条UITabBar（高度49）
- 添加控制器
```objc
// 两种方法
// 添加单个⼦子控制器
- (void)addChildViewController:(UIViewController *)childController
// 设置⼦子控制器数组
@property(nonatomic,copy) NSArray *viewControllers;
```
- 简单应用
```objc
// 设置窗口
self.window = [[UIwindow alloc] init];
// 初始化tabbar
UITabBarController *tabbar = [[UITabBarController alloc] init];
// 为tabbar添加子控制器
UIViewController *vc1 = [[UIViewController alloc] init];
vc1.view.backgroundColor = [UIColor redColor];
// 设置标题
vc1.tabBarItem.title = @"标题1";
// 设置提醒数字
vc1.tabBarItem.badgeValue = @"10";
[tabBar addChildViewController:vc1];
UIViewController *vc2 = [[UIViewController alloc] init];
vc2.view.backgroundColor = [UIColor yellowColor];
[tabBar addChildViewController:vc2];
UIViewController *vc3 = [[UIViewController alloc] init];
vc3.view.backgroundColor = [UIColor blueColor];
[tabBar addChildViewController:vc3];
// 设置窗口根控制器
self.window.rootViewController = tabbar;
// 显⽰示窗⼝口
[self.window makeKeyAndVisible];
```
- 管理原则
- `TabBarController`默认做法是把它的第⼀个子控制器的View添加到`TabBarController`存放子控制器的View当中
- 如果`TabBarController`有N个⼦控制器，那么`TabBar`内部就会有N个按钮
- 点击每一个按钮, 它会先把当前控制器的View从`TabBarController`存放⼦控件View的View当中移除(只是移除view，子控制器还在数组当中，没有被移除)
- 再把当前选中按钮对应子控制器的View添加到`TabBarController`存放⼦控件View中
- 主流框架搭建
- 窗口的根控制器为`UITabBarController`，再将`UINavigationController`添加到`UITabBarController`中









