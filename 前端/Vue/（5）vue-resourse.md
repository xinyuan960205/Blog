---
title: （5）vue-resource
date: 2019-01-15 11:02:32
tags:
- 前端
- vue
categories:
- 前端
- vue
---

# [vue-resource 实现 get, post, jsonp请求](https://github.com/pagekit/vue-resource)

除了 vue-resource 之外，还可以使用 `axios` 的第三方包实现实现数据的请求

1. 之前的学习中，如何发起数据请求？
2. 常见的数据请求类型？  get  post jsonp
3. 测试的URL请求资源地址：

- get请求地址： http://vue.studyit.io/api/getlunbo
- post请求地址：http://vue.studyit.io/api/post
- jsonp请求地址：http://vue.studyit.io/api/jsonp

## JSONP的实现原理

- 由于浏览器的安全性限制，不允许AJAX访问 协议不同、域名不同、端口号不同的 数据接口，浏览器认为这种访问不安全；
- 可以通过动态创建script标签的形式，把script标签的src属性，指向数据接口的地址，因为script标签不存在跨域限制，这种数据获取方式，称作JSONP（注意：根据JSONP的实现原理，知晓，JSONP只支持Get请求）；
- 具体实现过程：

- 先在客户端定义一个回调方法，预定义对数据的操作；
- 再把这个回调方法的名称，通过URL传参的形式，提交到服务器的数据接口；
- 服务器数据接口组织好要发送给客户端的数据，再拿着客户端传递过来的回调方法名称，拼接出一个调用这个方法的字符串，发送给客户端去解析执行；
- 客户端拿到服务器返回的字符串之后，当作Script脚本去解析执行，这样就能够拿到JSONP的数据了；

- 带大家通过 Node.js ，来手动实现一个JSONP的请求例子；

## vue-resource 的配置步骤

- 直接在页面中，通过`script`标签，引入 `vue-resource` 的脚本文件；
- 注意：引用的先后顺序是：先引用 `Vue` 的脚本文件，再引用 `vue-resource` 的脚本文件；

### 发送get请求

```javascript
getInfo() { // get 方式获取数据
  //  当发起get请求之后， 通过 .then 来设置成功的回调函数
  this.$http.get('http://127.0.0.1:8899/api/getlunbo').then(res => {
    // 通过 result.body 拿到服务器返回的成功的数据
    // console.log(result.body)
    console.log(res.body);
  })
}
```

### 发送post请求

```javascript
postInfo() {
  var url = 'http://127.0.0.1:8899/api/post';
  // post 方法接收三个参数：
  // 参数1： 要请求的URL地址
  // 参数2： 要发送的数据对象
  // 参数3： 指定post提交的编码类型为 application/x-www-form-urlencoded
  //  手动发起的 Post 请求，默认没有表单格式，所以，有的服务器处理不了
  //  通过 post 方法的第三个参数， { emulateJSON: true } 设置 提交的内容类型 为 普通表单数据格式
  this.$http.post(url, { name: 'zs' }, { emulateJSON: true }).then(res => {
    console.log(res.body);
  });
}
```

### 发送JSONP请求获取数据

```javascript
jsonpInfo() { // JSONP形式从服务器获取数据
  var url = 'http://127.0.0.1:8899/api/jsonp';
  this.$http.jsonp(url).then(res => {
    console.log(res.body);
  });
}
```

## 案例展示（品牌列表）

### 1、完成获取功能

```javascript
getAllList() { // 获取所有的品牌列表 
    // 分析：
    // 1. 由于已经导入了 Vue-resource这个包，所以 ，可以直接通过  this.$http 来发起数据请求
    // 2. 根据接口API文档，知道，获取列表的时候，应该发起一个 get 请求
    // 3. this.$http.get('url').then(function(result){})
    // 4. 当通过 then 指定回调函数之后，在回调函数中，可以拿到数据服务器返回的 result
    // 5. 先判断 result.status 是否等于0，如果等于0，就成功了，可以 把 result.message 赋值给 this.list ; 如果不等于0，可以弹框提醒，获取数据失败！
	
    this.$http.get('api/getprodlist').then(result => {
        // 注意： 通过 $http 获取到的数据，都在 result.body 中放着
        var result = result.body
        if (result.status === 0) {
            // 成功了
            this.list = result.message
        } else {
            // 失败了
            alert('获取数据失败！')
        }
    })
}
```

### 2、完成添加功能

```javascript
add() {  // 添加品牌列表到后台服务器
    // 分析：
    // 1. 听过查看 数据API接口，发现，要发送一个 Post 请求，  this.$http.post
    // 2. this.$http.post() 中接收三个参数：
    //   2.1 第一个参数： 要请求的URL地址
    //   2.2 第二个参数： 要提交给服务器的数据 ，要以对象形式提交给服务器 { name: this.name }
    //   3.3 第三个参数： 是一个配置对象，要以哪种表单数据类型提交过去， { emulateJSON: true }, 以普通表单格式，将数据提交给服务器 application/x-www-form-urlencoded
    // 3. 在 post 方法中，使用 .then 来设置成功的回调函数，如果想要拿到成功的结果，需要 result.body

    /* this.$http.post('api/addproduct', { name: this.name }, { emulateJSON: true }).then(result => {
      if (result.body.status === 0) {
          // 成功了！
          // 添加完成后，只需要手动，再调用一下 getAllList 就能刷新品牌列表了
          this.getAllList()
          // 清空 name 
          this.name = ''
      } else {
          // 失败了
          alert('添加失败！')
      }
    }) */

    this.$http.post('api/addproduct', { name: this.name }).then(result => {
        if (result.body.status === 0) {
            // 成功了！
            // 添加完成后，只需要手动，再调用一下 getAllList 就能刷新品牌列表了
            this.getAllList()
            // 清空 name 
            this.name = ''
        } else {
            // 失败了
            alert('添加失败！')
        }
    })
},
```

### 3、删除功能

```javascript
del(id) { // 删除品牌
    this.$http.get('api/delproduct/' + id).then(result => {
        if (result.body.status === 0) {
            // 删除成功
            this.getAllList()
        } else {
            alert('删除失败！')
        }
    })
}
```

### 4、全局配置

#### 配置数据接口的根域名

使用全局配置，这样就可以免去开头的IP了

在<script></script>代码块的开头中，添加改段代码

```javascript
// 如果我们通过全局配置了，请求的数据接口 根域名，则 ，在每次单独发起 http 请求的时候，请求的 url 路径，应该以相对路径开头，前面不能带 /  ，否则 不会启用根路径做拼接；
Vue.http.options.root = 'http://vue.studyit.io/';
```

get函数里面添加该不包含前面IP的url，注意开头不应包含“/”

```javascript
this.$http.get('api/getprodlist').then(result => {
```

#### emulateJSON选项

在<script></script>代码块的开头中，添加改段代码

```javascript
// 全局启用 emulateJSON 选项
Vue.http.options.emulateJSON = true;
```

全局启用就不用加{ emulateJSON: true }

```javascript
/*这是没有全局启用*/
this.$http.post('api/addproduct', { name: this.name }, { emulateJSON: true }).then(result => {
/*这是启用全局*/
this.$http.post('api/addproduct', { name: this.name }).then(result => {
```























