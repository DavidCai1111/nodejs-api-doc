# StringDecoder#


### 稳定度: 2 - 稳定

通过`require('string_decoder')`来使用这个模块。`StringDecoder`解码一个`buffer`为一个字符串。它是一个`buffer.toString()`的简单接口，但是提供了utf8的额外支持。

```js
var StringDecoder = require('string_decoder').StringDecoder;
var decoder = new StringDecoder('utf8');

var cent = new Buffer([0xC2, 0xA2]);
console.log(decoder.write(cent));

var euro = new Buffer([0xE2, 0x82, 0xAC]);
console.log(decoder.write(euro));
```

#### Class: StringDecoder#

接受一个单独的参数，即编码，默认为utf8。

#### decoder.write(buffer)#

返回被解码的字符串。

#### decoder.end()#

返回遗留在`buffer`中的所有末端字节。
