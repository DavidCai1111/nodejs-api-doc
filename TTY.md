# TTY#

### Stability: 2 - Stable

`tty`模块主要提供了`tty.ReadStream`和`tty.WriteStream`这两个类。大多数情况下，你都不需要直接使用这个模块。

当`node.js`检测到它运行于TTY上下文中，那么`process.stdin`将会是一个`tty.ReadStream`实例，`process.stdout`将会是一个`tty.WriteStream`实例。测试`node.js`是否运行在TTY上下文中的一个比较好的办法是检查`process.stdout.isTTY`：

```SHELL
$ iojs -p -e "Boolean(process.stdout.isTTY)"
true
$ iojs -p -e "Boolean(process.stdout.isTTY)" | cat
false
```

#### tty.isatty(fd)#

如果`fd`关联了终端，就返回`true`，反之返回`false`。

#### tty.setRawMode(mode)#

已弃用。使用`tty.ReadStream#setRawMode()`（如`process.stdin.setRawMode()`）代替。

#### Class: ReadStream#

一个`net.Socket`子类，代表了一个TTY中的可读部分。一般情况下，在任何`node.js`程序（仅当`isatty(0)`为`true`时）中，`process.stdin`将是仅有的`tty.ReadStream`实例。

#### rs.isRaw#

一个被初始化为`false`的布尔值。它代表了`tty.ReadStream`实例的“原始”状态。

#### rs.setRawMode(mode)#

`mode`必须为`true`或`false`。它设定`tty.ReadStream`的属性表现得像原始设备或默认值。`isRaw`将会被设置为结果模式（resulting mode）。

#### Class: WriteStream#

一个`net.Socket`子类，代表了一个TTY中的可写部分。一般情况下，在任何`node.js`程序（仅当`isatty(1)`为`true`时）中，`process.stdout`将是仅有的`tty.WriteStream`实例。

#### ws.columns#

一个表示了TTY当前拥有列数的数字。这个属性会通过`resize`事件被更新。

#### ws.rows#

一个表示了TTY当前拥有行数的数字。这个属性会通过`resize`事件被更新。

#### Event: 'resize'#

 - function () {}

当列属性或行属性被改变时，通过`refreshSize()`被触发。

```js
process.stdout.on('resize', function() {
  console.log('screen size has changed!');
  console.log(process.stdout.columns + 'x' + process.stdout.rows);
});
```