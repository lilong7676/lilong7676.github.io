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


## 尝试加入一个第三方库进行编译
这里我选择将 [cJSON](https://github.com/DaveGamble/cJSON) 这个库编译为 wasm:

1. 从 github 下载 cJSOM 源码到项目的 vendor 文件夹
   
2. 新建 main.c，内容为:
```c
#include <stdio.h>
#include "cJSON/cJSON.h"

int main() {
    const char jsonstr[] = "{\"data\":\"Hello World!!!\"}";
    cJSON *json = cJSON_Parse(jsonstr);

    const cJSON *data = cJSON_GetObjectItem(json, "data");
    printf("%s\n", cJSON_GetStringValue(data));

    cJSON_Delete(json);
    return 0;
}
```

3. 新建 CMakeLists.txt:
```cmake
cmake_minimum_required(VERSION 3.15) # 根据你的需求进行修改
project(sample C)

set(CMAKE_C_STANDARD 11) # 根据你的C编译器支持情况进行修改
set(CMAKE_EXECUTABLE_SUFFIX ".html") # 编译生成.html

include_directories(vendor) # 使得我们能引用第三方库的头文件
add_subdirectory(vendor/cJSON)

add_executable(sample main.c)

# 设置Emscripten的编译链接参数，我们等等会讲到一些常用参数

set_target_properties(sample PROPERTIES LINK_FLAGS "-s EXIT_RUNTIME=1")
target_link_libraries(sample cjson) # 将第三方库与主程序进行链接
```

4. 新建 build 文件夹，并执行编译操作：
```bash
$ mkdir build
$ cd build
$ emcmake cmake ..
$ emmake make
```

5. 此时会发现控制台报类似于下面的错误
```bash
[  6%] Linking C executable sample.html
wasm-ld: error: vendor/cJSON/libcjson.a(cJSON.c.o): undefined symbol: __stack_chk_guard
```

6. 解决上述错误的办法是修改 cJSON 的 CMakeLists.txt 中的 -fstack-protector-strong 修改为 -fno-stack-protector，然后重复步骤4

7. 此时 build 文件夹内已生成 wasm：
![生成的wasm](https://raw.githubusercontent.com/lilong7676/Picture/master/blog/image/20220826140809.png)

8. 执行以下命令后，在浏览器 localhost:8080/sample.html 查看即可：
```bash
emrun --no_browser --port 8080 .
```

## 后续再尝试将 [flutter-jsdom](https://github.com/lilong7676/flutter-jsdom) 的 bridge 层打包为 wasm，需要先好好消化下😂

#### 参考资料：
1. https://cloud.tencent.com/developer/news/690454

2. http://webassembly.org.cn/getting-started/developers-guide/