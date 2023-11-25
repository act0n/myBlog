---
title: webpack 中 loader 和 plugin 的不同
toc: true
categories:
  - Front-end Stud
tags:
  - webpack
date: 2023-11-25 22:42:46
---

## webpack 中 Loader 和 Plugin 的不同点

### 作用不同

- Loader 用于转换某些类型的模块，为其他模块提供加载支持。

  提供给 webpack 加载和解析非 JavaScript 文件的一个能力

- Plugin 用于改变 webpack 的打包行为。

  扩展 webpack 的功能，使 webpack 拥有更多的灵活性

<!-- more -->

### 用法不同

- Loader 是在 modules.rules 中配置的，可以作为模块的解析规则，类型是一个数组，每一项是一个对象，对象内声明了配置内容就是一个什么样类型的文件，可以使用什么样的 loader 去进行加载，以及有不同的参数 options。
- Plugin 是在 跟 modules 同级的 plugins 中单独配置的，也是一个数组，但是每一项是一个插件实例，每一个插件的配置参数内容是通过构造函数传入进去的

### 执行时机不同

在 Webpack 中，Loader 和 Plugin 的执行时间取决于它们在构建过程中所处的阶段。

- Loader 主要在模块加载阶段执行，即在模块被导入时执行。Loader 的执行时间通常在构建阶段的前端部分。例如，当使用 Babel 进行代码转换时，Loader 会在构建阶段的前端部分将 JavaScript 代码转换为浏览器支持的代码。

- 而 Plugin 主要在构建阶段的全局阶段执行，包括构建阶段的前端部分和后端部分。例如，当使用 UglifyJS 进行代码压缩时，Plugin 会在构建阶段的后端部分将代码压缩为更小的体积。

需要注意的是，Loader 和 Plugin 的执行时间可能会因为构建配置的变化而变化。例如，如果使用了动态导入，那么 Loader 的执行时间可能会在导入时发生改变。同样，如果使用了动态加载的 Plugin，那么 Plugin 的执行时间也会在构建阶段的全局阶段发生改变。

### 常用的 Loader 和 Plugin

1. file-loader：把文件输出到一个文件夹中，在代码中通过相对 URL 去引用输出的文件
2. url-loader：和 file-loader 类似，但是能在文件很小的情况下以 base64 的方式把文件内容注入到代码中去
3. source-map-loader：加载额外的 Source Map 文件以方便断点调试
4. image-loader：加载并且压缩图片文件
5. babel-loader：把 ES6 转换成 ES5
6. css-loader：加载 CSS，支持模块化、压缩、文件导入等特
   性
7. style-loader：把 PSS.代确法入到 paygScript 中，通过 DOM 操作去加载 CSS his loaderand our plugtn
8. eslint-loader：通过 ESLint 检查 JavaScript 代码太常用的 plugin 以及相应的作用如下：
9. Webpack-dashboard：可以更友好的展示相关打包信息
10. Webpack-merge： 提取公共配置，减少重复配置代码
11. speed-measure-Webpack-plugin：简称 SMP，分析出 webpack 打包过程中 Loader 和 plugin 的耗时，有助于找到构建过程中的性能瓶颈
12. size-plugin：监控资源体积变化，尽早发现问题
13. HotModuleReplacementPlugin :模块热替换


