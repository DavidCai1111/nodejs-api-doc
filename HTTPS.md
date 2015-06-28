# HTTPS#

### 稳定度: 2 - 稳定

HTTPS是建立在TLS/SSL之上的HTTP协议。在`io.js`中，它被作为单独模块实现。

#### Class: https.Server#

这个类是`tls.Server`的子类，并且和`http.Server`触发相同的事件。更多信息请参阅`http.Server`。

#### server.setTimeout(msecs, callback)#

参阅`http.Server#setTimeout()`。

#### server.timeout#

参阅`http.Server#timeout`。

#### https.createServer(options[, requestListener])#

返回一个新的HTTPS web服务器对象。`options`与`tls.createServer()`中的类似。`requestListener`会被自动添加为`request`事件的监听器。

例子：

```js
// curl -k https://localhost:8000/
var https = require('https');
var fs = require('fs');

var options = {
  key: fs.readFileSync('test/fixtures/keys/agent2-key.pem'),
  cert: fs.readFileSync('test/fixtures/keys/agent2-cert.pem')
};

https.createServer(options, function (req, res) {
  res.writeHead(200);
  res.end("hello world\n");
}).listen(8000);
```

或

```js
var https = require('https');
var fs = require('fs');

var options = {
  pfx: fs.readFileSync('server.pfx')
};

https.createServer(options, function (req, res) {
  res.writeHead(200);
  res.end("hello world\n");
}).listen(8000);
```

#### server.listen(port[, host][, backlog][, callback])#

#### server.listen(path[, callback])#

#### server.listen(handle[, callback])#

详情参阅`http.listen()`。

#### server.close([callback])#

详情参阅`http.close()`。

#### https.request(options, callback)#

向一个安全web服务器发送请求。

`options`可以是一个对象或一个字符串。如果`options`是一个字符串，它会自动被`url.parse()`解析。

所有的`http.request()`选项都是可用的。

例子：

```js
var https = require('https');

var options = {
  hostname: 'encrypted.google.com',
  port: 443,
  path: '/',
  method: 'GET'
};

var req = https.request(options, function(res) {
  console.log("statusCode: ", res.statusCode);
  console.log("headers: ", res.headers);

  res.on('data', function(d) {
    process.stdout.write(d);
  });
});
req.end();

req.on('error', function(e) {
  console.error(e);
});
```

`options`参数有以下选项：

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

以下来自`tls.connect()`的选项也可以被指定。但是，一个`globalAgent`会默默忽略这些。

 - pfx: 证书，SSL所用的私钥和CA证书。默认为`null`。
 - key: SSL所用的私钥。默认为`null`。
 - passphrase: 私钥或pfx的口令字符串。默认为`null`。
 - cert: 所用的公共x509证书。默认为`null`。
 - ca: 一个用来检查远程主机的权威证书或权威证书数组。
 - ciphers: 一个描述要使用或排除的密码的字符串。更多格式信息请查询`http://www.openssl.org/docs/apps/ciphers.html#CIPHER_LIST_FORMAT`。
 - rejectUnauthorized: 如果设置为`true`，服务器证书会使用所给的CA列表验证。验证失败时，一个`error`事件会被触发。验证发生于连接层，在HTTP请求发送之前。默认为`true`。
 - secureProtocol: 所用的SSL方法，如`SSLv3_method`强制使用SSL v3。可用的值取决你的OpenSSL安装和`SSL_METHODS`常量。

要指定这些选项，使用一个自定义的`Agent`。

例子：

```js
var options = {
  hostname: 'encrypted.google.com',
  port: 443,
  path: '/',
  method: 'GET',
  key: fs.readFileSync('test/fixtures/keys/agent2-key.pem'),
  cert: fs.readFileSync('test/fixtures/keys/agent2-cert.pem')
};
options.agent = new https.Agent(options);

var req = https.request(options, function(res) {
  ...
}
```

或不使用`Agent`。

例子：

```js
var options = {
  hostname: 'encrypted.google.com',
  port: 443,
  path: '/',
  method: 'GET',
  key: fs.readFileSync('test/fixtures/keys/agent2-key.pem'),
  cert: fs.readFileSync('test/fixtures/keys/agent2-cert.pem'),
  agent: false
};

var req = https.request(options, function(res) {
  ...
}
```

#### https.get(options, callback)#

类似于`http.get()`，但是使用HTTPS。

`options`可以是一个对象或一个字符串。如果`options`是一个字符串，它会自动被`url.parse()`解析。

例子：

```js
var https = require('https');

https.get('https://encrypted.google.com/', function(res) {
  console.log("statusCode: ", res.statusCode);
  console.log("headers: ", res.headers);

  res.on('data', function(d) {
    process.stdout.write(d);
  });

}).on('error', function(e) {
  console.error(e);
});
```

#### Class: https.Agent#

一个与`http.Agent`类似的HTTPS `Agent`对象。更多信息请参阅`https.request()`。

#### https.globalAgent#

所有HTTPS客户端请求的全局`https.Agent`实例。