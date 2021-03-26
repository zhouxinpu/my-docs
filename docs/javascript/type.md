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

### 用法和含义

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
### NaN
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