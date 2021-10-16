---
title: 解决53端口被svchost.exe占用的问题
date: 2021-10-16 15:36:00
urlname: solve_the_problem_that_port_53_is_occupied_by_svchost
tags: 
  - Windows
categories: 
  玩机
---
今天本地跑的dns服务突然报错了，跑不起来了，看下错误日志，53端口被占用了，蛋疼。  

## 1. 找到是谁占用的  
第一想法就是找到是谁在占用，干掉它就行了。这个简单，google一下就知道怎么查找端口占用的软件了，我用的ip雷达。然而，我想简单了，发现占用的进程是scvhost.exe，是windows系统进程，这可不能随便干掉。  
## 2. 找出元凶
scvhost肯定不是真正凶手，继续挖。虽然进程名字是scvhost，但这不是关键，记住这个进程的pid。打开任务管理器，点到**服务**tab，找到pid对应的服务，发现了，是**Internet Connection Sharing (ICS)**  
## 3. 干掉凶手
好了，找到是谁了，直接干掉它！打开windows的服务，找到Internet Connection Sharing (ICS)，停止。咦，报错了
```
---------------------------
服务
---------------------------
Windows 无法停止 Internet Connection Sharing (ICS) 服务(位于 本地计算机 上)。
服务并未返回错误。这可能是一个 Windows 内部错误或服务内部错误。
如果问题持续存在，请与你的系统管理员联系。
---------------------------
确定   
---------------------------
```

无法停止这个服务怎么办？Google一下，发现这个服务是跟**主机网络服务**关联的，那么先去停止**主机网络服务**。  
咦，停止了，又自动启动了。没办法，先禁用，然后停止**Internet Connection Sharing (ICS)**。好了，再看一下，53端口被释放了。好了，为了避免未知的错误，还是调过头来，把**主机网络服务**设置回`手动`。  


问题解决😎