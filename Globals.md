# Global Objects#

这些对象是所有模块都可用的。其中的一些对象不是真正的在全局作用域内，而是在模块作用域内 - 它将会在文档中被指出。

#### global#
 - {Object} 全局命名空间对象。
 
在浏览器，顶级作用域是全局作用域。这意味着在浏览器的全局作用域中，你创建了一个对象那么就是定义了一个全局对象。在`node.js`中是不同的，顶级作用域不是全局作用域，在`node.js`的模块中创建的对象只属于那个模块。

#### process#
 - {Object}
 
进程对象。参阅`process`章节。

#### console#
 - {Object}

被用来向`stdout`和`stderr`打印信息。参阅`console`章节。

#### Class: Buffer#
 - {Function}

被用来处理二进制数据。参阅`buffer`章节。

#### require()#
 - {Function}

用来引入模块。参阅`Modules`章节。`require`实际上不是全局的，而是每个模块本地的。

#### require.resolve()#

使用内部`require()`机制来查找模块位置，但是只返回被解析的模块路径，而不是加载模块。

#### require.cache#
 - Object

当模块被引入时，模块在这个对象中被缓存。通过删除这个对象的键值，下一次引入会重新加载模块。

#### require.extensions#

稳定度: 0 - 弃用
 - Object
  
指示`require`方法如何处理特定的文件扩展名。

将扩展名为`.sjs`的文件当做`.js`文件处理：

```js
require.extensions['.sjs'] = require.extensions['.js'];
```

在被弃用之前，这个列表被用于按需编译非`JavaScript`模块并加载入`node.js`。但是，在实践中，有更好地方法来实现这个功能，如使用其他的`node.js`程序来加载模块，或在预编译为`JavaScript`。

由于模块系统的API已被锁定，这个特性可能永远不会被去处。但是它可能有细微的bug和额外的复杂性，所以最好不要再使用它。

#### __filename#
 - {String}

当前被指定的代码的文件名。它被解析为绝对路径。对于主程序，它可能与命令行中使用的文件路径是不同的。在模块内这个值是该模块文件的路径。

例子：在`/Users/mjr`目录中执行`iojs example.js`

```js
console.log(__filename);
// /Users/mjr/example.js
```

`__filename`实际上不是全局的，而是每个模块本地的。

#### __dirname#
 - {String}
 
当前执行脚本所在的目录名。

例子：在`/Users/mjr`目录中执行`iojs example.js`

```js
console.log(__dirname);
// /Users/mjr
```

`__dirname`实际上不是全局的，而是每个模块本地的。

#### module#
 - {Object}
 
当前模块的一个引用。特别的，`module.exports`被用来指定模块需要对外暴露的东西，这些东西可以通过`require()`取得。

`module`实际上不是全局的，而是每个模块本地的。

更多信息请参阅模块系统文档。

#### exports#

`module.exports`的一个快捷引用。对于何时使用`exports`，何时使用`module.exports`，请参阅模块系统文档。

`exports`实际上不是全局的，而是每个模块本地的。

更多信息请参阅模块系统文档。

更多信息请参阅`module`章节。

#### setTimeout(cb, ms)#

在至少`ms`毫秒后，执行回调函数`cb`。实际的延时依赖于外部因素，如操作系统的定时器粒度和系统负载。

超时时间必须在1到2,147,483,647之间。如果超过了这个范围，它会被重置为1毫秒。换句话说，定时器的跨度不可以超过24.8天。

返回一个代表此定时器的句柄值。

#### clearTimeout(t)#

停止一个之前通过`setTimeout()`创建的定时器。它的回调函数将不会执行。

#### setInterval(cb, ms)#

以`ms`毫秒的间隔，重复地执行回调函数`cb`。实际的间隔可能会有浮动，这取决于外部因素，如操作系统的定时器粒度和系统负载。它永远不会比`ms`短只会比它长。

间隔值必须在1到2,147,483,647之间。如果超过了这个范围，它会被重置为1毫秒。换句话说，定时器的跨度不可以超过24.8天。

返回一个代表此定时器的句柄值。

#### clearInterval(t)#

停止一个之前通过`setInterval()`创建的定时器。它的回调函数将不会执行。

定时器函数都是全局变量。参阅定时器章节。