---
title: 在 vmware centos7 中安装与探索 k8s
date: 2022-09-08 10:02:11
tags:
categories: 
 - minikube
 - k8s
---



参考资料：
https://minikube.sigs.k8s.io/docs/start/  
https://docs.docker.com/engine/install/centos/#install-using-the-convenience-script
https://www.cnblogs.com/yyee/p/15071684.html

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
-  Add your user to the 'docker' group
  ```bash
  $ sudo usermod -aG docker $USER && newgrp docker
  ```
- 安装 minikube
  ```bash
  # 国内最好走代理访问，参考自 https://minikube.sigs.k8s.io/docs/handbook/vpn_and_proxy/#macos-and-linux
  # export HTTP_PROXY="http://yourip:port" 
  # export HTTPS_PROXY="http://yourip:port"
  # export NO_PROXY=localhost,127.0.0.1,10.96.0.0/12,192.168.59.0/24,192.168.49.0/24,192.168.39.0/24

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
  # 如果已经走代理了，则直接执行
  $ minikube start 
  # 否则国内环境最好使用镜像源
  $ minikube start --image-mirror-country cn \
  --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers
  # 如果出现下面提示，解决方案参考： https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user
  #❌  Exiting due to DRV_AS_ROOT: The "docker" driver should not be used with root privileges.

  # 执行 `minikube start`后，会下载安装 k8s等依赖，耐心等待
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

# 初步使用
-  使用 `kubectl` 查看当前集群
   ```bash
   $ kubectl get po -A
   ```
   ![使用 kubectl 查看当前集群](https://raw.githubusercontent.com/lilong7676/Picture/master/blog/image/20220909113846.png)

- minikube 集成了 k8s Dashboard,可以在浏览器中查看当前集群情况
  ```bash
  $ minikube dashboard
  ```
  ![minikube dashboard](https://raw.githubusercontent.com/lilong7676/Picture/master/blog/image/20220909114717.png)

  暴露端口给外部访问：
  ```bash
  $ kubectl proxy --address='0.0.0.0' --disable-filter=true
  ```
  ![暴露端口给外部访问](https://raw.githubusercontent.com/lilong7676/Picture/master/blog/image/20220909120236.png)

  发现暴露了8001端口给外部访问，但是此时在宿主机无法打开 `
  http://centos_ip:8001/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/` ,因为防火墙的配置不允许，所以👇下面是配置防火墙的步骤。

- 配置 centos 防火墙以暴露指定端口给宿主机访问（[参考自这里](https://www.cnblogs.com/yyee/p/15071684.html)）
  
  因为我是本地用的虚拟机安装的 centos7，所以上面的 dashboard 页面无法直接在宿主机内打开，所以参考这里修改centos 防火墙即可
  
  首先前提是我的虚拟机网络走的是 NAT转发模式，以下为在虚拟机内操作：
  ```bash
  # 设置防火墙允许NAT转发
  $ sudo firewall-cmd --zone=public --add-masquerade --permanent

  # 通过 ifconfig 命令查看 127.0.0.1 属于哪个网卡，我这里网卡的名字是 lo
  # 下面把 lo 网卡添加到 trusted 域
  $ sudo firewall-cmd --permanent --zone=trusted --change-interface=lo

  # PS: 我这里顺便把 docker0 网卡也添加到信任域了，方便以后访问docker内的端口
  # sudo firewall-cmd --permanent --zone=trusted --change-interface=docker0

  # 开放 8001 端口（minikube dashboard proxy 默认为 8001 端口）
  $ sudo firewall-cmd --add-port=8001/tcp --permanent

  # 重启防火墙即可
  $ firewall-cmd --reload
  ```
  此时在宿主机即可访问虚拟机8001端口： `http://vm_ip:8001/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/#/workloads?namespace=default`


# 部署一个应用试试
这里可以参考官网的教程（https://minikube.sigs.k8s.io/docs/start/），官网部署的是:`k8s.gcr.io/echoserver:1.4`,但是一直显示pull image 失败，应该是镜像地址访问问题，所以这了换成了另一个demo镜像，你可以换成任何docker镜像。

- 根据 demo 镜像创建一个 deployment
  ```bash
  $ kubectl create deployment wordpress --image=wordpress
  ```

- 暴露 NodePort 端口到 80，注意，这里的端口必须与docker镜像暴露的端口一致，否则无法访问到镜像内
  ```bash
  $ kubectl expose deployment wordpress --type=NodePort --port=80
  ```

- 显示访问地址
  ```bash
  # 如果不加 --url，则会自动在浏览器中打开
  $ minikube services wordpress --url
  ```

  ![显示访问地址](https://raw.githubusercontent.com/lilong7676/Picture/master/blog/image/20220909160506.png)

- 一些常用命令
  ```bash
  # 显示 services 列表
  $ kubectl get services #可简写为 kubectl get svc

  # 显示当前 deployment 列表
  $ kubectl get deployment
  
  # 删除一个 deployment
  $ kubectl delete deployment $deploymentname

  # 删除一个 service
  $ kubectl delete $servicename
  ```

# 遇到的问题
1. minikube services wordpress 在浏览器中打不开这个地址  
   解决方式：结果发现是 terminal 使用了代理，重新打开一个 terminal 再执行命令就可以了。