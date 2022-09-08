---
title: flutter中通过ffi使用quickjs
date: 2022-09-01 10:43:43
tags:
categories:
 - flutter
---


- quickjs中 判断 JSValue 的类型

    在 `quickjs.h` 中 搜索 `JS_Is`开头的方法:
  - JS_IsBool(JSValue)
  - JS_IsNull
  - JS_IsUndefined
  - JS_IsNumber
  - JS_IsString
  - JS_IsObject
  - JS_IsException
  - ...
  - 
<!-- more -->

- quickjs中的 JSValue 与 c 类型的转换
  
     在`quickjs.h`中搜索 `JS_To`开头的方法：
  - JS_ToBool(JSContext, JSValue)
  - JS_ToInt32
  - JS_ToInt64
  - JS_ToFloat64
  - JS_ToCString    需注意 JS_FreeCString


为什么 `webf` 中的 `NativeString` 类型，使用 uint16_t* 表示 string?
```c
struct NativeString {
  const uint16_t* string;
  uint32_t length;

  NativeString* clone();
  void free();
};
 /**
 首先， uint16_t 即为 unsigned short int 类型，值范围为 0 - 65535。

 **/

```


- 有关 JSString、JSAtom 的分析：
    - https://blog.csdn.net/jayyuz/article/details/124450556

