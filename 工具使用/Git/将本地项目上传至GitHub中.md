# 一、本地创建好对应的项目

测试一下，GitHub能不能更改11111

先在本地创建一个自己的项目，右键点击项目的文件夹，选择`git bash`弹出对应的界面。

# 二、执行git命令

## 2.1 执行命令：git init

通过git init命令让新建的文件夹成为git可管理的仓库

```bash
$ git init
```

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/%E6%89%A7%E8%A1%8Cgit%E5%91%BD%E4%BB%A4.PNG)

执行完这一步在test文件夹里就会看到.git文件夹，它是用来跟踪和管理版本库的，如果看不到说明.git文件是隐藏文件，解决办法，查看–>勾选隐藏的文件，这样就可以看到了。接下来将本地项目放到这个文件夹里 

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/%E9%9A%90%E8%97%8F%E6%96%87%E4%BB%B6.git.PNG)

## 2.2 执行命令：git add .

表示把该目录下的所有文件加入到本地暂存区中。执行成功后不会有任何提示

```bash
$ git add .      *//注意add后面有一个点，并且用空格分开
```

只要过程中不出现error提示，就都没什么问题，此时还可以通过git status 指令进行状态查看。

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/%E6%89%A7%E8%A1%8Cgit%20add%E5%91%BD%E4%BB%A4.PNG)

## 2.3 执行命令：git commit -m "注释"

该命令会把本地暂存区中的文件提交到本地历史区，注意只有在本地历史区中的内容才能提交到github。执行该命令后，我们所有的文件都只是在本地。和github没有任何关系。

```bash
$ git commit -m "first commit"
```

-m后面跟的是本次提交注释的内容，可以不写，但建议写上，不然容易报错，好了，到了这一步本地仓库创建就做好了，接下来就要连接github了 

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/git%20commit.PNG)



# 三、连接GitHub

在连接github之前我想说的是，你首先要有github账号，并且SSH KEY是已经创建好了，因为git仓库和github仓库的连接是通过ssh加密的，还没有创建的同学可以Google一下，在这里就不详细说了，直接切入主题 

## 3.1 创建仓库

进入github仓库管理界面，点击new创建 

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/GitHub%E5%88%9B%E5%BB%BA%E6%96%B0%E9%A1%B9%E7%9B%AE.PNG)

## 3.2 执行命令：git remote add origin 项目Clone地址

该命令是把本地历史区中的文件添加到github服务器的暂存区中。这一步是本地和远程服务器建立联系的一步。执行成功后不会显示任何结果：

```bash
$ git remote add origin https://github.com/xinyuan960205/Blog.git
```

origin后面的地址

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/origin%E7%9A%84%E5%9C%B0%E5%9D%80.PNG)

## 3.3 执行命令：git pull origin master

关联好之后就是将文件push到github上了 

如果github仓库的文件夹是空的执行

```bash
$ git push -u origin master
```

弹出再次输入GitHub的用户名密码

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/%E5%85%B3%E8%81%94%E5%A5%BD%E5%90%8E%E6%8E%A8%E9%80%81%E5%BC%B9%E5%87%BA%E8%BE%93%E5%85%A5%E7%94%A8%E6%88%B7%E5%90%8D%E5%AF%86%E7%A0%81.PNG)

结果出了错误

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/%E7%AC%AC%E4%B8%80git%20push%E5%A4%B1%E8%B4%A5.PNG)

不过此时会有一个坑，如果我们在创建github仓库是勾选了Initialize this repository with a README（就是创建仓库的时候自动给你创建一个README文件），此时我们就需要执行接下来的一步

```bash
$ git pull --rebase origin master
```

   ![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/%E4%BB%A3%E7%A0%81%E6%8B%89%E5%8F%96%E5%90%88%E5%B9%B6%E8%A7%A3%E5%86%B3%E9%97%AE%E9%A2%98.PNG)

## 3.4 执行命令：git push -u origin master

执行完这一步再来一遍$ git push origin master就可以了 

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/%E7%AC%AC%E4%B8%80%E6%AC%A1git%20push%E6%88%90%E5%8A%9F.PNG)

等远程仓库里面有了内容之后，下次再从本地库上传内容的时候只需下面这样就可以了

```bash
$ git push origin master
```

# 四、成功上传  

到这里就结束了，打开github就可以看到我们的项目就已经传到github上面了 



# 参考

- <https://blog.csdn.net/qq_28301007/article/details/77917957>
- <https://blog.csdn.net/YSBJ123/article/details/80408231>
- <https://blog.csdn.net/CHENYUFENG1991/article/details/48930471>