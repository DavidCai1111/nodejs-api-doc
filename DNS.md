# DNS#

### 稳定度: 2 - 稳定

通过`require('dns')`来获取这个模块。

这个模块包含以下两类函数：

1) 使用底层操作系统工具来进行域名解析的函数，并且不需要进行任何网络活动。这类函数只有一个：`dns.lookup`。希望与 在其他操作系统的其他应用 执行域名解析 有相同行为时，请使用`dns.lookup`。

下面是一个解析`www.google.com`的例子：

```js
var dns = require('dns');

dns.lookup('www.google.com', function onLookup(err, addresses, family) {
  console.log('addresses:', addresses);
});
```

2) 连接实际的DNS服务器来进行域名解析的函数，并且经常使用网络来执行DNS查找。除了`dns.lookup`外`DNS`模块的所有函数都属于这类。这类函数不与`dns.lookup`使用相同的配置文件。例如，它们不使用`/etc/hosts`配置文件。这类函数适合那些不希望使用底层操作系统工具来进行域名解析，总是想要执行DNS查询的开发者。

下面例子是，解析`'www.google.com'`，然后反向解析返回的IP地址。

```js
var dns = require('dns');

dns.resolve4('www.google.com', function (err, addresses) {
  if (err) throw err;

  console.log('addresses: ' + JSON.stringify(addresses));

  addresses.forEach(function (a) {
    dns.reverse(a, function (err, hostnames) {
      if (err) {
        throw err;
      }

      console.log('reverse for ' + a + ': ' + JSON.stringify(hostnames));
    });
  });
});
```

两者之间的选择会产生微妙的结果，更多信息请查询下文`实现注意事项`章节。

#### dns.lookup(hostname[, options], callback)#

解析`hostname`（如`'google.com'`）为第一个找到的A（IPv4）或AAAA（IPv6）记录。`options`可以是对象或者数组。如果`options`没有提供，那么IPv4和IPv6都是有效的。如果`options`是一个数组，那么它必须是`4`或`6`。

另外，`options`可以是一个含有以下属性的对象：

 - family: {Number} - 地址族。如果提供，必须为整数`4`或`6`。如果没有提供，那么IPv4和IPv6都是有效的。
 - hints: {Number} - 如果提供，它必须是一个或多个支持的`getaddrinfo`标识。如果没有提供，那么没有标识被传递给`getaddrinfo`。多个标识可以通过在逻辑上`ORing`它们的值，来传递给`hints`。支持的`getaddrinfo`标识请参阅下文。
 - all: {Boolean} - 如果`true`，那么回调函数以数组的形式返回所有解析的地址，否则只返回一个地址。默认为`false`。

所有的属性都是可选的，以下是一个`options`例子：

```js
{
  family: 4,
  hints: dns.ADDRCONFIG | dns.V4MAPPED,
  all: false
}
```

回调函数有参数（err, address, family）。`address`是IPv4或IPv6地址字符串。`family`是`adress`的协议族，即`4`或`6`。

如果`options`的所有参数都被设置，那么参数转变为（err, addresses），`addresses`是一个地址和协议族数组。

若发生错误，`err`是错误对象，`err.code`是错误码。不仅在`hostname`不存在时，在如没有可用的文件描述符等情况下查找失败，`err.code`也会被设置为`'ENOENT'`。

`dns.lookup`不需要与DNS协议有任何关系。它仅仅是一个连接名字和地址的操作系统功能。

在任何的`io.js`程序中，它的实现对表现有一些微妙但是重要的影响。在使用前，请花一些时间查阅`实现注意事项`章节。

#### dns.lookupService(address, port, callback)#

解析给定的`address`和`port`为一个主机名和使用`getnameinfo`的服务。

回调函数有参数（err, hostname, service）。`hostname`和`service`参数是字符串（如分别为`'localhost'`和`'http'`）。

若发生错误，`err`是错误对象，`err.code`是错误码。

#### dns.resolve(hostname[, rrtype], callback)#

使用指定的`rrtype`类型，解析主机名（如`'google.com'`）为一个记录数组。

有效的`rrtype`有：

 - 'A' (IPV4 地址，默认)
 - 'AAAA' (IPV6 地址)
 - 'MX' (邮件交换记录)
 - 'TXT' (文本记录)
 - 'SRV' (SRV记录)
 - 'PTR' (用于IP反向查找)
 - 'NS' (域名服务器记录)
 - 'CNAME' (别名记录)
 - 'SOA' (权限开始记录)

回调函数有参数（err, addresses）。`address`中每个元素的类型由记录类型所指定，并且在下文相应的查找方法中有描述。

若发生错误，`err`是错误对象，`err.code`是下文错误代码列表中的一个。

#### dns.resolve4(hostname, callback)#

与`dns.resolve()`相同，但只使用IPv4查询（一个记录）。地址是一个IPv4地址数组（如`['74.125.79.104', '74.125.79.105', '74.125.79.106']`）。

#### dns.resolve6(hostname, callback)#

与`dns.resolve4()`相同，除了使用IPv6查询（一个AAAA查询）。

#### dns.resolveMx(hostname, callback)#

与`dns.resolve()`相同，但是只用于邮件交换查询（MX记录）。

地址是一个MX记录数组，每一个元素都有一个`priority `和一个`exchange`属性（如`[{'priority': 10, 'exchange': 'mx.example.com'},...]`）。

#### dns.resolveTxt(hostname, callback)#

与`dns.resolve()`相同，但是只用于文本查询（TXT记录）。地址是一个`hostname`可用的2-d数组（如`[ ['v=spf1 ip4:0.0.0.0 ', '~all' ] ]`）。每个字数组包含一个记录的TXT数据块。根据使用场景的不同，它们可能被连接在一起也可能被分开。

#### dns.resolveSrv(hostname, callback)#

与`dns.resolve()`相同，但是只用于服务查询（SRV记录）。地址是一个`hostname`可用的SRV记录数组。SRV记录的属性有`priority`，`weight`，`port`，和`name`（如`[{'priority': 10, 'weight': 5, 'port': 21223, 'name': 'service.example.com'}, ...]`）。

#### dns.resolveSoa(hostname, callback)#

与`dns.resolve()`相同，但是只用于权限记录查询（SOA记录）。

地址是一个有以下结构的对象：

```js
{
  nsname: 'ns.example.com',
  hostmaster: 'root.example.com',
  serial: 2013101809,
  refresh: 10000,
  retry: 2400,
  expire: 604800,
  minttl: 3600
}
```

#### dns.resolveNs(hostname, callback)#

与`dns.resolve()`相同，但是只用于域名服务器查询（NS记录）。地址是一个`hostname`可用的域名服务器记录数组（如`['ns1.example.com', 'ns2.example.com']`）。

#### dns.resolveCname(hostname, callback)#

与`dns.resolve()`相同，但是只用于别名记录（别名记录）。地址是一个`hostname`可用的别名数组（如`['bar.example.com']`）。

#### dns.reverse(ip, callback)#

为得到一个主机名数组，反向查询一个IP。

回调函数有参数（err, hostnames）。

若发生错误，`err`是错误对象，`err.code`是下文错误代码列表中的一个。

#### dns.getServers()#

返回一个正在被用于解析的IP地址字符串数组。

#### dns.setServers(servers)#

给定一个IP地址字符串数组，将它们设置给用来解析的服务器。

如果你为地址指定了一个端口，端口会被忽略，因为底层库不支持。

如果你传递了非法输入，会抛出错误。

#### Error codes#

每一次DNS查询都可能返回以下错误码之一：

 - dns.NODATA: DNS服务器返回一个没有数据的应答。
 - dns.FORMERR: DNS服务器声明查询是格式错误的。
 - dns.SERVFAIL: DNS服务器返回一个普通错误。
 - dns.NOTFOUND: 域名没有找到。
 - dns.NOTIMP: DNS服务器没有实现请求的操作。
 - dns.REFUSED: DNS服务器拒绝查询。
 - dns.BADQUERY: 格式错误的DNS查询。
 - dns.BADNAME: 格式错误的主机名。
 - dns.BADFAMILY: 不支持的协议族。
 - dns.BADRESP: 格式错误的DNS响应。
 - dns.CONNREFUSED: 不能连接到DNS服务器。
 - dns.TIMEOUT: 连接DNS服务器超时。
 - dns.EOF: 文件末端。
 - dns.FILE: 读取文件错误。
 - dns.NOMEM: 内存溢出。
 - dns.DESTRUCTION: 通道被销毁。
 - dns.BADSTR: 格式错误的字符串。
 - dns.BADFLAGS: 指定了非法标志。
 - dns.NONAME: 给定的主机名不是数字。
 - dns.BADHINTS: 给定的提示标识非法。
 - dns.NOTINITIALIZED: `c-ares`库初始化未被执行。
 - dns.LOADIPHLPAPI: 加载`iphlpapi.dll`错误。
 - dns.ADDRGETNETWORKPARAMS: 找不到`GetNetworkParams`函数。
 - dns.CANCELLED: DNS查询被取消。

#### 支持的getaddrinfo标识

以下标识可以被传递给`dns.lookup`的`hints`：

 - dns.ADDRCONFIG: 返回的地址类型由当前系统支持的地址类型决定。例如，如果当前系统至少有一个IPv4地址被配置，那么将只会返回IPv4地址。回溯地址不被考虑。
 - dns.V4MAPPED: 如果IPv6协议族被指定，但是没有发现IPv6地址，那么返回IPv6地址的IPv4映射。

### 实现注意事项

尽管`dns.lookup`和`dns.resolve*/dns.reverse`函数都用于关联一个域名和一个地址（或反之亦然），它们的行为还是有些许区别。这些差别虽然微小，但是对于`io.js`程序的行为有重大影响。

#### dns.lookup

在引擎下，`dns.lookup`使用了和其他程序相同的操作系统功能。例如，`dns.lookup`将总是和`ping`命令一样解析一个给定的域名。在大多类POSIX操作系统上，`dns.lookup`函数的表现可以通过改变`nsswitch.conf(5)` 和/或 `resolv.conf(5)`的设置来调整，但是需要小心的是，改变这些文件将会影响这个操作系统上正在运行的所有其他程序。

虽然在`JavaScript`的角度，这个调用是异步的，但是它在libuv线程池中的实现是同步调用`getaddrinfo(3)`。因为libuv线程池有一个固定的大小，意味着如果`getaddrinfo(3)`花费了太多的时间，那么其他libuv线程池中的操作（如文件系统操作）会感觉到性能下降。为了缓解这个情况，一个潜在的解决方案是通过设置`'UV_THREADPOOL_SIZE'`环境变量大于4（当前默认值）来增加libuv线程池的大小。更多libuv线程池的信息，请参阅官方的libuv文档。

#### dns.resolve，以dns.resolve和dns.reverse开头的函数

这些函数的实现与`dns.lookup`相当不同。它们不使用`getaddrinfo(3)`并且它们总是通过网络执行一次DNS查询。这些网络通信通常是异步的，并且不使用libuv线程池。

作为结果，其他使用libuv线程池的`dns.lookup`方法的进程可能会有相同的负面影响，但这些函数没有。

它们与`dns.lookup`使用了不同的配置文件。例如，它们不使用`/etc/hosts`中的配置。