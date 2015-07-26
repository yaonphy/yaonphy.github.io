---
layout: post
title: "Unwind Segues"
date: 2014-05-20 15:04:06 +0800
comments: true
categories: iOSTips
---


##简述

Unwind segues 用于当前的 viewcontroller 与之前的 viewcontroller 之间的界面过渡与数据交互。一个 unwind segue 中需要的用于界面过渡的信息如下：

* sender，引发界面转换事件的对象，比如一个 button，一个 cell。
* source view controller,当前所在的 view controller。
* unwind action，destination view controller 中的响应 action。
* ID string,标示 segue 的唯一性。

<div class="alert alert-info">
<p>
    <span class="glyphicon glyphicon-info-sign"></span>
   上述信息中，并不包含跳回的具体 destination view controller。因为用户可能以随意的方式跳回之前的 controller，destination view controller 必须是在 runtime 阶段才确定的。
</div>


--------


##在 storyboard 中添加 unwind segue

### 首先在 destination view controller 中添加一个 IBAction 类型的方法。

destination view controller 是继承自 UIViewController 的，sender 类型精确为 UIStoryboardSegue，后面拖拽时才会识别到。

### 当前 view controller 中添加自动触发的 segue

自动触发类型的 segue，也就是 touch 一个 button 或者 Cell 等时，自动寻找相应的 segue，进行跳转，无需写其它代码。
按住 control 键，同时从 button，cell 或者其它 item 拖拽到当前 view controller 的 exit 绿色图标上，如下：

![figure 1](https://developer.apple.com/library/ios/technotes/tn2298/Art/tn2298_ExitIcon.png)

然后会弹出上一步中设置的 action 列表供你选择，选择想要跳往的 controller 中定义的 action 即可。

### 当前 view controller 中添加程序触发的 segue

这种类型的 segue，需要在程序中调用 performSegueWithIdentifier:sender 方法来寻找需要的 segue。

按住 control 键，同时从当前 view controller 的 controller 黄色 icon 拖拽到 exit 绿色图标上，如下：

![figure 1](https://developer.apple.com/library/ios/technotes/tn2298/Art/tn2298_SceneToExitIcon.png)

同上，在弹出的 action 列表中选择需要的即可。



--------


## unwind 的处理过程

* 一个 view controller 被选择为 destination view controller。
* 当前 view controller 中的 prepareForSegue:sender: 方法被调用，默认不会做任何操作，如果想要传递数据，复写这个方法。
* destination view controller 中相应的 action 被触发。
* unwind segue 进行当前 controller 和 destination view controller 间的界面过渡转换。

Container viewcontroller 中使用 unwind segue，需要额外的设置，自定义 Container viewcontroller 中的情况下面会讨论到。



--------


## 一个 unwind segue 是如何选择它的的 Destination 的？

在 unwind segue 定义时确定了所对应的 unwind action，在层级结构中距离其最近的实现该 action 的 view controller，便成为当前 view controller 的 destination view controller。如果找不到相应的 destination，unwind segue 将取消。搜寻顺序如下：

* viewControllerForUnwindSegueAction:fromViewController:withSender: 方法发送给当前（source）view controller 的 parent controller。
    
* 如果 parent controller 中没有实现相应的 action，接着会依次遍历其所有的 child controller , canPerformUnwindSegueAction:fromViewController:withSender: 方法用来检测某个 controller 是否能是 segue所要找的。
* 如果还没找到，viewControllerForUnwindSegueAction:fromViewController:withSender: 发送给下一个 parent view controller，依次类推。

在层级结构中，如果一个 destination view controller class 有多个实例化的对象，实现 canPerformUnwindSegueAction:fromViewController:withSender: 方法可以准确的定位。

## 自定义 Container View Controller 中使用 unwind segue

自定义的 Container View Controller 需要完成两件事，会在下面分别讨论。如果使用的是系统 SDK 提供的 Container，比如 UINavigationViewController，这些会被自动处理的。

### 选择一个子 view controller 来处理 unwind action

主要是复写下面的方法来搜寻能够处理相应 unwind action 的controller：

``` objc
// 通常使用 canPerformUnwindSegueAction:fromViewController:withSender: 方法来确定一个 view controller 是否能够处理给定的 action。

- (UIViewController *)viewControllerForUnwindSegueAction:(SEL)action fromViewController:(UIViewController *)fromViewController withSender:(id)sender

```

如果该 Container view controller 的所有子 controller 都无法处理，则交给该 Container 的父 controller 去搜寻，以此类推。

### Transitioning Between Two Child View Controllers

source view controller 和 destination view controller 之间的关系决定了运行时两个 controller 的 UI 之间各种各样的过渡效果。如果 source view controller 是 presented，那么它，以及它与 destination view controller 之间的所有 controller，都会被 dismissed。其它的情况，通过向 the destination view controller 的 parent 获取的 UIStoryboardSegue 来处理过渡过程的所有操作。因此需要复写下面的方法：

``` objc
- (UIStoryboardSegue *)segueForUnwindingToViewController:(UIViewController *)toViewController fromViewController:(UIViewController *)fromViewController identifier:(NSString *)identifier
```
<div class="alert alert-info">
<p>
    <span class="glyphicon glyphicon-info-sign"></span>
    source view controller 和 destination view controller 之前可能不是直接的过渡关系，它们中间可以存在其它的 controller。
</div>



--------


##preference

* [What are Unwind segues for and how do you use them?](http://stackoverflow.com/questions/12561735/what-are-unwind-segues-for-and-how-do-you-use-them)
* [Technical Note TN2298 Using Unwind Segues](https://developer.apple.com/library/ios/technotes/tn2298/_index.html#//apple_ref/doc/uid/DTS40013591-CH1-UNWINDPROC)
* [Adopting Storyboards in Your App](https://developer.apple.com/videos/wwdc/2012/?id=407)
