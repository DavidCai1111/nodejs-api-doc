# Modules#

### 稳定度: 3 - 锁定

`node.js`又一个简单的模块加载系统。在`node.js`中，文件和模块是一一对应的。以下例子中，`foo.js`加载的同目录下的`circle.js`。

`foo.js`的内容：

```js
var circle = require('./circle.js');
console.log( 'The area of a circle of radius 4 is '
           + circle.area(4));
```

`circle.js`的内容：

```js
var PI = Math.PI;

exports.area = function (r) {
  return PI * r * r;
};

exports.circumference = function (r) {
  return 2 * PI * r;
};
```

`circle.js`模块暴露了`area()`函数和`circumference()`函数。想要为你的模块添加函数或对象，你可以将它们添加至特殊的`exports`对象的属性上。

模块的本地变量是私有的，好似模块被包裹在一个函数中。在这个例子中变量`PI`是`circle.js`私有的。

如果想要你的模块暴露一个函数（例如一个构造函数），或者想要一次赋值就暴露一个完整的对象，而不是一次绑定一个属性，那就将之赋值给`module.exports`而不是`exports`。

以下，`bar.js`使用了暴露了一个构造函数的`square`模块：

```js
var square = require('./square.js');
var mySquare = square(2);
console.log('The area of my square is ' + mySquare.area());
```

`square`模块内部：

```js
// assigning to exports will not modify module, must use module.exports
module.exports = function(width) {
  return {
    area: function() {
      return width * width;
    }
  };
}
```

模块系统在`require("module")`中被实现。

### 循环依赖

当存在循环的`require()`调用。一个模块可能在返回时，被没有被执行完毕。

考虑一下情况：

`a.js`：

```js
console.log('a starting');
exports.done = false;
var b = require('./b.js');
console.log('in a, b.done = %j', b.done);
exports.done = true;
console.log('a done');
```

`b.js`：

```js
console.log('b starting');
exports.done = false;
var a = require('./a.js');
console.log('in b, a.done = %j', a.done);
exports.done = true;
console.log('b done');
```

`main.js`：

```js
console.log('main starting');
var a = require('./a.js');
var b = require('./b.js');
console.log('in main, a.done=%j, b.done=%j', a.done, b.done);
```

当`main.js`加载`a.js`，而后`a.js`会去加载`b.js`。与此同时，`b.js`尝试去加载`a.js`。为了避免一个无限循环，`a.js`会返回一个未完成的副本给`b.js`模块。`b.js`会接着完成加载，然后它所暴露的值再被提供给`a.js`模块。

这样`main.js`就完成了它们的加载。因此程序的输出是：

```SHELL
$ iojs main.js
main starting
a starting
b starting
in b, a.done = false
b done
in a, b.done = true
a done
in main, a.done=true, b.done=true
```

如果在你的程序里有循环依赖，请确保它们按你的计划工作。

#### 核心模块

`node.js`中有一些模块是被编译成二进制的。这些模块会在本文档的其他地方详细讨论。

核心模块被定义在`node.js`源码的`lib/`目录下。

当被`require()`时，核心模块总是被优先加载的。例如`require('http')`总是会返回内建的HTTP模块，甚至是有一个同名文件时。

#### 文件模块

如果准确的文件名没有被发现，那么`node.js`将会依次添加`.js`，`.json`或`.node`后缀名，然后试图去加载。

`.js`文件被解释为`JavaScript`文本文件，`.json`被解释为`JSON`文本文件，`.node`文件被解释为编译好的插件模块，然后被`dlopen`加载。

前缀是`'/'`则是文件的绝对路径。例如`require('/home/marco/foo.js')`将会加载`/home/marco/foo.js`。

前缀是`'./'`则是调用`require()`的文件的相对路径。也就是说，`circle.js`必须与`foo.js`在同一目录下，这样`require('./circle')`才能找到它。

如果没有`'/'`，`'./'`或`'../'`前缀，模块要么是一个核心模块，或是需要从`node_modules`目录中被加载。

如果指定的路径不存在，`require()`将会抛出一个`code`属性是`'MODULE_NOT_FOUND'`的错误。

#### 从node_modules目录中加载

如果传递给`require()`的模块标识符不是一个本地模块，也没有以`'/'`，`'../'`或`'./'`开始。那么`node.js`将会从当前目录的父目录开始，添加`/node_modules`，试图从这个路径来加载模块。

如果还是没有找到模块，那么它会再移至此目录的父目录，如此往复，直至到达文件系统的根目录。

例如，如果一个位于`'/home/ry/projects/foo.js'`的文件调用了`require('bar.js')`，那么`node.js`将会按照以下的路径顺序来查找：

```SHELL
/home/ry/projects/node_modules/bar.js
/home/ry/node_modules/bar.js
/home/node_modules/bar.js
/node_modules/bar.js
```

这要求程序本地化（localize）自己的依赖，防止它们崩溃。

你也可以在模块名中加入一个路径后缀，来引用这个模块中特定的一个文件或子模块。例如，`require('example-module/path/to/file')`将会从`example-module`的位置解析相对路径`path/to/file`。路径后缀遵循相同的模块解析语义。

#### 作为模块的目录

在一个单独目录下组织程序和库，然后提供一个单独的入口，是非常便捷的。有三种方法，可以将目录作为`require()`的参数，来加载模块。

第一种方法是，在模块的根目录下创建一个`package.json`文件，其中指定了`main`模块。一个示例`package.json`文件：

```json
{ "name" : "some-library",
  "main" : "./lib/some-library.js" }
```

如果这个文件位于`./some-library`，那么`require('./some-library')`将会试图去加载`./some-library/lib/some-library.js`。

这就是`node.js`所能够了解`package.json`文件的程度。

如果目录中没有`package.json`文件，那么`node.js`将会视图去加载当前目录中的`index.js`或`index.node`。例如，如果在上面的例子中没有`package.json`，那么`require('./some-library')`将会试图加载：

```SHELL
./some-library/index.js
./some-library/index.node
```

#### 缓存

模块在第一次被加载后，会被缓存。这意味着，如果都解析到了相同的文件，每一次调用`require('foo')`都将会返回同一个对象。

多次调用`require('foo')`可能不会造成模块代码被执行多次。这是一个重要的特性。有了它，“部分完成”的对象也可以被返回，这样，传递依赖也能被加载，即使它们可能会造成循环依赖。

如果你想要一个模块被多次执行，那么就暴露一个函数，然后执行这个函数。

##### 模块缓存警告

模块的缓存依赖于它们被解析后的文件名。所以调用模块的位置不同，可以会解析出不同的文件名（比如需要从node_modules目录中加载）。所以不能保证`require('foo')`总是会返回相同的对象，因为它们可能被解析为了不同的文件。

#### module对象
 - {Object}

每一个模块中，变量`module`是一个代表了当前模块的引用。为了方便，`module.exports`也可以通过模块作用域中的`exports`取得。`module`对象实际上不是全局的，而是每个模块本地的。

#### module.exports#

 - Object

`module.exports`对象是由模块系统创建的。有时这是难以接受的；许多人希望它们的模块是一些类的实例。如果需要这样，那么就将想要暴露的对象赋值给`module.exports`。注意，将想要暴露的对象传递给`exports`，将仅仅只会重新绑定（rebind）本地变量`exports`，所以不要这么做。

例如假设我们正在写一个叫做`a.js`的模块：

```js
var EventEmitter = require('events').EventEmitter;

module.exports = new EventEmitter();

// Do some work, and after some time emit
// the 'ready' event from the module itself.
setTimeout(function() {
  module.exports.emit('ready');
}, 1000);
```

那么在另一个文件中我们可以：

```js
var a = require('./a');
a.on('ready', function() {
  console.log('module a is ready');
});
```

主要，对`module.exports`的赋值必须立刻完成。它不能在任何的回调函数中完成。以下例子将不能正常工作：

`x.js`：

```js
setTimeout(function() {
  module.exports = { a: "hello" };
}, 0);
```

`y.js`：

```js
var x = require('./x');
console.log(x.a);
```

#### exports快捷方式

`exports`变量是一个`module.exports`的引用。如果你将一个新的值赋予它，那么它将不再指向先前的那个值。

为了说明这个行为，将`require()`的实现假设为这样：

```js
function require(...) {
  // ...
  function (module, exports) {
    // Your module code here
    exports = some_func;        // re-assigns exports, exports is no longer
                                // a shortcut, and nothing is exported.
    module.exports = some_func; // makes your module export 0
  } (module, module.exports);
  return module;
}
```

一个指导方针是，如果你弄不清楚`exports`和`module.exports`之间的关系，请只使用`module.exports`。

#### module.require(id)#

 - id String
 - Return: 被解析的模块的`module.exports`

`module.require`方法提供了一种像`require()`一样，从源模块中加载模块的方法。

注意，为了这么做，你必须取得`module`对象的引用。因为`require()`返回`module.exports`，并且`module`对象是一个典型的只在特定的模块作用域中有效的变量，如果要使用它，必须被明确地导出。

#### module.id#

 - String

模块的识别符。通常是被完全解析的文件名。

#### module.filename#

 - String

模块完全解析后的文件名。

#### module.loaded#

 - Boolean

模块是否加载完成，或者是正在加载的过程中。

#### module.parent#

 - Module Object
 
引用这个模块的模块。

#### module.children#

 - Array

这个模块所引入的模块。

#### 总体来说

为了获得`require()`被调用时将要被加载的准确文件名，使用`require.resolve()`函数。

综上所述，以下是一个`require.resolve`所做的事的高级算法伪代码：

```SHELL
require(X) from module at path Y
1. If X is a core module,
   a. return the core module
   b. STOP
2. If X begins with './' or '/' or '../'
   a. LOAD_AS_FILE(Y + X)
   b. LOAD_AS_DIRECTORY(Y + X)
3. LOAD_NODE_MODULES(X, dirname(Y))
4. THROW "not found"

LOAD_AS_FILE(X)
1. If X is a file, load X as JavaScript text.  STOP
2. If X.js is a file, load X.js as JavaScript text.  STOP
3. If X.json is a file, parse X.json to a JavaScript Object.  STOP
4. If X.node is a file, load X.node as binary addon.  STOP

LOAD_AS_DIRECTORY(X)
1. If X/package.json is a file,
   a. Parse X/package.json, and look for "main" field.
   b. let M = X + (json main field)
   c. LOAD_AS_FILE(M)
2. If X/index.js is a file, load X/index.js as JavaScript text.  STOP
3. If X/index.json is a file, parse X/index.json to a JavaScript object. STOP
4. If X/index.node is a file, load X/index.node as binary addon.  STOP

LOAD_NODE_MODULES(X, START)
1. let DIRS=NODE_MODULES_PATHS(START)
2. for each DIR in DIRS:
   a. LOAD_AS_FILE(DIR/X)
   b. LOAD_AS_DIRECTORY(DIR/X)

NODE_MODULES_PATHS(START)
1. let PARTS = path split(START)
2. let I = count of PARTS - 1
3. let DIRS = []
4. while I >= 0,
   a. if PARTS[I] = "node_modules" CONTINUE
   c. DIR = path join(PARTS[0 .. I] + "node_modules")
   b. DIRS = DIRS + DIR
   c. let I = I - 1
5. return DIRS
```

#### 从全局文件夹加载

如果`NODE_PATH`环境变量被设置为了一个以冒号分割的绝对路径列表，那么在找不到模块时，`node.js`将会从这些路径中寻找模块（注意：在Windows中，`NODE_PATH `是以分号间隔的）。

`NODE_PATH`最初被创建，是用来支持在当前的模块解析算法被冻结（frozen）前，从不同的路径加载模块的。

`NODE_PATH`仍然被支持，但是，如今`node.js`生态圈已经有了放置依赖模块的公约，它已经不那么必要的。有时，当人们没有意识到`NODE_PATH`有被设置时，依赖于`NODE_PATH`的部署可能会产生出人意料的表现。有时，一个模块的依赖改变了，造成了通过`NODE_PATH`，加载了不同版本的模块。

另外，`node.js`将会查找以下路径：

 - 1: $HOME/.node_modules
 - 2: $HOME/.node_libraries
 - 3: $PREFIX/lib/node

`$HOME`是用户的家目录，`$PREFIX`是`node.js`中配置的`node_prefix `。

由于一些历史原因，高度推荐你将依赖放入`node_modules`目录。它会被加载的更快，且可靠性更好。

#### 访问主模块

当一个文件直接由`node.js`执行，`require.main`将被设置为这个模块。这意味着你可以判断一个文件是否是直接被运行的。

```js
require.main === module
```

对于一个文件`foo.js`，如果通过`iojs foo.js`运行，以上将会返回`true`。如果通过`require('./foo')`，将会返回`false`。

因为`module`提供了一个`filename`属性（通常等于`__filename`），所以当前应用的入口点可以通过检查`require.main.filename`来获取。

#### 附录：包管理小贴士

`node.js`的`require()`函数的语义被设计得足够通用，来支持各种目录结构。包管理程序诸如`dpkg`，`rpm`和`npm`将可以通过不修改`node.js`模块，来构建本地包。

以下我们给出一个建议的可行的目录结构：

假设`/usr/lib/node/<some-package>/<some-version>`中有指定版本包的内容。

包可以依赖于其他包。为了安装`foo`包，你可能需要安装特定版本的`bar`包。`bar`包可能有它自己的依赖，在一些情况下，它们的依赖可以会冲突或者产生循环。

由于`node.js`会查找任何它加载的包得真实路径（也就是说，解析`symlinks`），解析以下结构的方案非常简单：

 - /usr/lib/node/foo/1.2.3/ - `foo`包的内容，`1.2.3`版本。
 - /usr/lib/node/bar/4.3.2/ - `foo`包所依赖的`bar`包的内容。
 - /usr/lib/node/foo/1.2.3/node_modules/bar - 指向`/usr/lib/node/bar/4.3.2/`的符号链接。
 - /usr/lib/node/bar/4.3.2/node_modules/* - 指向`bar`包所依赖的包的符号链接。

因此，即使有循环依赖，或者依赖冲突，每个模块都能够获取它们使用的特定版本的依赖。

当`foo`包中的代码执行`require('bar')`，将会获得符号链接`/usr/lib/node/foo/1.2.3/node_modules/bar`指向的版本。接着，`bar`包种的代码执行`require('quux')`，它将会获得符号链接`/usr/lib/node/bar/4.3.2/node_modules/quux`指向的版本。

此外，为了优化模块查找的过程，我们将模块放在`/usr/lib/node_modules/<name>/<version>`而不是直接放在`/usr/lib/node`中。然后在找不到依赖时，`node.js`就不会一直去查找`/usr/node_modules`或`/node_modules`目录了。

为了让模块在`node.js`的REPL中可用，可能需要将`/usr/lib/node_modules`目录加入到`$NODE_PATH`环境变量。因为使用`node_modules`目录的模块查找都是使用相对路径，且基于调用`require()`的文件的真实路径，因此包本身可以在任何位置。