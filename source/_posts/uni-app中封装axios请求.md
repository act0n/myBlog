---
title: uni-app中封装axios请求
date: 2021-02-01 15:05:39
excerpt: (^_^)
categories:
        - uni-app
tags:
        - Vue.js
        - axios
        - uni-app
---



# uni-app中封装axios请求

## 安装

**安装axios**

```shell
npm install axios --save
```

**安装qs**

```shell
npm install qs --save
```

## 配置

1.在`src/`下新建一个`utils/request.js`

```js
import axios from "axios"
import qs from "qs"
import Vue from "vue"

// Full config:  https://github.com/axios/axios#request-config
// axios.defaults.baseURL = process.env.baseURL || process.env.apiUrl || '';
// axios.defaults.headers.common['Authorization'] = AUTH_TOKEN;
// axios.defaults.headers.post["Content-Type"] =
// "application/x-www-form-urlencoded;charset=UTF-8";

const service = axios.create({
	withCredentials: true,
	crossDomain: true,
	baseURL: Vue.prototype.baseURL, //这个baseURL是我在main.js下配置的请求url
	timeout: 6000
})

// request拦截器,在请求之前做一些处理
service.interceptors.request.use(
	config => {
		// if (store.state.token) {
		//     // 给请求头添加user-token
		//     config.headers["user-token"] = store.state.token;
		// }
		config.method === 'post' ?
			config.data = qs.stringify({ ...config.data
			}) :
			config.params = { ...config.params
			};
		console.log('请求拦截成功')
		return config;
	},
	error => {
		console.log(error); // for debug
		return Promise.reject(error);
	}
);

//配置成功后的拦截器
service.interceptors.response.use(res => {
	if (res.data.status == 200) {
		return res.data
	} else {
		return Promise.reject(res.data.msg);
	}
}, error => {
	return Promise.reject(error)
})

axios.defaults.adapter = function(config) { //自己定义个适配器，用来适配uniapp的语法
	return new Promise((resolve, reject) => {
		// console.log(config)
		var settle = require('axios/lib/core/settle');
		var buildURL = require('axios/lib/helpers/buildURL');
		uni.request({
			method: config.method.toUpperCase(),
			url: config.baseURL + buildURL(config.url, config.params, config.paramsSerializer),
			header: config.headers,
			data: config.data,
			dataType: config.dataType,
			responseType: config.responseType,
			sslVerify: config.sslVerify,
			complete: function complete(response) {
				console.log("执行完成：", response)
				response = {
					data: response.data,
					status: response.statusCode,
					errMsg: response.errMsg,
					header: response.header,
					config: config
				};
				settle(resolve, reject, response);
			}
		})
	})
}
export default service
```

2.在`src/`下新建一个`api/user.js`，当作各种请求汇总文件夹

```js
//引入刚才创建的request.js
import request from '@/utils/request' 

//GTE 传参需要用 params
//POST 传参需要用 data

export function getMsg() {
	return request({
		url: '/homepage/vertical',
		method: 'get'
	})
}

```

3.可以结合`Vuex`在`store/actions`下使用，也可以直接使用，下面演示直接在`pages/index/index.vue`使用

```vue
<template>
	<view>
	</view>
</template>

<script>
    //引入函数
    import { getMsg } from '@/api/user'
    export default {
        data() {
            return {

            }
        },
        onLoad() {
            getMsg().then(res => {
                console.log("请求res内容",res);
            })
            .catch(err => {
                console.log("请求err内容",err)
            })
        }
    }
</script>

<style>
</style>

```

**结果显示：**请求到了内容，说明调通了

![alt](http://cdn.wydgit.top/res.png)
