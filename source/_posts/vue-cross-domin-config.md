---
title: vue跨域配置解析
toc: true
categories:
  - Config Share
tags:
  - Cross-domin
  - Vue
date: 2021-07-12 17:06:49
---
## 前言

`跨域`这个问题其实老生常谈了，简单来讲就是，浏览器由于`同源策略`规定，AJAX请求只能发给同源的网址，否则就报错。

所谓`同源`，就是**域名**、**协议**、**端口**相同。

> 同源策略是浏览器的行为，是为了保护本地数据不被JavaScript代码获取回来的数据污染，因此拦截的是客户端发出的请求回来的数据接收，即请求发送了，服务器响应了，但是无法被浏览器接收。

<!-- more -->

### 举个例子

我们请求一下百度首页：

```js
this.$axios.get("https://www.baidu.com")
.then(res=>{
    console.log(res)
})
.catch(err=>{
    console.log(err)
})
```

然后，浏览器检查一下控制台，报错情况如下：

```
Access to XMLHttpRequest at 'https://www.baidu.com/' from origin 'http://localhost:8080' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.
list.vue?61b8:36 Error: Network Error
    at createError (createError.js?16d0:16)
    at XMLHttpRequest.handleError (xhr.js?ec6c:84)
```

虽然报错了，但是浏览器其实是获取到数据的，只不过这样报错，不显示数据。

接下来让我们解决这个问题。

## Vue2

### 配置BaseUrl

首先在`main.js`中，配置下我们访问的Url前缀：

```js
import Vue from 'vue'
import App from './App'
import Axios from 'axios'

Vue.prototype.$axios = Axios
Axios.defaults.baseURL = '/test'
Axios.defaults.headers.post['Content-Type'] = 'application/json';

Vue.config.productionTip = false

/* eslint-disable no-new */
new Vue({
  el: '#app',
  components: { App },
  template: '<App/>'
})
```

关键代码是：`Axios.defaults.baseURL = '/test'`，这样每次发送请求都会带一个`/test`的前缀。

### 配置代理

修改`config`文件夹下的`index.js`文件，在`proxyTable`中加上如下代码：

```js
'/test':{
    target: "https://www.baidu.com",
    changeOrigin:true,
    pathRewrite:{
        '^/test':'/'
    }
}
```

这段代码，`vue-cli`启动了`http-proxy-middleware`代理服务。

第一个`/test`，相当于是当请求链接遇到`/test`时，会将`/test`拼接到`target`的链接下，比如我们在写`axios`请求的时候，只用写成`/test/1`就可以代表`https://www.baidu.com/test/1`。

```
// 举例  原先的请求
http://localhost:8080/test
// 代理后的请求
https://www.baidu.com/test
```

然后就是`pathRewrite`起作用了，它能够用正则匹配到相应的字符段，然后进行路径替换，将前者替换为后者，这里看个人习惯来命名参数。也就是说`pathRewrite`起重新加工处理的作用。

```
// 代理后的请求
https://www.baidu.com/test
// 重写后的请求
https://www.baidu.com/
```

如果想了解更多关于`pathRewrite`参数的讲解和实例，可以看看这里

[http-proxy-middleware中的pathRewrite](https://github.com/chimurai/http-proxy-middleware/blob/master/recipes/pathRewrite.md#rewrite-paths)

[pathRewrite](https://lvyongbo.gitbooks.io/http-proxy-middleware/content/pathrewrite.html)

### 修改请求Url

```js
this.$axios.get("/")
.then(res=>{
	console.log(res)
})
.catch(err=>{
	console.log(err)
})
```

### 重启服务

`ctrl + c`终端后，再重新启动一下，就OK了，就能够正常请求到了。

这是控制台打印出来的内容：

```json
{data: "<!DOCTYPE html><!--STATUS OK-->\n\n\n    <html><head>…5a9387c5b.js\"></script>\n</body>\n        \n\t</html>", status: 200, statusText: "OK", headers: {…}, config: {…}, …}
```

## Vue3

基本上跟vue2一样，只是配置代理的时候有点不同。

### 配置代理

由于`vue3`删除了`config`文件夹，所以没了对应的`index.js`文件，因此我们要自己新建一个`config`文件。

在项目根目录下新建一个`vue.config.js`文件，添加基础内容如下：

```js
module.exports = {
    outputDir: 'dist',   //build输出目录
    assetsDir: 'assets', //静态资源目录（js, css, img）
    lintOnSave: false, //是否开启eslint
    devServer: {
        open: true, //是否自动弹出浏览器页面
        host: "localhost", 
        port: '8080', 
        https: false,   //是否使用https协议
        hotOnly: false, //是否开启热更新
        proxy: null,
    }
}
```

修改`devServer`下的`proxy`：

```js
proxy: {
  '/test': {
    target: 'https://www.baidu.com', //API服务器的地址
    changeOrigin: true,
    pathRewrite: {
      '^/test': '/'
    }
  }
}
```

## 参考文档

- [Axiso解决跨域访问](https://blog.csdn.net/yuanlaijike/article/details/80522621)
- [思否](https://segmentfault.com/q/1010000012607105)
- [Vue-cli proxyTable 解决开发环境的跨域问题](https://www.jianshu.com/p/95b2caf7e0da)

