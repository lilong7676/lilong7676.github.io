---
title: 使用 ChromeRemoteInterface 调试 nodejs 代码
date: 2023-01-10 14:59:29
tags:
categories:
---

> 最近在调研如何远程调试一个 nodejs 进程，大致调研路径是 Chome DevTolls -> Chrome debugging protocol -> chrome-remote-interface，本文重点介绍下 chrome-remote-interface 的简单使用。 
  
根据其[文档](https://github.com/cyrus-and/chrome-remote-interface)，这个工具主要是提供一个 js api，在连接到对应的服务端（要调试的端）后，通过 Chrome Debugging Protocol 与之交互。  

所以其基本的使用分为以下几个步骤：  
- 启动 nodejs 实例，并打开调试模式, eg. `node --inspect-brk=9222 for-insepct.js`
- 通过 CDP 连接到远程调试实例
- 通过 CDP Client 操作远程调试实例

<!-- more -->

#### 准备待调试的 nodejs 实例

先新建 `for-inspect.js`，可以写入任意可执行的代码：  

![for-inspect.js](https://lilong7676-picture.oss-cn-hangzhou.aliyuncs.com/img/20230111114509.png)

执行代码，附加 `--inspect-brk=9222` 调试参数：
```shell
$ node --inspect-brk=9222 for-insepct.js
```
![执行结果](https://lilong7676-picture.oss-cn-hangzhou.aliyuncs.com/img/20230110170510.png)  
有关 inspect-brk 参数的作用可以参考 [node文档](https://nodejs.org/en/docs/guides/debugging-getting-started/#command-line-options)，简单来说就是打开调试模式，并在代码首行断点执行，默认实例调试地址为 localhost:9229  
![inspect-brk 参数](https://lilong7676-picture.oss-cn-hangzhou.aliyuncs.com/img/20230110171113.png) 


此时，nodejs 已经在 `localhost:9222` 上启动了调试实例，可以通过访问 `http://localhost:9222/json` 查看详细信息:
```json
[ {
  "description": "node.js instance",
  "devtoolsFrontendUrl": "devtools://devtools/bundled/js_app.html?experiments=true&v8only=true&ws=localhost:9222/4d7174b7-e4cf-4d60-84c5-80f3eed2477d",
  "devtoolsFrontendUrlCompat": "devtools://devtools/bundled/inspector.html?experiments=true&v8only=true&ws=localhost:9222/4d7174b7-e4cf-4d60-84c5-80f3eed2477d",
  "faviconUrl": "https://nodejs.org/static/images/favicons/favicon.ico",
  "id": "4d7174b7-e4cf-4d60-84c5-80f3eed2477d",
  "title": "for-insepct.js",
  "type": "node",
  "url": "file:///Users/lilonglong/Desktop/long_git/chrome-devtools-analysis/for-insepct.js",
  "webSocketDebuggerUrl": "ws://localhost:9222/4d7174b7-e4cf-4d60-84c5-80f3eed2477d"
} ]
```

可通过 Chrome 直接访问 `devtoolsFrontendUrl`:
![Chrome 访问 devtoolsFrontendUrl](https://lilong7676-picture.oss-cn-hangzhou.aliyuncs.com/img/20230110172257.png)  

当然，可以通过上面的 Chrome 来调试 nodejs 代码，所以此方案不是重点，重点是如何通过代码来远程调试 nodejs 代码。


#### 通过 chrome-remote-interface 远程调试 nodejs
```javascript
// cdp.js
const CDP = require("chrome-remote-interface");

// 通过默认的连接配置，连接到对应的 调试实例（inspector agent）
CDP(async (client) => {
  // 连接成功，通过 CDP client 即可操作调试实例
  const { Debugger, Runtime, Network } = client;
  try {
    // 监听 scriptParsed 事件，拿到 script source code
    Debugger.on("scriptParsed", async (params) => {
      const { scriptId, url } = params;
      // console.log(`scriptId: ${scriptId}, url: ${url}`);
      if (url.startsWith("file://")) {
        const source = await client.Debugger.getScriptSource({ scriptId });
        // 打印获取到的 script 源码
        console.log("script source", source);
      }
    });

    // 监听进入到断点事件
    Debugger.on("paused", (pausedEvent) => {
      // pausedEvent 里面包含当前断点的位置以及 callFrames 信息等
      // console.log('pausedEvent ', pausedEvent);
    });

    // 调试实例如果在等待附加调试器，则先继续运行调试实例
    await Runtime.runIfWaitingForDebugger();

    // 请求允许使用 agent 的 Debugger 功能
    await Debugger.enable();
    // 请求允许使用 agent 的 Runtime 功能
    await Runtime.enable();

    // try {
    //   // 网络请求域，这里注释的原因是 nodejs v8 不支持调试 Network 域
    //   await Network.enable({ maxPostDataSize: 65536 });
    // } catch (error) {
    //   console.error('Network attach failed', error);
    // }

    // 使用
    const testValue = await Runtime.evaluate({
      expression: "eval(1 + 1)",
    });
    console.log("evaluate result", testValue);
  } catch (err) {
    console.error(err);
  } finally {
    client.close();
  }
}).on("error", (err) => {
  console.error(err);
});

```

执行结果：
![node cdp.js](https://lilong7676-picture.oss-cn-hangzhou.aliyuncs.com/img/20230111113918.png)


#### Reference
- [v8 CDP 文档](https://chromedevtools.github.io/devtools-protocol/v8/)
- [Cloudflare Worker Debugger 设计](https://blog.cloudflare.com/better-workers-debugging-with-a-network-panel/)
- [JSON-RPC 是什么](https://www.jianshu.com/p/49cd5c9a1664)
- [node debugging guide](https://nodejs.org/en/docs/guides/debugging-getting-started/#command-line-options)
- [http-inspector — a DevTools Network tab for Node.js process](https://garnik.medium.com/http-inspector-a-devtools-network-tab-for-node-js-process-5a008d17bf28)