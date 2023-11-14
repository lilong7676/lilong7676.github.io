---
title: jest中使用msw（Mock Service Worker）
date: 2023-10-26 17:04:26
tags:
categories:
---

## 背景

在写单元测试的时候，经常会遇到需要mock请求响应内容，如果用传统的mock方式，比如mock fetch等，会比较繁琐。有没有类似于可以提供一个后端server的能力，使得请求时可以无感知而且也不需要手动patch一些依赖等。

msw（Mock Service Worker）就提供了这个能力，[Service Worker](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API/Using_Service_Workers)扮演的角色其实就很类似于一个Backend Server。所以，在浏览器端，可以很自然的利用浏览器标准的Service Worker API来拦截网络请求，有现成的为啥不用呢？

即使在nodejs端，虽然没有Service Worker的概念，但是msw也是基本抹平了使用上的差异，刚好手头有个node项目需要在单测中mock网络请求，下面简单记录下msw的使用方式。

<!-- more -->

## 基本使用

下面的代码基于 `msw@1.3.2` 版本  

首先我们需要在单元测试启动的时候让 msw 的拦截能力生效，所以在 `jest.config.js` 中需要配置 `setupFiles 或者 setupFilesAfterEnv`，这个根据实际情况配置即可。我配置的值为`['<rootDir>/test/jest.setup.ts']`，接着写入 msw 初始化内容：


```typescript
// jest.setup.ts

import server from './server/setup-server';

beforeAll(() => {
  // 开始监听拦截请求
  server.listen();
});

afterEach(() => {
  // 为了防止在运行时添加了额外的 handlers 影响到其他的单测，
  // 这里需要重置 handlers 为初始值
  server.resetHandlers();
});

afterAll(() => {
  // 关闭拦截 & 清理工作
  server.close();
});


```


```typescript
// server/setup-server.ts
import { setupServer } from 'msw/node';
import { handlers } from './handler';

// 为 request handlers 提供服务端 api 
const server = setupServer(...handlers);

export default server;

```

上面基本就是一些 msw 初始化的模板代码，开发者关心的真正核心逻辑其实就是 handlers 的编写了:  

```typescript
// server/handlers.ts
import { rest } from 'msw';

export const handlers = [
  rest.get(
    'http://www.xxx.com/',
    (req, res, ctx) => {
      return res(
        ctx.json({
          hello: 'msw'
        })
      );
    }
  )
];
```

使用方式还是很简单的，而且不局限在单元测试上，还可以在接口的 mock 上使用，使用体验还是比较友好的。