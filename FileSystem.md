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
Watch for changes on filename, where filename is either a file or a directory. The returned object is a fs.FSWatcher.

The second argument is optional. The options if provided should be an object. The supported boolean members are persistent and recursive. persistent indicates whether the process should continue to run as long as files are being watched. recursive indicates whether all subdirectories should be watched, or only the current directory. This applies when a directory is specified, and only on supported platforms (See Caveats below).

The default is { persistent: true, recursive: false }.

The listener callback gets two arguments (event, filename). event is either 'rename' or 'change', and filename is the name of the file which triggered the event.

#### Caveats#

The fs.watch API is not 100% consistent across platforms, and is unavailable in some situations.

The recursive option is currently supported on OS X. Only FSEvents supports this type of file watching so it is unlikely any additional platforms will be added soon.

#### Availability#

This feature depends on the underlying operating system providing a way to be notified of filesystem changes.

On Linux systems, this uses inotify.
On BSD systems, this uses kqueue.
On OS X, this uses kqueue for files and 'FSEvents' for directories.
On SunOS systems (including Solaris and SmartOS), this uses event ports.
On Windows systems, this feature depends on ReadDirectoryChangesW.
If the underlying functionality is not available for some reason, then fs.watch will not be able to function. For example, watching files or directories on network file systems (NFS, SMB, etc.) often doesn't work reliably or at all.

You can still use fs.watchFile, which uses stat polling, but it is slower and less reliable.

#### Filename Argument#

Providing filename argument in the callback is not supported on every platform (currently it's only supported on Linux and Windows). Even on supported platforms filename is not always guaranteed to be provided. Therefore, don't assume that filename argument is always provided in the callback, and have some fallback logic if it is null.

fs.watch('somedir', function (event, filename) {
  console.log('event is: ' + event);
  if (filename) {
    console.log('filename provided: ' + filename);
  } else {
    console.log('filename not provided');
  }
});
#### fs.exists(path, callback)#
fs.exists() is deprecated. For supported alternatives please check out fs.stat or fs.access.

Test whether or not the given path exists by checking with the file system. Then call the callback argument with either true or false. Example:

#### fs.exists('/etc/passwd', function (exists) {
  util.debug(exists ? "it's there" : "no passwd!");
});
fs.exists() is an anachronism and exists only for historical reasons. There should almost never be a reason to use it in your own code.

In particular, checking if a file exists before opening it is an anti-pattern that leaves you vulnerable to race conditions: another process may remove the file between the calls to fs.exists() and fs.open(). Just open the file and handle the error when it's not there.

#### fs.existsSync(path)#
Synchronous version of fs.exists. Returns true if the file exists, false otherwise.

fs.existsSync() is deprecated. For supported alternatives please check out fs.statSync or fs.accessSync.

#### fs.access(path[, mode], callback)#
Tests a user's permissions for the file specified by path. mode is an optional integer that specifies the accessibility checks to be performed. The following constants define the possible values of mode. It is possible to create a mask consisting of the bitwise OR of two or more values.

fs.F_OK - File is visible to the calling process. This is useful for determining if a file exists, but says nothing about rwx permissions. Default if no mode is specified.
fs.R_OK - File can be read by the calling process.
fs.W_OK - File can be written by the calling process.
fs.X_OK - File can be executed by the calling process. This has no effect on Windows (will behave like fs.F_OK).
The final argument, callback, is a callback function that is invoked with a possible error argument. If any of the accessibility checks fail, the error argument will be populated. The following example checks if the file /etc/passwd can be read and written by the current process.

fs.access('/etc/passwd', fs.R_OK | fs.W_OK, function(err) {
  util.debug(err ? 'no access!' : 'can read/write');
});
fs.accessSync(path[, mode])#
Synchronous version of fs.access. This throws if any accessibility checks fail, and does nothing otherwise.

#### Class: fs.Stats#
Objects returned from fs.stat(), fs.lstat() and fs.fstat() and their synchronous counterparts are of this type.

stats.isFile()
stats.isDirectory()
stats.isBlockDevice()
stats.isCharacterDevice()
stats.isSymbolicLink() (only valid with fs.lstat())
stats.isFIFO()
stats.isSocket()
For a regular file util.inspect(stats) would return a string very similar to this:

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
Please note that atime, mtime, birthtime, and ctime are instances of Date object and to compare the values of these objects you should use appropriate methods. For most general uses getTime() will return the number of milliseconds elapsed since 1 January 1970 00:00:00 UTC and this integer should be sufficient for any comparison, however there are additional methods which can be used for displaying fuzzy information. More details can be found in the MDN JavaScript Reference page.

#### Stat Time Values#

The times in the stat object have the following semantics:

atime "Access Time" - Time when file data last accessed. Changed by the mknod(2), utimes(2), and read(2) system calls.
mtime "Modified Time" - Time when file data last modified. Changed by the mknod(2), utimes(2), and write(2) system calls.
ctime "Change Time" - Time when file status was last changed (inode data modification). Changed by the chmod(2), chown(2), link(2), mknod(2), rename(2), unlink(2), utimes(2), read(2), and write(2) system calls.
birthtime "Birth Time" - Time of file creation. Set once when the file is created. On filesystems where birthtime is not available, this field may instead hold either the ctime or 1970-01-01T00:00Z (ie, unix epoch timestamp 0). On Darwin and other FreeBSD variants, also set if the atime is explicitly set to an earlier value than the current birthtime using the utimes(2) system call.
Prior to io.js v1.0 and Node v0.12, the ctime held the birthtime on Windows systems. Note that as of v0.12, ctime is not "creation time", and on Unix systems, it never was.

fs.createReadStream(path[, options])#
Returns a new ReadStream object (See Readable Stream).

options is an object or string with the following defaults:

{ flags: 'r',
  encoding: null,
  fd: null,
  mode: 0o666,
  autoClose: true
}
options can include start and end values to read a range of bytes from the file instead of the entire file. Both start and end are inclusive and start at 0. The encoding can be 'utf8', 'ascii', or 'base64'.

If fd is specified, ReadStream will ignore the path argument and will use the specified file descriptor. This means that no open event will be emitted.

If autoClose is false, then the file descriptor won't be closed, even if there's an error. It is your responsibility to close it and make sure there's no file descriptor leak. If autoClose is set to true (default behavior), on error or end the file descriptor will be closed automatically.

An example to read the last 10 bytes of a file which is 100 bytes long:

fs.createReadStream('sample.txt', {start: 90, end: 99});
If options is a string, then it specifies the encoding.

#### Class: fs.ReadStream#
ReadStream is a Readable Stream.

#### Event: 'open'#

fd Integer file descriptor used by the ReadStream.
Emitted when the ReadStream's file is opened.

fs.createWriteStream(path[, options])#
Returns a new WriteStream object (See Writable Stream).

options is an object or string with the following defaults:

{ flags: 'w',
  encoding: null,
  fd: null,
  mode: 0o666 }
options may also include a start option to allow writing data at some position past the beginning of the file. Modifying a file rather than replacing it may require a flags mode of r+ rather than the default mode w. The encoding can be 'utf8', 'ascii', binary, or 'base64'.

Like ReadStream above, if fd is specified, WriteStream will ignore the path argument and will use the specified file descriptor. This means that no open event will be emitted.

If options is a string, then it specifies the encoding.

#### Class: fs.WriteStream#
WriteStream is a Writable Stream.

#### Event: 'open'#

fd Integer file descriptor used by the WriteStream.
Emitted when the WriteStream's file is opened.

#### file.bytesWritten#

The number of bytes written so far. Does not include data that is still queued for writing.

#### Class: fs.FSWatcher#
Objects returned from fs.watch() are of this type.

#### watcher.close()#

Stop watching for changes on the given fs.FSWatcher.

#### Event: 'change'#

event String The type of fs change
filename String The filename that changed (if relevant/available)
Emitted when something changes in a watched directory or file. See more details in fs.watch.

#### Event: 'error'#

error Error object
Emitted when an error occurs. 