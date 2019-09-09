---
title: 修复一个由Hook系统类而引发的Crash
date: 2018-08-14 18:41:28
categories: iOS Development
tags:
description: 项目中存在着一个遗留了数个版本的小顽疾，在 iOS11以下版本的系统中，当输入框处于编辑状态，即键盘处于弹出状态下，按 Home 键 退入后台，必现 Crash。

---

### Crash 背景

今天在整理项目遗留 Crash 问题的时候，发现了一个经历数个版本仍未被成功修复的顽疾，在 Bugly 的反馈如下：

```objc
#0 Thread
SIGSEGV
SEGV_ACCERR


0 libobjc.A.dylib	objc_release + 16
1 libsystem_blocks.dylib	_Block_release + 156
2 UIKit	-[UIKeyboardTaskEntry dealloc] + 44
3 libobjc.A.dylib	(anonymous namespace)::AutoreleasePoolPage::pop(void*) + 508
4 CoreFoundation	_CFAutoreleasePoolPop + 28
5 UIKit	__prepareForCAFlush + 352
6 UIKit	__afterCACommitHandler + 160
7 CoreFoundation	___CFRUNLOOP_IS_CALLING_OUT_TO_AN_OBSERVER_CALLBACK_FUNCTION__ + 32
8 CoreFoundation	___CFRunLoopDoObservers + 372
9 CoreFoundation	___CFRunLoopRun + 928
10 CoreFoundation	CFRunLoopRunSpecific + 384
11 GraphicsServices	GSEventRunModal + 180
12 UIKit	UIApplicationMain + 204
13 Xitu	main (main.m:14)
14 libdyld.dylib	_start + 4
```

那么很明显，这是一个指向系统内部方法的野指针问题。

在最初看到这段调用栈信息的时候，是有些懵的，` -[UIKeyboardTaskEntry dealloc]` 很明显是系统内部方法，从名字上很容易看出是一个控制键盘任务的实例执行了析构函数，调用栈最终停在这里，就说明应用是在系统键盘回收的过程中出现了问题。

那什么样的情况才会导致系统的键盘出现问题，什么样的使用场景才会干扰到系统自身的处理？


### Crash 定位

最初我怀疑可能是某个页面的内存使用不当，从而引发了自动释放池的回收，于是通过 Bugly 查看了发生 Crash 的用户路径，发现路径并不是唯一的，甚至完全不相同，也就是说，在应用内任何能够唤起键盘的页面都有可能发生 Crash，这就比较令人头疼了。

可既然是批量出现的 Crash，就一定是有规律可循的，经过对比发现，所有发生 Crash 的用户都有一个退入后台的操作。那就是说，当一个用户在唤起键盘之后又将应用切换到后台，就会发生 Crash 。

由于是野指针的问题，所以直接开启了 Zombie 模式，用 iOS11.4 的模拟器反复测试，一切正常。模拟器不行的话，换成 iOS11.4 的真机测试，依旧无法复现。

不过，在翻阅了几页 Crash 记录后，有了一个惊人的发现：所有发生了 Crash 的用户的设备，全部都是 `iOS 9.3.x` 版本，无一例外，那这很有可能是这个版本操作系统自身的一些问题。

去 Google 一番，确实是有人反馈 `iOS 9.1 ~ 9.3`版本的操作系统会产生一系列奇怪的 Crash，那既然确实是系统存在问题，我们也干涉不了，这个 Crash 还处理吗 ？回答是肯定的，你总不能强制用户升级系统吧 orz 。

苦于身边实在找不到 iOS 9.0 + 的设备，目前我的 Xcode 也只装着 iOS 11.4 和 iOS 10.3.1 两个系统版本的模拟器，于是准备通过下载 iOS 9.3 的模拟器来复现这个 Crash。看了一下 iOS 9.3 的模拟器居然有 1.3G 大，想想公司最近糟糕的网络，突发奇想用 iOS 10.3.1 的模拟器试试能不能复现。

果然懒是第一生产力，在 iOS 10.3.1 的模拟器上按照用户路径跑了一遍，在键盘唤起时，从前台退入后台的一瞬间，Zombie 捕捉到了这个野指针，通过 lldb 的 `bt` 命令打印了当前调用栈，与 Bugly 上调用栈完全一致，这个 Crash 被成功复现了，控制台的 Log 显示 :

```objc
[UIKeyboardLayoutStar release]: message sent to deallocated instance
```


### Crash 修复

经过搜索发现，所有的疑点都指向了使用 Swizzle 来动态拦截NSArray、NSMutableArray、NSDictionary、NSMutableDictionary的方法，这种通过 Hook 系统类的方式来达到动态防护下标越界等行为的操作，隐患是非常大的。

这突然让我想起项目里曾添加过几个 Category 用以防止数组越界、字典插入 nil 等操作。

于是通过Build Phases中 -> Compile Sources 找到了这几个 Category，想起之前看到过的一个观点：涉及比较多 runtime 的代码，最好不要用 ARC，结合 Crash 的 Log 情况来看，我决定把这几个 Category 替换成 MRC 。

在改之前，我发现 NSArray 的运行环境已经被设置成了 MRC，既然没能修复这个 Crash，很明显不是 NSArray 这个 Category 造成的。

回想一下 iOS 的内存核心之一不就是散列表，也就是常说的哈希表吗，而 Objective-C 中的 NSDictionary 底层其实是就是一个哈希表，那么罪魁祸首很显然就应该是对 NSDictionary 以及 NSMutableDictionary 进行 Swizzle 的 Category 了。

在将这两个 Category 的 Compiler Flags 加入 -fno-objc-arc 后，重新复测 Crash 步骤，Zombie 没有再捕获到任何野指针的出现，这个 Crash 被彻底修复了。


#### PS:

后来发现 NSMutableArray 和 NSMutableDictionary 这两个 Category 中的 hook 方法是存在问题的，其获取到并不是真正的类，而是对类簇直接获取其实例方法，对系统类这样操作这本身就是错误而且十分危险的行为，虽然这个 Crash 已经被修复了，但是这样有问题的代码必须得处理，于是在原有的基础上进行了改进：

#### NSMutableArray

修改前：

```objc
@implementation NSMutableArray (NilSafe)

+ (void) load{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        id obj = [[self alloc] init];
        ///动态运行时校验两个方法
        [obj swizzleMethod:@selector(addObject:) withMethod:@selector(safe_addObject:)];
        [obj swizzleMethod:@selector(objectAtIndex:) withMethod:@selector(safe_objectAtIndex:)];
    });
    
}

- (void)swizzleMethod:(SEL)origSelector withMethod:(SEL)newSelector{
    
    Class class = [self class];
    Method originalMethod = class_getInstanceMethod(class, origSelector);
    Method swizzledMethod = class_getInstanceMethod(class, newSelector);
    
    BOOL didAddMethod = class_addMethod(class,
                                        origSelector,
                                        method_getImplementation(swizzledMethod),
                                        method_getTypeEncoding(swizzledMethod));
    
    if (didAddMethod) {
        
        class_replaceMethod(class,
                            newSelector,
                            method_getImplementation(originalMethod),
                            method_getTypeEncoding(originalMethod));
    } else {
        method_exchangeImplementations(originalMethod, swizzledMethod);
    }
    
}

- (void)safe_addObject:(id)value{
    
    if (value) {
        [self safe_addObject:value];
    }else {
        ///这里是设置obj为nil的时候进入
        NSLog(@"[NSMutableArray addObject:]%@", [self.superclass  class]);

    }
    
}

- (id)safe_objectAtIndex:(NSInteger)index{
    
    if (index < self.count) {
        
        return [self safe_objectAtIndex:index];
        
    }else {
        ///下标越界
        NSLog(@"NSMutableArray下标越界%@", [self.superclass  class]);
        return nil;
        
    }
    
}


```

修改后

```objc
@implementation NSMutableArray (NilSafe)

+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        
        Class __NSArrayM = NSClassFromString(@"__NSArrayM");
        
        Method originalMethod = class_getClassMethod(__NSArrayM, @selector(addObject:));
        Method swizzledMethod = class_getClassMethod(__NSArrayM, @selector(safe_addObject:));
        method_exchangeImplementations(originalMethod, swizzledMethod);
        
        Method originalMethod1 = class_getClassMethod(__NSArrayM, @selector(objectAtIndex:));
        Method swizzledMethod1 = class_getClassMethod(__NSArrayM, @selector(safe_objectAtIndex:));
        method_exchangeImplementations(originalMethod1, swizzledMethod1);
    });
}

- (void)swizzleMethod:(SEL)origSelector withMethod:(SEL)newSelector {
    Method originalMethod = class_getClassMethod([self class], origSelector);
    Method swizzledMethod = class_getClassMethod([self class], newSelector);
    method_exchangeImplementations(originalMethod, swizzledMethod);
}

- (void)safe_addObject:(id)value {
    if (value) {
        [self safe_addObject:value];
    } else {
        ///这里是设置obj为nil的时候进入
        NSLog(@"[NSMutableArray addObject:]%@", [self.superclass  class]);
    }
}

- (id)safe_objectAtIndex:(NSInteger)index {
    if (index < self.count) {
        return [self safe_objectAtIndex:index];
    } else {
        ///下标越界
        NSLog(@"NSMutableArray下标越界%@", [self.superclass class]);
        return nil;
    }
}
```

#### NSMutableDictionary

修改前：

```objc
@implementation NSMutableDictionary (NilSafe)

- (void)swizzleMethod:(SEL)origSelector withMethod:(SEL)newSelector{
    
    Class class = [self class];
    Method originalMethod = class_getInstanceMethod(class, origSelector);
    Method swizzledMethod = class_getInstanceMethod(class, newSelector);

    BOOL didAddMethod = class_addMethod(class,
                                        origSelector,
                                        method_getImplementation(swizzledMethod),
                                        method_getTypeEncoding(swizzledMethod));
    

    if (didAddMethod) {
        class_replaceMethod(class,
                            
                            newSelector,
                            
                            method_getImplementation(originalMethod),
                            
                            method_getTypeEncoding(originalMethod));
        
    } else {
        
        method_exchangeImplementations(originalMethod, swizzledMethod);
        
    }
    
}


+ (void)load {
    
    static dispatch_once_t onceToken;
    
    dispatch_once(&onceToken, ^{
        
        id obj = [[self alloc] init];
        
        [obj swizzleMethod:@selector(setObject:forKey:) withMethod:@selector(safe_setObject:forKey:)];
        
        
    });
    
    
}


- (void)safe_setObject:(id)value forKey:(NSString *)key {
    
    if (value) {
        
        [self safe_setObject:value forKey:key];
        
    }else {
          NSLog(@"[NSMutableDictionary setObject: forKey:]%@", [self.superclass class]);
    }
    
}

```

修改后:

```objc
@implementation NSMutableDictionary (NilSafe)

- (void)swizzleMethod:(SEL)origSelector withMethod:(SEL)newSelector {
    Method originalMethod = class_getClassMethod([self class], origSelector);
    Method swizzledMethod = class_getClassMethod([self class], newSelector);
    method_exchangeImplementations(originalMethod, swizzledMethod);
}

+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        Class __NSDictionaryM = NSClassFromString(@"__NSDictionaryM");
        Method originalMethod = class_getClassMethod(__NSDictionaryM, @selector(setObject:forKey:));
        Method swizzledMethod = class_getClassMethod(__NSDictionaryM, @selector(safe_setObject:forKey:));
        method_exchangeImplementations(originalMethod, swizzledMethod);
    });
}

- (void)safe_setObject:(id)value forKey:(NSString *)key  {
    if (value) {
        [self safe_setObject:value forKey:key];
    } else {
          NSLog(@"[NSMutableDictionary setObject: forKey:]%@", [self.superclass class]);
    } 
}


```

### 小结

1. 猜测是 iOS11.0 + 的系统对这部分内存管理做了特别的优化，才使得一直难以复现这个问题。

2. 尽可能的不要对系统的类做 Hook 操作，可能引发意想不到的后果。

3. 如果一定要做动态防护，那么 iOS11 以下版本的系统中则需要将这个部分代码放入 MRC 环境中运行。


最后推荐一个很不错的 Crash 动态防护库 [XXShield](https://github.com/ValiantCat/XXShield) 以及该库的 [实现思路](https://www.valiantcat.cn/#menu_index_21)

