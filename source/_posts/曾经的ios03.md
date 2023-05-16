---
title: 曾经的ios03
date: 2015-10-03 16:20:00
categories: 
- technology
- Objective-C 
tags: ios
---

今天的很多，有ARC 还有Oc的一些结构体和类，

- ARC

1. ARC的判断准则：只要没有强指针指向对象，就会释放对象

2. ARC是编译器特性，并不是Oc语言特性
<!-- more -->
3. ARC的特点：不允许在调用与内存管理相关的方法（retain、release、retainCount）；可以重写dealloc方法，但不再调用父类的dealloc方法

4. @property的参数：strong 成员变量是强指针（用于Oc对象）；weak成员变量是弱指针（用于Oc对象）；assing 用于非Oc对象——就是将以前的retain改为strong

5. 强指针：默认情况下，指针都是强类型 __strong； 弱指针：__weak

6. 循环引用：一端用strong 一段用weak

- block

7. 定义block变量： 

   ```            
int (^myBlock) (int, int);
void (^myBlock)();
   ```

8. 使用block封装代码：
   
   ```
   ^(int a, int b){
       return a + b;
   }
   ^(){
       NSLog(@”_____”);
	}
   ```
9. 访问外部变量

   默认是可以访问的，但是不能修改；要修改时，要在声明变量时前面加上__block

10. 使用typedef定义block类型
    
```
typedef int (^MyBlock) (int, int);
MyBlock b;
b = ^(int a, int b){
          return a + b;
}
```
11.   block和指向函数的指针很像
- protocol

12. 协议（protocol）的定义

```
@protocol 协议名称
//方法声明
@end
```
13. 如何遵守协议

   类：在类声明时，在父类后加<</span>协议名1，协议名2>

   协议：就是在协议名后加<</span>协议名1，协议名2>

14. 协议中方法的声明的关键字

   @required（默认）要求方法实现，不实现就会有警告

   @optional 不要求实现，怎么都不会有警告

15. 定义一个变量时，限制变量保存对象遵守某个协议：

   类名<</span>协议名> * 变量名；

   id<</span>协议名> 变量名；

   ——如果保存的变量没有遵守相应的协议就会发出警告

16. 在@property中要求成员变量遵守某一协议：

```
   @property (nonatomic,strong) 类名<</span>协议名> ＊成员名；
   @property (nonatomic,strong) id<</span>协议名> 成员名；
```
17. 协议可以定义在单独的.h文件中，也可以定义在某个类中；（如果协议自由某一个类使用，应该把他定义在该类中；都个类使用一个协议，就要单独写）

- Oc的结构体
18. 包括

```
   NSRange(location length)
   NSPoint\CGPoint
   NSSize\CGSize
   NSRect\CGRect(CGPoint CGSize)
```

```
// range
// @”I love oc”; love 的范围
NSRange r1 = {2, 4};
NSRange r2 = {.location = 2, .length = 4};
NSRange r3 = NSMakeRange(2, 4);   一般都用这个
// 查找某个字符串在str中的位置
NSRange range = [str rangeOfString:@”lo”];
// 如果没有找到length＝0、location＝－1
```

```
// 点
CGPoint p1 = NSMakePoint(10, 10);
NSPoint p2 = CGPointMake(10, 10);
// 长度和宽度
NSSize s1 = NSMakeSize(10,5);
CGSize s2 = CGSizeMake(10, 5);
// 矩形
CGRect r1 = CGRectMake(0, 0, 10, 5);
CGRect r2 = {{0, 0}, (10, 5)};
CGRect r3 = {p1, s1};
```
 

- OC的几个类

19. 字符串类NSString和NSMutableString

```
// 字符串的创建
NSString *s1 = @”Jack”;
NSString *s2 = [[NSString alloc] initWithString:@”Jack”];
NSString *s3 = [[NSString alloc] initWithFormat:@”age is %d”,12];
NSString *s4 = [[NSString alloc] initWithUTF8String:”Jack”;càOc
const char * cs = [s4 UTF8String];
NSString *s5 = [[NSString alloc] initWihtContentsOFFile:@”文件路径” encoding:NSUTF8StringEncoding erroc:nil];
NSString *s6 = [[NSString alloc] initWithContentOfURL:@”资源路径” encoding:NSUTF8StringEncoding error:nil];

// 字符串的文件操作
[@”xiuxiu” writeToFile:@”文件路径” atomically:YES encoding:NSUTF8StringEncoding error:nil];
[@”xiuxiu” writeToURL:@”资源路径” atomically:YES encoding:NSUTF8StringEncoding error:nil];

// 可变字符串
NSMutableString *s = [NSMutableString stringWuthFormat:@”xiuxiu”];
[s appendString:@”shilin”];
// 不可变字符串也可以拼接但会返回一个新的字符串
NSString *s2 =@”xiu”;
NSString *s3 = [s2 stringByAppendingString:@”shilin”];
```

20. 数组类NSArray和NSMutableArray

```
// Oc数组不能存放nil只能放Oc对象，不能放基本数据类型
// 数组创建
NSArray *a1 = [NSArray array];//a1永远都是空的
NSArray *a2 = [NSArray arrayWithObjict:@”Jack”];
NSArray *a3 = [NSArray arrayWithobjects:@”xiu”,@”shi”,nil];// 后面必须加上nil
NSArray *a4 = @[@”xiu”,@”shi”,@”lin”];// 快速创建数组
NSLog(@”%ld”,a3.count);// 数组元素个数
// 元素访问
NSLog(@“%@”,[a3 objectAtIndex:1]);
NSLog(@”%@”,a3[0]);
// 数组的遍历
for(int i=0;i
for(id obj in a) obj;
[a enumerateObjectUsingBlock:^(id obj, NSUInteger idx,BOOL *stop){obj}//stop控制是否遍历
// 可变数组
NSMutableArray *a = [NSmutableArray arrayWithObject:@”rose”, @”jim”, nil];
[a addObject:@”xx”];
[a removeAllObjects];
[a removeObject:@”xx”];
[a removeObjectAtIndex:0];
// 可变数组不能使用快速创建数组的方法
```

21. NSSet:没有顺序，其他与数组一样的

```
NSSet *s = [NSSet set];
NSSet *s2 = [NSSet setWithObjects:@”xiu”,@”shi”,nil];
NSString *s = [s2 anyObject];// 随机取出元素
NSLog(@”%ld”,s2.count);
NSMutableSet *s3 = [NSMutableSet set];
[s3 addObject:@”lin”];// 添加元素
[s3 removeObject:@”xiu”];// 删除
```

22. NSDictionary

```
// 键值对（python中的字典）
// 创建
NSDictionary *d = [NSDictionary dictionaryWithObject:@”jack” forKey:@”name”];
NSArray *k = @[@”name”,@”qq”];
NSArray *obj = @[@”xiu”,@”3213”];
NSDictionary *d2 = [NSDictionary dicationaryWithObjects:obj forKeys:k];
NSDictionary *d3 = [NSDictionary dicationaryWithObjectsAndKeys:@”xiu”:@”name”,@”23123”:@”qq”,nil];
NSDicationary *d4 = @{@”name”:@”xiu”,@”qq”:@”21312”};
id obj = d4[@”name”];//--xiu
d4.count;//键值对的个数
可变字典
NSMutableDicationary *d = [NSMutableDicationary dicationary];
[d setObject:@”xiu” forKey:@”name”];
NSMutableDicationary *d2 = @{@”name”:@”xiu”};//可以快速创建
[d removeObjectForKey:@”name”];
//字典的键不能有相同的
//字典遍历
for (int i=0;i
NSString *key = keys[i]   
NSString *obj = d [key]
[d enumrateKeysAndObjectsUsingBlock:^(id key,id obj,BOOL *stop){key  obj}
```
 

23. NSNumber
 
```
把基本数据类型包装成Oc对象
NSNumber *n = [NSNumber numberWithInt:10];
int a = [n intValue];
@10;//快速包装
包装成Oc对象就可以放到数组、字典中
```
24. NSValue

```
NSnumber继承自NSValue
// 将结构体包装成Oc对象
CGPoint p = CGPointMake(2,2);
NSValue *v = [NSValue valueWithPoint:p];
```
 

25.   NSDate
 
```
NSDate *d = [NSDate date];
// 直接打印d会打印零时区的时间，北京是东八区
// 日期格式化
NSDateformatter *df = [[NSDateFormatter alloc] init];
df.dateFormat = @“yy-MM-dd HH:mm:ss”;
NSString *s = [df stringFromDate:d];
NSLog(@”%@”,str);
``` 

