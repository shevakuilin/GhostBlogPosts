---
title: iOS开发－AutoLayout控件等比间距的百分比布局
date: 2015-12-04 10:48:36
categories: iOS Development
tags:
description: 多控件水平、垂直分布要求相同间距时，使用multiplier来进行布局，即百分比布局是最佳选择。

---

**多控件水平、垂直分布要求相同间距时，使用multiplier来进行布局，即百分比布局是最佳选择。**


在看本文前可以了解一下[iOS NSLayoutConstraint priority][1] 其中就提到过multiplier, 本文中的百分比布局都是基于 `multiplier` 实现的，下面来一一查看其实现。


----------

我们需要实现的功能很简单，以一个居中的按钮为基准，达成五个按钮等比间距水平分布在屏幕上的效果为例，首先约束居中按钮的宽高`Width`和`Height`，然后相对于父容器添加`Center Horizontally in Container`（水平居中）和`Center Vertically in Container`（垂直居中），剩下的四个按钮如出一辙的重复上述操作。

完成后，我们开始设置除去居中按钮外的其他按钮的`multiplier`，现在我们将其余四个按钮的`Center Horizontally in Container`约束中的`multiplier`从左到右依次
设为0.2， 0.6， 1.4， 1.8，这样就可实现按钮的水平位置为父容器宽度的xx倍，具体数值可以自行根据需求设定。**操作步骤如下：**


![](https://segmentfault.com/img/bVwAid)

![](https://segmentfault.com/img/bVwAik)

![](https://segmentfault.com/img/bVwAip)

我用文字取代来简单的说明一下重点，如何设置`multiplier`：当我们完成上面的步骤之后，会看到右边有一个`Align Center X to:`的约束，双击，进去就可以看见`multiplier`。

具体过程如下
![](http://upload-images.jianshu.io/upload_images/2660903-454f8153bc98e3af.gif?imageMogr2/auto-orient/strip)

还有疑问的同学可以在文章下留言或者联系我的邮箱
Email: **shevakuilin@gmail.com**


上述内容就是百分比布局的基本使用方法，垂直布局同理即可。
