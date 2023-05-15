---
title: 曾经的ios06
date: 2015-10-06 16:20:00
categories: 
- 技术
- Objective-C 
tags: ios
---

好久没写了，现在一起看完然后写的总结，代码没有在新浪发，感觉新浪对代码的支持不是很好，所以我把代码都放到了csdn，在这写个链接吧
自定义cell的代码01
自定义cell的代码02
<!-- more -->
iOS学习08
总结UITableViewCell的基本用法

- 使用xib自定义cell
  1. 如果每一行cell都一样，且系统提供的cell不能满足要求，就可以创建xib文件自定义cell，
  2. 为xib中的cell添加一个类，再类中解析xib创建自定义的cell，提供给别人用，还要让类拥有相应的模型，
  3. 在cell的类中实现cell的重用
  4. 在控制器的返回cell的方法中，初始化cell，并为cell传递模型，设置数据
  5. 关于cell重用的问题，当在缓存池中拿到cell的时候，要将控件的状态调整正确
- 使用代码自定义cell
  1. 当每一行的cell的高度不确定，内部显示的内容不确定时，就要使用代码创建我们需要的cell
  2. 创建一个SLOwnCell的类，用来自定义cell，继承自我们系统的UITableViewCell
  3. 通过重写父类的init方法，向cell中添加自己的控件，在这里设置控件不变的数据，（主要思想就是先将要显示的所有控件加进去，之后再根据情况设置是否显示--是否设置控件的Frame）
  4. 提供一个类方法，创建cell在类方法中实现cell的重用，调用init初始化cell
  5. 每个cell中的控件数据不同，所以每个控件位置和每个cell的高度就不一定，所以我们要创建一个位置模型（SLXxxFrame）
  6. 这个位置模型中包含数据模型，和之后每个控件的（CGRect）xxxFrame
  7. 在计算frame的时候，要计算文本在屏幕的位置，NSString中有一个对象方法，
  8. 在初始化cell后为SLOwnCell类中的位置模型传递值，在位置模型的setter方法的重写中，为控件设置位置和数据
- QQ聊天
  1. 也是用代码创建cell，就说一下几个重要的
  2. 相同时间就显示一个：在消息模型中声明一个BOOL型标识sameTime。在控制器的模型的getter方法中，使用定义的可变数组可以取出上一个元素，与当前时间判断，相同就让sameTime =YES；在设置time控件的Frame的时候，就不设置值。
  3. 聊天气泡：涉及到图片的拉伸，要保证图片的边缘不变，就要把中心放大，UIImage提供了一个方法
 4. 通知机制：

```
// 通知：NSNotification，一般包含三个属性：(NSString *)name通知的名称、(id)object通知的发布者、(NSDictionary *)userInfo通知的一些额外信息；初始化一个通知
+ (instancetype)notificationWithName:(NSString *)aName object:(id)anObject;
+ (instancetype)notificationWithName:(NSString *)aName object:(id)anObject userInfo:(NSDictionary *)aUserInfo;
- (instancetype)initWithName:(NSString *)name object:(id)object userInfo:(NSDictionary *)userInfo;
>通知中心：NSNotificationCenter，管理通知的接收，注册监听器；发布通知：
- (void)postNotification:(NSNotification *)notification; // 发布一个notification通知，可在notification对象中设置通知的名称、通知发布者、额外信息等
 

- (void)postNotificationName:(NSString *)aName object:(id)anObject; // 发布一个名称为aName的通知，anObject为这个通知的发布者
 

- (void)postNotificationName:(NSString *)aName object:(id)anObject userInfo:(NSDictionary *)aUserInfo;// 发布一个名称为aName的通知，anObject为这个通知的发布者，aUserInfo为额外信息

// 监听器：用来接收消息，监听是否发出我想要的消息，注册监听器，
- (void)addObserver:(id)observer selector:(SEL)aSelector name:(NSString *)aName object:(id)anObject;
observer：监听器，即谁要接收这个通知
aSelector：收到通知后，回调监听器的这个方法，并且把通知对象当做参数传入
aName：通知的名称。如果为nil，那么无论通知的名称是什么，监听器都能收到这个通知
anObject：通知发布者。如果为anObject和aName都为nil，监听器都收到所有的通知
一般在监听器销毁之前取消注册（如在监听器中加入下列代码）：
- (void)dealloc {

  //[super dealloc];  非ARC中需要调用此句

    [[NSNotificationCenter defaultCenter] removeObserver:self];

}
```

  5. UIDecive的通知：通过[UIDevice currentDevice]可以获取这个单粒对象
UIDecive会不断地发布一些通知：设备旋转、电池状态改变、电池电量改变、近距离传感器
  6. 键盘的通知：

```
>UIKeyboardWillShowNotification // 键盘即将显示
>UIKeyboardDidShowNotification // 键盘显示完毕
>UIKeyboardWillHideNotification // 键盘即将隐藏
>UIKeyboardDidHideNotification // 键盘隐藏完毕
>UIKeyboardWillChangeFrameNotification // 键盘的位置尺寸即将发生改变
>UIKeyboardDidChangeFrameNotification // 键盘的位置尺寸改变完毕
```

  7. 发送聊天消息：键盘属于调出他的那个对象（UITextFiled），通过他的一些属性可以修改键盘的一些属性，在UITextFiled的代理中有监听键盘点击事件的代理方法，可以让控制器称为代理，完成发送消息
- 静态cell
将stroyboard中的viewController换成tableViewController，并让tableViewController的类指向默认的SLViewController，修改其中的tableview，让他变成静态，添加静态的cell






