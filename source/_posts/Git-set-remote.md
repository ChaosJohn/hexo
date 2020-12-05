---
title: git 设置远端仓库
date: 2020-12-04 17:27:41
tags: [git]
thumbnail: https://image.blog.chaosjohn.com/Git-set-remote/banner.png
---

[原文链接](https://blog.chaosjohn.com/Git-set-remote.html)

前段时间，公司开了一个新项目，买了另一家公司的源码做二次开发。

项目进行了几天后，我突然听到参与开发的几个同学在讨论，关于 “不想把我们修改的版本推给他们”。

我就顿感奇怪，买了源码还要遵循他们的开源协议？

我跑过去问问怎么回事，一听就乐了。原来对方公司将代码部署在私有 git 服务器上，给了我们账号密码以供拉取源码。对方承诺对产品做后续的更新维护，新版本也发布在该 git 仓库上。但是我们对源码做二次开发，会进行很多改动，又不想把我们的改动推给他们。

啊这。。。明显是对 `git` 不熟悉啊，而且还不是一个同学，应该值得反省。平日里起草招聘需求时都会把 `git` 作为一个必备的技能项，结果轮到自己身上，却只略知皮毛。

我先代入他们的思维反过来推理：为什么一定要 `push`，他们的代码只做 `pull`，拉取新版合并到本地不就行了么。哦，原来不止一个小伙伴在协同开发，相互间要共享改动，比如 **A同学** 的改动 `push` 后 **B同学** `pull` 后才能看到。那 `push` 后不就推送到了对方公司的 `git` 服务器了么。

所以小伙伴们还停留在一个 `git` 仓库只有一个远端的层面。

其实一个 `git` 仓库是可以配置多个远端 `remote`。

---

我们举个例子来模拟一下，我们从 `github` 上随便 `clone` 一个项目下来
```
$ git clone git@github.com:taniarascia/takenote.git
Cloning into 'takenote'...
remote: Enumerating objects: 114, done.
remote: Counting objects: 100% (114/114), done.
remote: Compressing objects: 100% (92/92), done.
remote: Total 4672 (delta 44), reused 50 (delta 20), pack-reused 4558
Receiving objects: 100% (4672/4672), 10.13 MiB | 780.00 KiB/s, done.
Resolving deltas: 100% (2990/2990), done.
```

我们查看一下 `git配置`：
```
$ cd takenote
$ cat .git/config
[core]
  repositoryformatversion = 0
  filemode = true
  bare = false
  logallrefupdates = true
  ignorecase = true
  precomposeunicode = true
[remote "origin"]
  url = git@github.com:taniarascia/takenote.git
  fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
  remote = origin
  merge = refs/heads/master
```

我们可以看到
- 只有一个 `远端(remote)` - *origin*，并且指向了 **git@github.com:taniarascia/takenote.git**
- 只有一个 `分支(master)`

---

接下来去 [码云Gitee](https://gitee.com/) 上去创建一个 **空的** 私有仓库，模拟存放我们的 *已修改源码* 
![创建一个空仓库](https://image.blog.chaosjohn.com/Git-set-remote/create-empty-repository.png)

创建后会跳转到仓库主页
![初始化空仓库](https://image.blog.chaosjohn.com/Git-set-remote/init-empty-repository.png)

我们可以看到，对于已存在的本地仓库，是可以直接推送到 `码云` 的
```
git remote add origin git@gitee.com:ChaosJohn/takenote.git
git push -u origin master
```

解释一下第一行：添加一个 `远端(remote)`，取名为 `origin`，设置 **远端地址** 为 `git@gitee.com:ChaosJohn/takenote.git`

可以预见，如果直接执行，肯定会报错，先试试：
```
$ git remote add origin git@gitee.com:ChaosJohn/takenote.git
fatal: remote origin already exists.
```

报错提示说：名叫 `origin` 的远端已存在。

那咱就换一个呗，不如取名 `gitee` 吧，正好寓意这个远端是 **码云** 的，再次执行 `git remote add gitee git@gitee.com:ChaosJohn/takenote.git` 无报错。

查看一下配置文件，发现比之前的多了三行
```
$ cat .git/config
[core]
  repositoryformatversion = 0
  filemode = true
  bare = false
  logallrefupdates = true
  ignorecase = true
  precomposeunicode = true
[remote "origin"]
  url = git@github.com:taniarascia/takenote.git
  fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
  remote = origin
  merge = refs/heads/master
[remote "gitee"]
  url = git@gitee.com:ChaosJohn/takenote.git
  fetch = +refs/heads/*:refs/remotes/gitee/*
```

其实直接将这三行添加到 `.git/config` 文件内，等同与执行 `git remote add` 命令。

---

所以正确的 git 工作流程为：
1. 从当前 `master` 分支创建新的 **开发分支** `git checkout -b dev`
2. 在 `dev` 分支上做二次开发，提交并且推送到 *码云远端* `git push -u gitee dev`
3. 如果原项目出更新内容了，先切回到 `master` 分支，然后从 **github远端** 拉取新代码 `git pull origin master`
4. 再将 `master` 分支合并到 `dev` 分支
5. 切换到 `dev` 分支，继续做二次开发以及推送到码云，如此循环迭代

---

所以建议所有开发的小伙伴们，有时间要多去学习和熟悉 `git`。毕竟 **工欲善其事 必先利其器** 嘛！！！

最后，如果该文对读者有些许帮助，考虑下给点捐助鼓励一下呗😊
![](https://image.blog.chaosjohn.com/donate-me.png)