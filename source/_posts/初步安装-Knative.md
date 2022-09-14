---
title: 初步安装 knative
date: 2022-09-08 10:02:11
tags:
categories: 
 - Knative
---



参考资料：
https://knative.dev/docs/getting-started/quickstart-install/#before-you-begin

注意，由于我在虚拟机里走了vpn代理，所以以下命令都没有使用国内源。


前提条件：
1. 安装了 minikube、kubectl，参考 [在 vmware centos7 中安装 minikube](https://lilong7676.github.io/2022/09/08/minikube/k8s/%E5%9C%A8-vmware-centos7-%E4%B8%AD%E6%8E%A2%E7%B4%A2-k8s-%E4%B8%8E-KNative/).

<!-- more -->

# 1. 安装 Knative CLI （kn）([参考连接](https://knative.dev/docs/getting-started/quickstart-install/#install-the-knative-cli))
```bash
# 下载二进制包
$ curl -fsSL https://github.com/knative/client/releases/download/knative-v1.7.0/kn-linux-amd64 -O

# 重命名二进制包名为 kn
$ mv kn-linux-amd64 kn
$ chmod +x kn

# 移动 kn 到 bin 目录
$ sudo mv kn /usr/local/bin/

# 查看是否安装成功

$ kn version

```
![kn version](https://raw.githubusercontent.com/lilong7676/Picture/master/blog/image/20220914105352.png)

# 2. 安装 Knative quickstart 插件 ([参考链接](https://knative.dev/docs/getting-started/quickstart-install/#install-the-knative-quickstart-plugin))

```bash
# 下载 kn-quickstart-linux-amd64 二进制文件
$ curl -fsSL https://github.com/knative-sandbox/kn-plugin-quickstart/releases/download/knative-v1.7.1/kn-quickstart-linux-amd64 -O

# 重命名为 kn-quickstart
$ mv kn-quickstart-linux-amd64 kn-quickstart
$ chmod +x kn-quickstart


# 移动 kn-quickstart 到 bin 目录
$ sudo mv kn-quickstart /usr/local/bin/

# 验证是否安装成功
$ kn quickstart --help
```
![kn quickstart --help](https://raw.githubusercontent.com/lilong7676/Picture/master/blog/image/20220914110252.png)

# 3.运行 Knative quickstart 插件 （[参考](https://knative.dev/docs/getting-started/quickstart-install/#run-the-knative-quickstart-plugin)）

`quickstart` 插件会完成以下功能：  
1. 检查你是否已经安装了所选的 k8s 实例
2. 创建一个集群，名字为 `knative`
3. 安装 Knative Serving，使用 `Kourier` 作为默认的网络层，`sslip.io`作为 DNS
4. 安装 Knative Eventing，并且创建一个 in-memory Broker 和 Channel 的实现


为了在本地部署 Knative，需要运行 `quickstart` 插件:  
由于本地已经安装了 `minikube`，所以这里以 `minikube` 启动插件：  

1. 在 `minikube` 实例中安装 Knative 和 k8s:
> 注意：minikube 集群将使用 6GB 内存创建。如果没有足够的内存，则可以在该命令之前运行命令 `minikube config set memory 3078`，将其更改为不低于 `3 GB` 的其他值。


```bash
$ kn quickstart minikube
```
![kn quickstart minikube](https://raw.githubusercontent.com/lilong7676/Picture/master/blog/image/20220914145319.png)

2. 根据上一步中的提示，新开一个 terminal 窗口，执行:
```bash
$ minikube tunnel --profile knative
```
在使用 Knative `quickstart` 环境时，终端里必须运行此 `tunnel`

3. 回到上一个窗口，按 `Enter` 键继续安装

4. 验证安装是否成功
```bash
minikube profile list
```
![minikube profile list](https://raw.githubusercontent.com/lilong7676/Picture/master/blog/image/20220914145849.png)

# 4. 部署一个 Knative Service ([参考](https://knative.dev/docs/getting-started/first-service/#deploying-a-knative-service))

有两种方式部署，可选 `kn` 命令部署 或者编写 YAML 配置文件并使用 `kubectl apply` 部署，具体可参考官网。

1. 使用 `kn` 部署
```bash
$ kn service create hello \
--image gcr.io/knative-samples/helloworld-go \
--port 8080 \
--env TARGET=World
```