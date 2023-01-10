---
title: 使用 babel 实现一个自创的 js 语法
date: 2022-12-09 14:39:26
tags: Babel
categories: javascript
---


### 用 babel 实现用 `@!`标记一个函数时，将为函数自动加上调用日志打印的功能
```javascript
// 如
function @! foo() {}
// 转化为
function foo() {
    console.log('function name:', 'foo', 'date:', Date.now())
}
```

<!-- more -->

根据[文档](https://babeljs.io/docs/en/babel-parser#output)可知要在 `packages/babel-parser` 里找逻辑

**注意，请先 fork babel 源码**

```bash
# 在 babel 源码目录里执行
$ make watch # 编译并 watch
$ cd packages/babel-parse/test
# 在 babel-parse 里新建单元测试文件，用来验证逻辑
$ touch test-parser.js
```

test-parser.js 内容：
```javascript
// test-parser.js
import { parse } from "../lib/index.js";

function getParser(code) {
  // sourceType 可能的值为 "script", "module", or "unambiguous"，
  // 如果是 unambiguous，则 babel 会尝试判断代码中是否包含 import export 语句来确定具体类型
  return () => parse(code, { sourceType: "module" });
}

describe("测试自定义语法 @! parse -> ast", function () {
  it("should parse", function () {
    expect(getParser(`function @! foo() {}`)()).toMatchSnapshot();
  });
});
```
可通过下面的命令执行 jest：
```bash
# 通过下面的命令执行 jest
 $ BABEL_ENV=test node_modules/.bin/jest -u packages/babel-parser/test/test-parser.js
 ```

 执行结果报错信息：
 ```bash

  测试自定义语法 @@ parse -> ast
    ✕ should parse

测试自定义语法 @@ parse -> ast > should parse
    SyntaxError: Unexpected token (1:9)
        at instantiate (/babel-lib/packages/babel-parser/lib/parse-error/credentials.js:27:32)
        at constructor (/babel-lib/packages/babel-parser/lib/parse-error.js:41:41)
        at Parser.raise (/babel-lib/packages/babel-parser/lib/tokenizer/index.js:1062:19)
        at Parser.unexpected (/babel-lib/packages/babel-parser/lib/tokenizer/index.js:1095:16)
        at Parser.parseIdentifierName (/babel-lib/packages/babel-parser/lib/parser/expression.js:1641:18)
        at Parser.parseIdentifier (/babel-lib/packages/babel-parser/lib/parser/expression.js:1624:23)
        at Parser.parseFunctionId (/babel-lib/packages/babel-parser/lib/parser/statement.js:991:79)
        at Parser.parseFunction (/babel-lib/packages/babel-parser/lib/parser/statement.js:961:22)
        at Parser.parseFunctionStatement (/babel-lib/packages/babel-parser/lib/parser/statement.js:603:17)
        at Parser.parseStatementContent (/babel-lib/packages/babel-parser/lib/parser/statement.js:272:21) {
      code: 'BABEL_PARSER_SYNTAX_ERROR',
      reasonCode: 'UnexpectedToken',
      loc: Position { line: 1, column: 9, index: 9 },
      pos: [Getter/Setter]
    }

```

观察错误信息，发现关键错误点在 `parseIdentifierName` 里报 `unexpected` 错误，下面调试下逻辑。

```javascript
// packages/babel-parser/src/parser/expression.ts

 parseIdentifierName(liberal?: boolean): string {
    let name: string;

    const { startLoc, type } = this.state;

    console.log("identifier value", this.state.value);
    console.log("next identifier value", this.lookahead().value);

    if (tokenIsKeywordOrIdentifier(type)) {
      name = this.state.value;
    } else {
      throw this.unexpected();
    }
    // ...
 }

```

所以问题是 babel 不认识 @! 标识符，那么处理一下：
```javascript
// packages/babel-parser/src/tokenizer/types.js

export const tt = {
    // ...
    at: createToken("@"),
    atEM: createToken("@!"),
}
```
```javascript
// packages/babel-parser/src/parser/statement.ts

 parseFunction<T extends N.NormalFunction>(
    this: Parser,
    node: Undone<T>,
    statement: number = FUNC_NO_FLAGS,
    isAsync: boolean = false,
  ): T {
    // ...
    node.generator = this.eat(tt.star);
    // 实现自定义语法 @！
    node.logThisFunc = this.eat(tt.atEM);
  }
```
再次执行 jest 成功:
![执行成功](https://lilong7676-picture.oss-cn-hangzhou.aliyuncs.com/img/20221209155741.png)

查看 ast 结果，logThisFunc 已标记：
![查看 ast 结果](https://lilong7676-picture.oss-cn-hangzhou.aliyuncs.com/img/20221209155905.png)

// TODO 编写插件，实现转换逻辑