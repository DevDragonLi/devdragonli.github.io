---
layout: post
title: 2016-03-02-ScreenAdaptation屏幕适配
date: 2016-03-02
categories: blog
tags: [iOSDev]
description: 2016-03-02-ScreenAdaptation屏幕适配框架(针对自己写的屏幕适配框架介绍)

---

## [ScreenAdaptation地址](https://github.com/DevDragonLi/ScreenAdaptation-Rapid)

## 已上线项目采取此库适配APP:

- 中国文化与艺术

- Aparty

- 板凳足球

## User

```
UIView *view = [[UIView alloc]init];
view.backgroundColor = [UIColor redColor];
[self.view addSubview:view];
view.frame = RectMake_LFL(0,0, 100, 100);

```

## 总结:
- 主要是用了单例,获取缩放比例,网上很早之前4s-5适配采取的策略.只不过形成了工具类(重写系统的CGRectMake等不再控制器化),更加简化.比如说你是iphone6的UI图,采取这个,可极大的方便我们开发人员



## 代码实现

```

1.单例
+ (instancetype)sharedFrameAutoScaleLFL{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        isFirstAccess = NO;
        SINGLETON = [[super allocWithZone:NULL] init];
        
    });
    
    return SINGLETON;
}

2.策略 所有设置中只计算一次缩放比例  只采用  x缩放计算  放弃x和y同时
- (void)AutoSizeScale{
    _autoSizeScaleX = ScreenWidthLFL/RealUISrceenWidth;
//    _autoSizeScaleY = ScreenHightLFL/RealUISrceenHight;
}

3.重写CGSize 方法
CG_INLINE CGSize
CGSizeLFLMake(CGFloat width, CGFloat height)
{
    FrameAutoScaleLFL *LFL = [FrameAutoScaleLFL sharedFrameAutoScaleLFL];
    CGSize sizeLFL;
    sizeLFL.width = width* LFL.autoSizeScaleX;
    sizeLFL.height = height* LFL.autoSizeScaleX;
    return sizeLFL;
}


```



