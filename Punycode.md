# punycode#

#### Stability: 2 - Stable

`Punycode.js`在`io.js` v1.0.0+ 和 `Node.js` v0.6.2+ 中被内置。通过`require('punycode')`来获取它（要在其他版本的`io.js`中使用它，要先通过npm来安装`punycode`模块）。

#### punycode.decode(string)#

转换一个纯ASCII符号 `Punycode`字符串为一个`Unicode`符号的字符串。

```js
// decode domain name parts
punycode.decode('maana-pta'); // 'mañana'
punycode.decode('--dqo34k'); // '☃-⌘'
```

#### punycode.encode(string)#

转换一个`Unicode`符号的字符串为一个纯ASCII符号 `Punycode`字符串。

```js
// encode domain name parts
punycode.encode('mañana'); // 'maana-pta'
punycode.encode('☃-⌘'); // '--dqo34k'
```

#### punycode.toUnicode(domain)#

转换一个代表了一个域名的`Punycode`字符串为一个`Unicode`字符串。只有代表了域名的部分的`Punycode`字符串会被转换。也就是说，如果你调用了一个已经被转换为`Unicode`的字符串，也是没有问题的。

```js
// decode domain names
punycode.toUnicode('xn--maana-pta.com'); // 'mañana.com'
punycode.toUnicode('xn----dqo34k.com'); // '☃-⌘.com'
```

#### punycode.toASCII(domain)#

转换一个代表了一个域名的`Unicode`字符串为一个`Unicode`字符串。只有代表了域名的部分的非ASCII字符串会被转换。也就是说，如果你调用了一个已经被转换为ASCII的字符串，也是没有问题的。

```js
// encode domain names
punycode.toASCII('mañana.com'); // 'xn--maana-pta.com'
punycode.toASCII('☃-⌘.com'); // 'xn----dqo34k.com'
```

#### punycode.ucs2#
#### punycode.ucs2.decode(string)#


创建一个包含了 字符串中的每个`Unicode`符号的数字编码点 的数组。由于`JavaScript`在内部使用`UCS-2`，这个函数会将一对代理部分（surrogate halves）（UCS-2暴露的单独字符）转换为一个单独的编码点 来匹配UTF-16。

```js
punycode.ucs2.decode('abc'); // [0x61, 0x62, 0x63]
// surrogate pair for U+1D306 tetragram for centre:
punycode.ucs2.decode('\uD834\uDF06'); // [0x1D306]
```

#### punycode.ucs2.encode(codePoints)#

基于数字编码点的数组，创建一个字符串。

```js
punycode.ucs2.encode([0x61, 0x62, 0x63]); // 'abc'
punycode.ucs2.encode([0x1D306]); // '\uD834\uDF06'
```

#### punycode.version#

一个代表了当前`Punycode.js`版本号的数字。