# 接口设计

## vue-cli3根据不同环境配置axios的baseUrl

参考项目使用了window的方式进行的全局访问，但是在新项目中就访问不到了。

vue-cli3根据不同环境配置axios的baseUrl

在vue-cli3中，npm run serve时会把process.env.NODE_ENV设置为‘development’；

npm run build时会把process.env.NODE_ENV设置为‘production’；

1. 新建baseUrl.js,添加内容如下：

```js
let baseUrl= "";   //这里是一个默认的url，可以没有
switch (process.env.NODE_ENV) {
    case 'development':
        baseUrl = "http://localhost:3000/"  //开发环境url
        break
    case 'ceshi':   // 注意这里的名字要和步骤二中设置的环境名字对应起来
        baseUrl = "http://localhost:3000/"  //测试环境中的url
        break
    case 'production':
        baseUrl = "http://106.13.94.82:3000"   //生产环境url
        break
}

export default baseUrl;
```

2. axios中引入文件并使用

```js
import axios from 'axios';
import baseUrl from './baseUrl '
http.adornUrl = (actionName) => {
  // 非生产环境 && 开启代理, 接口前缀统一使用[/proxyApi/]前缀做代理拦截!
  return baseUrl + actionName
}
```





## httpRequest

/src/utils/httpRequest.js

### 设置头部等公有信息

```js
const http = axios.create({
  timeout: 1000 * 30,
  withCredentials: true, // 当前请求为跨域类型时是否在请求中协带cookie
  headers: {
    'Content-Type': 'application/json;charset=utf-8'
  }
})
```



### 请求拦截

```js
/**
 * 请求拦截
 */
http.interceptors.request.use(config => {
  // 处理请求之前的配置
  config.headers['token'] = Vue.cookie.get('token') // // 请求头带上token
  return config
}, error => {
  // 请求失败的处理
  return Promise.reject(error)
})
```

给请求带上token



### 响应拦截

```js
/**
 * 响应拦截
 */
http.interceptors.response.use(response => {
  if (response.data && response.data.code === 403) { // 401 token失效
    clearLoginInfo()
    router.push({ name: 'login' })
  }
  return response
}, error => {
  return Promise.reject(error)
})
```

如果是403就强制登出，其实这里401也应该强制登出。



### 请求地址处理

```js
/**
 * 请求地址处理
 * @param {*} actionName action方法名称
 */
http.adornUrl = (actionName) => {
  // 非生产环境 && 开启代理, 接口前缀统一使用[/proxyApi/]前缀做代理拦截!
  return baseUrl + actionName
}
```

根据不同的环境，接口前缀统一使用。



### 请求参数处理

```js
/**
 * get 请求参数处理
 * @param params
 * @param openDefaultParams
 * @returns {*}
 */
http.adornParams = (params = {}, openDefaultParams = true) => {
  var defaluts = {
    't': new Date().getTime()
  }
  return openDefaultParams ? merge(defaluts, params) : params
}
/**
 * post请求参数处理
 * @param data
 * @param openDefaultdata
 * @param contentType
 * @returns {string}
 */
http.adornData = (data = {}, openDefaultdata = true, contentType = 'json') => {
  var defaults = {
    't': new Date().getTime()
  }
  data = openDefaultdata ? merge(defaults, data) : data
  return contentType === 'json' ? JSON.stringify(data) : qs.stringify(data)
}
```

请求的参数都加上时间

## 使用

```js
this.$http({
  url: this.$http.adornUrl('/admin/sys/login'),
  method: 'post',
  data: this.$http.adornData({
    'username': this.dataForm.userName,
    'password': this.dataForm.password,
    'uuid': this.dataForm.uuid,
    'captcha': this.dataForm.captcha
  })
}).then(({data}) => {
  if (data && data.code === 200) {
    this.$cookie.set('token', data.token)
    this.$router.replace({ name: 'home' })
  } else {
    this.getCaptcha()
    this.$message.error(data.msg)
  }
})
```











