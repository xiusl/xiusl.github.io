---
title: my-ios-day7
date: 2016-09-01 16:00:45
categories: iOS
tags: 
- ios
- gitbook
---

这个是16年重新复习的iOS笔记，当时是放在了[gitbook](https://sleenxiu.gitbooks.io/my-learning-ios/content/)，现在转移过来！
<!-- more -->

# day 07
## UITableView
#### 简单的MVC
- MVC是一种程序设计思想，在iOS开发中十分重要
- MVC中的三个角色
- M：Model，模型
- V：View，视图
- C：Control，控制中心
- MVC的特征和体现
- View上面显示的什么，取决于Model
- 只要Model中的数据修改，View的显示状态会跟着改变
- Control负责初始化Model，并将Model传递给View去解析展示

#### 数据刷新
- 全局刷新
```objc
// 所有屏幕上的cell都会刷新
[self.tableView reloadData];
```
- 局部刷新
```objc
// 插入，
- (void)insertRowsAtIndexPaths:(NSArray<NSIndexPath *> *)indexPaths withRowAnimation:(UITableViewRowAnimation)animation;
// 删除
- (void)deleteRowsAtIndexPaths:(NSArray<NSIndexPath *> *)indexPaths withRowAnimation:(UITableViewRowAnimation)animation;
// 刷新部分cell
- (void)reloadRowsAtIndexPaths:(NSArray<NSIndexPath *> *)indexPaths
```
- 左滑删除
```objc
/**
* 只要实现这个方法，就拥有左滑删除的功能
* 点击左滑出现的Delete，就会调用这个方法
*/
- (void)tableView:(UITableView *)tableView commitEditingStyle:(UITableViewCellEditingStyle)editingStyle forRowAtIndexPath:(NSIndexPath *)indexPath;
/**
* 修改默认Delete按钮的文字
*/
- (NSString *)tableView:(UITableView *)tableView titleForDeleteConfirmationButtonForRowAtIndexPath:(NSIndexPath *)indexPath
{
return @"删除";
}
```
- 左滑出现多个按钮
<br/>实现tableView的代理方法
```objc
/**
* 只要实现了这个方法，左滑出现按钮的功能就有了
* 出现按钮后，tableView就会进入编辑模式
*/
- (void)tableView:(UITableView *)tableView commitEditingStyle:(UITableViewCellEditingStyle)editingStyle forRowAtIndexPath:(NSIndexPath *)indexPath
{

}
/**
* 左滑出现什么按钮
*/
- (NSArray *)tableView:(UITableView *)tableView editActionsForRowAtIndexPath:(NSIndexPath *)indexPath
{
UITableViewRowAction *action0 = [UITableViewRowAction rowActionWithStyle:UITableViewRowActionStyleNormal title:@"置顶" handler:^(UITableViewRowAction *action, NSIndexPath *indexPath) {
NSLog(@"点击了置顶");
// 置顶代码...
// 收回左滑出现的按钮(退出编辑模式)
tableView.editing = NO;
}];
UITableViewRowAction *action1 = [UITableViewRowAction rowActionWithStyle:UITableViewRowActionStyleDefault title:@"删除" handler:^(UITableViewRowAction *action, NSIndexPath *indexPath) {
[self.wineArray removeObjectAtIndex:indexPath.row];
[tableView deleteRowsAtIndexPaths:@[indexPath] withRowAnimation:UITableViewRowAnimationAutomatic];
}];
return @[action1, action0];
}
```
- 编辑模式
- 进入编辑模式
```objc
[self.tableView setEditing:YES animated:YES];
```
- 编辑模式多选
```objc
// 允许多选
self.tableVew.allowMultipleSelectDuringEditing = YES;
// 进入编辑模式
[self.tableView setEditing:YES animated:YES];
// 获得选中的所有行
self.tableView.indexPathsForSelectedRows;
```














