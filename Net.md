# net#

### 稳定度: 2 - 稳定
`net`模块为你提供了异步的网络调用的包装。它同时包含了创建服务器和客户端的函数。你可以通过`require('net')`来引入这个模块。

#### net.createServer([options][, connectionListener])#
创建一个新的TCP服务器。`connectionListener`参数会被自动绑定为`connection`事件的监听器。

`options`是一个包含下列默认值的对象：

```js
{
  allowHalfOpen: false,
  pauseOnConnect: false
}
```

如果`allowHalfOpen`是`true`，那么当另一端的`socket`发送一个`FIN`报文时`socket`并不会自动发送`FIN`报文。`socket`变得不可读，但是可写。你需要明确地调用`end()`方法。详见`end`事件。

如果`pauseOnConnect`是true，那么`socket`在每一次被连接时会暂停，并且不会读取数据。这允许在进程间被传递的连接不读取任何数据。如果要让一个被暂停的`socket`开始读取数据，调用`resume()`方法。

以下是一个应答服务器的例子，监听8124端口：

```js
var net = require('net');
var server = net.createServer(function(c) { //'connection' listener
  console.log('client connected');
  c.on('end', function() {
    console.log('client disconnected');
  });
  c.write('hello\r\n');
  c.pipe(c);
});
server.listen(8124, function() { //'listening' listener
  console.log('server bound');
});
```

使用`telnet`测试：

```
telnet localhost 8124
```

想要监听`socket``/tmp/echo.sock`，只需改变倒数第三行：

```js
server.listen('/tmp/echo.sock', function() { //'listening' listener
```

使用`nc`连接一个UNIX domain socket服务器：

```
nc -U /tmp/echo.sock
```

#### net.connect(options[, connectionListener])#
#### net.createConnection(options[, connectionListener])#

工厂函数，返回一个新的`net.Socket`实例，并且自动使用提供的`options`进行连接。

`options`会被同时传递给`net.Socket`构造函数和`socket.connect`方法。

参数`connectListener`将会被立即添加为`connect`事件的监听器。

下面是一个上文应答服务器的客户端的例子：

```js
var net = require('net');
var client = net.connect({port: 8124},
    function() { //'connect' listener
  console.log('connected to server!');
  client.write('world!\r\n');
});
client.on('data', function(data) {
  console.log(data.toString());
  client.end();
});
client.on('end', function() {
  console.log('disconnected from server');
});
```

要连接`socket``/tmp/echo.sock`只需要改变第二行为：

```js
var client = net.connect({path: '/tmp/echo.sock'});
```

#### net.connect(port[, host][, connectListener])#
#### net.createConnection(port[, host][, connectListener])#

工厂函数，返回一个新的`net.Socket`实例，并且自动使用指定的端口(port)和主机(host)进行连接。

如果`host`被省略，默认为`localhost`。

参数`connectListener`将会被立即添加为`connect`事件的监听器。

#### net.connect(path[, connectListener])#
#### net.createConnection(path[, connectListener])#

工厂函数，返回一个新的unix`net.Socket`实例，并且自动使用提供的路径(path)进行连接。

参数`connectListener`将会被立即添加为`connect`事件的监听器。

#### Class: net.Server#
这个类用于创建一个TCP或本地服务器。

#### server.listen(port[, hostname][, backlog][, callback])#

开始从指定端口和主机名接收连接。如果省略主机名，那么如果IPv6可用，服务器会接受从任何IPv6地址（::）来的链接，否则为任何IPv4地址（0.0.0.0）。如果端口为0那么将会为其设置一个随机端口。

积压量`backlog`是连接等待队列的最大长度。实际长度由你的操作系统的`sysctl`设置决定（如linux中的`tcp_max_syn_backlog`和`somaxconn`）。这个参数的默认值是511（不是512）。

这个函数式异步的。当服务器绑定了指定端口后，`listening`事件将会被触发。最后一个参数`callback`将会被添加为`listening`事件的监听器。

有些用户可能遇到的情况是收到`EADDRINUSE`错误。这意味着另一个服务器已经使用了该端口。一个解决的办法是等待一段时间后重试。

```js
server.on('error', function (e) {
  if (e.code == 'EADDRINUSE') {
    console.log('Address in use, retrying...');
    setTimeout(function () {
      server.close();
      server.listen(PORT, HOST);
    }, 1000);
  }
});
```

（注意，`io.js`中所有的`socket`都已经设置了`SO_REUSEADDR`）

#### server.listen(path[, callback])#

 - path String
 - callback Function

启动一个本地`socket`服务器，监听指定路径（`path`）上的连接。

这个函数式异步的。当服务器监听了指定路径后，`listening`事件将会被触发。最后一个参数`callback`将会被添加为`listening`事件的监听器。

在UNIX中，`local domain`经常被称作`UNIX domain`。`path`是一个文件系统路径名。它在被创建时会受相同文件名约定(same naming conventions)的限制并且进行权限检查(permissions checks)。它在文件系统中可见，并且在被删除前持续存在。

在Windows中，`local doamin`使用一个命名管道（named pipe）实现。`path`必须指向`\\?\pipe\`或`\\.\pipe\.`中的一个条目，但是后者可能会做一些命名管道的处理，如处理`..`序列。除去表现，命名管道空间是平坦的（flat）。管道不会持续存在，它们将在最后一个它们的引用关闭后被删除。不要忘记，由于`JavaScript`的字符串转义，你必须在指定`path`时使用双反斜杠：

```js
net.createServer().listen(
    path.join('\\\\?\\pipe', process.cwd(), 'myctl'))
```

#### server.listen(handle[, callback])#

 - handle Object
 - callback Function

`handle`对象可以被设置为一个服务器或一个`socket`（或者任意以下划线开头的成员`_handle`），或者一个`{fd: <n>}`对象。

这将使得服务器使用指定句柄接受连接，但它假设文件描述符或句柄已经被绑定至指定的端口或域名`socket`。

在Windows下不支持监听一个文件描述符。

这个函数式异步的。当服务器已被绑定后，`listening`事件将会被触发。最后一个参数`callback`将会被添加为`listening`事件的监听器。

#### server.listen(options[, callback])#

 - __options Object__
  - port Number 可选
  - host String 可选
  - backlog Number 可选
  - path String 可选
  - exclusive Boolean 可选

 - callback Function 可选

`port`，`host`和`backlog`属性，以及可选的`callback`函数，与`server.listen(port, [host], [backlog], [callback])`中表现一致。`path`可以被指定为一个UNIX `socket`。

如果`exclusive`是`false`（默认），那么工作集群（cluster workers）将会使用相同的底层句柄，处理的连接的职责将会被它们共享。如果`exclusive`是`true`，那么句柄是不被共享的，企图共享将得到一个报错的结果。下面是一个监听独有端口的例子：

```js
server.listen({
  host: 'localhost',
  port: 80,
  exclusive: true
});
```

#### server.close([callback])#

使服务器停止接收新的连接并且保持已存在的连接。这个函数式异步的，当所有的连接都结束时服务器会最终关闭，并处罚一个`close`事件。可选的，你可以传递一个回调函数来监听`close`事件。如果传递了，那么它的唯一的第一个参数将表示任何可能潜在发生的错误。

#### server.address()#

返回服务器绑定的地址，协议族名和端口通过操作系统报告。对查找操作系统分配的地址哪个端口被分配非常有用。返回一个有三个属性的对象。如`{ port: 12346, family: 'IPv4', address: '127.0.0.1' }`。

例子：

```js
var server = net.createServer(function (socket) {
  socket.end("goodbye\n");
});

// grab a random port.
server.listen(function() {
  address = server.address();
  console.log("opened server on %j", address);
});
```

在`listening`事件触发前，不要调用`server.address()`方法。

#### server.unref()#

调用一个`server`对象的`unref`方法将允许如果它是事件系统中唯一活跃的服务器，程序将会退出。如果服务器已经被调用过这个方法，那么再次调用这个方法将不会有任何效果。

返回`server`对象。

#### server.ref()#

与`unref`相反，在一个已经被调用`unref`方法的`server`中调用`ref`方法，那么如果它是唯一活跃的服务器时，程序将不会退出（默认）。如果服务器已经被调用过这个方法，那么再次调用这个方法将不会有任何效果。

返回`server`对象。

#### server.maxConnections#

设置了这个属性后，服务器的连接数达到时将会开始拒绝连接。

一旦`socket`被使用`child_process.fork()`传递给了子进程，这个属性就不被推荐去设置。

#### server.connections#

这个函数已经被弃用。请使用`server.getConnections()`替代。

服务器的当前连接数。

当使用`child_process.fork()`传递一个`socket`给子进程时，这个属性将变成`null`。想要得到正确的结果请使用`server.getConnections`。

#### server.getConnections(callback)#

异步地去获取服务器的当前连接数，在`socket`被传递给子进程时仍然可用。

回调函数的两个参数是`err`和`count`。

### `net.Server`是一个具有以下事件的`EventEmitter`：

#### Event: 'listening'#

当调用`server.listen`后，服务器已被绑定时触发。

#### Event: 'connection'#

 - Socket object 连接对象

当新的连接产生时触发。`socket`是一个`net.Socket`实例。

#### Event: 'close'#

当服务器关闭时触发。注意如果服务器中仍有连接存在，那么这个事件会直到所有的连接都关闭后才触发。

#### Event: 'error'#

 - Error Object

当发生错误时触发。`close`事件将会在它之后立即触发。参阅`server.listen`。

#### Class: net.Socket#

这个对象是一个TCP或本地`socket`的抽象。`net.Socket`实例实现了双工流（duplex Stream）接口。它可以被使用者创建，并且被作为客户端（配合`connect()`）使用。或者也可以被`io.js`创建，并且通过服务器的`connection`事件传递给使用者。

#### new net.Socket([options])#

创建一个新的`socket`对象。

`options`是一个有以下默认值的对象：

```js
{ fd: null
  allowHalfOpen: false,
  readable: false,
  writable: false
}
```

`fd`允许你使用一个指定的已存在的`socket`文件描述符。设置`readable` 和/或 `writable`为`true`将允许从这个`socket`中读 和/或 写（注意，仅在传递了`passed`时可用）。关于`allowHalfOpen`，参阅`createServer()`和`end`事件。

#### socket.connect(options[, connectListener])#

从给定的`socket`打开一个连接。

对于TCP`socket`，`options`参数需是一个包含以下属性的对象：

 - port: 客户端需要连接的端口（必选）。

 - host: 客户端需要连接的主机（默认：'localhost'）

 - localAddress: 将要绑定的本地接口，为了网络连接。

 - localPort: 将要绑定的本地端口，为了网络连接。

 - family : IP协议族版本，默认为`4`。

 - lookup : 自定义查找函数。默认为`dns.lookup`。

对于本地domain `socket`，`options`参数需是一个包含以下属性的对象：

 - path: 客户端需要连接的路径（必选）。

通常这个方法是不需要的，因为通过`net.createConnection`打开`socket`。只有在你自定义了`socket`时才使用它。

这个函数式异步的，当`connect`事件触发时，这个`socket`就被建立了。如果在连接的过程有问题，那么`connect`事件将不会触发，`error`将会带着这个异常触发。

`connectListener`参数会被自动添加为`connect`事件的监听器。

#### socket.connect(port[, host][, connectListener])#

#### socket.connect(path[, connectListener])#

参阅`socket.connect(options[, connectListener])`。

#### socket.bufferSize#

`net.Socket`的属性，用于`socket.write()`。它可以帮助用户获取更快的运行速度。计算机不能一直保持大量数据被写入`socket`的状态，网络连接可以很慢。`io.js`在内部会排队等候数据被写入`socekt`并确保传输连接上的数据完好。 (内部实现为：轮询`socekt`的文件描述符等待它为可写)。

内部缓存的可能结果是内存使用会增长。这个属性展示了缓存中还有多少待写入的字符（字符的数目约等于要被写入的字节数，但是缓冲区可能包含字符串，而字符串是惰性编码的，所以确切的字节数是未知的）。

遇到数值很大或增长很快的`bufferSize`时，应当尝试使用`pause()`和`resume()`来控制。

#### socket.setEncoding([encoding])#

设置`socket`的编码作为一个可读流。详情参阅`stream.setEncoding()`。

#### socket.write(data[, encoding][, callback])#

在套接字上发送数据。第二个参数指定了字符串的编码，默认为UTF8。

如果所有数据成功被刷新至了内核缓冲区，则返回`true`。如果所有或部分数据仍然在用户内存中排队，则返回`false`。`drain`事件将会被触发当`buffer`再次为空时。

当数据最终被写入时，`callback`回调函数将会被执行，但可能不会马上执行。

#### socket.end([data][, encoding])#

半关闭一个`socket`。比如，它发送一个`FIN`报文。可能服务器仍然在发送一些数据。

如果`data`参数被指定，那么等同于先调用`socket.write(data, encoding)`，再调用`socket.end()`。

#### socket.destroy()#

确保这个`socket`上没有I/O活动发生。只在发生错误情况才需要（如处理错误）。

#### socket.pause()#

暂停数据读取。`data`事件将不会再触发。对于控制上传非常有用。

#### socket.resume()#

用于在调用`pause()`后，恢复数据读取。

#### socket.setTimeout(timeout[, callback])#

如果`socket`在`timeout`毫秒中没有活动后，设置其为超时。默认情况下，`net.Socket`没有超时。

当超时发生，`socket`会收到一个`timeout`事件，但是连接将不会被断开。用户必须手动地调用`end()`或`destroy()`方法。

如果`timeout`是`0`，那么现有的超时将会被禁用。

可选的`callback`参数就会被自动添加为`timeout`事件的监听器。

返回一个`socket`。

#### socket.setNoDelay([noDelay])#

警用纳格算法（Nagle algorithm）。默认情况下TCP连接使用纳格算法，它们的数据在被发送前会被缓存。设置`noDelay`为`true`将会在每次`socket.write()`时立刻发送数据。`noDelay`默认为`true`。

返回一个`socket`。

#### socket.setKeepAlive([enable][, initialDelay])#

启用/警用长连接功能，并且在第一个在闲置`socket`的长连接`probe`被发送前，可选得设置初始延时。`enable`默认为`false`。

设定`initialDelay`(毫秒)，来设定在收到的最后一个数据包和第一个长连接`probe`之间的延时。将`initialDelay`设成`0`会让值保持不变(默认值或之前所设的值)。默认为`0`。

返回一个`socket`。

#### socket.address()#

返回绑定的地址，协议族名和端口通过操作系统报告。对查找操作系统分配的地址哪个端口被分配非常有用。返回一个有三个属性的对象。如`{ port: 12346, family: 'IPv4', address: '127.0.0.1' }`。

#### socket.unref()#

调用一个`socket`对象的`unref`方法将允许如果它是事件系统中唯一活跃的`socket`，程序将会退出。如果`socket`已经被调用过这个方法，那么再次调用这个方法将不会有任何效果。

返回`socket`对象。

#### socket.ref()#

与`unref`相反，在一个已经被调用`unref`方法的`socket`中调用`ref`方法，那么如果它是唯一活跃的`socket`时，程序将不会退出（默认）。如果`socket`已经被调用过这个方法，那么再次调用这个方法将不会有任何效果。

返回`socket`对象。

#### socket.remoteAddress#

远程IP地址字符串。例如，`'74.125.127.100'`或`'2001:4860:a005::68'`。

#### socket.remoteFamily#

远程IP协议族字符串。例如，`'IPv4'`或`'IPv6'`。

#### socket.remotePort#

远程端口数值。例如，`80`或`21`。

#### socket.localAddress#

远程客户端正连接的本地IP地址字符串。例如，如果你正在监听`'0.0.0.0'`并且客户端连接在`'192.168.1.1'`，其值将为`'192.168.1.1'`。

#### socket.localPort#

本地端口数值。例如，`80`或`21`。

socket.bytesRead#

接受的字节数。

socket.bytesWritten#

发送的字节数。

### net.Socket `net.Socket`实例是一个包含以下事件的`EventEmitter`：

#### Event: 'lookup'#

在解析主机名后，连接主机前触发。对UNIX `socket`不适用。

 - err {Error | Null} 错误对象，参阅 `dns.lookup()`
 - address {String} IP地址
 - family {String | Null} 地址类型。参阅'dns.lookup()`

#### Event: 'connect'#

在`socket`连接成功建立后触发。参阅`connect()`。

#### Event: 'data'#

 - Buffer object

在接受到数据后触发。参数将会是一个`Buffer`或一个字符串。数据的编码由`socket.setEncoding()`设置（更多详细信息请查看可读流章节）。

注意，当`socket`触发`data`事件时，如果没有监听器存在。那么数据将会丢失。

#### Event: 'end'#

当另一端的`socket`发送一个`FIN`报文时触发。

默认情况（`allowHalfOpen == false`）下，一旦一个`socket`的文件描述符被从它的等待写队列（pending write queue）中写出，`socket`会销毁它。但是，当设定`allowHalfOpen == true`后，`socket`不会在它这边自动调用`end()`，允许用户写入任意数量的数据，需要注意的是用户需要在自己这边调用`end()`。

#### Event: 'timeout'#

当`socket`因不活动而超时时触发。这只是来表示`socket`被限制。用户必须手动关闭连接。

参阅`socket.setTimeout()`。

#### Event: 'drain'#

当写缓冲为空时触发。可以被用来控制上传流量。

参阅`socket.write()`的返回值。

#### Event: 'error'#

 - Error object

当发生错误时触发。`close`事件会紧跟着这个事件触发。

#### Event: 'close'#

 - had_error 如果`socket`有一个传输错误时为`true`

当`socket`完全关闭时触发。参数`had_error`是一个表示`socket`是否是因为传输错误而关闭的布尔值。

#### net.isIP(input)#
测试`input`是否是一个IP地址。如果是不合法字符串时，会返回`0`。如果是IPv4地址则返回`4`，是IPv6地址则返回`6`。

#### net.isIPv4(input)#
如果`input`是一个IPv4地址则返回`true`，否则返回`false`。

#### net.isIPv6(input)#
如果`input`是一个IPv6地址则返回`true`，否则返回`false`。