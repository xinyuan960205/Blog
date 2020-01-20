---
title: vue项目 element国际化
date: 2019-01-16 11:02:32
tags:
- 前端
- vue
categories:
- 前端
- vue
---

# vue项目 element国际化

## 说明

Element组件自身提供国际化，支持多种语言，可以在`src/main.js`添加如下代码，来令Element的组件使用指定语言。

```javascript
import ElementUI from 'element-ui'
import locale from 'element-ui/lib/locale/lang/en'
Vue.use(ElementUI, { locale })
```

但上方代码只会令Element的组件使用指定语言，并不会让我们自己写的代码能支持国际化，因此我们要加入自己的`i18n`。

另外以上是使用指定的其他一种语言，为了能够切换多种语言，我们要导入多种语言。

## 实现

以下以实现中文和英文切换为例：

1. 安装`vue-i18n`：

```javascript
npm install vue-i18n --save
```

2. 创建本系统的中文语言文件`src/assets/lang/zh-cn.js`

   ```javascript
   export default {
     common: {
       ok: '确认',
       cancel: '取消'
     },
     login: {
       login: '登录',
       register: '注册',
       ...
     }
   }
   ```

3. 创建本系统的英文语言文件`src/assets/lang/en.js`

   ```javascript
   export default {
     common: {
       ok: 'OK',
       cancel: 'Cancle'
     },
     login: {
       login: 'Login',
       register: 'Register',
       ...
     }
   }
   ```

4. 修改`src/main.js`

   ```javascript
   // The Vue build version to load with the `import` command
   // (runtime-only or standalone) has been set in webpack.base.conf with an alias.
   import Vue from 'vue'
   import App from './App'
   import router from './router/index'
   import Mock from './mock/index'
   import ElementUI from 'element-ui'
   import 'element-ui/lib/theme-chalk/index.css'
   import 'font-awesome/scss/font-awesome.scss'
   import VueScroll from 'vuescroll'
   import 'vuescroll/dist/vuescroll.css'
   import './assets/css/common.scss'
   /* 1. 导入国际化相关依赖 */
   import VueI18n from 'vue-i18n'
   import myEnLocale from './assets/lang/en'
   import myZhLocale from './assets/lang/zh-cn'
   import enLocale from 'element-ui/lib/locale/lang/en'
   import zhLocale from 'element-ui/lib/locale/lang/zh-CN'
   
   Vue.config.productionTip = false
   /* mockjs */
   Mock.mockData()
   /* vue-i18n */
   Vue.use(VueI18n)
   // 2.1 支持两种语言，每种语言需要合并自己书写的语言文件和element-ui的同语言文件
   const messages = {
     'en': Object.assign(myEnLocale, enLocale),
     'zh-cn': Object.assign(myZhLocale, zhLocale)
   }
   // 2.2 加载用户语言设置，你也可以把此值存放在后台
   const lang = localStorage.getItem('user-language') || 'zh-cn'
   // 2. 实例化VueI18n
   const i18n = new VueI18n({
     locale: lang,  // 2.2
     messages       // 2.1
   })
   /* element-ui */
   Vue.use(ElementUI, {
     // 3. element-ui默认支持vue-i18n@5.x版本，6.x以上的版本需要添加此配置项，当前已8.x
     i18n: (key, value) => i18n.t(key, value)
   })
   /* vuescroll */
   Vue.use(VueScroll, {ops: {bar: {background: '#C0C4CC'}}})
   
   /* eslint-disable no-new */
   new Vue({
     el: '#app',
     router,
     i18n,  // 4. 将i18n挂载到根实例
     components: {App},
     template: '<App/>'
   })
   ```

5. 使用：以下代码举例了在vue组件中如何使用语言文件配置的值

   ```javascript
   <template>
     <div>
     <!-- 1. 属性中使用，记得使用v-bind，即不要遗漏前面的冒号 -->
     <input :placeholder="$t('login.login')"/>
     <!-- 2. HTML中使用 -->
     <button>{{ $t("common.ok") }}</button>
     </div>
   </template>
   
   <script>
   export default {
     data () {
       // 3. js中使用
       const cancel = this.$t('common.cancel')
     }
   }
   </script>
   ```

   代码里举了三个例子，分别是v-bind绑定的、HTML里直接使用的和js中使用的，直接用this引用

6. 切换语言：记录并切换语言

```javascript
localStorage.setItem('user-language', 'zh-cn')
this.$i18n.locale = 'zh-cn'
```



## 项目实现的具体代码

### TheLayoutHeader.vue

#### template部分

```javascript
<div class="navbar-text" @click="clickLangue">
          <el-dropdown trigger="click" @command="changeLanguage" id="langDropDown" size="medium">
            <p class="user-info" style="height: 10px">
              {{ $t('header.languageSelect') }}
              <i class="el-icon-arrow-down el-icon--right drop-icon" id="langDropIcon"></i>
            </p>
            <el-dropdown-menu slot="dropdown">
              <el-dropdown-item command="zh-cn" :disabled="this.lang==='zh-cn'">
                {{$t('header.langZh')}}
              </el-dropdown-item>
              <el-dropdown-item command="en" :disabled="this.lang==='en'">
                {{$t('header.langEn')}}
              </el-dropdown-item>
            </el-dropdown-menu>
          </el-dropdown>
</div>
```

使用了element的下拉菜单，显示菜单的名称已经实现国际化。

#### script部分

```javascript
     //改变语言
      changeLanguage (language) {
        localStorage.setItem('user-language', language)
        this.$i18n.locale = language
        this.lang = language
        this.timeFormate(new Date())
      },
     //     
      clickLangue () {
        let langDropIcon = document.getElementById('langDropIcon')
        if (this.langDrop) {
          langDropIcon.style.transform = 'rotate(0deg)'
        } else {
          langDropIcon.style.transform = 'rotate(-180deg)'
        }
        this.langDrop = !this.langDrop
      },
```



### 建立文字国际化库

在`./src/assets/lang/en.js`

```javascript
export default {
  common: {
    ok: 'OK',
    cancel: 'Cancel'
  },
  login: {
    login: 'Login',
    register: 'Register',
    account: 'Account',
    password: 'Password',
    showPass: 'show password',
    forgetPass: 'forget password?',
    modifyPass: 'Change password',
    oldPass: 'Old Password',
    newPass: 'New Password',
    checkPass: 'Repeat password',
    inputAccount: 'Please input account',
    inputPass: 'Please input password',
    inputOldPass: 'Please input old password',
    inputNewPass: 'Please input new password',
    inputCheckPass: 'Please input password again',
    errorCheckPass: 'Password not the same!',
    loginSuccess: 'Login Success!',
    registerSuccess: 'Register Success!',
    changePassMessage: 'Change password?',
    tip: 'Tip',
    changeSuccess: 'Change Success',
    cancel: 'Cancel!'
  },
```

在`./src/assets/lang/zh-cn.js`

```javascript
export default {
  common: {
    ok: '确认',
    cancel: '取消'
  },
  login: {
    login: '登录',
    register: '注册',
    account: '账号',
    password: '密码',
    showPass: '显示密码',
    forgetPass: '忘记密码？',
    modifyPass: '修改密码',
    oldPass: '旧密码',
    newPass: '新密码',
    checkPass: '重复密码',
    inputAccount: '请输入账号',
    inputPass: '请输入密码',
    inputOldPass: '请输入旧密码',
    inputNewPass: '请输入新密码',
    inputCheckPass: '请再次输入密码',
    errorCheckPass: '两次输入密码不一致!',
    loginSuccess: '登录成功！',
    registerSuccess: '注册成功！',
    changePassMessage: '确认修改密码吗？',
    tip: '提示',
    changeSuccess: '修改成功',
    cancel: '已取消！'
  },
```

自己建立自己的文本库，如果觉得可以是共用的可以放到common里面，最好一个vue组件就建立一个

例如：

TheLayoutHeader.vue

```javascript
  TheLayoutHeader:{
    lock:'锁定',
    unlock:'解锁',
    logout:'退出',
    year:'年',
    month:'月',
    day:'日',
    configurationManagement:'配置管理',
    basicConfigurationManagement:'基础配置管理',
    locationConfigurationManagement:'机房配置管理',
    cabinetConfigurationManagement:'机架配置管理',
    deviceCreatPort:'业务端口配置管理',
    topologyManagement:'拓扑管理',
    subnetConfigurationManagement:'子网配置管理',
    linkConfigurationManagement:'链路配置管理',
    deviceConfigurationManagement:'网元配置管理',
    controlManagement:'控制管理',
    businessManagement:'业务管理',
    alarmManagement:'故障管理',
    performanceManagement:'性能管理',
    safeManagemnet:'安全管理',

  },
```

```javascript
  TheLayoutHeader:{
    lock:'lock',
    unlock:'unlock',
    logout:'logout',
    year:'.',
    month:'.',
    day:'',
    configurationManagement:'configurationManagement',
    basicConfigurationManagement:'basicConfigurationManagement',
    locationConfigurationManagement:'locationConfigurationManagement',
    cabinetConfigurationManagement:'cabinetConfigurationManagement',
    deviceCreatPort:'deviceCreatPort',
    topologyManagement:'topologyManagement',
    subnetConfigurationManagement:'subnetConfigurationManagement',
    linkConfigurationManagement:'linkConfigurationManagement',
    deviceConfigurationManagement:'deviceConfigurationManagement',
    controlManagement:'controlManagement',
    businessManagement:'businessManagement',
    alarmManagement:'alarmManagement',
    performanceManagement:'performanceManagement',
    safeManagemnet:'safeManagemnet',
  },
```

























