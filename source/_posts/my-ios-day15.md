---
title: my-ios-day15
date: 2016-09-01 16:12:02
categories: 
- 技术
- Objective-C 
tags: 
- ios
- gitbook
---

这个是16年重新复习的iOS笔记，当时是放在了[gitbook](https://sleenxiu.gitbooks.io/my-learning-ios/content/)，现在转移过来！
<!-- more -->

# day 15
## 彩票项目
- √ 主流框架
- x 新特性界面
- x 设置界面
---
#### 主流框架搭建
- 主流框架
- 一个UITabBarController管理多个UINavigationController
- 框架使用纯代码方式
- 文件按照模块管理，内部为MVC模式
- 主要流程
- 创建单视图程序，配置项目基本信息
1. 删除系统创建的main.storyboard和ViewController
2. 配置General项删除Main Interface
3. 配置禁止屏幕旋转，lunch界面隐藏状态栏
4. 为AppIcon添加图片，给LunchScreen.storyboard设置图片 5. 按模块创建文件夹
- 在AppDelegate.m中初始化window
```objc
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
// 设置window
self.window = [[UIWindow alloc] initWithFrame:[UIScreen mainScreen].bounds];
// 添加根控制器
SLTabBarController *tabBar = [[SLTabBarController alloc] init];
self.window.rootViewController = tabBar;
// 让window显示
[self.window makeKeyAndVisible];
return YES;
}
```
- 直接使用了自定义的SLTabBarController，在SLTabBarController中添加子控制器
- 按模块分别创建自己的控制器
```objc
#import "SLHallTableViewController.h"
#import "SLArenaViewController.h"
#import "SLDiscoverTableViewController.h"
#import "SLHistoryTableViewController.h"
#import "SLMyLotteryViewController.h"
```
- 自定义TabBar
1. 移除系统tabBar中的所有item
```objc
// 在View将要出现的时候调用
- (void)viewWillAppear:(BOOL)animated {
[super viewWillAppear:animated];
for (UIView *view in self.tabBar.subviews) {
NSString *classStr = NSStringFromClass([view class]);
// if ([classStr hasPrefix:@"UITabBarButton"]) {
// [view removeFromSuperview];
// }
// if ([view isKindOfClass:[UIControl class]]) {
// [view removeFromSuperview];
// }
if(![view isKindOfClass:[SLTabBar class]]) {
[view removeFromSuperview];
}
}
}
```
2. 添加自定义SLTabBar
```objc
SLTabBar *tabBar = [[SLTabBar alloc] initWithFrame:self.tabBar.bounds];
tabBar.items = self.items;
tabBar.delegate = self;
[self.tabBar addSubview:tabBar];
```
3. 抽取添加控制器方法
```objc
- (void)setController:(UIViewController *)vc withImage:(NSString *)image selectImage:(NSString *)sImage title:(NSString *)title{
// 包装导航控制器
UINavigationController *nav =[[SLNavController alloc] initWithRootViewController:vc];
// 竞技场的导航控制器不同
if ([vc isKindOfClass:[SLArenaViewController class]]) {
nav = [[SLArenaNavController alloc] initWithRootViewController:vc];
}
[self addChildViewController:nav];
// 设置一些数据
vc.title = title;
vc.tabBarItem.image = [UIImage imageNamed:image];
vc.tabBarItem.selectedImage = [UIImage imageNamed:sImage];
// 保存数据，供SLTabBar使用
[self.items addObject:vc.tabBarItem];
}
```
4. 为SLTabBar保存相应数据值
```objc
@interface SLTabBar()
/** 保存系统原有tabBarItem */
@property (nonatomic, strong) NSMutableArray *items;
@end
@implementation SLTabBarController
// 实现懒加载
- (NSMutableArray *)items {
if (_items == nil) {
_items = [NSMutableArray array];
}
return _items;
}
@end
```
5. SLTabBar.m根据items中添加按钮，layoutSubViews布局
```objc
- (void)setItems:(NSArray *)items {
_items = items;
for (UITabBarItem *item in items) {
SLTabBarButton *btn = [SLTabBarButton buttonWithType:UIButtonTypeCustom];
[btn setBackgroundImage:item.image forState:UIControlStateNormal];
[btn setBackgroundImage:item.selectedImage forState:UIControlStateSelected];
[btn addTarget:self action:@selector(btnClick:) forControlEvents:UIControlEventTouchDown];
[self addSubview:btn];
}
}
- (void)layoutSubviews {
CGFloat btnW = self.width / self.items.count;
CGFloat btnH = self.height;
CGFloat btnY = 0;
CGFloat btnX = 0;
for (int i= 0; i < self.items.count; i++) {
SLTabBarButton *btn = self.subviews[i];
btnX = btnW * i;
btn.frame = CGRectMake(btnX, btnY, btnW, btnH);
btn.tag = i;
if (i == 0) {
btn.selected = YES;
self.curButton = btn;
}
}
}
```
6. 监听代按钮点击，并使用代理切换控制器
```objc
// 代理（SLTabBar.h）
@protocol SLTabBarDelegate <NSObject>
@optional
- (void)tabBar:(SLTabBar *)tabBar changeIndex:(NSInteger)index;
@end
// 按钮点击（SLTabBar.m）
- (void)btnClick:(SLTabBarButton *)btn {
self.curButton.selected = NO;
btn.selected = YES;
self.curButton = btn;
if ([self.delegate respondsToSelector:@selector(tabBar:changeIndex:)]) {
[self.delegate tabBar:self changeIndex:btn.tag];
}
}
// 代理实现（SLTarBarController.m）
#pragma mark - tabBar代理方法
- (void)tabBar:(SLTabBar *)tabBar changeIndex:(NSInteger)index {
self.selectedIndex = index;
}
```
- 自定义导航控制器SLNavController
- 在initialize中做一些全局设置
```objc
+ (void)initialize {
// 当时这个类本身第一次加载的时候调用一次
// 其子类加载时不调用
if (self == [SLNavController self]) {
// 获取全局导航条
UINavigationBar *navBar = [UINavigationBar appearanceWhenContainedInInstancesOfClasses:@[self]];
// 导航栏背景图片
[navBar setBackgroundImage:[UIImage imageNamed:@"NavBar64"] forBarMetrics:UIBarMetricsDefault];
// 导航栏标题属性
NSMutableDictionary *dict = [NSMutableDictionary dictionary];
dict[NSFontAttributeName] = [UIFont systemFontOfSize:22];
dict[NSForegroundColorAttributeName] = [UIColor whiteColor];
[navBar setTitleTextAttributes:dict];
// 主题色
navBar.tintColor = [UIColor whiteColor];
// 移除返回按钮文字
UIBarButtonItem *item = [UIBarButtonItem appearanceWhenContainedInInstancesOfClasses:@[self]];
[item setBackButtonTitlePositionAdjustment:UIOffsetMake(0, -64) forBarMetrics:UIBarMetricsDefault];
}
}
```
- 在各自的控制器中设置导航条左右按钮
```objc
// SLMyLotteryViewController.h
- (void)viewDidLoad {
[super viewDidLoad];
UIButton *btn = [UIButton buttonWithType:UIButtonTypeCustom];
[btn setImage:[UIImage imageNamed:@"FBMM_Barbutton"] forState:UIControlStateNormal];
[btn setTitle:@"客服" forState:UIControlStateNormal];
[btn sizeToFit];
self.navigationItem.leftBarButtonItem = [[UIBarButtonItem alloc] initWithCustomView:btn];
self.navigationItem.rightBarButtonItem = [[UIBarButtonItem alloc] initWithImage:[UIImage imageNamed:@"Mylottery_config"] style:UIBarButtonItemStylePlain target:self action:@selector(configClick)];
}
```
- 竞技场导航条设置
```objc
// SLArenaNavController.h
+ (void)initialize {
if (self == [SLArenaNavController self]) {
UINavigationBar *navBar = [UINavigationBar appearanceWhenContainedInInstancesOfClasses:@[self]];
// 导航栏背景图片
[navBar setBackgroundImage:[UIImage resizeImageName:@"NLArenaNavBar64"] forBarMetrics:UIBarMetricsDefault];
// 导航栏标题
NSMutableDictionary *dict = [NSMutableDictionary dictionary];
dict[NSFontAttributeName] = [UIFont systemFontOfSize:22];
dict[NSForegroundColorAttributeName] = [UIColor whiteColor];
[navBar setTitleTextAttributes:dict];
// 主题色
navBar.tintColor = [UIColor whiteColor];
// 移除返回按钮文字
UIBarButtonItem *item = [UIBarButtonItem appearanceWhenContainedInInstancesOfClasses:@[self]];
[item setBackButtonTitlePositionAdjustment:UIOffsetMake(0, -64) forBarMetrics:UIBarMetricsDefault];
}
}
// SLArenaViewController.m
- (void)viewDidLoad {
[super viewDidLoad];
UISegmentedControl *seg = [[UISegmentedControl alloc] initWithItems:@[@"足球", @"篮球"]];
[seg setTitleTextAttributes:@{
NSFontAttributeName : [UIFont systemFontOfSize:18],
NSForegroundColorAttributeName : [UIColor whiteColor]
}
forState:UIControlStateNormal];
[seg setBackgroundImage:[UIImage imageNamed:@"CPArenaSegmentBG"] forState:UIControlStateNormal barMetrics:UIBarMetricsDefault];
[seg setBackgroundImage:[UIImage imageNamed:@"CPArenaSegmentSelectedBG"] forState:UIControlStateSelected barMetrics:UIBarMetricsDefault];
seg.tintColor = SLColor(52, 137, 140, 1);
seg.selectedSegmentIndex = 0;
self.navigationItem.titleView = seg;
}
```
- 每个控制器的逻辑
- `SLHallTableViewController`（谁创建谁移除）
1. 导航条左边按钮
```objc
- (void)viewDidLoad {
[super viewDidLoad];
self.navigationItem.leftBarButtonItem = [[UIBarButtonItem alloc]initWithImage:[UIImage imageWithOriginal:@"CS50_activity_image"] style:UIBarButtonItemStylePlain target:self action:@selector(leftClick)];
}
```
2. 左边按钮点击事件，添加popMenu
```objc
- (void)leftClick {
[SLMaskView show];
SLPopMenu *popMenu = [SLPopMenu popMenuShowInCenter:self.view.center];
popMenu.delegate = self;
}
```
3. SLPopMenu代理
```objc
- (void)popMenu:(SLPopMenu *)menu cancelBtnClick:(UIButton *)btn {
[menu hiddenInCenter:CGPointMake(44, 44) completion:^{
[UIView animateWithDuration:0.25 animations:^{
[SLMaskView hidden];
}];
}];
}
```

4. 自定义蒙版
```objc
// SLMaskView.h
@interface SLMaskView : UIView
+ (void)show;
+ (void)hidden;
@end
// SLMaskView.m
@implementation SLMaskView
// static SLMaskView *mask;
+ (void)show {
SLMaskView *mask = [[SLMaskView alloc] initWithFrame:[UIScreen mainScreen].bounds];
mask.backgroundColor = [UIColor blackColor];
mask.alpha = 0.7;
[SLKwyWindow addSubview:mask];
}
+ (void)hidden {
for (UIView *view in SLKwyWindow.subviews) {
if ([view isKindOfClass:[self class]]) {
[view removeFromSuperview];
}
}
// [mask removeFromSuperview];
// mask = nil;
}
@end
```
5. 自定义popMenu
```objc
// SLPopMenu.h
@class SLPopMenu;
@protocol SLPopMenuDelegate <NSObject>
@optional
- (void)popMenu:(SLPopMenu *)menu cancelBtnClick:(UIButton *)btn;
@end
@interface SLPopMenu : UIView
/**
* 创建一个SLPopMenu，
* center：popMenu的中心位置
*/
+ (instancetype)popMenuShowInCenter:(CGPoint)center;
/**
* 隐藏popMenu
* center：popMenu隐藏到哪里
* completion：隐藏后做一些事情
*/
- (void)hiddenInCenter:(CGPoint)center completion:(void(^)())completion;
/** 代理 */
@property (nonatomic, weak) id<SLPopMenuDelegate> delegate;
@end
```
```objc
// SLPopMenu.m
@implementation SLPopMenu
+ (instancetype)popMenuShowInCenter:(CGPoint)center {
SLPopMenu *popMenu = [[[NSBundle mainBundle] loadNibNamed:NSStringFromClass([self class]) owner:nil options:nil] lastObject];
popMenu.center = center;
[SLKwyWindow addSubview:popMenu];
return popMenu;
}
- (IBAction)cancelBtnClick:(UIButton *)btn {
if ([self.delegate respondsToSelector:@selector(popMenu:cancelBtnClick:)]) {
[self.delegate popMenu:self cancelBtnClick:btn];
}
}
- (void)hiddenInCenter:(CGPoint)center completion:(void (^)())completion {
[UIView animateWithDuration:0.5 animations:^{
self.transform = CGAffineTransformMakeScale(0.01, 0.01);
self.center = center;
} completion:^(BOOL finished) {
[self removeFromSuperview];
if (completion) {
completion();
}
}];
}
```
- `SLArenaViewController`（loadView方法）
```objc
- (void)loadView {
UIImageView *imView = [[UIImageView alloc] initWithFrame:[UIScreen mainScreen].bounds];
imView.image = [UIImage imageNamed:@"NLArenaBackground"];
imView.userInteractionEnabled = YES;
self.view = imView;
}
```
- `SLDiscoverTableViewController`（storyboard，cell动画）
1. 界面固定使用storyboard创建
```objc
UIStoryboard *storyboard = [UIStoryboard storyboardWithName:NSStringFromClass([SLDiscoverTableViewController class]) bundle:nil];
SLDiscoverTableViewController *discover = [storyboard instantiateInitialViewController];
```
2. cell的动画
```objc
- (void)tableView:(UITableView *)tableView willDisplayCell:(UITableViewCell *)cell forRowAtIndexPath:(NSIndexPath *)indexPath {
cell.transform = CGAffineTransformMakeTranslation(self.view.width, 0);
[UIView animateWithDuration:0.5 animations:^{
cell.transform = CGAffineTransformIdentity;
}];
}
```
3. 按钮内部控件位置调整
```objc
// SLTitleButton.m
// 调整所有子控件的位置
- (void)layoutSubviews{
[super layoutSubviews];
if (self.imageView.x < self.titleLabel.x) { // 第一次调整两个控件的位置
// 这个方法调用了两次,这个位置调整了两次
// 1.调整label的位置
self.titleLabel.x = self.imageView.x;
// 2.调整imageView的位置
self.imageView.x = CGRectGetMaxX(self.titleLabel.frame);
}
}
- (void)setTitle:(NSString *)title forState:(UIControlState)state {
// 做完自己的事情之后 还原系统原有的方法
[super setTitle:title forState:state];
// 增加自己的方法
// 自动计算尺寸
[self sizeToFit];
}
- (void)setImage:(UIImage *)image forState:(UIControlState)state {
[super setImage:image forState:state];
[self sizeToFit];
}
```
4. 手指滑动切换控制器
```objc
// SLNavController.m
- (void)viewDidLoad {
[super viewDidLoad];
UIScreenEdgePanGestureRecognizer *gesture = (UIScreenEdgePanGestureRecognizer *)self.interactivePopGestureRecognizer;
NSArray *targets = [gesture valueForKeyPath:@"_targets"];
// 取出target
id target = [targets[0] valueForKeyPath:@"_target"];
// 禁止系统的
self.interactivePopGestureRecognizer.enabled = NO;
UIPanGestureRecognizer *pan = [[UIPanGestureRecognizer alloc] initWithTarget:target action:@selector(handleNavigationTransition:)];
[self.view addGestureRecognizer:pan];
pan.delegate = self;
}
// 返回YES收拾可以交互, 返回NO不能交互
- (BOOL)gestureRecognizerShouldBegin:(UIGestureRecognizer *)gestureRecognizer {
// != 1 非跟控制器 > 1
return self.childViewControllers.count > 1;
}
```
5. 隐藏TabBar底部的导航条
```objc
// SLNavController.m
- (void)pushViewController:(UIViewController *)viewController animated:(BOOL)animated {
// 一定要写在前面才有用，
if (self.childViewControllers.count != 0) {
viewController.hidesBottomBarWhenPushed = YES;
}
[super pushViewController:viewController animated:animated];
}
```





