---
title: nodejs中 的 AsyncLocalStorage
date: 2023-07-21 14:47:16
tags: AsyncLocalStorage
categories: javascript
---


## 开门见山：什么是 AsyncLocalStorage

根据 Node.js [官方文档](https://nodejs.org/docs/latest-v14.x/api/async_hooks.html#async_hooks_class_asynclocalstorage)："This class is used to create asynchronous state within callbacks and promise chains. It allows storing data throughout the lifetime of a web request or any other asynchronous duration. It is similar to thread-local storage in other languages."，

为了进一步简化解释，AsyncLocalStorage 允许你在执行异步函数时存储状态，然后使其可用于该函数中的所有代码路径。
<!-- more -->

### 场景：一个案例引入

如何实现请求的链路追踪？就是说如何确定一个Request从发起到返回处理结束过程中的所有调用路径？一般是通过发起方携带一个唯一请求标识，可以叫traceId，以此来实现链路追踪，实现日志溯源等。

### 如何实现请求的链路追踪机制？


#### 方案一：全局变量 globalTraceId ？
```javascript

// Raw Node.js HTTP server
const http = require('http');
let globalTraceId // 全局变量

// 0. 处理请求的方法
function handleRequest(req, res) {
  // 生成唯一 traceId，每次请求进入，复写 globalTraceId 的值
  globalTraceId = generateTraceId()

  // 检查用户cookie是否有效
  cookieValidator().then((res) => {
    // 校验成功，返回给用户他需要的内容
    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.write('Congrats! Your damn cookie is the best one!');
    res.end();
  }).catch((err) => {
    //  把 traceId 连同 error 上报给异常监控系统
    reportError(err, globalTraceId)

    // 写状态码500和错误信息等
    // ...
  });
}

// 1. 创建 server 
const server = http.createServer((req, res) => {
  handleRequest(req, res)
});

// 2. 让 server 和 port:3000 建立 socket 链接，持续接收端口信息
server.listen(3000, () => {
  console.log('Server listening on port 3000');
});
```

这里的会出现的问题是由于nodejs单线程执行，第一个请求进来的时候，globalTraceId被赋值，然后进入到异步检查用户cookie的逻辑中，此时node的main stack是空的，事件循环会执行处理下一个请求。此时就会导致globalTraceId被覆写，导致第一个请求的异步检查中错误上报的globalTraceId是错误的。


#### 方案二：直接透传参数
```javascript
const http = require('http');

function handleRequest(req, res) {
  const traceId = req.headers['x-trace-id'] || generateTraceId();
  // 把 traceId 写入 req 这个 object，将参数一路带下去
  req.traceId = traceId;

  // 同上
  cookieValidator().then((result) => {
    // 校验成功，返回给用户他需要的内容
    // ...
  }).catch((err) => {
    //  上报 traceId
    reportError(err, req.traceId)

    // 写状态码500和错误信息等
    // ...
  });
}

function cookieValidator() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      // do someting
      // ...
    }, 1000);
  });
}

// 此后省略监听等操作
// ...
```

这种方式简单粗暴，的确可行。问题就是每次都得透传参数，写起来有点麻烦。

### 方案三：使用 AsyncLocalStorage 实现

下面这个例子，实现了一个日志追踪的最简逻辑。

```javascript
import http from "http";
import { AsyncLocalStorage } from "async_hooks";

// 初始化一个 AsyncLocalStorage 实例，可以初始化多个实例，每一个实例都是互相独立的
const myAls = new AsyncLocalStorage();

function logWithId(msg) {
  // 取出当前的als上下文
  const context = myAls.getStore();
  const id = context.reqId;
  console.log(`${id !== undefined ? id : "-"}:`, msg);
}

let reqId = 0;

http
  .createServer((req, res) => {
    // 我们创建一个模拟的上下文，这个上下文会被传递给 als 回调函数
    const myContext = {
      reqId: reqId++,
    };

    /*
    在提供的上下文环境中同步执行一个函数并返回其返回值。这里的 store 在回调函数外部是不可访问的，
    但是在这个回调函数内部创建的任何异步操作都可以访问 store。
    */

    myAls.run(myContext, () => {
      logWithId("start");

      // 模拟一个异步操作
      setImmediate(() => {
        logWithId("end");
        res.end();
      });

    });
  })
  .listen(8080);

http.get("http://localhost:8080");
http.get("http://localhost:8080");

// 打印
// 0: start
// 1: start
// 0: end
// 1: end
```

## 总结  
主要是简单了解了下 nodejs 中 AsyncLocalStorage 的初步使用，当然更深层次的理解还需要具体的业务场景来体验。