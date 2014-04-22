---
layout: post
title: "揭开CocoaPods的神秘面纱"
date: 2014-04-02 10:32:50 +0800
comments: true
categories: CocoaPods
---



**本文是 objc 系列文章中 [Issue#6 BuildTools -- CocoaPods Under The Hood](http://www.objc.io/issue-6/cocoapods-under-the-hood.html) 的中文简体翻译，仅用于学习交流。原作者是 [Michele Titolo](https://twitter.com/micheletitolo)**


CocoaPods用于管理OSX和iOS应用程序中的第三方库。有了CocoaPods，你就可以自定义一种叫做Pods的依赖文件了，并且可以在不同的时间点和不同的开发环境下轻松的管理他们的版本。

CocoaPods的设计理念包括两个方面。首先，在自己的项目中导入第三方代码时可能会出现不少的“坑”。对于Objective-C的新手开发者来说，配置工程文件是一件很让他们抓狂的事情。在手动配置程序构建阶段的选项和连接器参数的过程中，很容易出现人为的错误。CocoaPods能够自动的配置编译器的各种设置选项，简化了所有的配置工作。

其次，使用CocoaPods可以更加容易的发现新的第三方库。但是，这并不是意味着你可以构建一个叫FrankApp的应用，而应用里的每一部分都是别人代码的简单拼凑。也不是说你可以发现能够缩短开发周期并能提升你的应用程序质量的优质代码库。


---------------------------


##核心组件


CocoaPods 实际上是由一些 Ruby Gems 构成的一个 Ruby 应用程序。探索其内部构成最需要关注的 gems 有：[CocoaPods/CocoaPods](https://github.com/CocoaPods/CocoaPods/), [CocoaPods/Core](https://github.com/CocoaPods/Core), [CocoaPods/XCodeproj](https://github.com/CocoaPods/Xcodeproj)（是的，CocoaPods 是由依赖管理工具构建成的依赖管理工具）。


###CocoaPods/CocoaPod


这是使用者直接调用的一个组件。任何时候，你调用一个 pod 命令，它便处于活动状态。它里面囊括了使用 CocoaPods 时用到的所有功能，充分利用其它 gems 来完成任务。


###CocoaPods/Core


Core gem 提供了对 CocoaPods 中的一些文件的处理操作，主要是 Podfile 和 podspecs 。

####Podfile

Podfile 是用来定义你想使用的pods的文件。它是可以高度自定义的，你可以根据具体需求，创建相应的pods文件。更多信息，请查看[Podfile guide](http://guides.cocoapods.org/syntax/podfile.html) 。

####Podspec

*.podspec 文件中定义了将某个具体的 pod 添加到工程中时所需的信息。这些信息包括源文件路径，使用到的官方框架，编译器参数，以及该 pod 所依赖的其它 pod 的细信息，等等。


###CocoaPods/Xcodeproj


这个 gem 用来处理所有工程文件之间的交互操作。它可以创建并修改 *.xcodeproj 和 *.xcworkspace 文件。它同样可以作为一个独立的 gem 来使用。如果你想通过脚本来轻松的修改工程文件，它可以帮到你。


---------------------------


##运行 pod install 命令


运行 pod install 命令时，进行了很多的操作。在运行该命令时，附加一个 - -verbose 参数，可以观察到整个内部的详细执行流程。现在，让我们运行 pod install --verbose 命令，将会出现下面所示的信息：

``` sh  通过 pod install - - verbose 命令观察详细执行流程
$ pod install --verbose

Analyzing dependencies

Updating spec repositories
Updating spec repo 'master'
  $ /usr/bin/git pull
  Already up-to-date.


Finding Podfile changes
  - AFNetworking
  - HockeySDK

Resolving dependencies of 'Podfile'
Resolving dependencies for target 'Pods' (iOS 6.0)
  - AFNetworking (= 1.2.1)
  - SDWebImage (= 3.2)
    - SDWebImage/Core

Comparing resolved specification to the sandbox manifest
  - AFNetworking
  - HockeySDK

Downloading dependencies

-> Using AFNetworking (1.2.1)

-> Using HockeySDK (3.0.0)
  - Running pre install hooks
    - HockeySDK

Generating Pods project
  - Creating Pods project
  - Adding source files to Pods project
  - Adding frameworks to Pods project
  - Adding libraries to Pods project
  - Adding resources to Pods project
  - Linking headers
  - Installing libraries
    - Installing target 'Pods-AFNetworking' iOS 6.0
      - Adding Build files
      - Adding resource bundles to Pods project
      - Generating public xcconfig file at 'Pods/Pods-AFNetworking.xcconfig'
      - Generating private xcconfig file at 'Pods/Pods-AFNetworking-Private.xcconfig'
      - Generating prefix header at 'Pods/Pods-AFNetworking-prefix.pch'
      - Generating dummy source file at 'Pods/Pods-AFNetworking-dummy.m'
    - Installing target 'Pods-HockeySDK' iOS 6.0
      - Adding Build files
      - Adding resource bundles to Pods project
      - Generating public xcconfig file at 'Pods/Pods-HockeySDK.xcconfig'
      - Generating private xcconfig file at 'Pods/Pods-HockeySDK-Private.xcconfig'
      - Generating prefix header at 'Pods/Pods-HockeySDK-prefix.pch'
      - Generating dummy source file at 'Pods/Pods-HockeySDK-dummy.m'
    - Installing target 'Pods' iOS 6.0
      - Generating xcconfig file at 'Pods/Pods.xcconfig'
      - Generating target environment header at 'Pods/Pods-environment.h'
      - Generating copy resources script at 'Pods/Pods-resources.sh'
      - Generating acknowledgements at 'Pods/Pods-acknowledgements.plist'
      - Generating acknowledgements at 'Pods/Pods-acknowledgements.markdown'
      - Generating dummy source file at 'Pods/Pods-dummy.m'
  - Running post install hooks
  - Writing Xcode project file to 'Pods/Pods.xcodeproj'
  - Writing Lockfile in 'Podfile.lock'
  - Writing Manifest in 'Pods/Manifest.lock'

Integrating client project
```

打印出了很多的信息，但是分解之后，整个流程显得清晰明了。下面，让我们来逐一分析。


###读取 Podfile 文件


如果你曾经感觉到 Podfile 文件中的语法看起来特别古怪，那是因为你正在写的是 Ruby 。相比其它的可用语言，它是一种更加简单的领域特定语言(DSL：Domain Specific Language)。

因此，安装过程中的第一步应当是搞清楚文件中显示地或者隐式地定义了那些 pods 。通过加载 podspecs ，CocoaPods 检索并建立一个包含所有这些 pods 和 它们的版本的列表。Podspecs 被存放在本地的 ~/.cocoapods 目录下。


###版本和冲突


CocoaPods 使用由 [Semantic Versioning](http://semver.org/) 定义的规则来解决依赖库的版本问题。由于这种冲突解决方案可以依赖于补丁版本间的非大量更改，所以可以更加容易的解决依赖问题。比如说，两个不同的 pods 依赖于两个不同版本的 CocoaLumberjack 。如果其中一个依赖于 2.3.1，而另一个依赖于2.3.3版本，解析器会使用最新的2.3.3版本，因为该版本是向后兼容2.3.1版本的。

但是，这并不总是有效。有很多的库不使用这种依赖规则，使得解决版本和冲突问题变得非常困难。

当然，也有很多手动解决冲突的方式。如果一个库依赖于 CocoaLumberjack 的1.2.5 版本，而另一个依赖于 2.3.1，最终的使用者可以通过设置一个确定的版本来解决冲突。


###下载第三方库源代码


处理过程的下一步是装载依赖库的源代码。每一个 *.podspec 中都包含一个变量，存放 git 远程资源的地址和 tag 标签。这些信息被SHA(Secure Hash Algorithm)算法处理后，存储在 ~/Library/Caches/CocoaPods 目录下。Core gem 负责创建该目录下的文件。

然后，通过使用 Podfile,.podspec,以及 caches 中的信息，将库源码文件下载到 Pods 目录下面。


###生成 Pods.xcodeproj 文件


每次运行 Pod install 命令时，一旦检测到 *.xcodeproj 文件内容有更改，Xcodeproj gem 将更新其内容。如果该文件不存在， Xcodeproj gem 将按照默认设置创建该文件。否则，将现有的设置信息加载到内存当中。


###安装第三方库


当 CocoaPods 添加一个库到工程中时，它除了添加库的源代码之外，还添加了很多其它东西。由于每个库的变动都会影响到它自己的 target ，所以对于每一个库，会被添加一些其它的文件。每一个库的源代码需要的附加文件：

* 一个 *.xcconfig 文件，包含工程构建阶段的设置。
* 一个私有的 *.xcconfig 文件，将上述文件中的设置与 CocoaPods 的默认设置合并。
* prefix.pch 文件，构建阶段所需要的。
* dummy.m 文件，同样在构建阶段需要。
    
每一个 Pod Target 中都完成这些工作之后，整个的 Pods Target 便被创建了。它同样添加了上述这些文件，同时还添加了一些其它的。如果哪个库源代码包含一个资源 bundle , 添加该 bundle 到 App’target 的说明信息将被添加到 Pods-Rosources.sh 文件中。同样，有一个 Pods-environment.h 文件，可以用其定义的宏来判断某一个组件是否来自某个 pod 。最后，两个声明文件被创建，一个 plist 文件，一个 markdown 文件，用来帮助使用者确认版权信息。


###写入到磁盘中


到目前为止，很多工作都是用内存中的对象来完成。为了使工作能够重复进行，我们需要一个文件保存所有的配置信息。因而，Pods.xcodeproj 文件，连同其它两个重要的文件  Podfile.lock 和 Manifest.lock，被保存到磁盘中。


####Podfile.lock

这是 CocoaPods 创建的最重要的文件之一。它追踪所有需要安装的 pods 的现有版本信息。如果你不确定某个已经安装了的 pod 的版本信息，那就查看该文件吧。如果将该文件添加到版本控制工具的资源文件中(*译者注：例如，不将该文件添加到.gitignore中*)，有利于团队协作开发的一致性，这也是大家推荐的做法。

####Manifest.lock

这是 Podfile.lock 文件的一个副本，它在你每次运行 pod install 命令时被创建。如果你从来没有看见过这样的错误 *sandbox is not in sync with the Podfile.lock*，那是因为这个文件不再和 Podfile.lock 完全一样了。由于 Pods 目录并不总是处于版本控制当中，这是一种确保开发者在运行程序前更新 pods 的方式，否则， App 将会崩溃，或者以另外一种不太明显的方式构建失败。

####xcproj

如果你按照我们推荐的方式，在你的系统中安装了 [xcproj](https://github.com/0xced/xcproj)，它会找到 Pods.xcodeproj 文件，并将其转换成旧的 ASCII 格式的 plist 。为什么呢？对这些文件的写操作将不再支持，而过一段时间后，Xcode 仍然依赖于它。如果不安装 xcproj，你的 Pods.xcodeproj 文件将会被保存为一个XML 格式的 plist 文件，当你打开 Xcode 的时候，它将会被重写，导致大量的文件 diffs 。


---------------------------


##完成


成功运行 pod install 命令之后，很多文件被创建，并添加到你的项目中。这个过程仅仅花费了几秒钟的时间。当然，当然，CocoaPods 所完成的工作，没有它，也是可以完成的。但是，这样的话，花费的就不仅仅是几秒钟的时间了。


---------------------------


##BTW: [持续集成](http://en.wikipedia.org/wiki/Continuous_integration)


CocoaPods 和持续集成并不冲突。对于不同的工程设置，让工程得以成功构建还是相对容易的。


###具有一个版本控制的 Pods 目录


如果你用版本控制工具来控制 Pods 目录及其中包含的所有文件。你不需要额外做一些特殊处理来完成持续集成。如果你需要指定一个特定的构建方案，选择正确的构建方案来构建 *.xcworkspace 就可以了。



###没有 Pods 目录


如果你没有使用版本控制工具来控制你的 Pods 目录，你需要做更多的事儿来促使持续集成能够正确的运行。至少，需要对 Podfile 文件进行版本控制。为了易于使用，建议对 .xcworkspace 和 Podfile.lock 文件同样进行版本控制。同样，建议确保被使用的 Pod 的版本是正确的。

一旦你完成了上述的工作，在持续集成环境下运行 CocoaPods 的关键是，确保 pod install 命令在构建(build)工程之前完成。在大多数集成系统中，比如 Jenkins 或者 Travis 中，你仅仅需要将其设置为一个构建步骤(实际上，Travis 将会自动的完成)。Xcode Bots 发布之后，写作此文之际，我们还没有想出一个流畅的处理过程。但是我们正在努力，一旦有所收获，将立即分享给你们。


---------------------------



##结束语


CocoaPods 简化了 Objective-C 的开发，我们的目标是快速发现和利用优质的第三方开源库，并积极参与开源项目的开发。深入理解隐藏在屏幕下面的技术仅仅帮你开发出更好的 apps，我们已经分析了整个执行流程，从加载 specs 和源代码开始，到创建 .xcodeproj 文件及其所有的组件，再到保存所有文件到磁盘中。所以了，下一次使用CocoaPods 时，运行 pod install -- verbose 命令，然后看看有哪些神奇的事情发生了。


###(本文完)











