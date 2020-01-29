# vue-router设置

router/inedx.js

## 根据环境设置

```js
// 开发环境不使用懒加载, 因为懒加载页面太多的话会造成webpack热更新太慢, 所以只有生产环境使用懒加载
const _import = require('./import-' + process.env.NODE_ENV)
```

process.env.NODE_ENV 是打包的环境

## 路由的配置

```js
const router = new Router({
  mode: 'hash',
  scrollBehavior: () => ({ y: 0 }), // 每次访问滚动条都置0
  isAddDynamicMenuRoutes: false, // 是否已经添加动态（菜单）路由
  routes: globalRoutes.concat(mainRoutes)
})
```

讲一下这几个属性：

**base**（这里就用了默认，所以没有出现）

浏览器url的前缀，默认为`'/'`，如果设置为`'/some/'`,则运行项目，浏览器url都是`'/some/...'`,不会对静态文件的引用产生影响，一般写网站都会有域名，都可以把域名指向某个服务器目录，所以默认`'/'`即可，如果是写公司内部后台管理，那么不一定会用域名，可能就是某个ip下的某个目录，比如打包后的文件在`111.22.33.44/admin`，这个时候路由的配置匹配浏览器路径的时候，会从这个`/admin`开始算，如果base还是默认的`/`，那么路由配置的`routes`的`path`就要全部加上`/admin/`前缀，并且`router-link`和`push`方法也要加上这个`/admin`，很麻烦,但是只要设置`base`为`'/admin/'`,路由内部配置以及所有相关的方法都可以忽视服务器ip下的目录名。这种情况，同样也要配置`webpack`的`publicPath`也为`/admin/`，这里不细说了。

**mode**

这个太简单了，一共三种模式，

`hash`:浏览器会有‘#’符号，参考锚点效果，缺点很丑，但是兼容性棒棒

`history`:去除‘#’符号，让url变好看，下面会讲服务端配置。

`abstract`:非浏览器环境，会强制使用这个模式,例如weex

**routes**

核心路由配置，[官网](https://router.vuejs.org/zh/guide)讲的很详细，我就讲几个注意点，所有自定义的配置，例如是否需要鉴权或者对应的icon等，需要规范化的话都写在`meta`对象中， 不规范的话 ，就和`path`同级咯，随意玩，玩坏了我不背锅。

`path`是必须指定的，`name`需要唯一，不是必须。

`routes`的配置遵循顺序匹配，当url成功匹配，就不会再往下匹配，所以像`403,404`的页面应当写在最后。

`alias`别名的使用，当需要在指定`router-view`显示某个组件，并且希望浏览器url是自己想要的语义时使用

`redirect`重定向，参数可以是路径，也可以是对象(重定向到某个name),注意重定向是改变内容+改变url

**scrollBehavior**

这个滚动行为，老实说，感觉很蹩脚，本人基本没有使用过，他控制的是**body**的滚动， 很多需求都是局部滚动。如果的确是需要控制body滚动，参考官方文档即可。



## 全局路由

```js
// 全局路由(无需嵌套上左右整体布局)
const globalRoutes = [
  { path: '/404', component: _import('common/404'), name: '404', meta: { title: '404未找到' } },
  { path: '/login', component: _import('common/login'), name: 'login', meta: { title: '登录' } }
]
```





## 主入口路由

```js
// 主入口路由（需嵌套上左右整体布局）
const mainRoutes = {
  path: '/',
  component: _import('main'),
  name: 'main',
  redirect: { name: 'home' },
  meta: { title: '主入口整体布局' },
  children: [
    // 通过meta对象设置路由展示方式
    // 1. isTab: 是否通过tab展示内容, true: 是, false: 否
    // 2. iframeUrl: 是否通过iframe嵌套展示内容, '以http[s]://开头': 是, '': 否
    // 提示: 如需要通过iframe嵌套展示内容, 但不通过tab打开, 请自行创建组件使用iframe处理!
    { path: '/home', component: _import('common/home'), name: 'home', meta: { title: '首页' } },
    { path: '/theme', component: _import('common/theme'), name: 'theme', meta: { title: '主题' } },
    { path: '/article/article/update/:id',
      component: _import('modules/article/article-add-or-update'),
      name: 'article-update',
      meta: {
        menuId: 'article-update',
        title: '博文修改',
        isTab: true
      }
    },
    { path: '/book/book/update/:id',
      component: _import('modules/book/book-add-or-update'),
      name: 'book-update',
      meta: {
        menuId: 'book-update',
        title: '阅读修改',
        isTab: true
      }
    },
    { path: '/book/note/update/:id',
      component: _import('modules/book/note-add-or-update'),
      name: 'book-note-update',
      meta: {
        menuId: 'book-note-update',
        title: '笔记修改',
        isTab: true
      }
    }
  ],
  beforeEnter (to, from, next) {
    let token = Vue.cookie.get('token')
    if (!token || !/\S/.test(token)) { // 正则：非空白就匹配
      clearLoginInfo()
      next({ name: 'login' })
    }
    next()
  }
}
```

路由的配置一开始只有根目录`/`，每写一层`children`就要写一层`router-view`，否则组件不显示。每一个嵌套`children`的层级和`router-view`的层级都是一一对应的。

核心路由配置，[官网](https://router.vuejs.org/zh/guide)讲的很详细，我就讲几个注意点，所有自定义的配置，例如是否需要鉴权或者对应的icon等，需要规范化的话都写在`meta`对象中， 不规范的话 ，就和`path`同级咯，随意玩，玩坏了我不背锅。

`path`是必须指定的，`name`需要唯一，不是必须。

`routes`的配置遵循顺序匹配，当url成功匹配，就不会再往下匹配，所以像`403,404`的页面应当写在最后。

`alias`别名的使用，当需要在指定`router-view`显示某个组件，并且希望浏览器url是自己想要的语义时使用

`redirect`重定向，参数可以是路径，也可以是对象(重定向到某个name),注意重定向是改变内容+改变url

### beforeEnter函数

用来判断token是否有值，没有的话就跳转到login界面

​	每个守卫方法接收三个参数：

- **`to: Route`**: 即将要进入的目标 [路由对象](https://router.vuejs.org/zh/api/#路由对象)
- **`from: Route`**: 当前导航正要离开的路由
- **`next: Function`**: 一定要调用该方法来 **resolve** 这个钩子。执行效果依赖 `next` 方法的调用参数。
  - **`next()`**: 进行管道中的下一个钩子。如果全部钩子执行完了，则导航的状态就是 **confirmed** (确认的)。
  - **`next(false)`**: 中断当前的导航。如果浏览器的 URL 改变了 (可能是用户手动或者浏览器后退按钮)，那么 URL 地址会重置到 `from` 路由对应的地址。
  - **`next('/')` 或者 `next({ path: '/' })`**: 跳转到一个不同的地址。当前的导航被中断，然后进行一个新的导航。你可以向 `next` 传递任意位置对象，且允许设置诸如 `replace: true`、`name: 'home'` 之类的选项以及任何用在 [`router-link` 的 `to` prop](https://router.vuejs.org/zh/api/#to) 或 [`router.push`](https://router.vuejs.org/zh/api/#router-push) 中的选项。
  - **`next(error)`**: (2.4.0+) 如果传入 `next` 的参数是一个 `Error` 实例，则导航会被终止且该错误会被传递给 [`router.onError()`](https://router.vuejs.org/zh/api/#router-onerror) 注册过的回调。

**确保要调用 `next` 方法，否则钩子就不会被 resolved。**

 注意：`let token = Vue.cookie.get('token')`需要安装vue-cookie，不然会报错找不到cookie



## beforeEach

全局钩子，在跳转之前执行

```js
router.beforeEach((to, from, next) => {
  // 添加动态路由（菜单）
  //先判断是否为全局路由，如果是就不需要获取动态菜单
  if (router.options.isAddDynamicMenuRoutes || fnCurrentRouteType(to) === 'global') {
    next()
  } else {
    http({
      //这个接口后端会根据用户权限返回menuList和permissions
      url: http.adornUrl('/admin/sys/menu/nav'),
      method: 'get',
      params: http.adornParams()
    }).then(({data}) => {
      if (data && data.code === 200) {
        fnAddDynamicMenuRoutes(data.menuList)
        router.options.isAddDynamicMenuRoutes = true
        //使用sessionStorage 存储了动态菜单和权限
        sessionStorage.setItem('menuList', JSON.stringify(data.menuList || []))
        sessionStorage.setItem('permissions', JSON.stringify(data.permissions || []))
        next({...to, replace: false}) // hack方法 确保addRoutes已完成 ,set the replace: true so the navigation will not leave a history record
      } else {
        sessionStorage.setItem('menuList', '[]')
        sessionStorage.setItem('permissions', '[]')
        next()
      }
    })
  }
})
```

这个函数的作用是，根据不同用户的不同权限，动态获取菜单。

sessionStorage - 针对一个 session 的数据存储（关闭窗口，存储的数据清空）



```js
/**
 * 判断当前路由类型： global：全局路由，main：主入口路由
 * @param route
 */
function fnCurrentRouteType (route) {
  for (var i = 0; i < globalRoutes.length; i++) {
    if (route.path === globalRoutes[i].path) {
      return 'global'
    }
  }
  return 'main'
}

/**
 * 添加动态(菜单)路由
 * @param {*} menuList 菜单列表
 * @param {*} routes 递归创建的动态(菜单)路由
 */
function fnAddDynamicMenuRoutes (menuList = [], routes = []) {
  var temp = []
  for (var i = 0; i < menuList.length; i++) {
    if (menuList[i].list && menuList[i].list.length >= 1) {
      temp = temp.concat(menuList[i].list)
    } else if (menuList[i].url && /\S/.test(menuList[i].url)) {
      menuList[i].url = menuList[i].url.replace(/^\//, '')
      var route = {
        path: menuList[i].url.replace(new RegExp('/', 'g'), '-'),
        component: null,
        name: menuList[i].url.replace(new RegExp('/', 'g'), '-'),
        meta: {
          menuId: menuList[i].menuId,
          title: menuList[i].name,
          isDynamic: true,
          isTab: true,
          iframeUrl: ''
        }
      }
      // url以http[s]://开头, 通过iframe展示
      if (isURL(menuList[i].url)) {
        route['path'] = `i-${menuList[i].menuId}`
        route['name'] = `i-${menuList[i].menuId}`
        route['meta']['iframeUrl'] = menuList[i].url
      } else {
        try {
          // 会同时获取子组件
          route['component'] = _import(`modules/${menuList[i].url}`) || null
        } catch (e) {}
      }
      routes.push(route)
    }
  }
  if (temp.length >= 1) {
    fnAddDynamicMenuRoutes(temp, routes)
  } else {
    mainRoutes.name = 'main-dynamic'
    mainRoutes.children = routes
    router.addRoutes([
      mainRoutes,
      { path: '*', redirect: { name: '404' } }
    ])
    sessionStorage.setItem('dynamicMenuRoutes', JSON.stringify(mainRoutes.children || '[]'))
    console.log('\n')
    console.log('%c!<-------------------- 动态(菜单)路由 s -------------------->', 'color:blue')
    console.log(mainRoutes.children)
    console.log('%c!<-------------------- 动态(菜单)路由 e -------------------->', 'color:blue')
  }
}
```



# 参考

- https://blog.csdn.net/qq_40701522/article/details/81317543
- 