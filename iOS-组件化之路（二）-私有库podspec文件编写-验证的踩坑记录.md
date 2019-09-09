---
title: iOS 组件化之路（二）----- 私有库podspec文件编写 & 验证的踩坑记录
date: 2018-11-08 22:09:36
categories: iOS Development
tags:
description: 该系列文章旨在分享 iOS 组件化的全过程 & 期间所遇问题的处理经验，本文是第二篇，重点介绍了podspec文件编写的详细方式与 pod 验证报错的定位与处理


---


## 前言

这是本系列的第二篇文章，主要介绍在私有库podspec文件编写与验证上的踩坑经验，这也是私有库创建发版过程中最麻烦，坑最多，也最为棘手的部分，所以单独拿出一篇来分享，也希望我的经验能够帮到你。如果需要了解如何创建私有库，请看[这里](https://shevakuilin.github.io/2018/08/20/iOS-%E7%BB%84%E4%BB%B6%E5%8C%96%E4%B9%8B%E8%B7%AF%EF%BC%88%E4%B8%80%EF%BC%89-CocoaPods%E7%A7%81%E6%9C%89%E5%BA%93%E7%9A%84%E5%88%9B%E5%BB%BA/)。

可以毫不夸张的说，只要podsepc文件的编写与验证的问题解决了，剩下的组件化之路便是一马平川，畅通无阻。

首先介绍的是 podspec 文件的详细编写，包括各个依赖关系的该如何处理、私有库依赖私有库、subspec依赖subspec等等


## Podspec文件的编写


#### 1.subspec的编写
