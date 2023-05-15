---
title: my-ios-day16
date: 2016-09-01 16:12:07
categories: 
- 技术
- Objective-C 
tags: 
- ios
- gitbook
---

这个是16年重新复习的iOS笔记，当时是放在了[gitbook](https://sleenxiu.gitbooks.io/my-learning-ios/content/)，现在转移过来！
<!-- more -->

# day 16
## 彩票项目
- √ 主流框架
- √ 新特性界面
- √ 设置界面
---
#### 新特性界面
- UICollectionView
- 拥有和UITableView一样的重用机制，但需要自己布局cell
- 系统提供UICollectionViewFlowLayout类来布局cell
- 使用步骤
1. 设置布局参数
```objc
UICollectionViewFlowLayout *flowLayout = [[UICollectionViewFlowLayout alloc] init];
flowLayout.itemSize = [UIScreen mainScreen].bounds.size;
// 滚动方向(水平)
flowLayout.scrollDirection = UICollectionViewScrollDirectionHorizontal;
// items 和items之间的间距
flowLayout.minimumInteritemSpacing = 0;
// 行间距
flowLayout.minimumLineSpacing = 0;
```
2. 注册cell
```objc
[self.collectionView registerClass:[XMGNewFeatureCollectionViewCell class] forCellWithReuseIdentifier:reuseIdentifier];
```
- 新特性
- 添加NewFeature文件夹
- 自定义SLCollectionViewController : UICollectionVieController
- 布局collectionView
```objc
- (instancetype)init {
// 创建流水布局
UICollectionViewFlowLayout *flowLayout = [[UICollectionViewFlowLayout alloc] init];
// 行间距
flowLayout.minimumLineSpacing = 0;
// 列间距
flowLayout.minimumInteritemSpacing = 0;
// 每个cell的尺寸
flowLayout.itemSize = [UIScreen mainScreen].bounds.size;
// 滚动方向，水平滚动
flowLayout.scrollDirection = UICollectionViewScrollDirectionHorizontal;
return [super initWithCollectionViewLayout:flowLayout];
}
```
- 注册cell，设置属性，添加图片
```objc
- (void)viewDidLoad {
[super viewDidLoad];
// Register cell classes
[self.collectionView registerClass:[SLNewFeatureCollectionViewCell class] forCellWithReuseIdentifier:reuseIdentifier];
// 设置分页
self.collectionView.pagingEnabled = YES;
// 取消弹簧效果
self.collectionView.bounces = NO;
// 取消滚动条
self.collectionView.showsHorizontalScrollIndicator = NO;
// 添加图片
[self addImageView];
}
- (void)addImageView {
// guide1
UIImageView *guideImageView = [[UIImageView alloc] initWithImage:[UIImage imageNamed:@"guide1"]];
[self.collectionView addSubview:guideImageView];
guideImageView.x += 50;
self.guideImageView = guideImageView;
// guideLine
UIImageView *guideLineImageView = [[UIImageView alloc] initWithImage:[UIImage imageNamed:@"guideLine"]];
[self.collectionView addSubview:guideLineImageView];
guideLineImageView.x -= 150;
// guideLargeText1
UIImageView *guideLargeTextImageView = [[UIImageView alloc] initWithImage:[UIImage imageNamed:@"guideLargeText1"]];
[self.collectionView addSubview:guideLargeTextImageView];
guideLargeTextImageView.center = CGPointMake(self.collectionView.width * 0.5f, self.collectionView.height * 0.7);
self.guideLargeTextImageView = guideLargeTextImageView;
// guideSmallText1
UIImageView *guideSmallTextImageView = [[UIImageView alloc] initWithImage:[UIImage imageNamed:@"guideSmallText1"]];
[self.collectionView addSubview:guideSmallTextImageView];
guideSmallTextImageView.center = CGPointMake(self.collectionView.width * 0.5f, self.collectionView.height * 0.8);
self.guideSmallTextImageView = guideSmallTextImageView;
}
```
- 设置图片的动画
```objc
// 当scrollView 减速的时候调用
- (void)scrollViewDidEndDecelerating:(UIScrollView *)scrollView{
// 当前偏差
CGFloat offsetX = scrollView.contentOffset.x;
// 计算页码
NSInteger page = offsetX / scrollView.width + 1;
// 换图片
NSString *name = [NSString stringWithFormat:@"guide%ld",page];
UIImage *image = [UIImage imageNamed:name];
self.guideImageView.image = image;
// 大标题
NSString *largeTextname = [NSString stringWithFormat:@"guideLargeText%ld",page];
UIImage *largeTextimage = [UIImage imageNamed:largeTextname];
self.guideLargeTextImageView.image = largeTextimage;
// 小标题
NSString *smallTextname = [NSString stringWithFormat:@"guideSmallText%ld",page];
UIImage *smallTextimage = [UIImage imageNamed:smallTextname];
self.guideSmallTextImageView.image = smallTextimage;
// 偏差值
CGFloat del = offsetX - self.lastOffsetX;
self.guideImageView.x += del * 2;
self.guideLargeTextImageView.x += del * 2;
self.guideSmallTextImageView.x += del * 2;
[UIView animateWithDuration:0.25f animations:^{
self.guideImageView.x -= del;
self.guideLargeTextImageView.x -= del;
self.guideSmallTextImageView.x -= del;
}];
// 记录上次偏差
self.lastOffsetX = offsetX;
}
```

- 设置cell样式
```objc
- (UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath {
SLNewFeatureCollectionViewCell *cell = [collectionView dequeueReusableCellWithReuseIdentifier:reuseIdentifier forIndexPath:indexPath];
NSString *imageName = [NSString stringWithFormat:@"guide%ldBackground",indexPath.item + 1];
cell.image = [UIImage imageNamed:imageName];
// 判断是否是最后一个cell
[cell setIndexPath:indexPath count:SLPageCount];
return cell;
}
```
- 自定义cell，添加bg图片、立即体验
1. 提供图片属性
```objc
/** 背景图片 */
@property (nonatomic, strong) UIImage *image;
```
2. 添加bgImageView
```objc
- (UIImageView *)bgImageView {
if (!_bgImageView) {
_bgImageView = [[UIImageView alloc] initWithFrame:self.bounds];
[self.contentView addSubview:_bgImageView];
}
return _bgImageView;
}
```
3. 重写image的set方法
```objc
- (void)setImage:(UIImage *)image {
_image = image;
// 设置image
self.bgImageView.image = image;
}
```
4. 提供设置立即体验方法
```objc
- (void)setIndexPath:(NSIndexPath *)indexPath count:(NSInteger)count {
// 当时最后一个cell的时候，添加立即体验按钮
if (indexPath.item + 1 == count) {
// 添加立即体验按钮
self.startBtn.hidden = NO;
}else { // 不是最后一个cell
// 隐藏立即体验按钮
self.startBtn.hidden = YES;
}
}
```
5. 立即体验按钮
```objc
- (UIButton *)startBtn {
if (_startBtn == nil) {
_startBtn = [[UIButton alloc] init];
[_startBtn setBackgroundImage:[UIImage imageNamed:@"guideStart"] forState:UIControlStateNormal];
[_startBtn sizeToFit];
// 一定要在按钮有尺寸的时候，再调整位置
_startBtn.center = CGPointMake(self.contentView.width * 0.5f, self.contentView.height * 0.9f);
[_startBtn addTarget:self action:@selector(startBtnOnclick:) forControlEvents:UIControlEventTouchUpInside];
[self.contentView addSubview:_startBtn];
}
return _startBtn;
}
// 开始按钮点击调用的方法
- (void)startBtnOnclick:(UIButton *)button {
SLTabBarController *tabBarVc = [[SLTabBarController alloc] init];
// 切换跟控制器
SLKeyWindow.rootViewController = tabBarVc;
}
```
- 工具类
- 在Other文件夹中添加Tool文件夹
- 数据存数工具SLSaveTool
```objc
// SLSaveTool.h
@interface SLSaveTool : NSObject
+ (id)objectForKey:(NSString *)defaultName;
+ (void)setObject:(id)value forKey:(NSString *)defaultName;
@end
// SLSaveTool.m
@implementation SLSaveTool
+ (id)objectForKey:(NSString *)defaultName {
return [[NSUserDefaults standardUserDefaults] objectForKey:defaultName];
}
+ (void)setObject:(id)value forKey:(NSString *)defaultName {
// 偏好设置不能存储nil
if (defaultName) {
// 保存当前版本
[[NSUserDefaults standardUserDefaults] setObject:value forKey:defaultName];
// 立即同步
[[NSUserDefaults standardUserDefaults] synchronize];
}
}
@end
```
- 根控制器选择工具SLRootVCTool
```objc
// SLRootVCTool.h
@interface SLRootVCTool : NSObject
/**
* 选择窗口的跟控制器
*/
+ (UIViewController *)chooseWindowRootVc;
@end
// SLRootVCTool.m
@implementation SLRootVCTool
+ (UIViewController *)chooseWindowRootVc {
UIViewController *rootVc;
// 取出当前版本
NSString *currVerson = [NSBundle mainBundle].infoDictionary[@"CFBundleShortVersionString"];
// 取出上一次保存的版本
NSString *odlVerson = [SLSaveTool objectForKey:SLVerson];
// 判断有无新版本
if ([currVerson isEqualToString:odlVerson]) {
// 没有新版本，进入主框架
// 设置窗口的根控制器
rootVc = [[SLTabBarController alloc] init];
}else {
// 有新版本
// 显示新特性界面
rootVc = [[SLNewFeatureViewController alloc] init];
[SLSaveTool setObject:currVerson forKey:SLVerson];
}
return rootVc;
}
@end
```
---
#### 设置界面
- 分析
- 界面较多，但内部结构基本相同，
- 使用纯代码，并依据MVC开发
- 添加Setting文件夹
- 设置组模型`SLSettingGroup`
```objc
// SLSettingGroup.h
@interface SLSettingGroup : NSObject
/** cell模型数组 */
@property (nonatomic, strong) NSArray *items;
/** 头部标题 */
@property (nonatomic, copy) NSString *headerTitle;
/** 尾部标题 */
@property (nonatomic, copy) NSString *footTitle;
/**
* 通过cell模型数组返回组模型
*/
+ (instancetype)groupWithItems:(NSArray *)items;
@end
// SLSettingGroup.m
@implementation SLSettingGroup
+ (instancetype)groupWithItems:(NSArray *)items{
SLSettingGroup *group = [[SLSettingGroup alloc] init];
group.items = items;
return group;
}
@end
```
- 设置cell模型
- 基本的cell，
```objc
// SLSettingItem.h
@interface SLSettingItem : NSObject
/** 图标 */
@property (nonatomic, strong) UIImage *image;
/** 标题 */
@property (nonatomic, copy) NSString *title;
/** 子标题 */
@property (nonatomic, copy) NSString *subTitle;
/** 点击这行cell要做的事情 */
@property (nonatomic, strong) void(^operation)(NSIndexPath *indexPath);
+ (instancetype)itemWithImage:(UIImage *)image title:(NSString *)title;
+ (instancetype)itemWithTitle:(NSString *)title;
@end
// SLSettingItem.m
@implementation SLSettingItem
+ (instancetype)itemWithImage:(UIImage *)image title:(NSString *)title{
SLSettingItem *item = [[self alloc] init];
item.image = image;
item.title = title;
return item;
}
+ (instancetype)itemWithTitle:(NSString *)title{
return [self itemWithImage:nil title:title];
}
@end
```
- 箭头和开关cell
```objc
// 带有箭头的cell是可以跳转狭义控制器的
// 跳转目标控制器 1.绑定类名字符串 2.绑定类名
// SLSettingArrowItem.h
@interface SLSettingArrowItem : SLSettingItem
/** 目标控制器类名 */
@property (nonatomic, assign) Class desVc;
@end
// 带有开关的cell是要有开关状态的
// SLSettingSwitchItem.m
@interface SLSettingSwitchItem : SLSettingItem
/** 状态 */
@property (nonatomic, assign, getter=isOpen) BOOL open;
@end
```
- 设置基础控制器`SLBaseTableViewController`
- 所有控制器都应使用`UITableViewController`且为组模式，结构基本相同，提取设置基础控制器
- 代码创建组类型，要重写init方法
```objc
- (instancetype)init{
return [super initWithStyle:UITableViewStyleGrouped];
}
```
- 基础控制器包含一个组数据（告诉控制器有多少组，且组中包含cell模型数组）
```objc
@interface SLBaseTableViewController : UITableViewController
/** 控制器的所有组 */
@property (nonatomic, strong) NSMutableArray *groups;
@end
```
- 数据源和代理
```objc
- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView {
return self.groups.count;
}
- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section {
// 取出组数组的当前组
SLSettingGroup *group = self.groups[section];
return group.items.count;
}
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
SLSettingTableViewCell *cell = [SLSettingTableViewCell cellWithTableView:tableView];
// 从总数组中取出组模型
SLSettingGroup *group = self.groups[indexPath.section];
// 从cell模型数组中取出当前cell模型
SLSettingItem *item = group.items[indexPath.row];
// 传递模型
cell.item = item;
return cell;
}
- (NSString *)tableView:(UITableView *)tableView titleForHeaderInSection:(NSInteger)section{
// 取出组模型
SLSettingGroup *group = self.groups[section];
return group.headerTitle;
}
- (NSString *)tableView:(UITableView *)tableView titleForFooterInSection: (NSInteger)section {
// 取出组模型
SLSettingGroup *group = self.groups[section];
return group.footTitle;
}
- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath {
[tableView deselectRowAtIndexPath:indexPath animated:YES];
// 从总数组中取出组模型数组
SLSettingGroup *group = self.groups[indexPath.section];
// 从行模型数组中取出cell模型
SLSettingItem *item = group.items[indexPath.row];
if (item.operation) {
item.operation(indexPath);
}else if ([item isKindOfClass:[SLSettingArrowItem class]]) {
// 只有剪头模型才具备跳转功能
SLSettingArrowItem *arrowItem = (SLSettingArrowItem *)item;
if (arrowItem.desVc) {
// 如果有目标控制器
// 拿到目标控制器类名 创建目标控制器
UIViewController *vc = [[arrowItem.desVc alloc] init];
vc.title = arrowItem.title;
[self.navigationController pushViewController:vc animated:YES];
}
}
}
```
- 设置控制器`SLSettingTableViewController`
- 添加模型
```objc
- (void)viewDidLoad {
[super viewDidLoad];
self.title = @"设置";
// 添加第0组
[self setupGroup0];
// 添加第1组
[self setupGroup1];
// 添加第2组
// ...
}
/**
* 第0组
*/
- (void)setupGroup0 {
SLSettingArrowItem *item = [SLSettingArrowItem itemWithImage:[UIImage imageNamed:@"RedeemCode"] title:@"使用兑换码"];
__weak typeof(self) weakSelf = self;
// 使用item的operation跳转控制器
item.operation = ^(NSIndexPath *indexPath) {
UIViewController *vc = [[UIViewController alloc] init];
vc.title = @"asdf";
vc.view.backgroundColor = [UIColor redColor];
[weakSelf.navigationController pushViewController:vc animated:YES];
};
// 创建组模型
SLSettingGroup *group = [SLSettingGroup groupWithItems:@[item]];
group.headerTitle = @"123";
group.footTitle = @"qeqrere";
// 添加到总数组中
[self.groups addObject:group];
}
/**
* 第1组
*/
- (void)setupGroup1 {
SLSettingArrowItem *item1 = [SLSettingArrowItem itemWithImage:[UIImage imageNamed:@"MorePush"] title:@"推送和提醒"];
// 箭头特有属性desVc实现跳转
item1.desVc = [SLPushTableViewController class];

SLSettingSwitchItem *item2 = [SLSettingSwitchItem itemWithImage:[UIImage imageNamed:@"handShake"] title:@"使用摇一摇机选"];
item2.open = YES;
SLSettingSwitchItem *item3 = [SLSettingSwitchItem itemWithImage:[UIImage imageNamed:@"sound_Effect"] title:@"声音效果"];
SLSettingSwitchItem *item4 = [SLSettingSwitchItem itemWithImage:[UIImage imageNamed:@"More_LotteryRecommend"] title:@"购彩小助手"];
// 创建组模型
SLSettingGroup *group = [SLSettingGroup groupWithItems:@[item1, item2, item3, item4]];
group.headerTitle = @"qwe";
group.footTitle = @"slll";
// 添加到总数组中
[self.groups addObject:group];
}
// ...
// 添加...
```
- 其他的控制器与设置控制器相同，只需创建并添加相应的组模型
- block简单介绍
- 封装一段代码，在合适的时候调用
- 两种定义方法（没有参数和返回值）
```objc
// 1. 第一种方法 这里的block是变量名
void(^block)() = ^() {
// 要保存的代码块...
};
// 2. 起别名的方式
// 这里的MyBlock是类型名，这个可以跨方法执行
typedef void(^MyBlock)();
@property (nonatomic, strong) MyBlock block;
```
- block可以当做参数来传递
- `inline`快速创建block
- 设置界面回顾
- MVC设计模式的强大，更改需求只需修改模型，不需要再修改view的显示代码




