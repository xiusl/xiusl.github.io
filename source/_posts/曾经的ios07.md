---
title: 曾经的ios07
date: 2015-10-07 16:20:00
categories: 
- 技术
- Objective-C 
tags: ios
---

这些天一直在忙期末复习的事，耽误了好些天，接下的考试周，课少了就可以有更多时间看视频和写代码了，下面是今天写的一点点。
<!-- more -->
iOS学习09
几个常用的控件
1206-PickerVIew  城市选择
数据源：UIPickerViewDataSource
设置有多少列和多少行
- (NSInteger)numberOfComponentsInPickerView:(UIPickerView *)pickerView;
- (NSInteger)pickerView:(UIPickerView *)pickerView numberOfRowsInComponent:(NSInteger)component;
代理：UIPickerViewDelegate
设置每一列每一行显示的是什么内容
- (CGFloat)pickerView:(UIPickerView *)pickerView widthForComponent:(NSInteger)component;
- (CGFloat)pickerView:(UIPickerView *)pickerView rowHeightForComponent:(NSInteger)component;

- (NSString *)pickerView:(UIPickerView *)pickerView titleForRow:(NSInteger)row forComponent:(NSInteger)component;
- (UIView *)pickerView:(UIPickerView *)pickerView viewForRow:(NSInteger)row forComponent:(NSInteger)component reusingView:(UIView *)view;

- (void)pickerView:(UIPickerView *)pickerView didSelectRow:(NSInteger)row inComponent:(NSInteger)component;


刷新数据
(void)reloadAllcomponent;// 所有列
(void)reloadComponent:(NSInteger)component;//指定列
主动选择第component列和第row行
(void)selectRow:(NSInterger)row inComponent:(NSInteger)component animated:(BOOL)animated;
得到第component列的选中行
(NSInteger)selectdRowInComponent:(NSinteger)component;
1206-UIDatePicker  

@property (nonatomic) UIDatePickerMode datePickerMode;// datePicker的显示模式

@property (nonatomic, retain) NSLocale   *locale;// 显示的区域语言



2.监听UIDatePicker的选择

* 因为UIDatePicker继承自UIControl,所以通过addTarget:...监听



1206-Picker作业（综合，键盘工具条）

通过textfield的inputView属性更改键盘

textFieid的inputAccessoryView属性设置键盘上面的工具条

注：加深对代理使用的理解

自己写这些就是想以后可以有东西复习，并不能给比别人带来什么，主要是为了自己，一直坚持吧！


跳一节，上一个写的太散了，就先不放这里了
iOS11
一、控制器的多种创建过程
控制器SLViewController：UIVIewController
*代码创建：
SLViewController *con = [SLViewController alloc] init];// init直接成为根控制器
*storyboard创建（不使用main.storyboard）
先加载storyboard文件
UIStoryboard *sb = [UIStoryboard storyboardWithName:@“sl” bundle:nil];
然后初始化storyboard中的控制器
SLViewController *con = [sb instantiateInitialViewController];//storyboard箭头指向的控制器
SLViewController *con = [sb instantiateViewControllerWithindetifier:@“sl”];//标识为sl的控制器
*xib文件创建
SLViewController *con = [[SLViewcontroller alloc] initWithNibName:@“SLViewController”];
二、控制器View的创建（SLViewController）
if loadView ：根据oadView的代码创建；
else if storyboard ：根据storyboard描述创建
else if nibName：根据nibName对应的xib创建
else if SLView.xib：根据SLView.xib创建
else if SLViewController.xib：根据SLViewController.xib创建
else :创建一个空的view
控制器的view是延时加载的，
三、多控制器的管理
*UINavigationController
在控制器上方会有一个导航栏（设置）
>初始化UINavigationController
>设置UIWindow的根控制器为UINavigationController
>使用push方法添加子控制器
UINavigationController以栈(先进后出)的形式保存子控制器，总是显示栈顶控制器的view
NSArray *viewControllers;/NSArray *childViewController;
使用push方法，把子控制器压入栈
- (void)pushViewController:(UIViewController)viewController animated:(BOOL)animated;
使用pop方法可以移除子控制器
- (UIViewController *)popViewControllerAnimated:(BOOL)animated;// 移除栈顶的控制器
- (NSArray *)popToViewController:(UIViewController *)viewController animated:(BOOL)animated;// 回到指定的控制器
- (NSArray *)popToRootViewControllerAnimated:(BOOL)animated;// 回到根控制器
导航栏的使用
显示的内容由栈顶控制器的navigationItem属性决定
UINavigationItem有以下属性：
@property(nonatomic, retain)UIBarButtonItem *backBarButtonItem;// 左上角的返回按钮
@property(nonatomic, retain)UIView *titleView;// 中间的标题视图
@property(nonatomic, copy)NSString *title;// 中间标题文字
@property(nonatomic, retain)UIBarButtonItem *leftBarButtonItem;// 左上角视图
@property(nonatomic, retain)UIBarButtonItem *rightBarButtonItem;// 右上角视图
使用storyboard时可以直接拖线实现跳转，在两个控制器之间会有一条线，这条线是一个UIStoryboardSegue对象(segue)
每个segue都有三个属性：identifier(唯一标识)、sourceViewController(来源控制器)、destinationViewController(目标控制器)
segue有自动跳转和手动跳转
>当点击某个控件不需要做一些事情的事后，就可以直接拖线，实现跳转
>如果需要在跳转前做一些判断(登陆)，需要手动跳转。将来源控制器拖线到目标控制器，为segue绑定唯一标识，执行[self performSegueWithIdentifier:@“login2other” sender:nil];
控制器之间的数据传递
顺传：
- (void)prepareForSegue:(UIStoryboardSegue *)segue sender:(id)sender;// 在准备跳转前调用，通过segue拿到目标控制器，各塔传递一些数据
逆传：
通过代理，让来源控制器成为目标控制器的代理，当目标控制器点击出栈的时候通知代理，并将相应的数据传过去
*UITabBarController
在控制器的底部有一个导航条（QQ）
>初始换UITabBarController
>设置UIWindow的根控制器为UITabBarController
>使用addChildViewController添加子控制器
UITabBarController会根据子控制器的个数将导航栏均分成几个部分，点击后就会实现自动跳转 
UITabBarItem决定每个部分的内容，有几个属性：NSString *title(标题文字)、UIImage *image(图标)、UIImage *selectedImage(选中图标)、NSString *badgeValue(提醒数字)
----通常的UI框架是使用一个UITabBarController管理几个UINavigationController，再由每个UINavigationController管理多个子控制器
*Modal显示控制器（多控制器），控制器会在底部向上钻出
- (void)presentViewController:(UIViewController *)viewControllerToPresent animated:(BOLL)flag completion:(void (^) (void))completion;// 展示控制器
- (void)dismissViewContollerAnimated:(BOOL)flag completion(void (^) (void))completion;// 退出控制器
一般modal方法显示的控制器应该与当前控制器没有太多的关联
四、应用程序目录结构（应用沙盒）
Documents：保存应用运行时生成的需要持久化的数据，iTune同步设备时会备份数据
tmp：保存应用运行时所需要的临时数据，系统会自动清理该目录文件，iTunes不会被分该目录
LIbrary/Caches：保存应用运行时生成的需要持久化的数据，iTunes不会备份该目录，一般是体积较大的文件，不需备份的非重要数据
Library/Preference：保存应用的偏好设置，iTunes会备份该目录
五、沙盒目录获取
NSHomeDirectory();
stringByAppendingPathComponment:@“”;//路径拼接
六、数据存储
1.通过plist文件存储（xml）
>获得路径
>writeToFile:atomically:，只能归档NSString、NSArray、NSDictionary、NSData、NSNumber
>arrayWithContentOfFile:……
2.通过preference（偏好设置）
>单粒对象NSUserDefaults *def = [NSUserDefault standardUserDefaults];
>[def setObject: forKey:];
>[def stringForKey];…… 
>[def synchornize];强制写入文件中（强退）
3.NSKeyedArchiver归档（NSCoding）
>获得路径
>[NSKeyedArchiver archiverRootObject: toFile: ];// 归档
>[NSKeyedUnrchiver unarchiverRootObjectWithFile: ];// 读取
>第一种方式的类型可以直接归档，自定义的类需要遵守协议并实现相应的方法才能使用
>在encodeWithCoder:中使用encodeObject:forKey:方法归档实例
>在initWithCoder:中使用decodeObject:forKey:方法解析实例
>存在继承关系的归档，父类归档自己的属性，子类重写时调用父类的方法，再归档自己的属性
4.SQLite3 // 嵌入式数据库，后面讲
5.Core Data // 对SQLite3的封装，C语言API