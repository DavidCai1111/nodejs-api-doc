# Assert

### 稳定度: 2 - 稳定
这个模块被用来为你的应用编写单元测试，你可以通过
`require('assert')`来取得它。

#### assert.fail(actual, expected, message, operator)

抛出一个显示用例实际值和期望值的异常，使用分隔符隔开。

#### assert(value[, message]), assert.ok(value[, message])#
测试`value`是否是真值，它等同于`assert.equal(true, !!value, message);`。

#### assert.equal(actual, expected[, message])
浅测试，等同于使用`==`进行比较。


#### assert.notEqual(actual, expected[, message])
浅测试，等同于使用`!=`进行比较。

#### assert.deepEqual(actual, expected[, message])
深度测试。原始类型值讲通过`==`进行比较。不要将对象原型进行比较。

#### assert.notDeepEqual(actual, expected[, message])
深度测试，与`assert.deepEqual`的结果相反。

#### assert.strictEqual(actual, expected[, message])
严格测试，使用`===`进行判断。

#### assert.notStrictEqual(actual, expected[, message])#
严格测试，使用`!==`进行判断。


#### assert.deepStrictEqual(actual, expected[, message])#
严格测试，原始类型值将使用`===`进行比较。

#### assert.notDeepStrictEqual(actual, expected[, message])#
严格测试，与`assert.deepStrictEqual`结果相反。

#### assert.throws(block[, error][, message])#
期望`block`抛出一个`error`。`error`可以是构造函数，正则表达式，或验证函数。

使用构造函数验证实例：
```js
assert.throws(
  function() {
    throw new Error("Wrong value");
  },
  Error
);
```
使用正则表达式验证错误信息:
```js
assert.throws(
  function() {
    throw new Error("Wrong value");
  },
  /value/
);
```
自定义错误验证:
```js
assert.throws(
  function() {
    throw new Error("Wrong value");
  },
  function(err) {
    if ( (err instanceof Error) && /value/.test(err) ) {
      return true;
    }
  },
  "unexpected error"
);
```
#### assert.doesNotThrow(block[, message])#
期望`block`不抛出一个错误，详情见`assert.throws`。

#### assert.ifError(value)#
测试`value`是否是一个假值，当`value`为真值时会抛出异常。在测试回调函数的一个`error`参数时非常有用。