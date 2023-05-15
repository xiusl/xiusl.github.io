---
title: 曾经的ios04
date: 2015-10-04 16:20:00
categories: 
- 技术
- Objective-C 
tags: ios
---

# iOS学习06
1-今天完成UIScrollView的小例子
2-实现图片的捏合缩放，1103-scrollview01，拖一个UIScrollView在控制器的view上，在UIScrollView上添加一个UIImageView，添加一张大的图片，把UIIamgeView的长宽设置成图片的宽度，<!-- more -->
3-先实现图片的滚动，设置scrollview的contentSize的大小为UIImageView的size大小，就能实现滚动
4-将UIScrollView的代理设置成控制器self，并让控制器遵守 ，并让控制器实现协议中的viewForZoomingInScrollView:方法，返回要实现缩放的控件，并要设置ScrollView的minimumZoomScale和maximumZoomScale(成倍的)，就可以实现缩放了
5-喜马拉雅简单实现，1103-scroolview02，主界面主要是UIScrollView的几个属性的应用，先直接把值写上了。
    
   ```
    self.scrollView.contentSize = CGSizeMake(0, 560);

    self.scrollView.contentInset = UIEdgeInsetsMake(64, 0, 44, 0);

    self.scrollView.contentOffset = CGPointMake(0, -64);
   ```

6-图片轮播的实现，1103-scrollview03，在控制器的view上放一个UIScrollView，宽度是控制器view的宽度，高度调整合适

7-通过代码生成一定数量的UIIamgeView，并设置图片

8-设置scrollview的pagingEnabled＝YES，会自动分页

9-设置自动滚动，定时器，将改变放到控制器中

10-当你拖住图片时，定时器会累积时间，松开后会变好几页，当你按住图片时，要停止定时器，松开后再开启定时器，要将定时器开启，用到代理

11-使用Page Control增强效果，设置pagecontrol的numberOfPages等于图片数，在changeImage方法中设置当前页

12-基本上功能就实现了

做的比较简单，没有什么特殊的知识，之前都学习过了，后来把图片轮播和喜马拉雅综合一起，没怎么写注释，把代码贴出来。

```
//
//  ViewController.m
//  1103-scrollview04
//
//  Created by 修修 on 15/11/3.
//  Copyright © 2015年 cn.Xsoft. All rights reserved.
//

#import "ViewController.h"

@interface ViewController () <</span>UIScrollViewDelegate>
@property (weak, nonatomic) IBOutlet UIScrollView *bigScroll;
@property (weak, nonatomic) IBOutlet UIScrollView *smallScroll;
@property (weak, nonatomic) IBOutlet UIPageControl *pageControl;
@property (weak, nonatomic) IBOutlet UIButton *lowBtn;
@property (weak, nonatomic) IBOutlet UIView *topView;
@property (weak, nonatomic) IBOutlet UIView *lowView;


@property (nonatomic, strong) NSTimer *time;
@end

@implementation ViewController


const int count = 5;
- (void)viewDidLoad {
    [super viewDidLoad];
   

    
    CGFloat interval = 10;
    CGFloat scrollH = self.lowBtn.frame.origin.y + interval + self.lowBtn.frame.size.height;
    
    self.bigScroll.contentSize = CGSizeMake(0, scrollH);
    
    self.bigScroll.showsVerticalScrollIndicator = NO;
    
    CGFloat insetTop = self.topView.frame.size.height - self.smallScroll.frame.origin.y + interval;
    CGFloat insetLow = self.lowView.frame.size.height;
    self.bigScroll.contentInset = UIEdgeInsetsMake(insetTop, 0, insetLow, 0);
    
    self.bigScroll.contentOffset = CGPointMake(0, 0 - insetTop);
    




    CGFloat imageW = self.smallScroll.frame.size.width;
    CGFloat imageH = self.smallScroll.frame.size.height;
    
    
    for (int i = 0; i < count; i++) {
        UIImageView *image = [[UIImageView alloc] init];
        NSString *imagename = [NSString stringWithFormat:@"img_0%d", i + 1];
        image.image = [UIImage imageNamed:imagename];
        CGFloat imageX = i * imageW;
        image.frame = CGRectMake(imageX, 0, imageW, imageH);
        [self.smallScroll addSubview:image];
    }
    
    
    self.smallScroll.showsHorizontalScrollIndicator = NO;
    self.smallScroll.contentSize = CGSizeMake(count *imageW, 0);
    self.smallScroll.pagingEnabled = YES;
    
    self.pageControl.numberOfPages = count;
    
    
    self.smallScroll.delegate = self;
    
    self.time = [NSTimer scheduledTimerWithTimeInterval:1.5 target:self selector:@selector(changeImage) userInfo:nil repeats:YES];
    
    [[NSRunLoop mainRunLoop] addTimer:self.time forMode:NSRunLoopCommonModes];


}

- (void)changeImage {
    
    if (self.pageControl.currentPage == count - 1) {
        self.pageControl.currentPage = 0;
    } else {
        self.pageControl.currentPage++;
    }
    
    
    CGFloat changeX =  self.smallScroll.frame.size.width * self.pageControl.currentPage;
    [self.smallScroll setContentOffset:CGPointMake(changeX, 0) animated:YES];

}


- (void)scrollViewDidEndDragging:(UIScrollView *)scrollView willDecelerate:(BOOL)decelerate
{
    self.time = [NSTimer timerWithTimeInterval:1.5 target:self selector:@selector(changeImage) userInfo:nil repeats:YES];
    [[NSRunLoop mainRunLoop] addTimer:self.time forMode:NSRunLoopCommonModes];
}

- (void)scrollViewWillBeginDragging:(UIScrollView *)scrollView
{
    [self.time invalidate];
    self.time = nil;
}

- (void)scrollViewDidScroll:(UIScrollView *)scrollView
{
    if (self.time) return;
    self.pageControl.currentPage = (scrollView.contentOffset.x + scrollView.frame.size.width * 0.5) / scrollView.frame.size.width;
}
@end
```