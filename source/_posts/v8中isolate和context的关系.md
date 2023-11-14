---
title: v8中isolate和context的关系
date: 2023-10-27 10:23:56
tags:
categories:
---

### 问题
一个 isolate 是否可以创建多个 context ?
如果可以，那这多个context是否会互相影响？是否会存在全局变量污染？  

### 背景
其实问题的真正背景是在做云函数运行时的性能优化时提出来的。所以主要是 nodejs 环境，使用的库是 (isolated-vm)[https://github.com/laverdet/isolated-vm]，底层使用的就是 v8 运行时，所以初步调研了一下。

#### isolate

isolate 是 v8 实例的概念。一般情况下，isolated 和 线程(threads)是 1:1 的关系，一个 isolate 与主线程(main thread)相关联，一个 isolate 也可以与工作线程相关联(worker thread)。

#### context
context(上下文)是 V8 中全局变量作用域的概念。粗略地说，一个 window 对象对应一个 context。例如，iframe 具有与其 parent 的 window 不同的 window 对象。因此 iframe 的 context 与 parent iframe 的 context 不同。由于这些 context 创建自己的全局变量 scope ，因此 iframe 的全局变量和原型链与 parent iframe 的全局变量和原型链是隔离的。

<!-- more -->

举个例子：  
```javascript
// main.html
<html><body>
<iframe src="iframe.html"></iframe>
<script>
var foo = 1234;
String.prototype.substr =
    function (position, length) { // Hijacks String.prototype.substr
        console.log(length);
        return "hijacked";
    };
</script>
</body></html>

// iframe.html
<script>
console.log(foo);  // undefined
var bar = "aaaa".substr(0, 2);  // Nothing is logged.
console.log(bar);  // "aa"
</script>

```

总的来说，每一个 frame 有一个 window 对象，每一个 window 对象有一个 context，每一个 context 有它自己的全局变量作用范围和原型链。

### 结论
结论很明显了，一个 isolate 可以创建多个 context，而且 context 之间不会存在全局变量污染的情况。

### reference

(What exactly is the difference between v8::Isolate and v8::Context?)[https://stackoverflow.com/questions/19383724/what-exactly-is-the-difference-between-v8isolate-and-v8context]

(Design of V8 bindings)[https://chromium.googlesource.com/chromium/src/+/master/third_party/blink/renderer/bindings/core/v8/V8BindingDesign.md]