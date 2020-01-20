---
title: （3）vue自定义按键、指令
date: 2019-01-15 11:02:32
tags:
- 前端
- vue
categories:
- 前端
- vue
---
# vue自定义按键

## vue自定义按键

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
  <script src="./lib/vue-2.4.0.js"></script>
  <link rel="stylesheet" href="./lib/bootstrap-3.3.7.css">
  <!-- 需要用到Jquery吗？？？ -->
</head>

<body>
  <div id="app">
    <!-- {{1+1}} -->
    <div class="panel panel-primary">
      <div class="panel-heading">
        <h3 class="panel-title">添加品牌</h3>
      </div>
      <div class="panel-body form-inline">
        <label>
          Name:
          //按enter实现
          <input type="text" class="form-control" v-model="name" @keyup.enter="add">
          //113是F2的码，所以按F2实现    
          <input type="text" class="form-control" v-model="name" @keyup.113="add">    
        </label>
      </div>
    </div>
  </div>

  <script>
    // 自定义全局按键修饰符
    Vue.config.keyCodes.f2 = 113
  </script>
</body>
</html>
```

## 自定义全局指令——让文本框获取焦点

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
  <script src="./lib/vue-2.4.0.js"></script>
  <link rel="stylesheet" href="./lib/bootstrap-3.3.7.css">
  <!-- 需要用到Jquery吗？？？ -->
</head>

<body>
  <div id="app">
    <!-- {{1+1}} -->


    <div class="panel panel-primary">
      <div class="panel-heading">
        <h3 class="panel-title">添加品牌</h3>
      </div>
      <div class="panel-body form-inline">
        <label>
          搜索名称关键字：
          <!-- 注意： Vue中所有的指令，在调用的时候，都以 v- 开头 -->
          <input type="text" class="form-control" v-model="keywords" id="search" v-focus v-color="'green'">
        </label>
      </div>

  <script>
    // 使用  Vue.directive() 定义全局的指令  v-focus
    // 其中：参数1 ： 指令的名称，注意，在定义的时候，指令的名称前面，不需要加 v- 前缀, 
    // 但是： 在调用的时候，必须 在指令名称前 加上 v- 前缀来进行调用
    //  参数2： 是一个对象，这个对象身上，有一些指令相关的函数，这些函数可以在特定的阶段，执行相关的操作
    Vue.directive('focus', {
      bind: function (el) { // 每当指令绑定到元素上的时候，会立即执行这个 bind 函数，只执行一次
        // 注意： 在每个 函数中，第一个参数，永远是 el ，表示 被绑定了指令的那个元素，这个 el 参数，是一个原生的JS对象
        // 在元素 刚绑定了指令的时候，还没有 插入到 DOM中去，这时候，调用 focus 方法没有作用
        //  因为，一个元素，只有插入DOM之后，才能获取焦点
        // el.focus()
      },
      inserted: function (el) {  // inserted 表示元素 插入到DOM中的时候，会执行 inserted 函数【触发1次】
        el.focus()
        // 和JS行为有关的操作，最好在 inserted 中去执行，放置 JS行为不生效
      },
      updated: function (el) {  // 当VNode更新的时候，会执行 updated， 可能会触发多次

      }
    })
      
  </script>
</body>
</html>
```

## 使用钩子函数的第二个binding获得参数值

在html中的输入框中，设置v-color

在script中，自定义的bind函数的第二个参数binding获取值“green”

```html
        <label>
          搜索名称关键字：
          <!-- 注意： Vue中所有的指令，在调用的时候，都以 v- 开头 -->
          <input type="text" class="form-control" v-model="keywords" id="search" v-focus v-color="'green'">
        </label>
```

```html
<script>
	// 自定义一个 设置字体颜色的 指令
    Vue.directive('color', {
      // 样式，只要通过指令绑定给了元素，不管这个元素有没有被插入到页面中去，这个元素肯定有了一个内联的样式
      // 将来元素肯定会显示到页面中，这时候，浏览器的渲染引擎必然会解析样式，应用给这个元素
      bind: function (el, binding) {
        // el.style.color = 'red'
        // console.log(binding.name)
        // 和样式相关的操作，一般都可以在 bind 执行

        // console.log(binding.value)
        // console.log(binding.expression)

        el.style.color = binding.value
      }
    })
</script>  
```

## 定义私有指令



```html
 <div id="app2">
    <h3 v-color="'pink'" v-fontweight="900" v-fontsize="50">{{ dt | dateFormat }}</h3>
  </div>
```



```html
<script>
// 如何自定义一个私有的过滤器（局部）
    var vm2 = new Vue({
      el: '#app2',
      data: {
        dt: new Date()
      },
      methods: {},
      filters: {},
      directives: { // 自定义私有指令
        'fontweight': { // 设置字体粗细的
          bind: function (el, binding) {
            el.style.fontWeight = binding.value
          }
        }
      }
    })
</script>
```

## 指令的简写方式



```html
 <div id="app2">
    <h3 v-color="'pink'" v-fontweight="900" v-fontsize="50">{{ dt | dateFormat }}</h3>
  </div>
```



```html
<script>
// 如何自定义一个私有的过滤器（局部）
    var vm2 = new Vue({
      el: '#app2',
      data: {
        dt: new Date()
      },
      methods: {},
      filters: {},
      directives: { // 自定义私有指令
        'fontsize': function (el, binding) { // 注意：这个 function 等同于 把 代码写到了 bind 和 update 中去
          el.style.fontSize = parseInt(binding.value) + 'px'
        }
      }
    })
</script>
```

























