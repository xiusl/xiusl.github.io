---
title: 曾经的ios05
date: 2015-10-05 16:20:00
categories: 
- technology
- Objective-C 
tags: ios
---

iOS学习07
学习一个新的控件UITableView，下面简称tableview
1-UITableView的数据源和代理，使用tableview显示数据要绑定他的数据源，监听tableview上的tableviewcell事件要指定代理。
<!-- more -->
都有两种方式：设置他的属性dataSource，或者拖线
2-绑定数据源就要遵守数据源的协议并实现相应的方法
3-tableview可以展示单组数据，也可以分组展示数据，tableview有两种样式UITableViewStylePlain和UITableViewStyleGrouped(分组比较明显)
4-协议的几个方法：

```
- (NSInteger)numberOfSectionInTableView:(UITableView *)tableView; //返回一共有多少组数据，传入当前的tableview
- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section;  // 返回每组有多少行数据，传入单前的tableview和当前的组序号
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath;  // 每一行显示的内容是什么，会返回一个UITableViewCell，就是tableview的一行。传入当前的tableview和NSIndexPath(组序号，列序号)
```
5-介绍一下UITableViewCell，他就是UITableView的一行
6-UITableViewCell中有一个默认的子视图：contentView，可以使用cell的accessoryView属性来指定一些辅助视图
7-contentView下还有三个子视图textLabel、detailTextLabel和UIIamgeView，通过设置当前cell的UITableViewCellStyle的属性来决定显示子视图的位置
8-几个小例子

汽车品牌：（没贴模型的代码，模型就是之前学过的，改下属性就好）

```
//
//  ViewController.m
//  1104-tableview01
//
//  Created by 修修 on 15/11/4.
//  Copyright © 2015年 cn.Xsoft. All rights reserved.
//

#import "ViewController.h"
#import "LSCargroup.h"

@interface ViewController () <</span>UITableViewDataSource>
@property (weak, nonatomic) IBOutlet UITableView *tableView;

 
@property (nonatomic, strong) NSArray *cargroups;
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
}
# pragma mark - getter方法懒加载
- (NSArray *)cargroups
{
    if (_cargroups == nil) {
        NSString *path = [[NSBundle mainBundle] pathForResource:@"cars_simple.plist" ofType:nil];
        NSArray *array = [NSArray arrayWithContentsOfFile:path];
        NSMutableArray *temp = [NSMutableArray array];
        for (NSDictionary *dict in array) {
            LSCargroup *group = [LSCargroup cargroupWithDict:dict];
            [temp addObject:group];
        }
        _cargroups = temp;
    }
    return _cargroups;
}

# pragma mark - 数据源方法

- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView
{
    return self.cargroups.count;
}
 
- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section
{
    // 拿到当前的模型，模型中cars数组的长度就是每组的行数
    LSCargroup *ss = _cargroups[section];
    return ss.cars.count;
}

 
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    static NSString *ID = @"car";
    // 性能优化
    // 先在缓存池中寻找有没有可以循环利用的cell，按照标识ID来寻找
    UITableViewCell *cell = [self.tableView dequeueReusableCellWithIdentifier:ID];
    // 要是找不到，就要自己去创建一个
    if (cell == nil) {
        cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:ID];
    }
    // 给cell添加数据
    // 拿到模型
    LSCargroup *ss = _cargroups[indexPath.section];
    NSString *carname = ss.cars[indexPath.row];
    // 赋值
    cell.textLabel.text = carname;
    
    return cell;
}
 
- (NSString *)tableView:(UITableView *)tableView titleForHeaderInSection:(NSInteger)section
{
    // 拿到模型
    LSCargroup *ss = _cargroups[section];
    return ss.title;
}
 
- (NSString *)tableView:(UITableView *)tableView titleForFooterInSection:(NSInteger)section
{
    // 拿到模型
    LSCargroup *ss = _cargroups[section];
    return ss.desc;
}
- (BOOL)prefersStatusBarHidden
{
    return YES;
}
@end
一些关键的注释贴不出来
```

```
//
//  ViewController.m
//  1104-tableview02
//
//  Created by 修修 on 15/11/5.
//  Copyright © 2015年 cn.Xsoft. All rights reserved.
//

#import "ViewController.h"
#import "SLHero.h"
@interface ViewController () <</span>UITableViewDataSource, UITableViewDelegate>
 
@property (nonatomic, strong) NSArray *heros;
@property (weak, nonatomic) IBOutlet UITableView *tableView;

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
   
    // 设置每一行的高度，默认44
    self.tableView.rowHeight = 60;
}

#pragma mark - getter方法重写
- (NSArray *)heros
{
    if (_heros == nil) {
        NSString *path = [[NSBundle mainBundle] pathForResource:@"heros.plist" ofType:nil];
        NSArray *herostemp = [NSArray arrayWithContentsOfFile:path];
        NSMutableArray *tempary = [NSMutableArray array];
        for (NSDictionary *dict in herostemp) {
            SLHero *hero = [SLHero heroWithDict:dict];
            [tempary addObject:hero];
        }
        _heros = tempary;
    }
    return _heros;
}

#pragma mark - 数据源方法
// 可以不写，默认就是一组
- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView
{
    return 1;
}

- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section
{
    return self.heros.count;
}

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    // 设置一个cell的唯一标识
    NSString * ID = @"hero";
    // 在缓存池中寻找有没有可以重复利用的cell，并且标识是ID
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:ID];
    // 如果没有找到，就自己创建一个cell，并让他的标识为ID
    if (cell == nil) {
        cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleSubtitle reuseIdentifier:ID];
    }
    // 设置cell显示的值
    // 拿到当前的模型
    SLHero *hero = self.heros[indexPath.row];
    cell.textLabel.text = hero.name;
    cell.detailTextLabel.text = hero.desc;
    cell.imageView.image = [UIImage imageNamed:hero.icon];
    // 返回cell
    return cell;
}

#pragma mark - 代理方法
 
- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath
{
    // 取出当前的模型
    SLHero *hero = self.heros[indexPath.row];
    // 弹框，iOS9中Apple推荐使用的弹框方法，之前的UIAlertView在iOS9不适用
    // 先定义一个弹框，标题、显示的信息、优先选取的style
    UIAlertController *alert = [UIAlertController alertControllerWithTitle:@"英雄信息" message:nil preferredStyle:UIAlertControllerStyleAlert];
    // 在定义弹框的按钮，标题、形式、一个block（用来写按钮的点击方法）
    // 取消按钮
    UIAlertAction *cancelBtn = [UIAlertAction actionWithTitle:@"取消" style:UIAlertActionStyleCancel handler:^(UIAlertAction * _Nonnull action) {
        
    }];
    // 修改按钮
    UIAlertAction *deBtn = [UIAlertAction actionWithTitle:@"修改" style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {
        // 拿到弹框上的textField
        UITextField *textField = alert.textFields[0];
        // 修改模型数据，
        hero.name = textField.text;
        // 局部刷新数据
        [self.tableView reloadRowsAtIndexPaths:@[indexPath] withRowAnimation:UITableViewRowAnimationBottom];
    }];
    // 给弹框添加一个UITextField，显示英雄的名字
    [alert addTextFieldWithConfigurationHandler:^(UITextField *textField) {
        textField.text = hero.name;
    }];
    
    // 将两个按钮放上去
    [alert addAction:cancelBtn];
    [alert addAction:deBtn];
    // 显示弹框
    [self presentViewController:alert animated:YES completion:nil];
}

@end
文档注释贴不出来，，，，
```