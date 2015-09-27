# Stream#

### 稳定度: 2 - 稳定
流是一个被`node.js`内部的许多对象所实现的抽象接口。例如一个发往HTTP服务器的请求是一个留，`stdout`也是一个流。流可以是可读的，可写的或双向的。所有的流都是`EventEmitter`实例。

你可以通过`require('stream')`来取货`Stream`的基类。其中包括了`Readable`流，`Writable`流，`Duplex`流和`Transform`流的基类。

此文档分为三个章节。第一章节解释了在你的编程中使用流时需要的API。如果你不需要实现你自己的流式API，你可以在这里停止。

第二章节解释了你在构建你自己的流时需要的API，这些API是为了方便你这么做而设计的。

第三章节深入讲述了流的工作机制，包括一些内部的机制和函数，你不应该去改动它们除非你知道你在做什么。

### 面向流消费者的API
流可以是可读的，可写的，或双工的。

所有的流都是`EventEmitters`。但是它们也各自有一些独特的方法和属性，这取决于它们是可读流，可写流或双工流。

如果一个流同时是可读的和可写的，那么表示它实现了以下所有的方法和事件。所以，这些API同时也涵盖`Duplex`或`Transform`流，即使它们的实现可能有些不同。

在你程序中，为了消费流而去实现流接口不是必须的。如果你确实正在你的程序中实现流接口，请参考下一章节`面向流实现者的API`。

几乎所有`node.js`程序，不论多简单，都使用了流。下面是一个在`node.js`是使用流的例子：

```js
var http = require('http');

var server = http.createServer(function (req, res) {
  // req is an http.IncomingMessage, which is a Readable Stream
  // res is an http.ServerResponse, which is a Writable Stream

  var body = '';
  // we want to get the data as utf8 strings
  // If you don't set an encoding, then you'll get Buffer objects
  req.setEncoding('utf8');

  // Readable streams emit 'data' events once a listener is added
  req.on('data', function (chunk) {
    body += chunk;
  });

  // the end event tells you that you have entire body
  req.on('end', function () {
    try {
      var data = JSON.parse(body);
    } catch (er) {
      // uh oh!  bad json!
      res.statusCode = 400;
      return res.end('error: ' + er.message);
    }

    // write back something interesting to the user:
    res.write(typeof data);
    res.end();
  });
});

server.listen(1337);

// $ curl localhost:1337 -d '{}'
// object
// $ curl localhost:1337 -d '"foo"'
// string
// $ curl localhost:1337 -d 'not json'
// error: Unexpected token o
```

#### Class: stream.Readable#

可读流接口是一个你可以从之读取数据的数据源的抽象。换句话说，数据从可读流而来。

除非你指示已经准备好接受数据，否则可读流不会开始发生数据。

可读流有两个“模式”：流动模式和暂停模式。当在流动模式时，数据由底层系统读出，并且会尽快地提供给你的程序。当在暂停模式时，你必须调用`stream.read()`方法来获取数据块。流默认是暂停模式。

注意：如果`data`事件没有被绑定监听器，并且没有导流（pipe）目标，并且流被切换到了流动模式，那么数据将会被丢失。

你可以通过下面任意一个做法切换到流动模式：

 - 添加一个`data`事件的监听器来监听数据。

 - 调用`resume()`方法来明确开启流动模式。

 - 调用`pipe()`方法将数据导入一个可写流。

你可以同意下面任意一种方法切换回暂停模式：

 - 如果没有导流（pipe）目标，调用`pause()`方法。

 - 如果有导流（pipe）目标，移除所有的`data`事件监听器，并且通过`unpipe()`方法移除所有导流目标。

注意，由于为了向后兼任的原因，移除`data`事件的监听器将不会自动暂停流。同样的，如果有导流目标，调用`pause()`方法将不会保证目标流排空并请求更多数据时保持暂停。

一些内置的可读流例子：

 - 客户端的HTTP请求
 - 服务端的HTTP响应
 - 文件系统读取流
 - `zlib`流
 - `crypto`流
 - tcp sockets
 - 子进程的stdout和stderr
 - `process.stdin`

#### Event: 'readable'#

当一个数据块能可以从流中被读出时，会触发一个`readable`事件。

某些情况下，监听一个`readable`事件会导致一些将要被读出的数据从底层系统进入内部缓冲，如果它没有准备好。

```js
var readable = getReadableStreamSomehow();
readable.on('readable', function() {
  // there is some data to read now
});
```

当内部缓冲被排空时，一旦有更多数据，`readable`事件会再次触发。

#### Event: 'data'#

 - chunk Buffer | String 数据块
 
为一个没有被暂停的流添加一个`data`事件的监听器会使其切换到流动模式。之后数据会被尽快得传递给用户。

如果你只是想尽快得从流中取得所有数据，这是最好的方式。

```js
var readable = getReadableStreamSomehow();
readable.on('data', function(chunk) {
  console.log('got %d bytes of data', chunk.length);
});
```

#### Event: 'end'#

当没有更多可读的数据时这个事件会被触发。

注意，除非数据被完全消费，`end`事件才会触发。这可以通过切换到流动模式，或重复调用`read()`方法。

```js
var readable = getReadableStreamSomehow();
readable.on('data', function(chunk) {
  console.log('got %d bytes of data', chunk.length);
});
readable.on('end', function() {
  console.log('there will be no more data.');
});
```

#### Event: 'close'#

当底层资源（如源头的文件描述符）被关闭时触发。不是所有的流都会触发这个事件。

#### Event: 'error'#

 - Error Object

当接受数据时有错误发生，会触发此事件。

#### readable.read([size])#

 - size Number 可选，指定读取数据的数量
 - Return String | Buffer | null


`read()`方法从内部缓冲中取出数据并返回它。如果没有可用数据，那么将返回`null`。

如果你传递了一个`size`参数，那么它将返回指定字节的数据。如果`size`参数的字节数不可用，那么将返回`null`。

如果你不指定`size`参数，那么将会返回内部缓冲中的所有数据。

这个方法只能在暂定模式中被调用。在流动模式下，这个方法会被自动地重复调用，知道内部缓冲被排空。

```js
var readable = getReadableStreamSomehow();
readable.on('readable', function() {
  var chunk;
  while (null !== (chunk = readable.read())) {
    console.log('got %d bytes of data', chunk.length);
  }
});
```

如果这个方法返回一个数据块，那么它也会触发`data`事件。

#### readable.setEncoding(encoding)#

 - encoding String 使用的编码
 - Return: this
 
调用这个函数会导致流返回指定编码的字符串而不是`Buffer`对象。例如，如果你调用`readable.setEncoding('utf8')`，那么输出的数据将被解释为UTF-8数据，并且作为字符串返回。如果你调用了`readable.setEncoding('hex')`，那么数据将被使用十六进制字符串的格式编码。

该方法可以正确地处理多字节字符。如果你只是简单地直接取出缓冲并且对它们调用`buf.toString(encoding)`，将会导致错位。如果你想使用字符串读取数据，请使用这个方法。

```js
var readable = getReadableStreamSomehow();
readable.setEncoding('utf8');
readable.on('data', function(chunk) {
  assert.equal(typeof chunk, 'string');
  console.log('got %d characters of string data', chunk.length);
});
```

#### readable.resume()#

 - Return: this
 
这个方法将会让可读流继续触发`data`事件。

这个方法将会使流切换至流动模式。如果你不想消费流中的数据，但你想监听它的`end`事件，你可以通过调用`readable.resume()`来打开数据流。

```js
var readable = getReadableStreamSomehow();
readable.resume();
readable.on('end', function() {
  console.log('got to the end, but did not read anything');
});
```

#### readable.pause()#

 - Return: this

这个方法会使一个处于流动模式的流停止触发`data`事件，并切换至暂停模式。所有可用的数据将仍然存在于内部缓冲中。

```js
var readable = getReadableStreamSomehow();
readable.on('data', function(chunk) {
  console.log('got %d bytes of data', chunk.length);
  readable.pause();
  console.log('there will be no more data for 1 second');
  setTimeout(function() {
    console.log('now data will start flowing again');
    readable.resume();
  }, 1000);
});
```

#### readable.isPaused()#

 - Return: Boolean

这个方法会返回流是否被客户端代码所暂停（调用`readable.pause()`，并且没有在之后调用`readable.resume()`）。

```js
var readable = new stream.Readable

readable.isPaused() // === false
readable.pause()
readable.isPaused() // === true
readable.resume()
readable.isPaused() // === false
```

#### readable.pipe(destination[, options])#

 - destination Writable Stream 写入数据的目标
 - __options Object__
  - end Boolean 当读取者结束时结束写入者。默认为`true`。

这个方法会取出可读流中所有的数据，并且将之写入指定的目标。这个方法会自动调节流量，所以当快速读取可读流时目标不会溢出。

可以将数据安全地导流至多个目标。

```js
var readable = getReadableStreamSomehow();
var writable = fs.createWriteStream('file.txt');
// All the data from readable goes into 'file.txt'
readable.pipe(writable);
```

这个函数返回目标流，所以你可以链式调用`pipe()`：

```js
var r = fs.createReadStream('file.txt');
var z = zlib.createGzip();
var w = fs.createWriteStream('file.txt.gz');
r.pipe(z).pipe(w);
```

例子，模仿UNIX的`cat`命令：

```js
process.stdin.pipe(process.stdout);
```

默认情况下，当源流触发`end`事件时，目标流会被调用`end()`方法，然后目标就不再是可写的了。将传递`{ end: false }`作为`options`参数，将保持目标流开启。

例子，保持被写入的流开启，所以“Goodbye”可以在末端被写入：

```js
reader.pipe(writer, { end: false });
reader.on('end', function() {
  writer.end('Goodbye\n');
});
```

注意，不论指定任何`options`参数，`process.stderr`和`process.stdout`在程序退出前永远不会被关闭。

#### readable.unpipe([destination])#

- destination Writable Stream 可选，指定解除导流的流

这方法会移除之前调用`pipe()`方法所设置的钩子。

如果没有指定目标，那么所有的导流都会被移除。

如果指定了目标，但是并没有为目标设置导流，那么什么都不会发生。

```js
var readable = getReadableStreamSomehow();
var writable = fs.createWriteStream('file.txt');
// All the data from readable goes into 'file.txt',
// but only for the first second
readable.pipe(writable);
setTimeout(function() {
  console.log('stop writing to file.txt');
  readable.unpipe(writable);
  console.log('manually close the file stream');
  writable.end();
}, 1000);
```

#### readable.unshift(chunk)#

 - chunk Buffer | String 要插回读取队列开头的数据块。

该方法在许多场景中都很有用，比如一个流正在被一个解析器消费，解析器可能需要将某些刚拉取出的数据“逆消费”回来源，以便流能将它传递给其它消费者。

如果你发现你必须经常在你的程序中调用`stream.unshift(chunk)`，你应该考虑实现一个`Transform`流（参阅下文的面向流实现者的API）。

```js
// Pull off a header delimited by \n\n
// use unshift() if we get too much
// Call the callback with (error, header, stream)
var StringDecoder = require('string_decoder').StringDecoder;
function parseHeader(stream, callback) {
  stream.on('error', callback);
  stream.on('readable', onReadable);
  var decoder = new StringDecoder('utf8');
  var header = '';
  function onReadable() {
    var chunk;
    while (null !== (chunk = stream.read())) {
      var str = decoder.write(chunk);
      if (str.match(/\n\n/)) {
        // found the header boundary
        var split = str.split(/\n\n/);
        header += split.shift();
        var remaining = split.join('\n\n');
        var buf = new Buffer(remaining, 'utf8');
        if (buf.length)
          stream.unshift(buf);
        stream.removeListener('error', callback);
        stream.removeListener('readable', onReadable);
        // now the body of the message can be read from the stream.
        callback(null, header, stream);
      } else {
        // still reading the header.
        header += str;
      }
    }
  }
}
```

#### readable.wrap(stream)#

 - stream Stream 一个“旧式”可读流

`Node.js` v0.10 以及之前版本的流没有完全包含如今的所有的流API（更多的信息请参阅下文的“兼容性”）。

如果你正在使用一个老旧的`node.js`库，它触发`data`时间并且有一个仅作查询用途的`pause()`方法，那么你可以调用`wrap()`方法来创建一个使用“旧式”流作为数据源的可读流。

你几乎不会用到这个函数，它的存在仅是为了老旧的`node.js`程序和库交互。

例子：

```js
var OldReader = require('./old-api-module.js').OldReader;
var oreader = new OldReader;
var Readable = require('stream').Readable;
var myReader = new Readable().wrap(oreader);

myReader.on('readable', function() {
  myReader.read(); // etc.
});
```

#### Class: stream.Writable#

可写流接口是一个你可以向其写入数据的目标的抽象。

一些内部的可写流例子：

 - 客户端的http请求
 - 服务端的http响应
 - 文件系统写入流
 - `zlib`流
 - `crypto`流
 - tcp `socket`
 - 子进程`stdin`
 - `process.stdout`，`process.stderr`
 
#### writable.write(chunk[, encoding][, callback])#

 - chunk String | Buffer 要写入的数据
 - encoding String 编码，如果数据块是字符串
 - callback Function 当数据块写入完毕后调用的回调函数
 - Returns: Boolean 如果被全部处理则返回`true`

该方法向底层系统写入数据，并且当数据被全部处理后调用指定的回调函数。

返回值指示了你是否可以立刻写入数据。如果数据需要被内部缓冲，会返回`false`。否则返回`true`。

返回值经供参考。即使返回`false`，你仍可以继续写入数据。但是，写入的数据将会被缓冲在内存里，所以最好不要这样做。应该在写入更多数据前等待`drain`事件。

#### Event: 'drain'#

如果一个`writable.write(chunk)`调用返回了`false`，那么`drain`事件会指示出可以继续向流写入数据的时机。

```js
// Write the data to the supplied writable stream 1MM times.
// Be attentive to back-pressure.
function writeOneMillionTimes(writer, data, encoding, callback) {
  var i = 1000000;
  write();
  function write() {
    var ok = true;
    do {
      i -= 1;
      if (i === 0) {
        // last time!
        writer.write(data, encoding, callback);
      } else {
        // see if we should continue, or wait
        // don't pass the callback, because we're not done yet.
        ok = writer.write(data, encoding);
      }
    } while (i > 0 && ok);
    if (i > 0) {
      // had to stop early!
      // write some more once it drains
      writer.once('drain', write);
    }
  }
}
```

#### writable.cork()#

强制滞留所有写入。

滞留的数据会在调用`.uncork()`或`.end()`方法后被写入。

#### writable.uncork()#

写入在调用`.cork()`方法所有被滞留的数据。

#### writable.setDefaultEncoding(encoding)#

 - encoding String 新的默认编码

设置一个可写流的默认编码。

#### writable.end([chunk][, encoding][, callback])#

 - chunk String | Buffer 可选，写入的数据
 - encoding String 编码，如果数据块是字符串
 - callback Function 可选，回调函数

当没有更多可写的数据时，调用这个方法。如果指定了回调函数，那么会被添加为`finish`事件的监听器。

在调用了`end()`后调用`write()`会导致一个错误。

```js
// write 'hello, ' and then end with 'world!'
var file = fs.createWriteStream('example.txt');
file.write('hello, ');
file.end('world!');
// writing more now is not allowed!
```

#### Event: 'finish'#

当调用了`end()`方法，并且所有的数据都被写入了底层系统，这个事件会被触发。

```js
var writer = getWritableStreamSomehow();
for (var i = 0; i < 100; i ++) {
  writer.write('hello, #' + i + '!\n');
}
writer.end('this is the end\n');
writer.on('finish', function() {
  console.error('all writes are now complete.');
});
```

#### Event: 'pipe'#

 - src Readable Stream 对这个可写流进行导流的源可读流

这个事件将会在可读流被一个可写流使用`pipe()`方法进行导流时触发。

```js
var writer = getWritableStreamSomehow();
var reader = getReadableStreamSomehow();
writer.on('pipe', function(src) {
  console.error('something is piping into the writer');
  assert.equal(src, reader);
});
reader.pipe(writer);
```

#### Event: 'unpipe'#

 - src Readable Stream 对这个可写流停止导流的源可读流

当可读流对其调用`unpipe()`方法，在源可读流的目标集合中删除这个可写流，这个事件将会触发。

```js
var writer = getWritableStreamSomehow();
var reader = getReadableStreamSomehow();
writer.on('unpipe', function(src) {
  console.error('something has stopped piping into the writer');
  assert.equal(src, reader);
});
reader.pipe(writer);
reader.unpipe(writer);
```

#### Event: 'error'#

 - Error object

在写入数据或导流发生错误时触发。

#### Class: stream.Duplex#

双工是同时实现了可读流与可写流的借口。它的用处请参阅下文。

内部双工流的例子：

 - tcp `socket`
 - `zlib`流
 - `crypto`流
 
#### Class: stream.Transform#

转换流是一种输出由输入计算所得的栓共流。它们同时集成了可读流与可写流的借口。它们的用处请参阅下文。

内部转换流的例子：

 - `zlib`流
 - `crypto`流
 
### 面向流实现者的API

实现所有种类的流的模式都是一样的：

 1. 为你的子类继承合适的父类（`util.inherits`非常合适于做这个）。
 2. 为了保证内部机制被正确初始化，在你的构造函数中调用合适的父类构造函数。
 3. 实现一个或多个特定的方法，参阅下文。

被扩展的类和要实现的方法取决于你要编写的流类的类型：

| 用途      | 类           | 需要实现的方法 |
| ------------- |:-------------:| -----:|
| 只读     | Readable | _read |
| 只写      | Writable      |   _write, _writev |
| 可读以及可写 | Duplex      |    _read, _write, _writev |
| 操作被写入数据，然后读出结果 | Transform      |    _transform, _flush |

在你的实现代码中，非常重要的一点是永远不要调用上文的面向流消费者的API。否则，你在程序中消费你的流接口时可能有潜在的副作用。

#### Class: stream.Readable#

`stream.Readable`是一个被设计为需要实现底层的`_read(size)`方法的抽象类。

请参阅上文的面向流消费者的API来了解如何在程序中消费流。以下解释了如果在你的程序中实现可读流。

例子：一个计数流

这是一个可读流的基础例子。它从1到1，000，000递增数字，然后结束。

```js
var Readable = require('stream').Readable;
var util = require('util');
util.inherits(Counter, Readable);

function Counter(opt) {
  Readable.call(this, opt);
  this._max = 1000000;
  this._index = 1;
}

Counter.prototype._read = function() {
  var i = this._index++;
  if (i > this._max)
    this.push(null);
  else {
    var str = '' + i;
    var buf = new Buffer(str, 'ascii');
    this.push(buf);
  }
};
```


例子：简单协议 v1 （次优）

这类似于上文中提到的`parseHeader `函数，但是使用一个自定义流实现。另外，注意这个实现不将流入的数据转换为字符串。

更好地实现是作为一个转换流实现，请参阅下文更好地实现。

```js
// A parser for a simple data protocol.
// The "header" is a JSON object, followed by 2 \n characters, and
// then a message body.
//
// NOTE: This can be done more simply as a Transform stream!
// Using Readable directly for this is sub-optimal.  See the
// alternative example below under the Transform section.

var Readable = require('stream').Readable;
var util = require('util');

util.inherits(SimpleProtocol, Readable);

function SimpleProtocol(source, options) {
  if (!(this instanceof SimpleProtocol))
    return new SimpleProtocol(source, options);

  Readable.call(this, options);
  this._inBody = false;
  this._sawFirstCr = false;

  // source is a readable stream, such as a socket or file
  this._source = source;

  var self = this;
  source.on('end', function() {
    self.push(null);
  });

  // give it a kick whenever the source is readable
  // read(0) will not consume any bytes
  source.on('readable', function() {
    self.read(0);
  });

  this._rawHeader = [];
  this.header = null;
}

SimpleProtocol.prototype._read = function(n) {
  if (!this._inBody) {
    var chunk = this._source.read();

    // if the source doesn't have data, we don't have data yet.
    if (chunk === null)
      return this.push('');

    // check if the chunk has a \n\n
    var split = -1;
    for (var i = 0; i < chunk.length; i++) {
      if (chunk[i] === 10) { // '\n'
        if (this._sawFirstCr) {
          split = i;
          break;
        } else {
          this._sawFirstCr = true;
        }
      } else {
        this._sawFirstCr = false;
      }
    }

    if (split === -1) {
      // still waiting for the \n\n
      // stash the chunk, and try again.
      this._rawHeader.push(chunk);
      this.push('');
    } else {
      this._inBody = true;
      var h = chunk.slice(0, split);
      this._rawHeader.push(h);
      var header = Buffer.concat(this._rawHeader).toString();
      try {
        this.header = JSON.parse(header);
      } catch (er) {
        this.emit('error', new Error('invalid simple protocol data'));
        return;
      }
      // now, because we got some extra data, unshift the rest
      // back into the read queue so that our consumer will see it.
      var b = chunk.slice(split);
      this.unshift(b);

      // and let them know that we are done parsing the header.
      this.emit('header', this.header);
    }
  } else {
    // from there on, just provide the data to our consumer.
    // careful not to push(null), since that would indicate EOF.
    var chunk = this._source.read();
    if (chunk) this.push(chunk);
  }
};

// Usage:
// var parser = new SimpleProtocol(source);
// Now parser is a readable stream that will emit 'header'
// with the parsed header data.
```

#### new stream.Readable([options])#

 - __options Object__
  - highWaterMark Number 在停止从底层资源读取之前，在内部缓冲中存储的最大字节数。默认为16kb，对于`objectMode`则是16
  - encoding String 如果被指定，那么缓冲将被利用指定编码解码为字符串，默认为`null`
  - objectMode Boolean 是否该流应该表现如一个对象的流。意思是说`stream.read(n)`返回一个单独的对象而不是一个大小为`n`的`Buffer`，默认为`false`
  
在实现了`Readable`类的类中，请确保调用了`Readable`构造函数，这样缓冲设置才能被正确的初始化。

#### readable._read(size)#

 - size Number 异步读取数据的字节数
 
注意：实现这个函数，而不要直接调用这个函数。

这个函数不应该被直接调用。它应该被子类实现，并且仅被`Readable`类的内部方法调用。

所有的可读流都必须实现这个方法用来从底层资源中获取数据。

这个函数有一个下划线前缀，因为它对于类是内部的，并应该直接被用户的程序调用。你应在你的拓展类里覆盖这个方法。

当数据可用时，调用`readable.push(chunk)`方法将之推入读取队列。如果方法返回`false`，那么你应当停止读取。当`_read`方法再次被调用，你应当推入更多数据。

参数`size`仅作查询。“read”调用返回数据的实现可以通过这个参数来知道应当抓取多少数据；其余与之无关的实现，比如TCP或TLS，则可忽略这个参数，并在可用时返回数据。例如，没有必要“等到”`size`个字节可用时才调用`stream.push(chunk)`。

#### readable.push(chunk[, encoding])#

 - chunk Buffer | null | String 被推入读取队列的数据块
 - encoding String 字符串数据块的编码。必须是一个合法的`Buffer`编码，如'utf8'或'ascii'
 - return Boolean 是否应该继续推入

注意：这个函数应该被`Readable`流的实现者调用，而不是消费者。

`_read()`函数在至少调用一次`push(chunk)`方法前，不会被再次调用。

`Readable`类通过在`readable`事件触发时，调用`read()`方法将数据推入 之后用于读出数据的读取队列 来工作。

`push()`方法需要明确地向读取队列中插入数据。如果它的参数为`null`，那么它将发送一个数据结束信号（`EOF`）。

这个API被设计为尽可能的灵活。例如，你可能正在包装一个有`pause/resume`机制和一个数据回调函数的低级别源。那那些情况下，你可以通过以下方式包装这些低级别源：

```js
// source is an object with readStop() and readStart() methods,
// and an `ondata` member that gets called when it has data, and
// an `onend` member that gets called when the data is over.

util.inherits(SourceWrapper, Readable);

function SourceWrapper(options) {
  Readable.call(this, options);

  this._source = getLowlevelSourceObject();
  var self = this;

  // Every time there's data, we push it into the internal buffer.
  this._source.ondata = function(chunk) {
    // if push() returns false, then we need to stop reading from source
    if (!self.push(chunk))
      self._source.readStop();
  };

  // When the source ends, we push the EOF-signaling `null` chunk
  this._source.onend = function() {
    self.push(null);
  };
}

// _read will be called when the stream wants to pull more data in
// the advisory size argument is ignored in this case.
SourceWrapper.prototype._read = function(size) {
  this._source.readStart();
};
```

#### Class: stream.Writable#

`stream.Writable`是一个被设计为需要实现底层的`_write(chunk, encoding, callback) `方法的抽象类。

请参阅上文的面向流消费者的API来了解如何在程序中消费流。以下解释了如果在你的程序中实现可写流。

#### new stream.Writable([options])#

 - __options Object__
  - highWaterMark Number `write()`方法开始返回`false`的缓冲级别。默认为16kb，对于`objectMode`流则是`16`
  - decodeStrings Boolean 是否在传递给`write()`方法前将字符串解码成`Buffer`。默认为`true`
  - objectMode Boolean 是否`write(anyObj)`为一个合法操作。如果设置为`true`你可以写入任意数据而不仅是`Buffer`或字符串数据。默认为`false`
 
在实现了`Writable `类的类中，请确保调用了`Writable `构造函数，这样缓冲设置才能被正确的初始化。

#### writable._write(chunk, encoding, callback)#

 - chunk Buffer | String 将要被写入的数据块。除非`decodeStrings `配置被设置为`false`，否则将一直是一个`buffer`
 - encoding String 如果数据块是一个字符串，那么这就是编码的类型。如果是一个`buffer`，那么则会忽略它
 - callback Function 当你处理完给定的数据块后调用这个函数

所有的`Writable`流的实现都必须提供一个`_write()`方法来给底层资源传输数据。


这个函数不应该被直接调用。它应该被子类实现，并且仅被`Writable`类的内部方法调用。

回调函数使用标准的`callback(error)`模式来表示这个写操作成功或发生了错误。

如果构造函数选项中设置了`decodeStrings`标志，那么数据块将是一个字符串而不是一个`Buffer`，编码将会决定字符串的类型。这个是为了帮助处理编码字符串的实现。如果你没有明确地将`decodeStrings`选项设为`false`，那么你会安全地忽略`encoding`参数，并且数据块是`Buffer`形式。

这个函数有一个下划线前缀，因为它对于类是内部的，并应该直接被用户的程序调用。你应在你的拓展类里覆盖这个方法。

#### writable._writev(chunks, callback)#

 - chunks Array 将被写入的数据块数组。其中每一个数据都有如下格式：`{ chunk: ..., encoding: ... }`
 - callback Function 当你处理完给定的数据块后调用这个函数

注意：这个函数不应该被直接调用。它应该被子类实现，并且仅被`Writable`类的内部方法调用。

这个函数对于你的实现是完全可选的。大多数情况下它是不必的。如果实现，它会被以所有滞留在写入队列中的数据块调用。

#### Class: stream.Duplex#

一个“双工”流既是可读的，又是可写的。如TCP`socket`连接。

注意，和你实现`Readable`或`Writable`流时一样，`stream.Duplex`是一个被设计为需要实现底层的`_read(size)`和`_write(chunk, encoding, callback)`方法的抽象类。

由于`JavaScript`并不具备多继承能力，这个类是继承于`Readable`类，并寄生于`Writable`类。所以为了实现这个类，用户需要同时实现低级别的`_read(n)`方法和低级别的`_write(chunk, encoding, callback)`方法。

#### new stream.Duplex(options)#

 - __options Object__ 同时传递给`Writable`和`Readable`构造函数。并且包含以下属性：
  - allowHalfOpen Boolean 默认为`true`。如果设置为`false`，那么流的可读的一端结束时可写的一端也会自动结束，反之亦然。
  - readableObjectMode Boolean 默认为`false`，为流的可读的一端设置`objectMode `。当`objectMode`为`true`时没有效果。
  - writableObjectMode Boolean 默认为`false`，为流的可写的一端设置`objectMode `。当`objectMode`为`true`时没有效果。
 
在实现了`Duplex`类的类中，请确保调用了`Duplex`构造函数，这样缓冲设置才能被正确的初始化。

#### Class: stream.Transform#

“转换”流是一个输出于输入存在对应关系的双工流，如一个`zilib`流或一个`crypto`流。

输出和输出并不需要有相同的大小，相同的数据块数或同时到达。例如，一个哈希流只有一个单独数据块的输出当输入结束时。一个`zlib`流的输出比其输入小得多或大得多。

除了实现`_read()`方法和`_write()`方法，转换流还必须实现`_transform()`方法，并且可选地实现`_flush()`方法（参阅下文）。

#### new stream.Transform([options])#

 - options Object 同时传递给`Writable`和`Readable`构造函数。
  
在实现了`Transform `类的类中，请确保调用了`Transform `构造函数，这样缓冲设置才能被正确的初始化。

#### transform._transform(chunk, encoding, callback)#

 - chunk Buffer | String 将要被写入的数据块。除非`decodeStrings`配置被设置为`false`，否则将一直是一个`buffer`
 - encoding String 如果数据块是一个字符串，那么这就是编码的类型。如果是一个buffer，那么则会忽略它
 - callback Function 当你处理完给定的数据块后调用这个函数

这个函数不应该被直接调用。它应该被子类实现，并且仅被`Transform `类的内部方法调用。

所有`Transform`流的实现都必须提供一个`_transform `方法来接受输入和产生输出。

在`Transform `类中，`_transform`可以做需要做的任何事，如处理需要写入的字节，将它们传递给可写端，异步I/O，等等。

调用`transform.push(outputChunk)`0次或多次来从输入的数据块产生输出，取决于你想从这个数据块中输出多少数据作为结果。

仅当目前的数据块被完全消费后，才会调用回调函数。注意，对于某些特殊的输入可能会没有输出。如果你将数据作为第二个参数传入回调函数，那么数据将被传递给`push`方法。换句话说，下面的两个例子是相等的：

```js
transform.prototype._transform = function (data, encoding, callback) {
  this.push(data);
  callback();
}

transform.prototype._transform = function (data, encoding, callback) {
  callback(null, data);
}
```

这个函数有一个下划线前缀，因为它对于类是内部的，并应该直接被用户的程序调用。你应在你的拓展类里覆盖这个方法。

#### transform._flush(callback)#

 - callback Function 当你排空了所有剩余数据后，这个回调函数会被调用

注意：这个函数不应该被直接调用。它应该被子类实现，并且仅被`Transform `类的内部方法调用。

在一些情景中，你的转换操作需要在流的末尾多发生一点点数据。例如，一个`Zlib`压缩流会存储一些内部状态以便它能优化压缩输出。但是在最后，它需要尽可能好得处理这些留下的东西来使数据完整。

在这种情况中，您可以实现一个`_flush`方法，它会在最后被调用，在所有写入数据被消费、但在触发`end`表示可读端到达末尾之前。和`_transform`一样，只需在写入操作完成时适当地调用`transform.push(chunk)`零或多次。

这个函数有一个下划线前缀，因为它对于类是内部的，并应该直接被用户的程序调用。你应在你的拓展类里覆盖这个方法。

#### Events: 'finish' 和 'end'#

`finish`和`end`事件分别来自于父类`Writable`和`Readable`。`finish`事件在`end()`方法被调用以及所有的输入被`_transform`方法处理后触发。`end`事件在所有的在`_flush`方法的回调函数被调用后的数据被输出后触发。

#### Example: SimpleProtocol 解释器 v2#

上文中的简单协议解释器可以简单地通过高级别的`Transform`流更好地实现。与上文例子中的`parseHeader`和`SimpleProtocol v1`相似。

在这个例子中，没有从参数中提供输入，然后将它导流至解释器中，这更符合`node.js`的使用习惯。

```j
var util = require('util');
var Transform = require('stream').Transform;
util.inherits(SimpleProtocol, Transform);

function SimpleProtocol(options) {
  if (!(this instanceof SimpleProtocol))
    return new SimpleProtocol(options);

  Transform.call(this, options);
  this._inBody = false;
  this._sawFirstCr = false;
  this._rawHeader = [];
  this.header = null;
}

SimpleProtocol.prototype._transform = function(chunk, encoding, done) {
  if (!this._inBody) {
    // check if the chunk has a \n\n
    var split = -1;
    for (var i = 0; i < chunk.length; i++) {
      if (chunk[i] === 10) { // '\n'
        if (this._sawFirstCr) {
          split = i;
          break;
        } else {
          this._sawFirstCr = true;
        }
      } else {
        this._sawFirstCr = false;
      }
    }

    if (split === -1) {
      // still waiting for the \n\n
      // stash the chunk, and try again.
      this._rawHeader.push(chunk);
    } else {
      this._inBody = true;
      var h = chunk.slice(0, split);
      this._rawHeader.push(h);
      var header = Buffer.concat(this._rawHeader).toString();
      try {
        this.header = JSON.parse(header);
      } catch (er) {
        this.emit('error', new Error('invalid simple protocol data'));
        return;
      }
      // and let them know that we are done parsing the header.
      this.emit('header', this.header);

      // now, because we got some extra data, emit this first.
      this.push(chunk.slice(split));
    }
  } else {
    // from there on, just provide the data to our consumer as-is.
    this.push(chunk);
  }
  done();
};

// Usage:
// var parser = new SimpleProtocol();
// source.pipe(parser)
// Now parser is a readable stream that will emit 'header'
// with the parsed header data.
```

#### Class: stream.PassThrough#

这是一个`Transform`流的实现。将输入的流简单地传递给输出。它的主要目的是用来演示和测试，但它在某些需要构建特殊流的情况下可能有用。

### 简化的构造器API

可以简单的构造流而不使用继承。

这可以通过调用合适的方法作为构造函数和参数来实现：

例子：

#### Readable#

```js
var readable = new stream.Readable({
  read: function(n) {
    // sets this._read under the hood
  }
});
```

#### Writable#

```js
var writable = new stream.Writable({
  write: function(chunk, encoding, next) {
    // sets this._write under the hood
  }
});

// or

var writable = new stream.Writable({
  writev: function(chunks, next) {
    // sets this._writev under the hood
  }
});
```

#### Duplex#

```js
var duplex = new stream.Duplex({
  read: function(n) {
    // sets this._read under the hood
  },
  write: function(chunk, encoding, next) {
    // sets this._write under the hood
  }
});

// or

var duplex = new stream.Duplex({
  read: function(n) {
    // sets this._read under the hood
  },
  writev: function(chunks, next) {
    // sets this._writev under the hood
  }
});
```

#### Transform#

```js
var transform = new stream.Transform({
  transform: function(chunk, encoding, next) {
    // sets this._transform under the hood
  },
  flush: function(done) {
    // sets this._flush under the hood
  }
});
```

### 流：内部细节
#### 缓冲

`Writable`流和`Readable`流都会分别在一个内部的叫`_writableState.buffer`或`_readableState.buffer`的对象里缓冲数据。

潜在的被缓冲的数据量取决于被传递给构造函数的`highWaterMark`参数。

在`Readable`流中，当其的实现调用`stream.push(chunk)`时就会发生缓冲。如果流的消费者没有调用`stream.read()`，那么数据就会保留在内部队列中直到它被消费。

在`Writable`流中，当用户重复调用`stream.write(chunk)`时就会发生缓冲，甚至是当`write()`返回`false`时。

流，尤其是`pipe()`方法的初衷，是限制数据的滞留量在一个可接受的水平，这样才使得不同传输速度的来源和目标不会淹没可用的内存。

#### stream.read(0)#

在一些情况下，你想不消费任何数据而去触发一次底层可读流机制的刷新。你可以调用`stream.read(0)`，它总是返回`null`。

如果内部的读缓冲量在`highWaterMark`之下，并且流没有正在读取，那么调用`read(0)`将会触发一次低级别的`_read`调用。

几乎永远没有必须这么做。但是，你可能会在`node.js`的`Readable`流类的内部代码的几处看到这个。

#### stream.push('')#

推入一个0字节的字符串或`Buffer`（不处于对象模式）有一个有趣的副作用。因为这是一个`stream.push()`的调用，它将会结束读取进程。但是，它不添加任何数据到可读缓冲中，所以没有任何用户可消费的数据。

在极少的情况下，你当下没有数据可以提供，但你的消费者同过调用`stream.read(0)`来得知合适再次检查。在这样的情况下，你可以调用`stream.push('')`。

至今为止，这个功能的唯一使用之处是在`tls.CryptoStream`类中，它将在`node.js`的1.0版本中被废弃。如果你发现你不得不使用`stream.push('')`，请考虑使用另外的方式。因为这几乎表示发生了某些可怕的错误。

### 与旧版本的`Node.js`的兼容性

在`Node.js`的0.10版本之前，可读流接口非常简单，并且功能和功用都不强。

 - `data`事件会立刻触发，而不是等待你调用`read()`方法。如果你需要进行一些`I/O`操作来决定是否处理数据，那么你只能将数据存储在某些缓冲区中以防数据流失。
 - `pause()`仅供查询，并不保证生效。这意味着你还是要准备接收`data`事件在流已经处于暂停模式中时。

在`node.js` v1.0 和`Node.js` v0.10中，下文所述的`Readable`类添加进来。为了向后兼容性，当一个`data`事件的监听器被添加时或`resume()`方法被调用时，可读流切换至流动模式。其作用是，即便您不使用新的`read()`方法和`readable`事件，您也不必担心丢失数据块。

大多数程序都会保持功能正常，但是，以下有一些边界情况：

 - 没有添加任何`data`事件
 - 从未调用`resume()`方法
 - 流没有被导流至任何可写的目标

例如，考虑以下代码：

```js
// WARNING!  BROKEN!
net.createServer(function(socket) {

  // we add an 'end' method, but never consume the data
  socket.on('end', function() {
    // It will never get here.
    socket.end('I got your message (but didnt read it)\n');
  });

}).listen(1337);
```

在`Node.js` v0.10前，到来的信息数据会被简单地丢弃。但是在`node.js` v1.0 和`Node.js` v0.10后，`socket`会被永远暂停。

解决方案是调用`resume()`方法来开启数据流：

```js
// Workaround
net.createServer(function(socket) {

  socket.on('end', function() {
    socket.end('I got your message (but didnt read it)\n');
  });

  // start the flow of data, discarding it.
  socket.resume();

}).listen(1337);
```

除了新的`Readable`流切换至流动模式之外，在v0.10之前的流可以被使用`wrap()`方法包裹。

#### 对象模式

通常情况下，流仅操作字符串和`Buffer`。

处于对象模式中的流除了`Buffer`和字符串外，还能读出普通的`JavaScirpt`值。

处于对象模式中的可读流在调用`stream.read(size)`后只会返回单个项目，不论`size`参数是什么。

处于对象模式中的可写流总是忽略`stream.write(data, encoding)`中的`encoding`参数。

对于处于对象模式中的流，特殊值`null`仍然保留它的特殊意义。也就是说，对于对象模式的可读流，`stream.read()`返回一个`null`仍意味着没有更多的数据了，并且`stream.push(null)`会发送一个文件末端信号（`EOF`）。

核心`node.js`中没有流是对象模式的。这个模式仅仅供用户的流库使用。

你应当在子类的构造函数的`options`参数对象中设置对象模式。在流的过程中设置对象模式时不安全的。

对于双工流，可以分别得通过`readableObjectMode`和`writableObjectMode`设置可读端和可写端。这些配置可以被用来通过转换流实现解释器和序列化器。

```js
var util = require('util');
var StringDecoder = require('string_decoder').StringDecoder;
var Transform = require('stream').Transform;
util.inherits(JSONParseStream, Transform);

// Gets \n-delimited JSON string data, and emits the parsed objects
function JSONParseStream() {
  if (!(this instanceof JSONParseStream))
    return new JSONParseStream();

  Transform.call(this, { readableObjectMode : true });

  this._buffer = '';
  this._decoder = new StringDecoder('utf8');
}

JSONParseStream.prototype._transform = function(chunk, encoding, cb) {
  this._buffer += this._decoder.write(chunk);
  // split on newlines
  var lines = this._buffer.split(/\r?\n/);
  // keep the last partial line buffered
  this._buffer = lines.pop();
  for (var l = 0; l < lines.length; l++) {
    var line = lines[l];
    try {
      var obj = JSON.parse(line);
    } catch (er) {
      this.emit('error', er);
      return;
    }
    // push the parsed object out to the readable consumer
    this.push(obj);
  }
  cb();
};

JSONParseStream.prototype._flush = function(cb) {
  // Just handle any leftover
  var rem = this._buffer.trim();
  if (rem) {
    try {
      var obj = JSON.parse(rem);
    } catch (er) {
      this.emit('error', er);
      return;
    }
    // push the parsed object out to the readable consumer
    this.push(obj);
  }
  cb();
};
```