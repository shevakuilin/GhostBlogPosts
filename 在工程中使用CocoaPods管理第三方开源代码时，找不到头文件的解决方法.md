---
title: 在Xcode中使用CocoaPods管理第三方开源代码时，找不到头文件的解决方法
date: 2016-10-22 16:28:20
categories: iOS Development
tags:
description: 现在几乎大多数项目都会用CocoaPods来管理自己使用的第三方开源代码，用CocoaPods虽然很方面，但也经常会出现一些意想不到的小问题，比如下文将要提到的“依托CocoaPods来管理的第三方开源代码的引用头文件file not found”

---

现在几乎大多数项目都会用CocoaPods来管理自己使用的第三方开源代码，用CocoaPods虽然很方便，但也经常会出现一些意想不到的小问题，比如下文将要提到的“依托CocoaPods来管理的第三方开源代码的引用头文件file not found”。
在Xcode里使用CocoaPods时，偶尔会发生头文件无法找到的问题，突然的出现会比较令人头疼，所以，我总结了目前广泛使用的两种解决方法，如果读者有其他更好的方法，欢迎在下方评论区补充交流。
方法一：
在TARGETS -> Build Settings -> Search Paths -> User Header Search Paths 中 写入 ${SRCROOT} 再将后面参数改为recursive
<img src="http://ofg0p74ar.bkt.clouddn.com/CE01F7E9-583F-48D1-9A7B-5C3417D050D2.png" width="951" height="502">

方法二:
在PROJECT -> info -> Configurations ->Debug/Release -> AppTests 后面的None改为Pods, 之后重新编译你的工程
<img src="http://ofg0p74ar.bkt.clouddn.com/6F3601F3-55D0-4660-BDA2-AFD432266938.png" width="951" height="502">

基本使用这两种方法后，问题都能得到解决，笔者亲测有效（方法二成功率较高）。如果仍然无法得到解决的话，请在下方评论区留言，一起交流讨论。
