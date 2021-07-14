---
title: uni-app学习基础
date: 2021-02-01 15:05:00
toc: true
categories:
  - Front-end Study
tags:
 - Vue
 - Uni-app

---



## 一、uni-app的初体验

### 1.1开发方式

- 通过HBuilderX可视化界面
  - 不多介绍
- 通过vue-cli命令行

<!-- more -->

### 1.2脚手架搭建项目

1. 环境安装

   全局安装vue-cli

   ```shell
   npm install -g @vue/cli
   ```

2. 创建项目

   **使用正式版**（对应HBuilderX最新正式版）

   ```shell
   vue  create dcloudio/uni-preset-vue my-project
   ```

   **使用alpha版**（对应HBuilderX最新alpha版）

   ```shell
   vue create -p dcloudio/uni-preset-vue#alpha my-alpha-project
   ```

   此时，会提示选择项目模板，初次体验建议选择 `hello uni-app` 项目模板，当然也可以使用默认模板（本项目是默认模板）

3. 启动项目（微信小程序）

   ```shell
   npm run dev:mp-weixin
   ```

   **当然，还有其他版本：支付宝小程序、qq小程序等**

4. 微信小程序开发者工具导入项目

   **需要把 `my-project/dist/dev/mp-weixin` 这个目录导入微信小程序开发者工具其中**

## 二、项目结构介绍

**src/**

![](http://cdn.wydgit.top/pp1.png)



## 三、样式和sass

- 支持小程序的 `rpx` 和h5的 `vw` 、 `vh`
  - `rpx`为小程序单位：750rpx = 屏幕宽度
  - `vw` 、`vh`是h5单位，100vw = 屏幕的宽度，100vh = 屏幕的高度
- 内置有sass的配置，只需要安装对应的依赖即可：`npm install sass-loader node-sass`
- vue组件中，在style标签上加上属性 `<style lang='scss'>` 即可

## 四、基本语法

### 4.1数据的展示

#### 定义

在js的**`data`**中定义数据

```js
data(){
    return{
        msg:"hello",
        color:'red'
    }
}
```

#### 使用

- 在template中通过**{{ 数据 }}**来展示

  `{{msg}}`

- 在标签的属性上通过 `:data-index='数据'` 来使用，从而数据对应标签属性

  `<view :data-color="color">nice</view>`

### 4.2数据的循环

- 通过 `v-for` 来指定要循环的数组

- `item` 和 `index` 分别为**循环项**和**循环索引**

- `:key`指定唯一的属性，用来提高循环效率

```js
data(){
    return{
        list:[
            {id:001,text:"苹果🍎"},
            {id:002,text:"香蕉🍌"},
            {id:003,text:"樱桃🍒"}
        ]
    }
}
```

```vue
<view v-for="(item,index) in list" :key="item.id">
{{index}} -- {{item.id}} -- {{item.text}}
</view>
```

### 4.3条件编译

- 通过 `v-if` 来决定显示和隐藏

**通过删除了标签来实现**，**不适合做频繁的切换显示**

- 通过 `v-show` 来决定显示和隐藏

**通过 `display: none;` 来实现**，**适合做频繁的切换显示**

```vue
<view>
	<view v-if="false">v-if</view>
    <view v-show="false">v-show</view>
    <!-- 通过变量来控制 -->
    <view v-if="isShow">显示或隐藏</view>
</view>
```

### 4.4计算属性

- 可以理解为是对 `data` 中的数据提供了一种加工或者过滤的能力
- 通过 `computed` 来定义计算属性

```vue
<!-- 计算能力-->
<view>
    {{getMoney}}
</view>
<!-- 过滤能力 -->
<view v-for="(item,index) in filterList" :key="item.id">
{{index}} -- {{item.id}} -- {{item.text}}
</view>
```

```js
computed: {
    getMoney(){
      return '￥'+ this.money;  
    },
    filterList(){
        //要把 id>0 的数据项都过滤掉
        return this.list.filter(v => v.id <= 0);
    },
    processList(){
        return this.list.map(v => {
            v.color = "red";
            return v;
        })
    }
}
```

## 五、事件

- 注册事件 `@click= "handleClick"`
- 定义事件监听函数，需要在 `methods` 中定义
- 事件的传参

```vue
<view @click="handleClick">please click here</view>
<view data-index="ppp" @click="printClick(1,$event)">click to print content of data-index</view>
```

```js
methods: {
    handleClick(){
        console.log("点击生效了！");
    },
    printClick(obj,event){
        console.log(obj);
        console.log(event);
        console.log(event.currentTarget.dataset.index);
    }
}
```

## 六、组件

### 6.1组件的简单使用

- 组件的定义

  - 在 `src` 目录下新建文件夹 `components` 用来存放组件
  - 在 `components` 目录下直接新建组件 `*.vue`

- 组件的引入

  - 在页面中引入组件 `import 组件名 from '组件路径'`

- 组件的注册

  - 在页面中的实例中，新增属性 `components`
  - 属性 `components` 是一个对象，把组件放进去注册

- 组件的使用

  - 在页面的标签中，直接使用引入的组件 `<组件></组件>

  components/img-border.vue

  ```vue
  <template>
  	<image src="http://example.com.1.jpg"></image>
  </template>
  
  <script>
  export default {
      
  }
  </script>
  
  <style>
  </style>
  ```

  pages/index/index.vue

  ```vue
  <template>
  	<view>
          <imgBorder></imgBorder>
          <!-- 或者下面这种格式,这两种格式是等价的 -->
          <img-border></img-border>
      </view>
  </template>
  
  <script>
  // @ 对应的目录是 src/
  import imgBorder from "@/components/img-border";
  export default {
      components: {
          imgBorder
      }
  }
  </script>
  
  <style>
  </style>
  ```

### 6.2组件传参

#### 父向子传递参数，通过**属性**的方式

- 父页面向子组件  `imgBorder` 或者 `img-border` 通过属性名 `src` 传递了一个数组数据
- 子组件通过 `props `进行接收数据·

pages/index/index.vue

```vue
<template>
	<view>
        <imgBorder :src="mySrc"></imgBorder>
    </view>
</template>

<script>
// @ 对应的目录是 src/
import imgBorder from "@/components/img-border";
export default {
    data(){
      return {
          mySrc:"http://www.baidu.com/example.png"
      }  
    },
    components: {
        imgBorder
    }
}
</script>

```

components/img-border.vue

```vue
<template>
	<!-- 把props中的src看作是data中的变量一样来使用 -->
	<image :src="src"></image>
</template>

<script>
export default {
    props:{
        src:String
    }
}
</script>
```

#### 子向父传递参数，通过**触发事件**的方式

- 子组件通过**触发事件**的方式向父组件传递数据
- 父组件通过**监听事件**的方式来接收数据

components/img-border.vue

```vue
<template>
	<image @click="handleClick" :src="src"></image>
</template>

<script>
export default {
    props:{
        src:String
    },
    methods:{
        handleClick(){
            //子向父传递数据通过 触发事件
            //this.$emit("自定义的事件名称","要传递的参数")；
            this.$emit("srcMsg",this.src);
        }
    }
}
</script>
```

pages/index/index.vue

```vue
<template>
	<view>子组件传递过来的路径：{{childSrc}}</view>
	<view>
        <imgBorder @srcMsg="handleSrcMsg" :src="mySrc"></imgBorder>
    </view>
</template>

<script>
// @ 对应的目录是 src/
import imgBorder from "@/components/img-border";
export default {
    data(){
      return {
          mySrc:"http://www.baidu.com/example.png",
          childSrc:""
      }  
    },
    components: {
        imgBorder
    },
    methods:{
        handleSrcMsg(e){
            this.childSrc = e;
        }
    }
}
</script>
```

#### 通过全局数据传递参数

- 通过挂载`vue`的原型上

  在main.js或者任何有 `import Vue from 'vue'` 的页面内定义

  ```vue
  Vue.prototype.baseURL = "http://www.baidu.com"
  ```

  当作是 `data` 中的变量一样来使用

  ```js
  this.baseURL
  ```

- 通过`globalData`的方式

  在 `App.vue` 中定义

  ```vue
  <script>
      export default {
          onShow: function(){
              
          },
          globalData:{
              base:"www.360.com"
          }
      }
  </script>
  ```

  使用

  ```js
  getApp().globalData.base
  ```

  修改

  ```js
  getApp().globalData.base = "xxx"
  ```

### 6.3组件插槽

- 标签其实也是数据中的一种，想实现动态的给子组件传递标签，就可以使用插槽slot
- 通过slot来实现占位符

components/my-form.vue

```vue
<template>
	<view class="form">
        <view class="form_title">标题</view>
        <view class="form_content">内容</view>
        <!-- 插槽 来占位置 -->
        <slot></slot>
    </view>
</template>

<script>
export default {
    
}
</script>

<style>
</style>
```

pages/index/index.vue

```vue
<template>
	<view>
        <myForm>
            <view>
                <imput placeholder="please input" type="text" />
                <radio></radio>
                <checkbox></checkbox>
    		</view>
    	</myForm>
    </view>
</template>

<script>
// @ 对应的目录是 src/
import myForm from "@/components/my-form";
export default {
    components: {
        myForm
    }
}
</script>

<style>
</style>
```

## 七、生命周期

- uni-app框架的生命周期结合了vue和微信小程序的生命周期
- 全局的APP中使用 `onLaunch` 表示应用启动时
- 页面中，使用 `onLoad` 或者 `onShow` 分别表示页面加载完毕时和页面显示时
- 组件中使用 `mounted` 表示组件挂载完毕时

[uniapp生命周期](http://uniapp.dcloud.io/frame?id=生命周期)

[Vue生命周期](http://cn.vuejs.org/v2/guide/instance.html?#生命周期图示)

