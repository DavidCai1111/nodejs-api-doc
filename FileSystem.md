# File System#

### 稳定度: 2 - 稳定

文件I/O是由标准POSIX函数的简单包装提供的。通过`require('fs')`来使用这个模块。所有的方法都有异步和同步两种形式。

异步形式的方法通常在最后一个参数上接受一个回调函数。回调函数的参数则取决于不同的方法，但是第一个参数总是为异常所保留。如果操作正常结束，那么第一个参数会是`null`或`undefined`。

当同步形式的方法产生异常时，会立刻抛出。你可以使用`try/catch`捕获，或让它们冒泡。

下面是一个异步方法的例子：

```js
var fs = require('fs');

fs.unlink('/tmp/hello', function (err) {
  if (err) throw err;
  console.log('successfully deleted /tmp/hello');
});
```

下面是一个同步方法的例子：

```js
var fs = require('fs');

fs.unlinkSync('/tmp/hello');
console.log('successfully deleted /tmp/hello');
```

因为异步方法不能够保证执行顺序，所以下面的例子很容易出错：

```js
fs.rename('/tmp/hello', '/tmp/world', function (err) {
  if (err) throw err;
  console.log('renamed complete');
});
fs.stat('/tmp/world', function (err, stats) {
  if (err) throw err;
  console.log('stats: ' + JSON.stringify(stats));
});
```

它需要在`fs.rename`后执行`fs.stat`。正确的执行方法应如下：

```js
fs.rename('/tmp/hello', '/tmp/world', function (err) {
  if (err) throw err;
  fs.stat('/tmp/world', function (err, stats) {
    if (err) throw err;
    console.log('stats: ' + JSON.stringify(stats));
  });
});
```

在繁忙的进程中，十分推荐使用异步版本的方法。同步版本的方法会阻塞进程，直到它们完成，也就是说它们会暂停所有连接。

文件的相对路径也可以被使用，记住路径是相对于`process.cwd()`的。

大多数的`fs`函数允许你省略回调函数。如果你省略了，将会由一个默认的回调函数来重抛出（rethrows）错误。要获得原始调用地点的堆栈追踪信息，请设置`NODE_DEBUG`环境变量：

```js
$ cat script.js
function bad() {
  require('fs').readFile('/');
}
bad();

$ env NODE_DEBUG=fs iojs script.js
fs.js:66
        throw err;
              ^
Error: EISDIR, read
    at rethrow (fs.js:61:21)
    at maybeCallback (fs.js:79:42)
    at Object.fs.readFile (fs.js:153:18)
    at bad (/path/to/script.js:2:17)
    at Object.<anonymous> (/path/to/script.js:5:1)
    <etc.>
```

#### fs.rename(oldPath, newPath, callback)#

异步版本的`rename(2)`。回调函数只有一个可能的异常参数。

#### fs.renameSync(oldPath, newPath)#

同步版本的`rename(2)`。返回`undefined`。

#### fs.ftruncate(fd, len, callback)#

异步版本的`ftruncate(2)`。回调函数只有一个可能的异常参数。

#### fs.ftruncateSync(fd, len)#

同步版本的`ftruncate(2)`。返回`undefined`。

#### fs.truncate(path, len, callback)#

异步版本的`truncate(2)`。回调函数只有一个可能的异常参数。第一个参数也可以接受一个文件描述符，这样的话，`fs.ftruncate()`会被调用。

#### fs.truncateSync(path, len)#

同步版本的`truncate(2)`。返回`undefined`。

#### fs.chown(path, uid, gid, callback)#

异步版本的`chown(2)`。回调函数只有一个可能的异常参数。

#### fs.chownSync(path, uid, gid)#

同步版本的`chown(2)`。返回`undefined`。

#### fs.fchown(fd, uid, gid, callback)#

异步版本的`fchown(2)`。回调函数只有一个可能的异常参数。

#### fs.fchownSync(fd, uid, gid)#

同步版本的`fchown(2)`。返回`undefined`。

#### fs.lchown(path, uid, gid, callback)#

异步版本的`lchown(2)`。回调函数只有一个可能的异常参数。

#### fs.lchownSync(path, uid, gid)#

同步版本的`lchown(2)`。返回`undefined`。

#### fs.chmod(path, mode, callback)#

异步版本的`chmod(2)`。回调函数只有一个可能的异常参数。

#### fs.chmodSync(path, mode)#

同步版本的`chmod(2)`。返回`undefined`。

#### fs.fchmod(fd, mode, callback)#

异步版本的`fchmod(2)`。回调函数只有一个可能的异常参数。

#### fs.fchmodSync(fd, mode)#

同步版本的`fchmod(2)`。返回`undefined`。

#### fs.lchmod(path, mode, callback)#

异步版本的`lchmod(2)`。回调函数只有一个可能的异常参数。

仅在Mac OS X中可用。

#### fs.lchmodSync(path, mode)#

同步版本的`lchmod(2)`。返回`undefined`。

#### fs.stat(path, callback)#

异步版本的`stat(2)`。回调函数有两个参数（err, stats），`stats`是一个`fs.Stats`对象。更多信息请参阅`fs.Stats`章节。

#### fs.lstat(path, callback)#

异步版本的`lstat(2)`。回调函数有两个参数（err, stats），`stats`是一个`fs.Stats`对象。`lstat()`与`stat()`是相同的，除了`path`是一个符号链接，连接自己本身就是`stat-ed`，而不是引用一个文件。

#### fs.fstat(fd, callback)#

异步版本的`fstat(2)`。回调函数有两个参数（err, stats），`stats`是一个`fs.Stats`对象。`fstat()`与`stat()`是相同的，除了将要被`stat-ed`的文件是通过文件描述符`fd`来指定的。

#### fs.statSync(path)#

同步版本的`stat(2)`。返回一个`fs.Stats`实例。

#### fs.lstatSync(path)#

同步版本的`lstat(2)`。返回一个`fs.Stats`实例。

#### fs.fstatSync(fd)#

同步版本的`fstat(2)`。返回一个`fs.Stats`实例。

#### fs.link(srcpath, dstpath, callback)#

异步版本的`link(2)`。回调函数只有一个可能的异常参数。

#### fs.linkSync(srcpath, dstpath)#

同步版本的`link(2)`。返回`undefined`。

#### fs.symlink(destination, path[, type], callback)#

异步版本的`symlink(2)`。回调函数只有一个可能的异常参数。`type`参数可以被设置为`'dir'`，`'file'`或`'junction'`（默认为`'file'`），并且仅在Windows平台下可用（其他平台下会被忽略）。注意Windows `junction`点 要求目标路径必须是绝对的。当使用`'junction'`时，`destination`参数会被自动转换为绝对路径。

#### fs.symlinkSync(destination, path[, type])#

同步版本的`symlink(2)`。返回`undefined`。

#### fs.readlink(path, callback)#

异步版本的`link(2)`。回调函数有两个参数（err, linkString）。

#### fs.readlinkSync(path)#

异步版本的`readlink(2)`，返回一个符号链接字符串值。

#### fs.realpath(path[, cache], callback)#

异步版本的`realpath(2)`。回调函数有两个参数（err, resolvedPath）。可能会使用`process.cwd`来解析相对路径。`cache`是一个包含了路径映射的对象，被用来 强制进行指定的路径解析 或 避免对真实路径调用额外的`fs.stat`。

例子：

```js
var cache = {'/etc':'/private/etc'};
fs.realpath('/etc/passwd', cache, function (err, resolvedPath) {
  if (err) throw err;
  console.log(resolvedPath);
});
```

#### fs.realpathSync(path[, cache])#

同步版本的`realpath(2)`，返回一个解析出的路径。

#### fs.unlink(path, callback)#

异步版本的`unlink(2)`。回调函数只有一个可能的异常参数。

#### fs.unlinkSync(path)#

同步版本的`unlink(2)`。返回`undefined`。

#### fs.rmdir(path, callback)#

异步版本的`rmdir(2)`。回调函数只有一个可能的异常参数。

#### fs.rmdirSync(path)#

同步版本的`rmdir(2)`。返回`undefined`。

#### fs.mkdir(path[, mode], callback)#

异步版本的`mkdir(2)`。回调函数只有一个可能的异常参数。`mode`默认为`0o777`。

#### fs.mkdirSync(path[, mode])#

同步版本的`mkdir(2)`。返回`undefined`。

#### fs.readdir(path, callback)#

异步版本的`readdir(3)`。读取目录内容。回调函数有两个参数（err, files），`files`是一个目录中的文件名数组（不包括`'.'`和`'..'`）。

#### fs.readdirSync(path)#

同步版本的`readdir(3)`。返回一个文件名数组（不包括`'.'`和`'..'`）。

#### fs.close(fd, callback)#

异步版本的`close(2)`。回调函数只有一个可能的异常参数。

#### fs.closeSync(fd)#

同步版本的`close(2)`。返回`undefined`。

#### fs.open(path, flags[, mode], callback)#

异步版本的文件打开。参阅`open(2)`。`flag`可以是：

 - 'r' - 以只读的方式打开文件。如果文件不存在则抛出异常。

 - 'r+' - 以读写的方式打开文件。如果文件不存在则抛出异常。

 - 'rs' - 同步地以只读的方式打开文件。绕过操作系统的本地文件系统缓存。

该功能主要用于打开NFS挂载的文件，因为它允许你跳过潜在的过时的本地缓存。它对I/O性能有非常大的影响，所以除非需要它，否则不应使用这个`flag`。

注意这个`flag`不会将`fs.open()`变为一个同步调用。因为如果你想要同步调用，你应使用`fs.openSync()`。

 - 'rs+' - 以读写的方式打开文件，告诉操作系统同步地打开它。注意事项请参阅`'rs'`。

 - 'w' - 以只写的方式打开文件。如果文件不存在，将会创建它。如果已存在，将会覆盖它。

 - 'wx' - 类似于`'w'`，但是路径不存在时会失败。

 - 'w+' - 以读写的方式打开文件。如果文件不存在，将会创建它。如果已存在，将会覆盖它。

 - 'wx+' - 类似于`'w+'`，但是路径不存在时会失败。

 - 'a' - 以附加的形式打开文件。如果文件不存在，将会创建它。

 - 'ax' - 类似于`'a'`，但是路径不存在时会失败。

 - 'a+' - 以读取和附加的形式打开文件。如果文件不存在，将会创建它。

 - 'ax+' - 类似于`'a+'`，但是路径不存在时会失败。
 
参数`mode`用于设置文件模式（权限和`sticky bits`），但是前提是文件已被创建。它默认为`0666 `，有可读和可写权限。

回调函数有两个参数（err, fd）。

排除标识`'x'`（`open(2)`中的`O_EXCL`标识）保证了目录是被新创建的。在POSIX系统上，即使路径指向了一个不存在的符号链接，也会被认定为文件存在。排除标识不能保证在网络文件系统中有效。

在Linux下，无法对以追加形式打开的文件，在指定位置写入数据。内核忽略了位置参数并且总是将数据追加到文件的末尾。

#### fs.openSync(path, flags[, mode])#

同步版本的`fs.open()`，返回代表文件描述符的一个整数。

#### fs.utimes(path, atime, mtime, callback)#

更改`path`所指向的文件的时间戳。

#### fs.utimesSync(path, atime, mtime)#

同步版本的`fs.utimes()`。返回`undefined`。

#### fs.futimes(fd, atime, mtime, callback)#

更改文件描述符`fd`所指向的文件的时间戳。

#### fs.futimesSync(fd, atime, mtime)#

同步版本的`fs.futimes()`。返回`undefined`。

#### fs.fsync(fd, callback)#

异步版本的`fsync(2)`。回调函数只有一个可能的异常参数。

#### fs.fsyncSync(fd)#

同步版本的`fsync(2)`。返回`undefined`。

#### fs.write(fd, buffer, offset, length[, position], callback)#

向文件描述符`fd`指向的文件写入`buffer`。

`offset`和`length`决定了`buffer`的哪一部分被写入文件。

`position`指定了文件中，数据被写入的开始位置的偏移量。如果`typeof position !== 'number'`，那么数据将会在当前位置被写入。参阅`pwrite(2)`。

回调函数有三个参数（err, written, buffer）。`written`指出了`buffer`中有多少字节被写入。

注意，不等待回调函数而多次执行`fs.write`是不安全的。这种情况下推荐使用`fs.createWriteStream`。

在Linux下，无法对以追加形式打开的文件，在指定位置写入数据。内核忽略了位置参数并且总是将数据追加到文件的末尾。

#### fs.write(fd, data[, position[, encoding]], callback)#

向文件描述符`fd`指向的文件写入`data`。如果`data`不是一个`Buffer`实例，那么其值将被强制转化为一个字符串。

`position`指定了文件中，数据被写入的开始位置的偏移量。如果`typeof position !== 'number'`，那么数据将会在当前位置被写入。参阅`pwrite(2)`。

`encoding`是期望的字符串编码。

回调函数有三个参数（err, written, buffer）。`written`指出了`buffer`中有多少字节被写入。注意，写入的字节与字符串字符是不同的。参阅`Buffer.byteLength`。

与写入`buffer`不同，整个字符串都必须被写入。不能指定子字符串。因为字节的偏移量可能与字符串的偏移量不相同。

注意，不等待回调函数而多次执行`fs.write`是不安全的。这种情况下推荐使用`fs.createWriteStream`。

在Linux下，无法对以追加形式打开的文件，在指定位置写入数据。内核忽略了位置参数并且总是将数据追加到文件的末尾。

#### fs.writeSync(fd, buffer, offset, length[, position])#
#### fs.writeSync(fd, data[, position[, encoding]])#

同步版本的`fs.write()`。返回被写入的字节数。

#### fs.read(fd, buffer, offset, length, position, callback)#

从文件描述符`fd`指向的文件读取数据。

`buffer`是数据将要被写入的缓冲区。

`offset`是开始向`buffer`写入数据的缓冲区偏移量。

`length`是一个指定了读取字节数的整数。

`position`是一个指定了从文件的何处开始读取数据的整数。如果`position`是`null`，数据将会从当前位置开始读取。

回调函数有三个参数（err, bytesRead, buffer）。

#### fs.readSync(fd, buffer, offset, length, position)#

同步版本的`fs.read`。返回读取字节的个数。

#### fs.readFile(filename[, options], callback)#
 - filename String
 - __options Object | String__
  - encoding String | Null 默认为`null`
  - flag String 默认为`'r'`
 - callback Function

异步得读取文件的所有内容。例子：

```js
fs.readFile('/etc/passwd', function (err, data) {
  if (err) throw err;
  console.log(data);
});
```

回调函数有两个参数（err, data），`data`是文件的内容。

如果没有指定编码，那么将会返回源`buffer`。

如果`options`是一个字符串，那么它将指定编码，例子：

```js
fs.readFile('/etc/passwd', 'utf8', callback);
```

#### fs.readFileSync(filename[, options])#

同步版本的`fs.readFile`。返回文件的内容。

如果指定了编码那么将会返回字符串。否则返回`buffer`。

#### fs.writeFile(filename, data[, options], callback)#
 - filename String
 - data String | Buffer
 - __options Object | String__
  - encoding String | Null 默认为`'utf8'`
  - mode Number 默认为`0o666`
  - flag String 默认为`'w'`
 - callback Function

异步地向文件写入数据，如果文件已经存在，那么会覆盖它。`data`可以是一个字符串或一个`buffer`。

如果数据时一个`buffer`那么编码会被忽略。编码默认为`'utf8'`。

例子：

```js
fs.writeFile('message.txt', 'Hello io.js', function (err) {
  if (err) throw err;
  console.log('It\'s saved!');
});
```

如果`options`是一个字符串，那么它将指定编码，例子：

```js
fs.writeFile('message.txt', 'Hello io.js', 'utf8', callback);
```

#### fs.writeFileSync(filename, data[, options])#

同步版本的`fs.writeFile`。返回`undefined`。

#### fs.appendFile(filename, data[, options], callback)#

 - filename String
 - data String | Buffer
 - __options Object | String__
  - encoding String | Null 默认为`'utf8'`
  - mode Number 默认为`0o666`
  - flag String 默认为`'a'`
 - callback Function

异步地向文件追加数据，如果文件不存在将会创建它。`data`可以是一个字符串或一个`buffer`。

例子：

```js
fs.appendFile('message.txt', 'data to append', function (err) {
  if (err) throw err;
  console.log('The "data to append" was appended to file!');
});
```

如果`options`是一个字符串，那么它将指定编码，例子：

```js
fs.appendFile('message.txt', 'data to append', 'utf8', callback);
```

#### fs.appendFileSync(filename, data[, options])#

同步版本的`fs.appendFile`。返回`undefined`。

#### fs.watchFile(filename[, options], listener)#

监视文件变化。回调函数`listener`会在文件每一次被访问时调用。

第二参数是可选的。如果`options`被提供，那么它必须是一个含有两个成员`persistent`和`interval`的对象。`persistent`表明了进程是否在文件被监视时继续执行。`interval`表明了文件被轮询的间隔（毫秒）。默认是`{ persistent: true, interval: 5007 }`。

`listener`有两个参数，当前状态对象和先前状态对象：

```js
fs.watchFile('message.text', function (curr, prev) {
  console.log('the current mtime is: ' + curr.mtime);
  console.log('the previous mtime was: ' + prev.mtime);
});
```

这两个状态对象都是`fs.Stat`实例。

如果你想要在文件被修改时被通知，而不仅仅是在被访问时，你需要比较`curr.mtime`和`prev.mtime`。

注意：`fs.watch`比`fs.watchFile`和`fs.unwatchFile`更高效。当可能时，请使用`fs.watch`替代它们。

#### fs.unwatchFile(filename[, listener])#

停止监视`filename`的变化。如果指定了`listener`，那么仅仅会移除指定的`listener`。否则所有的监听器都会被移除，并且停止继续监视文件。

对一个没有被监视的文件调用`fs.unwatchFile()`将不会发生任何事，而不是报错。

注意：`fs.watch`比`fs.watchFile`和`fs.unwatchFile`更高效。当可能时，请使用`fs.watch`替代它们。

#### fs.watch(filename[, options][, listener])#

监视`filename`的变化，`filename`指向的可以是文件也可以是目录。返回一个`fs.FSWatcher`对象。

第二个参数是可选的。`options`必须是一个对象。支持的布尔值属性是`persistent`和`recursive`。`persistent`表明了进程是否在文件被监视时继续执行。`recursive`表明了是否子目录也需要被监视，或仅仅监视当前目录。这只在支持的平台（参阅下方`警告`）下传递一个目录时有效。

默认是`{ persistent: true, recursive: false }`。

`listener`回调函数有两个参数（event, filename）。`event`是`'rename'`或`'change'`，`filename`是触发事件的文件名。

##### 警告

`fs.watch` API 不是在所有平台下都表现一致的，并且在一些情况下是不可用的。

`recursive`选项目前只支持OS X。只有`FSEvents`支持这种类型的文件监控，所有其他平台并不会很快都被支持。

##### 可用性

这个特性依赖于底层操作系统提供的文件变化提示。

 - 在Linux系统下，它使用`inotify`。
 - 在BSD系统下，它使用`kqueue`。
 - 在OS X下，对于文件它使用`kqueue`，对于目录它使用`FSEvents`。
 - 在SunOS系统（包括`Solaris`和`SmartOS`）下，它使用事件端口（event ports）。
 - 在Windows系统下，这个特性依赖于`ReadDirectoryChangesW`。

如果由于一些原因，底层功能不可用，那么`fs.watch`的功能也将不可用。例如，在网络文件系统（NFS，SMB等）中监视文件或目录变化，往往结果不可靠或完全不可用。

你仍可以使用`fs.watchFile`，它使用了状态轮询。但是性能更差且可靠性更低。

##### Filename 参数#

回调函数中提供的`filename`参数不是在所有平台上都支持的（目前只支持Linux和Windows）。即使是在支持的平台上，`filename`也不是总会被提供。因此，不要假设`filename`参数总会在回调函数中被提供，需要有一些检测它是否为`null`的逻辑。

```js
fs.watch('somedir', function (event, filename) {
  console.log('event is: ' + event);
  if (filename) {
    console.log('filename provided: ' + filename);
  } else {
    console.log('filename not provided');
  }
});
```
#### fs.exists(path, callback)#

`fs.exists()`已被弃用。请使用`fs.stat`或`fs.access`替代。

检查文件系统来测试提供的路径是否存在。然后在回调函数的参数中提供结果`true`或`false`：

```js
fs.exists('/etc/passwd', function (exists) {
  util.debug(exists ? "it's there" : "no passwd!");
});
```

`fs.exists()`是一个不符合潮流的函数，并且仅因一些历史原因所以仍然错在。在你的代码中，不应有任何原因要继续使用它。

特别的，在打开文件前检查文件是否存在 是一种反模式。因为竞态条件所以让你的代码十分脆弱：其他进程可能`fs.exists()`和`fs.open()`之间删除文件。所以仅仅就去打开一个文件，并且当它不存在时处理错误。

#### fs.existsSync(path)#

同步版本的`fs.exists`。当文件存在，返回`true`，否则返回`false`。

`fs.existsSync()`已被弃用。请使用`fs.statSync`或`fs.accessSync`替代。

#### fs.access(path[, mode], callback)#

对于指定的路径，检测用户的权限。`mode`是一个可选的整数，指定了要被执行的可访问性检查。以下是`mode`的一些可用的常量。可以通过“或”运算符（|）连接两个或以上的值。

 - fs.F_OK - 文件对于当前进程可见。这对于检查文件是否存在很有用，但是不提供任何`rwx`权限信息。这是默认值。
 - fs.R_OK - 文件对于当前进程可读。
 - fs.W_OK - 文件对于当前进程可写。
 - fs.X_OK - 文件对于当前进程可执行。这在Windows上无效（将会表现得像`fs.F_OK`一样）。

最后一个参数`callback`，是一个包含了潜在错误参数的回调函数。如果任何一个可访问检查失败了，错误参数就会被提供。以下是一个在当前进程中检查`/etc/passwd`
可读性和可写性的例子。

```js
fs.access('/etc/passwd', fs.R_OK | fs.W_OK, function(err) {
  util.debug(err ? 'no access!' : 'can read/write');
});
```

#### fs.accessSync(path[, mode])#

同步版本的`fs.access`。如果任何一个可访问性检查失败了，它会抛出异常。否则什么都不做。

#### Class: fs.Stats#

由`fs.stat()`，`fs.lstat()`，`fs.lstat()`和它们的同步版本函数所返回的对象。

 - stats.isFile()
 - stats.isDirectory()
 - stats.isBlockDevice()
 - stats.isCharacterDevice()
 - stats.isSymbolicLink() （仅在调用`fs.lstat()`时有效）
 - stats.isFIFO()
 - stats.isSocket()

对于一个普通的文件，`util.inspect(stats)`可能会返回：

```js
{ dev: 2114,
  ino: 48064969,
  mode: 33188,
  nlink: 1,
  uid: 85,
  gid: 100,
  rdev: 0,
  size: 527,
  blksize: 4096,
  blocks: 8,
  atime: Mon, 10 Oct 2011 23:24:11 GMT,
  mtime: Mon, 10 Oct 2011 23:24:11 GMT,
  ctime: Mon, 10 Oct 2011 23:24:11 GMT,
  birthtime: Mon, 10 Oct 2011 23:24:11 GMT }
```

请注意，`atime`，`mtime`，`birthtime`和`ctime`都是`Date`对象实例，并且你可以通过合适的方法来比较它们的值。普遍的使用方式是，调用`getTime()`来获取unix时间戳并且这个整数可以被用来进行任何比较。但是还有一些可以展示模糊信息的方法。更多的详细信息请参阅`MDN JavaScript Reference`页。

#### Stat 时间值

`stat`对象中的各个时间有如下语义：

 - atime "访问时间" - 文件数据最后一次被访问时的时间。由`mknod(2)`，`utimes(2)`和`read(2)`系统调用改变。
 - mtime "修改时间" - 文件数据最后一次被修改的时间。由`mknod(2)`，`utimes(2)`和`write(2)`系统调用改变。
 - ctime "改变时间" - 文件状态最后一次被改变（索引节点改变）的时间。由`chmod(2)`，`chown(2)`，`link(2)`，`mknod(2)`，`rename(2)`，`unlink(2)`，`utimes(2)`，`read(2)`和`write(2)`系统调用改变。
 - birthtime "创建时间" - 文件的创建时间。在文件被创建时设置。在创建时间不可用的的文件系统上，这个值可能会被`ctime`或是`1970-01-01T00:00Z`（unix时间戳0）填充。在Darwin或其他FreeBSD系统变体上，如果使用`utimes(2)`系统调用设置`atime`为一个比当前`birthtime`更早的时间，`birthtime`也会被这样填充。

在`io.js` v1.0 和 Node v0.12 前，Windows系统中`ctime`持有了`birthtime`值。但是在 v0.12 里，`ctime`不再是“创建时间”。在Unix系统中，它从来都不是。

#### fs.createReadStream(path[, options])#

返回一个新的可读流对象（参阅`Readable Stream`）。

`options`是一个有以下默认值的对象或字符串：

```js
{ flags: 'r',
  encoding: null,
  fd: null,
  mode: 0o666,
  autoClose: true
}
```

`options`可以包含`start`和`end`值来读取指定范围的文件数据。`start`和`end`这两个位置本身，也都是被包括的，并且`start`以`0`开始。编码可以是`'utf8'`，`'ascii'`或`'base64'`。

如果指定了`fd`，可读流将会忽略`path`参数并且将会使用指定的文件描述符。这意味`open`事件不再会触发。

如果`autoClose`为`false`，那么文件描述符将不会被关闭，甚至是有错误发生时。关闭它将是你的责任，并且要确保没有文件描述符泄漏。如果`autoClose`为`true`（默认），那么在发生错误时，或到达文件描述末端时，它会被自动关闭。

从一个100字节的文件中读取最后10字节数据的例子：

```js
fs.createReadStream('sample.txt', {start: 90, end: 99});
```

如果`options`是一个字符串，那么它表示指定的编码。

#### Class: fs.ReadStream#

`ReadStream`是一个可读流。

#### Event: 'open'#

 - fd Integer 被可读流使用的文件描述符

当可读流文件被打开时触发。

#### fs.createWriteStream(path[, options])#

返回一个新的可写流对象（参阅`Writable Stream`）。

`options`是一个有以下默认值的对象或字符串：

```js
{ flags: 'w',
  encoding: null,
  fd: null,
  mode: 0o666 }
```

`options`可以包含一个`start`选项来允许从指定位置开始写入数据。修改一个文件而不是替换它，需要一个`r+`标识，而不是默认的`w`。编码可以是`'utf8'`，`'ascii'`，`'binary'`或`'base64'`。

与上文的`ReadStream`类似，如果指定了`fd`，可写流会忽略`path`参数，并且使用指定的文件描述符。这意味`open`事件不再会触发。

如果`options`是一个字符串，那么它表示指定的编码。

#### Class: fs.WriteStream#

`WriteStream`是一个可写流。

#### Event: 'open'#

 - fd Integer `WriteStream`使用的文件描述符

当可写流文件被打开时触发。

#### file.bytesWritten#

至今为止写入的字节数。不包括仍在写入队列中的数据。

#### Class: fs.FSWatcher#

由`fs.watch()`返回的对象。

#### watcher.close()#

停止在指定的`fs.FSWatcher`上监视文件变化。

#### Event: 'change'#

 - event String 文件的改变类型
 - filename String The filename that changed (if relevant/available)被改变的文件（如果有意义/可用的话）

当被监视的目录或文件发生了改变时触发。详情参阅`fs.watch`。

#### Event: 'error'#

 - error Error object
 
当错误发生时触发。