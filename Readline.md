# Readline#

### 稳定度: 2 - 稳定

通过`require('readline')`来使用这个模块。`Readline`允许逐行读取一个流（如`process.stdin`）。

注意，一旦你执行了这个模块，你的`node.js`程序在你关闭此接口之前，将不会退出。以下是如何让你的程序优雅的退出的例子：

```js
var readline = require('readline');

var rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout
});

rl.question("What do you think of node.js? ", function(answer) {
  // TODO: Log the answer in a database
  console.log("Thank you for your valuable feedback:", answer);

  rl.close();
});
```

#### readline.createInterface(options)#

创建一个`readline`接口实例。接受一个`options`对象，接受以下值：

 - input - 需要监听的可读流（必选）。

 - output - 将逐行读取的数据写入的流（可选）。

 - completer - 用于Tab自动补全的可选函数。参阅下文的使用例子。

 - terminal - 如果`input`和`output`流需要被像一个TTY一样对待，并且被经由ANSI/VT100转义代码写入，就传递`true`。默认为在实例化时，检查出的`ouput`流的`isTTY`值。

 - historySize - 保留的历史记录行的最大数量。默认为`30`。

`completer`函数被给予了一个用户输入的当前行，并且支持返回一个含有两个元素的数组：

 1. 一个匹配当前输入补全的数组。

 2. 一个被用于匹配的子字符串。

最终形式如：`[[substr1, substr2, ...], originalsubstring]`。

例子：

```js
function completer(line) {
  var completions = '.help .error .exit .quit .q'.split(' ')
  var hits = completions.filter(function(c) { return c.indexOf(line) == 0 })
  // show all completions if none found
  return [hits.length ? hits : completions, line]
}
```

`completer`同样也可以以同步的方式运行，如果它接受两个参数：

```js
function completer(linePartial, callback) {
  callback(null, [['123'], linePartial]);
}
```

`createInterface`通常与`process.stdin`和`process.stdout`搭配，用来接受用户输入：

```js
var readline = require('readline');
var rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout
});
```

一旦你有了一个`readline`接口，你通常要监听一个`line`事件。

如果这个实例中，`terminal`为`true`。那么`output`流将会得到最好的兼容性，如果它定义了`output.columns`属性，并且在`output`的`columns`变化时（当它是一个TTY时，`process.stdout`会自动这么做），触发了一个`resize`事件。

#### Class: Interface#

一个代表了有`input`和`output`流的`readline`接口。

#### rl.setPrompt(prompt)#

设置提示符，例如当你在命令行运行`iojs`命令时，你看到了`>`，这就是`node.js`的提示符。

#### rl.prompt([preserveCursor])#

为用户的输入准备好`readline`，在新的一行放置当前的`setPrompt`选项，给予用户一个新的用于输入的地方。设置`preserveCursor`为`true`，来防止光标位置被重置为`0`。

仍会重置被`createInterface`使用的`input`流，如果它被暂停。

如果当调用`createInterface`时`output`被设置为`null`或`undefined`，提示符将不会被写入。

#### rl.question(query, callback)#

带着`query`来预先放置提示符，并且在用户应答时执行回调函数。给用户展示`query`，然后在用户输入了应答后调用`callback`。

仍会重置被`createInterface`使用的`input`流，如果它被暂停。

如果当调用`createInterface`时`output`被设置为`null`或`undefined`，什么都不会被展示。

例子：

```js
interface.question('What is your favorite food?', function(answer) {
  console.log('Oh, so your favorite food is ' + answer);
});
```

#### rl.pause()#

暂停`readline`的`input`流，允许它在晚些需要时恢复。

注意，带着事件的流不会立刻被暂停。在调用了`pause`后，许多事件可能被触发，包括`line`事件。

#### rl.resume()#

恢复`readline`的`input`流。

#### rl.close()#

关闭实例接口，放弃对`input`和`output`流的控制。`close`事件也会被触发。

#### rl.write(data[, key])#

向`output`流写入数据，除非当调用`createInterface`时`output`被设置为`null`或`undefined`。`key`是一个代表了键序列的对象；在当终端为TTY时可用。

如果`input`流被暂停，它也会被恢复。

例子：

```js
rl.write('Delete me!');
// Simulate ctrl+u to delete the line written previously
rl.write(null, {ctrl: true, name: 'u'});
```

#### Events#
#### Event: 'line'#

 - function (line) {}

当`input`流收到一个`\n`时触发，通常在用户敲下回车时触发。这是一个监听用户输入的好钩子。

例子：

```js
rl.on('line', function (cmd) {
  console.log('You just typed: '+cmd);
});
```

#### Event: 'pause'#

 - function () {}

当`input`流被暂停时触发。

也会在`input`没有被暂停并且收到一个`SIGCONT`事件时触发（参阅`SIGTSTP`事件和`SIGCONT`事件）。

例子：

```js
rl.on('pause', function() {
  console.log('Readline paused.');
});
```

#### Event: 'resume'#

 - function () {}

当`input`流被恢复时触发。

例子：

```js
rl.on('resume', function() {
  console.log('Readline resumed.');
});
```

#### Event: 'close'#

 - function () {}

当`close()`被调用时触发。

也会在`input`流收到它的`end`事件时触发。当这个事件触发时，接口实例需要考虑“被结束”。例如，当`input`流接收到`^D`（也被认作`EOT`）。

这个事件也会在如果当前没有`SIGINT`事件监听器，且`input`流接收到`^C`（也被认作`SIGINT`）时触发。

#### Event: 'SIGINT'#

 - function () {}

当`input`流接收到`^C`（也被认作`SIGINT`）时触发。如果当前没有`SIGINT`事件的监听器，`pause`事件将会被触发。

例子：

```js
rl.on('SIGINT', function() {
  rl.question('Are you sure you want to exit?', function(answer) {
    if (answer.match(/^y(es)?$/i)) rl.pause();
  });
});
```

#### Event: 'SIGTSTP'#

 - function () {}

在Windows平台下不能使用。

当`input`流接收到一个`^Z`（也被认作`SIGTSTP`）时触发。如果当前没有`SIGTSTP`事件的监听器，这个程序将会被送至后台运行。

当程序使用`fg`恢复，`pause`和`SIGCONT`事件都会被触发。你可以选择其中的一个来恢复流。

如果流在程序被送至后台前就被暂停，`pause`和`SIGCONT`事件将不会触发。

例子：

```js
rl.on('SIGTSTP', function() {
  // This will override SIGTSTP and prevent the program from going to the
  // background.
  console.log('Caught SIGTSTP.');
});
```

#### Event: 'SIGCONT'#

 - function () {}

在Windows平台下不能使用。

当`input`流被`^Z`（也被认作`SIGTSTP`）送至后台时触发，然后使用`fg(1)`继续执行。这个事件仅在程序被送至后台前流没有被暂停时触发。

例子：

```js
rl.on('SIGCONT', function() {
  // `prompt` will automatically resume the stream
  rl.prompt();
});
```
#### Example: Tiny CLI#

下面是一个使用以上方法来创建一个迷你的控制台接口的例子：

```js
var readline = require('readline'),
    rl = readline.createInterface(process.stdin, process.stdout);

rl.setPrompt('OHAI> ');
rl.prompt();

rl.on('line', function(line) {
  switch(line.trim()) {
    case 'hello':
      console.log('world!');
      break;
    default:
      console.log('Say what? I might have heard `' + line.trim() + '`');
      break;
  }
  rl.prompt();
}).on('close', function() {
  console.log('Have a great day!');
  process.exit(0);
});
```

#### readline.cursorTo(stream, x, y)#

在给定的TTY流中，将光标移动到指定位置。

#### readline.moveCursor(stream, dx, dy)#

在给定的TTY流中，相对于当前位置，将光标移动到指定位置。

#### readline.clearLine(stream, dir)#

用指定的方式，在给定的TTY流中，清除当前的行。`dir`可以是以下值之一：

 - -1 - 从光标的左边
 - 1 - 从光标的右边
 - 0 - 整行
#### readline.clearScreenDown(stream)#

从当前的光标位置，清除屏幕。