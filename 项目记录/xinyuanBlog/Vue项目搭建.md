# Vue项目搭建



## 依赖环境

1. node：点击[node](https://nodejs.org/zh-cn/)访问官方下载安装

   先安装node，具体查看安装node的教程，我安装的版本是v10.14.0

   > Node 版本要求
   >
   > Vue CLI 需要 [Node.js](https://nodejs.org/) 8.9 或更高版本 (推荐 8.11.0+)。你可以使用 [nvm](https://github.com/creationix/nvm) 或 [nvm-windows](https://github.com/coreybutler/nvm-windows) 在同一台电脑中管理多个 Node 版本。

2. vue-cli：`npm install -g vue-cli`

之前的vue-cli版本是2.X，所以进行了升级。

```bash
  npm uninstall -g vue-cli
  npm install -g @vue/cli
```

升级后的版本`@vue/cli 4.1.2`

安装之后，你就可以在命令行中访问 `vue` 命令。你可以通过简单运行 `vue`，看看是否展示出了一份所有可用命令的帮助信息，来验证它是否安装成功。

你还可以用这个命令来检查其版本是否正确：

```bash
vue --version
```

## 使用脚手架创建项目

这里采用UI的方式创建

```bash
vue ui
```

运行后浏览器会打开[localhost:8000](localhost:8000)，进入 项目管理器 > 创建 > 选择文件夹，点击在此创建新项目。

1. 详情

- 项目文件夹：`llplatform`
- 包管理器：`npm`
- 若文件夹存在则覆盖：√
- 初始化git仓库：√，`Init project`

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/%E5%88%9B%E5%BB%BA%E9%A1%B9%E7%9B%AE-%E8%AF%A6%E6%83%85.PNG)

2. 预设：选择手动

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/%E5%88%9B%E5%BB%BA%E9%A1%B9%E7%9B%AE-%E9%A2%84%E8%AE%BE.PNG)

3. 功能

- Babel：√
- TypeScript：×
- PWA：×
- Router：√
- Vuex：√
- CSS Pre-processors：√
- Linter / Formatter：√
- Unit Testing：×
- E2E Testing：×
- 使用配置文件：√

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/%E5%88%9B%E5%BB%BA%E9%A1%B9%E7%9B%AE-%E5%8A%9F%E8%83%BD.PNG)

4. 配置

- Use history mode for router：×
- Pick a CSS Pre-processors：`Sass/SCSS(with dart-sass)`
- Pick a linter / formatter config：`ESLint + Standard config`
- Pick additional lint features：
  - Lint on save：√
  - Lint and fix on commit：×

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/%E5%88%9B%E5%BB%BA%E9%A1%B9%E7%9B%AE-%E9%85%8D%E7%BD%AE.PNG)

稍等片刻，创建成功！

## 创建成功，运行

运行

```bash
npm run serve
```



![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/%E5%88%9B%E5%BB%BA%E9%A1%B9%E7%9B%AE-%E6%88%90%E5%8A%9F.PNG)



## 安装ELementUI

### 全局引入方式

本项目选择饿了么的[Element组件库](http://element-cn.eleme.io/#/zh-CN/component/installation)，个人感觉其风格看起来更舒服。

1. 安装Element

```bash
npm i element-ui -S
```

2. 引入Element：修改`src/main.js`文件，添加以下三行

```js
import Vue from 'vue'
import App from './App.vue'
import router from './router'
import store from './store'
import ElementUI from 'element-ui'
import 'element-ui/lib/theme-chalk/index.css'

Vue.config.productionTip = false
Vue.use(ElementUI)

new Vue({
  router,
  store,
  render: h => h(App)
}).$mount('#app')
```

3. 此时你已添加了Element，你可以修改`src/components/HelloWorld.vue`文件，添加一个Element的按钮组件来查看是否添加成功：

```html
<template>
  <div class="hello">
    <el-button>按钮</el-button>
    <h1>{{ msg }}</h1>
    <h2>Essential Links</h2>
    <ul>
       ......
```

### 部分引入方式

借助 [babel-plugin-component](https://github.com/QingWei-Li/babel-plugin-component)，我们可以只引入需要的组件，以达到减小项目体积的目的。

首先，安装 babel-plugin-component：

```bash
npm install babel-plugin-component -D
```

修改babel.config.js 配置文件

```json
module.exports = {
  'presets': [
    [
      '@babel/preset-env',
      {
        'modules': false
      }
    ]
  ],
  'plugins': [
    [
      'component',
      {
        'libraryName': 'element-ui',
        'styleLibraryName': 'theme-chalk'
      }
    ]
  ]
}
```

接下来，如果你只希望引入部分组件，比如 Button 和 Select，那么需要在 main.js 中写入以下内容：

```javascript
import Vue from 'vue'
import App from './App.vue'
import router from './router'
import store from './store'
import './plugins/element.js'

Vue.config.productionTip = false

/* eslint-disable no-new */
new Vue({
  el: '#app',
  router,
  store,
  render: h => h(App)
})
```

只需要引入`import './plugins/element.js'`就可以了，完整的组件列表太长了，单独建一个element.js存放。

还有注意`/* eslint-disable no-new */`是因为eslint不允许这样写，所以注释掉，不然编译通不过。



element.js完整组件列表和引入方式（完整组件列表以 [components.json](https://github.com/ElemeFE/element/blob/master/components.json) 为准）

```javascript
import Vue from 'vue';
import {
  Pagination,
  Dialog,
  Autocomplete,
  Dropdown,
  DropdownMenu,
  DropdownItem,
  Menu,
  Submenu,
  MenuItem,
  MenuItemGroup,
  Input,
  InputNumber,
  Radio,
  RadioGroup,
  RadioButton,
  Checkbox,
  CheckboxButton,
  CheckboxGroup,
  Switch,
  Select,
  Option,
  OptionGroup,
  Button,
  ButtonGroup,
  Table,
  TableColumn,
  DatePicker,
  TimeSelect,
  TimePicker,
  Popover,
  Tooltip,
  Breadcrumb,
  BreadcrumbItem,
  Form,
  FormItem,
  Tabs,
  TabPane,
  Tag,
  Tree,
  Alert,
  Slider,
  Icon,
  Row,
  Col,
  Upload,
  Progress,
  Spinner,
  Badge,
  Card,
  Rate,
  Steps,
  Step,
  Carousel,
  CarouselItem,
  Collapse,
  CollapseItem,
  Cascader,
  ColorPicker,
  Transfer,
  Container,
  Header,
  Aside,
  Main,
  Footer,
  Timeline,
  TimelineItem,
  Link,
  Divider,
  Image,
  Calendar,
  Backtop,
  PageHeader,
  CascaderPanel,
  Loading,
  MessageBox,
  Message,
  Notification
} from 'element-ui';

Vue.use(Pagination);
Vue.use(Dialog);
Vue.use(Autocomplete);
Vue.use(Dropdown);
Vue.use(DropdownMenu);
Vue.use(DropdownItem);
Vue.use(Menu);
Vue.use(Submenu);
Vue.use(MenuItem);
Vue.use(MenuItemGroup);
Vue.use(Input);
Vue.use(InputNumber);
Vue.use(Radio);
Vue.use(RadioGroup);
Vue.use(RadioButton);
Vue.use(Checkbox);
Vue.use(CheckboxButton);
Vue.use(CheckboxGroup);
Vue.use(Switch);
Vue.use(Select);
Vue.use(Option);
Vue.use(OptionGroup);
Vue.use(Button);
Vue.use(ButtonGroup);
Vue.use(Table);
Vue.use(TableColumn);
Vue.use(DatePicker);
Vue.use(TimeSelect);
Vue.use(TimePicker);
Vue.use(Popover);
Vue.use(Tooltip);
Vue.use(Breadcrumb);
Vue.use(BreadcrumbItem);
Vue.use(Form);
Vue.use(FormItem);
Vue.use(Tabs);
Vue.use(TabPane);
Vue.use(Tag);
Vue.use(Tree);
Vue.use(Alert);
Vue.use(Slider);
Vue.use(Icon);
Vue.use(Row);
Vue.use(Col);
Vue.use(Upload);
Vue.use(Progress);
Vue.use(Spinner);
Vue.use(Badge);
Vue.use(Card);
Vue.use(Rate);
Vue.use(Steps);
Vue.use(Step);
Vue.use(Carousel);
Vue.use(CarouselItem);
Vue.use(Collapse);
Vue.use(CollapseItem);
Vue.use(Cascader);
Vue.use(ColorPicker);
Vue.use(Transfer);
Vue.use(Container);
Vue.use(Header);
Vue.use(Aside);
Vue.use(Main);
Vue.use(Footer);
Vue.use(Timeline);
Vue.use(TimelineItem);
Vue.use(Link);
Vue.use(Divider);
Vue.use(Image);
Vue.use(Calendar);
Vue.use(Backtop);
Vue.use(PageHeader);
Vue.use(CascaderPanel);

Vue.use(Loading.directive);

Vue.prototype.$loading = Loading.service;
Vue.prototype.$msgbox = MessageBox;
Vue.prototype.$alert = MessageBox.alert;
Vue.prototype.$confirm = MessageBox.confirm;
Vue.prototype.$prompt = MessageBox.prompt;
Vue.prototype.$notify = Notification;
Vue.prototype.$message = Message;
```

## 安装axios

本项目使用[axios](https://github.com/axios/axios)（基于 promise 的 HTTP 库）来做网络请求。

```bash
npm install axios hq -S
```

创建`src/utils/request.js`文件，后续我们将对request请求进行统一封装。

## 安装mockjs

为了方便调试，本项目使用[mockjs](http://mockjs.com/)来拦截请求，生成模拟数据并返回给前端页面。

```bash
npm install mockjs -D
```

创建`src/mock/index.js`文件，用于模拟后台返回数据。

## 目录结构

1. 创建以下目录/文件：

- `src/api`目录：用于编写后台请求js
- `src/assets/css`目录：用于存放公共css文件
- `src/assets/img`目录：用于存放公共图片
- `src/mock`目录：用于编写模拟后台数据
- `src/router`目录：路由管理（将`src/router.js`移动为`src/router/index.js`）
- `src/store`目录：状态管理（将`src/store.js`移动为`src/store/index.js`）
- `src/utils`目录：用于存放公共工具文件
- `.env`文件：项目全局环境变量
- `.env.development`文件：项目开发环境变量
- `.env.development.local`文件：项目开发环境变量（本地）
- `.env.production`文件：项目生产环境变量
- `vue.config.js`文件：Vue-cli配置文件

整体目录结构如下：

```text
llplatform
|— dist                 // 构建产物
|— node_modules         // npm依赖包
|— public               // 第三方不打包资源
|— src                  // 源代码
|   |— api              // 接口处理（自行创建）
|   |— assets           // 资源文件
|   |— components       // 全局公用组件
|   |— router           // 路由规则（原文件router.js，建立文件夹管理）
|   |— store            // 状态管理（原文件store.js，建立文件夹管理）
|   |— views            // 所有视图页面
|   |— utils            // 公共工具（自行创建）
|   |— App.vue          // 入口页面
|   └─ main.js          // 入口 加载组件 初始化等
|— tests                // 自动化测试
|— .browserslistrc      //浏览器兼容配置
|— .editorconfig        //编辑器风格配置
|— .env.development     //开发环境变量配置（自行创建）
|— .env.production      //生产环境变量配置（自行创建）
|— .eslintrc.js         //ESLint规则配置
|— .gitignore           //git忽略配置
|— babel.config.js      //babel配置
|— package.json         //npm配置
|— package-lock.json    //npm依赖包锁定
|— postcss.config.js    //webpack的css-loader配置
|— vue.config.js        //vue-cli配置（自行创建）
└─ README.md            //项目简介
```

### main.js

```js
import Vue from 'vue'
import App from './App.vue'
import router from './router'
import store from './store'
import './plugins/element.js'

Vue.config.productionTip = false

/* eslint-disable no-new */
new Vue({
  el: '#app',
  router,
  store,
  components: { App },
  render: h => h(App)
}).$mount("#app")
```

这里曾经遇到过问题需要说一下：

还有一个老版本是这样的

```js
// compiler
new Vue({
  el: '#app',
  router: router,
  store: store,
  template: '<App/>',
  components: { App }
})
```

但是运行的时候，浏览器报错

> [Vue warn]: You are using the runtime-only build of Vue where the template compiler is not available. Either pre-compile the templates into render functions, or use the compiler-included build.

原因：

vue有两种形式的代码 compiler（模板）模式和runtime模式（运行时），vue模块的package.json的main字段默认为runtime模式， 指向了"dist/vue.runtime.common.js"位置。

这是vue升级到2.0之后就有的特点。

老版本的形式是compiler模式的，所以会出错。

到这里我们的问题还没完，那为什么之前是没问题的，之前vue版本也是2.x的呀？

这也是我要说的第二种解决办法

因为之前我们的webpack配置文件里有个别名配置，具体如下

```js
resolve: {
    alias: {
        'vue$': 'vue/dist/vue.esm.js' //内部为正则表达式  vue结尾的
    }
}
```


也就是说，import Vue from ‘vue’ 这行代码被解析为 import Vue from ‘vue/dist/vue.esm.js’，直接指定了文件的位置，没有使用main字段默认的文件位置

所以第二种解决方法就是，在vue.config.js文件里加上webpack的如下配置即可，

```js
configureWebpack: {
    resolve: {
      alias: {
        'vue$': 'vue/dist/vue.esm.js' 
      }
    }
```

既然到了这里我想很多人也会想到第三中解决方法，那就是在引用vue时，直接写成如下即可

import Vue from 'vue/dist/vue.esm.js'

### App.vue

```js
<template>
  <transition name="fade">
    <router-view/>
  </transition>
</template>

<script>
  export default {
  }
</script>
```

用<router-view/>占个坑，路由使用

## 更改端口号

由于是vue-cli3创建的项目，所以

在根目录下创建 vue.config.js 文件

```java
module.exports = {
    devServer: {
        port: 8888,     // 端口
    },
    lintOnSave: false   // 取消 eslint 验证
};
```

## 











