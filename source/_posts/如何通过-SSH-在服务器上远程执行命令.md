---
title: 如何通过 SSH 在服务器上远程执行命令
date: 2023-03-14 10:38:04
tags:
  - shell
categories:
  - Shell脚本
---

## 背景
最近买了一个腾讯云服务器，主要用来托管个人项目，经常执行的操作是ssh登陆到服务器，然后执行某些命令。这里有两个步骤： 
1. 打开ssh登陆软件，然后登陆到服务器
2. 执行相关命令
3. 关闭连接

那有没有办法一气呵成呢？一个命令完成这两个操作，查了下资料，当然可以：

```bash
ssh -t username@host 'commands'
```

比如我想看下 root 目录有哪些文件：
```bash
ssh -i ~/.ssh/tencent_ecs root@example.com "ls -a"
```
![](https://lilong7676-picture.oss-cn-hangzhou.aliyuncs.com/img/20230314105027.png?x-oss-process=image/resize,w_800)
<!-- more -->

## reference
[SSH tip: Send commands remotely](https://www.cnet.com/tech/computing/ssh-tip-send-commands-remotely/)