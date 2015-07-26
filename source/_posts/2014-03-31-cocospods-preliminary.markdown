---
layout: post
title: "CocoaPods —— 高效管理Xocde工程所依赖的第三方开源库(CocoaPods Guides)"
date: 2014-03-31 15:29:06 +0800
comments: true
categories: CocoaPods
---


###说明

对于CocoaPods的安装使用，[其官方网站](http://guides.cocoapods.org/)中有详细的说明。也有一些中文Blog中的文章，翻译、
总结了CocoaPods的使用技巧。既然有许多现成的学习资源，就没有必要再造轮子，自己闷头搞，这样吃力也没得好可讨。因此本篇文章的目
的是，搜罗并整理相关的资源。


---------------------------


### [cocoapods.org](http://guides.cocoapods.org/)

官方英文教材已经非常的详细和完善了，但是对于英文不是母语，又需要在短时间内将其用于现有项目中的开发者来说，这并不是最佳的选择。
我个人觉得，她适宜在时间比较充裕的时候仔细探究，如若兴趣使然，能力所及，还可以为其贡献源代码。

待细读。。。


---------------------------


### [Cocoapods 101](http://geeklu.com/2013/06/cocoapods-101/)

这篇技术博文介绍了CocoaPods简单使用。简单的介绍了下子CocoaPods的背景之后，比较详细的描述了其安装过程，包括rbenv的安装，ruby
的更新安装。最后通过一个简单的实例，用pod自动导入AFNetwork到项目中。

对于不知道的童靴，这个细节值得关注下：淘宝提供一个国内的[RubyGems镜像地址](http://ruby.taobao.org/),解救深受间接性网络连接失败
困扰的民众（15分钟更新一次）。如若这样，在安装CocoaPods前，需要添加新的Gem源地址，并且删掉默认的源地址。如下

```sh 更改Gem源地址为国内淘宝镜像地址
   $ gem sources --remove https://rubygems.org/
   $ gem sources -a http://ruby.taobao.org/
   $ gem sources -l   # *** CURRENT SOURCES ***

   # http://ruby.taobao.org
   # 请确保只有 ruby.taobao.org

   $ gem install cocoapods
```


---------------------------


###[使用CocoaPods来做iOS程序的包依赖管理](http://blog.devtang.com/blog/2012/12/02/use-cocoapod-to-manage-ios-lib-dependency/)

巧哥的这篇还是比较全面的，从其他语言的代码依赖管理工具引入，用他们团队开发的粉笔网客户端为例，阐释了为什么应该使用CocoaPods,以及其带来的好处。接着描述了CocoaPods的安装过程，用实例说明了如何用其导入[CocoaPods/Specs](https://github.com/CocoaPods/Specs)中提供的第三方库。另外，还介绍了pod search命令——根据关键字快速查看相匹配库的信息。当然，甄选优质的同类型库需要咨询使用过的开发者或者更进一步的实践。文章还提到，为了利于团队协作，不能将Podfile.lock加入.gitignore文件中，如果安装appledoc工具，CocoaPods还可以帮助生成帮助文档。最后，简单的阐释了下子CocoaPods的内部原理。


---------------------------


###[CocoaPods进阶：本地包管理](http://www.iwangke.me/2013/04/18/advanced-cocoapods/)

这篇文章列出了实际使用过程中遇到的两个问题，之后依次通过实例，细致的描述了相应的解决方案。对于[CocoaPods/Specs](https://github.com/CocoaPods/Specs)中提供的三方库版本更新滞后的问题，可以通过自己创建最新版本的PodSpec来解决。另外，对于不便于公开的代码库，可以通过本地路径来引入。两者的关键都在于创建新的*.podspec文件，并在该文件添加所引用库的路径，之后在Podfile文件中导入新建的podspec。最后还提到weak_frameworks选项的用途。


---------------------------



###[CocoaPods Under The Hood](http://www.objc.io/issue-6/cocoapods-under-the-hood.html)

objc.io 系列的文章真的是太好了，当然这一篇也不例外。我的简体中文翻译版本：[揭开CocoaPods的神秘面纱](http://yaonphy.github.io/blog/2014/04/02/cocoapods-under-the-hood/)

文章介绍了 CocoaPods 的几个核心组件：CocoaPods/CocoaPod，CocoaPods/Core，CocoaPods/Xcodeproj。之后通过 pod install --verbose 命令打印出信息入手，分析了详细的执行流程。之后又谈到 CocoaPod 如何在持续集成环境中更好的发挥作用。


---------------------------



###[Using CocoaPods to Modularize a Big iOS App](http://dev.hubspot.com/blog/architecting-a-large-ios-app-with-cocoapods)


这里有我的一篇翻译文章：[使用 CocoaPods 来模块化一个复杂庞大的 iOS App](http://yaonphy.github.io/blog/2014/04/02/using-cocoapods-to-modularize-big-app/)，仅供参考。

该文以作者开发的 HubSpot 项目为例，介绍了如何用 CocoaPods 来构建、管理功能丰富的大型 App。对于各个子功能之间耦合度比较松散，但是需要集成到一个整体 App 的情况，用 CocoaPods 将整体 App 模块化为若干个 sub-app，不但简化了架构，便于子功能层次的测试，还可以让代码管理变得更加轻松。

文章通过具体的代码，分享了如何用 CocoaPods 将各个 sub-app 粘合起来，构建完整的应用。


---------------------------



###[CocoaPodUI](https://github.com/Galeas/CocoaPodUI)

这是一个 CocoaPod 图形化界面的 Xcode 插件，感兴趣的话可以试试。我目前还没用过。


---------------------------


###[CocoaPods Specs 国内镜像](http://akinliu.github.io/2014/05/03/cocoapods-specs-/)

[@阿宽](http://weibo.com/325521517) 分享了两个国内的 CocoaPod Specs 镜像地址，分别在 [gitcafe](https://gitcafe.com/akuandev/Specs.git) 和 [oschina](http://git.oschina.net/akuandev/Specs)，每隔 10 分钟更新一次。

更改方法：

``` sh
pod repo remove master
# 使用 oschina 的地址，换为 http://git.oschina.net/akuandev/Specs.git
pod repo add master https://gitcafe.com/akuandev/Specs.git
pod repo update
```

第二条命令执行的时间比较长，会 clone 整个 repository。


---------------------------



####*2014.05.04 更新*






