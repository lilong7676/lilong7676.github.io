---
title: 一个简单的 WebAssembly 编译流程 demo
date: 2022-08-25 16:09:19
tags:
 - WebAssembly
categories:
 - WebAssembly
---

记录下如何讲一个简单的程序编译为 wasm。由于我是mac环境，所以以下以mac环境为主，其他环境请参考 [官方github](https://github.com/emscripten-core/emsdk)。

<!-- more -->

## 前置条件:
git、cmake、Xcode(主要是llvm, linux为GCC)、Python 2.7.x

## 编译 emsdk:

```bash
$ git clone https://github.com/juj/emsdk.git
$ cd emsdk
$ ./emsdk install latest
$ ./emsdk activate latest
```

如果希望将 emsdk 加入到终端的环境变量里，则执行：
```bash
$ source "{path_to_emsdk}/emsdk_env.sh"
```

## 编译一个demo试试吧：

1. 新建一个 hello.c，内容如下：
```c
#include<stdio.h>

int main(int argc, char **argv) {
printf("hello world\n");
}
```

2. 执行命令：
```bash
$ emcc hello.c -s WASM=1 -o hello.html
```
注意 *-s WASM=1 * 参数，不加这个话，会编译为asm.js。


如果想要生成一个模板 html 页面，则输出文件的格式需为 html，即 * -o hello.html *。

![生成的文件](https://raw.githubusercontent.com/lilong7676/Picture/master/blog/image/20220825180332.png)


3. 我们可以使用 emrun 命令来创建一个 http 协议的 web server 来展示我们编译后的文件。

```bash
$ emrun --no_browser --port 8080 .
```

4. 在浏览器打开 http://0.0.0.0:8080/hello.html：
![效果图](https://raw.githubusercontent.com/lilong7676/Picture/master/blog/image/20220825180019.png)
