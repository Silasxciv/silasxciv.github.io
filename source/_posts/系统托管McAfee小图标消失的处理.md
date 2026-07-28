---
layout: post
title: 系统托管McAfee小图标消失的处理
comments: true
tags:
  - McAfee
categories: 
  - 教程
date: 2020-06-30 20:09:54
---
McAfee的小图标不见了，但任务管理器能看到相关进程，那就重新让它显示吧。
<!--more-->

1. 检查任务管理器，看代理服务是不是正在运行。
![](/assets/images/200630_1.jpg)

2. 打开McAfee存放cmdagent.exe应用程序的文件夹，路径通常为C:\Program Files\McAfee\Agent。
![](/assets/images/200630_2.jpg)

3. 进入命令行程序，且此时命令行执行路径为该文件夹。
![](/assets/images/200630_3.jpg)
 
4. 英文输入下，在CMD命令行中键入”cmdagent.exe” -s，即可弹出McAfee代理监视器界面，同时系统托盘中出现了McAfee的代理小图标。
![](/assets/images/200630_4.jpg)

5. 结束，此时系统托盘的代理小图标就出现了。