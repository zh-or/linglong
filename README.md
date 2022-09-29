## 原文地址

[链接](https://zhuanlan.zhihu.com/p/486548159)

## 前言
谈及 Babel,必然离不开 AST。有关 AST 这个知识点其实是很重要的，但由于涉及到代码编译阶段，大多情况都是由各个框架内置相关处理，所以作为开发(使用)者本身，往往会忽视这个过程。希望通过这篇文章，带各位同学走进 AST，借助 AST 发挥更多的想象力。

## AST 概述
想必大家总是听到 AST 这个概念，那么到底什么是 AST？

AST 全称是是 Abstract Syntax Tree，中文为抽象语法树，将我们所写的代码转换为机器能识别的一种树形结构。其本身是由一堆节点（Node）组成，每个节点都表示源代码中的一种结构。不同结构用类型（Type）来区分，常见的类型有：Identifier(标识符)，Expression(表达式)，VariableDeclaration(变量定义)，FunctionDeclaration(函数定义)等。
AST 结构
随着 JavaScript 的发展，为了统一ECMAScript标准的语法表达。社区中衍生出了ESTree Spec，是目前社区所遵循的一种语法表达标准。

ESTree 提供了例如Identifier、Literal等常见的节点类型。

## 节点类型

| 类型 | 说明 |
| ---- | ---- |
|File	| 文件 (顶层节点包含 Program)|
|Program|	整个程序节点 (包含 body 属性代表程序体)|
|Directive|	指令 (例如 "use strict")|
|Comment	|代码注释|
|Statement|	语句 (可独立执行的语句)|
|Literal	|字面量 (基本数据类型、复杂数据类型等值类型)|
|Identifier	|标识符 (变量名、属性名、函数名、参数名等)|
|Declaration|	声明 (变量声明、函数声明、Import、Export 声明等)|
|Specifier|	关键字 (ImportSpecifier、ImportDefaultSpecifier、ImportNamespaceSpecifier、ExportSpecifier)|
|Expression	|表达式|

## 公共属性
| 类型	| 说明|
| ---- | ---- |
|type	|AST 节点的类型|
|start	|记录该节点代码字符串起始下标|
|end	|记录该节点代码字符串结束下标|
|loc|	内含 line、column 属性，分别记录开始结束的行列号|
|leadingComments	|开始的注释|
|innerComments	|中间的注释|
|trailingComments|	结尾的注释|
|extra|	额外信息|

## AST 示例

有的同学可能会问了，这么多类型都需要记住么? 其实并不是，我们可以借助以下两个工具来查询 AST 结构。

[AST Explorer (常用)](https://astexplorer.net/)

![image](https://user-images.githubusercontent.com/12293174/192929260-83745096-22db-4675-b88e-a56ac5f3d8a9.png)



## [AST 可视化](https://resources.jointjs.com/demos/rappid/apps/Ast/index.html)

![image](https://user-images.githubusercontent.com/12293174/192929303-21b9263f-7488-4508-ba1d-2a9fc3227f35.png)



结合一个示例，带大家快速了解一下 AST 结构。
```
function test(args) {
  const a = 1;
  console.log(args);
}
```
上述代码，声明了一个函数，名为`test`，有一个形参`args`。

函数体中:

声明了一个`const`类型变量`a`，值为` 1`
执行了一个 console.log 语句
将上述代码粘贴至AST Explorer，结果如图所示:


接下来我们继续分析内部结构，以const a = 1为例:


变量声明在 AST 中对应的就是 type 为VariableDeclaration的节点。该节点包含kind和declarations两个必须属性，分别代表声明的变量类型和变量内容。

细心的同学可能发现了declarations是一个数组。这是为什么呢？因为变量声明本身支持const a=1,b=2的写法，需要支持多个VariableDeclarator，故此处为数组。

而 type 为VariableDeclarator的节点代表的就是a=1这种声明语句，其中包含id和init属性。

id即为Identifier,其中的name值对应的就是变量名称。

init即为初始值，包含type,value属性。分别表示初始值类型和初始值。此处 type 为NumberLiteral，表明初始值类型为number类型。

Babel 概述
Babel 是一个 JavaScript 编译器，在实际开发过程中通常借助Babel来完成相关 AST 的操作。

Babel 工作流程

Babel AST
Babel 解析代码后生成的 AST 是以ESTree作为基础，并略作修改。

官方原文如下:

The Babel parser generates AST according to Babel AST format. It is based on ESTree spec with the following deviations:
Literal token is replaced with StringLiteral, NumericLiteral, BigIntLiteral, BooleanLiteral, NullLiteral, RegExpLiteral
Property token is replaced with ObjectProperty and ObjectMethod
MethodDefinition is replaced with ClassMethod
Program and BlockStatement contain additional directives field with Directive and DirectiveLiteral
ClassMethod, ObjectProperty, and ObjectMethod value property's properties in FunctionExpression is coerced/brought into the main method node.
ChainExpression is replaced with OptionalMemberExpression and OptionalCallExpression
ImportExpression is replaced with a CallExpression whose callee is an Import node.
Babel 核心包
工具包	说明
@babel/core	Babel 转码的核心包,包括了整个 babel 工作流（已集成@babel/types）
@babel/parser	解析器，将代码解析为 AST
@babel/traverse	遍历/修改 AST 的工具
@babel/generator	生成器，将 AST 还原成代码
@babel/types	包含手动构建 AST 和检查 AST 节点类型的方法
@babel/template	可将字符串代码片段转换为 AST 节点
npm i @babel/parser @babel/traverse @babel/types @babel/generator @babel/template -D
Babel 插件
Babel 插件大致分为两种：语法插件和转换插件。语法插件作用于 @babel/parser，负责将代码解析为抽象语法树（AST）（官方的语法插件以 babel-plugin-syntax 开头);转换插件作用于 @babel/core，负责转换 AST 的形态。绝大多数情况下我们都是在编写转换插件。

Babel 工作依赖插件。插件相当于是指令,来告知 Babel 需要做什么事情。如果没有插件，Babel 将原封不动的输出代码。

Babel 插件本质上就是编写各种 visitor 去访问 AST 上的节点，并进行 traverse。当遇到对应类型的节点，visitor 就会做出相应的处理，从而将原本的代码 transform 成最终的代码。

export default function (babel) {
  // 即@babel/types，用于生成AST节点
  const { types: t } = babel;

  return {
    name: "ast-transform", // not required
    visitor: {
      Identifier(path) {
        path.node.name = path.node.name.split("").reverse().join("");
      },
    },
  };
}
这是一段AST Explorer上的 transform 模板代码。上述代码的作用即为将输入代码的所有标识符(Identifier)类型的节点名称颠倒。

其实编写一个 Babel 插件很简单。我们要做的事情就是回传一个 visitor 对象，定义以Node Type为名称的函数。该函数接收path,state两个参数。

其中path（路径）提供了访问/操作AST 节点的方法。path 本身表示两个节点之间连接的对象。例如path.node可以访问当前节点，path.parent可以访问父节点等。path.remove()可以移除当前节点。具体 API 见下图。其他可见
handlebook。


Babel Types
Babel Types 模块是一个用于 AST 节点的 Lodash 式工具库，它包含了构造、验证以及变换 AST 节点的方法。

类型判断
Babel Types 提供了节点类型判断的方法，每一种类型的节点都有相应的判断方法。更多见babel-types API。

import * as types from "@babel/types";

// 是否为标识符类型节点
if (types.isIdentifier(node)) {
  // ...
}

// 是否为数字字面量节点
if (types.isNumberLiteral(node)) {
  // ...
}

// 是否为表达式语句节点
if (types.isExpressionStatement(node)) {
  // ...
}
创建节点
Babel Types 同样提供了各种类型节点的创建方法，详见下属示例。

注: Babel Types 生成的 AST 节点需使用@babel/generator转换后得到相应代码。

import * as types from "@babel/types";
import generator from "@babel/generator";

const log = (node: types.Node) => {
  console.log(generator(node).code);
};

log(types.stringLiteral("Hello World")); // output: Hello World
基本数据类型
types.stringLiteral("Hello World"); // string
types.numericLiteral(100); // number
types.booleanLiteral(true); // boolean
types.nullLiteral(); // null
types.identifier(); // undefined
types.regExpLiteral("\\.js?$", "g"); // 正则

"Hello World"
100
true
null
undefined
/\.js?$/g
复杂数据类型
数组
types.arrayExpression([
  types.stringLiteral("Hello World"),
  types.numericLiteral(100),
  types.booleanLiteral(true),
  types.regExpLiteral("\\.js?$", "g"),
]);

["Hello World", 100, true, /\.js?$/g];
对象
types.objectExpression([
  types.objectProperty(
    types.identifier("key"),
    types.stringLiteral("HelloWorld")
  ),
  types.objectProperty(
    // 字符串类型 key
    types.stringLiteral("str"),
    types.arrayExpression([])
  ),
  types.objectProperty(
    types.memberExpression(
      types.identifier("obj"),
      types.identifier("propName")
    ),
    types.booleanLiteral(false),
    // 计算值 key
    true
  ),
]);

{
  key: "HelloWorld",
  "str": [],
  [obj.propName]: false
}
JSX 节点
创建 JSX AST 节点与创建数据类型节点略有不同，此处整理了一份关系图。


JSXElement
types.jsxElement(
types.jsxOpeningElement(types.jsxIdentifier("Button"), []),
types.jsxClosingElement(types.jsxIdentifier("Button")),
[types.jsxExpressionContainer(types.identifier("props.name"))]
);

<Button>{props.name}</Button>
JSXFragment
types.jsxFragment(types.jsxOpeningFragment(), types.jsxClosingFragment(), [
types.jsxElement(
  types.jsxOpeningElement(types.jsxIdentifier("Button"), []),
  types.jsxClosingElement(types.jsxIdentifier("Button")),
  [types.jsxExpressionContainer(types.identifier("props.name"))]
),
types.jsxElement(
  types.jsxOpeningElement(types.jsxIdentifier("Button"), []),
  types.jsxClosingElement(types.jsxIdentifier("Button")),
  [types.jsxExpressionContainer(types.identifier("props.age"))]
),
]);

<>
<Button>{props.name}</Button>
<Button>{props.age}</Button>
</>
声明
变量声明 (variableDeclaration)
types.variableDeclaration("const", [
types.variableDeclarator(types.identifier("a"), types.numericLiteral(1)),
]);

const a = 1;
函数声明 (functionDeclaration)
  types.functionDeclaration(
    types.identifier("test"),
    [types.identifier("params")],
    types.blockStatement([
      types.variableDeclaration("const", [
        types.variableDeclarator(
          types.identifier("a"),
          types.numericLiteral(1)
        ),
      ]),
      types.expressionStatement(
        types.callExpression(types.identifier("console.log"), [
          types.identifier("params"),
        ])
      ),
    ])
  );

  function test(params) {
    const a = 1;
    console.log(params);
  }
React 函数式组件
综合上述内容，小小实战一下~

我们需要通过 Babel Types 生成button.js代码。乍一看不知从何下手?

// button.js
import React from "react";
import { Button } from "antd";

export default (props) => {
  const handleClick = (ev) => {
    console.log(ev);
  };
  return <Button onClick={handleClick}>{props.name}</Button>;
};
小技巧: 先借助AST Explorer网站，观察 AST 树结构。然后通过 Babel Types 逐层编写代码。事半功倍！

types.program([
  types.importDeclaration(
    [types.importDefaultSpecifier(types.identifier("React"))],
    types.stringLiteral("react")
  ),
  types.importDeclaration(
    [
      types.importSpecifier(
        types.identifier("Button"),
        types.identifier("Button")
      ),
    ],
    types.stringLiteral("antd")
  ),
  types.exportDefaultDeclaration(
    types.arrowFunctionExpression(
      [types.identifier("props")],
      types.blockStatement([
        types.variableDeclaration("const", [
          types.variableDeclarator(
            types.identifier("handleClick"),
            types.arrowFunctionExpression(
              [types.identifier("ev")],
              types.blockStatement([
                types.expressionStatement(
                  types.callExpression(types.identifier("console.log"), [
                    types.identifier("ev"),
                  ])
                ),
              ])
            )
          ),
        ]),
        types.returnStatement(
          types.jsxElement(
            types.jsxOpeningElement(types.jsxIdentifier("Button"), [
              types.jsxAttribute(
                types.jsxIdentifier("onClick"),
                types.jSXExpressionContainer(types.identifier("handleClick"))
              ),
            ]),
            types.jsxClosingElement(types.jsxIdentifier("Button")),
            [types.jsxExpressionContainer(types.identifier("props.name"))],
            false
          )
        ),
      ])
    )
  ),
]);
应用场景
AST 本身应用非常广泛，例如:Babel 插件(ES6 转化 ES5)、构建时压缩代码 、css 预处理器编译、 webpack 插件等等，可以说是无处不在。


如图所示，不难发现，一旦涉及到编译，或者说代码本身的处理，都和 AST 息息相关。下面列举了一些常见应用，让我们看看是如何处理的。

代码转换
// ES6 => ES5 let 转 var
export default function (babel) {
  const { types: t } = babel;

  return {
    name: "let-to-var",
    visitor: {
      VariableDeclaration(path) {
        if (path.node.kind === "let") {
          path.node.kind = "var";
        }
      },
    },
  };
}
babel-plugin-import
在 CommonJS 规范下，当我们需要按需引入antd的时候，通常会借助该插件。

该插件的作用如下：

// 通过es规范，具名引入Button组件
import { Button } from "antd";
ReactDOM.render(<Button>xxxx</Button>);

// babel编译阶段转化为require实现按需引入
var _button = require("antd/lib/button");
ReactDOM.render(<_button>xxxx</_button>);
简单分析一下，核心处理: 将 import 语句替换为对应的 require 语句。

export default function (babel) {
  const { types: t } = babel;

  return {
    name: "import-to-require",
    visitor: {
      ImportDeclaration(path) {
        if (path.node.source.value === "antd") {
          // var _button = require("antd/lib/button");
          const _botton = t.variableDeclaration("var", [
            t.variableDeclarator(
              t.identifier("_button"),
              t.callExpression(t.identifier("require"), [
                t.stringLiteral("antd/lib/button"),
              ])
            ),
          ]);
          // 替换当前import语句
          path.replaceWith(_botton);
        }
      },
    },
  };
}
TIPS: 目前 antd 包中已包含esm规范文件，可以依赖 webpack 原生 TreeShaking 实现按需引入。

LowCode 可视化编码
当下LowCode，依旧是前端一大热门领域。目前主流的做法大致下述两种。

Schema 驱动
目前主流做法，将表单或者表格的配置，描述为一份 Schema，可视化设计器基于 Schema 驱动，结合拖拽能力，快速搭建。
AST 驱动
通过CloudIDE，CodeSandbox等浏览器端在线编译，编码。外加可视化设计器，最终实现可视化编码。

大致流程如上图所示，既然涉及到代码修改，离不开AST的操作，那么又可以发挥 babel 的能力了。

假设设计器的初始代码如下:

import React from "react";

export default () => {
  return <Container></Container>;
};
此时我们拖拽了一个Button至设计器中，根据上图的流程，核心的 AST 修改过程如下:

新增 import 声明语句 import { Button } from "antd";
将插入至
话不多说，直接上代码:

import traverse from "@babel/traverse";
import generator from "@babel/generator";
import * as parser from "@babel/parser";
import * as t from "@babel/types";

// 源代码
const code = `
  import React from "react";

  export default () => {
    return <Container></Container>;
  };
`;

const ast = parser.parse(code, {
  sourceType: "module",
  plugins: ["jsx"],
});

traverse(ast, {
  // 1. 程序顶层 新增import语句
  Program(path) {
    path.node.body.unshift(
      t.importDeclaration(
        // importSpecifier表示具名导入，相应的匿名导入为ImportDefaultSpecifier
        // 具名导入对应代码为 import { Button as Button } from 'antd'
        // 如果相同会自动合并为 import { Button } from 'antd'
        [t.importSpecifier(t.identifier("Button"), t.identifier("Button"))],
        t.stringLiteral("antd")
      )
    );
  },
  // 访问JSX节点，插入Button
  JSXElement(path) {
    if (path.node.openingElement.name.name === "Container") {
      path.node.children.push(
        t.jsxElement(
          t.jsxOpeningElement(t.jsxIdentifier("Button"), []),
          t.jsxClosingElement(t.jsxIdentifier("Button")),
          [t.jsxText("按钮")],
          false
        )
      );
    }
  },
});

const newCode = generator(ast).code;
console.log(newCode);
结果如下:

import { Button } from "antd";
import React from "react";
export default () => {
  return (
    <Container>
      <Button>按钮</Button>
    </Container>
  );
};
ESLint
自定义 eslint-rule,本质上也是访问 AST 节点，是不是跟 Babel 插件的写法很相似呢？

module.exports.rules = {
  "var-length": (context) => ({
    VariableDeclarator: (node) => {
      if (node.id.name.length <= 2) {
        context.report(node, "变量名长度需要大于2");
      }
    },
  }),
};
Code2Code
以 Vue To React 为例，大致过程跟ES6 => ES5类似，通过vue-template-compiler编译得到Vue AST => 转换为 React AST => 输出 React 代码。

有兴趣的同学可以参考 vue-to-react

其他多端框架：一份代码 => 多端，大体思路一致。

总结
在实际开发中，遇到的情况往往更加复杂，建议大家多番文档，多观察，用心去感受 ~

参考文章
babel-handlebook
@babel/types
透過製作 Babel-plugin 初訪 AST
@babel/types 深度应用
