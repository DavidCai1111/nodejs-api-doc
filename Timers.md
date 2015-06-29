# Timers#

### 稳定度: 3 - 锁定

所有的定时器函数都是全局的。当需要使用它们时，不必通过`require()`。

#### setTimeout(callback, delay[, arg][, ...])#

在指定的延时（毫秒）后执行一次回调函数。返回一个可以被调用`clearTimeout()`的`timeoutObject`。可选的，你可以传递回调函数的参数。

需要注意的是，你的回调函数可以不会在精确的在指定的毫秒延时后执行 - `io.js`对回调函数执行的精确时间以及顺序都不作保证。回调函数的执行点会尽量接近指定的延时。

#### clearTimeout(timeoutObject)#

阻止一个`timeout`的触发。

#### setInterval(callback, delay[, arg][, ...])#

在每次到达了指定的延时后，都重复执行回调函数。返回一个可以被调用`clearInterval()`的`intervalObject`。可选的，你可以传递回调函数的参数。

#### clearInterval(intervalObject)#

阻止一个`interval`的触发。

#### unref()#

`setTimeout`和`setInterval`的返回值也有一个`timer.unref()`方法，这个方法允许你创建一个 当它是事件循环中的仅剩项时，它不会保持程序继续运行 的定时器。如果一个定时器已经被`unref`，再次调用`unref`不会有任何效果。

在`setTimeout`的情况下，当你调用`unref`时，你创建了一个将会唤醒事件循环的另一个定时器。创建太多这样的定时器会影响时间循环的性能 -- 请明智地使用。

#### ref()#

如果你先前对一个定时器调用了`unref()`，你可以调用`ref()`来明确要求定时器要保持程序运行。如果一个定时器已经被`ref`，再次调用`ref`不会有任何效果。

#### setImmediate(callback[, arg][, ...])#

在下一次I/O事件循环后，在`setTimeout`和`setInterval`前，“立刻”执行回调函数。返回一个可以被`clearImmediate()`的`immediateObject`。可选的，你可以传递回调函数的参数。

由`setImmediate`创建的回调函数会被有序地排队。每一次事件循环迭代时，整个回调函数队列都会被处理。如果你在一个执行中的回调函数里调用了`setImmediate`，那么这个`setImmediate`中的回调函数会在下一次事件循环迭代时被调用。

#### clearImmediate(immediateObject)#

阻止一个`immediate`的触发。