# Errors#

`node.js`生成的错误分为两类：`JavaScript`错误和系统错误。所有的错误都继承于`JavaScript`的`Error`类，或就是它的实例。并且都至少提供这个类中可用的属性。

当一个操作因为语法错误或语言运行时级别（language-runtime-level）的原因不被允许时，一个`JavaScript error`会被生成并抛出一个异常。如果一个操作因为系统级别（system-level）限制而不被允许时，一个系统错误会被生成。客户端代码接着会根据API传播它的方式来被给予捕获这个错误的机会。

API被调用的风格决定了生成的错误如何回送（handed back），传播给客户端。这反过来告诉客户端如何捕获它们。异常可以通过`try / catch`结构捕获；其他的捕获方式请参阅下文。

### JavaScript错误#

`JavaScript`错误表示API被错误的使用了，或者正在写的程序有问题。

#### Class: Error#

一个普通的错误对象。和其他的错误对象不同，`Error`实例不指示任何 为什么错误发生 的原因。`Error`在它们被实例化时，会记录下“堆栈追踪”信息，并且可以会提供一个错误描述。

注意：`node.js`会将系统错误以及`JavaScript`错误都封装为这个类的实例。

#### new Error(message)#

实例化一个新的`Error`对象，并且用提供的`message`设置它的`.message`属性。它的`.stack`属性将会描述`new Error`被调用时程序的这一刻。堆栈追踪信息隶属于V8堆栈追踪API。堆栈追踪信息只延伸到同步代码执行的开始，或`Error.stackTraceLimit`给出的帧数（number of frames），这取决于哪个更小。

#### error.message#

一个在`Error()`实例化时被传递的字符串。这个信息会出现在堆栈追踪信息的第一行。改变这个值将不会改变堆栈追踪信息的第一行。

#### error.stack#

这个属性返回一个代表错误被实例化时程序运行的那个点的字符串。

一个堆栈追踪信息例子：

```SHELL
Error: Things keep happening!
   at /home/gbusey/file.js:525:2
   at Frobnicator.refrobulate (/home/gbusey/business-logic.js:424:21)
   at Actor.<anonymous> (/home/gbusey/actors.js:400:8)
   at increaseSynergy (/home/gbusey/actors.js:701:6)
```

第一行被格式化为`<错误类名>: <错误信息>`，然后是一系列的堆栈信息帧（以`“at”`开头）。每帧都描述了一个最终导致错误生成的一次调用的地点。V8会试图去给出每个函数的名字（通过变量名，函数名或对象方法名），但是也有可能它找不到一个合适的名字。如果V8不能为函数定义一个名字，那么那一帧里只会展示出位置信息。否则，被定义的函数名会显示在位置信息之前。

帧只会由`JavaScript`函数生成。例如，如果在一个`JavaScript`函数里，同步执行了一个叫`cheetahify`的C++ `addon`函数，那么堆栈追踪信息中的帧里将不会有`cheetahify`调用：

```js
var cheetahify = require('./native-binding.node');

function makeFaster() {
  // cheetahify *synchronously* calls speedy.
  cheetahify(function speedy() {
    throw new Error('oh no!');
  });
}

makeFaster(); // will throw:
// /home/gbusey/file.js:6
//     throw new Error('oh no!');
//           ^
// Error: oh no!
//     at speedy (/home/gbusey/file.js:6:11)
//     at makeFaster (/home/gbusey/file.js:5:3)
//     at Object.<anonymous> (/home/gbusey/file.js:10:1)
//     at Module._compile (module.js:456:26)
//     at Object.Module._extensions..js (module.js:474:10)
//     at Module.load (module.js:356:32)
//     at Function.Module._load (module.js:312:12)
//     at Function.Module.runMain (module.js:497:10)
//     at startup (node.js:119:16)
//     at node.js:906:3
```

位置信息将会是以下之一：

 - `native`，如果帧代表了向V8内部的一次调用（如在`[].forEach`中）。
 - `plain-filename.js:line:column`，如果帧代表了向`node.js`内部的一次调用。
 - `/absolute/path/to/file.js:line:column`，如果帧代表了向用户程序或其依赖的一次调用。

关键的一点是，代表了堆栈信息的字符串只在需要被使用时生成，它是惰性生成的。

堆栈信息的帧数由 `Error.stackTraceLimit` 或 当前事件循环的`tick`里可用的帧数 中小的一方决定。

系统级别错误被作为增强的`Error`实例生成，参阅下文。

#### Error.captureStackTrace(targetObject[, constructorOpt])#

为`targetObject`创建一个`.stack`属性，它代表了`Error.captureStackTrace`被调用时，在程序中的位置。

```js
var myObject = {};

Error.captureStackTrace(myObject);

myObject.stack  // similar to `new Error().stack`
```

追踪信息的第一行，将是`targetObject.toString()`的结果，而不是一个带有`ErrorType: `前缀的信息。

可选的`constructorOpt`接收一个函数。如果指定，所有`constructorOpt`以上的帧，包括`constructorOpt`，将会被生成的堆栈追踪信息忽略。

这对于向最终用户隐藏实现细节十分有用。一个普遍的使用这个参数的例子：

```js
function MyError() {
  Error.captureStackTrace(this, MyError);
}

// without passing MyError to captureStackTrace, the MyError
// frame would should up in the .stack property. by passing
// the constructor, we omit that frame and all frames above it.

new MyError().stack
```

#### Error.stackTraceLimit#

一个决定了堆栈追踪信息的堆栈帧数的属性（不论是由`new Error().stack`或由`Error.captureStackTrace(obj)`生成）。

初始值是`10`。可以被设置为任何有效的`JavaScript`数字，当值被改变后，就会影响所有的堆栈追踪信息的获取。如果设置为一个非数字值，堆栈追踪将不会获取任何一帧，并且会在要使用时报告`undefined`。

#### Class: RangeError#

一个`Error`子类，表明了为一个函数提供的参数没有在可接受的值的范围之内；不论是在一个数字范围之外，或是在一个参数指定的参数集合范围之外。例子：

```js
require('net').connect(-1);  // throws RangeError, port should be > 0 && < 65536
```

`node.js`会立刻生成并抛出一个`RangeError`实例 -- 它们是参数验证的一种形式。

#### Class: TypeError#

一个`Error`子类，表明了提供的参数不是被允许的类型。例如，为一个期望收到字符串参数的函数，传入一个函数作为参数，将导致一个类型错误。

```js
require('url').parse(function() { }); // throws TypeError, since it expected a string
```

`node.js`会立刻生成并抛出一个`TypeError`实例 -- 它们是参数验证的一种形式。

#### Class: ReferenceError#

一个`Error`子类，表明了试图去获取一个未定义的对象的属性。大多数情况下它表明了一个输入错误，或者一个不完整的程序。客户端代码可能会生成和传播这些错误，但实际上只有V8会。

```js
doesNotExist; // throws ReferenceError, doesNotExist is not a variable in this program.
```

`ReferenceError`实例将有一个`.arguments`属性，它是一个包含了一个元素的数组。这个元素表示没有被定义的那个变量。

```js
try {
  doesNotExist;
} catch(err) {
  err.arguments[0] === 'doesNotExist';
}
```

除非用户程序是动态生成并执行的，否则，`ReferenceErrors`应该永远被认为是程序或其依赖模块的bug。

#### Class: SyntaxError#

一个`Error`子类，表明了程序代码不是合法的`JavaScript`。这些错误可能只会作为代码运行的结果生成。代码运行可能是`eval`，`Function`，`require`或`vm`的结果。这些错误经常表明了一个不完整的程序。

```js
try {
  require("vm").runInThisContext("binary ! isNotOk");
} catch(err) {
  // err will be a SyntaxError
}
```

`SyntaxError`对于创建它们的上下文来说是不可恢复的 - 它们仅可能被其他上下文捕获。

#### 异常 vs. 错误

一个`JavaScript`“异常”是一个无效操作或`throw`声明所抛出的结果的值。但是这些值不被要求必须继承于`Error`。所有的由`node.js`或`JavaScript`运行时抛出的异常都必须是`Error`实例。

一些异常在`JavaScript`层是无法恢复的。这些异常通常使一个进程挂掉。它们通常无法通过`assert()`检查，或C++层中的`abort()`调用。

### 系统错误

系统错误在程序运行时环境的响应中生成。理想情况下，它们代表了程序能够处理的操作错误。它们在系统调用级别生成：一个详尽的错误码列表和它们意义可以通过运行`man 2 intro`或`man 3 errno`在大多数`Unices`中获得；或在线获得。

在`node.js`中，系统错误表现为一个增强的`Error`对象 -- 不是完全的子类，而是一个有额外成员的`error`实例。

#### Class: System Error#

#### error.syscall#

一个代表了失败的系统调用的字符串。

#### error.errno#

#### error.code#

一个代表了错误码的字符串，通常是大写字母`E`，可在`man 2 intro`命令的结果中查阅。

#### 常见系统错误

这个列表不详尽，但是列举了许多在写`node.js`的过程中普遍发生的系统错误。详尽的列表可以在这里查阅：`http://man7.org/linux/man-pages/man3/errno.3.html`

#### EPERM: 操作不被允许

试图去执行一个需要特权的操作。

#### ENOENT: 指定的文件或目录不存在

通常由文件操作产生；指定的路径不存在 -- 通过指定的路径不能找到实例（文件或目录）。

#### EACCES: 没有权限

试图以禁止的方式去访问一个需要权限的文件。

#### EEXIST: 文件已存在

执行一个要求目标不存在的操作时，一个已存在文件已经是目标。

#### ENOTDIR: 非目录

给定的路径存在，但不是期望的目录。通常由`fs.readdir`产生。

#### EISDIR: 是目录

一个操作期望接收一个文件，但给定的路径是一个文件。

#### EMFILE: 系统中打开太多文件

达到了系统中允许的文件描述符的最大数量，那么下一个描述符请求，在已存在的最后一个描述符关闭之前，都不能被满足。

通常在并行打开太多文件时触发，特别是在那些将进程可用的文件描述符数量限制得很低的操作系统中（尤其是OS X）。为了改善这个限制，在同一个SHELL中运行`ulimit -n 2048`命令，再运行`node.js`进程。

#### EPIPE: 损坏的管道

向没有读取数据进程的管道，socket或FIFO中执行一个写操作。通常在网络和http层发生，表明需要被写入的远程流已经被关闭。

#### EADDRINUSE: 地址已被使用


试图给一个服务器（net，http或https）绑定一个本地地址失败，因为另一个本地系统中的服务器已经使用了那个地址。

#### ECONNRESET: 连接两方重置（Connection reset by peer）

连接的双方被强行关闭。通常是远程`socket`超时或重启的结果。通常由`http`和`net`模块产生。

#### ECONNREFUSED: 拒绝连接

由于目标机器积极拒绝，没有连接可以建立。通常是试图访问一个不活跃的远程主机的服务的结果。

#### ENOTEMPTY: 目录不为空

操作的实例要求是一个空目录，但目录不为空 -- 通常由`fs.unlink`产生。

#### ETIMEDOUT: 操作超时

因为被连接方在一段指定内未响应，连接或发送请求失败。通常由http或net产生 -- 经常是一个 被连接`socket`没有合适地调用`.end()`方法 的标志。

### 错误的传播和捕获

所有的`node.js`API将无效的参数视作异常 -- 也就是说，如果传递了非法的参数，他们会立刻生成并抛出一个`error`作为异常，甚至是异步API也会。

同步API（像`fs.readFileSync`）将会抛出一个错误。抛出值的行为是将值包装入一个异常。异常可以被使用`try { } catch(err) { }`结果捕获。

异步API有两种错误传播机制；一种代表了单个操作（Node风格的回调函数），另一种代表了多个操作（错误事件）。

#### Node风格的回调函数

单个操作使用`Node风格的回调函数` -- 一个提供给API作为参数的函数。Node风格的回调函数至少有一个参数 -- `error` -- 它可以是`null`（如果没有错误发生）或是`Error`实例。例子：

```js
var fs = require('fs');

fs.readFile('/some/file/that/does-not-exist', function nodeStyleCallback(err, data) {
  console.log(err)  // Error: ENOENT
  console.log(data) // undefined / null
});

fs.readFile('/some/file/that/does-exist', function(err, data) {
  console.log(err)  // null
  console.log(data) // <Buffer: ba dd ca fe>
})
```

注意，`try { } catch(err) { }`不能捕获异步API生成的错误。一个初学者的常见错误是尝试在Node风格的回调函数中抛出错误：

```js
// THIS WILL NOT WORK:
var fs = require('fs');

try {
  fs.readFile('/some/file/that/does-not-exist', function(err, data) {
    // mistaken assumption: throwing here...
    if (err) {
      throw err;
    }
  });
} catch(err) {
  // ... will be caught here -- this is incorrect!
  console.log(err); // Error: ENOENT
}
```

这将不会正常运行！在Node风格的回调函数执行时，外围的代码`try { } catch(err) { }`）已经退出了。在大多数情况，在Node风格的回调函数内部抛出错误会使进程挂掉。如果启用了`domain`，它们可以捕获了被抛出的错误；相似的，如果给`process.on('uncaughtException')`添加了监听器，那么它也将会捕获错误。

#### 错误事件

另一个提供错误的机制是`error`事件。这常被用在基于流或基于`event emitter`的API中，它们自身就代表了一系列的异步操作（每一个单一的操作都可能成功或失败）。如果在错误的源头没有添加`error`事件的监听器，那么`error`会被抛出。此时，进程会因为一个未处理的异常而挂掉，除非提供了合适的`domains`，或监听了`process.on('uncaughtException')`。

```js
var net = require('net');

var connection = net.connect('localhost');

// adding an "error" event handler to a stream:
connection.on('error', function(err) {
  // if the connection is reset by the server, or if it can't
  // connect at all, or on any sort of error encountered by
  // the connection, the error will be sent here.
  console.error(err);
});

connection.pipe(process.stdout);
```

“当没有没有监听错误时会抛出错误”这个行为不仅限与`node.js`提供的API -- 用户创建的基于流或`event emitters`的API也会如此。例子：

```js
var events = require('events');

var ee = new events.EventEmitter;

setImmediate(function() {
  // this will crash the process because no "error" event
  // handler has been added.
  ee.emit('error', new Error('This will crash'));
});
```

与Node风格的回调函数相同，这种方式产生的错误也不能被`try { } catch(err) { }`捕获 -- 它们发生时，外围的代码已经退出了。
