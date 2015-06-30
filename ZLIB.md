# Zlib#

### 稳定度: 2 - 稳定

要获取这个模块，你可以通过：

```js
var zlib = require('zlib');
```

它提供了`Gzip/Gunzip`，`Deflate/Inflate`和`DeflateRaw/InflateRaw`类的绑定。每个类都有相同的选项，并且都是可读/可写流。

#### 例子

可以通过将一个`fs.ReadStream`的数据导入一个`zlib`流，然后导入一个`fs.WriteStream`，来压缩或解压缩一个文件。

```js
var gzip = zlib.createGzip();
var fs = require('fs');
var inp = fs.createReadStream('input.txt');
var out = fs.createWriteStream('input.txt.gz');

inp.pipe(gzip).pipe(out);
```

通过使用便捷方法，可以在一个步骤里完成压缩或解压缩数据。

```js
var input = '.................................';
zlib.deflate(input, function(err, buffer) {
  if (!err) {
    console.log(buffer.toString('base64'));
  }
});

var buffer = new Buffer('eJzT0yMAAGTvBe8=', 'base64');
zlib.unzip(buffer, function(err, buffer) {
  if (!err) {
    console.log(buffer.toString());
  }
});
```

如果要在HTTP客户端或服务器上使用这个模块，在请求时需要带上`accept-encoding`头，在响应时需要带上`content-encoding`头。

注意，这些例子都只是非常简单的展示了一些基本的概念。`zlib`编码的开销是非常昂贵的，并且结果需要被缓存。更多关于速度/内存/压缩的权衡，请参阅下文的`内存使用调优`。

```js
// client request example
var zlib = require('zlib');
var http = require('http');
var fs = require('fs');
var request = http.get({ host: 'izs.me',
                         path: '/',
                         port: 80,
                         headers: { 'accept-encoding': 'gzip,deflate' } });
request.on('response', function(response) {
  var output = fs.createWriteStream('izs.me_index.html');

  switch (response.headers['content-encoding']) {
    // or, just use zlib.createUnzip() to handle both cases
    case 'gzip':
      response.pipe(zlib.createGunzip()).pipe(output);
      break;
    case 'deflate':
      response.pipe(zlib.createInflate()).pipe(output);
      break;
    default:
      response.pipe(output);
      break;
  }
});

// server example
// Running a gzip operation on every request is quite expensive.
// It would be much more efficient to cache the compressed buffer.
var zlib = require('zlib');
var http = require('http');
var fs = require('fs');
http.createServer(function(request, response) {
  var raw = fs.createReadStream('index.html');
  var acceptEncoding = request.headers['accept-encoding'];
  if (!acceptEncoding) {
    acceptEncoding = '';
  }

  // Note: this is not a conformant accept-encoding parser.
  // See http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.3
  if (acceptEncoding.match(/\bdeflate\b/)) {
    response.writeHead(200, { 'content-encoding': 'deflate' });
    raw.pipe(zlib.createDeflate()).pipe(response);
  } else if (acceptEncoding.match(/\bgzip\b/)) {
    response.writeHead(200, { 'content-encoding': 'gzip' });
    raw.pipe(zlib.createGzip()).pipe(response);
  } else {
    response.writeHead(200, {});
    raw.pipe(response);
  }
}).listen(1337);
```

#### zlib.createGzip([options])#

根据一个`options`，返回一个新的`Gzip`对象。

#### zlib.createGunzip([options])#

根据一个`options`，返回一个新的`Gunzip`对象。

#### zlib.createDeflate([options])#

根据一个`options`，返回一个新的`Deflate`对象。

#### zlib.createInflate([options])#

根据一个`options`，返回一个新的`Inflate`对象。

#### zlib.createDeflateRaw([options])#

根据一个`options`，返回一个新的`DeflateRaw`对象。

#### zlib.createInflateRaw([options])#

根据一个`options`，返回一个新的`InflateRaw`对象。

#### zlib.createUnzip([options])#

根据一个`options`，返回一个新的`Unzip`对象。

#### Class: zlib.Zlib#

这个类未被`zlib`模块暴露。它之所以会出现在这里，是因为它是`compressor/decompressor`类的基类。

#### zlib.flush([kind], callback)#

`kind`默认为`zlib.Z_FULL_FLUSH`。

冲刷等待中的数据。不要轻率地调用这个方法，过早的冲刷会给压缩算法带来消极影响。

#### zlib.params(level, strategy, callback)#

动态地更新压缩等级和压缩策略。只适用于`deflate`算法。

#### zlib.reset()#

将`compressor/decompressor`重置为默认值。只使用于`inflate`和`deflate`算法。

#### Class: zlib.Gzip#

使用`gzip`压缩数据。

#### Class: zlib.Gunzip#

解压一个`gzip`流。

#### Class: zlib.Deflate#

使用`deflate`压缩数据。

#### Class: zlib.Inflate#

解压一个`deflate`流。

#### Class: zlib.DeflateRaw#

使用`deflate`压缩数据，不添加`zlib`头。

#### Class: zlib.InflateRaw#

解压一个原始`deflate`流。

#### Class: zlib.Unzip#

通过自动探测头信息，解压`Gzip`或`Deflate`压缩流。

#### 便捷方法

所有的方法接受一个字符串或一个`buffer`作为第一个参数，并且第二个参数是一个可选的 `zlib`类的配置，并且会以`callback(error, result)`的形式执行提供的回调函数。

每一个方法都有一个同步版本，除去回调函数，它们接受相同的参数。

#### zlib.deflate(buf[, options], callback)#
#### zlib.deflateSync(buf[, options])#

使用`Deflate`压缩一个字符串。

#### zlib.deflateRaw(buf[, options], callback)#
#### zlib.deflateRawSync(buf[, options])#

使用`DeflateRaw`压缩一个字符串。

#### zlib.gzip(buf[, options], callback)#
#### zlib.gzipSync(buf[, options])#

使用`Gzip`压缩一个字符串。

#### zlib.gunzip(buf[, options], callback)#
#### zlib.gunzipSync(buf[, options])#

使用`Gunzip`压缩一个字符串。

#### zlib.inflate(buf[, options], callback)#
#### zlib.inflateSync(buf[, options])#

使用`Inflate`压缩一个字符串。

#### zlib.inflateRaw(buf[, options], callback)#
#### zlib.inflateRawSync(buf[, options])#

使用`InflateRaw`压缩一个字符串。

#### zlib.unzip(buf[, options], callback)#
#### zlib.unzipSync(buf[, options])#

使用`Unzip`压缩一个字符串。

#### Options#

每一个类都接受一个`options`对象。所有的`options`对象都是可选的。

注意一些选项只与压缩相关，会被解压缩类忽略：

 - flush (默认：`zlib.Z_NO_FLUSH`)
 - chunkSize (默认：`16*1024`)
 - windowBits
 - level (仅用于压缩)
 - memLevel (仅用于压缩)
 - strategy (仅用于压缩)
 - dictionary (仅用于`deflate/inflate`，默认为空目录)


参阅`http://zlib.net/manual.html#Advanced`中`deflateInit2`和`inflateInit2`的描述来获取更多信息。


#### 内存使用调优

来自`zlib/zconf.h`，将其修改为`io.js`的用法：

默认的内存要求（字节）为：

```js
(1 << (windowBits+2)) +  (1 << (memLevel+9))
```

换言之：`windowBits=15`的128K 加上 `menLevel = 8`（默认值）的128K 加上其他小对象的一些字节。

例子，如果你想要将默认内存需求从256K减少至128K，将选项设置为：

```js
{ windowBits: 14, memLevel: 7 }
```

当然，它会降低压缩等级（没有免费的午餐）。

`inflate`的内存需求（字节）为：

```js
1 << windowBits
```

换言之：`windowBits=15`（默认值）的32K加上其他小对象的一些字节。

这是内部输出缓冲外的`chunkSize`大小，默认为16K。

`zlib`压缩的速度动态得受设置的压缩等级的影响。高的等级会带来更好地压缩效果，但是花费的时间更长。低的等级会带来更少的压缩效果，但是更快。

通常，更高的内存使用选项意味着`io.js`会调用`zlib`更少次数，因为在一次单独的写操作中它可以处理更多的数据。所以，这是影响速度和内存占用的另一个因素。

#### 常量

所有在`zlib.h`中定义的常量，都也被定义在了`require('zlib')`中。大多数操作中，你都将不会用到它们。它们出现在这里只是为了让你对它们的存在不套感到惊讶。该章节几乎完全来自`zlib`文件。更多详情请参阅`http://zlib.net/manual.html#Constants`。


允许的冲刷值：

```
zlib.Z_NO_FLUSH
zlib.Z_PARTIAL_FLUSH
zlib.Z_SYNC_FLUSH
zlib.Z_FULL_FLUSH
zlib.Z_FINISH
zlib.Z_BLOCK
zlib.Z_TREES
```

`compression/decompression`函数的返回码。负值代表错误，正值代表特殊但是正常的事件：

```
zlib.Z_OK
zlib.Z_STREAM_END
zlib.Z_NEED_DICT
zlib.Z_ERRNO
zlib.Z_STREAM_ERROR
zlib.Z_DATA_ERROR
zlib.Z_MEM_ERROR
zlib.Z_BUF_ERROR
zlib.Z_VERSION_ERROR
```

压缩等级：

```
zlib.Z_NO_COMPRESSION
zlib.Z_BEST_SPEED
zlib.Z_BEST_COMPRESSION
zlib.Z_DEFAULT_COMPRESSION
```

压缩策略：

```
zlib.Z_FILTERED
zlib.Z_HUFFMAN_ONLY
zlib.Z_RLE
zlib.Z_FIXED
zlib.Z_DEFAULT_STRATEGY
```

`data_type`域的可能值：

```
zlib.Z_BINARY
zlib.Z_TEXT
zlib.Z_ASCII
zlib.Z_UNKNOWN
```

`deflate`压缩方法（当前版本只支持这一个）：

```
zlib.Z_DEFLATED
```

用于初始化`zalloc`，`zfree`，`opaque`：

```
zlib.Z_NULL
```