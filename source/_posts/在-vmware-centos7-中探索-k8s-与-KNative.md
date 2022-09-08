---
title: 在 vmware centos7 中安装 minikube
date: 2022-09-08 10:02:11
tags:
categories: 
 - minikube
 - k8s
---



参考资料：
https://minikube.sigs.k8s.io/docs/start/  
https://docs.docker.com/engine/install/centos/#install-using-the-convenience-script

注意，由于我在虚拟机里走了vpn代理，所以以下命令都没有使用国内源。

<!-- more -->

### 使用 SSH 连接 VMware 上的 CentOS 虚拟机（可选）  
  - 安装 openssh-server
  ```bash
  $ yum install openssh-server
  ```


  - 启用22端口监听
  ```bash
  $ vi /etc/ssh/sshd_config
  # 取消下面两行的注释
  # Port 22
  # ListenAddress 0.0.0.0
  ```

  - 开启 sshd 服务
  ```bash
  $ sudo service sshd start
  ```

  - 检测 22 端口
  ```bash
  $ netstat -an | grep 22
  ```
  ![检测 22 端口](https://raw.githubusercontent.com/lilong7676/Picture/master/blog/image/20220908143016.png)

  - 主机与虚拟机进行互相ping，如果ping通了，表示主机和虚拟机在同一个子网，则直接 `ssh root@ip` 即可。
  - 如果没有ping通，则表示主机和虚拟机没有在一个字网，需要修改网络配置，可参考[这里](https://www.jianshu.com/p/f38133c1485a)进行配置。
  

  ### 安装 docker 和 minikube

- 安装 docker engine
  ```bash
  # https://docs.docker.com/engine/install/centos/#install-using-the-convenience-script
  $ curl -fsSL https://get.docker.com -o get-docker.sh
  $ sudo sh get-docker.sh
  ```

- 启动 docker
  ```bash
  $ systemctl start docker
  ```

- 安装 minikube
  ```bash
  # https://minikube.sigs.k8s.io/docs/start/
  $ curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
  $ sudo install minikube-linux-amd64 /usr/local/bin/minikube
  ```

- [配置 minikube(可选)](https://minikube.sigs.k8s.io/docs/drivers/docker/#requirements)
  ```bash
  # Start a cluster using the docker driver:
  minikube start --driver=docker

  # To make docker the default driver:
  minikube config set driver docker
  ```

### 启动 minikube
  ```bash
  $ minikube start
  # 如果出现下面提示，解决方案参考： https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user
  #❌  Exiting due to DRV_AS_ROOT: The "docker" driver should not be used with root privileges.

  # 执行 `minikube start`后，会下载安装 k8s等依赖，耐心等待
  ```

  #### minikube start 报错解决
  我在本地的虚拟机执行这个命令会出现类似下面的报错：
  ```
   Preparing Kubernetes v1.24.3 on Docker 20.10.17 ...
    ▪ Generating certificates and keys ...  
    ▪ Booting up control plane ...-   
   this error is likely caused by:
   - The kubelet is not running
   - The kubelet is unhealthy due to a  ....  
  ```

  可以通过安装旧版本的 k8s临时解决：
  ```bash
  $ minikube delete
  $ minikube start --kubernetes-version=v1.23.8 

  😄  minikube v1.26.1 on Centos 7.9.2009
✨  Automatically selected the docker driver. Other choices: ssh, none
📌  Using Docker driver with root privileges
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
💾  Downloading Kubernetes v1.23.8 preload ...
    > preloaded-images-k8s-v18-v1...:  400.52 MiB / 400.52 MiB  100.00% 4.66 Mi
    > index.docker.io/kicbase/sta...:  0 B [___________________] ?% ? p/s 1m44s
❗  minikube was unable to download gcr.io/k8s-minikube/kicbase:v0.0.33, but successfully downloaded docker.io/kicbase/stable:v0.0.33 as a fallback image
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
❗  This container is having trouble accessing https://k8s.gcr.io
💡  To pull new external images, you may need to configure a proxy: https://minikube.sigs.k8s.io/docs/reference/networking/proxy/
🐳  Preparing Kubernetes v1.23.8 on Docker 20.10.17 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: default-storageclass, storage-provisioner
💡  kubectl not found. If you need it, try: 'minikube kubectl -- get pods -A'
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default

  ```

  此时执行 `$ docker ps`会发现 minikube 其实就是docker中运行的容器。
  ```bash
  $ docker ps
  CONTAINER ID   IMAGE                    COMMAND                  CREATED          STATUS          PORTS                                                                                                                                  NAMES
c07efbc6d848   kicbase/stable:v0.0.33   "/usr/local/bin/entr…"   14 minutes ago   Up 14 minutes   127.0.0.1:49162->22/tcp, 127.0.0.1:49161->2376/tcp, 127.0.0.1:49160->5000/tcp, 127.0.0.1:49159->8443/tcp, 127.0.0.1:49158->32443/tcp   minikube
  ```

### 安装 kubectl
 - 可参考这里： https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

```bash
$ curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

$ sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# 查看版本验证安装是否成功
$ kubectl version --client
```