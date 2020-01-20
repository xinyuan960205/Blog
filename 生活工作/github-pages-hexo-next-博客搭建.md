---
title: github pages-hexo-next 博客搭建
date: 2019-01-21 15:31:00
tags:
- 学习小记
- 学习小记
categories:
- 学习小记
- 学习小记
---

# 博客搭建记录

成功搭建了这个博客在GitHub上，主要记录一下为了以后使用方便。

主要的资源：

- 本博客使用的是hexo博客框架，官网在这[官网](https://hexo.io/zh-cn/docs/index.html) GitHub在这[GitHub](https://github.com/hexojs/hexo)

- 主题next的论坛，[这里](https://github.com/iissnan/hexo-theme-next/issues)

- hexo插件，[这里](https://hexo.io/zh-cn/)
- 

搭建博客主要借鉴的文章是有：

- [知乎的一篇文章](https://zhuanlan.zhihu.com/p/26625249?utm_source=qq&utm_medium=social)这是一片关于GitHub和hexo的文章，讲的很好，非常适合入门。

- [一位小姐姐的博客](http://jmyblog.top/Hexo-GithubPages-CodingPages%E6%90%AD%E5%BB%BA%E8%87%AA%E5%B7%B1%E7%9A%84%E4%B8%AA%E4%BA%BA%E5%8D%9A%E5%AE%A2/#more)对于博客文章的内容讲的非常详细，而且最重要的就是她汇总了网上所有大神的文章链接，也包括上面的那篇。我觉得跟着这个博客的文章做，就可以了。超级推荐，一定要看！！！

- [这是一篇很详细的hexo-next主题设置](https://zhuanlan.zhihu.com/p/28128674)基本上把设置的内容都介绍了，跟着操作就好了。

- [七牛云作为图床的使用](https://lwtang.github.io/2018/11/27/qiniu-store-image/)七牛云作为图床开始试了很多的方法，只有这个简单但是管用。

- [一位大神的博客](http://devinol.com/categories/Hexo/)对hexo有好多独有但是很有意思的设置，当你把前面的都做完或者试过了后，可以看看他的，有好多好玩的东西。
- 

下面就是记录一下搭建博客的主要内容摘抄，注意以下内容均来自以上的博客，如果想要详细了解可以去看上面的大神博客。

## 博客知识简介

### 博客生成器

目前主流的两种建站的方式：

- Wordpress
- GitHub Pages+Hexo

我使用的就是第二种，主要就是简单，但是有很好的成就感。。。

第一种Wordpress属于动态博客生成器，其中的代表还有FarBox、Ghost等，

第二种hexo属于静态博客生成器，其中的代表还有Jekyll、Octopress、Hugo等。

关于动态和静态的区别主要有以下几点:

- 资源占用上，静态的相比动态占用服务器资源少，还可以托管在Github Pages上；
- 发布更新操作上，由于静态博客没有管理后台，所以发布更新内容要比动态博客繁琐；
- 访问速度上，由于静态博客没有数据库，所以访问速度更快；
- 安全性上，静态博客相比动态博客免疫了很多Web攻击套路；

### web服务器

简单来说就是网站的部署位置，一般可以找国内的服务器提供商，如阿里云、腾讯云、七牛云等。也可以用免费的GitHub pages。一般个人博客使用GitHub pages就够用了。

### 图床服务器

~~上面的服务器用于存放网页，属于Web服务器，而图床是用于提供多媒体资源（图片、视频）存储的服务器，把网页和多媒体资源分开存储是有好处的，如果把图片都放Web服务器上，Web服务器的访问带宽会一下子就被占完，这样访问网站的体验会极差。我使用的是七牛图床，免费的而且非常好。[这是](https://yuchen-lea.github.io/2016-01-21-use-qiniu-store-file-for-hexo/)一个关于七牛云作为图床的介绍~~（目前七牛已经不能免费使用了，改为使用GitHub作为博客图床）

1，在github创建repository

2，利用github的windows客户端clone至本地

3，将需要上传的文件放入本地的文件夹

4，利用github的windows客户端commit

5，网页浏览复制图片URL，如`https://github.com/zhanghlHUST/figureForMarkdown/blob/master/17_urllib_parse_urlparse.JPG`，将URL中`blob`替换为`raw`,即`https://github.com/zhanghlHUST/figureForMarkdown/raw/master/17_urllib_parse_urlparse.JPG`

参考：<https://www.jianshu.com/p/33eeacac3344>

### 域名和备案

光有服务器还不够，此时你把网页部署上服务器后，只能通过服务器绑定的IP地址访问到你的站点。而这种对外开放的站点，基本没有使用IP来让人访问的，因为非常不方便记忆，所以，你需要购买一个域名。域名购买渠道很多，下面是三个我比较了解的渠道：

- Godaddy：[https://www.godaddy.com/](https://link.jianshu.com?t=https://www.godaddy.com/) ，老牌厂商。
- 阿里云：[https://wanwang.aliyun.com/domain/](https://link.jianshu.com?t=https://wanwang.aliyun.com/domain/) ，原中国万网，被阿里收购合并到阿里云。
- DNSPod：[https://domains.dnspod.cn/](https://link.jianshu.com?t=https://domains.dnspod.cn/) ，被腾讯收购。

### DNS解析

有了域名，等部署完服务器后，还要设置对应DNS解析，目的是为了告诉所有访问这个域名的浏览器，应该访问哪个IP地址的主机。关于DNS解析服务，这里推荐知名的老牌厂商[DNSPod](https://link.jianshu.com/?t=https://www.dnspod.cn)，服务不错，也有免费套餐。这些在阿里云上都有相应的教程，按照教程操作就可以了。

## 搭建过程

### 搭建步骤

一般来说，搭建博客有以下几个步骤：

1. 获得个人网站域名
2. GitHub创建个人仓库
3. 安装Git
4. 安装Node.js
5. 安装Hexo
6. 推送网站
7. 绑定域名
8. 更换主题
9. 初识MarkDown语法
10. 发布文章
11. 寻找图床
12. 个性化设置
13. 其他
14. 附录

以上绝大部分都是那种用一次就行的，看大神们的博客就可以了。这里我只是摘要hexo的一些重要的命令

### hexo的一些重要的命令

npm install hexo -g #安装Hexo
npm update hexo -g #升级 
hexo init #初始化博客

命令简写
hexo n "我的博客" == hexo new "我的博客" #新建文章
hexo g == hexo generate #生成
hexo s == hexo server #启动服务预览
hexo d == hexo deploy #部署

hexo server #Hexo会监视文件变动并自动更新，无须重启服务器
hexo server -s #静态模式
hexo server -p 5000 #更改端口
hexo server -i 192.168.1.1 #自定义 IP
hexo clean #清除缓存，若是网页正常情况下可以忽略这条命令  

- 如果是修改完本地的博客，想要预览效果的话，可以输入下面的三个命令（命令界面需要进入到博客的文件夹，按shift右击，选择在此处打开powershell窗口，就是npm的命令界面窗口）

```bash
hexo clean 
hexo g 
hexo s
```

以上命令成功后，在浏览器输入： http://localhost:4000，就可以看到新的预览界面

- 如果本地预览完成了，想要推送

```bash
hexo clean 
hexo g 
hexo d
```

以上命令成功后，就可以输入自己的GitHub网页的个人博客进行访问。

以上的命令需要了解就行，后面个性化的时候，可以把三个命令绑定为一个命令，更为快速。

## 个性化

讲道理，个性化才是搭建博客真正的乐趣所在，感觉就像小时候照着图纸拼四驱车一样，但是又比拼四驱车要好，这个做坏了可以回退版本，四驱车就废了!ヽ(￣▽￣)ﾉ

我使用的主题是next，为什么？因为大家都在用，教程多，不会还可以查啊。。。

以下就摘一些我觉得重要

### 添加菜单

进入theme目录，编辑_config_yml文件，找到menu:字段，在该字段下添加一个字段。

```
menu:
  home: /
  about: /about
  ......
```

然后找到lanhuages目录，编辑zh-Hans.yml文件：

```
menu:
  home: 首页
  about: 关于作者
  ......
```

更新页面显示的中文字符，最后进入theme目录下的Source目录，新增一个about目录，里面写一个index.html文件。



### 自定义hexo new生成md文件的选项

在/scaffolds/post.md文件中添加：

```
---
title: {{ title }}
date: {{ date }}
tags:
categories: 
copyright: true
permalink: 01
top: 0
password:
---
```



### 博文置顶

修改 hero-generator-index 插件，把文件：node_modules/hexo-generator-index/lib/generator.js 内的代码替换为：

```
'use strict';
var pagination = require('hexo-pagination');
module.exports = function(locals){
  var config = this.config;
  var posts = locals.posts;
    posts.data = posts.data.sort(function(a, b) {
        if(a.top && b.top) { // 两篇文章top都有定义
            if(a.top == b.top) return b.date - a.date; // 若top值一样则按照文章日期降序排
            else return b.top - a.top; // 否则按照top值降序排
        }
        else if(a.top && !b.top) { // 以下是只有一篇文章top有定义，那么将有top的排在前面（这里用异或操作居然不行233）
            return -1;
        }
        else if(!a.top && b.top) {
            return 1;
        }
        else return b.date - a.date; // 都没定义按照文章日期降序排
    });
  var paginationDir = config.pagination_dir || 'page';
  return pagination('', posts, {
    perPage: config.index_generator.per_page,
    layout: ['index', 'archive'],
    format: paginationDir + '/%d/',
    data: {
      __index: true
    }
  });
};
```

在文章中添加 top 值，数值越大文章越靠前，如:

```
---
......
copyright: true
top: 100
---
```

默认不设置则为0，数值相同时按时间排序。



### 添加顶部加载条

打开/themes/next/layout/_partials/head.swig文件，在maximum-scale=1”/>后添加如下代码:

```
<script src="//cdn.bootcss.com/pace/1.0.2/pace.min.js"></script>
<link href="//cdn.bootcss.com/pace/1.0.2/themes/pink/pace-theme-flash.css" rel="stylesheet">
```

但是，默认的是粉色的，要改变颜色可以在/themes/next/layout/_partials/head.swig文件中添加如下代码（接在刚才link的后面）

```
<style>
    .pace .pace-progress {
        background: #1E92FB; /*进度条颜色*/
        height: 3px;
    }
    .pace .pace-progress-inner {
         box-shadow: 0 0 10px #1E92FB, 0 0 5px     #1E92FB; /*阴影颜色*/
    }
    .pace .pace-activity {
        border-top-color: #1E92FB;    /*上边框颜色*/
        border-left-color: #1E92FB;    /*左边框颜色*/
    }
</style>
```

### 添加自动部署命令

在package.json里面添加如下命令：

```
"scripts": {
"build": "hexo clean && hexo g && gulp && hexo d",
"test": "hexo clean && hexo g && gulp && hexo s"
},
```

- 然后运行`npm run build`，我们就会自动删除老文件，生成新文件，压缩html、css然后发布到github或其他静态服务器资源。但是不能成功推送，不知道是为什么？？？
- 然后运行`npm run test`，我们就会自动删除老文件，生成新文件，压缩html、css然后发布到本地服务器做预览。

## 最后end

写这个文章就是记录分享，大家借鉴就好，主要也是做个记录，毕竟不能白搞了这么长的时间。

对了，关于markdown我用的是typora，感觉挺好的，可以直接用效果去写，连那些markdown的格式都可以忽略了。