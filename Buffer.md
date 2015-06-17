# Buffer#

### 稳定度: 2 - 稳定
纯粹的`JavaScript`是Unicode友好的，但是不能很好地处理二进制数据。当处理TCP流或者文件流时，操作八进制流式必要的。`io.js`提供了多种策略来操作，创建和使用八进制流。

原始的数据被存储在`Buffer`类的实例中，一个`Buffer`类似于一个整数数组但是使用了V8堆之外的内存分配。一个`Buffer`不能被改变大小。

`Buffer`类是全局的，所以它是少数的不用`require('buffer')`就能使用的对象之一。

`Buffer`和`JavaScript`字符串对象之间的转换需要指定一个明确地编码方法。以下是一些不同的字符串编码。

'ascii' - 仅供7位的ASCII数据使用，这个编码方法非常的快速，而且会剥离过高的位（如果有设置）。

'utf8' - 多字节编码的字符。许多web页面和一些其他文档都使用UTF-8编码。

'utf16le' - 2或4个字节，`little endian`编码字符。支持(U+10000 到 U+10FFFF)的代理对。

'ucs2' - 'utf16le'的别名。

'base64' - Base64 字符串编码。

'binary' - 一种通过使用每个字符的前八位来将二进制数据解码为字符串的方式。这个编码方法已经不被推荐使用，在处理`Buffer`对象时应避免使用它，这个编码将会在`io.js`的未来版本中移除。

'hex' - 把每个字节编码成2个十六进制字符。

从一个`Buffer`创建一个类型数组(typed array)遵循以下的说明：

1，`Buffer`的内存是被复制的，不是共享的。

2，`Buffer`的内存被解释当做一个数组，而不是一个字节数组(byte array)。换言之，`new Uint32Array(new Buffer([1,2,3,4]))`创建了一个4个元素（[1,2,3,4]）的`Uint32Array`,不是一个只有一个元素（[0x1020304] 或 [0x4030201]）的`Uint32Array`。


注意： Node.js v0.8 只是简单得在`array.buffer`中保留了`buffer`的引用，而不是复制一份。


#### Class: Buffer#
Buffer类是一个全局类用于直接处理二进制数据。它的实例可以被多种途径构建。

#### new Buffer(size)#

 - size Number

分配一个大小为指定`size`的八位字节的新`buffer`。注意，`size`不能超过kMaxLength，否则一个`RangeError`将会被抛出。

#### new Buffer(array)#

 - array Array

使用一个八进制数组分配一个新的`buffer`。

#### new Buffer(buffer)#

 - buffer Buffer

将传递的`buffer`复制进一个新的Buffer实例。

#### new Buffer(str[, encoding])#

 - str String - 传入buffer的字符串
 - encoding String - 可选，使用的编码

根据给定的`str`创建一个新的`buffer`，编码默认是UTF-8。

#### Class Method: Buffer.isEncoding(encoding)#

 - encoding String 将要被测试的编码，返回`true`若此编码是合法的，否则返回`false`。

#### Class Method: Buffer.isBuffer(obj)#

 - obj Object
 - Return: Boolean

测试`obj`是否为一个`buffer`。

#### Class Method: Buffer.byteLength(string[, encoding])

 - string String
 - encoding String, 可选， 默认： 'utf8'
 - Return: Number

给出`string`的实际字节长度。编码默认为UTF-8。这与`String.prototype.length`不同，因为`String.prototype.length`只返回字符串中字符的数量。

例子:

```js
str = '\u00bd + \u00bc = \u00be';

console.log(str + ": " + str.length + " characters, " +
  Buffer.byteLength(str, 'utf8') + " bytes");

// ½ + ¼ = ¾: 9 characters, 12 bytes
```

#### Class Method: Buffer.concat(list[, totalLength])#

 - list Array 需要被连接的Buffer对象
 - totalLength Number 将要被连接的buffers的
 - Returns 连接完毕的buffer

若`list`为空，或`totalLength`为0，那么将返回一个长度为0的buffer。
若`list`只有一个元素，那么这个元素将被返回。
若`list`有超过一个元素，那么将创建一个新的Buffer实例。
如果`totalLength`没有被提供，那么将会从`list`中计算读取。但是，这增加了函数的一个额外的循环，所以如果直接长度那么性能会更好。

#### Class Method: Buffer.compare(buf1, buf2)#

 - buf1 Buffer
 - buf2 Buffer
与buf1.compare(buf2)相同. 对于排序一个Buffers的数组非常有用:
```js
var arr = [Buffer('1234'), Buffer('0123')];
arr.sort(Buffer.compare);
```

#### buf.length#

 - Return Number 这个buffer的字节长度。注意这不一定是这个buffer中的内容长度。它是这个buffer对象所分配内存大小，并不会随着buffer的内容的改变而改变

```js
buf = new Buffer(1234);

console.log(buf.length);
buf.write("some string", 0, "ascii");
console.log(buf.length);

// 1234
// 1234
```

虽然`buffer`的`length`属性并不是不可变的，改变`length`属性的值可能会使之变成`undefined`或引起一些不一致的行为。希望去改变`buffer`的`length`的应用应当把它视作一个只读的值，并且使用`buf.slice`来创建一个新的`buffer`。

```js
buf = new Buffer(10);
buf.write("abcdefghj", 0, "ascii");
console.log(buf.length); // 10
buf = buf.slice(0,5);
console.log(buf.length); // 5
```

#### buf.write(string[, offset][, length][, encoding])#

 - string String 准备被写入buffer的数据
 - offset Number 可选，默认为0
 - length Number 可选，默认为 `buffer.length - offset`
 - encoding String 可选，默认为 'utf8'

从指定的偏移位置(offset)使用给定的编码向buffer中写入字符串，偏移位置默认为0，编码默认为UTF8。长度为将要写入的字符串的字节大小。返回被写入的八进制流的大小。如果buffer没有足够的空间写入整个字符串，那么它将只会写入一部分。`length`参数默认为`buffer.length - offset`，这个方法将不会只写入字符的一部分。

```js
buf = new Buffer(256);
len = buf.write('\u00bd + \u00bc = \u00be', 0);
console.log(len + " bytes: " + buf.toString('utf8', 0, len));
```

#### buf.writeUIntLE(value, offset, byteLength[, noAssert])#

#### buf.writeUIntBE(value, offset, byteLength[, noAssert])#

#### buf.writeIntLE(value, offset, byteLength[, noAssert])#

#### buf.writeIntBE(value, offset, byteLength[, noAssert])#

 - value {Number} 将要被写入buffer的字节
 - offset {Number} `0 <= offset <= buf.length`
 - byteLength {Number} `0 < byteLength <= 6`
 - noAssert {Boolean} 默认为`false`
 - Return: {Number}

根据指定的偏移位置(offset)和`byteLength`将`value`写入buffer。最高支持48位的精确度。例子：
```js
var b = new Buffer(6);
b.writeUIntBE(0x1234567890ab, 0, 6);
// <Buffer 12 34 56 78 90 ab>
```

将`noAssert`设置为true将会跳过`value`和`offset`的检验，默认为`false`。

#### buf.readUIntLE(offset, byteLength[, noAssert])#

#### buf.readUIntBE(offset, byteLength[, noAssert])#

#### buf.readIntLE(offset, byteLength[, noAssert])#

#### buf.readIntBE(offset, byteLength[, noAssert])#

 - offset {Number} `0 <= offset <= buf.length`
 - byteLength {Number} `0 < byteLength <= 6`
 - noAssert {Boolean} 默认为`false`
 - Return: {Number}

一个普遍的用来作数值读取的方法，最高支持48位的精确度。例子：

```js
var b = new Buffer(6);
b.writeUint16LE(0x90ab, 0);
b.writeUInt32LE(0x12345678, 2);
b.readUIntLE(0, 6).toString(16);  // Specify 6 bytes (48 bits)
// output: '1234567890ab'
```

将`noAssert`设置为true将会跳过`value`和`offset`的检验，这意味着`offset`将可能超过buffer的结束位置，默认为`false`。

#### buf.toString([encoding][, start][, end])#

 - encoding String, 可选，默认为 'utf8'
 - start Number, 可选，默认为 0
 - end Number, 可选默认为 `buffer.length`

从编码的buffer数据中使用指定的编码解码并返回结果字符串。如果`encoding`为`undefined`或`null`，那么`encoding`将默认为UTF8。`start`和`end`参数默认为`0`和`buffer.length`。

```js
buf = new Buffer(26);
for (var i = 0 ; i < 26 ; i++) {
  buf[i] = i + 97; // 97 is ASCII a
}
buf.toString('ascii'); // outputs: abcdefghijklmnopqrstuvwxyz
buf.toString('ascii',0,5); // outputs: abcde
buf.toString('utf8',0,5); // outputs: abcde
buf.toString(undefined,0,5); // encoding defaults to 'utf8', outputs abcde
```

#### buf.toJSON()#

返回一个Buffer实例的JSON形式。`JSON.stringify`被隐式得调用当转换Buffer实例时。

例子:

```js
var buf = new Buffer('test');
var json = JSON.stringify(buf);

console.log(json);
// '{"type":"Buffer","data":[116,101,115,116]}'

var copy = JSON.parse(json, function(key, value) {
    return value && value.type === 'Buffer'
      ? new Buffer(value.data)
      : value;
  });

console.log(copy);
// <Buffer 74 65 73 74>
```

#### buf[index]#

获取或设置指定位置的八位字节。这个值是指单个字节，所有合法范围在`0x00`到`0xFF` 或 `0` 到 `255`。

例子：复制一个ASCII字符串到一个buffer，一次一个字节：

```js
str = "io.js";
buf = new Buffer(str.length);

for (var i = 0; i < str.length ; i++) {
  buf[i] = str.charCodeAt(i);
}

console.log(buf);

// io.js
```

#### buf.equals(otherBuffer)#

 - otherBuffer Buffer

返回一个布尔值表示是否`buf`与`otherBuffer`具有相同的字节。

#### buf.compare(otherBuffer)#

 - otherBuffer Buffer

返回一个数字表示在排序上`buf`在`otherBuffer`之前，之后或相同。

#### buf.copy(targetBuffer[, targetStart][, sourceStart][, sourceEnd])#

 - targetBuffer Buffer 将要进行复制的Buffer
 - targetStart Number 可选，默认为 0
 - sourceStart Number 可选，默认为 0
 - sourceEnd Number 可选，默认为 `buffer.length`

从`buf`中的指定范围复制数据到`targetBuffer`中的指定范围，它们是可以重叠的。

例子：创建两个Buffer，然后复制buf1的第16字节到19字节到buf2，buf2的偏移位置从第8字节开始：

```js
buf1 = new Buffer(26);
buf2 = new Buffer(26);

for (var i = 0 ; i < 26 ; i++) {
  buf1[i] = i + 97; // 97 is ASCII a
  buf2[i] = 33; // ASCII !
}

buf1.copy(buf2, 8, 16, 20);
console.log(buf2.toString('ascii', 0, 25));

// !!!!!!!!qrst!!!!!!!!!!!!!
```

例子： 创建一个单独的Buffer，然后复制数据到自身的一个重叠的范围。

```js
buf = new Buffer(26);

for (var i = 0 ; i < 26 ; i++) {
  buf[i] = i + 97; // 97 is ASCII a
}

buf.copy(buf, 0, 4, 10);
console.log(buf.toString());

// efghijghijklmnopqrstuvwxyz
```

#### buf.slice([start][, end])#

 - start Number 可选，默认为 0
 - end Number 可选，默认为 `buffer.length`
 - 返回一个和旧的buffer引用了相同内存的新的buffer，但是被`start`和`end`参数所偏移和裁剪。

修改这个新的buffer的切片，也会改变内存中原来的buffer。

例子： 创建一个ASCII字母的Buffer，然后对其进行`slice`，然后修改源Buffer上的一个字节：

```js
var buf1 = new Buffer(26);

for (var i = 0 ; i < 26 ; i++) {
  buf1[i] = i + 97; // 97 is ASCII a
}

var buf2 = buf1.slice(0, 3);
console.log(buf2.toString('ascii', 0, buf2.length));
buf1[0] = 33;
console.log(buf2.toString('ascii', 0, buf2.length));

// abc
// !bc
```

#### buf.indexOf(value[, byteOffset])#

 - value String Buffer或Number
 - byteOffset Number 可选，默认为 0
 - Return: Number

行为和Array.indexOf()相似。接受一个字符串，Buffer或数字。字符串被解释为UTF8编码，Buffer将使用整个buffer，所以如果要比较部分的Buffer请使用`Buffer.slice()`，数字的范围需在0到255之间。

#### buf.readUInt8(offset[, noAssert])#

 - offset Number
 - noAssert Boolean 可选，默认为 `false`
 - Return: Number

根据制定偏移量从buffer中读取一个无符号8位整数。

将`noAssert`设置为true将会跳过`value`和`offset`的检验，这意味着`offset`将可能超过buffer的结束位置，默认为`false`。

例子:

```js
var buf = new Buffer(4);

buf[0] = 0x3;
buf[1] = 0x4;
buf[2] = 0x23;
buf[3] = 0x42;

for (ii = 0; ii < buf.length; ii++) {
  console.log(buf.readUInt8(ii));
}

// 0x3
// 0x4
// 0x23
// 0x42
```

#### buf.readUInt16LE(offset[, noAssert])#

#### buf.readUInt16BE(offset[, noAssert])#

 - offset Number
 - noAssert Boolean 可选，默认为 `false`
 - Return: Number

根据制定偏移量从buffer中根据特定的`endian`字节序读取一个无符号16位整数。

将`noAssert`设置为true将会跳过`value`和`offset`的检验，这意味着`offset`将可能超过buffer的结束位置，默认为`false`。

例子:

```js
var buf = new Buffer(4);

buf[0] = 0x3;
buf[1] = 0x4;
buf[2] = 0x23;
buf[3] = 0x42;

console.log(buf.readUInt16BE(0));
console.log(buf.readUInt16LE(0));
console.log(buf.readUInt16BE(1));
console.log(buf.readUInt16LE(1));
console.log(buf.readUInt16BE(2));
console.log(buf.readUInt16LE(2));

// 0x0304
// 0x0403
// 0x0423
// 0x2304
// 0x2342
// 0x4223
```

#### buf.readUInt32LE(offset[, noAssert])#

#### buf.readUInt32BE(offset[, noAssert])#

 - offset Number
 - noAssert Boolean 可选，默认为 `false`
 - Return: Number

根据制定偏移量从buffer中根据特定的`endian`字节序读取一个无符号32位整数。

将`noAssert`设置为true将会跳过`value`和`offset`的检验，这意味着`offset`将可能超过buffer的结束位置，默认为`false`。

例子：

```js
var buf = new Buffer(4);

buf[0] = 0x3;
buf[1] = 0x4;
buf[2] = 0x23;
buf[3] = 0x42;

console.log(buf.readUInt32BE(0));
console.log(buf.readUInt32LE(0));

// 0x03042342
// 0x42230403
```

#### buf.readInt8(offset[, noAssert])#

 - offset Number
 - noAssert Boolean 可选，默认为 `false`
 - Return: Number

根据制定偏移量从buffer中读取一个有符号8位整数。

将`noAssert`设置为true将会跳过`value`和`offset`的检验，这意味着`offset`将可能超过buffer的结束位置，默认为`false`。

运作和`buffer.readUInt8`相同，除非buffer内容中有包含了作为2的补码的有符号值。

#### buf.readInt16LE(offset[, noAssert])#

#### buf.readInt16BE(offset[, noAssert])#

 - offset Number
 - noAssert Boolean 可选，默认为 `false`
 - Return: Number

根据制定偏移量从buffer中根据特定的`endian`字节序读取一个有符号16位整数。

将`noAssert`设置为true将会跳过`value`和`offset`的检验，这意味着`offset`将可能超过buffer的结束位置，默认为`false`。

运作和`buffer.readUInt16`相同，除非buffer内容中有包含了作为2的补码的有符号值。

#### buf.readInt32LE(offset[, noAssert])#

#### buf.readInt32BE(offset[, noAssert])#

 - offset Number
 - noAssert Boolean 可选，默认为 `false`
 - Return: Number

根据制定偏移量从buffer中根据特定的`endian`字节序读取一个有符号32位整数。

将`noAssert`设置为true将会跳过`value`和`offset`的检验，这意味着`offset`将可能超过buffer的结束位置，默认为`false`。

运作和`buffer.readUInt32`相同，除非buffer内容中有包含了作为2的补码的有符号值。

#### buf.readFloatLE(offset[, noAssert])#

#### buf.readFloatBE(offset[, noAssert])#

 - offset Number
 - noAssert Boolean 可选，默认为 `false`
 - Return: Number

根据制定偏移量从buffer中根据特定的`endian`字节序读取一个32位浮点数。

将`noAssert`设置为true将会跳过`value`和`offset`的检验，这意味着`offset`将可能超过buffer的结束位置，默认为`false`。

例子：

```js
var buf = new Buffer(4);

buf[0] = 0x00;
buf[1] = 0x00;
buf[2] = 0x80;
buf[3] = 0x3f;

console.log(buf.readFloatLE(0));

// 0x01
```

#### buf.readDoubleLE(offset[, noAssert])#

#### buf.readDoubleBE(offset[, noAssert])#

 - offset Number
 - noAssert Boolean 可选，默认为 `false`
 - Return: Number

根据制定偏移量从buffer中根据特定的`endian`字节序读取一个64位双精度数。

将`noAssert`设置为true将会跳过`value`和`offset`的检验，这意味着`offset`将可能超过buffer的结束位置，默认为`false`。

例子：

```js
var buf = new Buffer(8);

buf[0] = 0x55;
buf[1] = 0x55;
buf[2] = 0x55;
buf[3] = 0x55;
buf[4] = 0x55;
buf[5] = 0x55;
buf[6] = 0xd5;
buf[7] = 0x3f;

console.log(buf.readDoubleLE(0));

// 0.3333333333333333
```

#### buf.writeUInt8(value, offset[, noAssert])#

 - value Number
 - offset Number
 - noAssert Boolean 可选，默认为 `false`

向`buffer`的指定偏移位置写入`value`。
注意，`value`必须是一个合法的无符号8位整形数。

将`noAssert`设置为`true`将跳过`value`和`offset`的验证。这意味着`value`可能会过大，或者`offset`超过`buffer`的末尾导致`value`被丢弃，这个参数除非你十分有把握否则你不应去使用它，默认为`false`。

例子：

```js
var buf = new Buffer(4);
buf.writeUInt8(0x3, 0);
buf.writeUInt8(0x4, 1);
buf.writeUInt8(0x23, 2);
buf.writeUInt8(0x42, 3);

console.log(buf);

// <Buffer 03 04 23 42>
```

#### buf.writeUInt16LE(value, offset[, noAssert])#

#### buf.writeUInt16BE(value, offset[, noAssert])#

 - value Number
 - offset Number
 - noAssert Boolean 可选，默认为 `false`

向`buffer`的指定偏移位置根据特定的`endian`字节序写入`value`。
注意，`value`必须是一个合法的无符号16位整形数。

将`noAssert`设置为`true`将跳过`value`和`offset`的验证。这意味着`value`可能会过大，或者`offset`超过`buffer`的末尾导致`value`被丢弃，这个参数除非你十分有把握否则你不应去使用它，默认为`false`。

例子：

```js
var buf = new Buffer(4);
buf.writeUInt16BE(0xdead, 0);
buf.writeUInt16BE(0xbeef, 2);

console.log(buf);

buf.writeUInt16LE(0xdead, 0);
buf.writeUInt16LE(0xbeef, 2);

console.log(buf);

// <Buffer de ad be ef>
// <Buffer ad de ef be>
```

#### buf.writeUInt32LE(value, offset[, noAssert])#

#### buf.writeUInt32BE(value, offset[, noAssert])#

 - value Number
 - offset Number
 - noAssert Boolean 可选，默认为 `false`

向`buffer`的指定偏移位置根据特定的`endian`字节序写入`value`。
注意，`value`必须是一个合法的无符号32位整形数。

将`noAssert`设置为`true`将跳过`value`和`offset`的验证。这意味着`value`可能会过大，或者`offset`超过`buffer`的末尾导致`value`被丢弃，这个参数除非你十分有把握否则你不应去使用它，默认为`false`。

例子：

```js
var buf = new Buffer(4);
buf.writeUInt32BE(0xfeedface, 0);

console.log(buf);

buf.writeUInt32LE(0xfeedface, 0);

console.log(buf);

// <Buffer fe ed fa ce>
// <Buffer ce fa ed fe>
```

#### buf.writeInt8(value, offset[, noAssert])#

 - value Number
 - offset Number
 - noAssert Boolean 可选，默认为 `false`

向`buffer`的指定偏移位置中写入`value`。
注意，`value`必须是一个合法的无符号32位整形数。

将`noAssert`设置为`true`将跳过`value`和`offset`的验证。这意味着`value`可能会过大，或者`offset`超过`buffer`的末尾导致`value`被丢弃，这个参数除非你十分有把握否则你不应去使用它，默认为`false`。

运作和`buffer.writeUInt8`相同，除非buffer内容中有包含了作为2的补码的有符号值。

#### buf.writeInt16LE(value, offset[, noAssert])#

#### buf.writeInt16BE(value, offset[, noAssert])#

 - value Number
 - offset Number
 - noAssert Boolean 可选，默认为 `false`

向`buffer`的指定偏移位置根据特定的`endian`字节序写入`value`。
注意，`value`必须是一个合法的有符号16位整形数。

将`noAssert`设置为`true`将跳过`value`和`offset`的验证。这意味着`value`可能会过大，或者`offset`超过`buffer`的末尾导致`value`被丢弃，这个参数除非你十分有把握否则你不应去使用它，默认为`false`。

运作和`buffer.writeUInt16`相同，除非buffer内容中有包含了作为2的补码的有符号值。

#### buf.writeInt32LE(value, offset[, noAssert])#

#### buf.writeInt32BE(value, offset[, noAssert])#

value Number
offset Number
noAssert Boolean, Optional, Default: false
Writes value to the buffer at the specified offset with specified endian format. Note, value must be a valid signed 32 bit integer.

Set noAssert to true to skip validation of value and offset. This means that value may be too large for the specific function and offset may be beyond the end of the buffer leading to the values being silently dropped. This should not be used unless you are certain of correctness. Defaults to false.

Works as buffer.writeUInt32*, except value is written out as a two's complement signed integer into buffer.

buf.writeFloatLE(value, offset[, noAssert])#

buf.writeFloatBE(value, offset[, noAssert])#

value Number
offset Number
noAssert Boolean, Optional, Default: false
Writes value to the buffer at the specified offset with specified endian format. Note, behavior is unspecified if value is not a 32 bit float.

Set noAssert to true to skip validation of value and offset. This means that value may be too large for the specific function and offset may be beyond the end of the buffer leading to the values being silently dropped. This should not be used unless you are certain of correctness. Defaults to false.

Example:

var buf = new Buffer(4);
buf.writeFloatBE(0xcafebabe, 0);

console.log(buf);

buf.writeFloatLE(0xcafebabe, 0);

console.log(buf);

// <Buffer 4f 4a fe bb>
// <Buffer bb fe 4a 4f>
buf.writeDoubleLE(value, offset[, noAssert])#

buf.writeDoubleBE(value, offset[, noAssert])#

value Number
offset Number
noAssert Boolean, Optional, Default: false
Writes value to the buffer at the specified offset with specified endian format. Note, value must be a valid 64 bit double.

Set noAssert to true to skip validation of value and offset. This means that value may be too large for the specific function and offset may be beyond the end of the buffer leading to the values being silently dropped. This should not be used unless you are certain of correctness. Defaults to false.

Example:

var buf = new Buffer(8);
buf.writeDoubleBE(0xdeadbeefcafebabe, 0);

console.log(buf);

buf.writeDoubleLE(0xdeadbeefcafebabe, 0);

console.log(buf);

// <Buffer 43 eb d5 b7 dd f9 5f d7>
// <Buffer d7 5f f9 dd b7 d5 eb 43>
buf.fill(value[, offset][, end])#

value
offset Number, Optional
end Number, Optional
Fills the buffer with the specified value. If the offset (defaults to 0) and end (defaults to buffer.length) are not given it will fill the entire buffer.

var b = new Buffer(50);
b.fill("h");
buffer.values()#

Creates iterator for buffer values (bytes). This function is called automatically when buffer is used in a for..of statement.

buffer.keys()#

Creates iterator for buffer keys (indices).

buffer.entries()#

Creates iterator for [index, byte] arrays.

buffer.INSPECT_MAX_BYTES#
Number, Default: 50
How many bytes will be returned when buffer.inspect() is called. This can be overridden by user modules.

Note that this is a property on the buffer module returned by require('buffer'), not on the Buffer global, or a buffer instance.

ES6 iteration#
Buffers can be iterated over using for..of syntax:

var buf = new Buffer([1, 2, 3]);

for (var b of buf)
  console.log(b)

// 1
// 2
// 3
Additionally, buffer.values(), buffer.keys() and buffer.entries() methods can be used to create iterators.

Class: SlowBuffer#
Returns an un-pooled Buffer.

In order to avoid the garbage collection overhead of creating many individually allocated Buffers, by default allocations under 4KB are sliced from a single larger allocated object. This approach improves both performance and memory usage since v8 does not need to track and cleanup as many Persistent objects.

In the case where a developer may need to retain a small chunk of memory from a pool for an indeterminate amount of time it may be appropriate to create an un-pooled Buffer instance using SlowBuffer and copy out the relevant bits.

// need to keep around a few small chunks of memory
var store = [];

socket.on('readable', function() {
  var data = socket.read();
  // allocate for retained data
  var sb = new SlowBuffer(10);
  // copy the data into the new allocation
  data.copy(sb, 0, 0, 10);
  store.push(sb);
});
Though this should be used sparingly and only be a last resort after a developer has actively observed undue memory retention in their applications.