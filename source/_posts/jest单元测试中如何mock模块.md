---
title: jest单元测试中如何mock模块
date: 2023-10-23 14:02:52
tags: unit test
categories: javascript
---

### 问题背景

> 在给一个nodejs express项目写jest单元测试的过程中，发现有些依赖在单元测试代码中会报错，原因是某些配置没有加载，当然在正常的运行中肯定是ok的，只有在单元测试中会出现。然而这些配置的加载逻辑是在所依赖的框架内部写的，而且有些配置是线上获取的配置，只有在内网环境才能获取到。所以问题就来了，怎么在jest中mock这个依赖（的配置加载逻辑）？


<!-- more -->

### 解决方式
经过查阅[jest文档](https://jestjs.io/docs/manual-mocks#mocking-node-modules)，下面简单记录下核心点。  
比如我想mock一个依赖名为 `@foo/bar`，那么可以在`node_modules`同级的地方新建一个文件夹 `__mocks__`，并且在其下建立`@foo/bar.js`。  

此时jest在执行单元测试的过程中，如果遇到如`import bar from '@foo/bar'`时，就会从 `__mocks__`中的对应目录中加载该依赖。
```
.
├── __mocks__/
│   └── @foo/
│       └── bar.js
├── node_modules
├── src
└── ...

```

此时，就可以针对依赖的逻辑进行随心所欲的修改（mock）了。
```javascript
// __mocks__/@foo/bar.js
module.exports = {
    // 逻辑重写
    // ...
}
```

那么此时问题又来了，一个模块会导出很多方法，只想覆盖某一个导出的方法逻辑，其他保持不变，如何实现？

也很简单，使用`jest.requireActual(moduleName)`即可:

```javascript
// __mocks__/@foo/bar.js

// 拿到真实的模块导出
const bar = jest.requireActual('@foo/bar');

// 重新导致 mock 后的模块
module.exports = {
    // 其他导出保持不变
    ...bar,
    // 只重写部分导出的逻辑
    getConfig() {
        return {
            // ...
        }
    }
}
```

### 参考
- [jest manual-mocks](https://jestjs.io/docs/manual-mocks)
