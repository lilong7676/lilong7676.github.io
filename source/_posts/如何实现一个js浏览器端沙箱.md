---
title: 如何实现一个js浏览器端沙箱
date: 2022-01-21 09:39:16
tags:
categories: javascript
---

- 什么是沙箱?基本原理是什么?  
  
沙箱的目的其实就是隔离变量,代理某些变量,防止污染宿主环境. 
<!-- more -->
在浏览器中,我们可以利用 js 闭包的能力,利用变量作用域去模拟一个沙箱环境,最简单的例子:
```javascript
function foo(window) {
    console.log(window.document);
}

foo({document: 'fakedocument'});
```
上面这段代码的输出肯定是 fakedocument, 而不是浏览器原生的document.  

- 实现难点:如何模拟浏览器原生window?

有两个方案,一是根据 ECMA 规范实现这一堆的浏览器原生对象,显然这个成本太高.
所以这里有个取巧的办法,我们可以创建一个 iframe并设置src="about:blank",这个时候可以确保 url 同域,也不会发生资源加载.
然后取出 iframe 对应的 **contentWindow**,利用其**天然隔离**特性,省去了自己实现的成本:
```javascript
  /**
   * 在沙盒环境中执行代码
   */
  evalScript(code: string) {
    /**
     * 利用 Function构造函数,构造沙箱环境,给 code 加一层 wrap,创建一个闭包,
     * 把需要隔离的浏览器原生对象变成从函数闭包中获取
     */
    const resolver = new Function(`
      return function({window, location, history, document}) {
        with(window.__GLOBAL_OVERRIDES_VARS__) {
          try {
            (function() {
              "use strict";
              ${code}
            })();
          } catch(e) {
            console.log(e);
          }
        }
      }
    `);
    resolver().call(this.window, { ...this });
  }
```

```javascript
/**
 * 通过proxy 代理 window的访问
 * */
  const iframe = document.createElement( 'iframe' );
  iframe.src = 'about:blank';
  document.body.appendChild(iframe);

  class Window {
    constructor(options, context, iframe) {
    return new Proxy(iframe.contentWindow, {
        set(target, name, value) {
        target[name] = value;
        return true;
      },
      
      get(target, name) {
        switch( name ) {
          case 'document':
            return context.document;
          default:
        }
        
        if( typeof target[ name ] === 'function' && /^[a-z]/.test( name ) ){
          return target[ name ].bind && target[ name ].bind( target );
        }else{
          return target[ name ];
        }
      }
    });
  }
} 

```