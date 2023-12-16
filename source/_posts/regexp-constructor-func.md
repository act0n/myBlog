---
title: 正则表达式构造函数的使用
toc: true
categories:
  - Front-end-Study
tags:
  - JavaScript
  - RegExp
date: 2023-03-12 22:02:32
---

通常情况下，我们构建正则表达式的方式都是：由斜杠 (/) 直接包围的表达式。但是，有些特殊情况，我们无法通过这种方式构建出来：比如**需要根据用户所选择的字段，来进行构建一个正则表达式**。这个时候我们无法在程序代码中定死一个表达式，而应该根据参数来构建正则表达式，此时就需要用到正则表达式的构造函数了。

<!-- more -->

```js
new RegExp(pattern[, flags])
```

而我们平常使用的那种形式，叫做`字面量`：由斜杠 (/) 包围而不是引号包围。

```js
const regexp = /abc/i; // 字面量形式
```

`构造函数的字符串参数`：由引号而不是斜杠包围。

```js
const regexp = new RegExp('abc', 'i'); // 首个参数为字符串模式的构造函数
```

另外，构造函数也有`字面量参数`

```js
const regexp = new RegExp(/abc/, 'i'); // 首个参数为常规字面量的构造函数
```

我们应对开头所说的*根据参数来构建正则表达式*，就要用到`字符串参数的构造函数`:

```js
// 需要根据用户所输入的字符串，进行校验：以用户输入的字符串为开头的字符串
const targetStr = 'xxxxxxxxx'
const userInputStr = 'abc'
const regexp = new RegExp(`^${userInputStr}`)
regexp.test(targetStr)
```

思考：正则表达式的字面量形式和字符串形式，在构建的时候还有什么差异？

答：由于是字符串，所以对于一些特定字符用转义符（\）进行转义