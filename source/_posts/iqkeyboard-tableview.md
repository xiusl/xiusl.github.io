---
title: UITableView 中使用 IQKeyboardManager
date: 2019-03-01 19:05:00
tags: [ios]
---

项目中的表单需要在第一次进入的时候自动弹出键盘，使用 `IQKeyboardManager` 来调整输入框的时候，键盘收起后 tableView 底部多出一部分。

<!--more-->

目前觉得主要的问题是，在 `tableView:cellForRowAtIndexPath:` 方法中，调用 `textField` 的 `becomeFirstResponder` 时，`tableView` 的 `contentOfSize` 还没有确定，所以调整 `view` 的偏移量存在误差。

目前的解决办法是在控制器 `viewDidAppear:` 方法中来弹出键盘

```objective-c
- (void)viewDidAppear:(BOOL)animated {
    [super viewDidAppear:animated];
    if (self.textField && self.firstEnter) {
        self.firstEnter = NO;
        [self.textField becomeFirstResponder];
    }
}
```
