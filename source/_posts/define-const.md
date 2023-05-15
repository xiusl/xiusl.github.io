---
title: 宏和常量（Objective-C）
date: 2018-08-15 14:41:11
categories:
- 技术
tags:
 
---

对于宏、常量使用的一些笔记
<!--more-->
### 宏

- 定义

    ```
    #define 宏名 语句

    #define PI 3.1415
    ```
- 在程序的预处理阶段，所有使用宏的地方，都会被宏名后面的语句替换
- 多行的情况使用反斜线`\`链接

    ```
    #define SOME "too many \
    chars"
    ```
- 命名是使用大写字母
- 带有参数的宏

    ```
    #define ANHLE2RADIAN(a) (PI*(a)/108)
    ```
- 可变宏`...`、`__VA_ARGS__`

    ```
    #define LOG(x, ...) NSLog(x, ##__VA_ARGS__)
    ```
- 括号()

### 常量
- 使用 const 修饰
- 只会分配一块空间
- 会进行类型检查
- 不能定义方法
- const 位置，一般使用第三种方法定义

    ```
    // *BaseUrl 不能修改，BaseUrl 可以修改
    const NSString *BaseUrl = @"http://api.someweb.com";
    // *BaseUrl 不能修改，BaseUrl 可以修改
    NSString const *BaseUrl = @"http://api.comeweb.com";
    // *BaseUrl 可以修改，BaseUrl 不能修改
    NSString *const BaseUrl = @"http://api.comeweb.com";
    ```
- 在`.h`中使用`extern`提供给外部使用

### 比较
- 尽量多使用常量来替换宏定义，在使用宏的时候思考是否可以使用常量来替换
- 宏在预处理阶段会进行文本替换，在宏修改后会重新编译，影响编译速度
- 大量的使用宏会增大二进制文件的大小
- 宏可以定义一些方法
- 宏可以重定义，`xCode`会提示警告



