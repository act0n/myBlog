---
title: js实现sleep函数
toc: true
categories:
  - Front-end Study
tags:
  - JavaScript
  - Promise
  - Generator
date: 2023-03-19 16:22:06
---

⚠️js中是没有sleep函数，那应该怎么实现呢？

经过尝试，有3种方法实现：`setTimeout`、`Promise`、`Generator`

<!-- more -->

```js
const sleep = (seconds) => {
	return new Promise((resolve) => {
		setTimeout(() => resolve(), seconds)
	})
}

const func = async () => {
	console.log("1")
	await sleep(1000)
	console.log("2")
}

func()
```

> 效果展示

![](result.jpeg)