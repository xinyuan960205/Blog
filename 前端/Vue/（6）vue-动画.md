---
title: （6）vue 动画&过渡
date: 2019-01-31 12:07:25
tags:
- 前端
- vue
categories:
- 前端
- vue
---

# vue动画&过渡

## 1、使用过渡类名

### 单元素/组件的过渡

1. 使用 transition 元素，把 需要被动画控制的元素，包裹起来
2. 自定义两组样式，来控制 transition 内部的元素实现动画

示例：

点击按钮，文字从右侧150px的位置向左侧移动出现，期间历时0.8s

HTML结构：

```html
<div id="app">
    <input type="button" value="toggle" @click="flag=!flag">
    <!-- 需求： 点击按钮，让 h3 显示，再点击，让 h3 隐藏 -->
    <!-- 1. 使用 transition 元素，把 需要被动画控制的元素，包裹起来 -->
    <!-- transition 元素，是 Vue 官方提供的 -->
    <transition>
      <h3 v-if="flag">这是一个H3</h3>
    </transition>
</div>
```

VM 实例：

```html
<script>
    // 创建 Vue 实例，得到 ViewModel
    var vm = new Vue({
        el: '#app',
        data: {
            flag: false
        },
        methods: {}
    });
</script>
```

css样式：

```html
<!-- 2. 自定义两组样式，来控制 transition 内部的元素实现动画 -->
<style>
    /* v-enter 【这是一个时间点】 是进入之前，元素的起始状态，此时还没有开始进入 */
    /* v-leave-to 【这是一个时间点】 是动画离开之后，离开的终止状态，此时，元素 动画已经结束了 */
    .v-enter,
    .v-leave-to {
        opacity: 0;
        transform: translateX(150px);
    }

    /* v-enter-active 【入场动画的时间段】 */
    /* v-leave-active 【离场动画的时间段】 */
    .v-enter-active,
    .v-leave-active{
        transition: all 0.8s ease;
    }
</style>
```

### 过渡的类名

在进入/离开的过渡中，会有 6 个 class 切换。

1. `v-enter`：定义进入过渡的开始状态。在元素被插入之前生效，在元素被插入之后的下一帧移除。
2. `v-enter-active`：定义进入过渡生效时的状态。在整个进入过渡的阶段中应用，在元素被插入之前生效，在过渡/动画完成之后移除。这个类可以被用来定义进入过渡的过程时间，延迟和曲线函数。
3. `v-enter-to`: **2.1.8版及以上** 定义进入过渡的结束状态。在元素被插入之后下一帧生效 (与此同时 `v-enter` 被移除)，在过渡/动画完成之后移除。
4. `v-leave`: 定义离开过渡的开始状态。在离开过渡被触发时立刻生效，下一帧被移除。
5. `v-leave-active`：定义离开过渡生效时的状态。在整个离开过渡的阶段中应用，在离开过渡被触发时立刻生效，在过渡/动画完成之后移除。这个类可以被用来定义离开过渡的过程时间，延迟和曲线函数。
6. `v-leave-to`: **2.1.8版及以上** 定义离开过渡的结束状态。在离开过渡被触发之后下一帧生效 (与此同时 `v-leave` 被删除)，在过渡/动画完成之后移除。

![Transition Diagram](https://cn.vuejs.org/images/transition.png)

### 自定义v-前缀

​	对于这些在过渡中切换的类名来说，如果你使用一个没有名字的 `<transition>`，则 `v-` 是这些类名的默认前缀。如果你使用了 `<transition name="my-transition">`，那么 `v-enter` 会替换为 `my-transition-enter`。

`v-enter-active` 和 `v-leave-active` 可以控制进入/离开过渡的不同的缓和曲线

示例：

在<transition>添加属性`name="my"`，css样式中将`v-`换成`my`

HTML结构

```html
<input type="button" value="toggle2" @click="flag2=!flag2">
<transition name="my">
    <h6 v-if="flag2">这是一个H6</h6>
</transition>
```

CSS

```css
.my-enter,
.my-leave-to {
   opacity: 0;
   transform: translateY(70px);
}

.my-enter-active,
.my-leave-active{
    transition: all 0.8s ease;
}
```

## 2、使用第三方类库

导入动画类库：

```html
  <link rel="stylesheet" href="./lib/animate.css">
```

html结构有两种写法

第一种：

注意：`enter-active-class`和`leave-active-class`两个属性都要加上`animated`

```html
<!-- 入场 bounceIn    离场 bounceOut -->
<input type="button" value="toggle" @click="flag=!flag">
<!-- 需求： 点击按钮，让 h3 显示，再点击，让 h3 隐藏 -->
<transition 
 enter-active-class="animated bounceIn" leave-active-class="animated bounceOut">
    <h3 v-if="flag">这是一个H3</h3>
</transition> 
```

第二种：

因为第一种办法设置属性比较麻烦，每个都要添加`animated`，所以第二种可以放到<h3>的`class`里面

```html
<!-- 使用  :duration="{ enter: 200, leave: 400 }"  来分别设置 入场的时长 和 离场的时长  -->
<transition 
    enter-active-class="bounceIn" 
    leave-active-class="bounceOut" 
    :duration="{ enter: 200, leave: 400 }">
    <h3 v-if="flag" class="animated">这是一个H3</h3>
</transition> 
```

## 3、钩子函数

可以在属性中声明 JavaScript 钩子

```js
<transition
  v-on:before-enter="beforeEnter"
  v-on:enter="enter"
  v-on:after-enter="afterEnter"
  v-on:enter-cancelled="enterCancelled"

  v-on:before-leave="beforeLeave"
  v-on:leave="leave"
  v-on:after-leave="afterLeave"
  v-on:leave-cancelled="leaveCancelled"
>
  <!-- ... -->
</transition>
// ...
methods: {
  // --------
  // 进入中
  // --------

  beforeEnter: function (el) {
    // ...
  },
  // 当与 CSS 结合使用时
  // 回调函数 done 是可选的
  enter: function (el, done) {
    // ...
    done()
  },
  afterEnter: function (el) {
    // ...
  },
  enterCancelled: function (el) {
    // ...
  },

  // --------
  // 离开时
  // --------

  beforeLeave: function (el) {
    // ...
  },
  // 当与 CSS 结合使用时
  // 回调函数 done 是可选的
  leave: function (el, done) {
    // ...
    done()
  },
  afterLeave: function (el) {
    // ...
  },
  // leaveCancelled 只用于 v-show 中
  leaveCancelled: function (el) {
    // ...
  }
}
```

这些钩子函数可以结合 CSS `transitions/animations` 使用，也可以单独使用。

当只用 JavaScript 过渡的时候，**在 enter 和 leave 中必须使用 done 进行回调**。否则，它们将被同步调用，过渡会立即完成。

推荐对于仅使用 JavaScript 过渡的元素添加 `v-bind:css="false"`，Vue 会跳过 CSS 的检测。这也可以避免过渡过程中 CSS 的影响。

示例：

定义 transition 组件以及三个钩子函数：

```js
<input type="button" value="快到碗里来" @click="flag=!flag">
<!-- 1. 使用 transition 元素把 小球包裹起来 -->
<transition
    @before-enter="beforeEnter"
    @enter="enter"
    @after-enter="afterEnter">
    <div class="ball" v-show="flag"></div>
</transition>
```

定义方法：

```js
methods: {
    // 注意： 动画钩子函数的第一个参数：el，表示 要执行动画的那个DOM元素，是个原生的 JS DOM对象
    // 大家可以认为 ， el 是通过 document.getElementById('') 方式获取到的原生JS DOM对象
    beforeEnter(el){
       // beforeEnter 表示动画入场之前，此时，动画尚未开始，可以 在 beforeEnter 中，设置元素开始动画之前的起始样式
       // 设置小球开始动画之前的，起始位置
       el.style.transform = "translate(0, 0)"
    },
    enter(el, done){
       // 这句话，没有实际的作用，但是，如果不写，出不来动画效果；
       // 可以认为 el.offsetWidth 会强制动画刷新
       el.offsetWidth
       // enter 表示动画 开始之后的样式，这里，可以设置小球完成动画之后的，结束状态
       el.style.transform = "translate(150px, 450px)"
       el.style.transition = 'all 1s ease'

       // 这里的 done， 起始就是 afterEnter 这个函数，也就是说：done 是 afterEnter 函数的引用
       done()
    },
    afterEnter(el){
       // 动画完成之后，会调用 afterEnter
       // console.log('ok')
       this.flag = !this.flag
    }
}
```

css样式：

```css
    .ball {
      width: 15px;
      height: 15px;
      border-radius: 50%;
      background-color: red;
    }
```

## 4、使用transition-group函数

HTML结构

```html
<div id="app">

<div>
    <label>
        Id:
        <input type="text" v-model="id">
    </label>

    <label>
        Name:
        <input type="text" v-model="name">
    </label>

    <input type="button" value="添加" @click="add">
</div>

    <!-- <ul> -->
    <!-- 在实现列表过渡的时候，如果需要过渡的元素，是通过 v-for 循环渲染出来的，不能使用 transition 包裹，需要使用 transitionGroup -->
    <!-- 如果要为 v-for 循环创建的元素设置动画，必须为每一个 元素 设置 :key 属性 -->
    <!-- 给 ransition-group 添加 appear 属性，实现页面刚展示出来时候，入场时候的效果 -->
    <!-- 通过 为 transition-group 元素，设置 tag 属性，指定 transition-group 渲染为指定的元素，如果不指定 tag 属性，默认，渲染为 span 标签 -->
    <transition-group appear tag="ul">
        <li v-for="(item, i) in list" :key="item.id" @click="del(i)">
            {{item.id}} --- {{item.name}}
        </li>
    </transition-group>
    <!-- </ul> -->

</div>
```

vm结构

```js
// 创建 Vue 实例，得到 ViewModel
var vm = new Vue({
    el: '#app',
    data: {
        id: '',
        name: '',
        list: [
            { id: 1, name: '赵高' },
            { id: 2, name: '秦桧' },
            { id: 3, name: '严嵩' },
            { id: 4, name: '魏忠贤' }
        ]
    },
    methods: {
        add() {
            this.list.push({ id: this.id, name: this.name })
            this.id = this.name = ''
        },
        del(i) {
            this.list.splice(i, 1)
        }
    }
});
```

css结构

```css
li {
    border: 1px dashed #999;
    margin: 5px;
    line-height: 35px;
    padding-left: 5px;
    font-size: 12px;
    width: 100%;
}

li:hover {
    background-color: hotpink;
    transition: all 0.8s ease;
}



.v-enter,
.v-leave-to {
    opacity: 0;
    transform: translateY(80px);
}

.v-enter-active,
.v-leave-active {
    transition: all 0.6s ease;
}

/* 下面的 .v-move 和 .v-leave-active 配合使用，能够实现列表后续的元素，渐渐地漂上来的效果 */
.v-move {
    transition: all 0.6s ease;
}
.v-leave-active{
    position: absolute;
}
```







