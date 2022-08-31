---
title: web前端Proxy沙箱实现时需注意RegExp等原生对象问题
date: 2022-08-31 14:52:01
tags:
categories: 
 - javascript
---


## 问题起因
起因是一个前端js沙箱的实现中，发现通过沙箱执行代码中访问 RegExp.$n 的值一直是空字符串，类似于：
```javaScript
const code = `
    const r = /^(\\d{4})-(\\d{1,2})-(\\d{1,2})$/
    r.exec('2022-08-31')
    return RegExp.$1;`;
evalScriptInSandbox(code); 
```

<!-- more -->

正常情况下，`evalScriptInSandbox(code)` 应该返回 `'2022'`,但是执行发现只会返回空字符串，最后发现是 RegExp 的执行上下文问题。

因为沙箱的实现类似于这种：
```javascript
const evalScriptSandbox = (code, fakeWindow) => {
  const resolver = new Function(`
    return function(window) {
      with(window) {
        try {
          return (function() {
            "use strict";
            ${code}
          })();
        } catch(e) {
          console.log(e);
        }
      }
    }
  `);

  // 正是这里的 call(fakeWindow)导致 RegExp 执行时的上下文是 fakeWindow 
  // 而不是真正的宿主 window,导致 bug 产生
  return resolver().call(fakeWindow, fakeWindow);
};
```

解决方式就是访问`fakeWindow`中的 `RegExp` 对象时，需要返回宿主环境的 `RegExp`，这和浏览器的内部实现有关系，暂时没发现其他的解决方式。

## Reproduce Step :
```bash
$ git clone https://github.com/lilong7676/js-regexp-bug.git
$ cd js-regexp-bug
$ npm i
$ npm run start
```

## ⚠️
⚠️另外需要注意，在 eval、Function 构造函数中执行正则表达式时，需要转义反斜线，即:
```javascript
eval(`
const r = /^(\d{4})-(\d{1,2})-(\d{1,2})$/
r.exec('2022-08-31'); // null
console.log(RegExp.$1) // ''
`)
```

在 eval 中需要写为:
```javascript
eval(`
const r = /^(\\d{4})-(\\d{1,2})-(\\d{1,2})$/
r.exec('2022-08-31');
console.log(RegExp.$1); // 2022
`)
```

### 类似的原生对象，如 eval 等，也会在这种沙箱的实现方式中出现作用域问题，需要注意。