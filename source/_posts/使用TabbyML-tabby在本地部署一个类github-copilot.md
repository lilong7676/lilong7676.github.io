---
title: 使用TabbyML/tabby在本地部署一个类github copilot
date: 2024-01-11 13:54:03
tags:
categories: GPT
---

刚好自己的 github copilot 过期了，就逛了下 github，发现了一个有趣的仓库 [tabby](https://github.com/TabbyML/tabby)，它的描述瞬间吸引了我哈哈：
> Tabby is a self-hosted AI coding assistant, offering an open-source and on-premises alternative to GitHub Copilot. It boasts several key features:

> Self-contained, with no need for a DBMS or cloud service.  
OpenAPI interface, easy to integrate with existing infrastructure (e.g Cloud IDE).  
Supports consumer-grade GPUs.  

逛了下官网，发现可以在 Apple M1/M2 芯片的电脑上本地安装和使用，所以试了下，这里记录下。

<!-- more -->

安装的命令很简单  
```bash
brew install tabbyml/tabby/tabby

# Start server with StarCoder-1B
tabby serve --device metal --model TabbyML/StarCoder-1B
```

第二步如果启动的时候报类似于这个错误，则考虑科学上网下：
![](https://lilong7676-picture.oss-cn-hangzhou.aliyuncs.com/img/202401111358602.png)

下面是启动成功的截图：
![](https://lilong7676-picture.oss-cn-hangzhou.aliyuncs.com/img/202401111359077.png)

下一步就是安装对应的 vscode 插件，参考[官网链接](https://tabby.tabbyml.com/docs/extensions/installation/vscode)


使用效果 ->
![](https://lilong7676-picture.oss-cn-hangzhou.aliyuncs.com/img/202401111505414.png)


![](https://lilong7676-picture.oss-cn-hangzhou.aliyuncs.com/img/202401111508794.png)


我目前的电脑配置 macbook air m1 丐中丐，总的来说，可以用，有的提示耗时过长了，有的直接没有提示。效果还需要后面用多了才能评价，但是总比没有的好。
还是很期待类似的技术能够硬件配置小型化，让个人电脑都能够部署。