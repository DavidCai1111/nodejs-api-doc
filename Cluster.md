# Cluster#


### 稳定度: 2 - 稳定

单个的`node.js`实例运行在单线程上。为了享受多核系统的优势，用户需要启动一个`node.js`集群来处理负载。

`cluster`模块允许你方便地创建共享服务器端口的子进程：

```js
var cluster = require('cluster');
var http = require('http');
var numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
  // Fork workers.
  for (var i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on('exit', function(worker, code, signal) {
    console.log('worker ' + worker.process.pid + ' died');
  });
} else {
  // Workers can share any TCP connection
  // In this case its a HTTP server
  http.createServer(function(req, res) {
    res.writeHead(200);
    res.end("hello world\n");
  }).listen(8000);
}
```

启动`node.js`将会在工作线程中共享8000端口：

```SHELL
% NODE_DEBUG=cluster iojs server.js
23521,Master Worker 23524 online
23521,Master Worker 23526 online
23521,Master Worker 23523 online
23521,Master Worker 23528 online
```

这个特性是最近才开发的，并且可能在未来有所改变。请试用它并提供反馈。

注意，在Windows中，在工作进程中建立命名管道服务器目前是不可行的。

#### 工作原理

工作进程通过`child_process.fork`方法被创建，所以它们可以与父进程通过IPC管道沟通以及相互传递服务器句柄。

集群模式支持两种分配传入连接的方式。

第一种（并且是除了Windows平台外默认的方式）是循环式。主进程监听一个端口，接受新连接，并且以轮流的方式分配给工作进程，并且以一些内建机制来避免一个工作进程过载。

第二种方式是，主进程建立监听`socket`并且将它发送给感兴趣的工作进程。工作进程直接接受传入的连接。

第二种方式理论上有最好的性能。但是在实践中，操作系统的调度不可预测，分配往往十分不平衡。负载曾被观察到8个进程中，超过70%的连接结束于其中的2个进程。

因为`server.listen()`将大部分工作交给了主进程，所以一个普通的`node.js`进程和一个集群工作进程会在三种情况下有所区别：

 1. `server.listen({fd: 7})` 因为消息被传递给了主进程，主进程的文件描述符`7`会被监听，并且句柄会被传递给工作进程而不是监听工作进程中文件描述符为`7`的东西。

 2. `server.listen(handle)` 明确地监听句柄，会让工作进程使用给定的句柄，而不是与主进程通信。如果工作进程已经有了此句柄，那么将假设你知道你在做什么。

 3. `server.listen(0)` 通常，这会导致服务器监听一个随机端口。但是，在集群中，每次调用`listen(0)`时，每一个工作进程会收到同样的“随机”端口。也就是说，端口只是在第一次方法被调用时是随机的，但在之后是可预知的。如果你想监听特定的端口，则根据工作进程的PID来生成端口号。

由于在`node.js`或你的程序中的工作进程间没有路由逻辑也没有共享的状态。所以，请不要为你的程序设计成依赖太重的内存数据对象，如设计会话和登陆时。

因为工作进程都是独立的进程，它们可以根据你程序的需要被杀死或被创建，并且并不会影响到其他工作进程。只要有活跃的工作进程，那么服务器就会继续接收连接。但是`node.js`不会自动地为你管理工作进程数。所以根据你的应用需求来管理工作进程池是你的责任。

#### cluster.schedulingPolicy#
调度策略，选择`cluster.SCHED_RR`来使用循环式，或选择`cluster.SCHED_NONE`来由操作系统处理。这是一个全局设定，并且在你第一次启动了一个工作进程或调用`cluster.setupMaster()`方法后就不可再更改。

`SCHED_RR`是除了Windows外其他操作系统中的默认值。一旦`libuv`能够有效地分配IOCP句柄并且没有巨大的性能损失，那么Windows下的默认值也会变为它。

`cluster.schedulingPolicy`也可以通过环境变量`NODE_CLUSTER_SCHED_POLICY`来设定。合法值为`rr`和`none`。

#### cluster.settings#
 - __Object__
  - execArgv Array 传递给`node.js`执行的字符串参数（默认为`process.execArgv`）
  - exec String 工作进程文件的路径（默认为`process.argv[1]`）
  - args Array 传递给工作进程的字符串参数（默认为`process.argv.slice(2)`）
  - silent Boolean 是否将工作进程的输出传递给父进程的`stdio `（默认为`false`）
  - uid Number 设置用户进程的ID
  - gid Number 设置进程组的ID

在调用`.setupMaster()`（或`.fork()`）方法之后，这个`settings`对象会存放方法的配置，包括默认值。

因为`.setupMaster()`仅能被调用一次，所以这个对象被设置后便不可更改。

这个对象不应由你来手工更改或设置。

#### cluster.isMaster#
 - Boolean

如果进程是主进程则返回`true`。这由`process.env.NODE_UNIQUE_ID`决定。如果`process.env.NODE_UNIQUE_ID`为`undefined`，那么就返回`true`。

#### cluster.isWorker#
 - Boolean

如果进程不是主进程则返回`true`。

#### Event: 'fork'#
 - worker Worker object

当一个新的工作进程由`cluster`模块所开启时会触发`fork`事件。这能被用来记录工作进程活动日志，或创建自定义的超时。

```js
var timeouts = [];
function errorMsg() {
  console.error("Something must be wrong with the connection ...");
}

cluster.on('fork', function(worker) {
  timeouts[worker.id] = setTimeout(errorMsg, 2000);
});
cluster.on('listening', function(worker, address) {
  clearTimeout(timeouts[worker.id]);
});
cluster.on('exit', function(worker, code, signal) {
  clearTimeout(timeouts[worker.id]);
  errorMsg();
});
```

#### Event: 'online'#
 - worker Worker object

当创建了一个新的工作线程后，工作线程必须响应一个在线信息。当主进程接收到在线信息后它会触发这个事件。`fork`和`online`事件的区别在于：`fork`是主进程创建了工作进程后触发，`online`是工作进程开始运行时触发。

```js
cluster.on('online', function(worker) {
  console.log("Yay, the worker responded after it was forked");
});
```

#### Event: 'listening'#
 - worker Worker object
 - address Object

当工作进程调用`listen()`方法。服务器会触发`listening`事件，集群中的主进程也会触发一个`listening`事件。

这个事件的回调函数包含两个参数，第一个`worker`是一个包含工作进程的对象，`address`对象是一个包含以下属性的对象：`address`，`port`和`addressType`。当工作进程监听多个地址时，这非常有用。

```
cluster.on('listening', function(worker, address) {
  console.log("A worker is now connected to " + address.address + ":" + address.port);
});
```

`addressType`是以下中的一个：

 - 4 (TCPv4)
 - 6 (TCPv6)
 - -1 (unix domain socket)
 - "udp4" 或 "udp6" (UDP v4 或 v6)

#### Event: 'disconnect'#
 - worker Worker object

当工作进程的IPC信道断开连接时触发。这个事件当工作进程优雅地退出，被杀死，或手工断开连接（如调用`worker.disconnect()`）后触发。

`disconnect`和`exit`事件之间可能存在延迟。这两个事件可以用来侦测是否进程在清理的过程中被阻塞，或者是否存在长连接。

```js
cluster.on('disconnect', function(worker) {
  console.log('The worker #' + worker.id + ' has disconnected');
});
```

#### Event: 'exit'#
 - worker Worker object
 - code Number 如果正常退出，则为退出码
 - signal String 导致进程被杀死的信号的信号名（如'SIGHUP'）

当任何一个工作进程结束时，`cluster`模块会触发一个`exit`事件。

这可以被用来通过再次调用`.fork()`方法重启服务器。

```js
cluster.on('exit', function(worker, code, signal) {
  console.log('worker %d died (%s). restarting...',
    worker.process.pid, signal || code);
  cluster.fork();
});
```

参阅`child_process`事件：`exit `。

#### Event: 'setup'#
 - settings Object

每次`.setupMaster()`方法被调用时触发。

这个`settings`对象与`.setupMaster()`被调用时`cluster.settings`对象相同，并且仅供查询，因为`.setupMaster()`可能在一次事件循环里被调用多次。

如果保持精确十分重要，请使用`cluster.settings`。

#### cluster.setupMaster([settings])#
 - __settings Object__
  - exec String 工作进程文件的路径（默认为`process.argv[1]`）
  - args Array 传递给工作进程的参数字符串（默认为`process.argv.slice(2)`）
  - silent Boolean 是否将工作进程的输出传递给父进程的`stdio`(默认为`false`)

`setupMaster`方法被用来改变默认的`fork`行为。一旦被调用，`settings`参数将被表现为`cluster.settings`。

注意：

 - 任何`settings`的改变仅影响之后的`.fork()`调用，而不影响已经运行中的工作进程
 - 工作进程中唯一不同通过`.setupMaster()`来设置的属性是传递给`.fork()`方法的`env`参数
 - 上文中的参数的默认值仅在第一次调用时被应用，之后的调用的默认值是当前`cluster.setupMaster()`被调用时的值。

例子：

```js
var cluster = require('cluster');
cluster.setupMaster({
  exec: 'worker.js',
  args: ['--use', 'https'],
  silent: true
});
cluster.fork(); // https worker
cluster.setupMaster({
  args: ['--use', 'http']
});
cluster.fork(); // http worker
```

这只能被主进程调用。

#### cluster.fork([env])#
 - env Object 将添加到工作进程环境变量的键值对
 - return Worker object

创建一个新的工作进程。

这只能被主进程调用。

#### cluster.disconnect([callback])#
 - callback Function 当所有的工作进程断开连接并且所有句柄关闭后调用

在`cluster.workers`中的每一个工作进程中调用`.disconnect()`。

当所有进程断开连接，所有内部的句柄都将关闭，如果没有其他的事件处于等待，将允许主进程优雅地退出。

这个方法接受一个可选的将会在结束时触发的回调函数参数。

这只能被主进程调用。

#### cluster.worker#
 - Object

当前工作进程对象的引用。对于主进程不可用。

```js
var cluster = require('cluster');

if (cluster.isMaster) {
  console.log('I am master');
  cluster.fork();
  cluster.fork();
} else if (cluster.isWorker) {
  console.log('I am worker #' + cluster.worker.id);
}
```

#### cluster.workers#
 - Object

一个储存了所有活跃的工作进程对象的哈希表，以`id`字段为主键。这使得遍历所有工作进程变得容易。仅在主进程中可用。

当工作进程断开连接或退出时，它会从`cluster.workers`中移除。这个两个事件的触发顺序不能被提前决定。但是，能保证的是，从`cluster.workers`移除一定发生在这两个事件触发之后。

```js
// Go through all workers
function eachWorker(callback) {
  for (var id in cluster.workers) {
    callback(cluster.workers[id]);
  }
}
eachWorker(function(worker) {
  worker.send('big announcement to all workers');
});
```

想要跨越通信信道来得到一个工作进程的引用时，使用工作进程的唯一`id`能简单找到工作进程。

```js
socket.on('data', function(id) {
  var worker = cluster.workers[id];
});
```

#### Class: Worker#

`Worker`对象包含了一个工作进程所有的公开信息和方法。在主进程中它可以通过`cluster.workers`取得。在工作进程中它可以通过`cluster.worker`取得。

#### worker.id#

 - String

每一个新的工作进程都被给予一个独一无二的id，这个id被存储在此`id`属性中。

当一个工作进程活跃时，这是它被索引在`cluster.workers`中的主键。

#### worker.process#

 - ChildProcess object

所有的工作进程都通过`child_process.fork()`被创建，返回的对象被作为`.process`属性存储。在一个工作进程中，全局的`process`被存储。

参阅`Child Process module`

注意，如果在进程中`disconnect`事件触发并且`.suicide`属性不为`true`，那么进程会调用`process.exit(0)`。这防止了意外的断开连接。

#### worker.suicide#

 - Boolean

通过调用`.kill()`或`.disconnect()`设置，在这之前他为`undefined`。

布尔值`worker.suicide`使你可以区别自发和意外的退出，主进程可以通过这个值来决定使用重新创建一个工作进程。

```js
cluster.on('exit', function(worker, code, signal) {
  if (worker.suicide === true) {
    console.log('Oh, it was just suicide\' – no need to worry').
  }
});

// kill worker
worker.kill();
```

#### worker.send(message[, sendHandle])#

 - message Object
 - sendHandle Handle object

给工作进程或主进程发生一个信息，可选得添加一个句柄。

在主进程中它将给特定的工作进程发送一个信息。它指向`child.send()`。

在工作进程中它将给主进程发送一个信息。它指向`process.send()`。

下面的例子将来自主进程的所有信息返回：

```js
if (cluster.isMaster) {
  var worker = cluster.fork();
  worker.send('hi there');

} else if (cluster.isWorker) {
  process.on('message', function(msg) {
    process.send(msg);
  });
}
```

#### worker.kill([signal='SIGTERM'])#

 - signal String 传递给工作进程的结束信号名

这个函数将会杀死工作进程。在主进程中，它通过断开`worker.process`做到，并且一旦断开，使用`signal`杀死进程。在工作进程中，它通过断开信道做到，然后使用退出码`0`退出。

会导致`.suicide`被设置。

为了向后兼任，这个方法的别名是`worker.destroy()`。

注意在工作进程中，`process.kill()`存在，但它不是这个函数。是`process.kill(pid[, signal])`。

#### worker.disconnect()#

在工作进程中，这个函数会关闭所有的服务器，等待这些服务器上的`close`事件，然后断开IPC信道。

在主进程中，一个内部信息会被传递给工作进程，至使它们自行调用`.disconnect()`。

会导致`.suicide`被设置。

注意在一个服务器被关闭后，它将不会再接受新连接，但是连接可能被其他正在监听的工作进程所接收。已存在的连接将会被允许向往常一样退出。当没有更多的连接存在时，工作进程的IPC信道会关闭并使之优雅地退出，参阅`server.close()`。

以上说明仅应用于服务器连接，客户端连接将不会自动由工作进程关闭，并且在退出前，不会等到连接退出。

注意在工作进程中，`process.disconnect`存在，但它不是这个函数。是`child.disconnect()`。

由于长连接可能会阻塞工作进程的退出，这时传递一个动作信息非常有用，应用来根据信息指定的动作来关闭它们。超时机制是上述的有用实现，在`disconnect`事件在指定时长后没有触发时，杀死工作进程。

```js
if (cluster.isMaster) {
  var worker = cluster.fork();
  var timeout;

  worker.on('listening', function(address) {
    worker.send('shutdown');
    worker.disconnect();
    timeout = setTimeout(function() {
      worker.kill();
    }, 2000);
  });

  worker.on('disconnect', function() {
    clearTimeout(timeout);
  });

} else if (cluster.isWorker) {
  var net = require('net');
  var server = net.createServer(function(socket) {
    // connections never end
  });

  server.listen(8000);

  process.on('message', function(msg) {
    if(msg === 'shutdown') {
      // initiate graceful close of any connections to server
    }
  });
}
```

#### worker.isDead()#

如果工作进程已经被关闭，则返回`true`。

#### worker.isConnected()#

如果工作进程通过它的IPC信道连接到主进程，则返回`true`。一个工作进程在被创建后连接到它的主进程。在`disconnect`事件触发后它会断开连接。

#### Event: 'message'#

 - message Object

这个事件与`child_process.fork()`所提供的事件完全相同。

在工作进程中你也可以使用`process.on('message')`。

例子，这里有一个集群，使用消息系统在主进程中统计请求的数量：

```js
var cluster = require('cluster');
var http = require('http');

if (cluster.isMaster) {

  // Keep track of http requests
  var numReqs = 0;
  setInterval(function() {
    console.log("numReqs =", numReqs);
  }, 1000);

  // Count requestes
  function messageHandler(msg) {
    if (msg.cmd && msg.cmd == 'notifyRequest') {
      numReqs += 1;
    }
  }

  // Start workers and listen for messages containing notifyRequest
  var numCPUs = require('os').cpus().length;
  for (var i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  Object.keys(cluster.workers).forEach(function(id) {
    cluster.workers[id].on('message', messageHandler);
  });

} else {

  // Worker processes have a http server.
  http.Server(function(req, res) {
    res.writeHead(200);
    res.end("hello world\n");

    // notify master about the request
    process.send({ cmd: 'notifyRequest' });
  }).listen(8000);
}
```

#### Event: 'online'#

与`cluster.on('online')`事件相似，但指向了特定的工作进程。

```js
cluster.fork().on('online', function() {
  // Worker is online
});
```

这不是在工作进程中触发的。

#### Event: 'listening'#

 - address Object

与`cluster.on('listening')`事件相似，但指向了特定的工作进程。

```js
cluster.fork().on('listening', function(address) {
  // Worker is listening
});
```

这不是在工作进程中触发的。

#### Event: 'disconnect'#

与`cluster.on('disconnect')`事件相似，但指向了特定的工作进程。

```js
cluster.fork().on('disconnect', function() {
  // Worker has disconnected
});
```

#### Event: 'exit'#

 - code Number 如果正常退出，则为退出码
 - signal String 导致进程被杀死的信号名（如`'SIGHUP'`）

与`cluster.on('exit')`事件相似，但指向了特定的工作进程。

```js
var worker = cluster.fork();
worker.on('exit', function(code, signal) {
  if( signal ) {
    console.log("worker was killed by signal: "+signal);
  } else if( code !== 0 ) {
    console.log("worker exited with error code: "+code);
  } else {
    console.log("worker success!");
  }
});
```

#### Event: 'error'#

这个事件与`child_process.fork()`所提供的事件完全相同。

在工作进程中你也可以使用`process.on('error')`。
