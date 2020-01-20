---
title: vuex
date: 2019-01-16 11:02:32
tags:
- 前端
- vue
categories:
- 前端
- vue
---

# vuex

![](http://plgod9q1n.bkt.clouddn.com/images/vuex%E6%A6%82%E5%BF%B5%E5%9B%BE.png)

## 为什么要使用

### 概念

vuex 是 Vue 配套的 公共数据管理工具，它可以把一些共享的数据，保存到 vuex 中，方便 整个程序中的任何组件直接获取或修改我们的公共数据；

Vuex 是一个专为 Vue.js 应用程序开发的**状态管理模式**。

## vuex安装

首先在webpack创建的vue-cli工程下，通过npm命令行，安装vuex

```
npm install vuex --save
```

然后 , 在 `main.js` 中导入包 :

```javascript
import vuex from 'vuex'
Vue.use(vuex);
var store = new vuex.Store({//store对象
    state:{
        show:false
    }
})
```

再然后 , 在实例化 Vue对象时加入 store 对象 :

```javascript
new Vue({
  el: '#app',
  router,
  store,  //将vuex创建的store挂载到VM实例上，只要挂载到VM上，就可以任何组件使用store来存取数据
  template: '<App/>',
  components: { App }
})
```

完成上述的步骤，vue中类似于`$store.state.show` 的操作就可以使用了

## vuex核心概念

### store

#### 什么是store

每一个 Vuex 应用的核心就是 store（仓库）。“store”基本上就是一个容器，它包含着你的应用中大部分的**状态 (state)**。Vuex 和单纯的全局对象有以下两点不同：

1. Vuex 的状态存储是响应式的。当 Vue 组件从 store 中读取状态的时候，若 store 中的状态发生变化，那么相应的组件也会相应地得到高效更新。
2. 你不能直接改变 store 中的状态。改变 store 中的状态的唯一途径就是显式地**提交 (commit) mutation**。这样使得我们可以方便地跟踪每一个状态的变化，从而让我们能够实现一些工具帮助我们更好地了解我们的应用。

#### 创建store

官网上和教程上给的都是在main.js上，也可以创建store.js把下面的代码写在store.js里面，在main.js里面引用store.js（后面给出了具体的操作）

```js
// 如果在模块化构建系统中，请确保在开头调用了 Vue.use(Vuex)

const store = new Vuex.Store({
  state: {
    //把state想象成组件中的data，专门用来存储数据的
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  }
})

//这里的state就相当于data
//这里的mutations就相当于methods
```



前面为了方便 , 我们把 store 对象写在了 main.js 里面 , 但实际上为了便于日后的维护 , 我们分开写更好 , 我们在 `src` 目录下 , 新建一个 `store` 文件夹 , 然后在里面新建一个 `index.js` :

```javascript
import Vue from 'vue'
import vuex from 'vuex'
Vue.use(vuex);

export default new vuex.Store({
    state:{
        show:false
    }
})
```

那么相应的 , 在 main.js 里的代码应该改成 :

```javascript
//vuex
import store from './store'

new Vue({
  el: '#app',
  router,
  store,//使用store
  template: '<App/>',
  components: { App }
})
```

这样就把 store 分离出去了 , 那么还有一个问题是 : 这里 `$store.state.show` 无论哪个组件都可以使用 , 那组件多了之后 , 状态也多了 , 这么多状态都堆在 store 文件夹下的 `index.js` 不好维护怎么办 ?

我们可以使用 vuex 的 `modules` , 把 store 文件夹下的 `index.js` 改成 :

```javascript
import Vue from 'vue'
import vuex from 'vuex'
Vue.use(vuex);

import dialog_store from '../components/dialog_store.js';//引入某个store对象

export default new vuex.Store({
    modules: {
        dialog: dialog_store
    }
})
```

这里我们引用了一个 `dialog_store.js` , 在这个 js 文件里我们就可以单独写 dialog 组件的状态了 :

```javascript
export default {
    state:{
        show:false
    }
}
```

做出这样的修改之后 , 我们将之前我们使用的 `$store.state.show` 统统改为 `$store.state.dialog.show` 即可。

如果还有其他的组件需要使用 vuex , 就新建一个对应的状态文件 , 然后将他们加入 store 文件夹下的 index.js 文件中的 `modules` 中。

```javascript
modules: {
    dialog: dialog_store,
    other: other,//其他组件
}
```

### mutations

#### 访问store中的数据

如果在组件中想要访问store中的数据，只能通过`this.$store.state.***`来访问

#### 修改store中的数据

```javascript
this.$store.state.count++; //千万不要这么做，这不符合vuex的设计理念
```

注意，如果要操作store中的state值，只能通过调用mutations提供的方法，才能操作对应的数据，不推荐直接操作state中的数据。因为，万一数据出现了紊乱，不能快速定位到错误的原因。因为每个组件都有操作数据的方法。

如果组件要调用mutations中的方法，只能使用`this.$store.commit('方法名')`。

#### mutations方法的提交参数

```javascript
const store = new Vuex.Store({
  state: {
    //把state想象成组件中的data，专门用来存储数据的
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    },
     substract(state,c){

     }
  }
})
```

```javascript
<script>
export default{
data(){

},
methods:{
    remove(){
        //vue组件中调用mutations中的方法
        this.$store.commit("substract",3);
    }
}
}
</script>
```

mutations的函数参数列表中，最多支持两个参数，其中，参数1：是state状态；参数2：通过commit提交的参数

### getters包装数据

```javascript
const store = new Vuex.Store({
  state: {
    //把state想象成组件中的data，专门用来存储数据的
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    },
    substract(state,c){
       //方法内容
    }
    getters(){
      optCount: function(state){
      	return '当前最新的count值是：' + state.count
  	  }
  	}
  }
})
```

注意：这里的getters，只负责对外提供数据，不负责修改数据，如果想要修改state中的数据，应该去找mutations中的方法

getters中的方法和过滤器中额方法类似，因为过滤器和getters都没有修改数据，只是把原数据做了一层包装，提供给了调用者。

其次，getters也和computed比较像，只要state中的数据发生了变化，那么如果getters正好也引用了这个数据，那么，就会立刻触发getters的重新求值。

### actions

多个 `state` 的操作 , 使用 `mutations` 会来触发会比较好维护 , 那么需要执行多个 mutations 就需要用 `action` 了:

```javascript
export default {
    state:{//state
        show:false
    },
    mutations:{
        switch_dialog(state){//这里的state对应着上面这个state
            state.show = state.show?false:true;
            //你还可以在这里执行其他的操作改变state
        }
    },
    actions:{
        switch_dialog(context){//这里的context和我们使用的$store拥有相同的对象和方法
            context.commit('switch_dialog');
            //你还可以在这里触发其他的mutations方法
        },
    }
}
```

那么 , 在之前的父组件中 , 我们需要做修改 , 来触发 action 里的 switch_dialog 方法:

```javascript
<template>
  <div id="app">
    <a href="javascript:;" @click="$store.dispatch('switch_dialog')">点击</a>
    <t-dialog></t-dialog>
  </div>
</template>

<script>
import dialog from './components/dialog.vue'
export default {
  components:{
    "t-dialog":dialog
  }
}
</script>
```

使用 `$store.dispatch('switch_dialog')` 来触发 `action` 中的 `switch_dialog` 方法。

官方推荐 , 将异步操作放在 action 中。

### mapState、mapGetters、mapActions(未看)

很多时候 , `$store.state.dialog.show` 、`$store.dispatch('switch_dialog')` 这种写法又长又臭 , 很不方便 , 我们没使用 vuex 的时候 , 获取一个状态只需要 `this.show` , 执行一个方法只需要 `this.switch_dialog` 就行了 , 使用 vuex 使写法变复杂了 ?

使用 `mapState、mapGetters、mapActions` 就不会这么复杂了。

以 mapState 为例 :

```javascript
<template>
  <el-dialog :visible.sync="show"></el-dialog>
</template>

<script>
import {mapState} from 'vuex';
export default {
  computed:{

    //这里的三点叫做 : 扩展运算符
    ...mapState({
      show:state=>state.dialog.show
    }),
  }
}
</script>
```

相当于 :

```javascript
<template>
  <el-dialog :visible.sync="show"></el-dialog>
</template>

<script>
import {mapState} from 'vuex';
export default {
  computed:{
    show(){
        return this.$store.state.dialog.show;
    }
  }
}
</script>
```

mapGetters、mapActions 和 mapState 类似 , `mapGetters` 一般也写在 `computed` 中 , `mapActions` 一般写在 `methods`中。





