---
title: 浏览器原理-js执行原理
date: 2022-11-13 20:54:35
tags: [JavaScript, 浏览器]
categories: [JavaScript]
index_img: /img/img1.jpg
banner_img: /img/img1.jpg
---

1. 浏览器如何执行 javascript 代码？🤔

在我们浏览器里输入 url 地址时，浏览器会先解析。如果是域名，则会获取对应的 ip 地址，如果有资源存在缓存中，先判断是否过期，没过期直接去缓存里取。过期后就会像正式服务器发起请求，一般网页会返回一个 html 文件，然后浏览器去解析 HTML。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4b24fc2d6ae942fe8485621d7035524e~tplv-k3u1fbpfcp-zoom-1.image)

2. 浏览器哪些部分负责解析呢？解析完怎么渲染呢？🤔

浏览器内核： 排版引擎（layout engine）,页面渲染引擎（rendering engine）

常见的浏览器内核： webkit，由两部分组成 webcore 和 jscore。webcore 常用于 html 解析，布局渲染，等；jscore 负责解析 js 代码

渲染流程（无 js）：

1.  解析 html，形成 dom tree
1.  解析 css，形成 css 规则
1.  dom 树和 css 规则进行连接合合并之后进行 layout 排版，形成 render 树
1.  生成 render 树之后就可以 paint 绘制，最后进行 display

经典流程图

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8c21c28be3624574a732074edba9016a~tplv-k3u1fbpfcp-zoom-1.image)

3.  如果在渲染的过程中，遇到 js 时，怎么解析呢？😲

js 是高级语言，需要转成机器指令给 cpu 执行。

常见的 js 引擎:

1.  jscore： webkit 内核中的 js 引擎，苹果公司开发（原生小程序用 jscore 来解析 js 代码）
1.  v8 引擎：谷歌开发

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/31a5e5f82ebe4c7b802dbadc1c2db325~tplv-k3u1fbpfcp-zoom-1.image)

- 首先对 js 代码进行词法分析和语法分析，把 js 代码转换成 ast 抽象语法树（<https://astexplorer.net/>）
- Ignition 是一个解释器，会将 AST 转换成 ByteCode（字节码）
- TurboFan 是一个编译器，可以将字节码编译为 CPU 可以直接执行的机器码；

如果一个函数被多次调用，那么就会被标记为热点函数，那么就会经过 TurboFan 转换成优化的机器码，提高代码的执行性能。

但是，机器码实际上也会被还原为 ByteCode，这是因为如果后续执行函数的过程中，类型发生了变化（比如 sum 函数原来执行的是 number 类型，后来执行变成了 string 类型），之前优化的机器码并不能正确的处理运算，就会逆向的转换成字节码；

4.  js 全局代码执行和作用域提升

上面讲到了 v8 如何执行 js 代码，下面我们写个简单的代码，来探索一下 js 的执行过程。

```
var name = "wkb";

var num1 = 20;
var num2 = 30;

var result = num1 + num2;
console.log(result);
```

我们知道，v8 会先解析 js 代码，在解析的过程中会创建一个全局对象 GlobalObject（又称 go）。

- go 里存放着 v8 自定义的对象以及方法等等，然后把我们定义的变量放到 go 中并赋值 undefined，如下

```
var GlobalObject = {
  Date: Object,
  String: Object,
  Window: GlobalObject,
  name : undefined,
  num1 : undefined,
  num2 : undefined,
  result: undefined
}
```

这是解析的过程，然后 v8 开始执行我们的代码。

- 为了执行 js 代码，v8 内部会创建一个执行上下文栈 Execution Context Stack（又称函数调用栈，ECS）。这个栈结构存放着函数执行顺序。

但是我们定义的是变量没有函数，那怎么将变量放入到执行上下文栈中呢？

- v8 引擎为了全局代码能执行，会创建一个全局执行上下文 Global Execution Context Stack（GECS）,其中这个全局执行上下文栈中有维护了一个对象 variable object（vo）变量对象，用来存放定义的变量以及函数。

执行过程如下

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0be118d6014144818a5f848f060a15eb~tplv-k3u1fbpfcp-zoom-1.image)

了解了 js 全局代码的执行过程，我们来看一下作用域的提升。

```
var name = "wkb";
console.log(num1);
var num1 = 20;
```

如果我们在定义 num1 之前打印 num1 会输出啥呢？

答案就是会输出 undefined。因为我们知道，执行代码之前首先要编译代码，这时变量会被赋值到 go 里，并用 undefined 初始化，所以会输出 undefined，这也是作用域的提升。
