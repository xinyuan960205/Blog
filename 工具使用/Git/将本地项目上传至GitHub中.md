# 一、本地创建好对应的项目

测试一下，GitHub能不能更改11111

先在本地创建一个自己的项目，右键点击项目的文件夹，选择`git bash`弹出对应的界面。

# 二、执行git命令

## 2.1 执行命令：git init

通过git init命令让新建的文件夹成为git可管理的仓库

```bash
$ git init
```

![](D:\projects\GitHub\Blog\工具使用\Git\picture\执行git命令.PNG)

执行完这一步在test文件夹里就会看到.git文件夹，它是用来跟踪和管理版本库的，如果看不到说明.git文件是隐藏文件，解决办法，查看–>勾选隐藏的文件，这样就可以看到了。接下来将本地项目放到这个文件夹里 

![](D:\projects\GitHub\Blog\工具使用\Git\picture\隐藏文件.git.PNG)

## 2.2 执行命令：git add .

表示把该目录下的所有文件加入到本地暂存区中。执行成功后不会有任何提示

```bash
$ git add .      *//注意add后面有一个点，并且用空格分开
```

只要过程中不出现error提示，就都没什么问题，此时还可以通过git status 指令进行状态查看。

![](D:\projects\GitHub\Blog\工具使用\Git\picture\执行git add命令.PNG)

## 2.3 执行命令：git commit -m "注释"

该命令会把本地暂存区中的文件提交到本地历史区，注意只有在本地历史区中的内容才能提交到github。执行该命令后，我们所有的文件都只是在本地。和github没有任何关系。

```bash
$ git commit -m "first commit"
```

-m后面跟的是本次提交注释的内容，可以不写，但建议写上，不然容易报错，好了，到了这一步本地仓库创建就做好了，接下来就要连接github了 

![](D:\projects\GitHub\Blog\工具使用\Git\picture\git commit.PNG)



# 三、连接GitHub

在连接github之前我想说的是，你首先要有github账号，并且SSH KEY是已经创建好了，因为git仓库和github仓库的连接是通过ssh加密的，还没有创建的同学可以Google一下，在这里就不详细说了，直接切入主题 

## 3.1 创建仓库

进入github仓库管理界面，点击new创建 

![](D:\projects\GitHub\Blog\工具使用\Git\picture\GitHub创建新项目.PNG)

## 3.2 执行命令：git remote add origin 项目Clone地址

该命令是把本地历史区中的文件添加到github服务器的暂存区中。这一步是本地和远程服务器建立联系的一步。执行成功后不会显示任何结果：

```bash
$ git remote add origin https://github.com/xinyuan960205/Blog.git
```

origin后面的地址

![](D:\projects\GitHub\Blog\工具使用\Git\picture\origin的地址.PNG)

## 3.3 执行命令：git pull origin master

关联好之后就是将文件push到github上了 

如果github仓库的文件夹是空的执行

```bash
$ git push -u origin master
```

弹出再次输入GitHub的用户名密码

![](D:\projects\GitHub\Blog\工具使用\Git\picture\关联好后推送弹出输入用户名密码.PNG)

结果出了错误

![](D:\projects\GitHub\Blog\工具使用\Git\picture\第一git push失败.PNG)

不过此时会有一个坑，如果我们在创建github仓库是勾选了Initialize this repository with a README（就是创建仓库的时候自动给你创建一个README文件），此时我们就需要执行接下来的一步

```bash
$ git pull --rebase origin master
```

   ![](D:\projects\GitHub\Blog\工具使用\Git\picture\代码拉取合并解决问题.PNG)

## 3.4 执行命令：git push -u origin master

执行完这一步再来一遍$ git push origin master就可以了 

![](D:\projects\GitHub\Blog\工具使用\Git\picture\第一次git push成功.PNG)

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