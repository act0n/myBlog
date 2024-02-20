---
title: 并发任务控制
toc: true
categories:
  - Front-end Study
tags:
  - JS 编程思路
date: 2024-02-20 22:44:44
---
遇到的问题如下，需要构建一个 `SuperTask` 这么一个类，来实现并发任务控制。

```js
class SuperTask {
  // todo
}

function sleep(time) {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve()
    }, time)
  })
}

const superTask = new SuperTask()
function addTask(time, name) {
  superTask
    .add(() => sleep(time))
    .then(() => {
      console.log(`任务${name}完成`)
  })
}

addTask(10000, 1)
addTask(5000, 2)
addTask(3000, 3)
addTask(4000, 4)
addTask(5000, 5)
```

<!-- more -->

解决如下：

```js
class SuperTask {
  constructor() {
    // 1. 首先需要考虑我们会用到的一些数据
    // 并行任务数量
    this.limitedCount = 2
    // 当前正在运行的任务数量
    this.runningCount = 0
    // 当前的任务列表
    this.tasks = []
  }
  
  // 2. 需要把当前任务以及 resolve, reject 存入我们的任务列表中等待运行和消费
  add(task) {
    return new Promise((resolve, reject) => {
      // 把 resolve, reject 存入，是为了保证当任务消费完成后，能够告知外部当前任务的状态
      this.tasks.push({
        task,
        resolve,
        reject
      })
      this._run()
    })
  }

	// 3. 依次运行 tasks 队列中的所有任务
  _run() {
    while (this.runningCount < this.limitedCount && this.tasks.length) {
      const { task, resolve, reject } = this.tasks.shift()
      this.runningCount++
      task()
        .then(resolve, reject)
        .finally(() => {
          this.runningCount--
          this._run()
        })
    }
  }
}
```

