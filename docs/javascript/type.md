## 数据类型

### 简介
> JavaScript 语言的每一个值，都属于某一种数据类型。JavaScript 的数据类型，共有六种。

1. 数值（number）：整数和小数（比如1和3.14）。
2. 字符串（string）：文本（比如Hello World）。
3. 布尔值（boolean）：表示真伪的两个特殊值，即true（真）和false（假）。
4. undefined：表示“未定义”或不存在，即由于目前没有定义，所以此处暂时没有任何值。
5. null：表示空值，即此处的值为空。
6. 对象（object）：各种值组成的集合。

> 对象是最复杂的数据类型，又可以分成三个子类型。
* 狭义的对象（object）
* 数组（array）
* 函数（function）

### typeof 运算符

> JavaScript 有三种方法，可以确定一个值到底是什么类型。

* typeof运算符
* instanceof运算符
* Object.prototype.toString方法

> typeof运算符可以返回一个值的数据类型。

```js
typeof 123 // "number"
typeof '123' // "string"
typeof false // "boolean"
function f() {}
typeof f // "function"
typeof undefined // "undefined"
```

> 对象返回object。
```js
typeof window // "object"
typeof {} // "object"
typeof [] // "object"
typeof null // "object" 历史原因： 1995年的 JavaScript 语言第一版，只设计了五种数据类型（对象、整数、浮点数、字符串和布尔值），没考虑null，只把它当作object的一种特殊值。后来null独立出来，作为一种单独的数据类型，为了兼容以前的代码，typeof null返回object就没法改变了。
```

### null 和 undefined
> null 和 undefined都可以表示“没有”，含义非常相似。

在 if 语句中，它们都会被转化为false，相等运算符（==）认为两者相等
```js
if (!undefined) {
  console.log('undefined is false');
}
// undefined is false

if (!null) {
  console.log('null is false');
}
// null is false

undefined == null
// true
```

> null 是一个表示“空”的对象，转为数值时为0；undefined 是一个表示"此处无定义"的原始值，转为数值时为 NaN。
```js
Number(null) // 0
Number(undefined) // NaN
```

#### 用法和含义

> null表示空值，即该处的值现在为空。调用函数时，某个参数未设置任何值，这时就可以传入null，表示该参数为空。

> undefined表示“未定义”，下面是返回undefined的典型场景。

```js
// 变量声明了，但没有赋值
var i;
i // undefined

// 调用函数时，应该提供的参数没有提供，该参数等于 undefined
function f(x) {
  return x;
}
f() // undefined

// 对象没有赋值的属性
var  o = new Object();
o.p // undefined

// 函数没有返回值时，默认返回 undefined
function f() {}
f() // undefined
```

### 布尔值

> 布尔值代表“真”（true）和“假”（false）两个状态。

> 下列运算符会返回布尔值：
1. 前置逻辑运算符： ! (Not)
2. 相等运算符：===，!==，==，!=
3. 比较运算符：>，>=，<，<=

> 转换规则是除了下面六个值被转为false，其他值都视为true。
1. undefined
2. null
3. false
4. 0
5. NaN
6. ""或''（空字符串）

> 空数组（[]）和空对象（{}）对应的布尔值，都是true。
```js
if ([]) {
  console.log('true');
}
// true

if ({}) {
  console.log('true');
}
// true
```

### 数值 (整数和浮点数)

> JavaScript 内部，所有数字都是以64位浮点数形式储存，即使整数也是如此。
```js
1 === 1.0  // 数字都是以64位浮点数形式，所以相等
```
#### NaN
> NaN是 JavaScript 的特殊值，表示“非数字”（Not a Number），主要出现在将字符串解析成数字出错的场合。
```js
5 - 'x' // NaN
0 / 0  // NaN
```

> NaN不等于任何值，包括它本身。
```js
NaN === NaN // false
```

> 数组的indexOf方法内部使用的是严格相等运算符，所以该方法对NaN不成立。
```js
[NaN].indexOf(NaN) // -1
```

> NaN在布尔运算时被当作false。
```js
Boolean(NaN) // false
```

> NaN与任何数（包括它自己）的运算，得到的都是NaN。
```js
NaN + 32 // NaN
NaN - 32 // NaN
NaN * 32 // NaN
NaN / 32 // NaN
```

#### 与数值相关的全局方法

##### parseInt()

> parseInt 方法用于将字符串转为整数。
```js
parseInt('123') // 123
```

> 如果字符串头部有空格，空格会被自动去除。
```js
parseInt('   81') // 81
```

> 如果parseInt 的参数不是字符串，则会先转为字符串再转换。
```js
parseInt(1.23) // 1
// 等同于
parseInt('1.23') // 1
```

> 字符串转为整数的时候，是一个个字符依次转换，如果遇到不能转为数字的字符，就不再进行下去，返回已经转好的部分。
```js
parseInt('12**') // 12
parseInt('12.34') // 12
```

> 如果字符串的第一个字符不能转化为数字（后面跟着数字的正负号除外），返回NaN
```js
parseInt('abc') // NaN
parseInt('.3') // NaN
parseInt('') // NaN
parseInt('+') // NaN
parseInt('+1') // 1
```

**parseInt 的返回值只有两种可能，要么是一个十进制整数，要么是NaN**

##### parseFloat()
> parseFloat方法用于将一个字符串转为浮点数。
```js
parseFloat('3.14') // 3.14
```

> 如果字符串包含不能转为浮点数的字符，则不再进行往后转换，返回已经转好的部分。
```js
parseFloat('3.14more non-digit characters') // 3.14
```

> parseFloat方法会自动过滤字符串前导的空格。
```js
parseFloat('\t\v\r12.34\n ') // 12.34
```

> 参数不是字符串，或者字符串的第一个字符不能转化为浮点数，则返回NaN。
```js
parseFloat([]) // NaN
parseFloat('FF2') // NaN
parseFloat('') // NaN
```

### 字符串

> 字符串就是零个或多个排在一起的字符，放在单引号或双引号之中。

#### 字符串与数组

> 字符串可以被视为字符数组，因此可以使用数组的方括号运算符，用来返回某个位置的字符（位置编号从0开始）。
```js
var s = 'hello';
s[0] // "h"
s[1] // "e"
s[4] // "o"

// 直接对字符串使用方括号运算符
'hello'[1] // "e"
```

> 如果方括号中的数字超过字符串的长度，或者方括号中根本不是数字，则返回undefined。
```js
'abc'[3] // undefined
'abc'[-1] // undefined
'abc'['x'] // undefined
```

**字符串与数组的相似性仅此而已。实际上，无法改变字符串之中的单个字符。**

#### 字符串的 length 属性
> length属性返回字符串的长度，该属性也是无法改变的。


### 对象

> 对象就是一组“键值对”（key-value）的集合，是一种无序的复合数据集合。

```js
var obj = {
  foo: 'Hello'
}
// 其中foo是“键名”（成员的名称），字符串Hello是“键值”（成员的值）。键名与键值之间用冒号分隔
```

> 对象的所有键名都是字符串（ES6 又引入了 Symbol 值也可以作为键名），所以加不加引号都可以

> 如果键名是数值，会被自动转为字符串。

```js
var obj = {
  100: true
}
obj[100] // true obj['100']
```

#### 对象的引用

不同的变量名指向同一个对象，那么它们都是这个对象的引用，也就是说指向同一个内存地址。修改其中一个变量，会影响到其他所有变量。这种引用只局限于对象，如果两个变量指向同一个原始类型的值。那么，变量这时都是值的拷贝。

#### 对象属性的读取操作

> 读取对象的属性，有两种方法，一种是使用点运算符，还有一种是使用方括号运算符。
```js
var obj = {
  p: 'Hello World'
};

obj.p // "Hello World"
obj['p'] // "Hello World"
```

> 使用方括号运算符，键名必须放在引号里面，否则会被当作变量处理。
```js
var foo = 'bar';

var obj = {
  foo: 1,
  bar: 2
};

obj.foo  // 1
obj[foo]  // 2
```

> 方括号运算符内部还可以使用表达式。
```js
obj['hello' + ' world']
obj[3 + 3]
```

#### 对象属性的赋值操作

> 点运算符和方括号运算符，不仅可以用来读取值，还可以用来赋值。
```js
var obj = {};

obj.foo = 'Hello';
obj['bar'] = 'World';
```

#### 对象属性的查看操作

> 查看一个对象本身的所有属性，可以使用Object.keys方法。
```js
var obj = {
  key1: 1,
  key2: 2
};

Object.keys(obj); // ['key1', 'key2']
```

#### 属性的删除操作

> delete命令用于删除对象的属性，删除成功后返回true。
```js
var obj = { p: 1 };
Object.keys(obj) // ["p"]

delete obj.p // true
obj.p // undefined
Object.keys(obj) // []
```

> 删除一个不存在的属性，delete不报错，而且返回true。
```js
var obj = {};
delete obj.p // true
```

> 只有一种情况，delete命令会返回false，那就是该属性存在，且不得删除。
```js
var obj = Object.defineProperty({}, 'p', {
  value: 123,
  configurable: false
});

obj.p // 123
delete obj.p // false
```
**上面代码之中，对象obj的p属性是不能删除的，所以delete命令返回false**

**需要注意的是，delete命令只能删除对象本身的属性，无法删除继承的属性**
```js
var obj = {};
delete obj.toString // true
obj.toString // function toString() { [native code] }  toString是从Object继承来的属性，所以不能删除。
```

#### 属性是否存在：in 运算符

> in运算符用于检查对象是否包含某个属性（注意，检查的是键名，不是键值）。
```js
var obj = { p: 1 };
'p' in obj // true
'toString' in obj // true
```

> in运算符的一个问题是，它不能识别哪些属性是对象自身的，哪些属性是继承的。可以使用对象的hasOwnProperty方法判断一下，是否为对象自身的属性。
```js
var obj = {};
if ('toString' in obj) {
  console.log(obj.hasOwnProperty('toString')) // false
}
```

#### 属性的遍历：for...in 循环
```js
var obj = {a: 1, b: 2, c: 3};

for (var i in obj) {
  console.log('键名：', i);
  console.log('键值：', obj[i]);
}
// 键名： a
// 键值： 1
// 键名： b
// 键值： 2
// 键名： c
// 键值： 3
```

for...in循环有两个使用注意点。

1. 它遍历的是对象所有可遍历（enumerable）的属性，会跳过不可遍历的属性。
2. 它不仅遍历对象自身的属性，还遍历继承的属性。

> **对象都继承了toString属性，但是for...in循环不会遍历到这个属性。**
```js
var obj = {};

// toString 属性是存在的
obj.toString // toString() { [native code] }

for (var p in obj) {
  console.log(p);
} // 没有任何输出
```

**对象obj继承了toString属性，该属性不会被for...in循环遍历到，因为它默认是“不可遍历”的。**

> 继承的属性是可遍历的，那么就会被for...in循环遍历到。但是，一般情况下，都是只想遍历对象自身的属性，所以使用for...in的时候，应该结合使用hasOwnProperty方法，在循环内部判断一下，某个属性是否为对象自身的属性。
```js
var person = { name: '老张' };

for (var key in person) {
  if (person.hasOwnProperty(key)) {
    console.log(key);
  }
}
// name
```