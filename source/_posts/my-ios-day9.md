---
title: my-ios-day9
date: 2016-09-01 16:00:51
categories: 
- technology
- Objective-C 
tags: 
- ios
- gitbook
---

这个是16年重新复习的iOS笔记，当时是放在了[gitbook](https://sleenxiu.gitbooks.io/my-learning-ios/content/)，现在转移过来！
<!-- more -->

#day 09
## UIPickerView

#### 简介
- 使用
- 通常应用在注册模块，当需要选择一些固定的选项时使用
- 老虎机效果
- 常见用法
- 单一列，菜单系统
- 关联的，省市选择
- 带图片的，国旗选择
- UIDatePicker
- 系统自带的日期选择

#### 基本使用
- 显示数据
- 与tableView一样，显示数据需要设置数据源和代理
```objc
self.pickerView.dataSource = self;
self.pickerView.delegate = self;
```
- 遵守相应协议
```objc
@interface ViewController () <UIPickerViewDataSource, UIPickerViewDelegate>
@property (weak, nonatomic) IBOutlet UIPickerView *pickerView;
@end
```
- 实现相应方法
```objc
// 总过有多少列
- (NSInteger)numberOfComponentsInPickerView:(UIPickerView *)pickerView{}
// 第component有多少行
- (NSInteger)pickerView:(UIPickerView *)pickerView numberOfRowsInComponent:(NSInteger)component{}
// 返回每一列的宽度
- (CGFloat)pickerView:(UIPickerView *)pickerView widthForComponent: (NSInteger)component{}
// 返回第一列的高度
- (CGFloat)pickerView:(UIPickerView *)pickerView rowHeightForComponent:(NSInteger)component{}
// 返回每⼀⾏的标题
- (nullable NSString *)pickerView:(UIPickerView *)pickerView titleForRow:(NSInteger)row forComponent:(NSInteger)component{}
// 返回每⼀一⾏行的视图UIView
- (UIView *)pickerView:(UIPickerView *)pickerView viewForRow: (NSInteger)row forComponent:(NSInteger)component reusingView: (nullable UIView *)view{}
```
---
#### 简单Demo
- 拦截用户输入—— UITextField的代理
- 控制器成为textField的代理，监听文本框的改变
- 实现代理方法
```objc
// 是否允许开始编辑
- (BOOL)textFieldShouldBeginEditing:(UITextField *)textField {
return YES;
}
// 开始编辑时调⽤用，(弹出键盘)第一响应者
- (void)textFieldDidBeginEditing:(UITextField *)textField {
NSLog(@"弹出键盘");
}
// 是否允许结束编辑
- (BOOL)textFieldShouldEndEditing:(UITextField *)textField {
return YES;
}
// 结束编辑时调⽤用
- (void)textFieldDidEndEditing:(UITextField *)textField {
NSLog(@"结束编辑时调⽤用.");
}
// 是否允许⽂文字改变，拦截⽤用户输⼊入
- (BOOL)textField:(UITextField *)textField shouldChangeCharactersInRange:(NSRange)range replacementString:(NSString *)string {
return NO;
}
```
- 自定义国旗选择
- 自定义省市选择





