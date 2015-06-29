# REPL#

### 稳定度: 2 - 稳定

一个 读取-执行-打印-循环（REPL）可以用于单独的程序，也能很容易的被集成在其他程序中。`REPL`提供了一种交互着运行`JavaScript`然后查看结果的方式。它可以被用来调试，测试或只是尝试一些东西。

在命令行中不带任何参数直接执行`iojs`，你会进入REPL界面。它有一个极简的emacs行编辑器。

```SHELL
mjr:~$ iojs
Type '.help' for options.
> a = [ 1, 2, 3];
[ 1, 2, 3 ]
> a.forEach(function (v) {
...   console.log(v);
...   });
1
2
3
```

要使用高级的行编辑器的话，带着环境变量`NODE_NO_READLINE=1`启动`io.js`。它将会在允许你使用`rlwrap`的终端设置中，启动一个主要的调试`REPL`（main and debugger REPL）。

例如，你可以把以下内容加入`bashrc`文件：

```SHELL
alias iojs="env NODE_NO_READLINE=1 rlwrap iojs"
```

内置的`REPL`（通过运行`iojs`或`iojs -i`启动）可以被以下环境变量所控制：

 - NODE_REPL_HISTORY_FILE - 如果指定，那必须是一个用户可读也可写的文件路径。当给定了一个可用的路径，将启用持久化的历史记录支持：`REPL`历史记录将会跨`iojs``REPL`会话持久化。
 - NODE_REPL_HISTORY_SIZE - 默认为`1000`。与`NODE_REPL_HISTORY_FILE`结合，控制需要持久化的历史记录数量。必须为正数。
 - NODE_REPL_MODE - 可以是`sloppy`，`strict`或`magic`中的一个。默认为`magic`，会自动在严格模式中执行`"strict mode only"`声明。

#### repl.start(options)#

返回并启动一个`REPLServer`实例，继承于`[Readline Interface][]`。接受一个包含以下值得`options`对象：

 - prompt - 所有`I/O`的提示符。默认为`>`。

 - input - 监听的可读流。默认为`process.stdin`。

 - output - 输出数据的可写流。默认为`process.stdout`。

 - terminal - 如果流需要被像TTY对待，并且有`ANSI/VT100`转义代码写入，设置其为`true`。默认为在实例化时检查到的`output`流的`isTTY`属性。

 - eval - 被用来执行每一行的函数。默认为被异步包装过的`eval()`。参阅下文的自定义`eval`的例子。

 - useColors - 一个表明了是否`writer`函数需要输出颜色的布尔值。如果设置了不同的`writer`函数，那么它什么都不会做。默认为`REPL`的终端值。

 - useGlobal - 若设置为`true`，那么`REPL`将使用全局对象，而不是运行每一个脚本在不同上下文中。默认为`false`。

 - ignoreUndefined - 若设置为`true`，那么如果返回值是`undefined`，`REPL`将不会输出它。默认为`false`。

 - writer - 当每一个命令被执行完毕时，都会调用这个函数，它返回了展示的格式（包括颜色）。默认为`util.inspect`。

 - __replMode__ - 控制是否`REPL`运行所有的模式在严格模式，默认模式，或混合模式（`"magic"`模式）。接受以下值：

  - repl.REPL_MODE_SLOPPY - 在混杂模式下运行命令。
  - repl.REPL_MODE_STRICT - 在严格模式下运行命令。这与在每个命令前添加`'use strict'`语句相等。
  - repl.REPL_MODE_MAGIC - 试图在默认模式中运行命令，如果失败了，会重新尝试使用严格模式。

你可以使用你自己的`eval`函数，如果它包含以下签名：

```js
function eval(cmd, context, filename, callback) {
  callback(null, result);
}
```

在用tab补全时 - `eval`将会带着一个作为输入字符串的`.scope`调用。它被期望返回一个`scope`名字数组，被用来自动补全。

多个`REPL`可以运行相同的`io.js`实例。共享同一个全局对象，但是各自的I/O独立。

下面是在`stdin`，Unix `socket` 和 TCP `socket` 上启动一个`REPL`的例子：

```js
var net = require("net"),
    repl = require("repl");

connections = 0;

repl.start({
  prompt: "io.js via stdin> ",
  input: process.stdin,
  output: process.stdout
});

net.createServer(function (socket) {
  connections += 1;
  repl.start({
    prompt: "io.js via Unix socket> ",
    input: socket,
    output: socket
  }).on('exit', function() {
    socket.end();
  })
}).listen("/tmp/iojs-repl-sock");

net.createServer(function (socket) {
  connections += 1;
  repl.start({
    prompt: "io.js via TCP socket> ",
    input: socket,
    output: socket
  }).on('exit', function() {
    socket.end();
  });
}).listen(5001);
```

在命令行中运行这个程序会在`stdin`上启动一个`REPL`。另外的`REPL`客户端将会通过Unix `socket`或TCP `socket`连接。`telnet`在连接TCP `socket`时非常有用，`socat`在连接Unix `socket`和TCP `socket`时都非常有用。

通过从基于Unix `socket` 的服务器启动`REPL`，你可以不用重启，而连接到一个长久执行的（long-running）`io.js`进程。

一个通过`net.Server`和`net.Socket`实例运行“全特性”（终端）`REPL`的例子，参阅`https://gist.github.com/2209310`。

一个通过`curl(1)`运行`REPL`的例子，参阅`https://gist.github.com/2053342`。

#### Event: 'exit'#

 - function () {}

当用户通过任意一种已定义的方式退出`REPL`时触发。具体地说，在`REPL`中键入`.exit`，两次按下`Ctrl+C`来发送`SIGINT`信号，按下`Ctrl+D`来发送结束信号。

例子：

```js
r.on('exit', function () {
  console.log('Got "exit" event from repl!');
  process.exit();
});
```

#### Event: 'reset'#

 - function (context) {}

当`REPL`内容被重置时触发。当你键入`.clear`时发生。如果你以`{ useGlobal: true }`启动`REPL`，那么这个事件将永远不会触发。

例子：

```js
// Extend the initial repl context.
r = repl.start({ options ... });
someExtension.extend(r.context);

// When a new context is created extend it as well.
r.on('reset', function (context) {
  console.log('repl has a new context');
  someExtension.extend(context);
});
```

### REPL 特性

在`REPL`内，按下`Control+D`将会退出。多行表达式可以被输入。Tab补全同时支持全局和本地变量。

核心模块将会被按需载入环境。例如，调用`fs`，将会从`global.fs`获取，作为`require()``fs`模块的替代。

特殊的变量`_`（下划线）包含了上一个表达式的结果。

```SHELL
> [ "a", "b", "c" ]
[ 'a', 'b', 'c' ]
> _.length
3
> _ += 1
4
```

`REPL`可以访问全局作用域里的任何变量。你可以通过将变量赋值给一个关联了所有`REPLServer`的`context`对象来暴露一个对象给`REPL`。例子：

```js
// repl_test.js
var repl = require("repl"),
    msg = "message";

repl.start("> ").context.m = msg;
```

`context`对象里的对象会表现得像`REPL`的本地变量：

```SHELL
mjr:~$ iojs repl_test.js
> m
'message'
```

以下是一些特殊的`REPL`命令：

 - .break - 当你输入一个多行表达式时，有时你走神了，或有时你不关心如何完成它了。`.break`将会让你重新来过。
 - .clear - 重置`context`对象为一个空对象并且清除所有多行表达式。
 - .exit - 关闭I/O流，意味着会导致`REPL`退出。
 - .help - 展示特殊命令列表。
 - __.save__ 将当前的`REPL`会话保存入一个文件。
  - `.save ./file/to/save.js`

 - __.load__ - 从一个文件中加载`REPL`会话。
  - `.load ./file/to/load.js`

这些组合键在`REPL`中有以下影响：

 - <ctrl>C - 与`.break`关键字相似。终止当前命令。在一个空行上连按两次会强制退出。
 - <ctrl>D - 与`.exit`关键字相似。
 - <tab> - 展示所有的全局和本地变量。