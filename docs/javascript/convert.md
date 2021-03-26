## 数据类型的转换

### 强制转换

#### Number()

##### 原始类型值
```js
/ 数值：转换后还是原来的值
Number(324) // 324

// 字符串：如果可以被解析为数值，则转换为相应的数值
Number('324') // 324

// 字符串：如果不可以被解析为数值，返回 NaN
Number('324abc') // NaN

// 空字符串转为0
Number('') // 0

// 布尔值：true 转成 1，false 转成 0
Number(true) // 1
Number(false) // 0

// undefined：转成 NaN
Number(undefined) // NaN

// null：转成0
Number(null) // 0
```

##### 对象

> 简单的规则是，Number方法的参数是对象时，将返回NaN，除非是包含单个数值的数组。

1. 第一步，调用对象自身的valueOf方法。如果返回原始类型的值，则直接对该值使用Number函数，不再进行后续步骤。
2. 第二步，如果valueOf方法返回的还是对象，则改为调用对象自身的toString方法。如果toString方法返回原始类型的值，则对该值使用Number函数，不再进行后续步骤。
3. 第三步，如果toString方法返回的是对象，就报错

```js
Number({a: 1}) // NaN
Number([1, 2, 3]) // NaN
Number([5]) // 5
```

#### String()

##### 原始类型值

* 数值：转为相应的字符串。
* 字符串：转换后还是原来的值。
* 布尔值：true转为字符串"true"，false转为字符串"false"。
* undefined：转为字符串"undefined"。
* null：转为字符串"null"。

##### 对象

> String方法的参数如果是对象，返回一个类型字符串；如果是数组，返回该数组的字符串形式。

* 先调用对象自身的toString方法。如果返回原始类型的值，则对该值使用String函数，不再进行以下步骤。
* 如果toString方法返回的是对象，再调用原对象的valueOf方法。如果valueOf方法返回原始类型的值，则对该值使用String函数，不再进行以下步骤。
* 如果valueOf方法返回的是对象，就报错。

```js
String({a: 1}) // "[object Object]"
String([1, 2, 3]) // "1,2,3"
```

如果toString法和valueOf方法，返回的都是对象，就会报错。
```js
var obj = {
  valueOf: function () {
    return {};
  },
  toString: function () {
    return {};
  }
};

String(obj)
// TypeError: Cannot convert object to primitive value
```

#### Boolean()

> Boolean()函数可以将任意类型的值转为布尔值。

**它的转换规则相对简单：除了以下五个值的转换结果为false，其他的值全部为true。**
* undefined
* null
* 0（包含-0和+0）
* NaN
* ''（空字符串）

### 自动转换
自动转换的规则是这样的：预期什么类型的值，就调用该类型的转换函数。比如，某个位置预期为字符串，就调用String()函数进行转换。如果该位置既可以是字符串，也可能是数值，那么默认转为数值。

> 遇到以下三种情况时，JavaScript 会自动转换数据类型，即转换是自动完成的，用户不可见

1. 遇到以下三种情况时，JavaScript 会自动转换数据类型，即转换是自动完成的，用户不可见
  ```js
    123 + 'abc' // "123abc"
  ```
2. 对非布尔值类型的数据求布尔值。
  ```js
    if ('abc') {
      console.log('hello')
    }  // "hello"
  ```
3. 对非数值类型的值使用一元运算符（即+和-）。
  ```js
    + {foo: 'bar'} // NaN
    - [1, 2, 3] // NaN
  ```

1. 自动转换为布尔值
> JavaScript 遇到预期为布尔值的地方（比如if语句的条件部分），就会将非布尔值的参数自动转换为布尔值。系统内部会自动调用Boolean()函数。
2. 自动转换为字符串
> JavaScript 遇到预期为字符串的地方，就会将非字符串的值自动转为字符串。具体规则是，先将复合类型的值转为原始类型的值，再将原始类型的值转为字符串。
3. 自动转换为数值
> JavaScript 遇到预期为数值的地方，就会将参数值自动转换为数值。系统内部会自动调用Number()函数。

### 错误处理 Error

JavaScript 解析或运行时，一旦发生错误，引擎就会抛出一个错误对象。JavaScript 原生提供Error构造函数，所有抛出的错误都是这个构造函数的实例。
```js
var err = new Error('出错了');
err.message // "出错了"
```

#### 原生错误类型
1. SyntaxError 对象
  > SyntaxError对象是解析代码时发生的语法错误。
2. ReferenceError 对象
  > ReferenceError对象是引用一个不存在的变量时发生的错误。
3. RangeError 对象
  > RangeError对象是一个值超出有效范围时发生的错误。主要有几种情况，一是数组长度为负数，二是Number对象的方法参数超出范围，以及函数堆栈超过最大值。
4. TypeError 对象
  > TypeError对象是变量或参数不是预期类型时发生的错误。比如，对字符串、布尔值、数值等原始类型的值使用new命令，就会抛出这种错误，因为new命令的参数应该是一个构造函数。
5. URIError 对象
  > URIError对象是 URI 相关函数的参数不正确时抛出的错误，主要涉及encodeURI()、decodeURI()、encodeURIComponent()、decodeURIComponent()、escape()和unescape()这六个函数。
6. EvalError 对象
  > eval函数没有被正确执行时，会抛出EvalError错误。该错误类型已经不再使用了，只是为了保证与以前代码兼容，才继续保留。

**总结以上这6种派生错误，连同原始的Error对象，都是构造函数。开发者可以使用它们，手动生成错误对象的实例。这些构造函数都接受一个参数，代表错误提示信息（message）。**

```js
var err1 = new Error('出错了！');
var err2 = new RangeError('出错了，变量超出有效范围！');
var err3 = new TypeError('出错了，变量类型无效！');

err1.message // "出错了！"
err2.message // "出错了，变量超出有效范围！"
err3.message // "出错了，变量类型无效！"
```

#### throw语句

> throw语句的作用是手动中断程序执行，抛出一个错误。
```js
if (x <= 0) {
  throw new Error('x 必须为正数');
}
```

#### try...catch 结构
```js
try {
  throw new Error('出错了!');
} catch (e) {
  console.log(e.name + ": " + e.message);
  console.log(e.stack);
}
```

#### finally 代码块

> try...catch结构允许在最后添加一个finally代码块，表示不管是否出现错误，都必需在最后运行的语句。

finally代码块用法的典型场景。
```js
openFile();

try {
  writeFile(Data);
} catch(e) {
  handleError(e);
} finally {
  closeFile();
}
```