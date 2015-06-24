# Console#

### 稳定度: 2 - 稳定

这个模块定义了一个控制台类，并且暴露了一个`console`对象。

`console`对象是一个特殊的`Console`实例，它的输出被传至`stdout`或`stderr`。

为了使用的方便，`console`被定义为一个全局对象，不需要通过`require`就可直接使用。

#### console#
 - Object
  
用来向`stdout`和`stderr`打印信息。与大多数浏览器提供的`console`对象的功能类似，只是这里输出被传至`stdout`或`stderr`。

当目的地是终端或文件时（为了避免过早退出丢失信息），`console`函数时同步的。当目的地是管道时（为了避免长时间阻塞），`console`函数时异步的。

下面的例子里，`stdout`是非阻塞的，`stderr`是阻塞的：

```SHELL
$ node script.js 2> error.log | tee info.log
```

日常使用时，除了你需要记录大量数量的数据，你不用担心阻塞/非阻塞。

#### console.log([data][, ...])#

向`stdout`打印一行新信息。这个函数可以像`printf()`那样接受多个参数，例子：

```js
var count = 5;
console.log('count: %d', count);
// prints 'count: 5'
```

如果第一个字符串中没有发现格式化元素，那么`util.inspect`将被应用到各个参数。详情参阅`util.format()`。

#### console.info([data][, ...])#

与`console.log`相同。

#### console.error([data][, ...])#

与`console.log`相同。但是输出至`stderr`。

#### console.warn([data][, ...])#

与`console.err`相同。

#### console.dir(obj[, options])#

对`obj`调用`util.inspect`并且将结果字符串输出至`stdout`。这个函数会忽略`obj`上的任何自定义`inspect()`函数。一个可选的`options`参数可以被传递用来格式化字符串的某些方面：

 - showHidden - 如果为`true`，`object`的不可枚举和标志属性也会被显示。默认为`false`。

 - depth - 告诉`inspect`在格式化对象时递归多少次。在检查大而复杂的对象时很有用。默认为2。若要递归到底则传递`null`。

 - colors - 如果为`true`，那么输出会以ANSI颜色码的形式输出。默认为`false`。颜色是可以自定义，参阅下文。

#### console.time(label)#

被用来计算指定操作之间时间间隔。为了开始一个`timer`，调用`console.time()`方法，作为唯一参数可以给它一个名字。为了关闭一个`timer`，并且得到毫秒间隔，仅仅以相同的名字参数调用一次`console.timeEnd()`。

#### console.timeEnd(label)#

停止一个之前通过`console.time()`开启的`timer`，并且向控制台打印结果。

例子：

```js
console.time('100-elements');
for (var i = 0; i < 100; i++) {
  ;
}
console.timeEnd('100-elements');
// prints 100-elements: 262ms
```

#### console.trace(message[, ...])#

向`stderr`打印`'Trace :'`，跟随着格式化信息和堆栈信息。

#### console.assert(value[, message][, ...])#

与`assert.ok()`类似，但是错误信息被像`util.format(message...)`一样格式化。

#### Class: Console#

使用`require('console')`后。`Console`或`console.Console`可以取得这个类。

```js
var Console = require('console').Console;
var Console = console.Console;
```

你可以调用`Console`类来自定义如`console`一样的简单日记记录器，但是有不同的输出流。

#### new Console(stdout[, stderr])#

通过传递一个或两个可写流实例，创建一个新的`Console`。`stdout`是一个用来打印日志和信息的输出流。`stderr`是一个被用来打印警告和错误输出的。如果`stderr`没有被传递，那么警告和错误信息将被传递至`stdout`。

```js
var output = fs.createWriteStream('./stdout.log');
var errorOutput = fs.createWriteStream('./stderr.log');
// custom simple logger
var logger = new Console(output, errorOutput);
// use it like console
var count = 5;
logger.log('count: %d', count);
// in stdout.log: count 5
```

全局的`console`是一个特殊的`Console`实例，它的输出被传递至`process.stdout`和`process.stderr`：

```js
new Console(process.stdout, process.stderr);
```