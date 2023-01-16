---
title: 编译并运行 Chrome Devtools Frontend
date: 2023-01-12 09:59:51
tags: ChromeDevTools
categories: javascript
---


> #### Chrome DevTools 工作原理简单介绍

Devtools 不只是我们在浏览器中按F12所打开的调试界面，它是 Chromium 的一部分，在 Electron 中也有集成。  

Devtools 主要由四部分组成：  
- [Frontend](https://github.com/ChromeDevTools/devtools-frontend): 调试器前端，默认由 Chromium 内核层集成，DevTools Frontend 是一个 Web 应用程序，就是 JS + CSS + HTML 组成的；
- [Backend](https://github.com/ChromeDevTools/devtools-frontend/tree/main/v8): 调试器后端，广义上主要由 Chromium、V8 或 Node.js(v8)提供，用来与 Frontend 连接与通信;
- [Protocol](https://chromedevtools.github.io/devtools-protocol/): 通信协议,调试器前端和后端通过此协议通信，协议遵循 [JSON RPC2.0](https://www.jianshu.com/p/49cd5c9a1664) 规范;
- [Connection](https://github.com/ChromeDevTools/devtools-frontend/blob/b29da32422767c87749ca8132c2c0c8f378da6d7/front_end/core/sdk/Connections.ts#L13): 连接层，目前有 `WebSocketConnection`、`StubConnection`、`MainConnection` 三种连接方式，查看其他[资料](https://zhaomenghuan.js.org/blog/chrome-devtools-frontend-analysis-of-principle.html#devtools-%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86)，也有说是消息通道，即 Embedder Channel、WebSocket Channel、Chrome Extensions Channel、USB/ADB Channel，但是我在当前的版本没有找到这部分代码；  
  
![devtools 图示](https://lilong7676-picture.oss-cn-hangzhou.aliyuncs.com/img/20230112104133.png?x-oss-process=image/resize,w_800)

<!-- more -->

> #### Devtools Frontend 编译

1. clone devtools-frontend 源码
```bash
$ git clone git@github.com:ChromeDevTools/devtools-frontend.git
```
1. 安装 depot_tools
```bash
# 必须安装此工具才能编译 devtools-frontend

# 下载 depot_tools 源码
$ git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git

# 将其添加到本机环境变量中
$ export PATH=$PATH:/path/to/depot_tools

# 使环境变量修改立即生效
$ source ~/.bash_profile
```
3. 只检出 DevTools frontend 的代码，因为 DevTools Frontend 可独立于 Chromium 作为独立项目构建，并且如果不执行这一步，编译会报错：`gn.py: Could not find checkout in any parent of the current path.
This must be run inside a checkout.`
```bash
$ mkdir devtools
$ cd devtools
$ fetch devtools-frontend
# 注意，上一步可能会由于网络问题导致 git 相关操作失败，可通过配置 git 和终端代理解决
```
4. 构建
```bash
$ cd devtools-frontend
$ gn gen out/Default
$ autoninja -C out/Default
# 构建产物在 devtools-frontend/devtools/devtools-frontend/out/Default 文件夹中
```


> #### 运行 devtools frontend 产物
```bash
# 进入到构建产物目录
$ cd out/Default/gen/front_end

# serve 构建产物
$ npx http-server

```

查看构建产物目录，这几个 html 其实就是调试前端面板：
![构建产物目录](https://lilong7676-picture.oss-cn-hangzhou.aliyuncs.com/img/20230112111618.png?x-oss-process=image/resize,w_800,h_600)

比如打开 `http://127.0.0.1:8080/devtools_app.html`，就是我们熟悉的 web 页面调试界面：
![](https://lilong7676-picture.oss-cn-hangzhou.aliyuncs.com/img/20230112111832.png?x-oss-process=image/resize,w_800)  

也可以自己尝试打开其他html，如 `http://127.0.0.1:8080/node_app.html`，`http://127.0.0.1:8080/inspector.html`等  


> #### 结合 node inspect 来调试 nodejs 代码

如[上一篇](https://lilong7676.github.io/2023/01/10/uncategorized/%E4%BD%BF%E7%94%A8-ChromeRemoteInterface-%E8%B0%83%E8%AF%95-nodejs-%E4%BB%A3%E7%A0%81/#%E5%87%86%E5%A4%87%E5%BE%85%E8%B0%83%E8%AF%95%E7%9A%84-nodejs-%E5%AE%9E%E4%BE%8B)所讲，先准备带调试的 nodejs 实例：
```bash
$ node --inspect=9222 for-inspect.js
```

然后访问 `http://localhost:9222/json` 获取到 `devtoolsFrontendUrl`，类似于 `devtools://devtools/bundled/js_app.html?experiments=true&v8only=true&ws=localhost:9222/8c4626a1-2903-4da8-9759-5de3c74c3a35`  

只需要将其 `query` 参数拼接到 `http://127.0.0.1:8080/***.html`后面即可，拼接后类似于 `http://127.0.0.1:8080/inspector.html?experiments=true&v8only=true&ws=localhost:9222/8c4626a1-2903-4da8-9759-5de3c74c3a35`，在浏览器中打开后的效果如图：
![](https://lilong7676-picture.oss-cn-hangzhou.aliyuncs.com/img/20230112135350.png?x-oss-process=image/resize,w_800)

> #### 从 devtools 中查看 Chrome Devtools Protocol 🧐

在 `Settings -> Experiments` 中打开 `Protocol Monitor` 选项：
![打开 Protocol Monitor 选项](https://lilong7676-picture.oss-cn-hangzhou.aliyuncs.com/img/20230112140245.png?x-oss-process=image/resize,w_800)


显示 `Protocol Monitor`：
![](https://lilong7676-picture.oss-cn-hangzhou.aliyuncs.com/img/20230112140441.png?x-oss-process=image/resize,w_800)

![CDP](https://lilong7676-picture.oss-cn-hangzhou.aliyuncs.com/img/20230112141045.png?x-oss-process=image/resize,w_800)


> #### 定制调试面板的显示，只需要 console、source、network 面板

> 很多情况下，我们想自定义调试界面需要显示哪些面板。比如我只需要 console、source、network 这三个面板，不需要其他的，而且全部集成也不太现实，因为加载速度太慢。  


观察其产物的js，这里以 `inspector.html` 解释：

![inspector.html](https://lilong7676-picture.oss-cn-hangzhou.aliyuncs.com/img/20230112165907.png?x-oss-process=image/resize,w_800)

说明其入口 js 为 `./entrypoints/inspector/inspector.ts`，查看其原始代码：
![](https://lilong7676-picture.oss-cn-hangzhou.aliyuncs.com/img/20230112172446.png?x-oss-process=image/resize,w_800)
这里能看出来，`inspector_app` 相比 `devtools_app` 增加了 `screencast` 面板的功能。  ![Alt text](https://lilong7676-picture.oss-cn-hangzhou.aliyuncs.com/img/20230112165907.png?x-oss-process%3Dimage%2Fresize%2Cw_800)
所以我们大致得出结论，如果想要定制调试面板的现实，只需要修改对应 `entrypoints/***_app` 下的代码即可。


> #### 改造 devtools_app

通过查看 devtools_app 的界面，因为我们只需要 console、source、network 三个面板，所以需要移除多余的调试面板:
![](https://lilong7676-picture.oss-cn-hangzhou.aliyuncs.com/img/20230112173848.png?x-oss-process=image/resize,w_800)

##### Reference
- [玩转 Chrome DevTools，定制自己的调试工具](https://www.51cto.com/article/716801.html)
- [Chrome DevTools Frontend 运行原理浅析](https://zhaomenghuan.js.org/blog/chrome-devtools-frontend-analysis-of-principle.html#%E6%BA%90%E7%A0%81%E4%B8%8B%E8%BD%BD%E5%8F%8A%E7%BC%96%E8%AF%91)