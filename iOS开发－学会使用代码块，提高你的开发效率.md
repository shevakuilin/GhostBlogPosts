---
title: iOS开发－学会使用代码块，提高你的开发效率
date: 2016-05-10 23:15:22
categories: iOS Development
tags:
description: 相信很多开发者在新手阶段都免不了记不住方法等各种各样的窘境，于是，很多时候，在遇到使用相同控件属性时，苦于记不住其种类繁多的代理方法，就只能照着之前写过的代码再照搬一遍，又或者稍有经验的开发者在遇到代码量略多但框架大体相同，只有细微几处修改的需求时，copy之前写过的代码片段并进行适当的修改，成了在日常开发中“提高开发效率”的常用手段，但是往往找寻之前的代码也是一件颇为耗时的事情。

---


相信很多开发者在新手阶段都免不了记不住方法等各种各样的窘境，于是，很多时候，在遇到使用相同控件属性时，苦于记不住其种类繁多的代理方法，就只能照着之前写过的代码再照搬一遍，又或者稍有经验的开发者在遇到代码量略多但框架大体相同，只有细微几处修改的需求时，copy之前写过的代码片段并进行适当的修改，成了在日常开发中“提高开发效率”的常用手段，但是往往找寻之前的代码也是一件颇为耗时的事情。


不过，好在苹果公司早就已经为开发者考虑到了这一点，在Xcode中为开发者准备好了“快捷方式”——`代码块`


代码块，很多刚接触iOS开发的新手可能并不知道这是什么，甚至已经有1-2年工作经验的开发者没有使用过代码块的也大有人在。那么这个代码块究竟是做什么的呢？


我先来演示一遍使用效果，相信大家便会一目了然。



----------
现在，我准备在viewController里使用一个tableView，需要用到其代理协议中的方法，于是：
<img src="https://segmentfault.com/img/bVvrYk" width="600" height ="1050" />



有没有觉得很神奇，这个效率如何呢，短短2秒钟的时间（可能还不到），就写完了tableView代理协议中的几个基本上必用到的方法，剩下只需要对没填写完成的占位符进行填写就完成了，效率不可谓不快，这就是`代码块`在日常开发中的作用。



----------


现在，大家对代码块的作用应该已经了解了，那么下面，就让我们来看看如何使用这个代码块呢。



### `代码块`，顾名思义，就是一“块”嵌入的代码框架，提前将所需的代码框架写入代码块，仅留出可能发生改动的地方用占位符代替，使用时，以自定义标记的按键呼出相应代码块，填写所需占位符即可完成高效率的开发。



#### 1.首先，我们要现在类当中将我们所需的代码写好，以刚才我所使用的tableView的代理方法为例：



```objectivec
#pragma mark -
#pragma mark - tableView
-(NSInteger)numberOfSectionsInTableView:(UITableView *)tableView
{
return 1;
}

-(NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section
{
return <#expression#>
}

-(UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
<#classCell#> * cell = [tableView dequeueReusableCellWithIdentifier:<#(nonnull NSString *)#>];

return cell;
}

-(void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath
{

}

-(CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath
{
return <#expression#>
}

**注：占位符的书写格式为<#name#>**
```


#### 2.写好代码之后，我们找到Xcode的右下角，如图的方式，找到代码块的存放处

<img src="https://segmentfault.com/img/bVvr3J" width="600" height ="1050" />


#### 3.这些便是我们存放代码块的地方，Xcode中提前已经准备了一些系统自带的方法

<img src="https://segmentfault.com/img/bVvr3Q" width="600" height ="1050" />


#### 4.然后，我们需要做的就是将我们写好的代码 丢进 存放代码块的地方，你没有看错，就是丢进去

<img src="https://segmentfault.com/img/bVvr4K" width="600" height ="1050" />


#### Title就是你这段代码在储存点要给展示出来的名字，图上标注的地方就是你呼出它所需键入的缩写，随便什么都可以，想些什么些什么，当然越短越好，这样，就大功告成了下次需要使用的时候就只需打出你的缩写，这段代码就自己调出来了

<img src="https://segmentfault.com/img/bVvr5b" width="600" height ="1050" />


#### 6.尝试呼出你新建的代码块，就如最开始我做的那样，如果代码块数量不多，也可以直接从储存点直接将其拖出来使用，像最开始存放时做的一样，只不过我们是反过来拖出来

#### 7.如果需要对已经存好的代码块进行修改，那么只需要找到你的代码块，然后单机它，点击`edit`即可，如果想要删除代码块，只需要选中代码块，然后轻敲`Backspace`键，弹出选项框时选择`delete`即可

