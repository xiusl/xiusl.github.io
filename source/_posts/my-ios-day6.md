---
title: my-ios-day6
date: 2016-09-01 16:00:42
categories: 
- 技术
- Objective-C 
tags: 
- ios
- gitbook
---

这个是16年重新复习的iOS笔记，当时是放在了[gitbook](https://sleenxiu.gitbooks.io/my-learning-ios/content/)，现在转移过来！
<!-- more -->

# day 06
## UITableView
#### 数据源
- 数据源简介
- tableView需要一个数据源来显示数据
- tableView会向数据源查询有多少行数据，每行数据是什么
- 凡是遵守UITableViewDataSource协议的OC对象，都可以成为数据源
- 数据源协议
```objectivec
// 一共有多少组数据
- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView;
// 每一组有多少行数据
- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section;
// 每一行显示什么内容
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath;
```
---
####常见设置
```objectivec
// 设置tableView每一行cell的高度，默认是44
self.tableView.rowHeight = 80;
// 设置tableView每⼀一组头部的⾼高度
￼self.tableView.sectionHeaderHeight = 50;
￼// 设置tableView每⼀一组尾部的⾼高度
￼self.tableView.sectionFooterHeight = 50;
￼// 设置分割线的颜色，如果设置[UIColor clearColor]隐藏分割线
￼self.tableView.separatorColor = [UIColor redColor];
￼// 设置分割线的样式
￼self.tableView.separatorStyle = UITableViewCellSeparatorStyleNone;
￼// 设置表头
￼self.tableView.tableHeaderView = [[UISwitch alloc] init] ;
￼// 设置表尾
￼self.tableView.tableFooterView = [UIButton
￼buttonWithType:UIButtonTypeContactAdd];
￼// 设置索引条上⽂文字颜⾊色
￼self.tableView.sectionIndexColor = [UIColor redColor];
￼// 设置索引条的背景颜⾊色
￼self.tableView.sectionIndexBackgroundColor = [UIColor blackColor];
```
---
####代理
```objectivec
/**
* 当选中某一行cell就会调用
*/
￼- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:￼(NSIndexPath *)indexPath{}
￼/**
￼* 当取消选中某⼀一⾏行cell就会调⽤用
￼*/
￼- (void)tableView:(UITableView *)tableView didDeselectRowAtIndexPath:￼(NSIndexPath *)indexPath{}
/**
* 返回每一组显示的头部控件
*/
- (UIView *)tableView:(UITableView *)tableView viewForHeaderInSection:(NSInteger)section{}
/**
* 返回每一组显示的尾部控件
*/
- (UIView *)tableView:(UITableView *)tableView viewForFooterInSection:(NSInteger)section{}
/**
* 返回每一组头部的高度
*/
￼- (CGFloat)tableView:(UITableView *)tableView heightForHeaderInSection:(NSInteger)section{}
/**
* 返回每一组尾部的高度
*/
￼- (CGFloat)tableView:(UITableView *)tableView heightForFooterInSection:(NSInteger)section{}
/**
* 返回tableView的每一行高度
*/
￼- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:￼(NSIndexPath *)indexPath
```
---
#### UITableViewCell
- 简介
- tableView的每一行都是一个UITableViewCell，通过dataSource的tableView: cellForRowAtIndexPath:方法初始化每一行
- UITableViewCell内部有一个默认的子视图contentView，contentView是cell内部所有控件的父控件
- 常见设置
```objectivec
// 设置cell右边的指⽰示控件
￼cell.accessoryView = [[UISwitch alloc] init];
￼// 设置cell右边的指⽰示样式(accessoryView优先级 > accessoryType)
￼cell.accessoryType = UITableViewCellAccessoryDisclosureIndicator;
￼// 设置cell的背景view
￼// backgroundView优先级 > backgroundColor
￼UIView *bg = [[UIView alloc] init];
￼bg.backgroundColor = [UIColor blueColor];
￼cell.backgroundView = bg;
￼// 设置cell的背景颜⾊色
￼cell.backgroundColor = [UIColor redColor];
￼// 设置cell选中的背景view
￼UIView *selectbg = [[UIView alloc] init];
￼selectbg.backgroundColor = [UIColor purpleColor];
￼cell.selectedBackgroundView = selectbg;
￼// 设置cell选中的样式
cell.selectionStyle = UITableViewCellSelectionStyleNone;
```
---
#### cell重用
- 第一种
```objectivec
- (void)viewDidLoad {
[super viewDidLoad];
￼// 根据ID 这个标识 注册对应的 cell类型
￼[self.tableView registerClass:[UITableViewCell class] ￼forCellReuseIdentifier:ID];
}
- (UITableViewCell *)tableViewew:(UITableView *)tableView￼cellForRowAtIndexPath:(NSIndexPath *)indexPath {
// 1.首先去缓存池中查找可循环利用的cell
UITableViewCell *cell = [tableview dequeueReusableCellWithIdentifier:ID];
// 2.设置数据
cell.textLabel.text = [NSString stringWithFormat:@"第%ld行数据", indexPath.row];
return cell;
}
```
- 第二种
```objc
- (UITableViewCell *)tableViewew:(UITableView *)tableView￼cellForRowAtIndexPath:(NSIndexPath *)indexPath {
// 1.首先去缓存池中找可以循环利用的cell
UITableViewCell *cell = [tableview dequeueReusableCellWithIdentifier:ID];
// 2.如果缓存池中的cell为空，就创建一个cell
if (cell == nil) {
cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleSubtitle reuseIdentifier:ID];
}
// 3.设置数据
cell.textLabel.text = [NSString stringWithFormat:@"第%ld行数据", indexPath.row];
return cell;
}
```
---
#### 自定义cell
- 自定义等高cell
- **代码方式**
1. 新建一个继承自`UITableViewCell`的子类，如SLOwnCell
```objc
@interface SLOwnCell : UITableViewCell
@end
```
2. 在SLOwnCell.m文件中重写`-initWithStyle:reuseIdentifier:`方法，添加子控件，给控件做一些一次性的初始化（如使用自动布局可在此方法中为控件添加完整约束）
```objc
// 添加所有子控件
- (instancetype)initWithStyle:(UITableViewCellStyle)style reuseIdentifier:(NSString *)reuseIdentifier
{
if (self = [super initWithStyle:style reuseIdentifier:reuseIdentifier]) {
// code...
}
return self;
}
```
3. 重写`-layoutSubviews`方法，一定要调用`[super layoutSubviews]`，在这个方法中计算和设置所有子控件的frame
```objc
// 布局内部子控件
- (void)layoutSubviews
{
[super layoutSubviews];
// code...
}
```
4. 在`SLOwnCell.h`中提供一个模型属性，如SLOwn模型
```objc
@class SLOwn;
@interface SLOwnCell : UITableViewCell
@property (nonatomic, strong) SLOwn *own;
@end
```
5. 在`SLOwnCell.m`中重写模型属性的set方法，同时为cell中的子控件添加数据
```objc
- (void)setOwn:(SLOwn *)own
{
_own = own; // 先要进行set方法的赋值
// 为子控件添加数据...
}
```
6. 在viewController中注册cell，并给cell传递模型
```objc
// 注册cell
[self.tableView registerClass:[SLOwnCell class] forCellReuseIdentifier:ID];
// 传递模型
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
SLOwnCell *cell = [tableView dequeueReusableCellWithIdentifier:ID];
cell.own = self.owns[indexPath.row];
return cell;
}
```
- **xib方式**
1. 新建一个继承自`UITableViewCell`的子类，如SLOwnCell
```objc
@interface SLOwnCell : UITableViewCell
@end
```
2. 新建一个xib文件，文件名最好和类名相同，如`SLOwnCell.xib`
- 修改cell的class为SLOwnCell
- 为cell绑定循环利用标识
- 添加子控件，设置完整约束
- 将子控件拖线到类扩展中
```objc
@interface SLOwnCell ()
@property (weak, nonatomic) IBOutlet UIImageView *iconView;
@property (weak, nonatomic) IBOutlet UILabel *titleLabel;
@property (weak, nonatomic) IBOutlet UILabel *priceLabel;
@end
```
3. 添加模型和重写set方法（与代码方式相同）
4. 在viewController中，需要注册xib文件
```objc
// 注册xib文件
[self.tableView registerNib:[UINib nibWithNibName:NSStringFromClass([XMGTgCell class]) bundle:nil] forCellReuseIdentifier:ID];
```
- **storyboard方式**
- 基本方法与xib相同，只是不需要注册
- 在添加控件的时候，要加入到cell的contentView中
- 自定义不等高cell
- 主要通过代码的方式实现
- 基本思路等高cell相同，主要介绍一些区别支出
1. 需要添加一个frame模型，如SLOwnFrame，
```objc
@class SLOwn;
@interface SLOwnFrame : NSObject
// 头像的frame
@property (nonatomic, assign, readonly) CGRect iconFrame;
// 标题的frame
@property (nonatomic, assign, readonly) CGRect titleFrame;
// 价格的frame
@property (nonatomic, assign, readonly) CGRect priceFrame;
// cell的高度
@property (nonatomic, assign, readonly) CGFloat cellHeight;
// 数据模型
@property (nonatomic, strong) SLOwn *own;
@end
```
2. 在frame模型的`setOwn:`方法中，根据own模型中的数据计算各个控件的frame
3. 在自定义的SLOwnCell文件中包含SLOwnFrame模型
```objc
@class SLOwnFrame;
@interface SLOwnCell : UITableViewCell
@property (nonatomic, strong) SLOwnFrame *ownFrame;
@end
```
4. 在重写`setOwnFrame:`方法的时候需要进行数据和frame的设置
```objc
- (void)setOwnFrame:(SLOwnFrame *)ownFrame
{
_ownFrame = ownFrame;
[self setData];
[self setFrame];
}
- (void)setData
{
// 设置子控件的数据
}
- (void)setFrame {
// 设置子控件的frame
}
```
5. 在viewController中字典转模型后，将own赋值给ownFrame，数组中存储的是`SLOwnFrame`模型
6. 通过数据源的`tableView: heightForRowAtIndexPath:`方法得到每一行cell的高度
```objc
- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath
{
SLOwnFrame *oframe = self.ownFrames[indexPath.row];
return oframe.cellHeight;
}
```





