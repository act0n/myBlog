---
title: event-bug结合vue生命周期使用
toc: true
categories:
  - Front-end Study
tags:
  - Vue
date: 2023-03-19 16:24:12
---

`event-bus` 在vue中使用时，为了避免在vue组件未实际生效时，event-bus就进行事件的监听。因此需要结合vue的生命周期进行代理实现。

<!-- more -->

```js
// 定义一个event-bus
import mitt from 'mitt'

class CustomEventBus {
  constructor() {
    this.keys = {}
    this.emitter = mitt()
  }

  emit(key, ...args) {
    this.keys[key.toUpperCase()] = key
    this.emitter.emit(key, ...args)
  }

  on(key, callback) {
    this.emitter.on(key, callback)
  }
  // 结合生命周期
  onInVue(key, callback, vm) {
    const callbackProxy = (...args) => {
      if (vm._inactive) return
      callback(...args)
    }
    this.emitter.on(key, callbackProxy)
    vm.$once('hook:beforeDestroy', () => {
      this.emitter.off(key, callbackProxy)
    })
  }

  off(key, callback) {
    this.emitter.off(key, callback)
  }
}

export default new CustomEventBus()
```

