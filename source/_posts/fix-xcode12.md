title: Xcode12 模拟器编译报错
date: 2020-09-27 11:38:31
tags: iOS
categories: technology
---

项目代码在 Xcode 12 版本模拟器编译报错记录
<!-- more -->

报错信息
`
No architectures to compile for (ONLY_ACTIVE_ARCH=YES, active arch=x86_64, VALID_ARCHS=armv7 armv7s arm64).
`

原因分析
`
Xcode 12 去掉了 Valid Architectures，模拟器的指令集为 x86_64
`

解决方法
`
在 Build Setting -> User-Defined -> VALID_ARCHS 增加 x86_64
`
[](!https://image.sleen.top/hexo/%E6%88%AA%E5%B1%8F2020-09-29%20%E4%B8%8B%E5%8D%882.45.29.png)

关于指令集
```
arm64: iPhone6s, iphone6s plus, iPhone6, iPhone6 plus, iPhone5S
armv7s: iPhone5, iPhone5C
armv7: iPhone4, iPhone4S

i386 是针对 intel 通用微处理器32位处理器
x86_64 是针对 x86 架构的64位处理器

模拟器32位处理器测试需要 i386 架构
模拟器64位处理器测试需要 x86_64 架构
真机32位处理器需要 armv7,或者 armv7s 架构
真机64位处理器需要 arm64 架构
```

