---
title:  Express中配置https
date: 2021-05-01 16:34:21
toc: true
tags:
 - express
 - https
 - nodejs
---

> 微信小程序要想发布就需要配置域名，而且必须是https，所以就根据自己的探索经历记录下来



<!-- more -->

## 流程

阿里云网站上的流程指示：

![guide img](https://img.alicdn.com/tfs/TB1Fe9fxQP2gK0jSZPxXXacQpXa-1006-109.svg) 

## 申请、下载证书

1. 登录[阿里云](https://www.aliyun.com/)，搜索SSL证书，然后点击`管理控制台`
2. 点击`SSL证书`，`立即购买`，个人的可以使用免费的
3. 之后`创建证书`然后`证书申请`
4. 一系列内容填写之后就能获取到`已签发`的证书了，然后可以对于自己的服务器下载相应的证书，由于我们是node来搭建的服务器，所以下载`其他`即可
5. 将下载的两个文件，放置到express应用实例的根目录下的key文件夹（当然，你们可以自定义，只要找得到这俩文件）

## 设置Express

express默认是http，并且express内置了https核心模块文件，所以我们要用https也很简单，直接引入、配置即可

这是我的app.js文件

```js
const express = require('express')
const fs = require('fs')
const app = express()
const http = require('http')
const https = require('https')
//8080端口开启http
http.createServer(app).listen(8080, "0.0.0.0",() => {
	console.log('Running 8080 ...')
})

const options = {
  key: fs.readFileSync('./key/server.key'),
  cert: fs.readFileSync('./key/server.pem')
}
//8443端口开启https
https.createServer(options, app).listen(8443, '0.0.0.0',() => {
	console.log('Running 8443 ...')
})
```

重启之后就可以了



## 生成

> 如果有的小伙伴想自己生成然后设置，也很简单

生成key和cert

```css
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365
```

有点不太的就是文件名和类型可能有点不同，但是实质都一样，设置即可

```jsx
const options = {
  key: fs.readFileSync('./key/key.pem'),
  cert: fs.readFileSync('./key/cert.pem')
}
```

其他内容跟上面讲的均相同

## 参考链接

[Express 配置https](https://www.jianshu.com/p/18b74c0df20c)

[nodejs配置 https服务](https://www.cnblogs.com/500m/p/11805384.html)