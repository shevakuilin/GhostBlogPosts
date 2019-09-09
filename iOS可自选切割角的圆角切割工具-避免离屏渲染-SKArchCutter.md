---
title: iOS可自选切割角的圆角切割工具 (避免离屏渲染) - SKArchCutter
date: 2017-03-28 10:30:35
categories: iOS Development
tags:
description: SKArchCutter，是一个可自选切割角的圆角切割工具，同时支持`UIView`、`UIImageView`、`UIButton`和`UILabel`的单角切圆/选角拱形切圆/全角切圆，并且避免了`UIImageView`使用系统圆角所导致的离屏渲染的问题，以及确保`layer`对象的`masksToBounds`属性始终为`NO`，从而使得项目中大量使用圆角时的性能得到很大程度的优化, 最重要的是使用简单、方便。如果觉得还不错，star支持下吧~

---

![](http://upload-images.jianshu.io/upload_images/2660903-b24d7872db20950b.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



# 最新更新

- 解决使用Masonry布局无法及时获取frame信息的兼容问题

- 解决使用border而导致的失效问题

- 改为类方法，使用更加简单方便

- 注意：如果之前设置了`border`和`backgroundColor`请取消，关闭`masksToBounds`(如果打开了话), 请在方法中进行设置


# 简述

[SKArchCutter](https://github.com/shevakuilin/SKArchCutter)，是一个可自选切割角的圆角切割工具，同时支持`UIView`、`UIImageView`、`UIButton`和`UILabel`的单角切圆/选角拱形切圆/全角切圆，并且避免了`UIImageView`使用系统圆角所导致的离屏渲染的问题，以及确保`layer`对象的`masksToBounds`属性始终为`NO`，从而使得项目中大量使用圆角时的性能得到很大程度的优化, 最重要的是使用简单、方便。如果觉得还不错，star支持下吧~


# 为什么要避免离屏渲染？

这里就先不对离屏渲染做过多讲解了，[关于离屏渲染的介绍，可以参考这篇文章](https://objccn.io/issue-3-1/)

我们先来说说离屏渲染的影响：
>在界面的滚动过程中如果有大量的离屏渲染发生时会严重影响帧率。

那么离屏渲染会因为什么原因被触发呢：

>官方公开的的资料提到了尽量避免会触发离屏渲染的效果:mask, shadow, group opacity, edge antialiasing。

>使用系统提供的圆角效果也会触发离屏渲染, 如：
imageView.layer.cornerRadius = 5
imageView.layer.masksToBounds = YES

当然，能够触发离屏渲染的因素远不止上述这些，这里仅仅是举例。


# SKArchCutter的作用是什么?
- 方便、快捷的帮助你从任意边角进行圆角的切割（如：半圆矩形、只有一个角是圆角的矩形、整体圆角切割等）

- 同时支持`UIView`、`UIImageView`、`UIButton`和`UILabel`

- 避免了系统圆角导致的离屏渲染问题，确保`laye`r对象的`masksToBounds`属性始终为`NO`，提升了大量使用圆角时的性能流畅性，减小了CPU和GPU的消耗

- 避免了因工作线程的延迟，而导致图片闪烁的现象，这里学习了[HJCornerRadius](https://github.com/panghaijiao/HJCornerRadius)的思路
### 效果图 
![](http://upload-images.jianshu.io/upload_images/2660903-f46c568bb12c6b9d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 测试性能

ps:录制的帧数已经跟不上屏幕滑动的速度了，足以说明滑动的流畅性有多高了

![](http://upload-images.jianshu.io/upload_images/2660903-4217dfeb2620cf79.gif?imageMogr2/auto-orient/strip)

# 如何开始

1.从GitHub上Clone-->[SKArchCutter](https://github.com/shevakuilin/SKArchCutter)，然后查看Demo

2.直接将目录下的SKArchCutter拷贝到工程中，或在podfile文件夹中添加 `pod 'SKArchCutter'`

3.觉得不错的话，点个star吧~



# 使用方法



#### 头文件导入


```objectivec
#import "SKArchCutter.h"
```

#### 初始化
```objectivec
SKArchCutter * archCutter = [[SKArchCutter alloc] init];
```

#### 进行圆角切割

UIView/UIButton/UILabel
```objectivec
[SKArchCutter cuttingView:self.centerView cuttingDirection:UIRectCornerTopRight | UIRectCornerTopLeft cornerRadii:self.centerView.frame.size.height / 2 borderWidth:1 borderColor:[UIColor purpleColor] backgroundColor:[UIColor redColor]];
```

UIImageView
```objectivec
[SKArchCutter cuttingImageView:self.topImageView cuttingDirection:UIRectCornerAllCorners cornerRadii:self.topImageView.frame.size.height / 2 borderWidth:1 borderColor:[UIColor blackColor] backgroundColor:[UIColor clearColor]];
```

## 喜欢的话点个star哦~
