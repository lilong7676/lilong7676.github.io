---
title: babel 的原理及插件的编写（简要）
date: 2022-11-23 10:39:06
tags: Babel
categories: javascript
---


<!-- # 什么是 AST 
# 访问者模式
# babel-parser
# babel-traverse -->

### 由浅入深，先从几个简单的插件的编写开始吧
（本次分析的 babel 版本为 7.20.4）
<!-- more -->
***

> #### *实现一个自动将 console.log 信息加上所属代码的位置（行列）信息的 babel 插件*


例子代码：
```javascript
function test() {
  const a = 2;
  console.log('a', a);
  return a;
}

test();

```

- 首先查看这段代码对应的[ AST 结构](https://astexplorer.net/#/gist/9357ea04e1cbba03852c2978bf5f4ecb/75b12e8e7c4e82217e17ba7ec6f9db1d7c555aae)


![AST 结构](https://lilong7676-picture.oss-cn-hangzhou.aliyuncs.com/img/20221207111654.png)

- 在上图可以看到 `console.log` 对应的节点类型为 `CallExpression -> MemberExpress -> Identifier`，所以我们的思路是：

  - 找到类型为 CallExpression 的节点
  - 判断其代码是否是 console.log
  - 拿到位置信息
  - 插入到 arguments 数组中

具体实现代码：
```javascript
const babel = require("@babel/core");

const testCode = `function test() {
  const a = 2;
  console.log('a', a);
  return a;
}

test();
`;

function myCustomPlugin({ types }) {
  return {
    visitor: {
      CallExpression(path, state) {
        // 判断 callee 是否是 console.log
        const callee = generate(path.node.callee).code;
        if ("console.log" === callee) {
          // 拿到其代码位置信息
          const { line, column } = path.node.loc.start;
          // 将位置信息插入到 arguments 数组中
          path.node.arguments.unshift(
            types.stringLiteral(`loc:[${line},${column}]`)
          );
        }
      },
    },
  };
}

const output = babel.transformSync(testCode, {
  plugins: [myCustomPlugin],
});

console.log(output.code);
```
执行结果：
![](https://lilong7676-picture.oss-cn-hangzhou.aliyuncs.com/img/20221207114055.png)


<br>

***

> #### *把代码里的 标识符n 转换为 标识符x*
```javascript
// 下面的逻辑是把 代码里的 标识符n 转换为 标识符x
const { parse } = require('../babel-lib/packages/babel-parser');
const traverse = require('../babel-lib/packages/babel-traverse').default;
const generate = require('../babel-lib/packages/babel-generator').default;

const code = 'const n = 1';

// parse the code -> ast
const ast = parse(code);

// transform the ast
traverse(ast, {
  enter(path) {
    // 'n' to 'x'
    if (path.isIdentifier({ name: 'n' })) {
      path.node.name = 'x';
    }
  },
});

// ast to code
const output = generate(ast);
console.log(output.code); // 'const x = 1;'
```

上面实现了一些简单的 babel 转换插件，实现思路也很清晰了，整体的思路就是:
``` 
code -> AST -> transformed AST -> transformed code  
```
下面分析下其基本原理。

<br>

### 首先简单介绍下什么是 AST
AST(abstract syntax tree)，即抽象语法树。 

#### 关键的概念
1. Tokenizer 分词： 将整个代码字符串分割成语法单元数组（token）

JS 代码中的语法单元主要指如标识符（if/else、return、function）、运算符、括号、数字、字符串、空格等等能被解析的最小单。 [可以看下这个在线分词工具](https://esprima.org/demo/parse.html#);

2.语法分析： 在分词结果的基础上分析语法单元之间的关系。

语义分析则是将得到的词汇进行一个立体的组合，确定词语之间的关系。

先理解两个重要概念，即语句和表达式。
```javascript
var a = 1 + 2;
```
语句(statement)，👆上面就是一条语句，一般情况下，在js里每一行就是一个语句。  
表达式(expression)，如上面的 `1 + 2`就是表达式，是指最终有返回结果的一小段代码，可以嵌入到另一个表达式，且包含在语句中。
简单来说语法分析是对语句和表达式识别，这是个递归过程，在解析中，babel 会在解析每个语句和表达式的过程中设置一个暂存器，用来暂存当前读取到的语法单元，如果解析失败，就会返回之前的暂存点，再按照另一种方式进行解析，如果解析成功，则将暂存点销毁，不断重复以上操作，直到最后生成对应的语法树。


先看下编译原理中，词法分析到语法分析阶段图：
![词法分析到语法分析](https://lilong7676-picture.oss-cn-hangzhou.aliyuncs.com/img/20221207165755.png)

上面的产物 parse tree 即解析树，代表了源码的所有信息，很详细，也很冗余（包含分号，冒号等），而 AST，抽象了一些，即精简了 parse tree：

![](https://lilong7676-picture.oss-cn-hangzhou.aliyuncs.com/img/20221207170423.png)

- AST不含有语法细节，比如冒号、括号、分号
- AST会压缩单继承节点
- 操作符会变成内部节点，不再会以叶子节点出现在树的末端。

### 那 babel 怎么将代码转为 AST 的？

根据[文档](https://babeljs.io/docs/en/babel-parser#output)可知要在 `packages/babel-parser` 里找逻辑
```bash
# 在 babel 源码目录里执行
$ make watch # 编译并 watch
$ cd packages/babel-parse/test
# 在 babel-parse 里新建单元测试文件，用来验证逻辑
$ touch test-parser.js
```
```javascript
// test-parser.js
import { parse } from "../lib/index.js";

function getParser(code) {
  // sourceType 可能的值为 "script", "module", or "unambiguous"，
  // 如果是 unambiguous，则 babel 会尝试判断代码中是否包含 import export 语句来确定具体类型
  return () => parse(code, { sourceType: "module" });
}

describe("测试 parse -> ast", function () {
  it("should parse", function () {
    expect(getParser(`function foo() {}`)()).toMatchSnapshot();
  });
});
```
```bash
# 通过下面的命令执行 jest
 $ BABEL_ENV=test node_modules/.bin/jest -u packages/babel-parser/test/test-parser.js
```

查看 `parse` 函数源码，得到其执行路径：
![parse()执行路径](https://lilong7676-picture.oss-cn-hangzhou.aliyuncs.com/img/20221208144240.png)

其中 Parser 类继承关系：
![Parser 继承路径](https://lilong7676-picture.oss-cn-hangzhou.aliyuncs.com/img/20221208140115.png)

关注点转移到 `parser.parse()` 函数中，查看[源码:](https://github.com/lilong7676/babel/blob/871fd5ec0c22829495d6a11b1729feb7e9e52ad7/packages/babel-parser/src/parser/index.ts#L39)：
```javascript
  parse(): N.File {
    // 进入到初始作用域中
    this.enterInitialScopes();
    // 初始化 file、program 节点
    const file = this.startNode() as N.File;
    const program = this.startNode() as N.Program;
    // 获取并更新下一个 token 信息
    this.nextToken();
    file.errors = null;
    // 从头节点开始解析
    this.parseTopLevel(file, program);
    file.errors = this.state.errors;
    // 返回解析结果，即 ast
    return file;
  }
```


关注点转移到 `this.nextToken()` 方法
```javascript
// 获取并更新下一个 token 信息
  nextToken(): void {
    // 忽略空白字符（空格、tab、换行等）并更新当前位置信息
    this.skipSpace();
    // 记录当前 token 开始字符位置
    this.state.start = this.state.pos;
    // 如果不是 “临时向前看”，则记录 token 开始位置（行、列、index）
    if (!this.isLookahead) this.state.startLoc = this.state.curPosition();
    // 如果当前 tokenizer 的位置已经到了 input 的末尾
    if (this.state.pos >= this.length) {
      // 则结束本次 token 查找，并更新当前 token 的信息（位置、token类型、token值）到 state 上
      this.finishToken(tt.eof);
      return;
    }
    // 根据当前的 字符，读取相应的 token
    this.getTokenFromCode(this.codePointAtPos(this.state.pos));
  }

```


> 感兴趣的可以继续查看 `getTokenFromCode` 的实现逻辑，这里其实就是 tokenzier 的核心逻辑

关注点转移到  `parseTopLevel() -> parseProgram -> parseBlockBody -> parseBlockOrModuleBlockBody ->  parseStatement -> parseStatementContent`  
具体逻辑可以查看 [github](https://github.com/lilong7676/babel/tree/main/packages/babel-parser/src) 上添加的代码注释。

***

### babel 是如何遍历 ast的？
在 `packages/babel-traverse` 有两个关键的对象:

**Visitor**
对于这个遍历过程，babel 通过实例化 visitor 对象完成，其实我们生成出来的 AST 结构都拥有一个 accept 方法用来接收 visitor 访问者对象的访问，而访问者其中也定义了 visit 方法(即开发者定义的函数方法)使其能够对树状结构不同节点做出不同的处理，借此做到在对象结构的一次访问过程中，我们能够遍历整个对象结构。（访问者设计模式：提供一个作用于某对象结构中的各元素的操作表示，它使得可以在不改变各元素的类的前提下定义作用于这些元素的新操作）

遍历结点让我们可以定位并找到我们想要操作的结点，在遍历每一个节点时，存在enter和exit两个时态周期，一个是进入结点时，这个时候节点的子节点还没触达，遍历子节点完成的后，会离开该节点并触发exit方法。

**Path**
Visitors 在遍历到每个节点的时候，都会给我们传入 path 参数，包含了节点的信息以及节点和所在的位置，供我们对特定节点进行修改，之所以称之为 path 是其表示的是两个节点之间连接的对象，而非指当前的节点对象。path属性有几个重要的组成，主要如下： 
![Path](https://lilong7676-picture.oss-cn-hangzhou.aliyuncs.com/img/20221209142949.png)

关键路径：
```
// babel-traverse
traverse(options) -> traverseNode(...) -> new TraversalContext() -> context.visit(node, key)
```

#### 参考资料
[Leveling Up One’s Parsing Game With ASTs](https://medium.com/basecs/leveling-up-ones-parsing-game-with-asts-d7a6fc2400ff)

#### 彩蛋：Babel song
Hallelujah ——  [In Praise of Babel](https://youtu.be/40abpedBKK8)