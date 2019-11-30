---
title: git版本控制工具介绍
date: 2019-04-13 18:04:15
tags:
- 学习小记
- 学习小记
categories:
- 学习小记
- 学习小记
---

# git版本控制工具

## 一、git常用命令

### Git Clone

用于初始检出远程git仓库。默认克隆远程git仓库的默认分支。

### Git Commit

用于提交修改。Git commit在概念上就是代码库的一个快照，是Git版本管理的基本单元。多个git仓库同步的基本内容也是commit。一般来说，git库的commit提交不会丢失。

注意，git commit只是提交到本地，还需要git push才能同步到远程git仓库。

### Git Pull

从远程git仓库拉取修改。拉取的内容主要包括Git远程仓库中新的提交，新建的分支等。

### Git Push

向远程git仓库推送本地的修改。推送的内容主要包括本地仓库中新的提交，新建的分支等。

### Git Branch

在git仓库中建立分支。在Git设计中，分支是指向某个提交的指针。这种实现非常轻量级，因此可以支持频繁的创建分支、合并分支。具体请参考Git工作流程。

### Git Merge

合并git仓库中的两个分支。

### Git Tag

在git仓库的某个分支的某个提交上打上标签，一般标签为发布版本号，用于标记正式发布。

### Git Log

查看git仓库的提交历史。

### Git Status

查看本地Git工作目录中的变化情况。

### Git Stash

用于在进行git操作之前保存本地（未经提交的）修改。

## 二、git提交

### Git提交（Commit）

在Git版本管理看来， Git提交（commit）就是代码库的一个快照，是版本管理的基本单元。在开发人员看来，Git提交（commit）就是一个不可分割的项目/开发任务步骤，这个步骤包括一组对程序/文档的修改（Changeset）。而提交注释（Commit Message）将Git提交与该提交的任务语义关联起来。这也是为什么Git强制要求必须写提交注释的原因。

> 必须在Git提交注释中明确地说明Git提交完成的任务步骤。
>
> 也就是说，不允许出现无信息、无意义的提交注释，例如“小修改”，“修改XXX文件”（因为修改的文件在提交中本来就能看见，因此这个注释没有提供信息）。

### Git提交的时机和频率

#### 提交和推送

请注意，提交（commit）只是提交到本地git仓库，还需要使用推送（push）到远程git仓库中。

一般来说，修改和提交应该共享给项目组其他成员，也就是说，

> 应该尽可能在提交（commit）之后马上推送（push）。
>
> 除非是在脱机工作（例如出差没有网络，或者无法访问公司内部的git服务器）。这时则允许只提交（commit）不推送（push），等能够连上git服务器后，再将提交都同步到远程git仓库。

#### 提交的时机

按照任务提交，每完成一个任务，必须提交一次。尽量不把多个任务的代码修改一起提交。

必须提交能通过编译代码。

最好提交通过单元测试的代码。

#### 提交的频率

 提交的尽可能频繁。但是需要遵循提交时机的原则。也就是说，

> 只要可以提交，就应该提交。

## 三、git工作流程

### 集中式工作流(Centralized Workflow)

![](<https://github.com/xinyuan960205/pic_resource/raw/master/%E5%AD%A6%E4%B9%A0%E5%B0%8F%E8%AE%B0/git/%E9%9B%86%E4%B8%AD%E5%BC%8F%E5%B7%A5%E4%BD%9C%E6%B5%81.png>)

首先，团队可以用和`Subversion`完全相同的方式来开发项目，即使用集中式工作流(Centralized Workflow)来进行开发。

> 集中式工作流只有在项目规模很小且项目团队人员很少的情况下才允许采用。采用时应在项目策划中申请公司批准。

但使用`Git`加强开发的工作流，`Git`比`SVN有`几个优势 ：

首先，每个开发可以有属于自己的整个工程的本地拷贝。隔离的环境让各个开发者的工作和项目的其他部分（修改）独立开来 —— 即自由地提交到自己的本地仓库，先完全忽略上游的开发，直到方便的时候再把修改反馈上去。

其次，`Git`提供了强壮的分支和合并模型。不像`SVN`，`Git`的分支设计成可以做为一种用来在仓库之间集成代码和分享修改的『失败安全』的机制。

#### 工作方式

像`Subversion`一样，集中式工作流以中央仓库作为项目所有修改的单点实体。相比`SVN`缺省的开发分支`trunk`，`Git`叫做`master`，所有修改提交到这个分支上。该工作流只用到`master`这一个分支。

开发者开始先克隆中央仓库。在自己的项目拷贝中，像`SVN`一样的编辑文件和提交修改；但修改是存在本地的，和中央仓库是完全隔离的。开发者可以把和上游的同步延后到一个方便时间点。

要发布修改到正式项目中，开发者要把本地`master`分支的修改『推（push）』到中央仓库中。这相当于`svn commit`操作，但`push`操作会把所有还不在中央仓库的本地提交都推上去。

![](<https://github.com/xinyuan960205/pic_resource/raw/master/%E5%AD%A6%E4%B9%A0%E5%B0%8F%E8%AE%B0/git/%E9%9B%86%E4%B8%AD%E5%BC%8F%E5%B7%A5%E4%BD%9C%E6%96%B9%E5%BC%8F.png>)

#### 冲突解决

中央仓库代表了正式项目，所以提交历史应该被尊重且是稳定不变的。如果开发者本地的提交历史和中央仓库有分歧，`Git`会拒绝`push`提交否则会覆盖已经在中央库的正式提交。

![](<https://github.com/xinyuan960205/pic_resource/raw/master/%E5%AD%A6%E4%B9%A0%E5%B0%8F%E8%AE%B0/git/%E9%9B%86%E4%B8%AD%E5%BC%8F%E5%86%B2%E7%AA%81%E8%A7%A3%E5%86%B3.png>)

在开发者提交自己功能修改到中央库前，需要先`fetch`在中央库的新增提交，`rebase`自己提交到中央库提交历史之上。这样做的意思是在说，『我要把自己的修改加到别人已经完成的修改上。』最终的结果是一个完美的线性历史，就像以前的`SVN`的工作流中一样。

如果本地修改和上游提交有冲突，`Git`会暂停`rebase`过程，给你手动解决冲突的机会。`Git`解决合并冲突，用和生成提交一样的`git status`和`git add`命令，很一致方便。还有一点，如果解决冲突时遇到麻烦，`Git`可以很简单中止整个`rebase`操作，重来一次（或者让别人来帮助解决）

#### 示例

让我们一起逐步分解来看看一个常见的小团队如何用这个工作流来协作的。有两个开发者小明和小红，看他们是如何开发自己的功能并提交到中央仓库上的。

##### 有人先初始化好中央仓库

![](<https://github.com/xinyuan960205/pic_resource/raw/master/%E5%AD%A6%E4%B9%A0%E5%B0%8F%E8%AE%B0/git/%E9%9B%86%E4%B8%AD%E5%BC%8F%E5%88%9D%E5%A7%8B%E5%8C%96%E4%BB%93%E5%BA%93.png>)

第一步，有人在服务器上创建好中央仓库。如果是新项目，你可以初始化一个空仓库；否则你要导入已有的`Git`或`SVN`仓库。

中央仓库应该是个裸仓库（`bare repository`），即没有工作目录（`working directory`）的仓库。可以用下面的命令创建：

```
ssh user@host
git init --bare /path/to/repo.git
```

确保写上有效的`user`（`SSH`的用户名），`host`（服务器的域名或IP地址），`/path/to/repo.git`（你想存放仓库的位置）。注意，为了表示是一个裸仓库，按照约定加上`.git`扩展名到仓库名上。

##### 所有人克隆中央仓库

![](<https://github.com/xinyuan960205/pic_resource/raw/master/%E5%AD%A6%E4%B9%A0%E5%B0%8F%E8%AE%B0/git/%E9%9B%86%E4%B8%AD%E5%BC%8F%E5%85%8B%E9%9A%86%E4%B8%AD%E5%A4%AE%E4%BB%93%E5%BA%93.png>)

下一步，各个开发者创建整个项目的本地拷贝。通过`git clone`命令完成：

```
git clone ssh://user@host/path/to/repo.git
```

基于你后续会持续和克隆的仓库做交互的假设，克隆仓库时`Git`会自动添加远程别名`origin`指回『父』仓库。

#####  小明开发功能

![](<https://github.com/xinyuan960205/pic_resource/raw/master/%E5%AD%A6%E4%B9%A0%E5%B0%8F%E8%AE%B0/git/%E9%9B%86%E4%B8%AD%E5%BC%8F%E5%B0%8F%E6%98%8E%E5%BC%80%E5%8F%91.png>)

在小明的本地仓库中，他使用标准的`Git`过程开发功能：编辑、暂存（`Stage`）和提交。如果你不熟悉暂存区（`Staging Area`），这里说明一下：**暂存区**的用来准备一个提交，但可以不用把工作目录中所有的修改内容都包含进来。这样你可以创建一个高度聚焦的提交，尽管你本地修改很多内容。

```
git status # 查看本地仓库的修改状态
git add # 暂存文件
git commit # 提交文件
```

请记住，因为这些命令生成的是本地提交，小明可以按自己需求反复操作多次，而不用担心中央仓库上有了什么操作。对需要多个更简单更原子分块的大功能，这个做法是很有用的。

##### 小红开发功能

![](<https://github.com/xinyuan960205/pic_resource/raw/master/%E5%AD%A6%E4%B9%A0%E5%B0%8F%E8%AE%B0/git/%E9%9B%86%E4%B8%AD%E5%BC%8F%E5%B0%8F%E7%BA%A2%E5%BC%80%E5%8F%91.png>)







与此同时，小红在自己的本地仓库中用相同的编辑、暂存和提交过程开发功能。和小明一样，她也不关心中央仓库有没有新提交；当然更不关心小明在他的本地仓库中的操作，因为所有本地仓库都是私有的。

##### 小明发布功能

![](<https://github.com/xinyuan960205/pic_resource/raw/master/%E5%AD%A6%E4%B9%A0%E5%B0%8F%E8%AE%B0/git/%E9%9B%86%E4%B8%AD%E5%BC%8F%E5%B0%8F%E6%98%8E%E5%8F%91%E5%B8%83.png>)

一旦小明完成了他的功能开发，会发布他的本地提交到中央仓库中，这样其它团队成员可以看到他的修改。他可以用下面的`git push`命令：

```
git push origin master
```

注意，`origin`是在小明克隆仓库时`Git`创建的远程中央仓库别名。`master`参数告诉`Git`推送的分支。由于中央仓库自从小明克隆以来还没有被更新过，所以`push`操作不会有冲突，成功完成。

##### 小红试着发布功能

![](<https://github.com/xinyuan960205/pic_resource/raw/master/%E5%AD%A6%E4%B9%A0%E5%B0%8F%E8%AE%B0/git/%E9%9B%86%E4%B8%AD%E5%BC%8F%E5%B0%8F%E7%BA%A2%E8%AF%95%E7%9D%80%E5%8F%91%E5%B8%83.png>)

一起来看看在小明发布修改后，小红`push`修改会怎么样？她使用完全一样的`push`命令：

```
git push origin master
```

但她的本地历史已经和中央仓库有分岐了，`Git`拒绝操作并给出下面很长的出错消息：

```
error: failed to push some refs to '/path/to/repo.git'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Merge the remote changes (e.g. 'git pull')
hint: before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

这避免了小红覆写正式的提交。她要先`pull`小明的更新到她的本地仓库合并上她的本地修改后，再重试。

##### 小红在小明的提交之上rebase

![](<https://github.com/xinyuan960205/pic_resource/raw/master/%E5%AD%A6%E4%B9%A0%E5%B0%8F%E8%AE%B0/git/%E9%9B%86%E4%B8%AD%E5%BC%8F%E5%B0%8F%E7%BA%A2rebase.png>)

小红用[`git pull`](https://www.atlassian.com/git/tutorial/remote-repositories#!pull)合并上游的修改到自己的仓库中。这条命令类似`svn update`——拉取所有上游提交命令到小红的本地仓库，并尝试和她的本地修改合并：

```
git pull --rebase origin master
```

`--rebase`选项告诉`Git`把小红的提交移到同步了中央仓库修改后的`master`分支的顶部，如下图所示：

![](<https://github.com/xinyuan960205/pic_resource/raw/master/%E5%AD%A6%E4%B9%A0%E5%B0%8F%E8%AE%B0/git/%E9%9B%86%E4%B8%AD%E5%BC%8F%E5%B0%8F%E7%BA%A2%E5%90%8C%E6%AD%A5.png>)

如果你忘加了这个选项，`pull`操作仍然可以完成，但每次`pull`操作要同步中央仓库中别人修改时，提交历史会以一个多余的『合并提交』结尾。对于集中式工作流，最好是使用`rebase`而不是生成一个合并提交。

##### 小红解决合并冲突

![](https://github.com/xinyuan960205/pic_resource/raw/master/学习小记/git/集中式小红解决冲突.png)

`rebase`操作过程是把本地提交一次一个地迁移到更新了的中央仓库`master`分支之上。这意味着可能要解决在迁移某个提交时出现的合并冲突，而不是解决包含了所有提交的大型合并时所出现的冲突。这样的方式让你尽可能保持每个提交的聚焦和项目历史的整洁。反过来，简化了哪里引入`Bug`的分析，如果有必要，回滚修改也可以做到对项目影响最小。

如果小红和小明的功能是相关的，不大可能在`rebase`过程中有冲突。如果有，`Git`在合并有冲突的提交处暂停`rebase`过程，输出下面的信息并带上相关的指令：

```
CONFLICT (content): Merge conflict in
```

![](<https://github.com/xinyuan960205/pic_resource/raw/master/%E5%AD%A6%E4%B9%A0%E5%B0%8F%E8%AE%B0/git/%E9%9B%86%E4%B8%AD%E5%BC%8F%E5%B0%8F%E7%BA%A2rebase2.png>)

`Git`很赞的一点是，任何人可以解决他自己的冲突。在这个例子中，小红可以简单的运行[`git status`](https://www.atlassian.com/git/tutorial/git-basics#!status)命令来查看哪里有问题。冲突文件列在`Unmerged paths`（未合并路径）一节中：

```
# Unmerged paths:# (use "git reset HEAD <some-file>..." to unstage)
# (use "git add/rm <some-file>..." as appropriate to mark resolution)
#
# both modified: <some-file>
```

接着小红编辑这些文件。修改完成后，用老套路暂存这些文件，并让[`git rebase`](https://www.atlassian.com/git/tutorial/rewriting-git-history#!rebase)完成剩下的事：

```
git add
git rebase --continue
```

要做的就这些了。`Git`会继续一个一个地合并后面的提交，如其它的提交有冲突就重复这个过程。

如果你碰到了冲突，但发现搞不定，不要惊慌。只要执行下面这条命令，就可以回到你执行`git pull --rebase`命令前的样子：

```
git rebase --abort
```

##### 小红成功发布功能

![](<https://github.com/xinyuan960205/pic_resource/raw/master/%E5%AD%A6%E4%B9%A0%E5%B0%8F%E8%AE%B0/git/%E9%9B%86%E4%B8%AD%E5%BC%8F%E5%B0%8F%E7%BA%A2%E6%88%90%E5%8A%9F%E5%8F%91%E5%B8%83.png>)

小红完成和中央仓库的同步后，就能成功发布她的修改了：

```
git push origin master
```

##### 下一站

如你所见，仅使用几个`Git`命令我们就可以模拟出传统`Subversion`开发环境。对于要从`SVN`迁移过来的团队来说这太好了，但没有发挥出`Git`分布式本质的优势。

如果你的团队适应了集中式工作流，但想要更流畅的协作效果，绝对值得探索一下功能分支工作流的收益。通过为一个功能分配一个专门的分支，能够做到一个新增功能集成到正式项目之前对新功能进行深入讨论。

### 功能分支工作流（Feature Branch Workflow）

![](<https://github.com/xinyuan960205/pic_resource/raw/master/%E5%AD%A6%E4%B9%A0%E5%B0%8F%E8%AE%B0/git/%E5%8A%9F%E8%83%BD%E5%88%86%E6%94%AF%E5%B7%A5%E4%BD%9C%E6%B5%81.png>)

一旦掌握了集中式工作流，在开发过程中可以很简单地加上功能分支，用来鼓励开发者之间协作和简化交流。理解功能分支工作流是掌握Gitflow工作流的基础，请开发人员务必掌握。

功能分支工作流背后的核心思路是所有的功能开发应该在一个专门的分支，而不是在`master`分支上。这个隔离可以方便多个开发者在各自的功能上开发而不会弄乱主干代码。另外，也保证了`master`分支的代码一定不会是有问题的，极大有利于集成环境。

功能开发隔离也让`pull requests`工作流成为可能，`pull requests`工作流能为每个分支发起一个讨论，在分支合入正式项目之前，给其它开发者有表示赞同的机会。另外，如果你在功能开发中有问题卡住了，可以开一个`pull requests`来向同学们征求建议。这些做法的重点就是，`pull requests`让团队成员之间互相评论工作变得非常方便！

#### 工作方式

功能分支工作流仍然用中央仓库，并且`master`分支还是代表了正式项目的历史。但不是直接提交本地历史到各自的本地`master`分支，开发者每次在开始新功能前先创建一个新分支。功能分支应该有个有描述性的名字，比如`animated-menu-items`或`issue-#1061`，这样可以让分支有个清楚且高聚焦的用途。

在`master`分支和功能分支之间，`Git`是没有技术上的区别，所以开发者可以用和集中式工作流中完全一样的方式编辑、暂存和提交修改到功能分支上。

另外，功能分支也可以（且应该）`push`到中央仓库中。这样不修改正式代码就可以和其它开发者分享提交的功能。由于`master`仅有的一个『特殊』分支，在中央仓库上存多个功能分支不会有任何问题。当然，这样做也可以很方便地备份各自的本地提交。

#### 拉取请求（Pull Requests）

功能分支除了可以隔离功能的开发，也使得通过`Pull Requests`讨论变更成为可能。一旦某个开发完成一个功能，不是立即合并到`master`，而是`push`到中央仓库的功能分支上并发起一个`Pull Request`请求去合并修改到`master`。在修改成为主干代码前，这让其它的开发者有机会先去`Review`变更。

`Code Review`是`Pull Requests`的一个重要的收益，但`Pull Requests`目的是讨论代码一个通用方式。你可以把`Pull Requests`作为专门给某个分支的讨论。这意味着可以在更早的开发过程中就可以进行`Code Review`。比如，一个开发者开发功能需要帮助时，要做的就是发起一个`Pull Request`，相关的人就会自动收到通知，在相关的提交旁边能看到需要帮助解决的问题。

一旦`Pull Request`被接受了，发布功能要做的就和集中式工作流就很像了。首先，确定本地的`master`分支和上游的`master`分支是同步的。然后合并功能分支到本地`master`分支并`push`已经更新的本地`master`分支到中央仓库。

仓库管理的产品解决方案像`Bitbucket`或`Stash`，可以良好地支持`Pull Requests`。可以看看`Stash`的`Pull Requests`文档。

#### 示例

下面的示例演示了如何把`Pull Requests`作为`Code Review`的方式，但注意`Pull Requests`可以用于很多其它的目的

##### 小红开始开发一个新功能

![](https://github.com/xinyuan960205/pic_resource/raw/master/%E5%AD%A6%E4%B9%A0%E5%B0%8F%E8%AE%B0/git/%E5%8A%9F%E8%83%BD%E5%88%86%E6%94%AF%E5%B0%8F%E7%BA%A2%E5%BC%80%E5%8F%911.png)

在开始开发功能前，小红需要一个独立的分支。使用下面的命令新建一个分支：

```
git checkout -b marys-feature master
```

这个命令检出一个基于`master`名为`marys-feature`的分支，`Git`的`-b`选项表示如果分支还不存在则新建分支。这个新分支上，小红按老套路编辑、暂存和提交修改，按需要提交以实现功能：

```
git statusgit addgit commit
```

##### 小红要去吃个午饭

![](<https://github.com/xinyuan960205/pic_resource/raw/master/%E5%AD%A6%E4%B9%A0%E5%B0%8F%E8%AE%B0/git/%E5%8A%9F%E8%83%BD%E5%88%86%E6%94%AF%E5%B0%8F%E7%BA%A2%E5%8D%88%E9%A5%AD.png>)

早上小红为新功能添加一些提交。去吃午饭前，`push`功能分支到中央仓库是很好的做法，这样可以方便地备份，如果和其它开发协作，也让他们可以看到小红的提交。

```
git push -u origin marys-feature
```

这条命令`push` `marys-feature`分支到中央仓库（`origin`），`-u`选项设置本地分支去跟踪远程对应的分支。设置好跟踪的分支后，小红就可以使用`git push`命令省去指定推送分支的参数。

##### 小红完成功能开发

![](<https://github.com/xinyuan960205/pic_resource/raw/master/%E5%AD%A6%E4%B9%A0%E5%B0%8F%E8%AE%B0/git/%E5%8A%9F%E8%83%BD%E5%88%86%E6%94%AF%E5%B0%8F%E7%BA%A2%E5%AE%8C%E6%88%90.png>)

小红吃完午饭回来，完成整个功能的开发。[在合并到`master`之前](https://www.atlassian.com/git/tutorial/git-branches#!merge)，她发起一个`Pull Request`让团队的其它人知道功能已经完成。但首先，她要确认中央仓库中已经有她最近的提交：

```
git push
```

然后，在她的`Git` `GUI`客户端中发起`Pull Request`，请求合并`marys-feature`到`master`，团队成员会自动收到通知。`Pull Request`很酷的是可以在相关的提交旁边显示评注，所以你可以对某个变更提问。

##### 小黑收到Pull Request

![](<https://github.com/xinyuan960205/pic_resource/raw/master/%E5%AD%A6%E4%B9%A0%E5%B0%8F%E8%AE%B0/git/%E5%8A%9F%E8%83%BD%E5%88%86%E6%94%AF%E5%B0%8F%E9%BB%91%E6%94%B6%E5%88%B0pull.png>)

小黑收到了`Pull Request`后会查看`marys-feature`的修改。决定在合并到正式项目前是否要做些修改，且通过`Pull Request`和小红来回地讨论。

##### 小红再做修改

![](<https://github.com/xinyuan960205/pic_resource/raw/master/%E5%AD%A6%E4%B9%A0%E5%B0%8F%E8%AE%B0/git/%E5%8A%9F%E8%83%BD%E5%88%86%E6%94%AF%E5%B0%8F%E7%BA%A2%E4%BF%AE%E6%94%B9.png>)

要再做修改，小红用和功能第一个迭代完全一样的过程。编辑、暂存、提交并`push`更新到中央仓库。小红这些活动都会显示在`Pull Request`上，小黑可以断续做评注。

如果小黑有需要，也可以把`marys-feature`分支拉到本地，自己来修改，他加的提交也会一样显示在`Pull Request`上。

##### 小红发布它的功能

 ![](<https://github.com/xinyuan960205/pic_resource/raw/master/%E5%AD%A6%E4%B9%A0%E5%B0%8F%E8%AE%B0/git/%E5%8A%9F%E8%83%BD%E5%88%86%E6%94%AF%E5%B0%8F%E7%BA%A2%E5%8F%91%E5%B8%83.png>)

一旦小黑可以的接受`Pull Request`，就可以合并功能到稳定项目代码中（可以由小黑或是小红来做这个操作）：

```
git checkout master
git pull
git pull origin marys-feature
git push
```

无论谁来做合并，首先要检出`master`分支并确认是它是最新的。然后执行`git pull origin marys-feature`合并`marys-feature`分支到和已经和远程一致的本地`master`分支。你可以使用简单`git merge marys-feature`命令，但前面的命令可以保证总是最新的新功能分支。最后更新的`master`分支要重新`push`回到`origin`。

这个过程常常会生成一个合并提交。有些开发者喜欢有合并提交，因为它像一个新功能和原来代码基线的连通符。但如果你偏爱线性的提交历史，可以在执行合并时`rebase`新功能到`master`分支的顶部，这样生成一个快进（`fast-forward`）的合并。

一些`GUI`客户端可以只要点一下『接受』按钮执行好上面的命令来自动化`Pull Request`接受过程。如果你的不能这样，至少在功能合并到`master`分支后能自动关闭`Pull Request`。

##### 与此同时，小明在做和小红一样的事

当小红和小黑在`marys-feature`上工作并讨论她的`Pull Request`的时候，小明在自己的功能分支上做完全一样的事。

通过隔离功能到独立的分支上，每个人都可以自主的工作，当然必要的时候在开发者之间分享变更还是比较繁琐的。

##### 下一站

到了这里，但愿你发现了功能分支可以很直接地在集中式工作流的仅有的`master`分支上完成多功能的开发。另外，功能分支还使用了`Pull Request`，使得可以在你的版本控制`GUI`客户端中讨论某个提交。

功能分支工作流是开发项目异常灵活的方式。问题是，有时候太灵活了。对于大型团队，常常需要给不同分支分配一个更具体的角色。`Gitflow`工作流是管理功能开发、发布准备和维护的常用模式。

### Gitflow工作流

![](<https://github.com/xinyuan960205/pic_resource/raw/master/%E5%AD%A6%E4%B9%A0%E5%B0%8F%E8%AE%B0/git/gitflow%E5%B7%A5%E4%BD%9C%E6%B5%81.png>)



这节介绍的`Gitflow`工作流借鉴自在nvie的*Vincent Driessen*。

> 在项目没有特殊说明的情况下，团队一律采用Gitflow工作流。

`Gitflow`工作流定义了一个围绕项目发布的严格分支模型。虽然比功能分支工作流复杂几分，但提供了用于一个健壮的用于管理大型项目的框架。

`Gitflow`工作流没有用超出功能分支工作流的概念和命令，而是为不同的分支分配一个很明确的角色，并定义分支之间如何和什么时候进行交互。除了使用功能分支，在做准备、维护和记录发布也使用各自的分支。当然你可以用上功能分支工作流所有的好处：`Pull Requests`、隔离实验性开发和更高效的协作。

#### 工作方式

`Gitflow`工作流仍然用中央仓库作为所有开发者的交互中心。和其它的工作流一样，开发者在本地工作并`push`分支到要中央仓库中。

##### 历史分支

相对使用仅有的一个`master`分支，`Gitflow`工作流使用2个分支来记录项目的历史。`master`分支存储了正式发布的历史，而`develop`分支作为功能的集成分支。这样也方便`master`分支上的所有提交分配一个版本号。

![](<https://github.com/xinyuan960205/pic_resource/raw/master/%E5%AD%A6%E4%B9%A0%E5%B0%8F%E8%AE%B0/git/gitflow%E5%8E%86%E5%8F%B2%E5%88%86%E6%94%AF.png>)

剩下要说明的问题围绕着这2个分支的区别展开。

##### 功能分支

每个新功能位于一个自己的分支，这样可以`push`到中央仓库以备份和协作。但功能分支不是从`master`分支上拉出新分支，而是使用`develop`分支作为父分支。当新功能完成时，合并回`develop`分支。新功能提交应该从不直接与`master`分支交互。

![](<https://github.com/xinyuan960205/pic_resource/raw/master/%E5%AD%A6%E4%B9%A0%E5%B0%8F%E8%AE%B0/git/gitflow%E5%8A%9F%E8%83%BD%E5%88%86%E6%94%AF.png>)

 注意，从各种含义和目的上来看，功能分支加上`develop`分支就是功能分支工作流的用法。但`Gitflow`工作流没有在这里止步。

##### 发布分支

![](<https://github.com/xinyuan960205/pic_resource/raw/master/%E5%AD%A6%E4%B9%A0%E5%B0%8F%E8%AE%B0/git/gitflow%E5%8F%91%E5%B8%83%E5%88%86%E6%94%AF.png>)

一旦`develop`分支上有了做一次发布（或者说快到了既定的发布日）的足够功能，就从`develop`分支上`fork`一个发布分支。新建的分支用于开始发布循环，所以从这个时间点开始之后新的功能不能再加到这个分支上 —— 这个分支只应该做`Bug`修复、文档生成和其它面向发布任务。一旦对外发布的工作都完成了，发布分支合并到`master`分支并分配一个版本号打好`Tag`。另外，这些从新建发布分支以来的做的修改要合并回`develop`分支。

使用一个用于发布准备的专门分支，使得一个团队可以在完善当前的发布版本的同时，另一个团队可以继续开发下个版本的功能。
这也打造定义良好的开发阶段（比如，可以很轻松地说，『这周我们要做准备发布版本4.0』，并且在仓库的目录结构中可以实际看到）。

常用的分支约定：

```
用于新建发布分支的分支: develop
用于合并的分支: master
分支命名: release-* 或 release/*
```

##### 维护分支

![](https://github.com/xinyuan960205/pic_resource/raw/master/%E5%AD%A6%E4%B9%A0%E5%B0%8F%E8%AE%B0/git/gitflow%E7%BB%B4%E6%8A%A4%E5%88%86%E6%94%AF.png)

维护分支或说是热修复（`hotfix`）分支用于生成快速给产品发布版本（`production releases`）打补丁，这是唯一可以直接从`master`分支`fork`出来的分支。修复完成，修改应该马上合并回`master`分支和`develop`分支（当前的发布分支），`master`分支应该用新的版本号打好`Tag`。

为`Bug`修复使用专门分支，让团队可以处理掉问题而不用打断其它工作或是等待下一个发布循环。你可以把维护分支想成是一个直接在`master`分支上处理的临时发布。

#### 示例

下面的示例演示本工作流如何用于管理单个发布循环。假设你已经创建了一个中央仓库。

##### 创建开发分支

![](<https://github.com/xinyuan960205/pic_resource/raw/master/%E5%AD%A6%E4%B9%A0%E5%B0%8F%E8%AE%B0/git/gitflow%E5%88%9B%E5%BB%BA%E5%88%86%E6%94%AF.png>)



第一步为`master`分支配套一个`develop`分支。简单来做可以[本地创建一个空的`develop`分支](https://www.atlassian.com/git/tutorial/git-branches#!branch)，`push`到服务器上：

```
`git branch develop``git push -u origin develop`
```

以后这个分支将会包含了项目的全部历史，而`master`分支将只包含了部分历史。其它开发者这时应该克隆中央仓库，建好`develop`分支的跟踪分支：

```
`git clone ssh:``//user@host/path/to/repo.git``git checkout -b develop origin/develop`
```

现在每个开发都有了这些历史分支的本地拷贝。

##### 小红和小明开始开发新功能

![](<https://github.com/xinyuan960205/pic_resource/raw/master/%E5%AD%A6%E4%B9%A0%E5%B0%8F%E8%AE%B0/git/gitflow%E5%BC%80%E5%8F%91%E6%96%B0%E5%8A%9F%E8%83%BD.png>)

这个示例中，小红和小明开始各自的功能开发。他们需要为各自的功能创建相应的分支。新分支不是基于`master`分支，而是应该基于`develop`分支：

```
`git checkout -b some-feature develop`
```

他们用老套路添加提交到各自功能分支上：编辑、暂存、提交：

```
`git status``git add``git commit`
```

##### 小红完成功能开发

![](https://github.com/xinyuan960205/pic_resource/raw/master/%E5%AD%A6%E4%B9%A0%E5%B0%8F%E8%AE%B0/git/gitflow%E5%B0%8F%E7%BA%A2%E5%AE%8C%E6%88%90%E5%BC%80%E5%8F%91.png)

 添加了提交后，小红觉得她的功能OK了。如果团队使用`Pull Requests`，这时候可以发起一个用于合并到`develop`分支。否则她可以直接合并到她本地的`develop`分支后`push`到中央仓库：

```
`git pull origin develop``git checkout develop``git merge some-feature``git push``git branch -d some-feature`
```

第一条命令在合并功能前确保`develop`分支是最新的。注意，功能决不应该直接合并到`master`分支。冲突解决方法和[集中式工作流](http://blog.jobbole.com/76847/)一样。

##### 小红开始准备发布

![](https://github.com/xinyuan960205/pic_resource/raw/master/%E5%AD%A6%E4%B9%A0%E5%B0%8F%E8%AE%B0/git/gitflow%E5%B0%8F%E7%BA%A2%E5%87%86%E5%A4%87%E5%8F%91%E5%B8%83.png)

这个时候小明正在实现他的功能，小红开始准备她的第一个项目正式发布。像功能开发一样，她用一个新的分支来做发布准备。这一步也确定了发布的版本号：

```
`git checkout -b release-``0.1` `develop`
```

这个分支是清理发布、执行所有测试、更新文档和其它为下个发布做准备操作的地方，像是一个专门用于改善发布的功能分支。

只要小红创建这个分支并`push`到中央仓库，这个发布就是功能冻结的。任何不在`develop`分支中的新功能都推到下个发布循环中。

##### 小红完成发布

![](<https://github.com/xinyuan960205/pic_resource/raw/master/%E5%AD%A6%E4%B9%A0%E5%B0%8F%E8%AE%B0/git/gitflow%E5%B0%8F%E7%BA%A2%E5%AE%8C%E6%88%90%E5%8F%91%E5%B8%83.png>)

一旦准备好了对外发布，小红合并修改到`master`分支和`develop`分支上，删除发布分支。合并回`develop`分支很重要，因为在发布分支中已经提交的更新需要在后面的新功能中也要是可用的。另外，如果小红的团队要求`Code Review`，这是一个发起`Pull Request`的理想时机。

```
`git checkout master``git merge release-``0.1``git push``git checkout develop``git merge release-``0.1``git push``git branch -d release-``0.1`
```

发布分支是作为功能开发（`develop`分支）和对外发布（`master`分支）间的缓冲。只要有合并到`master`分支，就应该打好`Tag`以方便跟踪。

```
`git tag -a ``0.1` `-m ``"Initial public release"` `master``git push --tags`
```

`Git`有提供各种勾子（`hook`），即仓库有事件发生时触发执行的脚本。可以配置一个勾子，在你`push`中央仓库的`master`分支时，自动构建好对外发布。

##### 最终用户发现Bug

![](<https://github.com/xinyuan960205/pic_resource/raw/master/%E5%AD%A6%E4%B9%A0%E5%B0%8F%E8%AE%B0/git/gitflow%E5%8F%91%E7%8E%B0bug.png>)

对外发布后，小红回去和小明一起做下个发布的新功能开发，直到有最终用户开了一个`Ticket`抱怨当前版本的一个`Bug`。为了处理`Bug`，小红（或小明）从`master`分支上拉出了一个维护分支，提交修改以解决问题，然后直接合并回`master`分支：

```
`git checkout -b issue-#``001` `master``# Fix the bug``git checkout master``git merge issue-#``001``git push`
```

就像发布分支，维护分支中新加这些重要修改需要包含到`develop`分支中，所以小红要执行一个合并操作。然后就可以安全地[删除这个分支](https://www.atlassian.com/git/tutorial/git-branches#!branch)了：

```
`git checkout develop``git merge issue-#``001``git push``git branch -d issue-#``001`
```

## 四、git拉取请求

Git拉取请求（Pull Request），其含义就是请求管理员（项目经理）将你所做的修改提交拉取到正式代码中。根据[Git工作流程](http://140.143.45.184/wiki/pages/viewpage.action?pageId=2982308)，当任务完成时，需要提交审核，这时需要发起拉取请求（Pull Request）。另外，拉取请求还可以被用于讨论代码中技术问题。

### 通过拉取请求提交工作结果

#### 拉取请求的发起

当发起拉取请求时，应该将一个git仓库中对应一个（jira）任务的所有提交一起汇总到一个拉取请求中提交审核。

在jira工作流中，这会自动的将相关的jira任务迁移到“待审核”状态，表示开发者完成了自己的工作。

#### 审核人的选择

审核人的选择按照以下要求执行：

1. 任何拉取请求，至少需要添加默认审核人进行评审。
2. 任务参与人发起拉取请求时，除了默认审核人，需要额外增加任务负责人到审核人中。
3. 子任务负责人发起拉去请求时，除了默认审核人，需要额外增加父任务负责人到审核人中。

## 五、git提交内容和.gitignore

一般来说，允许提交到Git服务器管理的文件要求是**非自动生成的文件**，尽可能只管理**文本文件**。

常见的不允许提交的文件包括：

1. 编译目标文件，包括静态库，动态库，可执行文件等。
2. 编译中间生成文件
3. IDE的工程文件/文件夹
4. 自动备份文件
5. 一般的二进制文件（需要具体情况具体判断）
6. 临时文件

对于不允许提交到Git服务器的文件，使用.gitignore设置忽略。典型的[.gitignore](http://140.143.45.184/wiki/download/attachments/2982311/.gitignore?version=1&modificationDate=1485239900000&api=v2)文件如下所示：

```
`.idea``cmake-build-*``*.dll``*.so``*.a``*.lib``*.bin``*.exe``.ipynb_checkpoints``*.pyc``*~``*.asv``*.log``*.aux``*.bcf``*.run.xml``*.bbl``*.blg``*.ttf``*.ttc``*.dat``*.lock``*~``*.sln``*.suo``*.dsp``*.vcxproj``*.vcxproj.*``*.rc``*.aps``*.rc2``*.pro.user``*.pro``*.pri``*.qrc``*.db``*.opt`
```























































  

























