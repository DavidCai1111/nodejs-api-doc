# TLS (SSL)#

### 稳定度: 2 - 稳定

通过`require('tls')`来使用这个模块。

`tls`模块使用OpenSSL来提供传输层的安全 和/或 安全`socket`层：已加密的流通信。

TLS/SSL是一种公/私钥架构。每个客户端和每个服务器都必须有一个私钥。一个私钥通过像如下的方式创建：

```SHELL
openssl genrsa -out ryans-key.pem 2048
```

所有的服务器和部分的客户端需要一个证书。证书是被CA签名或自签名的公钥。获取一个证书第一步是创建一个“证书签署请求（Certificate Signing Request）”（CSR）文件。通过：

```SHELL
openssl req -new -sha256 -key ryans-key.pem -out ryans-csr.pem
```

要通过CSR创建一个自签名证书，通过：

```js
openssl x509 -req -in ryans-csr.pem -signkey ryans-key.pem -out ryans-cert.pem
```

另外，你也可以把CSR交给一个CA请求签名。

为了完全向前保密（PFS），需要产生一个 迪菲-赫尔曼 参数：

```SHELL
openssl dhparam -outform PEM -out dhparam.pem 2048
```

创建`.pfx`或`.p12`，通过：

```SHELL
openssl pkcs12 -export -in agent5-cert.pem -inkey agent5-key.pem \
    -certfile ca-cert.pem -out agent5.pfx
```

 - in: 证书
 - inkey: 私钥
 - certfile: 将所有`CA certs`串联在一个文件中，就像`cat ca1-cert.pem ca2-cert.pem > ca-cert.pem`。

#### 客户端发起的重新协商攻击的减缓

TLS协议让客户端可以重新协商某些部分的TLS会话。不幸的是，会话重协商需要不相称的服务器端资源，这它可能成为潜在的DOS攻击。

为了减缓这种情况，重新协商被限制在了每10分钟最多3次。当超过阀值时，`tls.TLSSocket`会触发一个错误。阀值是可以调整的：

 - tls.CLIENT_RENEG_LIMIT: 重新协商限制，默认为`3`。

 - tls.CLIENT_RENEG_WINDOW: 重新协商窗口（秒），默认为10分钟。

除非你知道你在做什么，否则不要改变默认值。

为了测试你的服务器，使用`openssl s_client -connect address:port`来连接它，然后键入`R<CR>`（字母`R`加回车）多次。

#### NPN 和 SNI

`NPN`（下个协议协商）和SNI（服务器名称指示）都是TLS握手拓展，它们允许你：

 - NPN - 通过多个协议（HTTP，SPDY）使用一个TLS服务器。
 - SNI - 通过多个有不同的SSL证书的主机名来使用一个TLS服务器。

#### 完全向前保密

术语“向前保密”或“完全向前保密”描述了一个密钥-协商（如密钥-交换）方法的特性。事实上，它意味着，甚至是当（你的）服务器的私钥被窃取了，窃取者也只能在他成功获得所有会话产生的密钥对时，才能解码信息。

它通过在每次握手中（而不是所有的会话都是同样的密钥）随机地产生用于密钥-协商的密钥对来实现。实现了这个技术的方法被称作“ephemeral”。

目前有两种普遍的方法来实现完全向前保密：

 - DHE - 一个 迪菲-赫尔曼 密钥-协商 协议的`ephemeral`版本。
 - ECDHE - 一个椭圆曲线 迪菲-赫尔曼 密钥-协商 协议的`ephemeral`版本。

`ephemeral`方法可能有一些性能问题，因为密钥的生成是昂贵的。

#### tls.getCiphers()#

返回支持的SSL加密器的名字数组。

例子：

```js
var ciphers = tls.getCiphers();
console.log(ciphers); // ['AES128-SHA', 'AES256-SHA', ...]
```

#### tls.createServer(options[, secureConnectionListener])#

创一个新的`tls.Server`实例。`connectionListener`参数被自动添加为`secureConnection`事件的监听器。`options`参数可以有以下属性：

 - pfx: 一个包含`PFX`或`PKCS12`格式的私钥，加密凭证和CA证书的字符串或`buffer`。

 - key: 一个带着`PEM`加密私钥的字符串（可以是密钥数组）（必选）。

 - passphrase: 一个私钥或`pfx`密码字符串。

 - cert: 一个包含了`PEM`格式的服务器证书密钥的字符串或`buffer`（可以是`cert`数组）（必选）。

 - ca: 一个`PEM`格式的受信任证书的字符串或`buffer`数组。如果它被忽略，将使用一些众所周知的“根”CA，像`VeriSign`。这些被用来授权连接。

 - crl : 一个`PEM`编码的证书撤销列表（Certificate Revocation List）字符串或字符串列表。

 - ciphers: 一个描述要使用或排除的加密器的字符串，通过`:`分割。默认的加密器套件是：

```SHELL
ECDHE-RSA-AES128-GCM-SHA256:
ECDHE-ECDSA-AES128-GCM-SHA256:
ECDHE-RSA-AES256-GCM-SHA384:
ECDHE-ECDSA-AES256-GCM-SHA384:
DHE-RSA-AES128-GCM-SHA256:
ECDHE-RSA-AES128-SHA256:
DHE-RSA-AES128-SHA256:
ECDHE-RSA-AES256-SHA384:
DHE-RSA-AES256-SHA384:
ECDHE-RSA-AES256-SHA256:
DHE-RSA-AES256-SHA256:
HIGH:
!aNULL:
!eNULL:
!EXPORT:
!DES:
!RC4:
!MD5:
!PSK:
!SRP:
!CAMELLIA
```

默认的加密器套件更倾向于`Chrome's 'modern cryptography' setting`的GCM加密器，也倾向于PFC的ECDHE和DHE加密器，它们提供了一些向后兼容性。

鉴于`specific attacks affecting larger AES key sizes`，所以更倾向于使用128位的AES而不是192和256位的AES。

旧的依赖于不安全的和弃用的RC4或基于DES的加密器（像IE6）的客户端将不能完成默认配置下的握手。如果你必须支持这些客户端，`TLS推荐规范`可能提供了一个兼容的加密器套件。更多格式细节，参阅`OpenSSL cipher list format documentation`。

 - ecdhCurve: 一个描述用于`ECDH`密钥协商的已命名的椭圆的字符串，如果要禁用`ECDH`，就设置为`false`。

默认值为`prime256v1`（NIST P-256）。使用`crypto.getCurves()`来获取一个可用的椭圆列表。在最近的发行版中，运行`openssl ecparam -list_curves`命令也会展示所有可用的椭圆的名字和描述。

 - dhparam: 一个包含了迪菲-赫尔曼参数的字符串或`buffer`，要求有完全向前保密。使用`openssl dhparam`来创建它。它的密钥长度需要大于等于1024字节，否则会抛出一个错误。强力推荐使用2048或更多位，来获取更高的安全性。如果参数被忽略或不合法，它会被默默丢弃并且`DHE`加密器将不可用。

 - handshakeTimeout: 当SSL/TLS握手在这个指定的毫秒数后没有完成时，终止这个链接。默认为120秒。

当握手超时时，`tls.Server`会触发一个`clientError`事件。

 - honorCipherOrder : 选择一个加密器时，使用使用服务器的首选项而不是客户端的首选项。默认为`true`。

 - requestCert: 如果设置为`true`，服务器将会向连接的客户端请求一个证书，并且试图验证这个证书。默认为`true`。

 - rejectUnauthorized: 如果设置为`true`，服务器会拒绝所有没有在提供的CA列表中被授权的客户端。只有在`requestCert`为`true`时这个选项才有效。默认为`false`。

 - NPNProtocols: 一个可用的`NPN`协议的字符串或数组（协议应该由它们的优先级被排序）。

 - SNICallback(servername, cb): 当客户端支持`SNI TLS`扩展时，这个函数会被调用。这个函数会被传递两个参数：servername和cb。`SNICallback`必须执行`cb(null, ctx)`,`ctx`是一个`SecureContext`实例（你可以使用`tls.createSecureContext(...)`来获取合适的`SecureContext`）。如果`SNICallback`没有被提供 - 默认的有高层次API的回调函数会被使用（参阅下文）。

 - sessionTimeout: 一个指定在TLS会话标识符和TLS会话门票（tickets）被服务器创建后的超时时间。更多详情参阅`SSL_CTX_set_timeout`。

 - ticketKeys: 一个由16字节前缀，16字节hmac密钥，16字节AEC密钥组成的48字节`buffer`。你可以使用它在不同的`tls`服务器实例上接受`tls`会话门票。

注意：会在`cluster`模块工作进程间自动共享。

 - sessionIdContext: 一个包含了会话恢复标识符的字符串。如果`requestCert`为`true`，默认值是通过命令行生成的MD5哈希值。否则，就将不提供默认值。

 - secureProtocol: 将要使用的SSL方法，举例，`SSLv3_method`将强制使用SSL v3。可用的值取决于OpenSSL的安装和`SSL_METHODS`常量中被定义的值。

下面是一个简单应答服务器的例子：

```js
var tls = require('tls');
var fs = require('fs');

var options = {
  key: fs.readFileSync('server-key.pem'),
  cert: fs.readFileSync('server-cert.pem'),

  // This is necessary only if using the client certificate authentication.
  requestCert: true,

  // This is necessary only if the client uses the self-signed certificate.
  ca: [ fs.readFileSync('client-cert.pem') ]
};

var server = tls.createServer(options, function(socket) {
  console.log('server connected',
              socket.authorized ? 'authorized' : 'unauthorized');
  socket.write("welcome!\n");
  socket.setEncoding('utf8');
  socket.pipe(socket);
});
server.listen(8000, function() {
  console.log('server bound');
});
```

或

```js
var tls = require('tls');
var fs = require('fs');

var options = {
  pfx: fs.readFileSync('server.pfx'),

  // This is necessary only if using the client certificate authentication.
  requestCert: true,

};

var server = tls.createServer(options, function(socket) {
  console.log('server connected',
              socket.authorized ? 'authorized' : 'unauthorized');
  socket.write("welcome!\n");
  socket.setEncoding('utf8');
  socket.pipe(socket);
});
server.listen(8000, function() {
  console.log('server bound');
});
```

你可以通过`openssl s_client`来连接服务器：

```SHELL
openssl s_client -connect 127.0.0.1:8000
```

#### tls.connect(options[, callback])#
#### tls.connect(port[, host][, options][, callback])#

根据给定的 端口和主机（旧API）或 `options.port`和`options.host` 创建一个新的客户端连接。如果忽略了主机，默认为`localhost`。`options`可是一个含有以下属性的对象：

 - host: 客户端应该连接到的主机。

 - port: 客户端应该连接到的端口。

 - socket: 根据给定的`socket`的来建立安全连接，而不是创建一个新的`socket`。如果这个选项被指定，`host`和`port`会被忽略。

 - path: 创建到`path`的unix `socket`连接。如果这个选项被指定，`host`和`port`会被忽略。

 - pfx: 一个`PFX`或`PKCS12`格式的包含了私钥，证书和CA证书的字符串或`buffer`。

 - key: 一个`PEM`格式的包含了客户端私钥的字符串或`buffer`（可以是密钥的数组）。

 - passphrase: 私钥或`pfx`的密码字符串。

 - cert: 一个`PEM`格式的包含了证书密钥的字符串或`buffer`（可以是密钥的数组）。

 - ca: 一个`PEM`格式的受信任证书的字符串或`buffer`数组。如果它被忽略，将使用一些众所周知的CA，像`VeriSign`。这些被用来授权连接。

 - ciphers: 一个描述了要使用或排除的加密器，由`:`分割。使用的默认加密器套件与`tls.createServer`使用的一样。

 - rejectUnauthorized: 若被设置为`true`，会根据提供的CA列表来验证服务器证书。当验证失败时，会触发`error`事件；`err.code`包含了一个OpenSSL错误码。默认为`true`。

 - NPNProtocols: 包含支持的NPN协议的字符串或`buffer`数组。`buffer`必须有以下格式：`0x05hello0x05world`，第一个字节是下一个协议名的长度（传递数组会更简单：`['hello', 'world']`）。

 - servername: `SNI` TLS 扩展的服务器名。

 - checkServerIdentity(servername, cert): 为根据证书的服务器主机名检查提供了覆盖。必须在验证失败时返回一个错误，验证通过时返回`undefined`。

 - secureProtocol: 将要使用的SSL方法，举例，`SSLv3_method`将强制使用SSL v3。可用的值取决于OpenSSL的安装和`SSL_METHODS`常量中被定义的值。

 - session: 一个`Buffer`实例，包含了TLS会话。

`callback`参数会被自动添加为`secureConnect`事件的监听器。

`tls.connect()`返回一个`tls.TLSSocket`对象。

以下是一个上述应答服务器的客户端的例子：

```js
var tls = require('tls');
var fs = require('fs');

var options = {
  // These are necessary only if using the client certificate authentication
  key: fs.readFileSync('client-key.pem'),
  cert: fs.readFileSync('client-cert.pem'),

  // This is necessary only if the server uses the self-signed certificate
  ca: [ fs.readFileSync('server-cert.pem') ]
};

var socket = tls.connect(8000, options, function() {
  console.log('client connected',
              socket.authorized ? 'authorized' : 'unauthorized');
  process.stdin.pipe(socket);
  process.stdin.resume();
});
socket.setEncoding('utf8');
socket.on('data', function(data) {
  console.log(data);
});
socket.on('end', function() {
  server.close();
});
```

或

```js
var tls = require('tls');
var fs = require('fs');

var options = {
  pfx: fs.readFileSync('client.pfx')
};

var socket = tls.connect(8000, options, function() {
  console.log('client connected',
              socket.authorized ? 'authorized' : 'unauthorized');
  process.stdin.pipe(socket);
  process.stdin.resume();
});
socket.setEncoding('utf8');
socket.on('data', function(data) {
  console.log(data);
});
socket.on('end', function() {
  server.close();
});
```

#### Class: tls.TLSSocket#

`net.Socket`实例的包装，替换了内部`socket`的 读/写例程，来提供透明的对 传入/传出数据 的 加密/解密。

#### new tls.TLSSocket(socket, options)#

根据已存在的TCP`socket`，构造一个新的`TLSSocket`对象。

`socket`是一个`net.Socket`实例。

`options`是一个可能包含以下属性的对象：

 - secureContext: 一个可选的通过`tls.createSecureContext( ... )`得到的TLS内容对象。

 - isServer: 如果为`true`，TLS `socket`将会在服务器模式（server-mode）下被初始化。

 - server: 一个可选的`net.Server`实例。

 - requestCert: 可选，参阅`tls.createSecurePair`。

 - rejectUnauthorized: 可选，参阅`tls.createSecurePair`。

 - NPNProtocols: 可选，参阅`tls.createServer`。

 - SNICallback: 可选，参阅`tls.createServer`。

 - session: 可选，一个`Buffer`实例，包含了TLS会话。

 - requestOCSP: 可选，如果为`true`，`OCSP`状态请求扩展将会被添加到客户端 hello，并且`OCSPResponse`事件将会在建立安全通信前，于`socket`上触发。

#### tls.createSecureContext(details)#

创建一个证书对象，`details`有可选的以下值：

 - pfx : 一个含有`PFX`或`PKCS12`编码的私钥，证书和CA证书的字符串或`buffer`。
 - key : 一个含有`PEM`编码的私钥的字符串。
 - passphrase : 一个私钥或`pfx`密码字符串。
 - cert : 一个含有`PEM`加密证书的字符串。
 - ca : 一个用来信任的`PEM`加密CA证书的字符串或字符串列表。
 - crl : 一个`PEM`加密`CRL`的字符串或字符串列表。
 - ciphers:  一个描述需要使用或排除的加密器的字符串。更多加密器的格式细节参阅`http://www.openssl.org/docs/apps/ciphers.html#CIPHER_LIST_FORMAT`。
 - honorCipherOrder : 选择一个加密器时，使用使用服务器的首选项而不是客户端的首选项。默认为`true`。更多细节参阅`tls`模块文档。

如果没有指定`ca`，那么`io.js`将会使用`http://mxr.mozilla.org/mozilla/source/security/nss/lib/ckfw/builtins/certdata.txt`提供的默认公共可信任CA列表。

#### tls.createSecurePair([context][, isServer][, requestCert][, rejectUnauthorized])#

根据两个流，创建一个新的安全对（secure pair）对象，一个是用来读/写加密数据，另一个是用来读/写明文数据。通常加密的数据是从加密数据流被导流而来，明文数据被用来作为初始加密流的一个替代。

 - credentials: 一个通过`tls.createSecureContext( ... )`得到的安全内容对象。

 - isServer: 一个表明了 是否这个`tls`连接应被作为一个服务器或一个客户端打开 的布尔值。

 - requestCert: 一个表明了 是否服务器应该向连接的客户端请求证书 的布尔值。只应用于服务器连接。

 - rejectUnauthorized: 一个表明了 是否服务器应该拒绝包含不可用证书的客户端 的布尔值。只应用于启用了`requestCert`的服务器。

`tls.createSecurePair()`返回一个带有`cleartext`和 `encrypted`流 属性的对象。

注意：`cleartext`和`tls.TLSSocket`有相同的API。

#### Class: SecurePair#

由`tls.createSecurePair`返回。

#### Event: 'secure'#

当`SecurePair`成功建立一个安全连接时，`SecurePair`会触发这个事件

与检查服务器的`secureConnection`事件相似，`pair.cleartext.authorized`必须被检查，来确认证书是否使用了合适的授权。

#### Class: tls.Server#

这是一个`net.Server`的子类，并且与其有相同的方法。除了只接受源TCP连接，这个类还接受通过TLS或SSL加密的数据。

#### Event: 'secureConnection'#

 - function (tlsSocket) {}

当一个新连接被成功握手后，这个事件会被触发。参数是一个`tls.TLSSocket`实例。它拥有所有普通流拥有的事件和方法。

`socket.authorized`是一个表明了 客户端是否通过提供的服务器CA来进行了认证 的布尔值。如果`socket.authorized`为`false`，那么`socket.authorizationError`将被设置用来描述授权失败的原因。一个不明显的但是值得提出的点：依靠TLS服务器的设定，未授权的连接可能会被接受。`socket.npnProtocol`是一个包含了被选择的NPN协议的字符串。`socket.servernam`是一个包含了通过SNI请求的服务器名的字符串。

#### Event: 'clientError'#

 - function (exception, tlsSocket) { }

当安全连接被建立之前，服务器触发了一个`error`事件 - 它会被转发到这里。

`tlsSocket`是错误来自的`tls.TLSSocket`。

#### Event: 'newSession'#

 - function (sessionId, sessionData, callback) { }

在TLS会话创建时触发。可能会被用来在外部存储会话。`callback`必须最终被执行，否则安全连接将不会收到数据。

注意：这个事件监听器只会影响到它被添加之后建立的连接。

#### Event: 'resumeSession'#

 - function (sessionId, callback) { }

当客户端想要恢复先前的TLS会话时触发。事件监听器可能会在外部通过`sessionId`来寻找会话，并且在结束后调用`callback(null, sessionData)`。如果会话不能被恢复（例如没有找到），可能会调用`callback(null, null)`。调用`callback(err)`会关闭将要到来的连接并且销毁`socket`。

注意：这个事件监听器只会影响到它被添加之后建立的连接。

#### Event: 'OCSPRequest'#

 - function (certificate, issuer, callback) { }

当客户端发送一个证书状态请求时触发。你可以解释服务器当前的证书来获取OCSP url和证书id，并且在获取了OCSP响应后执行`callback(null, resp)`，`resp`是一个`Buffer`实例。`certificate`和`issuer`都是一个`Buffer`，即主键和发起人证书的DER代表（DER-representations）。它们可以被用来获取OCSP证书id 和 OCSP末端url。

另外，`callback(null, null)`可以被调用，意味着没有OCSP响应。

调用`callback(err)`，将会导致调用`socket.destroy(err)`。

典型的流程：

 1. 客户端连接到服务器，然后发送一个`OCSPRequest`给它（通过`ClientHello`中扩展的状态信息）。
 2. 服务器接受请求，然后执行`OCSPRequest`事件监听器（如果存在）。
 3. 服务器通过证书或发起人抓取OCSP url，然后向CA发起一个OCSP请求。
 4. 服务器从CA收到一个`OCSPResponse`，然后通过回调函数的参数将其返回给客户端。
 5. 客户端验证响应，然后销毁`socket`或者进行握手。

注意：`issuer`可以是`null`，如果证书是自签名的或`issuer`不在根证书列表之内（你可以通过`ca`参数提供一个`issuer`）。

注意：这个事件监听器只会影响到它被添加之后建立的连接。

注意：你可能想要使用一些如`asn1.js`的`npm`模块来解释证书。

#### server.listen(port[, hostname][, callback])#

从指定的端口和主机名接收连接。如果`hostname`被忽略，服务器会在当IPv6可用时，接受任意IPv6地址（`::`）上的连接，否则为任意IPv4（`0.0.0.0`）上的。将`port`设置为`0`则会赋予其一个随机端口。

这个函数是异步的。最后一个参数`callback`会在服务器被绑定后执行。

更多信息请参阅`net.Server`。

#### server.close([callback])#

阻止服务器继续接收新连接。这个函数是异步的，当服务器触发一个`close`事件时，服务器将最终被关闭。可选的，你可以传递一个回调函数来监听`close`事件。

#### server.address()#

返回绑定的地址，服务器地址的协议族名和端口通过操作系统报告。更多信息请参阅`net.Server.address()`。

#### server.addContext(hostname, context)#

添加安全内容，它将会在如果客户端请求的SNI主机名被传递的主机名匹配（可以使用通配符）时使用。`context`可以包含密钥，证书，CA 和/或 其他任何`tls.createSecureContext`的`options`参数的属性。

#### server.maxConnections#

当服务器连接数变多时，设置这个值来拒绝连接。

#### server.connections#

服务器上的当前连接数。

#### Class: CryptoStream#

> 稳定度: 0 - 弃用。 使用`tls.TLSSocket `替代。

这是一个加密流。

#### cryptoStream.bytesWritten#

一个底层`socket`的`bytesWritten`存取器的代理，它会返回写入`socket`的总字节数，包括TLS开销。

#### Class: tls.TLSSocket#

这是一个`net.Socket`的包装，但是对写入的数据做了透明的加密，并且要求TLS协商。

这个实例实现了一个双工流接口。它有所有普通流所拥有的事件和方法。

#### Event: 'secureConnect'#

在一个新连接成功握手后，这个事件被触发。无论服务器的证书被授权与否，这个监听器都会被调用。测试`tlsSocket.authorized`来 验证服务器证书是否被一个指定CA所签名 取决于用户。如果`tlsSocket.authorized === false`那么错误可以从`tlsSocket.authorizationError`里被发现。如果`NPN`被使用，你可以通过`tlsSocket.npnProtocol`来检查已协商协议。

#### Event: 'OCSPResponse'#

 - function (response) { }

如果`requestOCSP`选项被设置，这个事件会触发。`response`是一个`buffer`对象，包含了服务器的OCSP响应。

习惯上，`response`是一个来自服务器的CA（包含服务器的证书撤销状态）的已签名对象。

#### tlsSocket.encrypted#

静态布尔变量，总是`true`。可能会被用来区分TLS `socket`和普通的`socket`。

#### tlsSocket.authorized#

如果对等（peer）证书通过一个指定的CA被签名，那么这个值为`true`。否则为`false`。

#### tlsSocket.authorizationError#

对等（peer）的证书没有被验证的原因。这个值只在`tlsSocket.authorized === false`时可用。

#### tlsSocket.getPeerCertificate([ detailed ])#

返回了一个代表了对等证书的对象。返回的对象有一些属性与证书的属性一致。如果`detailed`参数被设置为`true`，`issuer`属性的完整链都会被返回，如果为`false`，只返回不包含`issuer`属性的顶端的证书。

例子：

```js
{ subject:
   { C: 'UK',
     ST: 'Acknack Ltd',
     L: 'Rhys Jones',
     O: 'io.js',
     OU: 'Test TLS Certificate',
     CN: 'localhost' },
  issuerInfo:
   { C: 'UK',
     ST: 'Acknack Ltd',
     L: 'Rhys Jones',
     O: 'io.js',
     OU: 'Test TLS Certificate',
     CN: 'localhost' },
  issuer:
   { ... another certificate ... },
  raw: < RAW DER buffer >,
  valid_from: 'Nov 11 09:52:22 2009 GMT',
  valid_to: 'Nov  6 09:52:22 2029 GMT',
  fingerprint: '2A:7A:C2:DD:E5:F9:CC:53:72:35:99:7A:02:5A:71:38:52:EC:8A:DF',
  serialNumber: 'B9B0D332A1AA5635' }
```

如果`peer`没有提供一个证书，那么会返回`null`或空对象。

#### tlsSocket.getCipher()#

返回一个代表了当前连接的加密器名和SSL/TLS协议版本的对象。

例子： `{ name: 'AES256-SHA', version: 'TLSv1/SSLv3' }`

参阅`http://www.openssl.org/docs/ssl/ssl.html#DEALING_WITH_CIPHERS`中`SSL_CIPHER_get_name()`和`SSL_CIPHER_get_version()`。

#### tlsSocket.renegotiate(options, callback)#

初始化TLS重新协商过程。`optios`可以包含以下属性：`rejectUnauthorized`，`requestCert`（详情参阅`tls.createServer`）。一旦重协商成功，`callback(err)`会带着`err`为`null`执行。

注意：可以被用来请求对等（peer）证书在安全连接建立之后。

另一个注意点：当作为服务器运行时，`socekt`在`handshakeTimeout`超时后，会带着一个错误被销毁。

#### tlsSocket.setMaxSendFragment(size)#

设置TLS碎片大小的最大值（默认最大值为`16384`，最小值为`512`）。若设置成功返回`true`，否则返回`false`。

更小的碎片大小来减少客户端的缓冲延迟：大的碎片通过TLS层缓冲，直到收到全部的碎片并且它的完整性被验证；大碎片可能会跨越多次通信，并且可能会被报文丢失和重新排序所延迟。但是，更小的碎片增加了额外的TLS框架字节和CPU开销，可能会减少总体的服务器负载。

#### tlsSocket.getSession()#

返回`ASN.1`编码的TLS会话，如果没有被协商，返回`undefined`。可以被用在重新连接服务器时，加速握手的建立。

#### tlsSocket.getTLSTicket()#

注意：仅在客户端TLS `socket`中工作。仅在调试时有用，因为会话重新使用了给`tls.connect`提供的`session`选项。

返回TLS会话门票（ticket），如果没有被协商，返回`undefined`。

#### tlsSocket.address()#

返回绑定的地址，协议族名和端口由底层系统报告。返回一个含有三个属性的对象，例如：`{ port: 12346, family: 'IPv4', address: '127.0.0.1' }`。

#### tlsSocket.remoteAddress#

代表了远程IP地址的字符串。例子：`'74.125.127.100'`或`'2001:4860:a005::68'`。

#### tlsSocket.remoteFamily#

代表了远程IP协议族的字符串。`'IPv4'`或`'IPv6'`。

#### tlsSocket.remotePort#

代表了远程端口数字。例子：`443`。

#### tlsSocket.localAddress#

代表了本地IP地址的字符串。

#### tlsSocket.localPort#

代表了本地端口的数字。