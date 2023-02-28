---
title: '如何构建并发布自己的docker镜像 '
date: 2023-02-28 14:28:21
tags:
categories:
---


最近想在云服务器上部署一个自己的 nginx 服务，通过docker 部署最方便。之前也发布过自己的docker镜像，但是时间长了老是忘记具体的命令是啥，这里做一个笔记记录下。  

## 构建镜像

`docker cli` 提供了两个命令来完成构建\发布：
```bash
docker build # 构建镜像
docker pull/push # 拉取/推送镜像
```
整体流程参考下图：  

![docker构建发布流程](https://lilong7676-picture.oss-cn-hangzhou.aliyuncs.com/img/20230228143743.png?x-oss-process=image/resize,w_800)

### docker build 构建镜像
首先命令格式[来自文档](https://docs.docker.com/engine/reference/commandline/build/)：
```bash
docker build [OPTIONS] PATH | URL | -
```
`docker build `是利用 Dockerfile 内的指令来构建镜像。通常我们会把 Dockerfile 也加入版本管理，所以对镜像的修改操作也会有记录，这相比 `docker commit`的黑箱操作好很多。Dockerfile 就不细讲了。

<!-- more -->

下面就以发布一个自定义配置的 nginx 镜像为例子，记录一下操作步骤:
```bash
mkdir nginx && cd nginx
vi Dockerfile
```  
输入如下内容：
![](https://lilong7676-picture.oss-cn-hangzhou.aliyuncs.com/img/20230228145132.png?x-oss-process=image/resize,w_800)  

构建镜像：
```bash
docker build -t lilong7676/nginx:v1 .
```

也可以通过以下命令打标签：
```bash
docker tag local-image:tagname new-repo:tagname
# 即
docker tag nginx:v1 lilong7676/nginx:v1
```

构建完成后，查看本地镜像列表，可以看到已经打上了标签v1：
```bash
docker image list
```
![](https://lilong7676-picture.oss-cn-hangzhou.aliyuncs.com/img/20230228145327.png?x-oss-process=image/resize,w_800)


## 发布镜像
登陆 [docker hub](https://hub.docker.com/)，创建仓库
![](https://lilong7676-picture.oss-cn-hangzhou.aliyuncs.com/img/20230228150507.png?x-oss-process=image/resize,w_800)

发布镜像：
```bash
docker push lilong7676/nginx:v1
```

![](https://lilong7676-picture.oss-cn-hangzhou.aliyuncs.com/img/20230228151436.png?x-oss-process=image/resize,w_800)  

docker hub查看：

![](https://lilong7676-picture.oss-cn-hangzhou.aliyuncs.com/img/20230228151455.png?x-oss-process=image/resize,w_800)