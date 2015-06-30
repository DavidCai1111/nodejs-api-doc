# UDP / Datagram Sockets#

### 稳定度: 2 - 稳定

数据报`socket`通过`require('dgram')`使用。

重要提示：`dgram.Socket#bind()`的表现在v0.10中被改变，并且现在总是异步的，如果你有像这样的代码： 

```js
var s = dgram.createSocket('udp4');
s.bind(1234);
s.addMembership('224.0.0.114');
```

你必须改成这样：

```js
var s = dgram.createSocket('udp4');
s.bind(1234, function() {
  s.addMembership('224.0.0.114');
});
```

#### dgram.createSocket(type[, callback])#
 - type String. `'udp4'`或`'udp6'`，两者之一
 - callback Function. 可选，会被添加为`message`事件的监听器
 - Returns: `socket`对象
 
创建一个指定类型的数据报`socket`。可用类型是udp4和udp6.

接受一个可选的回调函数，它会被自动添加为`message`事件的监听器。

如果你想要接收数据报，调用`socket.bind()`。`socket.bind()`将会到`所有网络接口`地址中的一个随机端口（不论udp4和upd6 `socket`，它都可以正常工作）。你可以从`socket.address().address`和`socket.address().port`中获取地址和端口。

#### dgram.createSocket(options[, callback])#
 - options Object
 - callback Function. 会被添加为`message`事件的监听器
 - Returns: `socket`对象

`options`对象必须包含一个`type`属性，可是udp4或udp6。还有一个可选的`reuseAddr`布尔值属性。

当`reuseAddr`为`true`时，`socket.bind()`会重用地址，甚至是当另一个进程已经在这之上绑定了一个`socket`时。默认为`false`。

接受一个可选的回调函数，它会被自动添加为`message`事件的监听器。

如果你想要接收数据报，调用`socket.bind()`。`socket.bind()`将会到`所有网络接口`地址中的一个随机端口（不论udp4和upd6 `socket`，它都可以正常工作）。你可以从`socket.address().address`和`socket.address().port`中获取地址和端口。

#### Class: dgram.Socket#

`dgram.Socket`类封装了数据报的功能。它必须被`dgram.createSocket(...)`创建。

#### Event: 'message'#

 - msg Buffer object. 消息
 - rinfo Object. 远程地址信息

当在`socket`中一个新的数据报可用时触发。`msg`是一个`buffer`并且`rinfo`是一个包含发送者地址信息的对象：

```js
socket.on('message', function(msg, rinfo) {
  console.log('Received %d bytes from %s:%d\n',
              msg.length, rinfo.address, rinfo.port);
});
```

#### Event: 'listening'#

当一个`socket`开始监听数据报时触发。在UDP `socket`被创建时触发。

#### Event: 'close'#

在一个`socket`通过`close()`被关闭时触发。这个`socket`中不会再触发新的`message`事件。

#### Event: 'error'#

 - exception Error object
 
当错误发生时触发。

#### socket.send(buf, offset, length, port, address[, callback])#

 - buf Buffer object or string. 要被发送的信息。
 - offset Integer. 信息在`buffer`里的初始偏移位置。
 - length Integer. 信息的字节数。
 - port Integer. 目标端口。
 - address String. 目标主机或IP地址。
 - callback Function. 可选，当信息被发送后调用。

对于UDP `socket`，目标端口和地址都必须被指定。`address`参数需要提供一个字符串，并且它会被DNS解析。

如果`address`被忽略，或者是一个空字符串。将会使用`'0.0.0.0'`或`'::0'`。这取决于网络配置，这些默认值 可能会 或 可能不会 正常工作；所以最好还是明确指定目标地址。

如果一个`socket`先前没有被调用`bind`来绑定，它将会赋于一个随机端口数并且被绑定到“所有网络接口”地址（udp4 `socket`为`'0.0.0.0'`，udp6则为`'::0'`）。

一个可选的回调函数可以被指定，用来检测DNS错误，或决定重用`buf`对象是否安全。注意，DNS查找至少会延迟一个事件循环。唯一能确定数据报被发送的方法就是使用一个回调函数。

出于对多字节字符的考虑，`offset`和`length`将会根据字节长度而不是字符位置被计算。

一个向`localhost`上的一个随机端口发送UDP报文的例子：

```js
var dgram = require('dgram');
var message = new Buffer("Some bytes");
var client = dgram.createSocket("udp4");
client.send(message, 0, message.length, 41234, "localhost", function(err) {
  client.close();
});
```


##### UDP数据报大小的注意事项

IPv4/v6数据报的最大大小取决于`MTU`（最大传输单位），和`Payload Length`字段大小。

 - `Payload Length`是16字节宽的，意味着一个正常的负载不能超过64K 八位字节，包括网络头和数据（65,507 字节 = 65,535 − 8 字节 UDP 头 − 20 字节 IP 头）；对于环回接口总是`true`，但是如此大的数据报对于大多数主机和网络来说都是不现实的。

 - `MTU`是指定的链路层技术支持的报文的最大大小。对于所有连接，IPv4允许最小`MTU`为68八位字节，而推荐的IPv4 `MTU`是576（通常作为拨号类应用的推荐`MTU`），无论它们是完整的还是以碎片形式到达。

 - 对于IPv6，最小MTU是1280八位字节，但是，允许的最小`buffer`重组大小是1500八位字节。68八位字节非常小，所以大多数的当前链路层技术的最小`MTU`都是1500（如`Ethernet`）。

注意，不可能提前知道一个报文可能经过的每一个连接`MTU`，并且通常不能发送一个大于（接收者）`MTU`的数据报（报文会被默默丢弃，不会通知源头：这个数据没有到达已定的接收方）。

#### socket.bind(port[, address][, callback])#

 - port Integer
 - address String, 可选
 - callback Function 可选，没有参数。当绑定完毕后触发。
 
对于UDP `socket`，监听一个具名的端口和一个可选的地址上的数据报。如果`address`没有被指定，操作系统将会试图监听所有端口。在绑定完毕后，`listening`事件会被吃法，并且回调函数（如果指定了）会被调用。同时指定`listening`事件的监听器和`callback`没有危险，但是不是很有用。

一个绑定的数据报`socket`将会保持`io.js`进程的运行，来接受数据报。

如果绑定失败，一个`error`事件会产生。极少数情况下（例如绑定一个关闭的`socket`），这个方法会抛出一个错误。

一个监听41234端口的UDP服务器：

```js
var dgram = require("dgram");

var server = dgram.createSocket("udp4");

server.on("error", function (err) {
  console.log("server error:\n" + err.stack);
  server.close();
});

server.on("message", function (msg, rinfo) {
  console.log("server got: " + msg + " from " +
    rinfo.address + ":" + rinfo.port);
});

server.on("listening", function () {
  var address = server.address();
  console.log("server listening " +
      address.address + ":" + address.port);
});

server.bind(41234);
// server listening 0.0.0.0:41234
```

#### socket.bind(options[, callback])#

 - __options Object__ - 必选，支持以下属性：
  - port Number - 必须
  - address String - 可选
  - exclusive Boolean - 可选
 - callback Function - 可选

`options`的`prot`和`address`属性，以及可选的回调函数，与`socket.bind(port, [address], [callback])`中它们的表现一致。

如`exclusive`为`false`（默认），那么集群的工作进程将会使用相同的底层句柄，允许共享处理连接的职责。当为`true`时，句柄不被共享，企图共享端口会导致一个错误。一个监听一个`exclusive`端口的例子：

```js
socket.bind({
  address: 'localhost',
  port: 8000,
  exclusive: true
});
```

#### socket.close([callback])#

关闭底层`socket`，并且停止监听新数据。如果提供了回调函数，它会被添加为`close`事件的监听器。

#### socket.address()#

返回一个包含`socket`地址信息的对象。对于UDP `socket`，这个对象将会包含`address`，`family`和`port`。

#### socket.setBroadcast(flag)#

 - flag Boolean
 
设置或清除`SO_BROADCAST` `socket`设置。当这个选项被设置，UDP报文将会被送至本地接口的广播地址。

#### socket.setTTL(ttl)#

 - ttl Integer

设置`IP_TTL` `socket`选项。`TTL`的意思是“生存时间（Time to Live）”，但是在这里的上下文中，它值一个报文通过的IP跃点数。每转发报文的路由或网关都会递减`TTL`。如果`TTL`被一个路由递减为`0`，它将不再被转发。改变`TTL`值常用于网络探测器或多播。

`setTTL()`的参数是一个`1`到`225`之间的跃点数。多数系统中的默认值为`64`。

#### socket.setMulticastTTL(ttl)#

 - ttl Integer

设置`IP_MULTICAST_TTL` `socket`选项。`TTL`的意思是“生存时间（Time to Live）”，但是在这里的上下文中，它值一个报文通过的IP跃点数，特别是组播流量。每转发报文的路由或网关都会递减`TTL`。如果`TTL`被一个路由递减为`0`，它将不再被转发。

`setMulticastTTL()`的参数是一个`0`到`225`之间的跃点数。多数系统中的默认值为`1`。

#### socket.setMulticastLoopback(flag)#

 - flag Boolean
 
设置或清除`IP_MULTICAST_LOOP` `socket`选项。当这个选项被设置，组播报文也将会在本地接口上接收。

#### socket.addMembership(multicastAddress[, multicastInterface])#

 - multicastAddress String
 - multicastInterface String, 可选

告诉内核加入一个组播分组，通过`IP_ADD_MEMBERSHIP` `socket`选项。

如果`multicastInterface`没有被指定，那么操作系统将会尝试加入成为所有可用的接口的成员。

#### socket.dropMembership(multicastAddress[, multicastInterface])#

 - multicastAddress String
 - multicastInterface String, 可选

与`addMembership`相反 - 告诉内核离开一个组播分组，通过`IP_DROP_MEMBERSHIP` `socket`选项。当`socket`被关闭或进程结束时，它会被内核自动调用。所以大多数应用不需要亲自调用它。

如果`multicastInterface`没有被指定，那么操作系统将会尝试脱离所有可用的接口。

#### socket.unref()#

在一个`socket`上调用`unref`将会在它是事件系统中唯一活跃的`socket`时，允许程序退出。如果`socket`已经被`unref`，再次调用将不会有任何效果。

返回一个`socket`。

#### socket.ref()#

与`unref`相反，在一个先前被`unref`的`socket`上调用`ref`，那么在它是唯一的剩余的`socket`（默认行为）时，将不允许程序退出。如果`socket`已经被`ref`，再次调用将不会有任何效果。

返回一个`socket`。