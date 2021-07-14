---
title: Windows配置nvm所遇到的一些问题
toc: true
date: 2021-07-08 15:36:59
categories:
  - Config Share
tags:
 - NVM
---



之前一直单独用的`node	`，对于`node`的版本也不敏感，然后组长说之后项目会需要不同的`node`版本，就让我们去了解安装一下`nvm`。今天第一次接触到`nvm`这个工具，就先下载了，虽然也提示了：可能会删掉部分原先`node`的配置，但我没放心上，结果有的`node`安装的命令找不到了，然后花了大量时间才算是了解、配置成功。

<!-- more -->

## 什么是nvm

> nvm is a version manager for [node.js](https://nodejs.org/en/), designed to be installed per-user, and invoked per-shell. `nvm` works on any POSIX-compliant shell (sh, dash, ksh, zsh, bash), in particular on these platforms: unix, macOS, and windows WSL.

简单点说：`nvm`是一个`node`的版本管理工具。

因为`node`有很多版本嘛，所以这个工具就是用来下载、安装和管理`node`的各个版本

## 准备
如果曾经安装过`node`，需要先删除卸载掉`node`，从`控制面板` => `卸载` => 找到node，卸载。

如果没有安装过，可以忽略这一步。 

## 下载安装

### 命令下载

> 本人未使用这种方式，谨慎使用🙃，Windows的话直接官网下载比较好

```shell
# 1. curl方式
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash
# 2. wget方式
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash
```

### 官网下载

地址在这里，直接下载安装[nvm for Windows](https://github.com/coreybutler/nvm-windows/releases)

- nvm-noinstall.zip：绿色免安装版，但使用时需进行配置。
- nvm-setup.zip：安装版，推荐使用

## 安装配置

### npm配置

下载成功之后，会有两个步骤，这里不贴图片了。

1. 选择安装`nvm`的目录：

   建议放到`d:\nvm\nvm\`

2. 选择安装`node`的目录

   建议放到`d:\nvm\nodejs\`

之所以这样配置，是方便管理和使用，其他目录也是可以的。

> **注意**：安装完成之后，这个`nodejs`文件夹是还没有创建的，因为还没有安装`node`

进入`d:\nvm\nvm\`，打开`settings.txt`添加这两行内容，分别是`node`和`npm`的下载源

```yaml
node_mirror: https://npm.taobao.org/mirrors/node/
npm_mirror: https://npm.taobao.org/mirrors/npm/
```

然后下载`npm`的某一版本，可以参考以下命令

```shell
nvm install 12.22.3  # 下载安装 12.22.3版本的node，并且会把对应版本的npm也下载好
nvm use 12.22.3      # 本地环境切换到 12.22.3 版本的node
```

这个时候，`d:\nvm\nodejs\`就创建成功了，然后在里面创建`node_global`和`node_cache`两个文件夹，它们分别用来保存`npm`全局安装包和缓存的。

**文件名称千万别拼写错了**！

然后执行下面两条命令来修改`npm`全局安装包和缓存的位置：

```shell
npm config set prefix="D:\nvm\nvm\nodejs\node_global"

npm config set cache="D:\nvm\nvm\nodejs\node_cache"
```

这里需要用**引号**包裹住，否则会出现目录问题，可以用以下命令检测一下 ，看看目录是否有一致

```shell
npm config get prefix

npm config get cache
```

这时候使用`npm`安装的全局包都会安装到指定的路径，可以在命令行工具执行`npm i express -g`，然后去`D:\nvm\nodejs\node_global\node_modules`目录查看是否有相应的文件。如果没有，需要再检查下前面设置的目录是否出错。

配置好后在`C:\Users\administrator`下会出现`.npmrc`这个文件，说明配置成功。

这个时候，我们使用`npm`全局安装包的时候，就已经能够安装到我们指定的目录下，但是我们还没有办法使用安装好的**全局命令**。

### 环境变量配置

一般环境变量在你安装的时候就已经配置好了，可以检查一下，如果没有，需要自己配置进去。

**用户变量**

|    变量     |                   值                   | 备注 |
| :---------: | :------------------------------------: | :--: |
|  NVM_HOME   |               D:\nvm\nvm               |      |
| NVM_SYMLINK |             D:\nvm\nodejs              |      |
|  NODE_PATH  | D:\nvm\nodejs\node_global\node_modules |      |



**系统变量**

| 变量 |            值             | 备注 |
| :--: | :-----------------------: | :--: |
| Path |        %NVM_HOME%         |      |
|      |       %NVM_SYMLINK%       |      |
|      | D:\nvm\nodejs\node_global |      |

**NODE_PATH 和 Path 中 node相关路径的区分**

操作系统中都会有一个`Path `（它是不区分大小写）环境变量，当系统调用一个**命令**的时候，就会在`Path`变量中注册的路径中寻找，如果注册的路径中有就调用，否则就提示命令没找到。

> 这也就是为什么有时候我们安装好了相关包文件，但是使用不了相应的命令：xxx not found

那 `NODE_PATH` 就是`node`中用来寻找**模块**所提供的路径注册环境变量。

> 比如一些常用的模块文件

简单来说，`PATH`用来找命令，`NODE_PATH`用来找模块。

`node`安装的相应的全局命令，要么在对应文件的`bin`下，要么在`node_global`目录下，所以我们选择后者，一次配置就可以了😁。

而相应的全局文件，则是在`node_global/node_modules`下，所以这两者略有差异。

> 终于把它们区分清楚了！

## 其他注意事项

1. 使用`nvm`管理版本的时候，各个`npm`版本的包文件是不相通的，也就是说`12.x`版本的`npm`用不了`11.x`版本的包文件，虽然有点麻烦，但是也比较合理，毕竟各个版本用途可能会有变动。



## 参考资料

[nvm安装以及node配置](https://juejin.cn/post/6844903983161540622)

[安装node.js和Hexo](https://zhuanlan.zhihu.com/p/105715224)

