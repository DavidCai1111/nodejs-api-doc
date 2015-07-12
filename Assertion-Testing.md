# Assert

### 稳定度: 2 - 稳定
本模块被用来为你的应用编写单元测试，你可以通过
`require('assert')`来使用它。

#### assert.fail(actual, expected, message, operator)

抛出一个打印实际值`actual`和期望值`expected`的异常，使用分隔符`operator`隔开。

#### assert(value[, message]), assert.ok(value[, message])#
测试`value`是否为真，它等同于`assert.equal(true, !!value, message);`。

#### assert.equal(actual, expected[, message])
判等`actual`与`expected`是否相等，等同于使用`==`进行比较。


#### assert.notEqual(actual, expected[, message])
判断`actual`与`expected`是否相等，等同于使用`!=`进行比较。

#### assert.deepEqual(actual, expected[, message])
深度判断相等，通过比较`actual`与`expected`所有原型（prototype）之外的属性是否相等（`==`)来判断二者是否相等。

#### assert.notDeepEqual(actual, expected[, message])
深度判断不相等，与`assert.deepEqual`的结果相反。

#### assert.strictEqual(actual, expected[, message])
判断`actual`与`expected`是否“全等（`===`）”。

#### assert.notStrictEqual(actual, expected[, message])#
判断`actual`与`expected`是否“不全等（`!==`）”。


#### assert.deepStrictEqual(actual, expected[, message])#
深度判断全等，通过比较`actual`与`expected`所有原型（prototype）之外的属性是否全等（`===`）来判断二者是否相等。

#### assert.notDeepStrictEqual(actual, expected[, message])#
深度判断不全等，与`assert.deepStrictEqual`结果相反。

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
期望`block`不抛出错误，详情见`assert.throws`。

#### assert.ifError(value)#
测试`value`是否为假，当`value`为真时会抛出异常。通常用来判断回调函数中第一个`error`参数。
