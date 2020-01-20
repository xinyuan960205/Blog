---
title: （8）vue路由
date: 2019-02-01 10:13:06
tags:
- 前端
- vue
categories:
- 前端
- vue
---

# vue路由

## 什么是路由

1. 对于普通的网站，所有的超链接都是URL地址，所有的URL地址都对应服务器上对应的资源；
2. 对于单页面应用程序来说，主要通过URL中的hash(#号)来实现不同页面之间的切换，同时，hash有一个特点：HTTP请求中不会包含hash相关的内容；所以，单页面程序中的页面跳转主要用hash实现；
3. 在单页面应用程序中，这种通过hash改变来切换页面的方式，称作前端路由（区别于后端路由）；

## 使用vue-router

1. 导入 vue-router 组件类库

```html
  <!-- 1. 安装 vue-router 路由模块 -->
  <script src="./lib/vue-router-3.0.1.js"></script>
```

2. 使用 router-link 组件来导航

- router-link 默认渲染为一个a 标签，如果想要渲染成span标签，可以使用`tag=“span”`进行设置

```html
    <router-link to="/login" tag="span">登录</router-link>
    <router-link to="/register">注册</router-link>
```

3. 使用 router-view 组件来显示匹配到的组件

- 这是 vue-router 提供的元素，专门用来 当作占位符的，将来，路由规则，匹配到的组件，就会展示到这个 router-view 中去。
- 所以： 我们可以把 router-view 认为是一个占位符。
- 注意：这里的transition是给路由添加了动画。

```html
    <transition mode="out-in">
      <router-view></router-view>
    </transition>
```

4. 创建使用`Vue.extend`创建组件

```js
    // 组件的模板对象
    var login = {
      template: '<h1>登录组件</h1>'
    }

    var register = {
      template: '<h1>注册组件</h1>'
    }

```

5. 创建一个路由 router 实例，通过 routers 属性来定义路由匹配规则

- 每个路由规则，都是一个对象，这个规则对象，身上，有两个必须的属性：
- 属性1 是 path， 表示监听 哪个路由链接地址；
- 属性2 是 component， 表示，如果 路由是前面匹配到的 path ，则展示 component 属性对应的那个组件
- 注意： component 的属性值，必须是一个 组件的模板对象， 不能是 组件的引用名称；

```js
    var routerObj = new VueRouter({
      // route // 这个配置对象中的 route 表示 【路由匹配规则】 的意思
      routes: [ // 路由匹配规则       
        { path: '/', redirect: '/login' }, // 这里的 redirect 和 Node 中的 redirect 完全是两码事
        { path: '/login', component: login },
        { path: '/register', component: register }
      ],
      linkActiveClass: 'myactive'
    })
```

6. 使用 router 属性来使用路由规则

```js
    var vm = new Vue({
      el: '#app',
      data: {},
      methods: {},
      router: routerObj // 将路由规则对象，注册到 vm 实例上，用来监听 URL 地址的变化，然后展示对应的组件
    });
```

## router-link使用

- 使用tag指定router-link渲染的标签类型

```html
    <router-link to="/login" tag="span">登录</router-link>
```

## redirect重定向

这样就可以解决一开始打开网页时，没有匹配的组件；而且当不会产生匹配到login组件，但是URL却显示“/”而不是显示“/login”的问题。

```js
    var routerObj = new VueRouter({
      routes: [ // 路由匹配规则 
        // { path: '/', component: login },
        { path: '/', redirect: '/login' }, // 这里的 redirect 和 Node 中的 redirect 完全是两码事
        { path: '/login', component: login },
        { path: '/register', component: register }
      ],
    })
```

## 设置路由高亮

可以在style中设置router-link-active的属性，这个应该是默认的

```css
    .router-link-active{
      color: red;
      font-weight: 800;
      font-style: italic;
      font-size: 80px;
      text-decoration: underline;
      background-color: green;
    }
```

当然你可以自己定义名称，然后在style中去定义,例如定义了“myactive”

style：

```css
    .router-link-active,
    .myactive {
      color: red;
      font-weight: 800;
      font-style: italic;
      font-size: 80px;
      text-decoration: underline;
      background-color: green;
    }
```

js:这里面的“linkActiveClass”就是自定义的高亮显示设置命名

```js
    var routerObj = new VueRouter({
      routes: [ 
        { path: '/', redirect: '/login' }, // 这里的 redirect 和 Node 中的 redirect 完全是两码事
        { path: '/login', component: login },
        { path: '/register', component: register }
      ],
      linkActiveClass: 'myactive'
    })
```

## 路由切换动画

和组件的动画一样，在外层套上一个<transition>

```html
   <transition mode="out-in">
      <router-view></router-view>
    </transition>
```

在style里面设置好

```css
    .v-enter,
    .v-leave-to {
      opacity: 0;
      transform: translateX(140px);
    }

    .v-enter-active,
    .v-leave-active {
      transition: all 0.5s ease;
    }
```

## 路由传参

### query方式

- 如果在路由中，使用 查询字符串，给路由传递参数，则 不需要修改 路由规则的 path 属性 

```php+HTML
    <router-link to="/login?id=10&name=zs">登录</router-link>
```

- $route.query."变量名"就可以访问

```js
    var login = {
      template: '<h1>登录 --- {{ $route.query.id }} --- {{ $route.query.name }}</h1>',
      data(){
        return {
          msg: '123'
        }
      },
      created(){ // 组件的生命周期钩子函数
        // console.log(this.$route)
        // console.log(this.$route.query.id)
      }
    }
```

login这的path不用改变

```js
    var router = new VueRouter({
      routes: [
        { path: '/login', component: login },
        { path: '/register', component: register }
      ]
    })
```

### params方式

传参方式

```html
   <router-link to="/login/12/ls">登录</router-link>
```

获取方式没有区别

```js
    var login = {
      template: '<h1>登录 --- {{ $route.params.id }} --- {{ $route.params.name }}</h1>',
      data(){
        return {
          msg: '123'
        }
      },
      created(){ // 组件的生命周期钩子函数
        console.log(this.$route.params.id)
      }
    }
```

path需要匹配

```js
var router = new VueRouter({
  routes: [
    { path: '/login/:id/:name', component: login },
    { path: '/register', component: register }
  ]
})
```
## 路由嵌套

先定义好父组件account

```html
  <div id="app">

    <router-link to="/account">Account</router-link>

    <router-view></router-view>

  </div>

  <template id="tmpl">
    <div>
      <h1>这是 Account 组件</h1>

      <router-link to="/account/login">登录</router-link>
      <router-link to="/account/register">注册</router-link>

      <router-view></router-view>
    </div>
  </template>
```

定义父子组件的模板

```js
    var account = {
      template: '#tmpl'
    }

    var login = {
      template: '<h3>登录</h3>'
    }

    var register = {
      template: '<h3>注册</h3>'
    }
```

定义router

- 使用 children 属性，实现子路由，同时，子路由的 path 前面，不要带 / ，否则永远以根路径开始请求，这样不方便我们用户去理解URL地址
- 注意不要写成注释中的代码，子组件的注册必须写在父组件的里面。

```js
    var router = new VueRouter({
      routes: [
        {
          path: '/account',
          component: account,         
          children: [
            { path: 'login', component: login },
            { path: 'register', component: register }
          ]
        }
        // { path: '/account/login', component: login },
        // { path: '/account/register', component: register }
      ]
    })
```

## 实例：使用命名视图实现经典布局

```js
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
  <script src="./lib/vue-2.4.0.js"></script>
  <script src="./lib/vue-router-3.0.1.js"></script>
  <style>
    html,
    body {
      margin: 0;
      padding: 0;
    }

    .header {
      background-color: orange;
      height: 80px;
    }

    h1 {
      margin: 0;
      padding: 0;
      font-size: 16px;
    }

    .container {
      display: flex;
      height: 600px;
    }

    .left {
      background-color: lightgreen;
      flex: 2;
    }

    .main {
      background-color: lightpink;
      flex: 8;
    }
  </style>
</head>

<body>
  <div id="app">

    <router-view></router-view>
    <div class="container">
      <router-view name="left"></router-view>
      <router-view name="main"></router-view>
    </div>

  </div>

  <script>

    var header = {
      template: '<h1 class="header">Header头部区域</h1>'
    }

    var leftBox = {
      template: '<h1 class="left">Left侧边栏区域</h1>'
    }

    var mainBox = {
      template: '<h1 class="main">mainBox主体区域</h1>'
    }

    // 创建路由对象
    var router = new VueRouter({
      routes: [
        /* { path: '/', component: header },
        { path: '/left', component: leftBox },
        { path: '/main', component: mainBox } */


        {
          path: '/', components: {
            'default': header,
            'left': leftBox,
            'main': mainBox
          }
        }
      ]
    })

    // 创建 Vue 实例，得到 ViewModel
    var vm = new Vue({
      el: '#app',
      data: {},
      methods: {},
      router
    });
  </script>
</body>

</html>
```

效果图

![](https://github.com/xinyuan960205/pic_resource/raw/master/java%E5%AD%A6%E4%B9%A0/%E8%B7%AF%E7%94%B1%E6%95%88%E6%9E%9C%E5%9B%BE.PNG)



