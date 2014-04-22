---
layout: post
title: "使用 CocoaPods 来模块化一个复杂庞大的 iOS App"
date: 2014-04-02 18:33:20 +0800
comments: true
categories: CocoaPods
---


####这是一篇对英文技术文章 [Using CocoaPods to Modularize a Big iOS App](http://dev.hubspot.com/blog/architecting-a-large-ios-app-with-cocoapods)的简体中文翻译，仅供学习交流。
####原作者：[Anthony Roldan](http://dev.hubspot.com/blog/author/anthony-roldan)


为你的移动 App 选择正确的架构确实是一个大问题。架构会影响你个日常开发流程，决定了你在开发中可能遇到的问题，可以让你变得富可敌国，也可以让你变得穷困潦倒。

 HubSpot 的移动端 App 功能非常的丰富。它是一个分析统计 App，也是一个社交媒体 App，还是一个邮件 App，还是一个联系人管理类 App（多对一），所有的这些都同处于一个屋檐下。去年夏天，当我们计划开发这个相当复杂的 App 时，我们知道我们必须有一个与其规模相符合的架构。
 
 我们最终把每一个 sub-app 当作一个完全独立的 App 来开发，然后用 [CocoaPods](http://cocoapods.org/) 来将它们集成到 main App 当中。
 
 从下面的截图中，你可以看到，Sources, Dashboard, Social Media 这些 sub-app 都是独立的 iPhone App，同时它们又是 main App 中的组成模块。
 
 ![哈哈](http://cdn2.hubspot.net/hub/51294/file-435767237-png/app_example.png?t=1396302307000)

这给我们带来了很多优势：

* 最关键的是，我们能够确定，每个 sub-app 的主分支是稳定可用的，而且 sub-app 的某个特定版本能够很快的被集成到 main app 中。
* 我们花了更多的时间去 build，花费更少的时间去 merge 。沙盒使得每个 sub-app 内的迭代变得非常容易，而且能使不同 app 之间的集成花费更少的时间。如果你在多于一个人的 iOS 开发团队工作过，你肯定遇到过大量的 .xcodeproj 合并问题。尽管这是[可以解决的](http://stackoverflow.com/questions/2615378/how-to-use-git-properly-with-xcode)，但是过程非常的令人痛苦。使用 CocoaPods 来构建应用，几乎让我们完全远离了这种痛苦。
* 如果需要，我们可以打包部署每个独立的 app ,这对于独立 app 层次的功能测试来说，简直爽爆了。在类似 Navigation 这样的粘合功能没有被完成之前，我们都可以为测试人员提供独立的 app 来测试。这让我们能够收集到高质量，有针对性的反馈。
* 由于 sub-apps 之间的用户使用流程是通过基于URL的路径(稍后讨论)来实现，这意味着路径是内置的，并且是规划好的。而不是通过检索一大堆 UIViewController 来寻找需要显示的 View 。这对于一些基本功能，比如引导教程和新的推送通知，是非常有帮助的。

这种架构方式，对于构建功能丰富多样的 iOS App 的多人协作团队来说，可以节省大量时间。听起来很合你的口味? 那就别停下来。


---------------------------------


###灵感来自于 Web 架构


将我们的移动应用分解成多个子应用的灵感，来自于已经非常成功的 HubSpot 的 web 架构。

HubSpot 的 web 架构，专为快速开发和可扩展性而设计。就像我的同事在[一篇文章中](http://dev.hubspot.com/blog/how-we-deploy-300-times-a-day)所写的一样，我们使用各种工具和技术，使得我们的整体部署达到300次/每天。最主要的是，HubSpot 的产品系列是由一些松耦合，完全不同功能的应用组成——分析统计，社交媒体，邮件，博客，以及报告工具。

在Web端，我们可以独立地构建、测试、部署 HubSpot 应用中的一小部分。包括后台的 APIs 和 Java 实现的功能，前端的 CoffeeScript 项目，以及 Python 项目。移动端为什么也不这样做呢？


---------------------------------


###用 CocoaPods 吧


[CocoaPods](http://cocoapods.org/),为iOS而生的极佳依赖管理工具，是将所有东西粘合到一块的关键。


一个多应用架构对于你的需求来说可能是大材小用，但是 CocoaPods 绝对不是这样的。就算你仅仅想添加一些第三方库到项目中时，比如应用统计的库，View 组件，网络库等，它能在短短几分钟的时间内帮你搞定一切设置，节省不少时间。Ruby 风格的语法，能无缝地集成一些第三方库到项目中。

核心的库和可以分享的资源，比如登录，定义了样式的类，访问认证接口，连同 [Kiwi](https://github.com/allending/Kiwi) 测试和一个 podspec 文件,构建成一个独立的工程。将其发布到我们的一个私有 [Repo](http://guides.cocoapods.org/making/private-cocoapods.html) 中,然后在我们的完整项目中引入。更进一步，每一个子应用连同一个 Podspec 文件单独构建成一个工程之后，用 CocoaPods 将它们构建到一个完整的工程中。

这就意味着，我们可以在一个单独的子应用内部构建测试版本，进行快速的迭代开发，而不用考虑会影响到其它不相关的子应用的开发。

我们整体的应用的 Podfile 应该是这个样子的：

{% gist 8224121 %}



---------------------------------


###将所有的东西粘合到一块


聪明的读者一定注意到了，我们在主应用中使用了很多的第三方开源工具，比如 [IIViewDeck](https://github.com/Inferis/ViewDeck) 和 [JLRoutes](https://github.com/joeldev/JLRoutes)。这也是将所有的子应用粘合到一块的关键。

我们不需要在主应用中提供每个子应用能够处理的不同menu items 和 routes 信息。每一个子应用提供了一个单独的类，这个类实现了一个 HSBaseApp protocol 和一些方法，如下：

{% gist 8224163 %}

下面给出一个具体的实现例子：

{% gist 8224273 %}

我们使用 routes 来实现可能出现的 push 通知，我们同样使用这种模式来将子应用串到主应用中。比如这样的场景——我们从 Source 或者 Social Media 中跳到 Contact 。

HSRoutingDelegate 在绕过当前活动的 UINavigationController 时，施展了一些小魔法。因此，我们可以根据特定的场景，Push 某个 controller 到屏幕顶层，或者创建一个模型化的 controller 。然而，它只是对 JLRoutes 的漂亮的基于 Block 的语法的简单封装。


###我们还可以做些什么？


从长远的角度来看，我们想把简单的 Kiwi 测试添加到一些共享的库中，并且构建到 KIF 测试中。这样每一个通过 Kiwi 和 KIF 测试的子应用都可以构建到持续集成环境中。我们可以将当前最好的子应用版本拼合起来构成主应用的发布版本。

你是如果管理多人团队的大型 iOS Apps 的开发的？有没有更好的方式？我们非常期待！


####（本文完）





