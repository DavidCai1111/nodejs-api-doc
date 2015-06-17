# Assert

### 稳定度: 2 - 稳定
这个模块被用来为你的应用编写单元测试，你可以通过
require('assert')来取得它。

#### assert.fail(actual, expected, message, operator)

抛出一个显示用例实际值和期望值的异常，使用分隔符隔开。

#### assert(value[, message]), assert.ok(value[, message])#
测试`value`是否是真值，它等同于`assert.equal(true, !!value, message);`。

#### assert.equal(actual, expected[, message])#
浅测试，等同于使用`==`进行比较。


#### assert.notEqual(actual, expected[, message])#
前测试，等同于使用`!=`进行比较。

assert.deepEqual(actual, expected[, message])#
Tests for deep equality. Primitive values are compared with the equal comparison operator ( == ). Doesn't take object prototypes into account.

assert.notDeepEqual(actual, expected[, message])#
Tests for any deep inequality. Opposite of assert.deepEqual.

assert.strictEqual(actual, expected[, message])#
Tests strict equality, as determined by the strict equality operator ( === )

assert.notStrictEqual(actual, expected[, message])#
Tests strict non-equality, as determined by the strict not equal operator ( !== )

assert.deepStrictEqual(actual, expected[, message])#
Tests for deep equality. Primitive values are compared with the strict equality operator ( === ).

assert.notDeepStrictEqual(actual, expected[, message])#
Tests for deep inequality. Opposite of assert.deepStrictEqual.

assert.throws(block[, error][, message])#
Expects block to throw an error. error can be constructor, RegExp or validation function.

Validate instanceof using constructor:

assert.throws(
  function() {
    throw new Error("Wrong value");
  },
  Error
);
Validate error message using RegExp:

assert.throws(
  function() {
    throw new Error("Wrong value");
  },
  /value/
);
Custom error validation:

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
assert.doesNotThrow(block[, message])#
Expects block not to throw an error, see assert.throws for details.

assert.ifError(value)#
Tests if value is not a false value, throws if it is a true value. Useful when testing the first argument, error in callbacks.