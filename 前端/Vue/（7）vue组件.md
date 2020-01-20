---
title: （7）vue组件
date: 2019-02-01 10:13:06
tags:
- 前端
- vue
categories:
- 前端
- vue
---

# vue组件

## 组件概念

什么是组件： 组件的出现，就是为了拆分Vue实例的代码量的，能够让我们以不同的组件，来划分不同的功能模块，将来我们需要什么样的功能，就可以去调用对应的组件即可；
组件化和模块化的不同：

- 模块化： 是从代码逻辑的角度进行划分的；方便代码分层开发，保证每个功能模块的职能单一；
- 组件化： 是从UI界面的角度进行划分的；前端的组件化，方便UI组件的重用；

## 全局vue组件定义

### 1、使用 Vue.extend 配合 Vue.component 

第一步，使用 Vue.extend 来创建全局的Vue组件

```js
var com1 = Vue.extend({
   // 通过 template 属性，指定了组件要展示的HTML结构 
   template: '<h3>这是使用 Vue.extend 创建的组件</h3>' 
})
```

第二步，使用 Vue.component（'组件的名称', 创建出来的组件模板对象）

> 如果使用 Vue.component 定义全局组件的时候，组件名称使用了 驼峰命名，则在引用组件的时候，需要把 大写的驼峰改为小写的字母，同时，两个单词之前，使用 - 链接；

script部分：

```js
Vue.component('myCom1', com1)
```

html部分：

```html
<my-com1></my-com1>
```

> 如果不使用驼峰,则直接拿名称来使用即可;

script部分：

```js
Vue.component('mycom1', com1)
```

html部分：

```html
<mycom1></mycom1>
```

### 2、直接使用 Vue.component 

相较于第一种方法，将Vue.extend 构建template的步骤直接放到Vue.component里面，更为简单

```js
// Vue.component 第一个参数:组件的名称,将来在引用组件的时候,就是一个 标签形式 来引入 它的
// 第二个参数: Vue.extend 创建的组件  ,其中 template 就是组件将来要展示的HTML内容
Vue.component('mycom1', Vue.extend({
    template: '<h3>这是使用 Vue.extend 创建的组件</h3>'
}))
```

其实还可以更简单一点，

```js
// 注意:不论是哪种方式创建出来的组件,组件的 template 属性指向的模板内容,必须有且只能有唯一的一个根元素
Vue.component('mycom2', {
    template: '<div><h3>这是直接使用 Vue.component 创建出来的组件</h3><span>123</span>					</div>'
})
```

> 但是这种方法有一个问题就是，写在template中的HTML代码IDE没有办法进行辅助提示、排版等功能，影响工作效率。

### 3、将模板字符串，定义到script标签种

先在<template>里面定义好组件，设置好id；然后在`Vue.component`中引用该<template>

html部分：

```html
<div id="app">
    <mycom3></mycom3>
</div>
<!-- 在 被控制的 #app 外面,使用 template 元素,定义组件的HTML模板结构  -->
<template id="tmpl">
    <div>
        <h1>这是通过 template 元素,在外部定义的组件结构,这个方式,有代码的只能提示和高亮</h1>
        <h4>好用,不错!</h4>
    </div>
</template>
```

> 注意： 组件中的DOM结构，有且只能有唯一的根元素（Root Element）来进行包裹！

script部分：

```js
Vue.component('mycom3', {
    template: '#tmpl'
})
```

> 第三种方法最好，推荐使用。

## 私有组件定义

定义私有的组件，就需要在VM实例中的components中去定义。

vm实例：

```js
var vm2 = new Vue({
    el: '#app2',
    data: {},
    methods: {},
    filters: {},
    directives: {},
    components: { // 定义实例内部私有组件的
        login: {
            template: '#tmpl2'
        }
    },

    beforeCreate() { },
    created() { },
    beforeMount() { },
    mounted() { },
    beforeUpdate() { },
    updated() { },
    beforeDestroy() { },
    destroyed() { }
})
```

HTML：

```html
<div id="app2">
    <login></login>
</div>
<template id="tmpl2">
    <h1>这是私有的 login 组件</h1>
</template>
```

## 组件中的data

1. 组件可以有自己的 data 数据

2. 组件的 data 和 实例的 data 有点不一样,实例中的 data 可以为一个对象,但是 组件中的 data 必须是一个方法

3. 组件中的 data 除了必须为一个方法之外,这个方法内部,还必须返回一个对象才行;

4. 组件中 的data 数据,使用方式,和实例中的 data 使用方式完全一样!!!

```js
Vue.component('mycom1', {
    template: '<h1>这是全局组件 --- {{msg}}</h1>',
    data: function () {
        return {
            msg: '这是组件的中data定义的数据'
        }
    }
})
```

## 组件切换

### 1、v-if和v-else结合flag

示例是有两个链接登录和注册，点击不同的链接显示不同的组件。

HTML：

```html
<a href="" @click.prevent="flag=true">登录</a>
<a href="" @click.prevent="flag=false">注册</a>

<login v-if="flag"></login>
<register v-else="flag"></register>
```

script：

```js
Vue.component('login', {
    template: '<h3>登录组件</h3>'
})

Vue.component('register', {
    template: '<h3>注册组件</h3>'
})

// 创建 Vue 实例，得到 ViewModel
var vm = new Vue({
    el: '#app',
    data: {
        flag: false
    },
    methods: {}
});
```

### 2、vue提供component元素提供组件的切换

通过设置<component>里面的is属性值为组件名称，起到切换组件的作用。

HTML部分：

```html
<a href="" @click.prevent="comName='login'">登录</a>
<a href="" @click.prevent="comName='register'">注册</a>

<!-- Vue提供了 component ,来展示对应名称的组件 -->
<!-- component 是一个占位符, :is 属性,可以用来指定要展示的组件的名称 -->
<component :is="comName"></component>
```

script部分：

```js
// 组件名称是 字符串
Vue.component('login', {
    template: '<h3>登录组件</h3>'
})

Vue.component('register', {
    template: '<h3>注册组件</h3>'
})

// 创建 Vue 实例，得到 ViewModel
var vm = new Vue({
    el: '#app',
    data: {
        comName: 'login' // 当前 component 中的 :is 绑定的组件的名称
    },
    methods: {}
});
```

### 3、添加动画

实际上就是<component>套上<transition>，在css中设置动画效果

HTML：

```html
<!-- 通过 mode 属性,设置组件切换时候的 模式 -->
<transition mode="out-in">
    <component :is="comName"></component>
</transition>
```

CSS：

```css
<style>
.v-enter,
.v-leave-to {
    opacity: 0;
    transform: translateX(150px);
}

.v-enter-active,
.v-leave-active {
    transition: all 0.5s ease;
}
</style>
```

## 组件传值

### 父组件向子组件传值

> 子组件中，默认无法访问到 父组件中的 data 上的数据 和 methods 中的方法

1. 父组件，可以在引用子组件的时候， 通过 属性绑定（v-bind:） 的形式, 把 需要传递给 子组件的数据，以属性绑定的形式，传递到子组件内部，供子组件使用。
2. 把父组件传递过来的 parentmsg 属性，先在 props 数组中，定义一下，这样，才能使用这个数据

HTML：

```html
//parentmsg是需要在props中定义的，msg是父组件的变量  
<com1 v-bind:parentmsg="msg"></com1>
```

VM：

```js
    var vm = new Vue({
      el: '#app',
      data: {
        msg: '123 啊-父组件中的数据'
      },
      methods: {},

      components: {
        com1: {
          data() {           
            return {
              title: '123',
              content: 'qqq'
            }
          },
          template: '<h1 @click="change">这是子组件 --- {{ parentmsg }}</h1>',
          props: ['parentmsg'], 
          directives: {},
          filters: {},
          components: {},
          methods: {
            change() {
              this.parentmsg = '被修改了'
            }
          }
        }
      }
    });
```

### data和props的区别

子组件中的 data 数据，并不是通过 父组件传递过来的，而是子组件自身私有的，比如： 子组件通过 Ajax ，请求回来的数据，都可以放到 data 身上；

data 上的数据，都是可读可写的；

组件中的 所有 props 中的数据，都是通过 父组件传递给子组件的

props 中的数据，都是只读的，无法重新赋值

### 子组件向父组件传值

1. 原理：父组件将方法的引用，传递到子组件内部，子组件在内部调用父组件传递过来的方法，同时把要发送给父组件的数据，在调用方法的时候当作参数传递进去；
2. 父组件将方法的引用传递给子组件，其中，`getMsg`是父组件中`methods`中定义的方法名称，`func`是子组件调用传递过来方法时候的方法名称

```html
    <com2 @func="show"></com2>
```

3. 子组件内部通过`this.$emit('方法名', 要传递的数据)`方式，来调用父组件中的方法，同时把数据传递给父组件使用

```js
 this.$emit('func', this.sonmsg)
```

VM：

```js
   // 定义了一个字面量类型的 组件模板对象
    var com2 = {
      template: '#tmpl', // 通过指定了一个 Id, 表示 说，要去加载 这个指定Id的 template 元素中的内容，当作 组件的HTML结构
      data() {
        return {
          sonmsg: { name: '小头儿子', age: 6 }
        }
      },
      methods: {
        myclick() {
          // 当点击子组件的按钮的时候，如何 拿到 父组件传递过来的 func 方法，并调用这个方法？？？
          //  emit 英文原意： 是触发，调用、发射的意思
          // this.$emit('func123', 123, 456)
          this.$emit('func', this.sonmsg)
        }
      }
    }


    // 创建 Vue 实例，得到 ViewModel
    var vm = new Vue({
      el: '#app',
      data: {
        datamsgFormSon: null
      },
      methods: {
        show(data) {
          // console.log('调用了父组件身上的 show 方法: --- ' + data)
          // console.log(data);
          this.datamsgFormSon = data
        }
      },

      components: {
        com2
        // com2: com2
      }
    });
```

## 组件案例：评论发表&显示

#### 使用localStorage 存储数据

分析：发表评论的业务逻辑

             1. 评论数据存到哪里去？？？   存放到了 localStorage 中  localStorage.setItem('cmts', '')
             出一个最新的评论数据对象
             2. 先组织出一个最新的评论数据对象
             3. 想办法，把 第二步中，得到的评论对象，保存到 localStorage 中：
             3.1 localStorage 只支持存放字符串数据， 要先调用 JSON.stringify 
             3.2 在保存 最新的 评论数据之前，要先从 localStorage 获取到之前的评论数据（string）， 转换为 一个  数组对象， 然后，把最新的评论， push 到这个数组
             3.3 如果获取到的 localStorage 中的 评论字符串，为空不存在， 则  可以 返回一个 '[]'  让 JSON.parse 去转换
             3.4  把 最新的  评论列表数组，再次调用 JSON.stringify 转为  数组字符串，然后调用    localStorage.setItem()

```js
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
  <script src="./lib/vue-2.4.0.js"></script>
  <link rel="stylesheet" href="./lib/bootstrap-3.3.7.css">
</head>

<body>
  <div id="app">
    <cmt-box @func="loadComments"></cmt-box>
    <ul class="list-group">
      <li class="list-group-item" v-for="item in list" :key="item.id">
        <span class="badge">评论人： {{ item.user }}</span>
        {{ item.content }}
      </li>
    </ul>
  </div>


  <template id="tmpl">
    <div>

      <div class="form-group">
        <label>评论人：</label>
        <input type="text" class="form-control" v-model="user">
      </div>

      <div class="form-group">
        <label>评论内容：</label>
        <textarea class="form-control" v-model="content"></textarea>
      </div>

      <div class="form-group">
        <input type="button" value="发表评论" class="btn btn-primary" @click="postComment">
      </div>

    </div>
  </template>

  <script>

    var commentBox = {
      data() {
        return {
          user: '',
          content: ''
        }
      },
      template: '#tmpl',
      methods: {
        postComment() { // 发表评论的方法         
          var comment = { id: Date.now(), user: this.user, content: this.content }

          // 从 localStorage 中获取所有的评论
          var list = JSON.parse(localStorage.getItem('cmts') || '[]')
          list.unshift(comment)
          // 重新保存最新的 评论数据
          localStorage.setItem('cmts', JSON.stringify(list))

          this.user = this.content = ''

          // this.loadComments() // ?????
          this.$emit('func')
        }
      }
    }

    // 创建 Vue 实例，得到 ViewModel
    var vm = new Vue({
      el: '#app',
      data: {
        list: [
          { id: Date.now(), user: '李白', content: '天生我材必有用' },
          { id: Date.now(), user: '江小白', content: '劝君更尽一杯酒' },
          { id: Date.now(), user: '小马', content: '我姓马， 风吹草低见牛羊的马' }
        ]
      },
      beforeCreate(){ // 注意：这里不能调用 loadComments 方法，因为在执行这个钩子函数的时候，data 和 methods 都还没有被初始化好

      },
      created(){
        this.loadComments()
      },
      methods: {
        loadComments() { // 从本地的 localStorage 中，加载评论列表
          var list = JSON.parse(localStorage.getItem('cmts') || '[]')
          this.list = list
        }
      },
      components: {
        'cmt-box': commentBox
      }
    });
  </script>
</body>

</html>
```

结果展示：
![](https://github.com/xinyuan960205/pic_resource/raw/master/vue%E5%AD%A6%E4%B9%A0/%E8%AF%84%E8%AE%BA%E5%8F%91%E8%A1%A8%E6%98%BE%E7%A4%BA%E6%95%88%E6%9E%9C.PNG)

## ref获取dom元素和组件

 `this.$refs."组件名称".方法/变量`

HTML：

```html
    <h3 id="myh3" ref="myh3">哈哈哈， 今天天气太好了！！！</h3>

    <login ref="mylogin"></login>
```

script：

```js
    var vm = new Vue({
      el: '#app',
      data: {},
      methods: {
        getElement() {
          // console.log(document.getElementById('myh3').innerText)

          //  ref  是 英文单词 【reference】   值类型 和 引用类型  referenceError
           console.log(this.$refs.myh3.innerText)
			
           console.log(this.$refs.mylogin.msg)
           this.$refs.mylogin.show()
        }
      },
      components: {
        login
      }
    });
```

























