---
layout: post
title: "这是一篇测试用的文章标题啊！！"
date: 2014-03-06 17:23:22 +0800
comments: true
categories: test

---

## 第一篇blog

```	objc let me test code highlighting http://www.github.com/ source code

#import "Cocoa1AppDelegate.h"

@implementation Cocoa1AppDelegate

@synthesize window,siteUrl,pageContents;

- (void)applicationDidFinishLaunching:(NSNotification *)aNotification
{
    // Insert code here to initialize your application
    model = [[Cocoa1Model alloc] init];
}

- (IBAction)getSiteContents:(id)sender {
    [model setPageUrl:[siteUrl stringValue]];
    NSString* reply = [model getUrlAsString];
    NSLog(@"pageSrc: %@", reply);
    [pageContents setString:reply];
    [[[pageContents textStorage] mutableString] appendString:reply];
}

@end

```
    
``` ruby Discover if a number is prime http://www.noulakaz.net/weblog/2007/03/18/a-regular-expression-to-check-for-prime-numbers/ Source Article
class Fixnum
  def prime?
    ('1' * self) !~ /^1?$|^(11+?)\1+$/
  end
end
```

{% blockquote %}
Last night I lay in bed looking up at the stars in the sky and I thought to myself, where the heck is the ceiling.
{% endblockquote %}

{% blockquote @allanbranch https://twitter.com/allanbranch/status/90766146063712256 %}
Over the past 24 hours I've been reflecting on my life & I've realized only one thing. I need a medieval battle axe.
{% endblockquote %}

# 添加些东西


{% gist 4321346 %}