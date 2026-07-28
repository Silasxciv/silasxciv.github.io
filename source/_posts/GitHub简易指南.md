---
layout: post
title: GitHub简易指南
comments: true
date: 2018-03-16 08:46:40
tags:
- GitHub
categories: 
  - 教程
---
![](/assets/images/180316.jpg)
前几天知名帅哥stormzhang在其公众号共享了一份程序员大礼包，
即各种教程的PDF文档，其中有一篇《从0开始学习GitHub系列》，
记录其中的常用命令行指令，以便日后查阅。
<!--more-->
## GitHub和Git的区别
GitHub是一个基于Git的开源代码托管社区，即可以通过Git来将代码上传到GitHub云端，或者将GitHub云端的程序下载到本地进行修改的一个服务。
Git是一个分布式版本控制器，即一个代码管理工具，用法如同步代码、查看更改、代码还原等。用前需安装。

## Git本地命令

``` bash
$ git init   #初始化Git仓库
```
``` bash
$ git status   #查看Git仓库当前状态
```
``` bash
$ git add readme.md    #将readme.md文件放入暂存区，等待提交
```
``` bash
$ git commit -m "first"    #提交暂存区的文件，并注释为first，-m代表提交信息
```
``` bash
$ git log   #查看所有提交记录
```
``` bash
$ git branch develop   #新建名为develop的分支
```
``` bash
$ git checkout develop   #切换到develop分支
```
``` bash
$ git checkout -b develop   #新建的同时转到develop分支
```
``` bash
$ git merge develop    #将develop分支合并到当前分支
```
``` bash
$ git branch -d develop   #删除develop分支
```
``` bash
$ git branch -D develop   #强制删除develop分支
```
``` bash
$ git tag v1.0   #贴一个名为v1.0的标签
```
``` bash
$ git checkout v1.0   #切换到v1.0标签
```

## Git同步命令
``` bash
$ ssh-keygen -t rsa   #生成密钥
```
通过密钥链接本地与GitHub仓库，连按三次回车，生成密钥id_rsa和公钥id_rsa.pub，将公钥添加到GitHub。
``` bash
git config —global user.name "用户名"
git config —global user.email "邮箱"
```
设置自己的用户名与邮箱，信息会出现在所有的提交记录里。
``` bash
$ git push origin master   #将本地代码推到GitHub中默认分支
```
``` bash
$ git pull origin master   #将GitHub默认分支中代码更新到本地
```
``` bash
$ git diff   #查看当前文件和暂存区文件差异，
$ git diff <$id1> <$id2>   #比较两次提交之间的差异
$ git diff <branch1>..<branch2>   #在两个分支之间比较
$ git diff --staged   #比较暂存区和版本库差异
```
``` bash
$ git stash   #暂存没有提交的代码
```
``` bash
$ git stash list   #查看暂存区记录
```
``` bash
$ git stash   #查看暂存区记录
```
``` bash
$ git stash apply   #还原代码
```
``` bash
$ git stash drop   #删除暂存区记录
```
``` bash
$ git stash clear   #清空所有暂存区记录
```
``` bash
$ git rebase featureA   #合并代码
```

## GitHub简单用法
1. 创建仓库；
2. 下载到本地；
3. 修改文件；
4. add到暂存区；
5. commit提交；
6. push到GitHub。