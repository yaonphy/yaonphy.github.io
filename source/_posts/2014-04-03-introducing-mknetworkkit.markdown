---
layout: post
title: "iOS Framework: Introducing MKNetworkKit（简体中文翻译）"
date: 2014-04-03 17:51:49 +0800
comments: true
categories: MKNetworkKit
---


#### 文章的原版（英文原文）[链接](http://blog.mugunthkumar.com/products/ios-framework-introducing-mknetworkkit/)，看到教程已经被翻译为[塞尔维亚 – 克罗地亚语](http://science.webhostinggeeks.com/predstavljanje-mknetworkkit) by [Jovana Milutinovich](http://webhostinggeeks.com/)，[日语](http://odoruinu.net/blog/mknetworkkit/) by [@noradaiko](http://odoruinu.net/)，[德语](http://blog.mugunthkumar.com/products/ios-framework-einfuhrung-in-mknetworkkit-german-translation-of-mknetworkkit-documentation/) by [Jonas Pencke](https://twitter.com/jonaspencke)。故而有了翻译为中文的念头。


如果有一个网络库能够自动的帮你搞定缓存处理，那该是多么爽的一件事。当你的 App 的网络变为离线状态时，要是有一个网络库能够自动的保留未完成的网络操作，那简直爽歪歪的赶脚啊。

你在离线状态下，将某个Tweet 标记为喜欢，或者将某个推荐内容标记为已读。你不用额外多敲任何代码，网络库会在网络处于连接状态时自动完成所有的离线任务。让我们一起走进 MKNetworkKit。


--------------------------



## MKNetwork 渊源


MKNetwork 使用 Objective-C 编写，是一款与 apple 官方网络接口无缝对接，基于 block，支持 ARC，并且非常简单易用的网络库。编写MKNetwork 的灵感来源于 ASIHTTPRequest 和 AFNetworking 这两款非常流行的OC网络库。

在吸收这两款网络库的优秀功能的基础之上，MKNetwork 增加了一些新的功能。除此之外，相比于其他网络库，MKNetwork 能够让你写出更多清晰、简洁的代码。使用 MKNetwork，你就很难写出非常恶心的网络接口代码了。


--------------------------



##功能


###超轻量级

这个kit由两个主要的类，以及一些 category(附加)方法组成。这意味着，使用 MKNetwork 是超简单的一件事。

为整个 App 提供单一共享的请求队列

基于大量网络请求的 apps 应当优化并发的网络请求操作。非常遗憾的是，目前没有一个网络库能够彻底的实现。如果你的应用程序没有优化/控制并发网络请求操作数，将会出现哪些问题呢？我下面一一举例说明。

假设你正在往服务器传送一批图片（color or batch）。对于某一个IP地址，大多数的移动网络(3G)不允许超过两个的并发 HTTP 请求。也就是说，在3G 网络环境下，你的移动设备不能开启多于两个的并发网络请求。在 [EDGE](http://zh.wikipedia.org/wiki/GSM%E5%A2%9E%E5%BC%BA%E6%95%B0%E6%8D%AE%E7%8E%87%E6%BC%94%E8%BF%9B)（2.75G）网络环境下，情况更糟糕，很多情况下，你都不能建立多于一个的网络连接。在 Wifi 环境下，情况会好一些，并发网络请求的上限个数是6。由于移动设备会在不同的网络环境中使用，所以应当根据网络环境对网络请求数做出相应的限制。通常，移动设备更多的使用 3G 网络，也就是说，同时最多上传两张图片。然而，问题不是缓慢的上传速度。当你在一个 View（其他的View）中下载图片的缩略图，同时，上传图片的操作在后台运行。这样，真正的问题便出现了。如果你的App没能正确的控制网络连接队列中连接数的大小，缩略图的加载操作将会超时。这显然不是你想要的结果吧。遇到上述的情况，应该这样处理：提高下载缩略图的网络请求的优先级，或者等到上传图片的操作完成之后再加载缩略图。这就要求在整个 App 中始终保持单一的网络请求列。MKNetwork 自动的为其每一个实例对象提供单一共享的请求队列。尽管 MKNetwork 本身不是一个单例，共享的 queue 是。


###及时准确的显示网络活动指示图标


有很多第三方库的类追踪网络请求个数，通过网络请求个数的加加减减来显示/隐藏网络活动指示图标。回归到单一共享队列的原则，通过监听（KVO）operationCount 属性，MKNetwork自动地显示/隐藏网络活动指示图标。作为一个 developer，你不再需要手动的控制网络活动指示图标，永远不需要。

```objc 
if (object == _sharedNetworkQueue &amp;&amp; [keyPath isEqualToString:@"operationCount"]) {
 
        [UIApplication sharedApplication].networkActivityIndicatorVisible =
        ([_sharedNetworkQueue.operations count] &gt; 0);
    }
```

###自动调整的队列大小


继续前面的讨论，我说大多数的网络不允许多于两个的并发网络连接。所以在 3G 网络环境下，你的 queue 的大小应当设置为2。MKNetwork 自动的为你处理这些问题。当网络环境为 3G/EDGE/GPRS,它自动的将当前并发网络连接数调整为2.当切换到wifi网络时，又自动的调整为6.有了这种技术，在 3G 网络下，从远程服务器的图片库中下载缩略图（或者大量相似的请求）将会得到很大的性能提升。


###自动缓存


MKNetwork能够自动的缓存所有的 "GET" 请求。当你再次发送相同的请求时，MKNetworkKit 会立即调用请求完成的回调方法，并传送缓存的数据。它也会再次向服务器发送请求。当从远程服务器获取到新的数据，请求完成的回调方法会被再次调用，而这时传送的是最新的数据。因此，你不需要手动的处理缓存。只需要写一行代码，调用一个方法即可，仅此而已。

```objc 
[[MKNetworkEngine sharedEngine] useCache];
```

或者，你可以复写 MKNetworkEngine 子类中的方法，自定义缓存的路径和内存中缓存的消耗。


###操作冻结


使用 MKNetworkKit，你可以冻结网络操作。如果你冻结了一个网络请求操作，当网络连接中断时，它将被自动的加入冻结操作的线性队列中。一旦网络恢复，网络请求将会继续进行。比如微博中的草稿箱功能，就是一个很好的例子。

当你发送了一个微博，并将其网络请求标记为冻结模式，这样 MKNetworkKit 将会自动的处理冻结操作，并为你保存这个请求。这样的微博将会在稍后自动发布，而你不用写一行额外的代码的。这种方式也可以用到其它网络请求中，比如喜欢某个微博，在 Google reader 客户端分享一篇文章，向 Instapaper 中添加一条链接。

对于相似的网络请求，只执行一次。当你加载缩略图（for a twitter stream）时，你最终可能会为每一个缩略图都创建一个网络请求。事实上，你所需要的的请求数和独立的（unique URLs） URL个数是一致的。使用 MKNetworkKit ，在 queue 中的每个 GET 请求，实际上只被执行了一次。MKNetworkKit 还不能做到对“POST”方式的请求进行智能缓存处理。


###图片缓存


MKNetworkKit 可以完美的实现对图片的缓存处理。通过复写一些方法，你可以设置需要缓存的图片的数量，内存内缓存的负载，以及缓存的路径等。对于这些方法的复写完全是可选的。


###性能


两个字，速度。MKNetworkKit 进行了无缝的缓存处理。它的处理方式就像NSCache一样，除此之外，当出现内存警告时，内存中的缓存会被转移到磁盘的缓存目录中。


###完全支持ARC


你通常会为新的项目选择新的网络库。MKNetworkKit 并不是想让你替换掉正在使用的网络库（尽管可以，但是那是一件非常恶心的事儿）。在新的项目中，你总会想着去启用ARC，在写本篇文章的时候，MKNetworkKit 也许是唯一完全支持ARC的网络库。基于 ARC 的内存管理要比 non-ARC（RRC）的内存管理快一个数量级。


--------------------------


##如何使用


好了，不再自我陶醉了，让我们来看看如何使用这个网络库。


###添加 MKNetworkKit 到你的项目中


1. 将 MKNetworkKit 文件拖拽到你的项目中。
2. 添加 CFNetwork.Framework, SystemConfiguration.framework,Security.framework，以及ImageIO.Framework。
3. 将 MKNetworkKit.h 添加到 PCH 文件中
4. 如果你开发的是iOS项目，删除 NSAlert+MKNetworkKitAdditions.h。
5. 如果你开发的是Mac项目，删除 UIAlertView+MKNetworkKitAdditions.h。

仅仅需要5个核心文件，一个功能强大的网络库就配置好了。


###MKNetworkKit 中的类


1. MKNetworkOperation
2. MKNetworkEngine
3. 其它辅助类(苹果官方用于判断网络是否可连接的类)以及附加类（categories）

我推崇简洁易用的原则。苹果已经为开发者完成了繁重的网络连接工作。一个三方库应该做的是，提供一个基于网络的任务队列，并实现可选的缓存功能。我认为所有的三方库都应当不超过10个类文件（无论这个三方库是网络库，或者UIKit的封装库，或者其它任何类型的库。反之，超过10个类文件的库都是臃肿的。 Three 20 是一个臃肿库的例子，ShareKit 也是。也许它们是功能丰富的，但是太大，太臃肿。不像RESTKit那样，ASIHttpRequest 和 AFNetworking 是精悍轻巧的。JSONKit 比 TouchJSON（或者 TouchCode 库中的其它分支类库）轻量级。这也许只是我个人的观点，但是，如果在我的项目中，超过三分之一的代码都来自三方库，那我是无法接受的。

使用臃肿的三方库带来的问题是，我们需要理解它的内部机制，进而进行自定义，使其与自己的需求融合（如果有这个需要的话）。我之前写的一个三方库（ MKStoreKit：帮你添加应用内置购买功能）超轻量级，并且非常容易上手。我相信 MKNetworkKit 也会保持这样的优点。要使用MKNetworkKit，你仅仅需要理解 MKNetworkOperation 和 MKNetworkEngine 这两个类提供的一些共用方法。MKNetworkOperation 类和 ASIHttpRequest 类相似。它是 NSOperation 类的子类，并且进一步封装了所用到的request类和response类。你为你的App中的每一个网络操作创建了一个 MKNetworkOperation 类。

MKNetworkEngine 是一个管理网络队列的"假"单例类。由于是假的单例，对于一些简单的请求，你可以直接使用MKNetworkEngine中的方法。在需要更多自定义功能的情况下，应该创建一个它的子类。每一个MKNetworkEngine的子类都有一个属于它自己的用于监测网络连接性的对象，此对象会在网络状态发生变化的时候给它发送通知。你应当为你所使用的每一个独立REST服务器创建一个 MKNetworkEngine 类的子类。由于是伪单例，所以在它的子类中的每一个单独请求都共享一个全局的单一队列。

你可以在 application delegate 中保留一次 MKNetworkEngine 的实例对象，就像对 CoreData 中 managedObjectContext 类一样。在你使用 MKNetworkKit 的时候，你创建了一个 MKnetworkEngine 类的子类去有序的管理网络请求。比如，所有向Yahoo的请求方法定义在一个单独的类中，所有向 Facebook 的请求放在另一个单独的类中。我将展示使用该框架的三个不同的案例。


###示例1：


我们现在创建一个从雅虎财经栏目拉取数据的“YahooEngine”类

####步骤1：创建一个 YahooEngine 类，并继承 MKNetworkEngine。

MKNetworkEngine 的初始化方法需要主机名和自定义的协议头部内容（可选）。这些自定义的协议头是可选的，并且可以为空。如果你使用的自己的 REST 服务器（不同于该示例），你可能需要传递 app 的版本，或者其它需要的信息，比如客户端 ID。

```objc 
NSMutableDictionary *headerFields = [NSMutableDictionary dictionary];
[headerFields setValue:@"iOS" forKey:@"x-client-identifier"];
 
self.engine = [[YahooEngine alloc] initWithHostName:@"download.finance.yahoo.com"
                   customHeaderFields:headerFields];
```

雅虎有可能没有要求你在协议头中包含 x-client-identifier 字段的值，这个例子只是为了展示这样的功能。

由于使用的是 ARC ，所以是否需要设置对引擎实例对象的强引用关系（strong reference）完全取决于你自己。当你所访问的服务器宕机了，或者由于其它无法预知的因素，你的主机无法访问了，你当前的请求将被自动的添加到冻结请求队列中。对于冻结网络请求的更多信息，请阅读本篇文章中冻结网络操作的部分。

####步骤2：设计引擎类（Separation of concerns）

让我们现在开始在yahoo Engine 中写用来获取汇率的方法。这些引擎方法将会在你的 view Controller 中调用。一个好的实践经验是，你的引擎类最好不要向调用类暴露URL或者 HTTPHeaders。你的 view 应当不“知道”所需的 URL 端口或者其它参数。也就是说，引擎中的方法的参数应该是货币数量以及货币种类，方法的返回值应该是浮点型的汇率或者获取数据时的时间戳。由于操作是被异步执行的，所以你应该在 blocks 中返回这些值。下面举一个例子：

```objc 
-(MKNetworkOperation*) currencyRateFor:(NSString*) sourceCurrency inCurrency:(NSString*) targetCurrency onCompletion:(CurrencyResponseBlock) completion onError:(ErrorBlock) error;
```

父类—— MKNetworkEngine 中定义了如下的三种类型的方法：

```objc 
typedef void (^ProgressBlock)(double progress);
typedef void (^ResponseBlock)(MKNetworkOperation* operation);
typedef void (^ErrorBlock)(NSError* error);
```

在我们定义的 YahooEngine 中，我们使用一种新的类型的 block，返回汇率值的 CurrencyResponseBlock。定义如下：

```objc 
typedef void (^CurrencyResponseBlock)(double rate);
```

在一般的App中，你应当使用类似于定义 CurrencyResponseBlock 的方式定义你自己的 block，从而可以在 controller 中获取到需要的数据。

####步骤3：处理数据

数据处理，说白了就是转换你从服务器获取到的数据，这些数据有可能是 JSON 格式，XML 格式，或者是二进制的 plists。你应当在你的引擎类中进行数据格式的转换。同样，不要让 controller 去完成数据解析的任务。你的引擎类应当返回相应的模块化的数据对象，或者包含模块化对象的数组（数据列表）。在引擎类中将 JSON/XML 格式的数据转换成模块化的数据，考虑到需要降低耦合度，你的 view controller 不应该知道获取 JSON 数据中某个值（value）的键（key）。

上面介绍了如何设计你的自定义引擎类。大多数的网络引擎不要求你降低 controller 和数据处理之前的耦合度，减轻 controller 的负载，但是我要求这样，因为我们在乎你

####步骤4：方法的实现

我们现在讨论计算汇率的方法的实现细节。

从 yahoo 网站获取货币信息，只需要简单的发送一个GET请求即可。

我写了一个宏定义，通过给定的一对货币单位值来格式化URL。

```objc 
#define YAHOO_URL(__C1__, __C2__) [NSString stringWithFormat:@"d/quotes.csv?e=.csv&amp;f=sl1d1t1&amp;s=%@%@=X", __C1__, __C2__]
```

你的引擎类中定义的方法，应当有序的完成下面的任务：

1. 根据传递的参数，格式化好所需的 URL
2. 创建一个 MKNetworkOperation 的实例对象用于发送请求
3. 设置 HTTP 请求类型
4. 为创建好的 Operation 添加请求成功和请求失败的回调 block（请求成功的回调 block 中，通常是处理 response 并将其数据进行模块化）
5. 可选，为 operation 添加进度处理（或者在 view controller 中进行请求进度处理）
6. 可选，如果是下载文件的请求，需要为 operation 设置一个下载流（通常是一个文件）
7. 当请求完成的时候，处理请求结果，并将数据返回给调用方法（通常是 controller 中的方法）

上代码：

```objc 
MKNetworkOperation *op = [self operationWithPath:YAHOO_URL(sourceCurrency, targetCurrency) params:nil httpMethod:@"GET"];
[op onCompletion:^(MKNetworkOperation *completedOperation)
{
     DLog(@"%@", [completedOperation responseString]);
     // do your processing here
    completionBlock(5.0f);

}onError:^(NSError* error) {

    errorBlock(error);
}];

[self enqueueOperation:op];

return op;
```

上面的实例代码中，格式化了 URL，并且创建了一个 MKNetworkOperation。在分别添加了请求成功和失败的回调 block 之后，它又调用了父类的 equequeOperation 方法，并返回创建好的 MKNetworkOperation 对象的指针。你的 viewcontroller 应当保留保留对该指针的引用关系，当该 viewcontroller 从整个 viewcontroller 体系中移除时，应当取消此次请求操作。因此，如果你在 viewDidAppear 方法中调用了引擎中的方法，应该在 viewWillDisappear 中取消此次请求操作。取消请求操作将会让请求队列去执行其它的网络请求操作（不知道你还记不记得，在移动网络中，仅仅允许两个并发的网络请求，当某个网络请求不再需要时，将其释放掉，能够提升你的 App 的性能和速度）

你的 viewcontroller 也可以（可选）添加请求进度处理，并更新界面。下面的代码实现了进度条的时时更新

```objc 
[self.uploadOperation onUploadProgressChanged:^(double progress) {
 
       DLog(@"%.2f", progress*100.0);
       self.uploadProgessBar.progress = progress;
   }];
```

MKNetworkEngine 也提供了通过一个 URL 来创建 MKNetworkOperation 对象的便捷方法，本部分前面用于创建 MKNetworkOperation 对象的代码也可以用下面的代码替换：

```objc 
MKNetworkOperation *op = [self operationWithPath:YAHOO_URL(sourceCurrency, targetCurrency)];
```

请求URL会被自动的添加相应的前缀域名，这个域名是你在创建引擎类的时候提供的。

创建一个 POST 类型，DELETE 类型，或者 PUT 类型的请求，仅仅需要改变 httpMethod 的参数值即可。MKNetworkEngine 提供了许多这样的便捷方法，你可以在头文件中找到的。

###示例2：


向服务器传送一张图片（比如发布一条 twit 时附加的图片）

现在让我们来看一个如何向服务器传图片的例子。显然，上传图片的请求必须被编码为 multi-part form 的数据格式。MKNetworkKit 使用和ASIHttpRequest 相似的模型。通过调用 MKNetworkOperation 中的 addFile：forKey：方法，可以添加一个 multi-part form 格式的数据到你的请求中。就是这样简单。

MKNetworkOperation 同时还提供了一个简单的方法，通过 NSData 类型的指针来添加图片。也就是说，你可以调用 addData：forKey：方法来上传图片到远程服务器上。（可以想想从相册里直接上传图片的情形）


###示例3：


下载图片到本地的目录中（缓存）

使用 MKNetworkKit，从远程服务器下载文件，并保存到本地的操作变的异常容易。

设置 MKNetworkOperation 的 outputStream 即可：

```objc 
[operation setDownloadStream:[NSOutputStream
    outputStreamToFileAtPath:@"/Users/mugunth/Desktop/DownloadedFile.pdf"
              append:YES]];
```

你可以设置多个 outputStream 到一个单独的 operation 中，这样可以将相同的文件保存到不同的位置（保存到缓存目录的同时还保存到工作目录中）


###示例4：


图片缩略图的缓存

为了下载图片，你需要提供一个绝对的 URL 地址，而不是一个路径。MKNetworkEngine 中有一个方便的方法来实现。使用 operationWithURLString:params:httpMethod: 就可以创建一个基于绝地 URL 的 operation。

MKNetworkEngine 是智能的。它将多个基于同一 URL 的 GET 请求合成为一个，在这个请求完成时，通知（回调）所有的 blocks。
这大幅度的提升了下载缩略图的速度。

创建继承于 MKNetworkEngine 的子类，复写相应的方法可以设置缓存的目录和缓存的消耗控制参数。如果你不想自定义这些，你可以直接使用 MKNetworkEngine 方法来下载图片。我建议你自己配置相关的缓存设置。


###缓存操作


MKNetworkKit 默认会缓存所有的操作。你所要做的是，为你的引擎打开缓存操作。当一个GET类型的请求被执行时，如果对应的response之前被缓存过，缓存的 response 马上会被返回。使用 isCachedResponse 方法可以判断一个 response 是不是缓存过的。下面的代码作了进一步的说明：

```objc 
[op onCompletion:^(MKNetworkOperation *completedOperation)
 {
     if([completedOperation isCachedResponse]) {
         DLog(@"Data from cache");
     }
     else {
         DLog(@"Data from server");
     }
 
     DLog(@"%@", [completedOperation responseString]);
 }onError:^(NSError* error) {
 
     errorBlock(error);
 }];
```


###冻结操作


可以说，MKNetworkKit 中内置的自动冻结操作的功能是最让人来劲的。只需设置一下就可以使用该功能了，基本上零花费啊！

```objc 
[op setFreezable:YES];
```

网络不可用时，操作被冻结，并被序列化到一个队列中。当网络恢复到可用状态时，冻结的操作继续执行。在 tweeter 中，离线的时候喜欢某个 tweet，等会上线之后，网络操作被自动执行，就是一个很好的例子。

当App进入后台时，冻结的操作会被保存到磁盘中，当app从后台中被唤起之后，冻结的操作也会被自动的执行。

MKNetworkOperation 中方便的方法：

MKNetworkOperation 中提供了如下一些用来格式化网络请求返回数据的便捷方法：

1. responseData
2. responseString
3. responseJSON (Only on iOS 5)
4. responseImage
5. responseXML
6. error

当网络请求成功后处理 response 时，使用它们会显得非常便捷。如果数据格式不对，这些方法将会返回 nil。比如，使用 responseImage 来返回图片，但是 response 中的数据却是 HTML 格式，这样便会得到一个 nil。唯一一个不会出现错误的方法是 responseData，使用其它的方法时你必须确保自己知道 response中数据的类型。


---------------------------



##方便的宏定义


DLog 和 ALog 这两个宏定义是我公然从 Stackoverflow 里偷来的，我也忘了是从哪里偷来的了。如果这是你写的话，那就麻烦告诉我一声。



---------------------------



##关于GCD的一点说明


我故意没有使用 GCD，因为网络操作需要随时停止和优化。尽管 GCD 要比 NSOperation 更加高效，但是它无法做这些工作。我不建议你的网络操作使用基于 GCD 的队列。



---------------------------



##文档


所有的头文件中都添加了注释，我尝试使用 apple 的文档风格来加注释。顺便说一句，代码你可以随便用哦。



---------------------------



##源代码


代码在 github 上，并且包含一个 demo

[MKNetworkKit on Github](https://github.com/mugunthkumar/mknetworkkit)



---------------------------



##功能建议


功能建议不要以邮件的方式发给我，最好的方式是[在github上添加issue](https://github.com/mugunthkumar/mknetworkkit/issues)



---------------------------



##许可

遵循 MIT License



---------------------------



##——Mugunth


----------------------------



####*本文最初写在我的[老博客里面](http://yaongphy.sinaapp.com/?p=531)，之前格式太乱。*
