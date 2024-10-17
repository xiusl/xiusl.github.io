---
title: iOS 中的 UITest
date: 2019-10-18 15:11:41
tags: [objc, test]
caegories: technology
---

在 iOS 开发中使用 UITest 对各种点击进行测试，避免上线后出现低级 bug。
<!-- more -->

在创建项目的时候可以勾选 `Include UI Tests` 初始化，或者在项目的 `Targets` 中新增 `UITesting Bundle`

项目中会出现相应的文件夹 `xxUITests`，和一个临时的代码文件 `xxUITests.m`

```
#import <XCTest/XCTest.h>

@interface XXUITests : XCTestCase

@end

@implementation XXUITests
- (void)setUp {
    // 在一个测试失败后不再继续
    self.continueAfterFailure = NO;

    // 创建 XCUIApplication 实例，并启动
    // 测试必须要确保在每个测试前启动它
    [[[XCUIApplication alloc] init] launch];
}
- (void)tearDown {
    
}
- (void)testExample {
    // 测试代码


}
@end
```

UITest 是基于下面三个类的实现
```
XCUIApplication
XCUIElement
XCUIElementQuery
```

可以使用录制的方式来进行测试
将鼠标定位 `testExample` 的代码块中，点击下面的红色录制按钮，应用会重启，进行操作后会自动生成测试代码



UITest 的基本原则

- 查找到对应元素（XCUIElementQuery）
- 明确元素的预期行为
- 点击元素触发响应
- 确认预期行为和响应结果是否匹配（assertion）


查找元素

UI对象会有四个属性，不同控件会有不同的默认值，可以在项目中自定义，

- accessibilityTitle
- accessibilityPlaceholderValue
- accessibilityLabel
- accessibilityValue




