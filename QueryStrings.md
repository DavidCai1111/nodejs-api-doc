# Query String#


### 稳定度: 2 - 稳定

这个模块提供了处理 查询字符串 的工具。它提供了以下方法：

#### querystring.stringify(obj[, sep][, eq][, options])#

序列化一个对象为一个查询字符串。可以可选地覆盖默认的分隔符（`'&'`）和赋值符号（`'='`）。

`options`对象可以包含`encodeURIComponent`属性（默认为`querystring.escape`），它被用来在需要时，将字符串编码为非utf-8编码。

例子：

```js
querystring.stringify({ foo: 'bar', baz: ['qux', 'quux'], corge: '' })
// returns
'foo=bar&baz=qux&baz=quux&corge='

querystring.stringify({foo: 'bar', baz: 'qux'}, ';', ':')
// returns
'foo:bar;baz:qux'

// Suppose gbkEncodeURIComponent function already exists,
// it can encode string with `gbk` encoding
querystring.stringify({ w: '中文', foo: 'bar' }, null, null,
  { encodeURIComponent: gbkEncodeURIComponent })
// returns
'w=%D6%D0%CE%C4&foo=bar'
```

#### querystring.parse(str[, sep][, eq][, options])#

反序列化一个查询字符串为一个对象。可以可选地覆盖默认的分隔符（`'&'`）和赋值符号（`'='`）。

`options`可以包含`maxKeys`属性（默认为`1000`）。它被用来限制被处理的键。将其设置为`0`会移除限制。

`options`可以包含`decodeURIComponent`属性（默认为`querystring.unescape`），它被用来在需要时，解码非uft8编码字符串。

例子：

```js
querystring.parse('foo=bar&baz=qux&baz=quux&corge')
// returns
{ foo: 'bar', baz: ['qux', 'quux'], corge: '' }

// Suppose gbkDecodeURIComponent function already exists,
// it can decode `gbk` encoding string
querystring.parse('w=%D6%D0%CE%C4&foo=bar', null, null,
  { decodeURIComponent: gbkDecodeURIComponent })
// returns
{ w: '中文', foo: 'bar' }
```

#### querystring.escape#

`querystring.stringify`使用的转义函数，在需要时可以被覆盖。

#### querystring.unescape#

`querystring.parse`使用的反转义函数，在需要时可以被覆盖。

首先它会尝试使用`decodeURIComponent`，但是如果失败了，它就转而使用一个不会在畸形URL上抛出错误的更安全的等价方法。
