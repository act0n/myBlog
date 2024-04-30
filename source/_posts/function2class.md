---
title: function 转 class
toc: true
categories:
  - Front-end Study
tags:
  - JavaScript
date: 2024-01-05 21:39:54
---


如何将下面的代码转换为**普通构造函数**的写法？
```js
class Animal {
  constructor(name) {
    this.name = name
  }
  run() {
    console.log(this.name + ' is running!')
  }
}
```

<!-- more -->

有 4 点需要注意：

- 严格模式
- 构造函数只能通过 new 关键字创建，不能够直接调用
- 属性方法不能够被枚举
- 属性方法不能够通过 new 关键字创建

```js
'use strict' // 1. 严格模式

function Animal(name) {
  // 2. 只能通过 new 关键字创建，不能够直接调用
  if (!new.target) {
    throw new TypeError(`Class constructor ${this.constructor.name} cannot be invoked without 'new'`)
  }
  this.name = name
}

Object.defineProperty(Animal.prototype, 'run', {
  value: function() {
  	// 4. 不能通过 new 关键字创建
    if (new.target) {
      throw new TypeError(`Animal.prototype.run cannot be invoked with 'new'`)
    }
    console.log(this.name + ' is running!')
  },
  // 3. 当前属性方法不能够枚举
  enumerable: false
})
```



