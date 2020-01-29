# Vue使用icon

一共有三种使用方法，本项目使用第三种**symbol**

## 存放SVG

在src目录新建一个icons目录，目录结构如下

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/icons%E7%BB%93%E6%9E%84%E5%9B%BE.PNG)

svg下面放iconfont的icon标签的svg标签文件，下下来复制粘贴即可使用



index.js文件：

```js
/**
 * 字体图标, 统一使用SVG Sprite矢量图标(http://www.iconfont.cn/）
 *
 * 使用:
 *  1. 在阿里矢量图标站创建一个项目, 并添加图标(这一步非必须, 创建方便项目图标管理)
 *  2-1. 添加icon, 选中新增的icon图标, 复制代码 -> 下载 -> SVG下载 -> 粘贴代码(重命名)
 *  2-2. 添加icons, 下载图标库对应[iconfont.js]文件, 替换项目[./iconfont.js]文件
 *  3. 组件模版中使用 [<icon-svg name="canyin"></icon-svg>]
 *
 * 注意:
 *  1. 通过2-2 添加icons, getNameList方法无法返回对应数据
 */
import Vue from 'vue'
import IconSvg from '@/components/icon-svg'//引入svg组件
import './iconfont.js'

Vue.component('IconSvg', IconSvg)//全局注册icon-svg

const svgFiles = require.context('./svg', true, /\.svg$/)
const iconList = svgFiles.keys().map(item => svgFiles(item))

export default {
  // 获取图标icon-(*).svg名称列表, 例如[shouye, xitong, zhedie, ...]
  getNameList () {
    return iconList.map(item => item.default.id.split('-')[1])
  }
}
```

- require.context("./test", false, /.test.js$/); 这行代码就会去 test 文件夹（不包含子目录）下面的找所有文件名以 `.test.js` 结尾的文件能被 require 的文件。 更直白的说就是 我们可以通过正则匹配引入相应的文件模块。



## 创建 icon-component 组件

如上面代码而言，我们需要封装一下icon，在components下新建icon-svg/index.vue文件

```js
<template>
  <svg
    :class="getClassName"
    :width="width"
    :height="height"
    aria-hidden="true">
    <use :xlink:href="getName"></use>
  </svg>
</template>

<script>
export default {
  name: 'icon-svg',
  props: {
    name: {
      type: String,
      required: true
    },
    className: {
      type: String
    },
    width: {
      type: String
    },
    height: {
      type: String
    }
  },
  computed: {
    getName () {
      return `#icon-${this.name}`
    },
    getClassName () {
      return [
        'icon-svg',
        `icon-svg__${this.name}`,
        this.className && /\S/.test(this.className) ? `${this.className}` : ''
      ]
    }
  }
}
</script>

<style>
  .icon-svg {
    width: 1em;
    height: 1em;
    fill: currentColor;
    overflow: hidden;
  }
</style>
```

## 使用 svg-sprite

当然如参考文章所言，我们须加载依赖svg-sprite-loader，在webpack.base.conf.js 中

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/svg%E9%85%8D%E7%BD%AE.PNG)

注意，还要安装`npm install svg-sprite-loader -D`

这里要使用到 [svg-sprite-loader](https://github.com/kisenka/svg-sprite-loader) 这个神器了， 它是一个 webpack loader ，可以将多个 svg 打包成 `svg-sprite` 。

我们发现`vue-cli`默认情况下会使用 `url-loader` 对svg进行处理，会将它放在`/img` 目录下，所以这时候我们引入`svg-sprite-loader` 会引发一些冲突。

解决方案有两种，最简单的就是你可以将 test 的 svg 去掉，这样就不会对svg做处理了，当然这样做是很不友善的。

- 你不能保证你所有的 svg 都是用来当做 icon的，有些真的可能只是用来当做图片资源的。
- 不能确保你使用的一些第三方类库会使用到 svg。

所以最安全合理的做法是使用 webpack 的 [exclude](https://webpack.js.org/configuration/module/#rule-exclude) 和 [include](https://webpack.js.org/configuration/module/#rule-include) ，让`svg-sprite-loader`只处理你指定文件夹下面的 svg，`url-loaer`只处理除此文件夹之外的所以 svg，这样就完美解决了之前冲突的问题。

## 使用

因为组件已经全局注册，即在组件中  <svg-icon iconClass="money"></svg-icon> 使用即可，这种用法好处很多。

## 参考

- https://juejin.im/post/59bb864b5188257e7a427c09
- 