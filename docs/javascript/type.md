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

### 函数

> 函数是一段可以反复调用的代码块。函数还能接受输入的参数，不同的参数会返回不同的值。

函数的声明有三种方式

> function命令
```js
function f(){}
```

> 函数表达式
```js
var print = function(){}
```

**函数表达式声明函数时，function命令后面不带有函数名。如果加上函数名，该函数名只在函数体内部有效，在函数体外部无效。**
```js
var print = function x(){
  console.log(typeof x);
};

x
// ReferenceError: x is not defined

print()
// function
```

> Function 构造函数
```js
var add = new Function(
  'x',
  'y',
  'return x +y'
)

// 等价于
function add (x,y){
  return x + y;
}
```

**可以传递任意数量的参数给Function构造函数，只有最后一个参数会被当做函数体，如果只有一个参数，该参数就是函数体**

#### 函数重复声明

> 同一个函数被多次声明，后面的声明就会覆盖前面的声明。

#### 函数的递归

> 函数可以调用自身，这就是递归（recursion）。递归函数一定要有一个终止条件，不然会导致堆栈溢出。

#### 函数是第一等公民

> JavaScript 语言将函数看作一种值，与其它值（数值、字符串、布尔值等等）地位相同。凡是可以使用值的地方，就能使用函数。比如，可以把函数赋值给变量和对象的属性，也可以当作参数传入其他函数，或者作为函数的结果返回

#### 函数名的提升

> JavaScript 引擎将函数名视同变量名，所以采用function命令声明函数时，整个函数会像变量声明一样，被提升到代码头部
```js
f()

function f(){}
```

表面上，上面代码好像在声明之前就调用了函数f。但是实际上，由于“变量提升”，函数f被提升到了代码头部，也就是在调用之前已经声明了。但是，如果采用赋值语句定义函数，JavaScript 就会报错。

```js
f();
var f = function (){};
// TypeError: undefined is not a function

// 等同于
var f;
f();
f = function () {};
```

> 采用function命令和var赋值语句声明同一个函数，由于存在函数提升，最后会采用var赋值语句的定义。
```js
var f = function () {
  console.log('1');
}

function f() {
  console.log('2');
}

f() // 1
```

#### 函数的属性和方法

> 函数的name属性返回函数的名字。

> 函数的length属性返回函数预期传入的参数个数，即函数定义之中的参数个数。

> 函数的toString()方法返回一个字符串，内容是函数的源码。

#### 函数的作用域

> 作用域（scope）指的是变量存在的范围。在 ES5 的规范中，JavaScript 只有两种作用域：一种是全局作用域，变量在整个程序中一直存在，所有地方都可以读取；另一种是函数作用域，变量只在函数内部存在。ES6 又新增了块级作用域

对于顶层函数来说，函数外部声明的变量就是全局变量（global variable），它可以在函数内部读取。
```js
var v = 1;

function f() {
  console.log(v);
}

f()
// 1
```

函数内部定义的变量，外部无法读取，称为“局部变量”（local variable）。
```js
function f(){
  var v = 1;
}

v // ReferenceError: v is not defined
```

**注意，对于var命令来说，局部变量只能在函数内部声明，在其他区块中声明，一律都是全局变量。**
```js
if (true) {
  var x = 5;
}
console.log(x);  // 5
```

#### 函数内部的变量提升

> 与全局作用域一样，函数作用域内部也会产生“变量提升”现象。var命令声明的变量，不管在什么位置，变量声明都会被提升到函数体的头部。
```js
function foo(x) {
  if (x > 100) {
    var tmp = x - 100;
  }
}

// 等同于
function foo(x) {
  var tmp;
  if (x > 100) {
    tmp = x - 100;
  };
}
```

#### 函数本身的作用域

> **函数本身也是一个值，也有自己的作用域。它的作用域与变量一样，就是其声明时所在的作用域，与其运行时所在的作用域无关**
```js
var a = 1;
var x = function () {
  console.log(a);
};

function f() {
  var a = 2;
  x();
}

f() // 1
```

**函数执行时所在的作用域，是定义时的作用域，而不是调用时所在的作用域。**

#### 参数

> 函数运行的时候，有时需要提供外部数据，不同的外部数据会得到不同的结果，这种外部数据就叫参数

1. 函数允许省略参数，但是没有办法只省略靠前的参数，而保留靠后的参数。如果一定要省略靠前的参数，只有显式传入undefined。
2. 函数参数如果是原始类型的值（数值、字符串、布尔值），传递方式是传值传递（passes by value）。这意味着，在函数体内修改参数值，不会影响到函数外部。
3. 如果函数参数是复合类型的值（数组、对象、其他函数），传递方式是传址传递（pass by reference）。也就是说，传入函数的原始值的地址，因此在函数内部修改参数，将会影响到原始值。
4. **如果函数内部修改的，不是参数对象的某个属性，而是替换掉整个参数，这时不会影响到原始值**。
5. 同名的参数，则取最后出现的那个值。


#### arguments

> JavaScript 允许函数有不定数目的参数，所以需要一种机制，可以在函数体内部读取所有参数。这就是arguments对象的由来。

**正常模式下，arguments对象可以在运行时修改。**
```js
var f = function(a, b) {
  arguments[0] = 3;
  arguments[1] = 2;
  return a + b;
}

f(1, 1) // 5
```

**严格模式下，arguments对象与函数参数不具有联动关系。也就是说，修改arguments对象不会影响到实际的函数参数。**
```js
var f = function(a, b) {
  'use strict'; // 开启严格模式
  arguments[0] = 3;
  arguments[1] = 2;
  return a + b;
}

f(1, 1) // 2
```

*需要注意的是，虽然arguments很像数组，但它是一个对象。数组专有的方法（比如slice和forEach），不能在arguments对象上直接使用。*

#### 闭包

> 闭包简单理解成“定义在一个函数内部的函数”。闭包的最大用处有两个，一个是可以读取外层函数内部的变量，另一个就是让这些变量始终保持在内存中，即闭包可以使得它诞生环境一直存在

#### 立即调用函数

> 函数定义后立即调用的解决方法，就是不要让function出现在行首，让引擎将其理解成一个表达式。最简单的处理，就是将其放在一个圆括号里面。
```js
(function(){ /* code */ }());
// 或者
(function(){ /* code */ })();
```

### 数组

> 数组（array）是按次序排列的一组值。每个值的位置都有编号（从0开始），整个数组用方括号表示。

1. 数组属于一种特殊的对象。typeof运算符会返回数组的类型是object。
2. Object.keys方法返回数组的所有键名。可以看到数组的键名就是整数0、1、2。
3. 数组的length属性，返回数组的成员数量。

#### in 运算符

> 检查某个键名是否存在的运算符in，适用于对象，也适用于数组。
```js
var arr = [ 'a', 'b', 'c' ];
2 in arr  // true
'2' in arr // true
4 in arr // false
```
**上面代码表明，数组存在键名为2的键。由于键名都是字符串，所以数值2会自动转成字符串**

> 注意，如果数组的某个位置是空位，in运算符返回false。
```js
var arr = [];
arr[100] = 'a';

100 in arr // true
1 in arr // false
```

#### for...in
> 可以遍历对象，也可以遍历数组，毕竟数组只是一种特殊对象。

#### 数组的空位
> 当数组的某个位置是空元素，即两个逗号之间没有任何值，我们称该数组存在空位（hole）。数组的空位不影响length属性。组的空位是可以读取的，返回undefined。

#### 类似数组的对象
> 一个对象的所有键名都是正整数或零，并且有length属性，那么这个对象就很像数组，语法上称为“类似数组的对象”（array-like object）
```js
var obj = {
  0: 'a',
  1: 'b',
  2: 'c',
  length: 3
};
```