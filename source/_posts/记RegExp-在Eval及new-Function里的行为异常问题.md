---
title: 记 RegExp 字面量形式在 eval 及 new Function里的行为异常问题
date: 2022-08-29 09:31:43
tags:
categories:
---


首先看一下最简单的代码
```javascript
const r = /^(\d{4})-(\d{1,2})-(\d{1,2})$/;
r.exec('2019-10-08');
console.log(RegExp.$1); // 2019
```
上面这段代码在 Chrome console 中执行结果为 2019，一切正常。


```javascript
eval(`
const r = /^(\d{4})-(\d{1,2})-(\d{1,2})$/;
r.exec('2019-10-08');
RegExp.$1;
`); // ''
```
`重新打开一个 Chrome console`, 然后执行上面这段代码，结果为`空字符串`，
经过查阅资料，发现在 eval 中执行 regx的字面量时，需要`额外转义`反斜线，即：

```javascript
eval(`
const r = /^(\\d{4})-(\\d{1,2})-(\\d{1,2})$/;
r.exec('2019-10-08');
RegExp.$1;
`); // '2019'

```


<!-- more -->

参考资料：
1. https://stackoverflow.com/a/61147060/11394539