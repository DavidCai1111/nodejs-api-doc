# process#

`process`对象是一个全局对象，并且何以被在任何地方调用。这是一个`EventEmitter`实例。

### Exit Codes#
当没有任何异步操作在等待时，`io.js`通常将会以一个0为退出码退出。以下这些状态码会在其他情况下被用到：

 - 1 未捕获的致命异常。这是一个未捕获的异常，并且它没有被`domain`处理，也没有被`uncaughtException`处理。
 - 2 未使用（由`Bash`为内建误操作保留）。
 - 3 内部的`JavaScript`解析错误。`io.js`内部的`JavaScript`源码引导（bootstrapping）造成的一个解释错误。这极其罕见。并且常常只会发生在`io.js`自身的开发过程中。
 - 4 内部的`JavaScript`求值错误。`io.js`内部的`JavaScript`源码引导（bootstrapping）未能在求值时返回一个函数值。这极其罕见。并且常常只会发生在`io.js`自身的开发过程中。
 - 5 致命错误。这是V8中严重的不可恢复的错误。典型情况下，一个带有`FATAL ERROR`前缀的信息会被打印在`stderr`。
 - 6 内部异常处理函数丧失功能。这是一个未捕获异常，但是内部的致命异常处理函数被设置为丧失功能，并且不能被调用。
 - 7 内部异常处理函数运行时失败。这是一个未捕获异常，并且内部致命异常处理函数试图处理它时，自身抛出了一个错误。例如它可能在当`process.on('uncaughtException')`或`domain.on('error')`处理函数抛出错误时发生。
 - 8 未使用。`io.js`的之前版本中，退出码`8`通常表示一个未捕获异常。
 - 9 无效参数。当一个位置的选项被指定，或者一个必选的值没有被提供。
 - 10 内部的`JavaScript`运行时错误。`io.js`内部的`JavaScript`源码引导（bootstrapping）函数被调用时抛出一个错误。这极其罕见。并且常常只会发生在`io.js`自身的开发过程中。
 - 12 无效的调试参数。`--debug`和/或`--debug-brk`选项被设置，当时选择了一个无效的端口。
 - 大于128 信号退出。如果`io.js`收到了一个如`SIGKILL`或`SIGHUP`的致命信号，那么它将以一个`128`加上 信号码的值 的退出码退出。这是一个标准的Unix实践，因为退出码由一个7位整数定义，并且信号的退出设置了一个高顺序位（high-order bit），然后包含一个信号码的值。

#### Event: 'exit'#

进程即将退出时触发。在这个时刻已经没有办法可以阻止事件循环的退出，并且一旦所有的`exit`监听器运行结束时，进程将会退出。因此，在这个监听器中你仅仅能调用同步的操作。这是检查模块状态（如单元测试）的好钩子。回调函数有一个退出码参数。

例子：

```js
process.on('exit', function(code) {
  // do *NOT* do this
  setTimeout(function() {
    console.log('This will not run');
  }, 0);
  console.log('About to exit with code:', code);
});
```

#### Event: 'beforeExit'#

这个事件在`io.js`清空了它的事件循环并且没有任何已安排的任务时触发。通常`io.js`当没有更多被安排的任务时就会退出，但是`beforeExit`中可以执行异步调用，让`io.js`继续运行。

`beforeExit`在程序被显示终止时不会触发，如`process.exit()`或未捕获的异常。除非想去安排更多的任务，否则它不应被用来做为`exit`事件的替代。

#### Event: 'uncaughtException'#

当一个异常冒泡回事件循环时就会触发。如果这个时间被添加了监听器，那么默认行为（退出程序且打印堆栈跟踪信息）将不会发生。

例子：

```js
process.on('uncaughtException', function(err) {
  console.log('Caught exception: ' + err);
});

setTimeout(function() {
  console.log('This will still run.');
}, 500);

// Intentionally cause an exception, but don't catch it.
nonexistentFunc();
console.log('This will not run.');
```

注意，`uncaughtException`来处理异常是非常粗糙的。

请不要使用它，使用`domain`来替代。如果你已经使用了它，请在不处理这个异常之后重启你的应用。

请不要像`io.js`的`Error Resume Next`这样使用。一个未捕获异常意味着你的应用或拓展有未定义的状态。盲目地恢复意味着任何事都可能发生。

想象你在升级你的系统时电源被拉断了。10次中前9次都没有问题，但是第10次时，你的系统崩溃了。

你已经被警告。

#### Event: 'unhandledRejection'#

在一个事件循环中，当一个`promise`被“拒绝”并且没有附属的错误处理函数时触发。当一个带有`promise`异常的程序被封装为被“拒绝”的`promise`时，这样的程序的错误可以被`promise.catch(...)`捕获处理并且“拒绝”会通过`promise`链冒泡。这个事件对于侦测和保持追踪那些“拒绝”没有被处理的`promise`非常有用。这个事件会带着以下参数触发：

 - reason `promise`的“拒绝”对象（通常是一个错误实例）
 - p 被“拒绝”的`promise`

下面是一个把所有未处理的“拒绝”打印到控制台的例子：

```js
process.on('unhandledRejection', function(reason, p) {
    console.log("Unhandled Rejection at: Promise ", p, " reason: ", reason);
    // application specific logging, throwing an error, or other logic here
});
```

下面是一个会触发`unhandledRejection`事件的“拒绝”：

```js
somePromise.then(function(res) {
  return reportToUser(JSON.pasre(res)); // note the typo
}); // no `.catch` or `.then`
```

#### Event: 'rejectionHandled'#

当一个`Promise`被“拒绝”并且一个错误处理函数被附给了它（如`.catch()`）时的下一个事件循环之后触发。这个事件会带着以下参数触发：

- p 一个在之前会被触发在`unhandledRejection`事件中，但现在被处理函数捕获的`promise`

一个`promise`链的顶端没有 “拒绝”可以总是被处理 的概念。由于其异步的本质，一个`promise`的“拒绝”可以在未来的某一个时间点被处理，可以是在事件循环中被触发`unhandledRejection`事件之后。

另外，不像同步代码中是一个永远增长的 未捕获异常 列表，`promise`中它是一个可伸缩的 未捕获`拒绝` 列表。在同步代码中，`uncaughtException`事件告诉你 未捕获异常 列表增长了。但是在`promise`中，`unhandledRejection`事件告诉你 未捕获“拒绝” 列表增长了，`rejectionHandled`事件告诉你 未捕获“拒绝” 列表缩短了。

使用“拒绝”侦测钩子来保持一个被“拒绝”的`promise`列表：

```js
var unhandledRejections = [];
process.on('unhandledRejection', function(reason, p) {
    unhandledRejections.push(p);
});
process.on('rejectionHandled', function(p) {
    var index = unhandledRejections.indexOf(p);
    unhandledRejections.splice(index, 1);
});
```

#### Signal Events#

当一个进程收到一个信号时触发。参阅`sigaction(2)`。

监听`SIGINT`信号的例子：

```js
// Start reading from stdin so we don't exit.
process.stdin.resume();

process.on('SIGINT', function() {
  console.log('Got SIGINT.  Press Control-D to exit.');
});
```

一个发送`SIGINT`信号的快捷方法是在大多数终端中按下 Control-C 。

注意：

 - SIGUSR1 是`io.js`用于开启调试的保留信号。可以为其添加一个监听器，但不能阻止调试的开始。
 - SIGTERM 和 SIGINT在非Windows平台下有在以 128 + 信号 退出码退出前重置终端模式的默认监听器。如果另有监听器被添加，默认监听器会被移除（即`io.js`将会不再退出）。
 - SIGPIPE 默认被忽略，可以被添加监听器。
 - SIGHUP 当控制台被关闭时会在Windows中产生，或者其他平台有其他相似情况时（参阅`signal(7)`）。它可以被添加监听器，但是Windows中`io.js`会无条件的在10秒后关闭终端。在其他非Windows平台，它的默认行为是结束`io.js`，但是一旦被添加了监听器，默认行为会被移除。
 - SIGTERM 在Windows中不被支持，它可以被监听。
 - SIGINT 支持所有的平台。可以由 CTRL+C 产生（尽管它可能是可配置的）。当启用终端的`raw mode`时，它不会产生。
 - SIGBREAK 在Windows中，按下 CTRL+BREAK 时它会产生。在非Windows平台下，它可以被监听，但它没有产生的途径。
 - SIGWINCH 当终端被改变大小时产生。Windows下，它只会在当光标被移动时写入控制台或可读tty使用`raw mode`时发生。
 - SIGKILL 可以被添加监听器。它会无条件得在所有平台下关闭`io.js`。
 - SIGSTOP 可以被添加监听器。
 
注意Windows不支持发送信号，但`io.js`通过`process.kill()`和`child_process.kill()`提供了模拟：- 发送信号`0`被用来检查进程的存在 - 发送SIGINT， SIGTERM 和 SIGKILL 会导致目标进程的无条件退出。

#### process.stdout#

一个指向`stdout`的可写流。

例如，`console.log`可能与这个相似：

```js
console.log = function(msg) {
  process.stdout.write(msg + '\n');
};
```

在`io.js`中，`process.stderr`和`process.stdout`与其他流不同，因为他们不能被关闭（调用`end()`会报错）。它们永远不触发`finish`事件并且写操作通常是阻塞的。

 - 当指向普通文件或TTY文件描述符时，它们是阻塞的。

 - __以下情况下他们指向流__
  - 他们在Linux/Unix中阻塞
  - 他们在Windows中的其他流里不阻塞

若要检查`io.js`是否在一个TTY上下文中运行，读取`process.stderr`，`process.stdout`或`process.stdin`的`isTTY`属性：

```SHELL
$ iojs -p "Boolean(process.stdin.isTTY)"
true
$ echo "foo" | iojs -p "Boolean(process.stdin.isTTY)"
false

$ iojs -p "Boolean(process.stdout.isTTY)"
true
$ iojs -p "Boolean(process.stdout.isTTY)" | cat
false
```

更多信息请参阅tty文档。

#### process.stderr#

一个指向`stderr`的可写流。

在`io.js`中，`process.stderr`和`process.stdout`与其他流不同，因为他们不能被关闭（调用`end()`会报错）。它们永远不触发`finish`事件并且写操作通常是阻塞的。

 - 当指向普通文件或TTY文件描述符时，它们是阻塞的。

 - __以下情况下他们指向流__
  - 他们在Linux/Unix中阻塞
  - 他们在Windows中的其他流里不阻塞
  
#### process.stdin#

一个指向`stdin`的可读流。

一个打开标准输入并且监听两个事件的例子：

```js
process.stdin.setEncoding('utf8');

process.stdin.on('readable', function() {
  var chunk = process.stdin.read();
  if (chunk !== null) {
    process.stdout.write('data: ' + chunk);
  }
});

process.stdin.on('end', function() {
  process.stdout.write('end');
});
```

作为一个流，`process.stdin`可以被切换至“旧”模式，这样就可以兼容`node.js` v0.10 前所写的脚本。更多信息请参阅 流的兼容性 。

在“旧”模式中`stdin`流默认是被暂停的。所以你必须调用`process.stdin.resume()`来读取。注意调用`process.stdin.resume()`这个操作本身也会将流切换至旧模式。

如果你正将开启一个新的工程。你应该要更常使用“新”模式的流。

#### process.argv#

一个包含了命令行参数的数组。第一次元素将会是`'iojs'`，第二个元素将会是`JavaScript`文件名。之后的元素将会是额外的命令行参数。

```js
// print process.argv
process.argv.forEach(function(val, index, array) {
  console.log(index + ': ' + val);
});
```

这将会是：

```SHELL
$ iojs process-2.js one two=three four
0: iojs
1: /Users/mjr/work/iojs/process-2.js
2: one
3: two=three
4: four
```

#### process.execPath#

这将是开启进程的可执行文件的绝对路径名：

例子：

```js
/usr/local/bin/iojs
```

#### process.execArgv#

这是在启动时`io.js`自身参数的集合。这些参数不会出现在`process.argv`中，并且不会包含`io.js`可执行文件，脚本名和其他脚本名之后的参数。这些参数对开启和父进程相同执行环境的子进程非常有用。

例子：

```SHELL
$ iojs --harmony script.js --version
```

`process.execArgv`将会是：

```js
['--harmony']
```

`process.argv`将会是：

```js
['/usr/local/bin/iojs', 'script.js', '--version']
```

#### process.abort()#

这将导致`io.js`触发`abort`事件。这个将导致`io.js`退出，并创建一个核心文件。

#### process.chdir(directory)#

为进程改变当前工作目录，如果失败，则抛出一个异常。

```js
console.log('Starting directory: ' + process.cwd());
try {
  process.chdir('/tmp');
  console.log('New directory: ' + process.cwd());
}
catch (err) {
  console.log('chdir: ' + err);
}
```

#### process.cwd()#

返回进程的当前工作目录。

```js
console.log('Current directory: ' + process.cwd());
```

#### process.env#

包含用户环境变量的对象。参阅`environ(7)`。

一个例子：

```js
{ TERM: 'xterm-256color',
  SHELL: '/usr/local/bin/bash',
  USER: 'maciej',
  PATH: '~/.bin/:/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin',
  PWD: '/Users/maciej',
  EDITOR: 'vim',
  SHLVL: '1',
  HOME: '/Users/maciej',
  LOGNAME: 'maciej',
  _: '/usr/local/bin/iojs' }
  ```
  
你可以改写这个对象，但是改变不会反应在你的进程之外。这以为着以下代码不会正常工作：

```SHELL
$ iojs -e 'process.env.foo = "bar"' && echo $foo
```

但是以下代码会：

```js
process.env.foo = 'bar';
console.log(process.env.foo);
```

#### process.exit([code])#

使用指定的退出码退出程序，如果忽略退出码。那么将使用“成功”退出码`0`。

以一个“失败”退出码结束：

```js
process.exit(1);
```

在执行`io.js`的shell中可以看到为`1`的退出码。

#### process.exitCode#

将是程序退出码的数字，当程序优雅退出 或 被`process.exit()`关闭且没有指定退出码时。

为`process.exit(code)`指定一个退出码会覆盖之前的`process.exitCode`设置。

#### process.getgid()#

注意：这个函数只在POSIX平台上有效（如在Windows，Android中无效）。

获取进程的群组标识（参阅`getgid(2)`）。这是一个群组id数组，不是群组名。

```js
if (process.getgid) {
  console.log('Current gid: ' + process.getgid());
}
```

#### process.getegid()#

注意：这个函数只在POSIX平台上有效（如在Windows，Android中无效）。

获取进程的有效群组标识（参阅`getgid(2)`）。这是一个群组id数组，不是群组名。

```js
if (process.getegid) {
  console.log('Current gid: ' + process.getegid());
}
```

#### process.setgid(id)#

注意：这个函数只在POSIX平台上有效（如在Windows，Android中无效）。

设置进程的群组标识（参阅`setgid(2)`）。它接受一个数字ID或一个群组名字符串。如果群组名被指定，那么这个方法将在解析群组名为一个ID的过程中阻塞。

```js
if (process.getgid && process.setgid) {
  console.log('Current gid: ' + process.getgid());
  try {
    process.setgid(501);
    console.log('New gid: ' + process.getgid());
  }
  catch (err) {
    console.log('Failed to set gid: ' + err);
  }
}
```

#### process.setegid(id)#

注意：这个函数只在POSIX平台上有效（如在Windows，Android中无效）。

设置进程的有效群组标识（参阅`setgid(2)`）。它接受一个数字ID或一个群组名字符串。如果群组名被指定，那么这个方法将在解析群组名为一个ID的过程中阻塞。

```js
if (process.getegid && process.setegid) {
  console.log('Current gid: ' + process.getegid());
  try {
    process.setegid(501);
    console.log('New gid: ' + process.getegid());
  }
  catch (err) {
    console.log('Failed to set gid: ' + err);
  }
}
```

#### process.getuid()#

注意：这个函数只在POSIX平台上有效（如在Windows，Android中无效）。

获取进程的用户id（参阅`getuid(2)`）。这是一个数字用户id，不是用户名。

```js
if (process.getuid) {
  console.log('Current uid: ' + process.getuid());
}
```

#### process.geteuid()#

注意：这个函数只在POSIX平台上有效（如在Windows，Android中无效）。

获取进程的有效用户id（参阅`getuid(2)`）。这是一个数字用户id，不是用户名。

```js
if (process.geteuid) {
  console.log('Current uid: ' + process.geteuid());
}
```

#### process.setuid(id)#

注意：这个函数只在POSIX平台上有效（如在Windows，Android中无效）。

设置进程的用户ID（参阅`setuid(2)`）。它接受一个数字ID或一个用户名字符串。如果用户名被指定，那么这个方法将在解析用户名为一个ID的过程中阻塞。

```js
if (process.getuid && process.setuid) {
  console.log('Current uid: ' + process.getuid());
  try {
    process.setuid(501);
    console.log('New uid: ' + process.getuid());
  }
  catch (err) {
    console.log('Failed to set uid: ' + err);
  }
}
```

#### process.seteuid(id)#

注意：这个函数只在POSIX平台上有效（如在Windows，Android中无效）。

设置进程的有效用户ID（参阅`seteuid(2)`）。它接受一个数字ID或一个用户名字符串。如果用户名被指定，那么这个方法将在解析用户名为一个ID的过程中阻塞。

```js
if (process.geteuid && process.seteuid) {
  console.log('Current uid: ' + process.geteuid());
  try {
    process.seteuid(501);
    console.log('New uid: ' + process.geteuid());
  }
  catch (err) {
    console.log('Failed to set uid: ' + err);
  }
}
```

#### process.getgroups()#

注意：这个函数只在POSIX平台上有效（如在Windows，Android中无效）。

返回一个补充群组ID的数组。如果包含了有效的组ID，POSIX将不会指定。但`io.js`保证它始终是。

#### process.setgroups(groups)#

注意：这个函数只在POSIX平台上有效（如在Windows，Android中无效）。

设置一个补充群组ID。这是一个特殊的操作，意味着你需要拥有`root`或`CAP_SETGID`权限才可以这么做。

列表可以包含群组ID，群组名，或两者。

#### process.initgroups(user, extra_group)#

注意：这个函数只在POSIX平台上有效（如在Windows，Android中无效）。

读取`/etc/group`并且初始化群组访问列表，使用用户是组员的所有群组。这是一个特殊的操作，意味着你需要拥有`root`或`CAP_SETGID`权限才可以这么做。

`user`是一个用户名或一个用户ID。`extra_group`是一个群组名或群组ID。

当你注销权限时有些需要关心的：

```js
console.log(process.getgroups());         // [ 0 ]
process.initgroups('bnoordhuis', 1000);   // switch user
console.log(process.getgroups());         // [ 27, 30, 46, 1000, 0 ]
process.setgid(1000);                     // drop root gid
console.log(process.getgroups());         // [ 27, 30, 46, 1000 ]
```

#### process.version#

一个暴露`NODE_VERSION`的编译时存储属性。

```js
console.log('Version: ' + process.version);
```

#### process.versions#

一个暴露io.js版本和它的依赖的字符串属性。

```js
console.log(process.versions);
```

将可能打印：

```js
{ http_parser: '2.3.0',
  node: '1.1.1',
  v8: '4.1.0.14',
  uv: '1.3.0',
  zlib: '1.2.8',
  ares: '1.10.0-DEV',
  modules: '43',
  openssl: '1.0.1k' }
```

#### process.config#

一个表示用于编译当前`io.js`执行文件的配置的JavaScript对象。这和运行`./configure`脚本产生的`config.gypi`一样。

一个可能的输出：

```js
{ target_defaults:
   { cflags: [],
     default_configuration: 'Release',
     defines: [],
     include_dirs: [],
     libraries: [] },
  variables:
   { host_arch: 'x64',
     node_install_npm: 'true',
     node_prefix: '',
     node_shared_cares: 'false',
     node_shared_http_parser: 'false',
     node_shared_libuv: 'false',
     node_shared_zlib: 'false',
     node_use_dtrace: 'false',
     node_use_openssl: 'true',
     node_shared_openssl: 'false',
     strict_aliasing: 'true',
     target_arch: 'x64',
     v8_use_snapshot: 'true' } }
```

#### process.kill(pid[, signal])#

给进程传递一个信号。`pid`是进程id，`signal`是描述信号的字符串。信号码类似于`'SIGINT'`或`'SIGHUP'`。如果忽略，那么信号将是`'SIGTERM'`。更多信息参阅`Signal Events`和`kill(2)`。

如果目标不存在将会抛出一个错误，并且在一些情况下，`0`信号可以被用来测试进程的存在。

注意，这个函数仅仅是名字为`process.kill`，它只是一个信号发送者。发送的信号可能与杀死进程无关。

一个发送信号给自身的例子：

```js
process.on('SIGHUP', function() {
  console.log('Got SIGHUP signal.');
});

setTimeout(function() {
  console.log('Exiting.');
  process.exit(0);
}, 100);
```

process.kill(process.pid, 'SIGHUP');

注意：当`SIGUSR1 `被`io.js`收到，它会开始调试。参阅`Signal Events`。

process.pid#

进程的PID。

```js
console.log('This process is pid ' + process.pid);
```

process.title#

设置/获取 `'ps'` 中显示的进程名。

当设置该属性时，所能设置的字符串最大长度视具体平台而定，如果超过的话会自动截断。

在 Linux 和 OS X 上，它受限于名称的字节长度加上命令行参数的长度，因为它有覆盖参数内存。

v0.8 版本允许更长的进程标题字符串，也支持覆盖环境内存，但是存在潜在的不安全和混乱。

#### process.arch#

返回当前的处理器结构：`'arm'`，`'ia32'`或`'x64'`。

```js
console.log('This processor architecture is ' + process.arch);
```

### process.platform#

放回当前的平台：`'darwin'`，`'freebsd'`，`'linux'`，`'sunos'`或`'win32'`。

```js
console.log('This platform is ' + process.platform);
```

#### process.memoryUsage()#

返回当前`io.js`进程内存使用情况（用字节描述）的对象。

```js
var util = require('util');

console.log(util.inspect(process.memoryUsage()));
```

可能的输出：

```js
{ rss: 4935680,
  heapTotal: 1826816,
  heapUsed: 650472 }
```

`heapTotal`和`heapUsed`指向V8的内存使用。

#### process.nextTick(callback[, arg][, ...])#
 - callback Function

在事件循环的下一次循环中调用回调函数。

这不是`setTimeout(fn, 0)`的简单别名，它更有效率。在之后的`tick`中，它在任何其他的I/O事件（包括`timer`）触发之前运行。

```js
console.log('start');
process.nextTick(function() {
  console.log('nextTick callback');
});
console.log('scheduled');
// Output:
// start
// scheduled
// nextTick callback
```

这对于开发你想要给予用户在对象被构建后，任何I/O发生前，去设置事件监听器的机会时，非常有用。

```js
function MyThing(options) {
  this.setupOptions(options);

  process.nextTick(function() {
    this.startDoingStuff();
  }.bind(this));
}

var thing = new MyThing();
thing.getReadyForStuff();
// thing.startDoingStuff() gets called now, not before.
```

这对于100%同步或100%异步的API非常重要。考虑一下例子：

```js
// WARNING!  DO NOT USE!  BAD UNSAFE HAZARD!
function maybeSync(arg, cb) {
  if (arg) {
    cb();
    return;
  }

  fs.stat('file', cb);
}
```

这个API是危险的，如果你这样做：

```js
maybeSync(true, function() {
  foo();
});
bar();
```

`foo()`和`bar()`的调用次序是不确定的。

更好的做法是：

```js
function definitelyAsync(arg, cb) {
  if (arg) {
    process.nextTick(cb);
    return;
  }

  fs.stat('file', cb);
}
```

注意：`nextTick`队列在每一次事件循环的I/O开始前都要完全执行完毕。所以，递归地设置`nextTick`回调会阻塞I/O的方法，就像一个`while(true);`循环。

#### process.umask([mask])#

设置或读取进程的文件模式的创建掩码。子进程从父进程中继承这个掩码。返回旧的掩码如果`mask`参数被指定。否则，会返回当前掩码。

```js
var oldmask, newmask = 0022;

oldmask = process.umask(newmask);
console.log('Changed umask from: ' + oldmask.toString(8) +
            ' to ' + newmask.toString(8));
```

#### process.uptime()#

`io.js`进程已执行的秒数。

#### process.hrtime()#

以`[seconds, nanoseconds]`元组数组的形式返回高分辨时间。是相对于过去的任意时间。它与日期无关所以不用考虑时区等因素。它的主要用途是衡量程序性能。

你可以将之前的`process.hrtime()`返回传递给一个新的`process.hrtime()`来获得一个比较。衡量性能时非常有用：

```js
var time = process.hrtime();
// [ 1800216, 25 ]

setTimeout(function() {
  var diff = process.hrtime(time);
  // [ 1, 552 ]

  console.log('benchmark took %d nanoseconds', diff[0] * 1e9 + diff[1]);
  // benchmark took 1000000527 nanoseconds
}, 1000);
```

#### process.mainModule#

检索`require.main`的备用方式。区别是，如果主模块在运行时改变，`require.main`可能仍指向改变发生前的被引入的原主模块。通常，假设它们一样是安全的。

与`require.main`一样，当如果没有入口脚本时，它将是`undefined`。