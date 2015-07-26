---
layout: post
title: "review.io.01"
date: 2014-04-29 14:49:13 +0800
comments: true
categories: review-io
---



### review.io 标签


社交网络的发展使得各种信息以井喷状流窜，关注最新的技术流向能够让自己的技术血液保鲜。从泛滥的信息中甄选出精华的部分吸收掉，不但能快速的提升自己，还能在潜意识中帮助自己做出正确的抉择。所以，沉下心来掌握一些有价值的、适合自己的最新技术才应该是一个务实者所应该做的，而不是以烦躁的心态仅仅浏览下最新的技术文章。除了需要从最便捷的渠道第一时间捕获到最新的信息，还要付诸以极强的执行力，将其真正变成自己的实践积累。


-----------


##[Empathy](http://nshipster.com/empathy/)<[中文](http://nshipster.com/empathy/)>


Empathy 的语意应该是，换个角度看待事物。这是 Mattt Thompson 写的一篇随笔，非技术类。


### 关于产品

好的软件产品应该恰如其分的瘙到用户的痒处；对产品追求极致，可以带给你敏锐的洞察力，以及坚韧不拔的意志力。

### 分享，自由，包容，耐心

人的本性是乐于和他人分享的，甚至不求回报。无论是在线上还是线下，与他人交流时，别太偏激，要有耐心，要学会变换角度。作为一名 programmer ，或者网络社区中的一员，更应该这样。如果有人用英文和你交流有困难，你应该用简洁、清晰的语句来和他对话。一名好的程序员应该时刻都懂得从不同的角度来看待问题。


-----------


##[NSTemporary​Directory / NSItem​Replacement​Directory / mktemp(3)](http://nshipster.com/nstemporarydirectory/)<[中文](http://nshipster.cn/nstemporarydirectory/)>


磁盘是用来存放需要长久保留的数据的，对于临时数据（文件），就不能这样了。临时文件实际上是将缓冲区里的数据写入磁盘中（某个角落里），然后通过某种方式来使用，再然后被删除掉。使用临时文件的过程可以归纳为：找到文件系统的一个合适位置来存放 --> 取一个唯一的名字 --> 不再使用时删除。


###寻找一个适合的系统目录

临时文件不应该存放在会被 Time Machine 备份，或者被 iCloud 同步的目录下。在 Unix 系统中，/tmp 目录确实是可以用来存放临时文件的，但是现在的 iOS 和 Mac OS X 应用中都采用了沙盒机制，其 NSTemporaryDirectory 专门设计为存放临时文件。

苹果推荐使用基于 NSURL 的 APIs 来进行文件操作，而放弃老的基于 NSString APIs 的方式。实际情况是，这种转换进行的并不顺畅。比如 NSFileManager 中的
URLForDirectory:inDomain:appropriateForURL:create:error: 方法：

``` objc 
- (NSURL *)URLForDirectory:(NSSearchPathDirectory)directory inDomain:(NSSearchPathDomainMask)domain appropriateForURL:(NSURL *)url create:(BOOL)shouldCreate error:(NSError **)error
{
    ...
}
```
将 NSItemReplacementDirectory 作为 directory 参数，NSUserDomainMask 作为 domain 参数，一个有效的父目录作为 url 参数，可以返回一个创建好的目录地址。但是 Mattt 觉得很难弄明白如何使用它。他认为，这个方法似乎是为了配合 replaceItemAtURL:withItemAtURL:backupItemName:options:resultingItemURL:error: 方法，来移动临时文件用的。

####干脆直接用下面的方式：

``` objc 
[NSURL fileURLWithPath:NSTemporaryDirectory() isDirectory:YES];
```

### 生成唯一的文件名/目录名

取的名字一定要具备唯一性。创建一个唯一标识符的方式是：


``` objc
//return a string in the format: 5BD255F4-CA55-4B82-A555-0F4BC5CA2AD6-479-0000018E14D059CC

NSString *identifier = [[NSProcessInfo processInfo] globallyUniqueString];

```
<div class="alert alert-info">
<p>
    <span class="glyphicon glyphicon-info-sign"></span>
   有人建议直接调用系统的 mktemp(3) 方法来防止命名冲突，但是上述方法不大可能会出现命名冲突。
</div>

使用如下方式也可以：

``` objc
//This produces a string in the format: 22361D15-E17B-4C48-AEA6-C73BBEA17011

[[NSUUID UUID] UUIDString]
```

####然后创建临时文件的路径：

``` objc
NSString *fileName = [NSString stringWithFormat:@"%@_%@", [[NSProcessInfo processInfo] globallyUniqueString], @"file.txt"];
NSURL *fileURL = [NSURL fileURLWithPath:[NSTemporaryDirectory() stringByAppendingPathComponent:fileName]];
```

####如何创建一个临时子目录：（便于一次性操作多个临时文件）

``` objc
NSURL *directoryURL = [NSURL fileURLWithPath:[NSTemporaryDirectory() stringByAppendingPathComponent:[[NSProcessInfo processInfo] globallyUniqueString]] isDirectory:YES];
[[NSFileManager defaultManager] createDirectoryAtURL:directoryURL withIntermediateDirectories:YES attributes:nil error:&error];
```

获取该临时子目录下的文件路径：

``` objc
NSURL *fileURL = [directoryURL URLByAppendingPathComponent:fileName];
```

###向临时文件中写入数据

####最直接的方式：

``` objc
NSData *data = ...;
NSError *error = nil;
[data writeToURL:fileURL options:NSDataWritingAtomic error:&error];
```

另一种方式：
``` objc
NSOutputStream *outputStream = [NSOutputStream outputStreamToFileAtPath:[fileURL absoluteString] append:NO];
```

###清除临时文件

在 Mattt 写该文时，并没有相关的资料明确指出，系统过多长的时间会自动的清理系统临时目录下的文件。但是自己来维护绝对是好习惯。
下面的方式，对于删除临时目录和临时文件，都是可行的

``` objc
NSError *error = nil;
[[NSFileManager defaultManager] removeItemAtURL:fileURL error:&error];
```
###感悟

在一个应用程序的整个生命周期中，其实一切东西都是临时的，一些东西只是比另一些东西存在的时间更长一些。

{% blockquote %}
一切都会过去的（This too shall pass）
{% endblockquote%}

我们短暂而辉煌的生命也是这个道理啊。


-----------


##电子书网站

临时收集三个电子书收集站点：

* [www.it-ebooks.info](http://www.it-ebooks.info)
* [www.salttiger.com](http://www.salttiger.com)
* [www.ppurl.com](http://www.ppurl.com)

之后，又发现 [合法地免费下载电子书的站点有哪些推荐?](http://www.zhihu.com/question/19734795?rf=19620112) 罗列的更加全面！！


-----------


##Octopress 中点击链接之后，跳转到新打开的页面。

可以在 Octopress根目录/source/_includes/custom/head.html 文件中加入如下代码

``` html
<script language = "JavaScript">
function addBlankTargetForLinks () {
  $('a[href^="http"]').each(function(){
		$(this).attr('target', '_blank');
	});
}
 
$(document).bind('DOMNodeInserted', function(event) {
	addBlankTargetForLinks();
});
</script>
```
也可直接使用 HTML 的标签

``` html
    <a href="http://newPage.html" target="_blank">new page</a>
```

#### 经过测试，在本地 rake preview 模式下，以及 deploy 之后，上述两种跳转方式都无效。


-----------


##[NSCache(|kæʃ|)](http://nshipster.com/nscache/)[中文](http://nshipster.cn/nscache/)


NSCache 被很多开发者忽略了。它和 NSMutableDictionary 一样，都具有垃圾回收功能。唯一不同于 NSMutableDictionary 的是，NSCache 中保存的 keys 并不是复制的，这反而成了它的一个优势。
NSCache 可以说是一个烫手的山芋。为什么呢？
其 setObject:forKey:cost: 方法和 NSMutableDictionary 的 setObject:forKey: 很相似，只是多了一个 cost 参数。

>官方文档中说，const 值用来衡量 Cache 中的所有对象所要消耗的资源。
>当内存不足，或者该值超出最大允许范围之后，Cache 将开始清理一些无用的元素。

<div class="alert alert-info">
<p>
    <span class="glyphicon glyphicon-info-sign"></span>
   但是，清理的过程并不能确保按照确定的顺序进行。所以，当你想通过控制 cost 参数来进行某些特殊操作时，结果可能不是你想要的。
   最直观的 const 值应该是所占的字节数，如果不能很容易的读取到的话，也就没必要花太多功夫去计算了。这时可以传 0 ，或者选用 setObject:forKey: 方法。
   使用 evictsObjectsWithDiscardedContent 和 NSDiscardableContent 来控制自动回收，也许同样会出现问题。
</div>

总之，Mattt 认为，如果你不认识苹果内部写这些方法的人的话，最好别用它们了。

####尽管如此，大家应该更多的使用 NSCache 来实现缓存处理，但是要用下面的这些不烫手的方法：

* objectForKey:
* setObject:forKey
* removeObjectForKey:


