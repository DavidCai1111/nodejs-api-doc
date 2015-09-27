# 执行`JavaScript`


### 稳定度: 2 - 稳定

要获取这个模块，你可以通过：

```js
var vm = require('vm');
```

`JavaScript`代码会被编译且立刻执行 或 编译，保存，并且稍后执行。

#### vm.runInThisContext(code[, options])#

`vm.runInThisContext()`编译代码，运行它，然后返回结果。运行中的代码不能访问本地作用域，但是可以访问当前的全局对象。

使用`vm.runInThisContext`和`eval`运行相同代码的例子：

```js
var vm = require('vm');
var localVar = 'initial value';

var vmResult = vm.runInThisContext('localVar = "vm";');
console.log('vmResult: ', vmResult);
console.log('localVar: ', localVar);

var evalResult = eval('localVar = "eval";');
console.log('evalResult: ', evalResult);
console.log('localVar: ', localVar);

// vmResult: 'vm', localVar: 'initial value'
// evalResult: 'eval', localVar: 'eval'
```

`vm.runInThisContext`不能访问本地作用域，所以`localVar`没有改变。`eval`可以访问本地作用域，所以`localVar`改变了。

这种情况下，`vm.runInThisContext`更像是一个间接的`eval`调用，像`(0,eval)('code')`。但是，它还有以下这些额外的选项：

 - filename: 允许你控制提供在堆栈追踪信息中的文件名。
 - displayErrors: 是否在抛出异常前向`stderr`打印任何的错误，并且造成错误的行会被高亮。会捕获编译代码时的语法错误和编译完的代码运行时抛出的异常。默认为`true`。
 - timeout: 在关闭之前，允许代码执行的时间（毫秒）。如果超时，一个错误被会抛出。

#### vm.createContext([sandbox])#

如果指定了一个`sandbox`对象，则将`sandbox`“上下文化”，这样它才可以被`vm.runInContext`或`script.runInContext`使用。在脚本内部，`sandbox`将会是全局对象，保留了它自己所有的属性，并且包含内建对象和标准全局对象的所有函数。在由`vm`模块运行的脚本之外的地方，`sandbox`将不会被改变。

如果没有指定`sandbox`对象，将会返回一个你可以使用的新的，无内容的`sandbox`对象。

这个函数在创建被用来运行多个脚本的沙箱时十分有用，例如，如果你正在模拟一个web浏览器，则可以创建一个代表了`window`全局对象的沙箱，然后在沙箱内运行所有的`<script>`标签。

#### vm.isContext(sandbox)#

返回一个沙箱是否已经通过调用`vm.createContext`上下文化。

#### vm.runInContext(code, contextifiedSandbox[, options])#

`vm.runInContext`编译代码，然后将其在`contextifiedSandbox`中运行，然后返回结果。运行的代码不能访问本地作用域。`contextifiedSandbox`必须通过`vm.createContext`上下文化；它被用来当做代码的全局对象。

`vm.runInContext`的选项和`vm.runInThisContext`相同。

例子：编译并执行不同的脚本，在同一个已存在的上下文中。

```js
var util = require('util');
var vm = require('vm');

var sandbox = { globalVar: 1 };
vm.createContext(sandbox);

for (var i = 0; i < 10; ++i) {
    vm.runInContext('globalVar *= 2;', sandbox);
}
console.log(util.inspect(sandbox));

// { globalVar: 1024 }
```

注意，运行不受信任的代码是一个十分棘手的工作，需要十分小心。`vm.runInContext`是十分有用的，但是为了安全的运行不受信任的代码，还是将它们放在另一个单独的进程中为好。

#### vm.runInNewContext(code[, sandbox][, options])#

`vm.runInNewContext`编译代码，接着，如果传递了`sandbox`则上下文化`sandbox`，如果没有就创建一个新的已上下文化的沙箱，然后将沙箱作为全局对象运行代码并返回结果。

`vm.runInNewContext `的选项和`vm.runInThisContext`相同。

例子：编译并执行一个 自增一个全局变量然后设置一个新的全局变量 的代码。这些全局变量包含在沙箱中。

```js
var util = require('util');
var vm = require('vm');

var sandbox = {
  animal: 'cat',
  count: 2
};

vm.runInNewContext('count += 1; name = "kitty"', sandbox);
console.log(util.inspect(sandbox));

// { animal: 'cat', count: 3, name: 'kitty' }
```

注意，运行不受信任的代码是一个十分棘手的工作，需要十分小心。`vm.runInNewContext`是十分有用的，但是为了安全的运行不受信任的代码，还是将它们放在另一个单独的进程中为好。

#### vm.runInDebugContext(code)#

`vm.runInDebugContext`编译代码，然后将它们在V8调试上下文中执行。主要的用途是访问V8调试对象：

```js
var Debug = vm.runInDebugContext('Debug');
Debug.scripts().forEach(function(script) { console.log(script.name); });
```

注意，调试上下文和对象与V8的调试实现联系紧密，它们可能在没有事先提醒的情况就发生改变（或被移除）。

调试对象也可以通过`--expose_debug_as= switch`被暴露。

#### Class: Script#

一个包含预编译代码，然后将它们运行在指定沙箱中的类。

#### new vm.Script(code, options)#

创建一个编译代码但不执行它的新`Script`类。也就是说，一个创建好的`vm.Script`对象代表了它的编译完毕的代码。这个脚本已经通过下文的方法在晚些时候被调用多次。返回的脚本没有被绑定在任何的全局对象上。它可以在每次运行前被绑定，所以只在那次运行时有效。

`options`可以有以下属性：

 - filename: 允许你控制提供在堆栈追踪信息中的文件名。
 - displayErrors: 是否在抛出异常前向`stderr`打印任何的错误，并且造成错误的行会被高亮。会捕获编译代码时的语法错误和运行时由脚本的方法的配置所控制的代码抛出的错误。

#### script.runInThisContext([options])#

与`vm.runInThisContext`相似，但是是一个预编译的`Script`对象的方法。`script.runInThisContext`运行脚本被编译完毕的代码，然后返回结果。运行中的代码不能访问本地作用域，但是可以访问当前的全局对象。

一个使用`script.runInThisContext`来编译一次代码，然后运行多次的例子：

```js
var vm = require('vm');

global.globalVar = 0;

var script = new vm.Script('globalVar += 1', { filename: 'myfile.vm' });

for (var i = 0; i < 1000; ++i) {
  script.runInThisContext();
}

console.log(globalVar);

// 1000
```

`options`可以有以下属性：

 - displayErrors: 是否在抛出异常前向`stderr`打印任何的错误，并且造成错误的行会被高亮。只会应用于运行中代码的执行错误；创建一个有语法错误的`Script`实例是不可能的，因为构造函数会抛出异常。

 - timeout: 在关闭之前，允许代码执行的时间（毫秒）。如果超时，一个错误被会抛出。

#### script.runInContext(contextifiedSandbox[, options])#

与`vm.runInContext `相似，但是是一个预编译的`Script`对象的方法。`script.runInContext `运行脚本被编译完毕的代码，然后返回结果。运行中的代码不能访问本地作用域。

`script.runInContext`的选项和`script.runInThisContext`相同。

例子：编译一段 自增一个全局对象并且创建一个全局对象 的代码，然后执行多次，这些全局对象包含在沙箱中。

```js
var util = require('util');
var vm = require('vm');

var sandbox = {
  animal: 'cat',
  count: 2
};

var context = new vm.createContext(sandbox);
var script = new vm.Script('count += 1; name = "kitty"');

for (var i = 0; i < 10; ++i) {
  script.runInContext(context);
}

console.log(util.inspect(sandbox));

// { animal: 'cat', count: 12, name: 'kitty' }
```

注意，运行不受信任的代码是一个十分棘手的工作，需要十分小心。`script.runInContext`是十分有用的，但是为了安全的运行不受信任的代码，还是将它们放在另一个单独的进程中为好。

#### script.runInNewContext([sandbox][, options])#

与`vm.runInNewContext`相似，但是是一个预编译的`Script`对象的方法。如果传递了`sandbox`,`script.runInNewContext`将上下文化`sandbox`，如果没有就创建一个新的已上下文化的沙箱，然后将沙箱作为全局对象运行代码并返回结果。运行中的代码不能访问本地作用域。

`script.runInNewContext `的选项和`script.runInThisContext `相同。

例子：编译一段 设置一个全局对象 的代码，然后在不同的上下文中多次执行它。这些全局对象包含在沙箱中。

```js
var util = require('util');
var vm = require('vm');

var sandboxes = [{}, {}, {}];

var script = new vm.Script('globalVar = "set"');

sandboxes.forEach(function (sandbox) {
  script.runInNewContext(sandbox);
});

console.log(util.inspect(sandboxes));

// [{ globalVar: 'set' }, { globalVar: 'set' }, { globalVar: 'set' }]
```

注意，运行不受信任的代码是一个十分棘手的工作，需要十分小心。`script.runInNewContext`是十分有用的，但是为了安全的运行不受信任的代码，还是将它们放在另一个单独的进程中为好。
