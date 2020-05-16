---
title: 第一次用Git必看
toc: true
top: true
categories:
  - Diary
tags:
  - 随笔
  - Git
description: Git
abbrlink: ff30c72a
date: 2020-05-06 17:06:26
---

# git 使用细则

## git 安装

> [官网下载地址](https://git-scm.com/)git， 安装目录尽量不要选在系统盘，更改环境变量 HOME 的的值，可以更改 git bash 的默认工作区，即“~”的位置。在 HOME 值对应的目录下，.gitconfig 文件是 git 的全局配置文件。

 <!-- more -->

## git 账户配置

```bash
git config --global user.name  "username"  #--global全局配置，用户级别
git config --global user.email "youremail@example.com"
```

## git 远程仓库使用

- git 常用的远程仓库有[github](https://github.com/),[gitee](https://gitee.com/),[gitlab](https://about.gitlab.com/)等
- git 采用 ssh 协议传输，生成传输公钥：

```bash
ssh-keygen -t rsa -C "youremail@example.com"
```

接下来系统提示命名文件名，并输入密码

```bash
Generating public/private rsa key pair.
Enter file in which to save the key (/e/Git/.ssh/id_rsa): a #<filename>默认 id_rsa
Enter passphrase (empty for no passphrase):#<password> 一般两次Enter即可
Enter same passphrase again:
Your identification has been saved in a.
Your public key has been saved in a.pub.
The key fingerprint is:......
```

- 生成的公钥是在.ssh 文件夹下的 xx.pub 文件，在 git 远程仓库账户下添加公钥。

### 远程仓库项目本地更改

#### 从远程仓库 clone

```bash
git clone <远程仓库地址>  # 结尾是.git
cd 仓库名
git config --local -l  #check the config information of local
或者 git remote -v     #查看远程仓库地址
```

- 一般是 clone 的远程仓库地址，.git 文件夹存在证明是 git 本地仓库。
  .gitignore 文件是配置 git 上传时忽略哪些文件。

#### push 到远程仓库

```bash
git add * 或 git add <filename> #添加到本地仓库暂存区（Staging area）
git commit -m "做了哪些更改"     #提交到本地仓库版本库
git remote add origin <远程仓库地址>#要push的目标远程仓库地址
git status                     #查看git工作区（ working tree）状态
git push -u origin master # 将工作区的master分支推送到远程仓库master分支
```

- -u 参数：远程仓库没有该分支即创建，并且把本地的 master 分支和远程的 master 分支关联起来，在以后的推送或者拉取时就可以简化命令。

### 本地新项目上传到远程仓库

- 在项目根目录 git bash here

```bash
git init  #初始化git版本库，生成.git文件夹
git remote add origin <远程仓库地址>
git add *
git commit -m "做了哪些更改"
git push -u origin master
```

- ##### 错误一：远程仓库已经更改
  如果此时报错如下：

```bash
! [rejected]        master -> master (fetch first)
error: failed to push some refs to 'https://github.com/SingleMY/MoYang.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

> 说明远程仓库和本地版本库比对后，有的文件只在远程仓库有，本地版本库中没有，此时需要 pull 远程仓库文件到本地版本库：

```bash
git branch --set-upstream-to=origin/master  master     #把远程分支与本地分支关联
git pull
```

- ##### 错误二：未关联分支
  如果不关联分支会报错：

```bash
There is no tracking information for the current branch.
Please specify which branch you want to merge with.
See git-pull(1) for details.

    git pull <remote> <branch>

If you wish to set tracking information for this branch you can do so with:

    git branch --set-upstream-to=origin/<branch> master
```

> 使用 git 在本地新建一个分支后，需要做远程分支关联。如果没有关联，git 会在下面的操作中提示你显示的添加关联。
> 关联目的是在执行 git pull, git push 操作时就不需要指定对应的远程分支，你只要没有显示指定，git pull 的时候，就会提示你。解决办法如上。

- ##### 错误三

```bash
fatal: refusing to merge unrelated histories
```

> 这个问题是因为两个根本不相干的 git 库，一种方法是从远端库拉下来代码，本地要加入的代码放到远端库下载到本地的库，然后提交上去因为这样的话，你基于的库就是远端的库。当然这样处理过于麻烦，可以使用强制指令：

```bash
 git pull --allow-unrelated-histories
```

然后正常 push 就可以了，以上问题一般是因为远程仓库有个 README.md 文件，**最好的解决办法就是建仓库时别勾选任何，建一个空的仓库。**

### git 版本控制技术的工作原理

![git原理](http://qiniu.1542051400.club/blog/git1.png)
