# Path#


### 稳定度: 2 - 稳定

这个模块提供了处理和转换文件路径的工具。几乎所有的方法都仅提供字符串转换功能。文件系统不会去检查路径是否可用。

通过`require('path')`来使用这个模块。以下是提供的方法：

#### path.normalize(p)#

规范化字符串路径，注意`'..'`和`'.'`部分。

当有多个连续斜杠时，它们会被替换为一个斜杠；当路径的最后有一个斜杠，它会被保留。在Windows下使用反斜杠。

例子：

```js
path.normalize('/foo/bar//baz/asdf/quux/..')
// returns
'/foo/bar/baz/asdf'
```

#### path.join([path1][, path2][, ...])#

连接所有的参数，并且规范化结果路径。

参数必须是字符串。在0.8版本中，非字符串参数会被忽略，在0.10版本及之后，会抛出一个异常。

例子：

```js
path.join('/foo', 'bar', 'baz/asdf', 'quux', '..')
// returns
'/foo/bar/baz/asdf'

path.join('foo', {}, 'bar')
// throws exception
TypeError: Arguments to path.join must be strings
```

#### path.resolve([from ...], to)#

将`to`解析为绝对路径。

如果`to`不已经是相对于`from`参数的绝对路径，`to`会被添加到`from`的右边，直到找出绝对了路径。如果使用了`from`中所有的路径仍没有找出绝对路径，当前的工作路径也会被使用。结果路径会被规范化，并且结尾的斜杠会被移除，除非解析得到了一个根路径。非字符串参数会被忽略。

另一个思路是将它看做shell中一系列的`cd`命令：

```js
path.resolve('foo/bar', '/tmp/file/', '..', 'a/../subfile')
```

相似于：

```SHELL
cd foo/bar
cd /tmp/file/
cd ..
cd a/../subfile
pwd
```

区别是不同的路径不需要一定存在，并且可以是文件。

例子：

```js
path.resolve('/foo/bar', './baz')
// returns
'/foo/bar/baz'

path.resolve('/foo/bar', '/tmp/file/')
// returns
'/tmp/file'

path.resolve('wwwroot', 'static_files/png/', '../gif/image.gif')
// if currently in /home/myself/iojs, it returns
'/home/myself/iojs/wwwroot/static_files/gif/image.gif'
```

#### path.isAbsolute(path)#

判断`path`是否是一个绝对路径。一个绝对路径总是被解析为相同的路径，无论当前工作目录是哪里。

Posix例子：

```js
path.isAbsolute('/foo/bar') // true
path.isAbsolute('/baz/..')  // true
path.isAbsolute('qux/')     // false
path.isAbsolute('.')        // false
```

Windows例子：

```js
path.isAbsolute('//server')  // true
path.isAbsolute('C:/foo/..') // true
path.isAbsolute('bar\\baz')   // false
path.isAbsolute('.')         // false
```

#### path.relative(from, to)#

解析从`from`到`to`的相对路径。

当我们有两个绝对路径，并且我们要得到它们间一个对于另外一个的相对路径。这实际上是`path.resolve`的相反操作。我们可以看看这是什么意思：

```js
path.resolve(from, path.relative(from, to)) == path.resolve(to)
```

例子：

```js
path.relative('C:\\orandea\\test\\aaa', 'C:\\orandea\\impl\\bbb')
// returns
'..\\..\\impl\\bbb'

path.relative('/data/orandea/test/aaa', '/data/orandea/impl/bbb')
// returns
'../../impl/bbb'
```

#### path.dirname(p)#

返回路径的目录名。与Unix `dirname` 命令相似。

例子：

```js
path.dirname('/foo/bar/baz/asdf/quux')
// returns
'/foo/bar/baz/asdf'
```

#### path.basename(p[, ext])#

返回路径中的最后一部分。与Unix `basename` 命令相似。

例子：

```js
path.basename('/foo/bar/baz/asdf/quux.html')
// returns
'quux.html'

path.basename('/foo/bar/baz/asdf/quux.html', '.html')
// returns
'quux'
```

#### path.extname(p)#

返回路径的扩展名，即从路径的最后一部分中的最后一个`'.'`到末尾之间的字符串。如果路径的最后一部分没有`'.'`，或者第一个字符是`'.'`，那么将返回一个空字符串，例子：

```js
path.extname('index.html')
// returns
'.html'

path.extname('index.coffee.md')
// returns
'.md'

path.extname('index.')
// returns
'.'

path.extname('index')
// returns
''
```

#### path.sep#

返回特定平台的文件分隔符。`'\\'`或`'/'`。

一个*nix上的例子：

```js
'foo/bar/baz'.split(path.sep)
// returns
['foo', 'bar', 'baz']
```

一个Windows上的例子：

```js
'foo\\bar\\baz'.split(path.sep)
// returns
['foo', 'bar', 'baz']
```

#### path.delimiter#

特定平台的路径分隔符，`';'`或`':'`。

一个*nix上的例子：

```js
console.log(process.env.PATH)
// '/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin'

process.env.PATH.split(path.delimiter)
// returns
['/usr/bin', '/bin', '/usr/sbin', '/sbin', '/usr/local/bin']
```

一个Windows上的例子：

```
console.log(process.env.PATH)
// 'C:\Windows\system32;C:\Windows;C:\Program Files\iojs\'

process.env.PATH.split(path.delimiter)
// returns
['C:\\Windows\\system32', 'C:\\Windows', 'C:\\Program Files\\iojs\\']
```

#### path.parse(pathString)#

根据一个路径字符串返回一个对象。

一个*nix上的例子：

```js
path.parse('/home/user/dir/file.txt')
// returns
{
    root : "/",
    dir : "/home/user/dir",
    base : "file.txt",
    ext : ".txt",
    name : "file"
}
```

一个Windows上的例子：

```js
path.parse('C:\\path\\dir\\index.html')
// returns
{
    root : "C:\\",
    dir : "C:\\path\\dir",
    base : "index.html",
    ext : ".html",
    name : "index"
}
```

#### path.format(pathObject)#

根据一个对象，返回一个路径字符串，与`path.parse`相反。

```js
path.format({
    root : "/",
    dir : "/home/user/dir",
    base : "file.txt",
    ext : ".txt",
    name : "file"
})
// returns
'/home/user/dir/file.txt'
```

#### path.posix#

提供对上述的路径方法的访问，但是总是以兼容posix的方式交互（interact）。

#### path.win32#

提供对上述的路径方法的访问，但是总是以兼容win32的方式交互（interact）。
