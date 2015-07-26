---
layout: post
title: "review.io.02"
date: 2014-05-01 08:39:37 +0800
comments: true
categories: review-io
---


##[Associated Object](http://nshipster.com/associated-objects/)[<中文_1>](http://nshipster.cn/associated-objects/)[<中文_2>](http://esoftmobile.com/2014/02/18/associated-objects/)

``` objc

#import <objc/runtime.h>

```

使用 <objc/runtime.h> 中的方法是有风险的。可以用它里面的方法来实现其它任何方法都无法实现的功能，也可以对系统产生比其它方法更加恶劣的影响。
Associated Objects/Associative References 就是其中的一个主题。其在 Objective-C 2.0 中被引入，适用于 Mac OS X 10.6 Snow Leopard (及iOS4)之后的系统环境。主要包括三个方法：

``` objc

void objc_setAssociatedObject(id object, void *key, id value, objc_AssociationPolicy policy);
id objc_getAssociatedObject(id object, void *key)
void objc_removeAssociatedObjects(id object);

```

####最主要的一点是，它允许开发者在已经存在的类的 category 中添加自定义的 property。举个栗子：

``` objc NSObject + AssociatedObject.h

@interface NSObject (AssociatedObject)
@property (nonatomic, strong) id associatedObject;
@end

```

``` objc  NSObject + AssociatedObject.m
@implementation NSObject (AssociatedObject)
@dynamic associatedObject;

- (void)setAssociatedObject:(id)object {
     objc_setAssociatedObject(self, @selector(associatedObject), object, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}

- (id)associatedObject {
    return objc_getAssociatedObject(self, @selector(associatedObject));
}

```

key 最好是 static char，是一个指针的话更好,必须保证唯一性。

``` objc

static char kAssociatedObjectKey;
objc_getAssociatedObject(self, &kAssociatedObjectKey);

```

用 selector 的话也行，是一种更加简洁的方式。

>Since SELs are guaranteed to be unique and constant, you can use _cmd as the key for objc_setAssociatedObject(). [#objective-c](https://twitter.com/search?q=%23objective&src=hash) [#snowleopard](https://twitter.com/search?q=%23snowleopard&src=hash)
>
>— Bill Bumgarner ([@bbum](https://twitter.com/bbum/statuses/3609098005)) August 28, 2009

### AssociationPolicy

objc_AssociationPolicy 参数指定了 property 以何种方式与对象关联，具体对应关系查看原文列表，或者[官方文档](file:///Users/yaongfeng/Library/Developer/Shared/Documentation/DocSets/com.apple.adc.documentation.AppleiOS7.1.iOSLibrary.docset/Contents/Resources/Documents/documentation/Cocoa/Reference/ObjCRuntimeRef/Reference/reference.html#//apple_ref/c/func/objc_setAssociatedObject)。

需要注意的是，OBJC_ASSOCIATION_ASSIGN 方式关联的 property 和 直接用 weak 修饰的 property 并不完全一样，反而更像 unsafe_unretained 修饰的property。
>According to the Deallocation Timeline described in [WWDC 2011, Session 322 (~36:00)](https://developer.apple.com/videos/wwdc/2011/#322-video), associated objects are erased surprisingly late in the object lifecycle, in object_dispose(), which is invoked by NSObject -dealloc.

### Removing Objects

####不要直接使用 objc_removeAssociatedObjects() 来移除对象，因为该方法会移除目标对象关联的所有对象。可行的方法是，调用 objc_setAssociatedObject 方法，并传递 nil 为参数。

###应该这样使用

* 添加 private variable，以便于实现细节的东西。比如，[AFNetworking 的实现中](https://github.com/AFNetworking/AFNetworking/blob/2.1.0/UIKit%2BAFNetworking/UIImageView%2BAFNetworking.m#L57-L63)，为 UIImageView category 关联了一个 AFHTTPRequestOperation 类型的 property，用于异步地从远程服务器下载图片。（用于追踪额外的信息）
* 为 category 添加 public property，以便于扩展。同样，在 [AFNetworking 的实现中](https://github.com/AFNetworking/AFNetworking/blob/2.1.0/UIKit%2BAFNetworking/UIImageView%2BAFNetworking.h#L60-L65),为 UIImageView category 关联了一个 imageResponseSerializer 类型的 property，可以在设置其 UIImage （保存到磁盘）之前，对 UIImage 做一些渲染更改，而是否更改是可选的。

* 为 KVO category 创建一个 associated observer。最好用 associated-object 作为 Observer，而不是对象自己 Observer 自己。


###不要这样使用

* 如果 associated object 的值仅仅用到一次，那么最好不要用这种方式。
* 如果 associated object 的值很容易通过其它方式获取到时，没必要这样使用。
* 使用 associated object 来替代下面的选项时，也最好别用。
    * [Subclassing](https://developer.apple.com/library/ios/documentation/cocoa/conceptual/ProgrammingWithObjectiveC/CustomizingExistingClasses/CustomizingExistingClasses.html)
    * [Target-Action](https://developer.apple.com/library/ios/documentation/general/conceptual/Devpedia-CocoaApp/TargetAction.html)
    * [Gesture Recognizers](https://developer.apple.com/library/ios/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/GestureRecognizer_basics/GestureRecognizer_basics.html)
    * [Delegation](https://developer.apple.com/library/ios/documentation/general/conceptual/DevPedia-CocoaCore/Delegation.html)
    * [NSNoification&NSNotificationCenter](http://nshipster.com/nsnotification-and-nsnotificationcenter/)

#### category 一般也不会成为最先考虑的技术，associated object 最好在别无选择时使用。一些 hack 形式的技术，很多人往往急着拿来一试身手。这样不好，最好先对其进行深入透彻的了解，再在合适的地方运用。


---------------------


##[开发一款 App 的成本](http://jianshu.io/p/86190cb19e56)

这篇文章主要讨论了开发一款 APP 需要的人力成本，组件团队的时间成本，开发时间成本，产品的运营成本，以及对于当前市场的认知。

单就开发 iphone 客户端来计算，需要 3 ~ 6 人的的人员配备，花费 1 ~ 6 个月的时间来开发。其中 1 ~ 3 个月可以开发出基本版本，继续完善为成熟版本，总共需要大概 6 个月时间。组建团队所花费的时间成本可能比人员的工资成本更高。然而，更大的成本是运营成本。产品的营销可以通过分发渠道的曝光，产品本身的价值和品质，产品的社交属性来进行，同时还面临着被大厂临摹的风险。总之，让一款产品一直稳稳的浮在水面是非常不容易的。


---------------------


##显示原中文字符串

用 Xcode 调试时，在终端会出现如下的信息：

``` sh
message = "\U5bc6\U7801 \U4e0d\U53ef\U4e3a\U7a7a\U767d.";
```
message 本来是中文字符串的，但是以上面的编码方式来显示。要看到原来的中文字符串，打开 terminal，输入如下代码：

``` sh
yaongfengdeMacBook-Air:~ yaongfeng$ zsh
yaongfengdeMacBook-Air% echo "\U5bc6\U7801 \U4e0d\U53ef\U4e3a\U7a7a\U767d."
密码 不可为空白.

```


---------------------


##[Animations Explained](http://www.objc.io/issue-12/animations-explained.html)[<中文>](http://objccn.io/issue-12-1/)

动画使 APP 中的内容以更加自然、流畅、真切的方式展现在屏幕中。在 UIKit 中，Core Animation(CA) 提供了基本的动画实现方式，CALayer 是展示动态画面大基础类。
CA 中提供了两个平行层级的图形层：the model layer tree and the presentation layer tree。文中对于这两个层的解释并不是非常的详细和准确，推荐参考下面大资料进行深入了解：

* 《Core Animation Advanced Techniques》
* [Core Animation Programming Guide](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/CoreAnimation_guide/Introduction/Introduction.html)
* [CALayer’s Parallel Universe](http://blog.spacemanlabs.com/2011/08/calayers-parallel-universe/)

另外，文中[CALayer presentationLayer]的获取方式不太准确，- (id)modelLayer;和- (id)presentationLayer;并不是类方法。
[Interpolation](https://en.wikipedia.org/wiki/Interpolation) 表示在已知的数据集中构建新的数据集的方法。
默认情况下，动画结束之后会恢复为动画刚开始时的状态。可以在动画结束的时候，额外设置一下最终状态，或者通过设置属性fillMode为kCAFillModeForward来保留最终的状态。
CAKeyframeAnimation 主要是设置 values 和 keyTimes 两个属性。
通过设置 path 属性来实现沿着复杂路径变化的动画，可以使用矢量绘图工具来协助设计炫酷的动画效果。

CAMediaTimingFunction 提供了一些常用的动画时间生成方法，也可以用如下的初始化方法自定义。

``` objc
+ (id)functionWithControlPoints:(float)c1x :(float)c1y :(float)c2x :(float)c2y
```
作者自己实现了一些可以实现物理模拟效果的 CAMediaTimingFunction —— [RBBAnimation](https://github.com/robb/RBBAnimation)


---------------------
