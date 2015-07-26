---
layout: post
title: "在新的环境（电脑）配置已有的 Octopress 博客"
date: 2014-04-22 16:10:41 +0800
comments: true
categories: Octopress
---

于2014.04.14在苹果官方商城下单，预定了第一台属于自己的 Mac 电脑。内心自然是非常喜欢 MBA 的，也算是实现了近期的一个愿望。之前用过非Retina的15英寸 MBP，毕竟是配备的，不是自己的，感觉是完全不一样的（是不是有点偏执^_^）。离开的时候还得留下设备，细心点的话，还需要清空下浏览器的cookie啦，保留的密码啦，浏览历史记录啦等等个人信息。如果是个好人，还会清理掉其它所有自己认为不应该留给第二个机主看到的东西。拿到新的配备机后，重新配置环境，设置自己的喜好风格，太麻烦！


再三纠结之后，选择了 i7 8G内存 128G闪存的配置，官网价格约9500人民币，使用教育优惠渠道（随机抽查（偷笑））购买约9000人民币。历经4天后，于4.18日下午到手，组装地在上海南汇，EMS的快递一般吧。


-----------------------------------


##安装 Homebrew（可选）

由于后续操作依赖于Homebrew，所以必须要装的。可以按照 Homebrew [官网的说明](http://brew.sh/)来安装。需要注意的是，最好将网络DNS设置为Google的8.8.8.8 DNS 服务，否则拉不下来数据，我是新机子，还没来得及设置 DNS ，试了好几次，下载都中断了。。。

``` sh
ruby -e "$(curl -fsSL https://raw.github.com/Homebrew/homebrew/go/install)"

```


-----------------------------------


##安装 Ruby 的相关工具（可选）

如果你的机器中已经安装好了下面的工具，请忽略。

###安装RVM

RVM可以方便的管理 Ruby 的各个版本，在 Terminal 输入下面的命令即可：

``` sh
curl -L https://get.rvm.io | bash -s stable --ruby

```

###安装 bundle

``` sh
gem install bundler
bundle install
```


-----------------------------------


##确保 remote 端的 repository 中 source 分支是最新的

在老机器（老环境中），如果一直使用 rake deploy 命令来 push 生成的 blog 资源到远程 repository 中，更新的只是 master 分支。看一下本地 source 分支（即 Octopress 根目录）中的内容是否和远程 source 分支中的内容同步，若不是，在老机器中，用下面的命令来 push 下：

``` sh
#切换到 Octopress 目录下
$ rake generate
$ git add --a
$ git commit -a "Some comment here."
$ git push origin source  # update the remote source branch
```


-----------------------------------


##在新环境中拉取 source 分支资源

在你想要的地方，将远程 source 分支拉取到 Octopress 目录，将 repository 地址更换为自己的而已。

``` sh
$ git clone -b source https://github.com/username/username.github.io.git Octopress

```


-----------------------------------




##在新环境中拉取 master 分支资源

紧接着上一步，进入 Octopress 目录（随意），将远程 repository 中的 master 分支拉取到 _deploy 目录中，将 repository 地址更换为自己的而已

``` sh
$ cd Octopress
$ git clone -b master https://github.com/username/username.github.io.git _deploy
```


-----------------------------------




##完成

好了，现在可以在新环境下，使用 rake generate 和 rake preview 命令来生成和本地查看 blog 了。如果有在两个环境下同时更新博客的需求，就要像多人协作开发那样，提交更新前，需要先拉取远程资源，合并后再 push，以保持一致。


-----------------------------------




##参考

* [Clone Your Octopress to Blog From Two Places](http://blog.zerosharp.com/clone-your-octopress-to-blog-from-two-places/)
* [Octopress](http://octopress.org/docs/)
* [Homebrew](http://brew.sh/)




