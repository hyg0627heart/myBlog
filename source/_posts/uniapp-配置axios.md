---
title: uniapp 配置axios
date: 2022-01-05 16:51:10
tags:
---

```
// http.js文件

import Vue from 'vue'
import axios from 'axios'
let baseURL = 'http://'
const service = axios.create({
    withCredentials: true,
    crossDomain: true,
    baseURL,
    timeout: 6000
})

// request拦截器,在请求之前做一些处理
service.interceptors.request.use(
    config => {
        const config = { ...defaults}
		const user = uni.getStorageSync('user');
		if(user){
			const token = user.token
			config.headers = {token}
		}
		return config
    },
    error => {
        console.log(error); // for debug
        return Promise.reject(error);
    }
);

//配置成功后的拦截器
service.interceptors.response.use(res => {
    if (res.data.status== 200) {
        return res.data
    } else {
        return Promise.reject(res.data.msg);
    }
}, error => {
    return Promise.reject(error)
})


axios.defaults.adapter = function(config) { //自己定义个适配器，用来适配uniapp的语法
    return new Promise((resolve, reject) => {
        console.log(config)
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
                console.log("执行完成：",response)
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

```
// main.js文件

import Vue from 'vue';
import App from './App';
import axios from './api/http.js'

Vue.config.productionTip = false;
Vue.prototype.$http = axios
```

```
// login.vue 组件文件中
const res = await this.$http.post('api/auth/login', sendData);
if (res.code == 1) {
	this.login(res.data);
		uni.navigateBack();
	} else {
		this.$api.msg(res.msg);
		this.logining = false;
	}
```

