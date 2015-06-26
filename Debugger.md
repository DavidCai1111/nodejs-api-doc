# Debugger#

稳定度: 2 - 稳定

V8自带了一个强大的调试器，可以从外部通过TCP协议访问。`io.js`为这个调试器内建了一个客户端。要使用它的话，使用`debug`参数启动`io.js`；会出现提示符：

```SHELL
% iojs debug myscript.js
< debugger listening on port 5858
connecting... ok
break in /home/indutny/Code/git/indutny/myscript.js:1
  1 x = 5;
  2 setTimeout(function () {
  3   debugger;
debug>
```

`io.js`的调试器客户端并未支持所有的命令，但是简单的步进和调试都是可以的。通过在源代码就放置`debugger;`语句，你可以启用一个断点。

例如，假设又一个这样的`myscript.js`：

```js
// myscript.js
x = 5;
setTimeout(function () {
  debugger;
  console.log("world");
}, 1000);
console.log("hello");
```

那么一旦你打开调试器，它会在第四行中断。

```SHELL
% iojs debug myscript.js
< debugger listening on port 5858
connecting... ok
break in /home/indutny/Code/git/indutny/myscript.js:1
  1 x = 5;
  2 setTimeout(function () {
  3   debugger;
debug> cont
< hello
break in /home/indutny/Code/git/indutny/myscript.js:3
  1 x = 5;
  2 setTimeout(function () {
  3   debugger;
  4   console.log("world");
  5 }, 1000);
debug> next
break in /home/indutny/Code/git/indutny/myscript.js:4
  2 setTimeout(function () {
  3   debugger;
  4   console.log("world");
  5 }, 1000);
  6 console.log("hello");
debug> repl
Press Ctrl + C to leave debug repl
> x
5
> 2+2
4
debug> next
< world
break in /home/indutny/Code/git/indutny/myscript.js:5
  3   debugger;
  4   console.log("world");
  5 }, 1000);
  6 console.log("hello");
  7
debug> quit
%
```

`repl`命令允许你远程地执行代码。`next`命令步进到下一行。还有一些其他的可用命令，输入`help`查看它们。

#### Watchers#

在调试代码时，你可监视表达式和变量的值。在每个断点，监视器列表上的每个表达式会被在当前上下文执行，并且断点的源代码前展示。

为了开始监视一个表达式，输入`watch("my_expression")`。`watchers`打印可用的监视器。为了移除一个监视器，输入`unwatch("my_expression")`。

#### 命令参考

__Stepping__
 - cont, c - 继续执行
 - next, n - 下一步
 - step, s - 介入（Step in）
 - out, o - 离开（Step out）
 - pause - 暂停代码执行（类似开发者工具中的暂停按钮）

__Breakpoints__
 - setBreakpoint(), sb() - 在当前行设置一个断点
 - setBreakpoint(line), sb(line) -  在指定行设置一个断点
 - setBreakpoint('fn()'), sb(...) - 在函数体的第一个语句上设置断点
 - setBreakpoint('script.js', 1), sb(...) - 在`script.js`的第一行设置断点
 - clearBreakpoint('script.js', 1), cb(...) - 清除`script.js`第一行的断点

同样也可以在一个还未载入的文件（模块）中设置断点：

```SHELL
% ./iojs debug test/fixtures/break-in-module/main.js
< debugger listening on port 5858
connecting to port 5858... ok
break in test/fixtures/break-in-module/main.js:1
  1 var mod = require('./mod.js');
  2 mod.hello();
  3 mod.hello();
debug> setBreakpoint('mod.js', 23)
Warning: script 'mod.js' was not loaded yet.
  1 var mod = require('./mod.js');
  2 mod.hello();
  3 mod.hello();
debug> c
break in test/fixtures/break-in-module/mod.js:23
 21
 22 exports.hello = function() {
 23   return 'hello from module';
 24 };
 25
debug>
```

__Info__
 - backtrace, bt - 打印当前执行框架的回溯
 - list(5) - 列出脚本源代码的5行上下文（前5行和后5行）
 - watch(expr) - 为监视列表添加表达式
 - unwatch(expr) - 从监视列表中移除表达式
 - watchers - 列出所有的监视器和它们的值（会在每一个断点自动列出）
 - repl - 在所调试的脚本上下文中打开调试器的`repl`

__Execution control__
 - run - 运行脚本（在调试器开始时自动运行）
 - restart - 重启脚本
 - kill - 结束脚本

__Various__
 - scripts - 列出所有载入的脚本
 - version - 展示V8版本

#### 高级使用

V8调试器可以通过 使用`--debug`命令行参数打开`io.js` 或 向一个已存在的`io.js`进程发送`SIGUSR1`信号 来启用。

一旦一个进程被设置为了调试模式，它就可以被连接到`io.js`调试器。可以通过pid或URI来连接，语法为：

 - iojs debug -p <pid> - 通过pid连接进程
 - iojs debug <URI> - 通过URI（如`localhost:5858`）连接进程