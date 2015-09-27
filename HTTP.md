# HTTP#

### 稳定度: 2 - 稳定

你必须通过`require('http')`来使用HTTP服务器和客户端。

`node.js`中的HTTP接口被设置来支持许多HTTP协议里原本用起来很困难的特性。特别是大且成块的有编码的消息。这个接口从不缓冲整个请求或响应。用户可以对它们使用流。

HTTP消息头可能是一个类似于以下例子的对象：

```JS
{ 'content-length': '123',
  'content-type': 'text/plain',
  'connection': 'keep-alive',
  'host': 'mysite.com',
  'accept': '*/*' }
```

键是小写的。值没有被修改。

为了全方位的支持所有的HTTP应用。`node.js`的HTTP API是非常底层的。它只处理流以及解释消息。它将消息解释为消息头和消息体，但是不解释实际的消息头和消息体。

被定义的消息头允许以多个`,`字符分割，除了`set-cookie`和`cookie`头，因为它们表示值得数组。如`content-length`这样只有单个值的头被直接将解析，并且成为解析后对象的一个单值。

收到的原始消息头会被保留在`rawHeaders`属性中，它是一个形式如`[key, value, key2, value2, ...]`的数组。例如，之前的消息头可以有如下的`rawHeaders`：

```js
[ 'ConTent-Length', '123456',
  'content-LENGTH', '123',
  'content-type', 'text/plain',
  'CONNECTION', 'keep-alive',
  'Host', 'mysite.com',
  'accepT', '*/*' ]
```

#### http.METHODS#
 - Array

一个被解析器所支持的HTTP方法的列表。

#### http.STATUS_CODES#
 - Object

一个所有标准HTTP响应状态码的集合，以及它们的简短描述。例如，`http.STATUS_CODES[404] === 'Not Found'`。

#### http.createServer([requestListener])#
 - 返回一个新的`http.Server`实例

`requestListener`是一个会被自动添加为`request`事件监听器的函数。

#### http.createClient([port][, host])#

这个函数已经被启用。请使用`http.request()`替代。构造一个新的HTTP客户端。`port`和`host`指定了需要连接的目标服务器。

#### Class: http.Server#

这是一个具有以下事件的`EventEmitter`：

#### Event: 'request'#

 - function (request, response) { }

当有请求来到时触发。注意每一个连接可能有多个请求（在长连接的情况下）。请求是一个`http.IncomingMessage`实例，响应是一个`http.ServerResponse`实例。

#### Event: 'connection'#

 - function (socket) { }

当一个新的TCP流建立时触发。`socket`是一个`net.Socket`类型的实例。用户通常不会接触这个事件。特别的，因为协议解释器绑定它的方式，`socket`将不会触发`readable`事件。这个`socket`可以由`request.connection`得到。


#### Event: 'close'#

 - function () { }

当服务器关闭时触发。

#### Event: 'checkContinue'#

 - function (request, response) { }

当每次收到一个HTTP`Expect: 100-continue`请求时触发。如果不监听这个事件，那么服务器会酌情自动响应一个`100 Continue`。

处理该事件时，如果客户端可以继续发送请求主体则调用`response.writeContinue()`， 如果不能则生成合适的HTTP响应（如`400 Bad Request`）。

注意，当这个事件被触发并且被处理，`request`事件则不会再触发。

#### Event: 'connect'#

 - function (request, socket, head) { }

每当客户端发起一个http`CONNECT`请求时触发。如果这个事件没有被监听，那么客户端发起http`CONNECT`的连接会被关闭。

 - `request`是一个http请求参数，它也被包含在`request`事件中。
 - `socket`是一个服务器和客户端间的网络套接字。
 - `head`是一个`Buffer`实例，隧道流中的第一个报文，该参数可能为空

在这个事件被触发后，请求的`socket`将不会有`data`事件的监听器，意味着你将要绑定一个`data`事件的监听器来处理这个`socket`中发往服务器的数据。

#### Event: 'upgrade'#

 - function (request, socket, head) { }

每当客户端发起一个http`upgrade`请求时触发。如果这个事件没有被监听，那么客户端发起`upgrade`的连接会被关闭。

 - `request`是一个http请求参数，它也被包含在`request`事件中。
 - `socket`是一个服务器和客户端间的网络套接字。
 - `head`是一个`Buffer`实例，升级后流中的第一个报文，该参数可能为空
 
在这个事件被触发后，请求的`socket`将不会有`data`事件的监听器，意味着你将要绑定一个`data`事件的监听器来处理这个`socket`中发往服务器的数据。

#### Event: 'clientError'#

 - function (exception, socket) { }

如果一个客户端连接发生了错误，这个事件将会被触发。

`socket`是一个错误来源的`net.Socket`对象。

#### server.listen(port[, hostname][, backlog][, callback])#

从指定的端口和主机名开始接收连接。如果`hostname`被忽略，那么如果IPv6可用，服务器将接受任意IPv6地址（::），否则为任何IPv4地址（0.0.0.）。`port`为`0`将会设置一个随机端口。

如果要监听一个unix `socket`，请提供一个文件名而不是端口和主机名。

`backlog`是连接等待队列的最大长度。它的实际长度将有你操作系统的`sysctl`设置（如linux中的`tcp_max_syn_backlog`和`somaxconn`）决定。默认值为`511`（不是`512`）。

这个函数式异步的。最后一个`callback`参数将会添加至`listening`事件的监听器。参阅`net.Server.listen(port)`。

#### server.listen(path[, callback])#

通过给定的`path`，开启一个监听连接的 UNIX `socket`服务器。

这个函数式异步的。最后一个`callback`参数将会添加至`listening`事件的监听器。参阅`net.Server.listen(path)`。

#### server.listen(handle[, callback])#

 - handle Object
 - callback Function

`handle`对象是既可以是一个server可以是一个`socket`（或者任意以下划线开头的成员`_handle`），或一个`{fd: <n>}`对象。

这将使得服务器使用指定句柄接受连接，但它假设文件描述符或句柄已经被绑定至指定的端口或域名`socket`。

在Windows下不支持监听一个文件描述符。

这个函数式异步的。最后一个`callback`参数将会添加至`listening`事件的监听器。参阅`net.Server.listen()`。

#### server.close([callback])#

让服务器停止接收新的连接。参阅`net.Server.close()`。

#### server.maxHeadersCount#

限制最大请求头数量，默认为`1000`。如果设置为`0`，则代表无限制。

#### server.setTimeout(msecs, callback)#

 - msecs Number
 - callback Function

设置`socket`的超时值，并且如果超时，会在服务器对象上触发一个`timeout`事件，并且将传递`socket`作为参数。

如果在服务器对象时又一个`timeout`事件监听器，那么它将会被调用，而超时的`socket`将会被作为参数。

默认的，服务器的超时值是两分钟，并且如果超时，`socket`会被自动销毁。但是，如果你给`timeout`事件传递了回调函数，那么你必须为要亲自处理`socket`超时。

返回一个`server`对象。

#### server.timeout#

 - Number 默认为`120000`（两分钟）

一个`socket`被判定为超时之前的毫秒数。

注意，`socket`的超时逻辑在连接时被设定，所以改变它的值仅影响之后到达服务器的连接，而不是所有的连接。

设置为`0`将会为连接禁用所有的自动超时行为。

#### Class: http.ServerResponse#

这个对象由HTTP服务器内部创建，而不是由用户。它会被传递给`request`事件监听器的第二个参数。

这个对象实现了`Writable`流接口。它是一个具有以下事件的`EventEmitter`：

#### Event: 'close'#

 - function () { }

表明底层的连接在`response.end()`被调用或能够冲刷前被关闭。

#### Event: 'finish'#

 - function () { }

当响应被设置时触发。更明确地说，这个事件在当响应头的最后一段和响应体为了网络传输而交给操作系统时触发。它并不表明客户端已经收到了任何信息。

这个事件之后，`response`对象不会再触发任何事件。

#### response.writeContinue()#

给客户端传递一个`HTTP/1.1 100 Continue`信息，表明请求体必须被传递。参阅服务器的`checkContinue`事件。

#### response.writeHead(statusCode[, statusMessage][, headers])#

为请求设置一个响应头。`statusCode`是一个三位的HTTP状态码，如`404`。最后一个参数`headers`，是响应头。第二个参数`statusMessage`是可选的，表示状态码的一个可读信息。

例子：

```js
var body = 'hello world';
response.writeHead(200, {
  'Content-Length': body.length,
  'Content-Type': 'text/plain' });
```

这个方法对于一个信息只能调用一次，并且它必须在`response.end()`之前被调用。

如果你在调用这个方法前调用了`response.write()`或`response.end()`，将会调用这个函数，并且一个`implicit/mutable`头会被计算使用。

注意，`Content-Length`是以字节计，而不是以字符计。上面例子能正常运行时因为字符串`'hello world'`仅包含单字节字符。如果响应体包含了多字节编码的字符，那么必须通过指定的编码来调用`Buffer.byteLength()`来确定字节数。并且`node.js`不会检查`Content-Length`与响应体的字节数是否相等。

#### response.setTimeout(msecs, callback)#

 - msecs Number
 - callback Function

设置`socket`的超时值（毫秒），如果传递了回调函数，那么它将被添加至`response`对象的`timeout`事件的监听器。

如果没有为`request`，`request`或服务器添加`timeout`监听器。那么`socket`会在超时时销毁。如果你为`request`，`request`或服务器添加了`timeout`监听器，那么你必须为要亲自处理`socket`超时。

返回一个`response`对象。

#### response.statusCode#

当使用隐式响应头（不明确调用`response.writeHead()`）时，这个属性控制了发送给客户端的状态码，在当响应头被冲刷时。

例子：

```js
response.statusCode = 404;
```

在响应头发送给客户端之后，这个属性表明了被发送的状态码。

#### response.statusMessage#

当使用隐式响应头（不明确调用`response.writeHead()`）时，这个属性控制了发送给客户端的状态信息，在当响应头被冲刷时。当它没有被指定（`undefined`）时，将会使用标准HTTP状态码信息。

例子：

```js
response.statusMessage = 'Not found';
```

在响应头发送给客户端之后，这个属性表明了被发送的状态信息。

#### response.setHeader(name, value)#

为一个隐式的响应头设置一个单独的头内容。如果这个头已存在，那么将会被覆盖。当你需要发送一个同名多值的头内容时请使用一个字符串数组。

例子：

```js
response.setHeader("Content-Type", "text/html");
//or

response.setHeader("Set-Cookie", ["type=ninja", "language=javascript"]);
```

#### response.headersSent#

布尔值（只读）。如果响应头被发送则为`true`，反之为`false`。

#### response.sendDate#

当为`true`时，当响应头中没有`Date`值时会被自动设置。默认为`true`。

这个值只会为了测试目的才会被禁用。HTTP协议要求响应头中有`Date`值。

#### response.getHeader(name)#

读取已经被排队但还未发送给客户端的响应头。注意`name`是大小写敏感的。这个函数只能在响应头被隐式冲刷前被调用。

例子：

```JS
var contentType = response.getHeader('content-type');
```

#### response.removeHeader(name)#

取消一个在队列中等待隐式发送的头。

例子：

```js
response.removeHeader("Content-Encoding");
```

#### response.write(chunk[, encoding][, callback])#

如果这个方法被调用并且`response.writeHead()`没有备调用，那么它将转换到隐式响应头模式，并且刷新隐式响应头。

这个方法传递一个数据块的响应体。这个方法可能被调用多次来保证连续的提供响应体。

数据块可以是一个字符串或一个`buffer`。如果数据块是一个字符串，那么第二个参数是它的编码。默认是UTF-8.最后一个回调函数参数会在数据块被冲刷后触发。
注意：这是一个底层的HTTP报文，高级的多部分报文编码无法使用。

第一次调用`response.write()`时，它会传递缓存的头信息以及第一个报文给客户端。第二次调用时，`node.js`假设你将发送数据流，然后分别发送。这意味着响应式缓冲到第一个报文的数据块中。

如果整个数据都成功得冲刷至内核缓冲，则放回`true`。如果用户内存中有部分或全部的数据在队列中，那么返回`false`。`drain`事件将会在缓冲再次释放时触发。

#### response.addTrailers(headers)#

这个方法添加HTTP尾随头（一个在消息最后的头）给响应。

只有当数据编码被用于响应时尾随才会触发。如果不是（如请求是`HTTP/1.0`），它们将被安静地丢弃。

注意，如果你要触发尾随消息，HTTP要求传递一个包含报文头场列表的尾随头：

```js
response.writeHead(200, { 'Content-Type': 'text/plain',
                          'Trailer': 'Content-MD5' });
response.write(fileData);
response.addTrailers({'Content-MD5': "7895bf4b8828b55ceaf47747b4bca667"});
response.end();
```

#### response.end([data][, encoding][, callback])#

这个方法告知服务器所有的响应头和响应体都已经发送；服务器会认为这个消息完成了。这个方法必须在每次响应完成后被调用。

如果指定了`data`，就相当于调用了`response.write(data, encoding)`之后再调用`response.end(callback)`。

如果指定了回调函数，那么它将在响应流结束后触发。

#### http.request(options[, callback])#

`node.js`为每个服务器维护了几个连接，用来产生HTTP请求。这函数允许你透明地发送请求。

`options`参数可以是一个对象或一个字符串，如果`options`是一个字符串，它将自动得被`url.parse()`翻译。

__Options:__

 - host: 一个将要向其发送请求的服务器域名或IP地址。默认为`localhost`。
 - hostname: `host`的别名。为了支持`url.parse()`的话，`hostname`比`host`更好些。
 - family: 解析`host`和`hostname`时的IP地址协议族。合法值是`4`和`6`。当没有指定时，将都被使用。
 - port: 远程服务器端口。默认为`80`。
 - localAddress: 用于绑定网络连接的本地端口。
 - socketPath: Unix域`socket`（使用`host:port`或`socketPath`）。
 - method: 指定HTTP请求方法的字符串。默认为`GET`。
 - path: 请求路径。默认为`/`。如果有查询字符串，则需要包含。例如'/index.html?page=12'。请求路径包含非法字符时抛出异常。目前，只否决空格，不过在未来可能改变。
 - headers: 一个包含请求头的对象。
 - auth: 用于计算认证头的基本认证，即`'user:password'`。
 - __agent__: 控制`agent`行为。当使用一个代理时，请求将默认为`Connection: keep-alive`。可能值有：
  - undefined (默认): 在这个主机和端口上使用全局`agent。
  - Agent object: 在`agent`中显示使用passed。
  - false: 跳出`agent`的连接池。默认请求为`Connection: close`。

可选的回调函数将会被添加为`response`事件的“一次性”监听器（one time listener）。

`http.request()`返回一个`http.ClientRequest`类的实例。这个`ClientRequest`实例是一个可写流。如果你需要使用`POST`请求上传一个文件，那么就将之写入这个`ClientRequest `对象。

例子：

```js
var postData = querystring.stringify({
  'msg' : 'Hello World!'
});

var options = {
  hostname: 'www.google.com',
  port: 80,
  path: '/upload',
  method: 'POST',
  headers: {
    'Content-Type': 'application/x-www-form-urlencoded',
    'Content-Length': postData.length
  }
};

var req = http.request(options, function(res) {
  console.log('STATUS: ' + res.statusCode);
  console.log('HEADERS: ' + JSON.stringify(res.headers));
  res.setEncoding('utf8');
  res.on('data', function (chunk) {
    console.log('BODY: ' + chunk);
  });
  res.on('end', function() {
    console.log('No more data in response.')
  })
});

req.on('error', function(e) {
  console.log('problem with request: ' + e.message);
});

// write data to request body
req.write(postData);
req.end();
```

注意，在例子中调用了`req.end()`。使用`http.request()`时必须调用`req.end()`来表明你已经完成了请求（即使没有数据要被写入请求体）。

如果有一个错误在请求时发生（如DNS解析，TCP级别错误或实际的HTTP解析错误），一个`error`事件将会在返回对象上触发。

下面有一些特殊的需要主要的请求头：

 - 发送`'Connection: keep-alive'`会告知`node.js`保持连直到下一个请求发送。

 - 发送`'Content-length'`头会禁用默认的数据块编码。

 - 发送`'Expect'`头将会立刻发送一个请求头。通常，当发送`'Expect: 100-continue'`时，你需要同时设置一个超时和监听后续的时间。参阅RFC2616的8.2.3章节来获取更多信息。

 - 发送一个授权头将会覆盖使用`auth`选项来进行基本授权。

#### http.get(options[, callback])#
由于大多数请求是没有请求体的`GET`请求。`node.js`提供了这个简便的方法。这个方法和`http.request()`方法的唯一区别是它设置请求方法为`GET`且自动调用`req.end()`。

例子：

```js
http.get("http://www.google.com/index.html", function(res) {
  console.log("Got response: " + res.statusCode);
}).on('error', function(e) {
  console.log("Got error: " + e.message);
});
```

#### Class: http.Agent#

HTTP Agent是用来把HTTP客户端请求中的`socket`做成池。

HTTP Agent 也把客户端的请求默认为使用`Connection:keep-alive`。如果没有HTTP请求正在等待成为空闲的套接字的话，那么套接字将关闭。这意味着`node.js`的资源池在负载的情况下对`keep-alive`有利，但是仍然不需要开发人员使用KeepAlive来手动关闭HTTP客户端。

如果你选择使用`HTTP KeepAlive`，那么你可以创建一个标志设为`true`的Agent对象（见下面的构造函数选项）。然后，Agent将会在资源池中保持未被使用的套接字，用于未来使用。它们将会被显式标记，以便于不保持`node.js`进程的运行。但是当KeepAlive agent没有被使用时，显式地`destroy()` KeepAlive agent仍然是个好主意，这样`socket`会被关闭。

当`socket`触发了`close`事件或者特殊的`agentRemove`事件的时候，套接字们从agent的资源池中移除。这意味着如果你打算保持一个HTTP请求长时间开启，并且不希望它保持在资源池中，那么你可以按照下列几行的代码做事：

```js
http.get(options, function(res) {
  // Do stuff
}).on("socket", function (socket) {
  socket.emit("agentRemove");
});
```

另外，你可以使用`agent:false`来停用池：

```js
http.get({
  hostname: 'localhost',
  port: 80,
  path: '/',
  agent: false  // create a new agent just for this one request
}, function (res) {
  // Do stuff with response
})
```

new Agent([options])#

__options Object 为agent设置可配置的选项。可以有以下属性__:
 - keepAlive Boolean 在未来保持池中的`socket`被其他请求所使用，默认为`false`
 - keepAliveMsecs Integer 当使用HTTP KeepAlive时，通过被保持连接的`socket`发送TCP KeepAlive 报文的间隔。默认为`1000`。只在`KeepAlive`被设置为`true`时有效
 - maxSockets Number 每个主机允许拥有的`socket`的最大数量。默认为`Infinity `
 - maxFreeSockets Number 在空闲状态下允许打开的最大`socket`数。仅在`keepAlive`为`true`时有效。默认为`256`

`http.request`使用的默认的`http.globalAgent`包含它们属性的各自的默认值。

为了配置它们中的任何一个，你必须创建你自己的`Agent`对象。

```js
var http = require('http');
var keepAliveAgent = new http.Agent({ keepAlive: true });
options.agent = keepAliveAgent;
http.request(options, onResponseCallback);
```

#### agent.maxSockets#

默认为`Infinity `。决定了每个源上可以拥有的并发的`socket`的数量。源为`'host:port'`或`'host:port:localAddress'`结合体。

#### agent.maxFreeSockets#

默认为`256`。对于支持HTTP KeepAlive的Agent，这设置了在空闲状态下保持打开的最大`socket`数量。

#### agent.sockets#

这个对象包含了正在被Agent使用de`socket`数组。请不要修改它。

#### agent.freeSockets#

这个对象包含了当HTTP KeepAlive被使用时正在等待的`socket`数组。请不要修改它。

#### agent.requests#

这个对象包含了还没有被分配给`socket`的请求队列。请不要修改它。

#### agent.destroy()#

销毁正在被agent使用的所有`socket`。

通常没有必要这么做。但是，如果你正在使用一个启用了`KeepAlive `的agent，那么最好明确地关闭agent当你知道它不会再被使用时。否则，在服务器关闭它们前`socket`可能被闲置。

#### agent.getName(options)#

通过一个请求选项集合来获取一个独一无二的名字，来决定一个连接是否可被再使用。在http代理中，这返回`host:port:localAddress`。在https代理中，`name`包括了CA，cert，ciphers和`HTTPS/TLS-specific`配置来决定一个`socket`是否能被再使用。

#### http.globalAgent#

所有的http客户端请求使用的默认全局`Agent`实例。

#### Class: http.ClientRequest#

这个对象时被内部创建的，并且通过`http.request()`被返回。它代表了一个正在处理的请求，其头部已经进入了队列。这个头部仍然可以通过`setHeader(name, value)`，`getHeader(name)`和`removeHeader(name)`修改。实际的头部会随着第一个数据块发送，或在关闭连接时发送。

要获得响应对象，请为`response`事件添加一个监听器。`request`对象的`response`事件将会在收到响应头时触发。这个`response`事件的第一个参数是一个`http.IncomingMessage`的实例。

在`response`事件期间，可以给响应对象添加监听器；尤其是监听`data`事件。

如果没有添加`response`事件监听器，那么响应会被完全忽略。但是，如果你添加了`response`事件，那么你必须通过调用`response.read()`，添加`data`事件监听器或调用`.resume()`方法等等，来从响应对象中消耗数据。在数据被消费之前，`end`事件不会触发。如果数据没有被读取，它会消耗内存，最后导致`'process out of memory'`错误。

注意：`node.js`不会检查`Content-Length`和被传输的响应体长度是否相同。

这个请求实现了`Writable`流接口。这是一个包含了以下事件的`EventEmitter`：

#### Event: 'response'#

 - function (response) { }

当这个请求收到一个响应时触发。这个事件只会被触发一次。`response`参数是一个`http.IncomingMessage`实例。

 - __Options:__
  - host: 一个向其发送请求的服务器的域名或IP地址
  - port: 远程服务器的端口
  - socketPath: Unix域`socket`（使用`host:port`或`socketPath`中的一个）

#### Event: 'socket'#

 - function (socket) { }

当一个`socket`被分配给一个请求时触发。

#### Event: 'connect'#

 - function (response, socket, head) { }

每次服务器使用`CONNECT`方法响应一个请求时触发。如果这个事件没有被监听，那么接受`CONNECT`方法的客户端将会关闭它们的连接。

以下是一对客户端/服务器代码，展示如何监听`connect`事件。

```js
var http = require('http');
var net = require('net');
var url = require('url');

// Create an HTTP tunneling proxy
var proxy = http.createServer(function (req, res) {
  res.writeHead(200, {'Content-Type': 'text/plain'});
  res.end('okay');
});
proxy.on('connect', function(req, cltSocket, head) {
  // connect to an origin server
  var srvUrl = url.parse('http://' + req.url);
  var srvSocket = net.connect(srvUrl.port, srvUrl.hostname, function() {
    cltSocket.write('HTTP/1.1 200 Connection Established\r\n' +
                    'Proxy-agent: node.js-Proxy\r\n' +
                    '\r\n');
    srvSocket.write(head);
    srvSocket.pipe(cltSocket);
    cltSocket.pipe(srvSocket);
  });
});

// now that proxy is running
proxy.listen(1337, '127.0.0.1', function() {

  // make a request to a tunneling proxy
  var options = {
    port: 1337,
    hostname: '127.0.0.1',
    method: 'CONNECT',
    path: 'www.google.com:80'
  };

  var req = http.request(options);
  req.end();

  req.on('connect', function(res, socket, head) {
    console.log('got connected!');

    // make a request over an HTTP tunnel
    socket.write('GET / HTTP/1.1\r\n' +
                 'Host: www.google.com:80\r\n' +
                 'Connection: close\r\n' +
                 '\r\n');
    socket.on('data', function(chunk) {
      console.log(chunk.toString());
    });
    socket.on('end', function() {
      proxy.close();
    });
  });
});
```

#### Event: 'upgrade'#

 - function (response, socket, head) { }

Emitted each time a server responds to a request with an upgrade. If this event isn't being listened for, clients receiving an upgrade header will have their connections closed.
每次服务器返回`upgrade`响应给请求时触发。如果这个事件没有被监听，客户端接收一个`upgrade`头时会关闭它们的连接。

以下是一对客户端/服务器代码，展示如何监听`upgrade `事件。

```js
var http = require('http');

// Create an HTTP server
var srv = http.createServer(function (req, res) {
  res.writeHead(200, {'Content-Type': 'text/plain'});
  res.end('okay');
});
srv.on('upgrade', function(req, socket, head) {
  socket.write('HTTP/1.1 101 Web Socket Protocol Handshake\r\n' +
               'Upgrade: WebSocket\r\n' +
               'Connection: Upgrade\r\n' +
               '\r\n');

  socket.pipe(socket); // echo back
});

// now that server is running
srv.listen(1337, '127.0.0.1', function() {

  // make a request
  var options = {
    port: 1337,
    hostname: '127.0.0.1',
    headers: {
      'Connection': 'Upgrade',
      'Upgrade': 'websocket'
    }
  };

  var req = http.request(options);
  req.end();

  req.on('upgrade', function(res, socket, upgradeHead) {
    console.log('got upgraded!');
    socket.end();
    process.exit(0);
  });
});
```

#### Event: 'continue'#

 - function () { }

当服务器发出一个`'100 Continue'`HTTP响应时，通常这是因为请求包含`'Expect: 100-continue'`。这是一个客户端须要发送请求体的指示。

#### Event: 'abort'#

 - function () { }

当请求被客户端中止时触发。这个事件只会在第一次调用`abort()`时触发。

#### request.flushHeaders()#

冲刷请求头。

由于效率原因，`node.js`通常在直到你调用`request.end()`或写入第一个数据块前都会缓冲请求头，然后努力将请求头和数据打包为一个TCP报文。

这通常是你想要的（它节约了一个TCP往返）。但当第一份数据会等待很久才被发送时不是。`request.flushHeaders()`使你能绕过这个优化并且启动请求。

#### request.write(chunk[, encoding][, callback])#

发送一个响应块。当用户想要将请求体流式得发送给服务器时，可以通过调用这个方法多次来办到--在这种情况下，建议在创建请求时使用`['Transfer-Encoding', 'chunked']`头。

`chunk`参数必须是一个`Buffer`或一个字符串。

`encoding`参数是可选的，并且仅当`chunk`是字符串时有效。默认为`'utf8'`。

`callback`参数是可选的，并且当数据块被冲刷时被调用。

#### request.end([data][, encoding][, callback])#

结束发送请求。如果有任何部分的请求体未被发送，这个函数将会将它们冲刷至流中。如果请求是成块的，它会发送终结符`'0\r\n\r\n'`。

如果`data`被指定，那么这与调用`request.write(data, encoding)`后再调用`request.end(callback)`相同。

如果`callback`被指定，那么它将在请求流结束时被调用。

#### request.abort()#

中止请求。

#### request.setTimeout(timeout[, callback])#

一旦一个`socket`被分配给这个请求并且完成连接，`socket.setTimeout()`会被调用。

返回`request`对象。

#### request.setNoDelay([noDelay])#

一旦一个`socket`被分配给这个请求并且完成连接，`socket.setNoDelay()`会被调用。

#### request.setSocketKeepAlive([enable][, initialDelay])#

一旦一个`socket`被分配给这个请求并且完成连接，`socket.setKeepAlive()`会被调用。

#### http.IncomingMessage#

一个`IncomingMessage`对象被`http.Server`或`http.ClientRequest`创建，并且分别被传递给`request`和`response`事件的第一个参数。它被用来取得响应状态，响应头和响应体。

它实现了`Readable`流接口，并且有以下额外的事件，方法和属性。

#### Event: 'close'#

 - function () { }

表明底层连接被关闭。与`end`相同，这个时间每次响应只会触发一次。

#### message.httpVersion#

当向服务器发送请求时，客户端发送的HTTP版本。向客户端发送响应时，服务器响应的HTTP版本。通常是`'1.1'`或`'1.0'`。

另外，`response.httpVersionMajor`是第一个整数，`response.httpVersionMinor`是第二个整数。

#### message.headers#

请求/响应头对象。

只读的头名称和值映射。头名称是小写的，例子：

```js
// Prints something like:
//
// { 'user-agent': 'curl/7.22.0',
//   host: '127.0.0.1:8000',
//   accept: '*/*' }
console.log(request.headers);
```

#### message.rawHeaders#

接受到的原始请求/响应头列表。

注意键和值在同一个列表中，它并非一个元组列表。于是，偶数偏移量为键，奇数偏移量为对应的值。

头名称不是必须小写的，并且重复也没有被合并。

```js
// Prints something like:
//
// [ 'user-agent',
//   'this is invalid because there can be only one',
//   'User-Agent',
//   'curl/7.22.0',
//   'Host',
//   '127.0.0.1:8000',
//   'ACCEPT',
//   '*/*' ]
console.log(request.rawHeaders);
```

#### message.trailers#

请求/响应尾部对象。只在`end`事件中存在。

#### message.rawTrailers#

接受到的原始请求/响应头尾部键值对。只在`end`事件中存在。

#### message.setTimeout(msecs, callback)#

 - msecs Number
 - callback Function

调用`message.connection.setTimeout(msecs, callback)`。

返回`message`。

#### message.method#

仅对从`http.Server`获得的请求有效。

请求方法是字符串。只读。例如：`'GET'`，`'DELETE'`。

#### message.url#

仅对从`http.Server`获得的请求有效。

请求的URL字符串。这仅仅只包含实际HTTP请求中的URL。如果请求是：

```SHELL
GET /status?name=ryan HTTP/1.1\r\n
Accept: text/plain\r\n
\r\n
```

那么`request.url`将是：

```js
'/status?name=ryan'
```

如果你想分块地解释URL。你可以调用`require('url').parse(request.url)`。例子：

```js
iojs> require('url').parse('/status?name=ryan')
{ href: '/status?name=ryan',
  search: '?name=ryan',
  query: 'name=ryan',
  pathname: '/status' }
 ```
 
如果你想从查询字符串中提取参数，你可以使用`require('querystring').parse`函数，或者给`require('url').parse`方法的第二个参数传递`true`，例子：

```js
iojs> require('url').parse('/status?name=ryan', true)
{ href: '/status?name=ryan',
  search: '?name=ryan',
  query: { name: 'ryan' },
  pathname: '/status' }
```

#### message.statusCode#

只对从`http.ClientRequest`到来的响应有效。

3位整数HTTP状态码。如`404`。

#### message.statusMessage#

只对从`http.ClientRequest`到来的响应有效。

HTTP响应状态信息。如`OK`或`Internal Server Error`。

#### message.socket#

与此连接关联的`net.Socket`对象。

通过HTTPS的支持，使用`request.socket.getPeerCertificate()`来获取客户端的身份细节。