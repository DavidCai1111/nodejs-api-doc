# Events#

### 稳定度: 2 - 稳定

`node.js`中的许多对象触发事件：一个`net.Server`每次被连接时触发事件，一个`fs.readStream`当文件打开时触发事件。所有触发事件的对象都是`events.EventEmitter`的实例。你可以通过`require("events");`来取得这个模块。

通常，事件名以驼峰字符串来命令，但是这不是严格要求的，任何字符串都是可以接受的。

为了处理触发的事件，我们将函数关联到对象上。这些函数被称为监听器。在监听器中，`this`指向监听器所关联的`EventEmitter`实例。

#### Class: events.EventEmitter#

使用`require('events')`来获取这个`EventEmitter`类。

```js
var EventEmitter = require('events');
```

当一个`EventEmitter`实例发生了一个错误，一个典型的做法是触发一个`error`事件。`error`事件在`node.js`中被视为一个特殊的事件，如果没有为其添加监听器，默认的行为是打印堆栈追踪信息并推出程序。

所有的`EventEmitter`实例，在被添加新的监听器时，都会触发`newListener`事件。当有监听器被移除时，都会触发`removeListener `事件。

#### emitter.addListener(event, listener)#

#### emitter.on(event, listener)#

为指定的事件，在其监听器数组的末尾添加一个新的监听器。不会去检查这个事件是否已经被监听过。事件的多次触发会导致监听器的多次被调用。

```js
server.on('connection', function (stream) {
  console.log('someone connected!');
});
```

返回一个`emitter`，所以可以被链式调用。

#### emitter.once(event, listener)#

为事件添加一个 一次性 监听器。这个监听器只会在下次事件触发时被调用，之后被移除。

```js
server.once('connection', function (stream) {
  console.log('Ah, we have our first user!');
});
```

返回一个`emitter`，所以可以被链式调用。

#### emitter.removeListener(event, listener)#

从监听器数组中移除指定事件的一个监听器。注意：在数组中，此监听器被移除后，其之后的监听器的索引会被改变。

```js
var callback = function(stream) {
  console.log('someone connected!');
};
server.on('connection', callback);
// ...
server.removeListener('connection', callback);
```

`removeListener`一次只会从监听器数组中移除一个监听器。如果特定事件的单个的监听器被添加了多次，`removeListener`也必须调用同样多次来移除它们。

返回一个`emitter`，所以可以被链式调用。

#### emitter.removeAllListeners([event])#

移除指定事件的所有监听器。使用这个方法来 移除不是在你的代码中创建的`emitter`（如`socket`和`fs`）的所有监听器 ，并不是一个明智的选择。

返回一个`emitter`，所以可以被链式调用。

#### emitter.setMaxListeners(n)#

默认的，当一个特定事件被添加了超过10个监听器时，`EventEmitter`会打印一个警告。这是一个对于发现内存泄露非常有用的默认警告。但是显然，并不是所有的`emitter`都应当被限制。这个函数可以用来增加这个上限。如果想要无限制，请设置`0`。

返回一个`emitter`，所以可以被链式调用。

#### emitter.getMaxListeners()#

返回`emitter`当前的最大监听器数的值，可能是`emitter.setMaxListeners(n)`设置的值，或者是`EventEmitter.defaultMaxListeners`。

这个值对于调节最大监听器数来避免 不负责任的警告 或 最大监听器数过大，都非常有用。

```js
emitter.setMaxListeners(emitter.getMaxListeners() + 1);
emitter.once('event', function () {
  // do stuff
  emitter.setMaxListeners(Math.max(emitter.getMaxListeners() - 1, 0));
});
```

#### EventEmitter.defaultMaxListeners#

`emitter.setMaxListeners(n)`在实例级别设置最大监听器数。这个类属性让你可以设置所有`EventEmitter`的默认最大监听器数，对当前已创建的和未来创建的`EventEmitter`都有效。请谨慎使用它。

注意，`emitter.setMaxListeners(n)`仍优先于`EventEmitter.defaultMaxListeners`。

#### emitter.listeners(event)#

返回指定事件的监听器数组。

```js
server.on('connection', function (stream) {
  console.log('someone connected!');
});
console.log(util.inspect(server.listeners('connection'))); // [ [Function] ]
```

#### emitter.emit(event[, arg1][, arg2][, ...])#

使用提供的参数，执行每一个监听器。

如果事件有监听器，那么返回`true`，否则返回`false`。

#### Class Method: EventEmitter.listenerCount(emitter, event)#

返回指定事件的监听器数。

#### Event: 'newListener'#

 - event String 事件名
 - listener Function 事件监听器函数

这个事件在监听器被添加前触发。当这个事件被触发时，监听器还没有被添加到事件的监听器数组中。在`newListener`事件的回调函数中拿到事件名时，监听器还没有开始被添加到该事件。

#### Event: 'removeListener'#

 - event String 事件名
 - listener Function 事件监听器函数
 
这个事件在监听器被移除后触发。当这个事件被触发时，监听器已经从事件的监听器数组中被移除了。
