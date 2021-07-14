---
title: nodejs中的package.json相关问题
toc: true
date: 2021-07-12 10:51:37
categories:
  - Study Share
tags:
  - Node
  - NPM
---

将之前遇到的一些关于node工程目录的问题总结一下，偶尔看看来加深印象，未来会持续更新内容。

<!-- more -->

## 版本号中`~`和`^`的区别

node项目的package.json文件列出了项目所依赖的插件和库，同时也给出了对应的版本，但是在版本前面还有符号：`^`（插入符号）和`~`（波浪符号）,介绍下两个符号的区别：

```json
"dependencies": {
  "axios": "^0.18.0",
  "cross-env": "~5.2.0"
}
```

1. `^`插入符号
   它将会把当前库的版本更新到当前主版本（也就是第一位数字）中最新的版本。放到我们的例子中是：

   ```
   "axios": "^0.18.0"
   ```

   这个库会去匹配**0.x.x**中最新的版本，但是它不会自动更新到**1.0.0**。

2. `~`波浪符号
   它会更新到当前次版本号（也就是中间的那位数字）中最新的版本。放到我们的例子中就是：

   ```
   "cross-env": "~5.2.0"
   ```

   这个库会去匹配更新到**5.2.x**的最新版本，如果出了一个新的版本为**5.3.0**，则不会自动升级。

3. `~`波浪符号是曾经`npm`安装时候的默认符号，现在已经改为了`^`插入符号。

## package与package-lock

### 区别

1. **package.json的作用**

   package.json记录你项目中所需要的所有模块，当你执行`npm install`的时候，node会先从package.json中读取所有`dependencies`信息，然后根据`dependencies`中的信息与node_modules中的模块进行对比，没有的直接下载，已有的检查更新（最新版本的nodejs不会更新，因为有package-lock.json，下面再说）。

   另外，package.json只记录你通过`npm install`方式安装的模块信息，而这些模块所依赖的其他子模块的信息不会记录。

2. **package-lock.json的作用**

   package-lock.json锁定所有模块的版本号，包括主模块和所有依赖子模块。当你执行`npm install`的时候，node从package.json读取模块名称，从package-lock.json中获取版本号，然后进行下载或者更新。

   因此，正因为有了package-lock.json锁定版本号，所以当你执行`npm install`的时候，node不会自动更新package.json中的模块，必须用`npm install packagename（自动更新小版本号）`或者`npm install packagename@x.x.x（指定版本号）`来进行安装才会更新，package-lock.json中的版本号也会随着更新。

### 其他

当package.json与package-lock.json都不存在，执行`npm install`时，node会重新生成package-lock.json文件，然后把node_modules中的模块信息全部记入package-lock.json文件，但不会生成package.json文件，此时，你可以通过`npm init --yes`来初始化生成package.json文件。

### 总结

项目中引入的包版本号之前经常会加`^`，每次在执行`npm install`之后，下载的包都会发生变化，为了系统的稳定性考虑，每次执行完`npm install`之后会创建或者更新package-lock文件。该文件记录了上一次安装的具体的版本号，相当于是提供了一个参考，在出现版本兼容性问题的时候，就可以参考这个文件来修改版本号即可。

