# URL#


### 稳定度: 2 - 稳定

这个模块提供了URL解析和解释的工具。通过`require('url')`使用它。

解释URL为一个含有以下部分或全部属性的对象，依赖于它们是否在URL字符串中存在。任何不存在的部分都不会出现在解释后的对象中。一个下面URL的例子：

`'http://user:pass@host.com:8080/p/a/t/h?query=string#hash'`

 - href: 最初传递的全部URL。协议和主机都是小写的。

例子： `'http://user:pass@host.com:8080/p/a/t/h?query=string#hash'`

 - protocol: 请求的协议，小写。

例子： `'http:'`

 - slashes: 协议要求冒号后有斜杠。

例子： `true` 或 `false`

 - host: URL的所有主机部分，包括端口，小写。

例子： `'host.com:8080'`

 - auth: URL的认证信息部分。

例子： `'user:pass'`

 - hostname: 小写的主机名部分。

例子： `'host.com'`

 - port: 主机部分的端口号。

例子： `'8080'`

 - pathname: URL的路径部分，在主机之后，在查询之前，包括最前面的斜杠，如果存在的话。不提供解码。

例子： `'/p/a/t/h'`

 - search: URL的“查询字符串”部分，包括前导的问号标志。

例子： `'?query=string'`

 - path: 路径和查询的连接体。不提供解码。

例子： `'/p/a/t/h?query=string'`

 - query: 查询字符串的“参数”部分，或查询字符串被解释后的对象。

例子： `'query=string'` 或 `{'query':'string'}`

 - hash: URL的“碎片”部分，包括英镑符号。

例子： `'#hash'`

以下是URL模块提供的方法：

#### url.parse(urlStr[, parseQueryString][, slashesDenoteHost])#

接收一个URL字符串，然后返回一个对象。

对第二个参数传递`true`，将使用`querystring`模块来解释查询字符串。如果为`true`，那么最后的对象中一定存在`query`属性，并且`search`属性将总是一个字符串（可能为空）。如果为`false`，那么`query`属性将不会被解释或解码。默认为`false`。

对第三个参数传递`true`，将会把`//foo/bar`解释为`{ host: 'foo', pathname: '/bar' }`，而不是`{ pathname: '//foo/bar' }`。默认为`false`。

#### url.format(urlObj)#

接受一个解释完毕的URL对象，返回格式化URL字符串。

以下是格式化过程：

 - `href`将会被忽略。
 - `path`将会被忽略。
 - __协议无论是否有末尾的冒号，都会被同样处理。__
  - http，https，ftp，gopher，file协议的后缀是`://`。
  - 所有其他如mailto，xmpp，aim，sftp，foo等协议的后缀是`:`。
 - __如果协议要求有 `://` ，`slashes`会被设置为`true`__
  - 只有之前没有列出的要求有斜线的协议才需要被设置。如`mongodb://localhost:8000/`。
 - `auth`会被使用，如果存在的话。
 - 只有当缺少`host`时，才会使用`hostname`。
 - 只有当缺少`host`时，才会使用`port`。
 - `host`将会替代`hostname`和`port`。
 - 无论有没有前导 / （斜线），`pathname`都会被相同对待。
 - 只有在缺少`search`时，才会使用`query`（对象；参阅`querystring`）。
 - __`search`将会替代`query`__
  - 无论有没有前导 ?（问号），它都会被相同对待。
 - 无论有没有前导 #（英镑符号），`hash`都会被相同对待。

#### url.resolve(from, to)#

接受一个基础URL，和一个路径URL，并且带上锚点像浏览器一样解析他们。例子：

```js
url.resolve('/one/two/three', 'four')         // '/one/two/four'
url.resolve('http://example.com/', '/one')    // 'http://example.com/one'
url.resolve('http://example.com/one', '/two') // 'http://example.com/two'
```
