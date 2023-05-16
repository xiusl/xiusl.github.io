---
title: my-ios-day3
date: 2016-09-01 16:00:33
categories: 
- technology
- Objective-C 
tags: 
- ios
- gitbook
---

这个是16年重新复习的iOS笔记，当时是放在了[gitbook](https://sleenxiu.gitbooks.io/my-learning-ios/content/)，现在转移过来！
<!-- more -->

##回顾
- UIImageView图片的两种加载方式
没有缓存
NSString *file = [[NSBundle mainBundle] pathForResource:@"图片名" ofType:@"图片扩展名"];
UIImage *image = [UIImage imageWithContentOfFile:file];
*只要方法名带有file的，都是传全路径
使用场合：图片比较大，使用频率比较低
建议：不需要缓存的图片不能放在Assets.xcassets中
放到Assets.xcassets中的图片只能通过图片名去加载，苹果会压缩图片，而且默认带有缓存。
UIImage *image = [UIImage imageNamed:@"图片名"];
- 延迟做一些事情
iOS中有很多方式(GCD,dispatch...)
[abc performSelector:@selector(stand:) withObject:@"123" afterDelay:10];
// 10秒后调用abc的stand：方法，并且传递@“123”参数
- 简单的音频播放
使用AVFoundation框架
AVPlayer *player = [[AVPlayer alloc] init];
NSURL *url = [[NSBundle mainBundle] URLForResource:@"yinyue.mp3" withExtension:nil];
AVPlayerItem *item = [[AVPlayerItem alloc] initWithURL:url];
[player replaceCurrentItemWithPlayerItem:item];
[player play];

# day 03
## 购物车简单实现
![](https://thumbnail0.baidupcs.com/thumbnail/9f9d244f14d78ba7a3acfcb6a4fa45a9?fid=3191038685-250528-979017796389468&time=1472378400&rt=pr&sign=FDTAER-DCb740ccc5511e5e8fedcff06b081203-08Sg5zeqlDKiivuQmaXtVD9Burk%3d&expires=8h&chkbd=0&chkv=0&dp-logid=5583740125953609836&dp-callid=0&size=c10000_u10000&quality=90)
- 九宫格布局
- 横向，y值：不变，x值：（间距 + 宽度）* （下标 % 总列数）
- 纵向，x值：不变，y值：（间距 + 高度）* （下标 / 总行数）
- Plist文件的读取
NSBundle *bundle = [NSBundle mainBundle];
NSString *path = [bundle pathForResource:@"filename" ofType:@"plist"];
_data = [NSArray arrayWithContentsOfFile:path]; // _data保存数据的数组
- 懒加载
```objectivec
- (NSArray *)data
{
if(_data == nil) {
// 初始化data的代码
}
return _data;
}
```
- 面向字典编程
- 将数据以字典的方式存储，存取数据要使用key，容易敲错
- dict[@"name"] = @"Sleen";
- 面向模型编程
- Model，将数据对象封装成一个继承自NSObject的类，数据对象的属性作为成员变量
- 存取数据依靠属性，属性名写错就会直接报错，提高编程效率
- person.name = @"Sleen";
- xib的使用
- 注册
```objectivec
@interface SLXibView : UIView
+ (instancetype)xibView;
@end
@implementation SLXibView
+ (instancetype)xibView {
return [[[NSBundle mainBundle] loadNibNamed:@"SLXibView" owner:nil options:nil] lastobject];
}
```

- 简单注意
- 控件有两种创建方式
1. 通过代码创建，初始化的时候一定会调用initWithFrame:方法
2. 通过xib/storyboard创建，初始化时不会调用initWithFrame:方法，只会调用initWithCoder:方法，初始化完毕会调用awakerFromNib方法
- 有时候需要在初始化时做一些事情，如：添加子控件，设置属性
- 要根据控件的创建方式来选择对应的方法




