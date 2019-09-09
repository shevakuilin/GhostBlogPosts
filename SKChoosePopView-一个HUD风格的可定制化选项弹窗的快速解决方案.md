---
title: 两行代码搞定iOS自定义HUD风格动画弹窗(支持选择记录) - SKChoosePopView的使用和实现思路
date: 2017-03-25 10:00:51
categories: iOS Development
tags:
description: HUD风格的选项弹窗是我们在日常开发中经常会碰到的一类需求，通常因为项目周期等因素，很少会专门抽出时间来对此类弹窗进行专门的定制开发和维护。常见的情况就是google类似的效果控件，如果恰好匹配需求，效果上说得过去，那么便可以节省不少的时间和精力，但更多的情况是，我们花费了更多的时间去修改、去填坑，效果缺不见得如意，导致开发者最后不得不吐槽：还不如自己写。对于注重追求效率的开发者，似乎什么轮子都可以上路跑，只要它ok，但对于更注重用户体验和效果的开发者而言，就很难将就了，但人的精力总归有限，抽不开身怎么办呢？这个时候就很需要一款效果很赞，使用方便、简洁、可快速集成的组件了：SKChoosePopView便应运而生了。


---

# 简述

SKChoosePopView是一个HUD风格的可定制化选项弹窗的快速解决方案，集成了上、下、左、右、中5个进场方向的6种动画效果，如果不能满足你对酷炫效果的需要，SKChoosePopView同样支持自定义动画，以及选择记录、动画的开闭、点击特效、行列数量控制等。如果你觉得还不错，star支持一下吧！



### 效果图 
<img src="http://ofg0p74ar.bkt.clouddn.com/SKPopViewExample.gif" width="370" height ="665" />



# 一.如何使用


## 1.如何开始 

1.从GitHub上Clone-->[SKChoosePopView](https://github.com/shevakuilin/SKChoosePopView), 然后查看Demo (由于使用cocoaPods管理，请打开xcworkspace工程进行查看)


2.请仔细阅读下方特别指出的部分和需要注意问题


3.在项目中使用SKChoosePopView，直接将SKChoosePopView文件夹拷贝到工程中，或在podfile文件中添加pod 'SKChoosePopView'


4.SKChoosePopView基于Masonry布局，请确保你的工程里已存在Masonry，[下载地址](https://github.com/SnapKit/Masonry)


## 2.使用方法



#### 头文件导入


```objectivec
#import "SKPopView.h"
```


#### 初始化
```objectivec
/** 初始化方法
* @param title 标题数组
* @param iconNormal 默认图标数组
* @param iconSelected 选中图标数组, 若不需要传入nil即可
* @param titleColor 选中的选项标题字体颜色, 若不需要传入nil即可
* @param delegate 代理协议
* @param completion 弹窗出现后的回调操作，若不需要传入nil即可
*/
SKPopView * popView = [[SKPopView alloc] initWithOptionsTitle:kDate.title 
                                        OptionsIconNormal:kDate.normalIcons
                                    OptionsIconSelected:kDate.selectedIcons 
                                        selectedTitleColor:[UIColor orangeColor] 
                                            delegate:self  completion:^{
        // TODO: 如果这里不需要就nil
}];
```



### 显示
```objectivec
[popView show];
```


### 消失
```objectivec
[popView dismiss];
```


### 设置动画类型
```objectivec
popView.animationType = SK_TYPE_SPRING;
```


### 设置动画方向
```objectivec
popView.animationDirection = SK_SUBTYPE_FROMBOTTOM;
```


### 动画时间
```objectivec
popView.animationDuration = 0.5;
```


### 开启/关闭选择记录
```objectivec
popView.enableRecord = YES;
```


### 开启/关闭动画效果
```objectivec
popView.enableAnimation = YES;
```


### 行数设置
```objectivec
popView.optionsLine = 2;
```


### 列数设置
```objectivec
popView.optionsRow = 3;
```


### 最小行间距
```objectivec
popView.minLineSpacing = 10;
```


### 最小列间距
```objectivec
popView.minRowSpacing = 10;
```

### 动画设置
```objectivec
SK_TYPE_SPRING,// 弹簧效果
SK_TYPE_ROTATION,// 旋转效果
SK_TYPE_FADE,// 渐变效果
SK_TYPE_LARGEN,// 变大效果
SK_TYPE_ROTATION_LARGEN,// 旋转变大效果
SK_TYPE_TRANSFORMATION// 变形效果
```

### 动画进场方向
```objectivec
SK_SUBTYPE_FROMRIGHT,// 从右侧进入
SK_SUBTYPE_FROMLEFT,// 从左侧进入
SK_SUBTYPE_FROMTOP,// 从顶部进入
SK_SUBTYPE_FROMBOTTOM,// 从底部进入
SK_SUBTYPE_FROMCENTER// 从屏幕中间进入
```

### 获取已选择的选项row(代理协议方法)
```objectivec
- (void)selectedWithRow:(NSUInteger)row;
```

## 注意事项

1.`optionsLine`和`optionsRow`属性是必须设置的, 且遵循垂直布局原则，请确保optionsLine * optionsRow于选项数量相等


2.最小行、列间距如不需要可以不设置，默认为0


3.如果开启动画，请确保`animationType`、`animationDirection`和`animationDuration`属性已经设置


4.使用代理协议方法前，请确保已遵循`<SKPopViewDelegate>`


4.如果遇到其它问题，欢迎提交issues，我会及时回复



# 二.实现思路


## 1.设计思路


- 首先，在总体设计上我们采取模块化的思路，主要分为两部分：`SKPopView`（视图）和`SKPopAnimationManage`（动画管理），各司其职。


- `SKPopView`负责处理界面控件的创建和布局约束、外部对弹窗配置信息设置的响应(如是否开启动画效果、点击效果、选择记录、动画类型/方向、需要显示的行/列等)、已选择的选项回调等。


- `SKPopAnimationManage`则负责动画的相关管理，如对动画进场方向的轨迹控制、动画效果的具体实现等。



## 2.功能实现


#### 布局

- 在整体上，`SKPopView`充当了一个父`UIView`的角色，在其内部添加`grayBackground1`（灰色的背景）和`popView`（HUD弹窗部分），总体结构简单明了，所以在调用的时候我们是直接将`SKPopView`整个添加到我们需要显示的界面当中来使用的。


- 在`SKPopView`中，`grayBackground`作为灰色背景率先addSubview到`SKPopView`中`[self addSubview:grayBackground]`
，而弹窗则作为插入部分，插入到`grayBackground`之上`[self insertSubview:self.popView aboveSubview:grayBackground]`, 而在`popView`内部，我们内嵌了一个`UICollectionView`，作为选项按钮的展示。


#### 弹窗的出现与消失


- 弹窗的出现原理：当弹窗被调用时，`SKPopView`整体便会添加到当前视图的图层之上，而弹窗`popView`此时会根据调用时在初始化方法里配置的SK_TYPE(动画类型)和SK_SUBTYPE(动画方向)，在屏幕外选择一个入场前的坐标和入场时的动画效果，当初始化完成开始调用`SKPopView`的`show`方法时，`popView`便开始入场、完成动画。

```objectivec
#pragma mark - 外部调用
- (void)show
{
    if (self.enableAnimation == YES) {// 如果开启动画效果
        [self displayAnimation];
    }
}
```


```objectivec
#pragma mark - 动画设置
- (void)displayAnimation
{
    self.animationManage = [[SKPopAnimationManage alloc] init];
    switch (self.animationType) {// 动画类型
        case SK_TYPE_SPRING:
            self.animationManage.type = SK_ANIMATION_TYPE_SPRING;
            break;
        case SK_TYPE_ROTATION:
            self.animationManage.type = SK_ANIMATION_TYPE_ROTATION;
            break;
        case SK_TYPE_FADE:
            self.animationManage.type = SK_ANIMATION_TYPE_FADE;
            break;
        case SK_TYPE_LARGEN:
            self.animationManage.type = SK_ANIMATION_TYPE_LARGEN;
            break;
        case SK_TYPE_ROTATION_LARGEN:
            self.animationManage.type = SK_ANIMATION_TYPE_ROTATION_LARGEN;
            break;
        case SK_TYPE_TRANSFORMATION:
            self.animationManage.type = SK_ANIMATION_TYPE_TRANSFORMATION;
        break;
}

    switch (self.animationDirection) {// 动画进场方向
        case SK_SUBTYPE_FROMRIGHT:
            self.animationManage.animationDirection = SK_ANIMATION_SUBTYPE_FROMRIGHT;
            break;
        case SK_SUBTYPE_FROMLEFT:
            self.animationManage.animationDirection = SK_ANIMATION_SUBTYPE_FROMLEFT;
            break;
        case SK_SUBTYPE_FROMTOP:
            self.animationManage.animationDirection = SK_ANIMATION_SUBTYPE_FROMTOP;
            break;

        case SK_SUBTYPE_FROMBOTTOM:
            self.animationManage.animationDirection = SK_ANIMATION_SUBTYPE_FROMBOTTOM;
            break;

        case SK_SUBTYPE_FROMCENTER:
            self.animationManage.animationDirection = SK_ANIMATION_SUBTYPE_FROMCENTER;
            break;


        default:
        break;
}
        // 对进场动画进行设置
        [self.animationManage animateWithView:self.popView Duration:self.animationDuration animationType:self.animationManage.type animationDirection:self.animationManage.animationDirection];
}
```


- 由于支持对已选择的按钮进行记录保存，所以在使用上，用户仅能通过对灰色背景的点击来达到让弹窗消失的目的，我们通过对`grayBackground`添加点击手势来达到目的, 注意：`grayBackground`在这里是一个`UIImageView`，在添加手势时，一定要将`userInteractionEnabled = YES`，开启用户交互，否则手势将失效。

```objectivec
// 灰色背景
UIImageView * grayBackground = [UIImageView new];
[self addSubview:grayBackground];
grayBackground.backgroundColor = [UIColor colorWithWhite:0.3 alpha:0.5];;
grayBackground.userInteractionEnabled = YES;
[grayBackground mas_makeConstraints:^(MASConstraintMaker *make) {
    make.edges.equalTo(self).with.insets(UIEdgeInsetsMake(0, 0, 0, 0));
}];
// 添加手势
UITapGestureRecognizer * dismissGesture = [[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(cancel)];
[grayBackground addGestureRecognizer:dismissGesture];
```


```objectivec
#pragma mark - 手势响应
- (void)cancel
{
    [self dismissAnimation];
}
```


```objectivec
- (void)dismissAnimation
{
    if (self.enableAnimation == YES) {// 如果开启动画效果
        [self.animationManage dismissAnimationForRootView:self];// 使用离场动画
    } else {
        [self removeFromSuperview];// 直接移除整个SKPopView
    }
}
```

- 如果需求上有取消按钮，可以在取消按钮的点击方法内手动调用`SKPopView`的`dismiss`方法，十分方便。

如```objectivec
- (void)clickCancel // 取消按钮方法
{
    [popView dismiss];
}```


### 选项的数量与显示把控

- 因为我们在`popView`中镶嵌了一个`UICollectionView`，而`UICollectionView`提供了几个很好的方法可以让我们相对轻松的对选项的数量及在弹窗内的显示方式进行把控，我们主要应用了下面几个方法



```objectivec
// 设置item的size
- (CGSize)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout*)collectionViewLayout sizeForItemAtIndexPath:(NSIndexPath *)indexPath
```



```objectivec
// 最小列（横向）间距
- (CGFloat)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout *)collectionViewLayout minimumInteritemSpacingForSectionAtIndex:(NSInteger)section
```



```objectivec
// 最小行（纵向）间距
- (CGFloat)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout *)collectionViewLayout minimumLineSpacingForSectionAtIndex:(NSInteger)section
```



    然后根据我们在`SKPopView`的头文件中暴露的外部配置信息`optionsLine`(显示的行数)、`optionsRow`(显示的列数)、`minLineSpacing`(最小行间距)、`minRowSpacing`(最小列间距)，来对具体的样式进行修改，具体转换算法请参考SKPopView.m中`外部配置`标签下的代码段




### 动画



- 动画部分我们分为：进场动画、撤场动画和选项点击动画三个部分



1.进场动画：
    


- 首先我们要弄明白一点，进场动画，顾名思义：就是由进场+动画两个效果部分组成的



- 由于默认集成了来自5个方向的6种不同效果的动画，导致动画在进行相应配置时，对于每种动画效果都需要考虑5种不同的进场起始坐标，所以，我们用一个统一的方法对动画的进场方向进行统一管理，同样的，考虑到一些动画效果需要使用[动画组](https://developer.apple.com/reference/quartzcore/caanimationgroup), 对动画在进场过程中的位移路径也需要做一个统一管理



```objectivec
#pragma mark - 动画方向初始化
- (void)animationDirectionInitialize
{
    if (_animationDirection == SK_ANIMATION_SUBTYPE_FROMRIGHT) {// 从右侧进场
        _animationView.center = CGPointMake(MyWidth + 1000, WindowCenter.y);

    } else if (_animationDirection == SK_ANIMATION_SUBTYPE_FROMLEFT) {// 从左侧进场
        _animationView.center = CGPointMake(MyWidth - 1000, WindowCenter.y);

    } else if (_animationDirection == SK_ANIMATION_SUBTYPE_FROMTOP) {// 从顶部进场
        _animationView.center = CGPointMake(WindowCenter.x, MyHeight - 1000);

    } else if (_animationDirection == SK_ANIMATION_SUBTYPE_FROMBOTTOM) {// 从底部进场
        _animationView.center = CGPointMake(WindowCenter.x, MyHeight + 1000);

    } else {// 中间进场
        _animationView.center = WindowCenter;
    }

}

#pragma mark - 动画通用位移
- (CAKeyframeAnimation *)partOfTheAnimationGroupPosition:(CGPoint)startPosition
{
    CAKeyframeAnimation * animation = [CAKeyframeAnimation animationWithKeyPath:@"position"];// 创建关键帧位移动画
    NSValue * startValue, * endValue;
    startValue = [NSValue valueWithCGPoint:startPosition];// 起始坐标
    endValue = [NSValue valueWithCGPoint:WindowCenter];// 结束坐标
    animation.values = @[startValue, endValue];// 设置动画路径
    animation.duration = _animationDuration;// 持续时间

    return animation;
}
```




- 其中`弹簧效果`的动画由于不是并发动画效果，所以是没有用到`通用位移`的，所以它的位移方法是独立出来的




```objectivec
#pragma mark - 弹簧效果
- (void)springAnimation
{
    [self animationDirectionInitialize];// 初始化方向
    [self displacementWithStartPosition:_animationView.center];
}

/** 弹簧效果位移部分
* @param startPosition 位移起始坐标
*/
- (void)displacementWithStartPosition:(CGPoint)startPosition{
    CAKeyframeAnimation * animation = [CAKeyframeAnimation animationWithKeyPath:@"position"];// 创建关键帧动画
    NSValue * startValue, * endValue;
    startValue = [NSValue valueWithCGPoint:startPosition];
    endValue = [NSValue valueWithCGPoint:WindowCenter];
    animation.values = @[startValue, endValue];
    animation.duration = _animationDuration;
    animation.delegate = self;// 设置CAAnimationDelegate
    [_animationView.layer addAnimation:animation forKey:@"pathAnimation"];
}

/** 弹簧效果晃动部分
*/
- (void)partOfTheSpringGroupShaking
{
    NSString * keyPath = @"";
if (_animationDirection == SK_ANIMATION_SUBTYPE_FROMLEFT || _animationDirection == SK_ANIMATION_SUBTYPE_FROMRIGHT) {// 判断弹窗的来向
    keyPath = @"transform.translation.x";
} else {
    keyPath = @"transform.translation.y";
}
    CABasicAnimation * animation = [CABasicAnimation animationWithKeyPath:keyPath];
    animation.fromValue = [NSNumber numberWithFloat:-20.0];
    animation.toValue = [NSNumber numberWithFloat:20.0];
    animation.duration = 0.1;
    animation.autoreverses = YES;// 是否重复
    animation.repeatCount = 2;// 重复次数
    [_animationView.layer addAnimation:animation forKey:@"shakeAnimation"];
}

- (void)animationDidStop:(CAAnimation *)anim finished:(BOOL)flag// 监听动画停止
{
    [self partOfTheSpringGroupShaking];
}
```




2.撤场动画



- 撤场动画就相对简单很多, 由于撤场方式是统一的，只需要对撤出坐标做处理，然后removeFromSuperview



```objectivec
- (void)dismissAnimationForRootView:(UIView *)view
{
    [UIView animateWithDuration:0.5 animations:^{
        _animationView.center = CGPointMake(WindowCenter.x, MyHeight + 1000);

    } completion:^(BOOL finished) {
        [view removeFromSuperview];
    }];
}
```




3.选项点击动画



- 当点击动画效果被开启时，才会调用这个方法，即`enableClickEffect = YES`时, `SKPopViewCollectionViewCell`中会调用点击动画



```objectivec
- (void)setEnableClickEffect:(BOOL)enableClickEffect
{
    _enableClickEffect = enableClickEffect;
    if (enableClickEffect == YES) {
        SKPopAnimationManger * animationManger = [[SKPopAnimationManger alloc] init];
        [animationManger clickEffectAnimationForView:self.basementView];
    }
}
```




而在`SKPopAnimationManger`中，我们只是巧妙的对其做了一个缩放的效果，即可达到果冻式的触感



```objectivec
- (void)clickEffectAnimationForView:(UIView *)view
{
    CABasicAnimation * scaleAnimation = [CABasicAnimation animationWithKeyPath:@"transform.scale"];
    scaleAnimation.fromValue = [NSNumber numberWithFloat:1];
    scaleAnimation.toValue = [NSNumber numberWithFloat:0.8];
    scaleAnimation.duration = 0.1;
    scaleAnimation.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseInEaseOut];
    [view.layer addAnimation:scaleAnimation forKey:nil];
}
```



# 感谢你花时间阅读以上内容, 如果这个项目能够帮助到你，记得告诉我
Email: shevakuilin@gmail.com


或者直接在文章下留言哦~
