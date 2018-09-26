---
title: Define 宏
date: 2018-01-18 16:21:20
tags: [animation, amber, blog, uiview]
---


# 首要任务

针对 Core Animation 进行探讨

虽然很多东西也可以用更高层级的 UIKit 的方法来完成，但是 Core Animation 将会让你更好的理解正在发生什么。它以一种更明确地方式来描述动画。

再看动画如何与我们在屏幕上看到的内容交互之前，我们需要快速浏览一下 Core Animation 的 CALayer，这是动画产生作用的地方。

UIView 实例以及 layer-backed 的 NSView，修改他们的 layer 来委托强大的 Core Graphics 框架来进行渲染。然而，你务必要理解，当把动画添加到一个 layer 时，是不直接修改它的属性的。

取而代之，Core Animation 维护了两个平行 layer 层次结构：model layer tree （模型层树）和 presentation layer tree (表示层树)。前者中的 layers 反映了我们能直接看到的 layers 的状态，而后者的 layers 则是动画正在表现的值的近似。

实际上还有所谓的第三个 layer 树，叫做 rendering tree（渲染书）。因为它对 Core Animation 而言是私有的，所以暂时不讨论。

考虑在 view 上增加一个渐出动画。如果在动画中的任意时刻，查看 layer 的 opacity 值，你是得不到与屏幕内容对应的透明度的。取而代之，你需要查看 presentation layer 以获得正确的结果。

虽然你可能不会去直接设置 presentation layer 的属性，但是使用它的当前值来创建新的动画，或者在动画发生时与 layers 交互是非常有用的。

通过使用 - [CALayer presentationLayer] 和 - [CALayer modelLayer]，你可以在两个 layer 之间轻松切换。

# 基本动画

使用 CABasicAnimation 实现 线性动画

```
CABasicAnimation *animation = [CABasicAnimation animation];
animation.keyPath = @"position.x";
animation.fromValue = @77;
animation.toValue = @455;
animation.duration = 1;
 
[rocket.layer addAnimation:animation forKey:@"basic"];
```

注意我们的动画的键路径，也就是 position.x，实际上包含一个存储在 position 属性中的 CGPoint 结构体成员。这是 Core Animation 一个非常方便的特性。

**完整列表
https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CoreAnimation_guide/Key-ValueCodingExtensions/Key-ValueCodingExtensions.html**

然而，当我们运行该代码时，我们意识到完成动画后对象马上回到了初始位置。这是因为在默认情况下，动画不会在超出其持续时间后还修改 presentation layer。实际上，在结束时它甚至会被彻底移除。

一但动画被移除，presentation layer 将回到 model layer 的值，并且因为我们从未修改该 layer 的 postion 属性，所以对象将重新出现在它开始的地方。

这里有两种解决这个问题的方法：
1. 直接在 model layer 上更新属性。这个是推荐的做法，因为它是的动画完全可选。
一旦动画完成并且从 layer 中移除，presentation layer 将回到 model layer 设置的值，而这个值抢号与动画最后一个步骤相匹配。

恰好与动画最后一个步骤相匹配。


```
CABasicAnimation *animation = [CABasicAnimation animation];
animation.keyPath = @"position.x";
animation.fromValue = @77;
animation.toValue = @455;
animation.duration = 1;
 
[rocket.layer addAnimation:animation forKey:@"basic"];
 
rocket.layer.position = CGPointMake(455, 61);
```

或者，你可以通过设置动画的 fillModel 属性为 kCAFillModeForward 以留在最终状态，并设置 removedOnCompletion 为 NO 以防止它被自动移除：


```
CABasicAnimation *animation = [CABasicAnimation animation];
animation.keyPath = @"position.x";
animation.fromValue = @77;
animation.toValue = @455;
animation.duration = 1;
 
animation.fillMode = kCAFillModeForward;
animation.removedOnCompletion = NO;
 
[rectangle.layer addAnimation:animation forKey:@"basic"];
```

如果将已完成的动画保持在 layer 上时，会造成额外的开销，因为渲染器回去进行额外的绘画工作。

值得提出的是，实际我们创建的动画对象被添加到 layer 时立刻就复制了一份。这个特性在多个 view 中重用动画时这非常有用。比如想要第二个对象在第一个动画不久后也开始动画：

```
CABasicAnimation *animation = [CABasicAnimation animation];
animation.keyPath = @"position.x";
animation.byValue = @378;
animation.duration = 1;
 
[rocket1.layer addAnimation:animation forKey:@"basic"];
rocket1.layer.position = CGPointMake(455, 61);
 
animation.beginTime = CACurrentMediaTime() + 0.5;
 
[rocket2.layer addAnimation:animation forKey:@"basic"];
rocket2.layer.position = CGPointMake(455, 111);
```

设置动画的 beginTime 为未来 0.5秒将智慧影响 rocket2，因为动画在执行语句 [rocket1.layer addAnimation:animation forKey:@"basic"]; 时已经被赋值了，并且之后 rocket1 也不会考虑对动画对象的改变。

我决定再使用 CABasicAnimation 的 byValue 属性创建一个动画，这个动画从 presentation layer 的当前值开始，加上 byValue的值后结束。这使得动画更易于重用，因为你不需要精确的指定可能无法提前知道的 from- 和 toValue 的值。

fromValue, byValue 和 toValue 的不同组合可以用来实现不同的效果

文档：
**https://developer.apple.com/reference/quartzcore/cabasicanimation#//apple_ref/doc/uid/TP40004496-CH1-SW4**

# 多步动画

如果想要为你的动画定义超过两个步骤，可以使用更通用的 CAKeyframeAnimation，而不是链接多个 CABasicAnimation 实例。

关键帧(keyframe) 使我们能够定义动画中任意的一个点，然后让 Core Animation 填充所谓的中间帧。

比如晃动的效果：

```
CAKeyframeAnimation *animation = [CAKeyframeAnimation animation];
animation.keyPath = @"position.x";
animation.values = @[ @0, @10, @-10, @10, @0 ];
animation.keyTimes = @[ @0, @(1 / 6.0), @(3 / 6.0), @(5 / 6.0), @1 ];
animation.duration = 0.4;
 
animation.additive = YES;
 
[form.layer addAnimation:animation forKey:@"shake"];
```

values 数组定义了表单应该到哪些位置。

设置 keyTimes 属性让我们能够制定关键帧动画发生的时间。他们被指定为关键帧动画总只需时间的一个分数。

请注意我


## 参考

http://ios.jobbole.com/69111/

http://wiki.jikexueyuan.com/project/ios-core-animation/

