---
title: rust学习有感-1
date: 2023-12-20 11:17:06
tags:
categories: rust
---

前端都已经这么卷了，现在要往“锈化“卷了。当然这是玩笑话，毕竟目前锈化的只有一些前端基础工具而已。
但是结合工作实际，学习Rust也是有必要的，比如现在正在完善的云函数运行时服务，采用 nodejs + isolated-vm 的实现形式，
受限于 node 本身的性能限制，还有 isolated-vm 与底层 v8 通信的性能损失等。
一些开源库已经迁移到基于 rust 封装的v8了，所以顺便学习下 rust。

现在除了学校里学过的静态编译型语言（C\C++\Java），基于这些知识，学习 rust 应该很简单吧。（哈哈，等着后面打脸）。

主要记录一些学习过程中记忆深刻的点。

PS: 
- 学习网站[Rust 程序设计语言 简体中文版](https://kaisery.github.io/trpl-zh-cn)
- 很有意思的rust学习网站[rustlings](https://github.com/rust-lang/rustlings/tree/rustlings-1?tab=readme-ov-file)

<!-- more -->

### 猜数字游戏
[猜数字游戏](https://kaisery.github.io/trpl-zh-cn/ch02-00-guessing-game-tutorial.html),这一章写得很不错，一个小示例，能够窥视 rust 的很多东西了。
- 依赖管理工具 cargo 的简单使用
- 怎么引入依赖
- 怎么定义变量，变量的可变性
- 怎么处理Result类型的错误
- 新的 match 模式匹配语法（和Flutter的模式匹配好像哈哈）
- 怎么使用 loop 循环
- 怎么做类型转换

### 常见编程概念
- rust中，if 是一个表达式，而不是语句。所以
  - if 表达式的返回值可以赋值给一个变量
- loop 循环中，break 语句可以后接一个表达式，该表达式的值会返回给loop循环的执行结果中
- 循环标签（loop label），可以使用 break 语句指定要退出的循环体

### 所有权
- 关键词：移动、浅拷贝、深拷贝
- 没有默认实现 Copy 方法的类型的值，在被赋值给一个变量时（直接赋值、作为函数参数赋值），其所有权会移动（转移），转移后，原有值的变量不可使用
  ```rust
    // 这段代码会报错
    let s1 = String::from("hello");
    let s2 = s1;

    println!("{}, world!", s1);
  ```  
- 如果不想变量转移所有权，可以通过值的引用传递（跟C中的指针一样）
- 值的引用（&val1）只是一个指向值的指针，并不拥有这个值，所以不会发生所有权移动。当不再使用值的饮用时，并不影响其所指向的值
  ```rust
  fn calculate_length(s: &String) -> usize { // s 是 String 的引用
      s.len()
  } // 这里，s 离开了作用域。但因为它并不拥有引用值的所有权，
    // 所以什么也不会发生
  ```  
- 引用 与 可变引用
  ```rust
  fn main() {
    let mut s = String::from("hello");

    change(&mut s);
  }

  fn change(some_string: &mut String) {
      some_string.push_str(", world");
  }
  ```  
  - 数据竞争（data race），不能在同一时间创建多个可变引用
  - slice
  ```rust
  fn main() {
    let mut str = String::from("hello world");
    let index = first_word(&str);
    println!("{}", index);
    str.clear();

  }

  fn first_word(s: &str) -> &str {
      let bytes = s.as_bytes();

      for (i, &item) in bytes.iter().enumerate() {
          if item == b' ' {
              return &s[0..i];
          }
      }
      return &s[..];
  }

  ```  

### 结构体
  - 结构体定义
  ```rust
  struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
  }
  ```  
  - 结构体初始化
  ```rust
  let user1 = User {
        active: true,
        username: String::from("someusername123"),
        email: String::from("someone@example.com"),
        sign_in_count: 1,
  };
  ```  
  - 结构体更新语法(跟 es6 对象解构赋值差不多，但是是两个点 `..`)
  ```rust
   let user2 = User {
        email: String::from("another@example.com"),
        ..user1
    };
  ```  
  - 元组结构体（tuple structs）
  ```rust
    struct Color(i32, i32, i32);
    struct Color2(i32, i32, i32);

    let c1 = Color(11, 12, 13);
    let c2 = Color2(11, 12, 13);
  ```  
  - 没有任何字段的类单元结构体（unit-like structs）
  - 如何使用 `println!` 打印一个结构体？
  - 初步接触到 “外部属性 #[derive(Debug)]”
  ```rust
    #[derive(Debug)]
    struct Rectangle {
        width: i32,
        height: i32,
    }

    fn main() {
        let rect =  Rectangle {
            width: 10,
            height: 20,
        };
        dbg!(&rect);
        println!("area is {}", area(&rect));
    }

    fn area(rect: &Rectangle) -> i32 {
        return rect.height * rect.width;
    }
  ```  
  - `dbg!` 宏的用法
    - `dbg!` 宏接收一个表达式的所有权（`println!` 接收的是值的引用），打印出代码中调用 dbg! 宏时所在的文件和行号，以及该表达式的结果值，并返回该值的所有权。
  - 结构体中的方法（不就是类中的方法么）
  ```rust
    #[derive(Debug)]
    struct Rectangle {
        width: i32,
        height: i32,
    }

    impl Rectangle {
        fn area(&self) -> i32 {
            return self.height * self.width;
        }
    }

    fn main() {
        let rect = Rectangle {
            width: 10,
            height: 20,
        };
        dbg!(&rect);
        println!("area is {}", rect.area());
    }
  ```  
  - 关联函数与命名空间 (不就是类的静态方法么？)
  ```rust
  impl Rectangle {
    // 这是一个关联函数，可以不以 self 作为第一参数
    // 调用方式: Rectangle::square(3)
    fn square(size: u32) -> Self {
        Self {
            width: size,
            height: size,
        }
    }
  }
  ```  
  - rust 支持自动引用和解引用（automatic reference and dereference）
    - 不像c++ 那样还要显式的标记


### 目前为止的学习感想
到目前，初步了解了 rust 中所有权的概念和基本使用，也了解结构体的使用。
总的来说，语法上的确吸收了很多语言的有点，比如自动引用与解引用、解构赋值、元组等。
也从编译阶段消除了一些开发者可能会难以注意到的陷阱，比如所有权的转移、变量的生命周期等，这是目前最需要适应的地方，但也是我看到的 rust 目前最大的亮点吧。