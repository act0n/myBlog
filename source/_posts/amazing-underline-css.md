---
title: 一个很棒的下划线css 效果
toc: true
categories:
  - Front-end Study
tags:
  - CSS
date: 2024-01-01 19:36:25
---

最近刷视频刷到一个分享，发现是一个很棒的 css 效果：通过使用`background`相关属性来实现一段文本的多彩下划线样式，并且在鼠标悬浮时进行显示。

先思考一下自己可以有哪些手段来实现，然后再来了解一下这里的实现手段吧：

<!-- more -->
如图所示：
![](example.gif)

1. 定义一个容器

```html
<div>
  <h2 class="title">
  	<span>Migrating From MySQL to YygabyteDB Using YugabyteDB Voyager</span>
  </h2>
</div>
```

2. 增加一些样式，核心内容就是对 `background`属性的熟练使用，定义一个渐变背景，初始化时背景宽度为` 0`，背景的位置放到 `rigth bottom`，并且设置一个过度效果。当鼠标悬浮在文本中时，将背景宽度设置为 `100%`，并且将背景的位置设置为`left bottom`。

```css
.title {
  color: #333;
  line-height: 2;
}
.titile span {
  background: linear-gradient(to right, #ec695c, #61c454) no-repeat right bottom;
  background-size: 0 2px;
  transition: background-size 1300ms;
}
.title span:hover {
  background-position-x: left;
  background-size: 100% 2px;
}
```

来看一下展示效果：

![](result.gif)