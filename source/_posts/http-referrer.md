---
title: 防盗链？
toc: true
categories:
  - Front-end Study
tags:
  - Network
date: 2021-12-05 17:20:07
---

> 说是有用，但是为什么加上meta标签配置之后，就有能够访问显示了呢？

<!-- more -->

### 防盗链是什么？    

首先代入一下情景：

如果你刚刚开发完一个没有防盗链的带有文件下载功能的网站，挂上internet，然后上传几个时下非常热门的软件或电影并在网站内公布下载地址，让MSN上的所有好友都来体验一下你的杰作。不用多久就会发现网速出奇地变慢，甚至服务器托管中心的服务员会热情地打电话告诉你的网站流量很大，估计是网站受欢迎起来了，问你是不是该考虑加钱租用带宽更宽但价格更贵的网线了。

在这个值得庆祝的时候赶快打开Google Analytics看看有多少人来光顾你的网站了吧，如果发现访客每天才十来个人，很遗憾地告诉你：你的网站资源不幸地被人盗链了。而且更糟糕的是，当你把网站上的文件和电影通通删光之后，网站仍然没有变快多少，从web服务器的访问日志里会发现疯狂的访问请求正从四面八方涌过来，web服务器为了迎接这批访客而没有时间处理正常的页面，这种状况可能会一直持续好几个周时间。

或者另一种场景：

公司开发一个富文本功能，你使用了`wangEditor`这个插件来使用，但是后来发现里面的一个功能：插入网络图片，会出现一些bug，有的网络图片能够成功插入，而有的网络图片却没办法插入，并且报错**403**被拒绝访问了？然后网上发现只需要一行代码就能够让**403**的图片成功插入，这又是为啥？

```html
<meta name="referrer" content="no-referrer" />
<!-- 或者 -->
<img src="xxxx.jpg"  referrerPolicy="no-referrer" />
```

#### HTTP首部：referrer

HTTP Referer是header的一部分，当浏览器向web服务器发出请求的时候，一般会带上Referer,告诉服务器用户从那个页面连接过来的。

可以判断网站来源,相应的做一些校验,比如只允许某网站的请求,那么就可以通过获取referer,加以判断即可.

注意：是这个请求是哪个页面发出的,referer就是哪个页面

想要了解更多，可以参考：

[referrer策略和meta标签的问题](https://segmentfault.com/a/1190000017896469)

[HTTP首部---referrer 知识点](https://blog.csdn.net/java_zhangshuai/article/details/81627365)

### 如何避免和使用

#### 使用加了防盗链的资源

```html
<meta name="referrer" content="no-referrer" />
<!-- 或者 -->
<img src="xxxx.jpg"  referrerPolicy="no-referrer" />
```

在相应的html文件上加上以上代码，就可以获取到加了防盗链的资源，原因是：

本页面内通过发起的http请求都不会加上referrer这个属性，对方也就判断不了是不是来自其他网站的请求，就能够获取到资源了。

**但是！！！**



#### 避免被盗

基本的就是服务器上面对http请求头的referrer属性的值进行判断，如果不是本网站的url，则拒绝访问......

~~之后再补充内容~~

### 参考资料

[八种常见的防盗链方法总结及分析](https://www.cnblogs.com/davinc1/p/10985873.html)

[什么是防盗链 ](https://www.jianshu.com/p/0a1338db6cab)

