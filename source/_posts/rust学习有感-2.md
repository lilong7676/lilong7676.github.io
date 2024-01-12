---
title: rust学习有感-2
date: 2023-12-28 17:09:06
tags:
categories: rust
---

上一篇已经学习到了第五章 rust 结构体，看了下教程共有 20 章，继续学吧。。

### 枚举和模式匹配
- 枚举定义
  ```rust
  enum ipAddrKind {
    v4,
    v6,
  }

  #[derive(Debug)]
  enum Message {
      Quit,
      Move { x: i32, y: i32 },
      Write(String),
      ChangeColor(i32, i32, i32),
  }

  impl Message {
      fn call(&self) {
          dbg!("call {}", self);
      }
  }

  fn main() {
      let m = Message::Write(String::from("hello"));
      m.call();

      let change_color = Message::ChangeColor(12, 255, 255);
      change_color.call();

      let move_msg = Message::Move { x: 18, y: 18 };
      // 怎么取出枚举中的值？
      // println!("move.x {} move.y {}", move_msg.x ???);
      move_msg.call();

      let quit = Message::Quit;
      quit.call();
  }

  ```
- `match` 控制流
  ```rust
  enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
  }

  fn value_in_cents(coin: Coin) -> u8 {
      match coin {
          Coin::Penny => {
            // 也可以写上大括号，表达式末尾是返回值
            println!("i am penny");
            1
          } // 如果有大括号，则表达式末尾的逗号可以去掉
          Coin::Nickel => 5,
          Coin::Dime => 10,
          Coin::Quarter => 25,
      }
  }

  ```
  - 首先，`match` 是一个表达式，所以会返回某一个值
  - `match` 每一个分支相关联的代码也是一个表达式，而表达式的结果将作为整个 `match` 表达式的返回值
  - 穷举性、通配符 `_`
  - `if let` 简洁控制流
    ```rust
      // if let 的使用
      let config_max = Some(20);
      if let Some(max) = config_max {
          println!("config_max is {}", max);
      } else {
        println!("not config_max");
      }
    ```

  总的来说，rust 的模式匹配是为了减少代码出现“漏分支”的情况，也提高了一些编码效率。而 rust 的枚举就变得不像其他语言那么纯粹了。感觉是得适应一段时间。

  ### 包、crate、模块管理


