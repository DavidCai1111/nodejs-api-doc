# HTTP#

### 稳定度: 2 - 稳定

你必须通过`require('http')`来使用HTTP服务器和客户端。

`io.js`中的HTTP接口被设置来支持许多HTTP协议里原本用起来很困难的特性。特别是大且成块的有编码的消息。这个接口从不缓冲整个请求或响应。用户可以对它们使用流。

HTTP消息头可能是一个类似于以下例子的对象：

```JS
{ 'content-length': '123',
  'content-type': 'text/plain',
  'connection': 'keep-alive',
  'host': 'mysite.com',
  'accept': '*/*' }
```

键是小写的。值没有被修改。

为了全方位的支持所有的HTTP应用。`io.js`的HTTP API是非常底层的。它只处理流以及解释消息。它将消息解释为消息头和消息体，但是不解释实际的消息头和消息体。

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

注意，`Content-Length`是以字节计，而不是以字符计。上面例子能正常运行时因为字符串`'hello world'`仅包含单字节字符。如果响应体包含了多字节编码的字符，那么必须通过指定的编码来调用`Buffer.byteLength()`来确定字节数。并且`io.js`不会检查`Content-Length`与响应体的字节数是否相等。

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

If this method is called and response.writeHead() has not been called, it will switch to implicit header mode and flush the implicit headers.

This sends a chunk of the response body. This method may be called multiple times to provide successive parts of the body.

chunk can be a string or a buffer. If chunk is a string, the second parameter specifies how to encode it into a byte stream. By default the encoding is 'utf8'. The last parameter callback will be called when this chunk of data is flushed.

Note: This is the raw HTTP body and has nothing to do with higher-level multi-part body encodings that may be used.

The first time response.write() is called, it will send the buffered header information and the first body to the client. The second time response.write() is called, io.js assumes you're going to be streaming data, and sends that separately. That is, the response is buffered up to the first chunk of body.

Returns true if the entire data was flushed successfully to the kernel buffer. Returns false if all or part of the data was queued in user memory. 'drain' will be emitted when the buffer is free again.

#### response.addTrailers(headers)#

This method adds HTTP trailing headers (a header but at the end of the message) to the response.

Trailers will only be emitted if chunked encoding is used for the response; if it is not (e.g., if the request was HTTP/1.0), they will be silently discarded.

Note that HTTP requires the Trailer header to be sent if you intend to emit trailers, with a list of the header fields in its value. E.g.,

response.writeHead(200, { 'Content-Type': 'text/plain',
                          'Trailer': 'Content-MD5' });
response.write(fileData);
response.addTrailers({'Content-MD5': "7895bf4b8828b55ceaf47747b4bca667"});
response.end();
response.end([data][, encoding][, callback])#

This method signals to the server that all of the response headers and body have been sent; that server should consider this message complete. The method, response.end(), MUST be called on each response.

If data is specified, it is equivalent to calling response.write(data, encoding) followed by response.end(callback).

If callback is specified, it will be called when the response stream is finished.

#### http.request(options[, callback])#
io.js maintains several connections per server to make HTTP requests. This function allows one to transparently issue requests.

options can be an object or a string. If options is a string, it is automatically parsed with url.parse().

Options:

host: A domain name or IP address of the server to issue the request to. Defaults to 'localhost'.
hostname: Alias for host. To support url.parse() hostname is preferred over host.
family: IP address family to use when resolving host and hostname. Valid values are 4 or 6. When unspecified, both IP v4 and v6 will be used.
port: Port of remote server. Defaults to 80.
localAddress: Local interface to bind for network connections.
socketPath: Unix Domain Socket (use one of host:port or socketPath).
method: A string specifying the HTTP request method. Defaults to 'GET'.
path: Request path. Defaults to '/'. Should include query string if any. E.G. '/index.html?page=12'. An exception is thrown when the request path contains illegal characters. Currently, only spaces are rejected but that may change in the future.
headers: An object containing request headers.
auth: Basic authentication i.e. 'user:password' to compute an Authorization header.
agent: Controls Agent behavior. When an Agent is used request will default to Connection: keep-alive. Possible values:
undefined (default): use globalAgent for this host and port.
Agent object: explicitly use the passed in Agent.
false: opts out of connection pooling with an Agent, defaults request to Connection: close.
The optional callback parameter will be added as a one time listener for the 'response' event.

http.request() returns an instance of the http.ClientRequest class. The ClientRequest instance is a writable stream. If one needs to upload a file with a POST request, then write to the ClientRequest object.

Example:

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
Note that in the example req.end() was called. With http.request() one must always call req.end() to signify that you're done with the request - even if there is no data being written to the request body.

If any error is encountered during the request (be that with DNS resolution, TCP level errors, or actual HTTP parse errors) an 'error' event is emitted on the returned request object.

There are a few special headers that should be noted.

Sending a 'Connection: keep-alive' will notify io.js that the connection to the server should be persisted until the next request.

Sending a 'Content-length' header will disable the default chunked encoding.

Sending an 'Expect' header will immediately send the request headers. Usually, when sending 'Expect: 100-continue', you should both set a timeout and listen for the continue event. See RFC2616 Section 8.2.3 for more information.

Sending an Authorization header will override using the auth option to compute basic authentication.

#### http.get(options[, callback])#
Since most requests are GET requests without bodies, io.js provides this convenience method. The only difference between this method and http.request() is that it sets the method to GET and calls req.end() automatically.

Example:

http.get("http://www.google.com/index.html", function(res) {
  console.log("Got response: " + res.statusCode);
}).on('error', function(e) {
  console.log("Got error: " + e.message);
});
#### Class: http.Agent#
The HTTP Agent is used for pooling sockets used in HTTP client requests.

The HTTP Agent also defaults client requests to using Connection:keep-alive. If no pending HTTP requests are waiting on a socket to become free the socket is closed. This means that io.js's pool has the benefit of keep-alive when under load but still does not require developers to manually close the HTTP clients using KeepAlive.

If you opt into using HTTP KeepAlive, you can create an Agent object with that flag set to true. (See the constructor options below.) Then, the Agent will keep unused sockets in a pool for later use. They will be explicitly marked so as to not keep the io.js process running. However, it is still a good idea to explicitly destroy() KeepAlive agents when they are no longer in use, so that the Sockets will be shut down.

Sockets are removed from the agent's pool when the socket emits either a "close" event or a special "agentRemove" event. This means that if you intend to keep one HTTP request open for a long time and don't want it to stay in the pool you can do something along the lines of:

http.get(options, function(res) {
  // Do stuff
}).on("socket", function (socket) {
  socket.emit("agentRemove");
});
Alternatively, you could just opt out of pooling entirely using agent:false:

http.get({
  hostname: 'localhost',
  port: 80,
  path: '/',
  agent: false  // create a new agent just for this one request
}, function (res) {
  // Do stuff with response
})
new Agent([options])#

options Object Set of configurable options to set on the agent. Can have the following fields:
keepAlive Boolean Keep sockets around in a pool to be used by other requests in the future. Default = false
keepAliveMsecs Integer When using HTTP KeepAlive, how often to send TCP KeepAlive packets over sockets being kept alive. Default = 1000. Only relevant if keepAlive is set to true.
maxSockets Number Maximum number of sockets to allow per host. Default = Infinity.
maxFreeSockets Number Maximum number of sockets to leave open in a free state. Only relevant if keepAlive is set to true. Default = 256.
The default http.globalAgent that is used by http.request has all of these values set to their respective defaults.

To configure any of them, you must create your own Agent object.

var http = require('http');
var keepAliveAgent = new http.Agent({ keepAlive: true });
options.agent = keepAliveAgent;
http.request(options, onResponseCallback);
agent.maxSockets#

By default set to Infinity. Determines how many concurrent sockets the agent can have open per origin. Origin is either a 'host:port' or 'host:port:localAddress' combination.

#### agent.maxFreeSockets#

By default set to 256. For Agents supporting HTTP KeepAlive, this sets the maximum number of sockets that will be left open in the free state.

#### agent.sockets#

An object which contains arrays of sockets currently in use by the Agent. Do not modify.

#### agent.freeSockets#

An object which contains arrays of sockets currently awaiting use by the Agent when HTTP KeepAlive is used. Do not modify.

#### agent.requests#

An object which contains queues of requests that have not yet been assigned to sockets. Do not modify.

#### agent.destroy()#

Destroy any sockets that are currently in use by the agent.

It is usually not necessary to do this. However, if you are using an agent with KeepAlive enabled, then it is best to explicitly shut down the agent when you know that it will no longer be used. Otherwise, sockets may hang open for quite a long time before the server terminates them.

#### agent.getName(options)#

Get a unique name for a set of request options, to determine whether a connection can be reused. In the http agent, this returns host:port:localAddress. In the https agent, the name includes the CA, cert, ciphers, and other HTTPS/TLS-specific options that determine socket reusability.

#### http.globalAgent#
Global instance of Agent which is used as the default for all http client requests.

#### Class: http.ClientRequest#
This object is created internally and returned from http.request(). It represents an in-progress request whose header has already been queued. The header is still mutable using the setHeader(name, value), getHeader(name), removeHeader(name) API. The actual header will be sent along with the first data chunk or when closing the connection.

To get the response, add a listener for 'response' to the request object. 'response' will be emitted from the request object when the response headers have been received. The 'response' event is executed with one argument which is an instance of http.IncomingMessage.

During the 'response' event, one can add listeners to the response object; particularly to listen for the 'data' event.

If no 'response' handler is added, then the response will be entirely discarded. However, if you add a 'response' event handler, then you must consume the data from the response object, either by calling response.read() whenever there is a 'readable' event, or by adding a 'data' handler, or by calling the .resume() method. Until the data is consumed, the 'end' event will not fire. Also, until the data is read it will consume memory that can eventually lead to a 'process out of memory' error.

Note: io.js does not check whether Content-Length and the length of the body which has been transmitted are equal or not.

The request implements the Writable Stream interface. This is an EventEmitter with the following events:

#### Event: 'response'#

function (response) { }

Emitted when a response is received to this request. This event is emitted only once. The response argument will be an instance of http.IncomingMessage.

Options:

host: A domain name or IP address of the server to issue the request to.
port: Port of remote server.
socketPath: Unix Domain Socket (use one of host:port or socketPath)
Event: 'socket'#

function (socket) { }

Emitted after a socket is assigned to this request.

#### Event: 'connect'#

function (response, socket, head) { }

Emitted each time a server responds to a request with a CONNECT method. If this event isn't being listened for, clients receiving a CONNECT method will have their connections closed.

A client server pair that show you how to listen for the connect event.

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
                    'Proxy-agent: io.js-Proxy\r\n' +
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
#### Event: 'upgrade'#

function (response, socket, head) { }

Emitted each time a server responds to a request with an upgrade. If this event isn't being listened for, clients receiving an upgrade header will have their connections closed.

A client server pair that show you how to listen for the upgrade event.

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
#### Event: 'continue'#

function () { }

Emitted when the server sends a '100 Continue' HTTP response, usually because the request contained 'Expect: 100-continue'. This is an instruction that the client should send the request body.

#### Event: 'abort'#

function () { }

Emitted when the request has been aborted by the client. This event is only emitted on the first call to abort().

#### request.flushHeaders()#

Flush the request headers.

For efficiency reasons, io.js normally buffers the request headers until you call request.end() or write the first chunk of request data. It then tries hard to pack the request headers and data into a single TCP packet.

That's usually what you want (it saves a TCP round-trip) but not when the first data isn't sent until possibly much later. request.flushHeaders() lets you bypass the optimization and kickstart the request.

#### request.write(chunk[, encoding][, callback])#

Sends a chunk of the body. By calling this method many times, the user can stream a request body to a server--in that case it is suggested to use the ['Transfer-Encoding', 'chunked'] header line when creating the request.

The chunk argument should be a Buffer or a string.

The encoding argument is optional and only applies when chunk is a string. Defaults to 'utf8'.

The callback argument is optional and will be called when this chunk of data is flushed.

#### request.end([data][, encoding][, callback])#

Finishes sending the request. If any parts of the body are unsent, it will flush them to the stream. If the request is chunked, this will send the terminating '0\r\n\r\n'.

If data is specified, it is equivalent to calling request.write(data, encoding) followed by request.end(callback).

If callback is specified, it will be called when the request stream is finished.

#### request.abort()#

Aborts a request. (New since v0.3.8.)

#### request.setTimeout(timeout[, callback])#

Once a socket is assigned to this request and is connected socket.setTimeout() will be called.

Returns request.

#### request.setNoDelay([noDelay])#

Once a socket is assigned to this request and is connected socket.setNoDelay() will be called.

#### request.setSocketKeepAlive([enable][, initialDelay])#

Once a socket is assigned to this request and is connected socket.setKeepAlive() will be called.

#### http.IncomingMessage#
An IncomingMessage object is created by http.Server or http.ClientRequest and passed as the first argument to the 'request' and 'response' event respectively. It may be used to access response status, headers and data.

It implements the Readable Stream interface, as well as the following additional events, methods, and properties.

#### Event: 'close'#

function () { }

Indicates that the underlying connection was closed. Just like 'end', this event occurs only once per response.

#### message.httpVersion#

In case of server request, the HTTP version sent by the client. In the case of client response, the HTTP version of the connected-to server. Probably either '1.1' or '1.0'.

Also response.httpVersionMajor is the first integer and response.httpVersionMinor is the second.

#### message.headers#

The request/response headers object.

Read only map of header names and values. Header names are lower-cased. Example:

// Prints something like:
//
// { 'user-agent': 'curl/7.22.0',
//   host: '127.0.0.1:8000',
//   accept: '*/*' }
console.log(request.headers);
#### message.rawHeaders#

The raw request/response headers list exactly as they were received.

Note that the keys and values are in the same list. It is not a list of tuples. So, the even-numbered offsets are key values, and the odd-numbered offsets are the associated values.

Header names are not lowercased, and duplicates are not merged.

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
#### message.trailers#

The request/response trailers object. Only populated at the 'end' event.

#### message.rawTrailers#

The raw request/response trailer keys and values exactly as they were received. Only populated at the 'end' event.

#### message.setTimeout(msecs, callback)#

msecs Number
callback Function
Calls message.connection.setTimeout(msecs, callback).

Returns message.

#### message.method#

Only valid for request obtained from http.Server.

The request method as a string. Read only. Example: 'GET', 'DELETE'.

#### message.url#

Only valid for request obtained from http.Server.

Request URL string. This contains only the URL that is present in the actual HTTP request. If the request is:

GET /status?name=ryan HTTP/1.1\r\n
Accept: text/plain\r\n
\r\n
Then request.url will be:

'/status?name=ryan'
If you would like to parse the URL into its parts, you can use require('url').parse(request.url). Example:

iojs> require('url').parse('/status?name=ryan')
{ href: '/status?name=ryan',
  search: '?name=ryan',
  query: 'name=ryan',
  pathname: '/status' }
If you would like to extract the params from the query string, you can use the require('querystring').parse function, or pass true as the second argument to require('url').parse. Example:

iojs> require('url').parse('/status?name=ryan', true)
{ href: '/status?name=ryan',
  search: '?name=ryan',
  query: { name: 'ryan' },
  pathname: '/status' }
#### message.statusCode#

Only valid for response obtained from http.ClientRequest.

The 3-digit HTTP response status code. E.G. 404.

#### message.statusMessage#

Only valid for response obtained from http.ClientRequest.

The HTTP response status message (reason phrase). E.G. OK or Internal Server Error.

#### message.socket#

The net.Socket object associated with the connection.

With HTTPS support, use request.socket.getPeerCertificate() to obtain the client's authentication details.