---
title: 如何提取iOS App切图
date: 2016-10-24 20:15:52
categories: iOS Development
tags:
description: 在一些诸如GitHub的开源网站上，经常能够看到不少高仿的市场上知名App的作品，其中不乏一些精品之作。那么，既然希望做到高仿，就必然要做最大程度的还原，这其中UI又是最直接且最不可获取的一环因素，如何最大程度的还原UI便成了让人十分头疼的问题，虽然存在像iconfont这类的开源icon网站，但是绝大多数精品App都是原创切图，甚至是纯手绘，这是无法从开源网站获取到的。

---

在一些诸如GitHub的开源网站上，经常能够看到不少高仿的市场上知名App的作品，其中不乏一些精品之作。诚然，选取钟意的App来尝试实战训练，是提升和磨练自己技术的一个好机会，但是，既然希望做到高仿，就必然要做最大程度的还原，这其中UI又是最直接且最不可获取的一环因素，如何最大程度的还原UI便成了让人十分头疼的问题，虽然存在像iconfont这类的开源icon网站，但是绝大多数精品App都是原创切图，甚至是纯手绘，这是无法从开源网站获取到的。对于绝大多数没有美术功底的个人开发者来说，这似乎成为一道不可逾越的沟壑，想要高仿的念头因为切图而被浇灭在萌芽阶段，十分可惜。那么既然从美术角度复制不可行，就没有其他方法了吗？答案是有的，正所谓“All roads lead to Rome”。下文将要提到的，就是 如何提取出iOS App的切图。
为了更加直观，我们首先以一款App Store中的应用为例，来展示如何通过ipa来提取出App的切图：

环境/工具：
·Mac OS X 10.11.6
·iTunes 12.5.1.24
·xcode 8.0
·Images-Extractor v0.2.5 [GitHub下载地址](https://github.com/devcxm/iOS-Images-Extractor)

方法/步骤:
1.打开iTunes，切换到我的应用，然后选择一款App。如果你还没有下载你希望提取切图的App，先通过iTunes Store搜索该App并下载下来，这里以MissEvan为例，如下所示
<img src="http://ofg0p74ar.bkt.clouddn.com/8FB9C80C-6169-4CB8-A576-028ED25B17D8.png" width="951" height="502">

2.点击“MissEvan”App选中，然后拖动它到桌面上，这样就拿到了ipa包，如下图所示
<img src="http://ofg0p74ar.bkt.clouddn.com/81B1A19D-45C6-4623-9EA2-6D17F44A560C.png" width="951" height="502">

3.接下来，将ipa后缀修改为zip后缀，然后解压得到目录，如下所示
<img src="http://ofg0p74ar.bkt.clouddn.com/5953794C-316C-481E-BE26-0EDC44F3BFD3.png" width="951" height="502">

4.找到刚才解压得到的目录，然后进入“Payload”目录，找到“MissEvanApp”文件，并右键显示包内容，如下所示
<img src="http://ofg0p74ar.bkt.clouddn.com/82F0883F-DB9E-446E-8FB1-D4474ED99EBD.png" width="951" height="502">

5.进入到“MissEvanApp”包内容后，找到“Assets.car”文件，里面就是我们需要的最主要的切图文件了，而当前目录下只有一些启动app的icon图标，如下所示
<img src="http://ofg0p74ar.bkt.clouddn.com/0F08F7FF-B31B-4EAC-8AB2-61003CCA6841.png" width="951" height="502">

6.到GitHub上搜索iOS-Images-Extractor或点击上面的下载链接，按照README的说明（Build）clone到本地，然后打开Xcode运行一下，然后将“Assets.car”文件拖动到如下图的软件中，然后点击“start”开始提取，在出现"Jobs done,have fun"这句话后，点击“Output Dir”会自动生成一个文件夹，里面就是这个App全部的切图。
<img src="http://ofg0p74ar.bkt.clouddn.com/73EF4ED4-771C-459B-9169-E6B1D5A0CB3B.png" width="951" height="502">


如果有任何疑问，欢迎在下方评论区提出，我会及时回复。
