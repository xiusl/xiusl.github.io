---
title: 曾经的ios02
date: 2015-10-02 16:20:00
categories: 
- technology
- Objective-C 
tags: ios
---


1. 使用注释 // 要空一格在写

2. 要有一定的编程思想，在对象方法和类方法都计算两点距离时，可以在对象方法（类方法）中调用类方法（对象方法）
<!-- more -->
3. 返回值时BOOL类型时，方法或函数用is开头

4. ＃pragm mark – 一段注释

  ＃pragm mark 一段注释，可以在快速查看方法的地方看到注释

5. 点语法：是指实质就是方法的调用，要注意在类中使用点语法不要引发死循环

6. 成员变量作用域：@public在任何地方都能直接访问对象的成员变量；@private 只能在当前类中直接访问（@implementation中如果声明比变量，默认就是@private）；@protected 在当前类和子类中可以直接访问（@interface 中就是默认的@protected）

7. @property 可以自动生成某个成员变量的setter和getter方法的声明

8. @synthesize 可以自动生成某个成员变量setter和getter方法的实现@synthesize age = _age;会访问_age变量，如果不写=_age就会访问age变量；如果类中没有当前要访问的变量，就会自动生成一个（类型为 @private）

9. 在新的版本中，@property的功能更强大：在@interface中直接写 “@property int age；”就会自动有一个­­_age的成员变量，并且生成他的setter和getter方法的声明和实现

1. id万能指针，能指向、操作任何OC对象

2. 构造方法：用来初始化对象的方法，是一个对象方法，-开头，可以在对象初始的时候成员变量就有一定的值；重写的时候要调用父类的构造方法，在进行子类对象的初始化

3. 自定义构造方法：对象方法，以 – 开头，返回值一般为 id 类型，方法名以 initWith开头

4. 分类：在不改变原来类内容的基础上，为类增加一些新的方法；分类只能增加方法，不能增加成员变量／分类的实现可以访问类中声明的成员变量／分类可以重新实现原类的方法，但会覆盖掉原类的方法／方法的调用顺序：分类，原类，父类

5. 分类：可以为系统的类添加分类。网上有很多写好的分类，拓展了系统类的功能。

6. 类的本质：类也是一个对象，是class类型的对象——类­­­­对象，类＝＝类对象，每个类只有一个类对象；获取内存中的类对象：Class c = [Person class];

7. ＋load方法：在程序启动时，就会加载项目中所有的类和分类，而加载后就会调用＋load方法，只调用一次

8. ＋initialize方法：在第一次使用某个类时，就会调用当前类的＋initialize方法

9. 注意：先加载父类，后加载子类－－先调用父类的＋load方法，在调用子类的＋load方法；先初始化父类，后初始化子类——先调用父类的＋initialize方法，在调用子类的＋initialize方法

10. －description方法：NSLog(@”%d”,p); 就会调用该方法；默认情况下，利用NSLog 和 ％@输出对象时，结果显示<</span>类名:内存地址>；在类中对－description进行重写，让其返回类的所有属性值，

11. ＋description方法：Class c = [Person class]; NSLog(@”%d”,c); 就会调用该方法，

12. SEL 类型：每个类方法列表都存储在类对象中，每个方法都有一个与之对应的SEL类型的对象，根据SEL对象就可以找到方法的地址从而调用方法；

13. _cmd ：代表当前方法

昨天太晚了，宿舍没网就没能贴上今天补上
- OC内存管理

1. 每一个Oc对象都有一个引用计数器（占4 个字节），用来保存有多少人在使用该对象，当使用alloc、new、copy创建一个新对象时，对象的引用计数器默认为1，当计数器的值＝0时，对象就会被回收

2. 操作计数器的一些对象方法：

  retain方法： 计数器加1，会返回对象本身；

  release方法： 计数器减1，没有返回值；

  dealloc方法： 返回当前计数器值（ld类型）

3. dealloe方法：当一个对象被回收的时候就会调用，一定要调用会父类的dealloc（[super dealloc];）放在最后面

4. 几个概念：

  僵尸对象：所占用的内存已经被回收的对象，不能在进行使用，

  野指针：指向僵尸对象（不可用内存）的指针，给野指针发送消息（调用方法）就会报错（EXC_BAD_ACCESS）

  空指针：没有指向任何东西的指针（nil、NULL、0），空指针调用方法不会报错

- 多对象的内存管理

5. 要使用某个对象的时候，就要让对象的计数器加1（让对象做一次retain操做）；不想在使用莫个对象时，就要让对象的计数器减1（让对象做一次release操作）

6. 谁retain 谁release；谁alloc 谁release

- 类的set方法和dealloc方法

7. 基本数据类型：直接赋值

8. Oc对象类型：先判断是不是新对象，如果是，对旧对象做一次release，在对新对象做一次retain并赋值

9. dealloc方法：要先对当前对象拥有的所有Oc对象进行一次release操作，最后调用[super dealloc];

- @property的参数

10. retain：生成的set方法里面，release旧值，retain新值（用于Oc对象）

    assign：生成的set方法里面，直接赋值（用于非Oc对象，默认的）

    copy：release旧值，copy新值

11. readonly：只生成get方法（只读）

    readwrite：同时生成set和get方法（默认的）

12. 多线程管理

    nonatomic：性能高（一定要写这个）

    atomic：性能低（默认的）

13. set和get方法名

    setter重新定义set的方法名；getter重新定义get的方法名；（一般set不会改，当get方法返回BOOL时会改为is开头）

- 循环引用（你中有我，我中有你）

14. @class xxx：告诉编译器xxx这是一个类，不会包含xxx类的任何内容

15. 开发中：在 .h 文件中使用@class来声明类，在 .m 文件中使用#import来包含类的所有东西

16. 两端循环的解决办法：在@property中，一端用retain，一端用assign

- autorelease方法

17. 将对象放到一个自动释放池中，当自动释放池被销毁时，会对池子中的所有对象进行一次release操作

18. 会返回对象本身，调用后，对象的计数器数并不变

19. 开发中，经常会将
	```
    [[[Car alloc] init] autorelease]封装到一个类方法（car）中；要是想要带初始值的（carWithSpeed：//方法中创建对象使用self）
   ```
 


