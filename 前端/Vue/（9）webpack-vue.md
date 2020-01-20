---
title: （9）webpack-vue
date: 2019-02-18 20:24:47
tags:
- 前端
- vue
categories:
- 前端
- vue
---

# 一、render方法渲染组件

## 1、在页面中渲染基本的组件

声明一个login组件：

```js
    var login = {
      template: '<h1>这是登录组件</h1>'
    }
```

在VM实例中渲染：

```js
    var vm = new Vue({
      el: '#app',
      data: {},
      methods: {},
      components: {
        login
      }
    });
```

## 2、在页面中使用render函数渲染组件

声明一个login组件：

```js
    var login = {
      template: '<h1>这是登录组件</h1>'
    }
```

在VM实例中渲染：

```js
    // 创建 Vue 实例，得到 ViewModel
    var vm = new Vue({
      el: '#app',
      data: {},
      methods: {},
      render: function (createElements) { // createElements 是一个 方法，调用它，能够把 指定的 组件模板，渲染为 html 结构
        return createElements(login)
        // 注意：这里 return 的结果，会 替换页面中 el 指定的那个 容器
      }
    });
```

# 二、webpack导入vue

## 1、webpack.config.js

```js
// 由于 webpack 是基于Node进行构建的，所有，webpack的配置文件中，任何合法的Node代码都是支持的
var path = require('path')
// 在内存中，根据指定的模板页面，生成一份内存中的首页，同时自动把打包好的bundle注入到页面底部
// 如果要配置插件，需要在导出的对象中，挂载一个 plugins 节点
var htmlWebpackPlugin = require('html-webpack-plugin')

// 当以命令行形式运行 webpack 或 webpack-dev-server 的时候，工具会发现，我们并没有提供 要打包 的文件的 入口 和 出口文件，此时，他会检查项目根目录中的配置文件，并读取这个文件，就拿到了导出的这个 配置对象，然后根据这个对象，进行打包构建
module.exports = {
  entry: path.join(__dirname, './src/main.js'), // 入口文件
  output: { // 指定输出选项
    path: path.join(__dirname, './dist'), // 输出路径
    filename: 'bundle.js' // 指定输出文件的名称
  },
  plugins: [ // 所有webpack  插件的配置节点
    new htmlWebpackPlugin({
      template: path.join(__dirname, './src/index.html'), // 指定模板文件路径
      filename: 'index.html' // 设置生成的内存页面的名称
    })
  ],
  module: { // 配置所有第三方loader 模块的
    rules: [ // 第三方模块的匹配规则
      { test: /\.css$/, use: ['style-loader', 'css-loader'] }, // 处理 CSS 文件的 loader
      { test: /\.less$/, use: ['style-loader', 'css-loader', 'less-loader'] }, // 处理 less 文件的 loader
      { test: /\.scss$/, use: ['style-loader', 'css-loader', 'sass-loader'] }, // 处理 scss 文件的 loader
      { test: /\.(jpg|png|gif|bmp|jpeg)$/, use: 'url-loader?limit=7631&name=[hash:8]-[name].[ext]' }, // 处理 图片路径的 loader
      // limit 给定的值，是图片的大小，单位是 byte， 如果我们引用的 图片，大于或等于给定的 limit值，则不会被转为base64格式的字符串， 如果 图片小于给定的 limit 值，则会被转为 base64的字符串
      { test: /\.(ttf|eot|svg|woff|woff2)$/, use: 'url-loader' }, // 处理 字体文件的 loader 
      { test: /\.js$/, use: 'babel-loader', exclude: /node_modules/ }, // 配置 Babel 来转换高级的ES语法
      { test: /\.vue$/, use: 'vue-loader' } // 处理 .vue 文件的 loader
    ]
  },
  resolve: {
    alias: { // 修改 Vue 被导入时候的包的路径
      // "vue$": "vue/dist/vue.js"
    }
  }
}
```

## 2、在 webpack 构建的项目中，使用 Vue 进行开发

### 在普通网页中如何使用vue（复习）： 

1. 使用 script 标签 ，引入 vue 的包

2. 在 index 页面中，创建 一个 id 为 app div 容器·

3. 通过 new Vue 得到一个 vm 的实例

### 在webpack 中尝试使用 Vue

1. 安装vue的包：  cnpm i vue -S

2. 由于 在 webpack 中，推荐使用 .vue 这个组件模板文件定义组件，所以，需要安装 能解析这种文件的 loader    cnpm i vue-loader vue-template-complier -D

3.  在 main.js 中，导入 vue 模块  import Vue from 'vue'

```js
import Vue from 'vue'
```

- 注意： 在 webpack 中， 使用 import Vue from 'vue' 导入的 Vue 构造函数，功能不完整，只提供了 runtime-only 的方式，并没有提供 像网页中那样的使用方式；

- import Vue from '../node_modules/vue/dist/vue.js'

   回顾 包的查找规则：

  1. 找 项目根目录中有没有 node_modules 的文件夹

  2. 在 node_modules 中 根据包名，找对应的 vue 文件夹

  3. 在 vue 文件夹中，找 一个叫做 package.json 的包配置文件

  4. 在 package.json 文件中，查找 一个 main 属性【main属性指定了这个包在被加载时候，的入口文件】

4. 定义一个 .vue 结尾的组件，其中，组件有三部分组成： template script style

5. 使用 import login from './login.vue' 导入这个组件

   ```js
   import login from './login.vue'
   ```

   例如：导入 login 组件

-  默认，webpack 无法打包 .vue 文件，需要安装 相关的loader： 

- cnpm i vue-loader vue-template-compiler -D

- 在配置文件中，新增loader哦配置项 { test:/\.vue$/, use: 'vue-loader' }

6.  创建 vm 的实例 var vm = new Vue({ el: '#app', render: c => c(login) })

7. 在页面中创建一个 id 为 app 的 div 元素，作为我们 vm 实例要控制的区域；

```js
var vm = new Vue({
  el: '#app',
  data: {
    msg: '123'
  },
  // components: {
  //   login
  // }
  /* render: function (createElements) { // 在 webpack 中，如果想要通过 vue， 把一个组件放到页面中去展示，vm 实例中的 render 函数可以实现
    return createElements(login)
  } */

  render: c => c(login)
})
```

main.js代码：

```js
import Vue from 'vue'
import login from './login.vue'
var vm = new Vue({
  el: '#app',
  data: {
    msg: '123'
  },
  render: c => c(login)
})
```

# 三、export default 和export

这是 Node 中向外暴露成员的形式：

module.exports = {}

在 ES6中，也通过 规范的形式，规定了 ES6 中如何 导入 和 导出 模块

ES6中导入模块，使用   import 模块名称 from '模块标识符'    import '表示路径'

在 ES6 中，使用 export default 和 export 向外暴露成员：

```js
export default info
```

注意： export default 向外暴露的成员，可以使用任意的变量来接收

注意： 在一个模块中，export default 只允许向外暴露1次

注意： 在一个模块中，可以同时使用 export default 和 export 向外暴露成员

```js
export var title = '小星星'
export var content = '哈哈哈'
```

注意： 使用 export 向外暴露的成员，只能使用 { } 的形式来接收，这种形式，叫做 【按需导出】

注意： export 可以向外暴露多个成员， 同时，如果某些成员，我们在 import 的时候，不需要，则可以 不在 {}  中定义

注意： 使用 export 导出的成员，必须严格按照 导出时候的名称，来使用  {}  按需接收；

注意： 使用 export 导出的成员，如果 就想 换个 名称来接收，可以使用 as 来起别名；

test.js中使用export和export default：

```js
var info = {
  name: 'zs',
  age: 20
}

export default info

export var title = '小星星'
export var content = '哈哈哈'
```

main.js：

```js
import m222, { title as title123, content } from './test.js'
console.log(m222)
console.log(title123 + ' --- ' + content)
```

# 四、webpack使用vue-router

## 使用vue-router

1. 导入 vue-router 包

```js
import VueRouter from 'vue-router'
```

2. 手动安装 VueRouter 

```js
Vue.use(VueRouter)
```

3. 创建路由对象

```js
var router = new VueRouter({
  routes: [
    // account  goodslist
    { path: '/account', component: account },
    { path: '/goodslist', component: goodslist }
  ]
})
```

4. 将路由对象挂载到 vm 上

main.js

```js
import Vue from 'vue'
// 1. 导入 vue-router 包
import VueRouter from 'vue-router'
// 2. 手动安装 VueRouter 
Vue.use(VueRouter)

// 导入 app 组件
import app from './App.vue'
// 导入 Account 组件
import account from './main/Account.vue'
import goodslist from './main/GoodsList.vue'

// 3. 创建路由对象
var router = new VueRouter({
  routes: [
    // account  goodslist
    { path: '/account', component: account },
    { path: '/goodslist', component: goodslist }
  ]
})

var vm = new Vue({
  el: '#app',
  render: c => c(app), // render 会把 el 指定的容器中，所有的内容都清空覆盖，所以 不要 把 路由的 router-view 和 router-link 直接写到 el 所控制的元素中
  router // 4. 将路由对象挂载到 vm 上
})

// 注意： App 这个组件，是通过 VM 实例的 render 函数，渲染出来的， render 函数如果要渲染 组件， 渲染出来的组件，只能放到 el: '#app' 所指定的 元素中；
// Account 和 GoodsList 组件， 是通过 路由匹配监听到的，所以， 这两个组件，只能展示到 属于 路由的 <router-view></router-view> 中去；
```

## 抽离路由模块

router.js

```js
import VueRouter from 'vue-router'

// 导入 Account 组件
import account from './main/Account.vue'
import goodslist from './main/GoodsList.vue'

// 导入Account的两个子组件
import login from './subcom/login.vue'
import register from './subcom/register.vue'

// 3. 创建路由对象
var router = new VueRouter({
  routes: [
    // account  goodslist
    {
      path: '/account',
      component: account,
      children: [
        { path: 'login', component: login },
        { path: 'register', component: register }
      ]
    },
    { path: '/goodslist', component: goodslist }
  ]
})

// 把路由对象暴露出去
export default router
```

main.js

```js
import Vue from 'vue'
// 1. 导入 vue-router 包
import VueRouter from 'vue-router'
// 2. 手动安装 VueRouter 
Vue.use(VueRouter)

// 导入 app 组件
import app from './App.vue'

// 导入 自定义路由模块
import router from './router.js'

var vm = new Vue({
  el: '#app',
  render: c => c(app), // render 会把 el 指定的容器中，所有的内容都清空覆盖，所以 不要 把 路由的 router-view 和 router-link 直接写到 el 所控制的元素中
  router // 4. 将路由对象挂载到 vm 上
})

// 注意： App 这个组件，是通过 VM 实例的 render 函数，渲染出来的， render 函数如果要渲染 组件， 渲染出来的组件，只能放到 el: '#app' 所指定的 元素中；
// Account 和 GoodsList 组件， 是通过 路由匹配监听到的，所以， 这两个组件，只能展示到 属于 路由的 <router-view></router-view> 中去；
```

# 五、style标签lang和scope

- 普通的 style 标签只支持 普通的 样式，如果想要启用 scss 或 less ，需要为 style 元素，设置 lang 属性

- 只要 咱们的 style 标签， 是在 .vue 组件中定义的，那么，推荐都为 style 开启 scoped 属性

```css
<style lang="scss" scoped>
body {
  div {
    font-style: italic;
  }
}
</style>
```



























