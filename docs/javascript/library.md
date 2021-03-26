## 标准库

### Object

> Object对象本身的方法,就是直接定义在Object对象的方法。

> 实例方法就是定义在Object原型对象Object.prototype上的方法。它可以被Object实例直接使用。

#### Object()

> Object本身是一个函数，可以当作工具方法使用，将任意值转为对象。这个方法常用于保证某个值一定是对象。
```js
var obj = Object();
// 等同于
var obj = Object(undefined);
var obj = Object(null);

obj instanceof Object // true
```
**instanceof运算符用来验证，一个对象是否为指定的构造函数的实例。obj instanceof Object返回true，就表示obj对象是Object的实例。**

#### Object 构造函数

> Object构造函数的首要用途，是直接通过它来生成新对象。

Object构造函数的用法与工具方法很相似，几乎一模一样。使用时，可以接受一个参数，如果该参数是一个对象，则直接返回这个对象；如果是一个原始类型的值，则返回该值对应的包装对象

#### Object的静态方法

> Object.keys()，Object.getOwnPropertyNames()

Object.keys()和Object.getOwnPropertyNames()返回的结果是一样的。只有涉及不可枚举属性时，才会有不一样的结果。Object.keys方法只返回可枚举的属性，Object.getOwnPropertyNames方法还返回不可枚举的属性名。


> 对象属性模型的相关方法

* Object.getOwnPropertyDescriptor()：获取某个属性的描述对象。
* Object.defineProperty()：通过描述对象，定义某个属性。
* Object.defineProperties()：通过描述对象，定义多个属性。

> 控制对象状态的方法

* Object.preventExtensions()：防止对象扩展。
* Object.isExtensible()：判断对象是否可扩展。
* Object.seal()：禁止对象配置。
* Object.isSealed()：判断一个对象是否可配置。
* Object.freeze()：冻结一个对象。
* Object.isFrozen()：判断一个对象是否被冻结。

> 原型链相关方法

* Object.create()：该方法可以指定原型对象和属性，返回一个新的对象。
* Object.getPrototypeOf()：获取对象的Prototype对象。

> Object 的实例方法

* Object.prototype.valueOf()：返回当前对象对应的值。
* Object.prototype.toString()：返回当前对象对应的字符串形式。
* Object.prototype.toLocaleString()：返回当前对象对应的本地字符串形式。
* Object.prototype.hasOwnProperty()：判断某个属性是否为当前对象自身的属性，还是继承自原型对象的属性。
* Object.prototype.isPrototypeOf()：判断当前对象是否为另一个对象的原型。
* Object.prototype.propertyIsEnumerable()：判断某个属性是否可枚举。

> 判断类型 Object.prototype.toString.call()

* 数值：返回 [object Number]。
* 字符串：返回 [object String]。
* 布尔值：返回 [object Boolean]。
* undefined：返回 [object Undefined]。
* null：返回 [object Null]。
* 数组：返回 [object Array]。
* arguments 对象：返回 [object Arguments]。
* 函数：返回 [object Function]。
* Error 对象：返回 [object Error]。
* Date 对象：返回 [object Date]。
* RegExp 对象：返回 [object RegExp]。
* 其他对象：返回 [object Object]。

### 属性描述对象

> JavaScript 提供了一个内部数据结构，用来描述对象的属性，控制它的行为
```js
{
  value: 123,
  writable: false,
  enumerable: true,
  configurable: false,
  get: undefined,
  set: undefined
}
```
1. value是该属性的属性值，默认为undefined。
2. writable是一个布尔值，表示属性值（value）是否可改变（即是否可写），默认为true。
3. enumerable是一个布尔值，表示该属性是否可遍历，默认为true。如果设为false，会使得某些操作（比如for...in循环、Object.keys()、JSON.stringify方法）跳过该属性。
4. configurable是一个布尔值，表示可配置性，默认为true。如果设为false，将阻止某些操作改写该属性，比如无法删除该属性，也不得改变该属性的属性描述对象（value属性除外）。也就是说，configurable属性控制了属性描述对象的可写性。
5. get是一个函数，表示该属性的取值函数（getter），默认为undefined。
6. set是一个函数，表示该属性的存值函数（setter），默认为undefined。

#### Object.getOwnPropertyDescriptor()

> Object.getOwnPropertyDescriptor()方法可以获取属性描述对象。它的第一个参数是目标对象，第二个参数是一个字符串，对应目标对象的某个属性名。

**注意，Object.getOwnPropertyDescriptor()方法只能用于对象自身的属性，不能用于继承的属性。**

#### Object.getOwnPropertyNames()

> Object.getOwnPropertyNames方法返回一个数组，成员是参数对象自身的全部属性的属性名，不管该属性是否可遍历。

#### Object.defineProperty()，Object.defineProperties()

> Object.defineProperty()方法允许通过属性描述对象，定义或修改一个属性，然后返回修改后的对象，它的用法如下。
```js
Object.defineProperty(object, propertyName, attributesObject)
```
Object.defineProperty方法接受三个参数，依次如下。

1. object：属性所在的对象
2. propertyName：字符串，表示属性名
3. attributesObject：属性描述对象

```js
var obj = Object.defineProperty({}, 'p', {
  value: 123,
  writable: false,
  enumerable: true,
  configurable: false
});

obj.p // 123

obj.p = 246;
obj.p // 123
```

> 一次性定义或修改多个属性，可以使用Object.defineProperties()方法。
```js
var obj = Object.defineProperties({}, {
  p1: { value: 123, enumerable: true },
  p2: { value: 'abc', enumerable: true },
  p3: { get: function () { return this.p1 + this.p2 },
    enumerable:true,
    configurable:true
  }
});

obj.p1 // 123
obj.p2 // "abc"
obj.p3 // "123abc"
```

### Array对象

> Array是 JavaScript 的原生对象，同时也是一个构造函数，可以用它生成新的数组。

#### 静态方法

Array.isArray()  方法返回一个布尔值，表示参数是否为数组

#### 实例方法

> 数组方法中，返回的还是数组，就可以链式调用。

valueOf() 方法是一个所有对象都拥有的方法，表示对该对象求值

toString() 方法也是对象的通用方法，数组的toString方法返回数组的字符串形式

push() 方法用于在数组的末端添加一个或多个元素，并返回添加新元素后的数组长度。注意，该方法会改变原数组。

pop() 方法用于删除数组的最后一个元素，并返回该元素。注意，该方法会改变原数组。

shift() 方法用于删除数组的第一个元素，并返回该元素。注意，该方法会改变原数组。

unshift() 方法用于在数组的第一个位置添加元素，并返回添加新元素后的数组长度。注意，该方法会改变原数组。

join() 方法以指定参数作为分隔符，将所有数组成员连接为一个字符串返回。如果不提供参数，默认用逗号分隔。

concat() 方法用于多个数组的合并。它将新数组的成员，添加到原数组成员的后部，然后返回一个新数组，原数组不变。

reverse() 方法用于颠倒排列数组元素，返回改变后的数组。注意，该方法将改变原数组。

slice(start, end) 方法用于提取目标数组的一部分，返回一个新数组，原数组不变。

sort() 方法对数组成员进行排序，默认是按照字典顺序排序。排序后，原数组将被改变。

splice() 方法用于删除原数组的一部分成员，并可以在删除的位置添加新的数组成员，返回值是被删除的元素。注意，该方法会改变原数组。
```js
// splice的第一个参数是删除的起始位置（从0开始），第二个参数是被删除的元素个数。如果后面还有更多的参数，则表示这些就是要被插入数组的新元素。
arr.splice(start, count, addElement1, addElement2, ...);
```

map() 方法将数组的所有成员依次传入参数函数，然后把每一次的执行结果组成一个新数组返回。

forEach() 方法与map() 方法很相似，也是对数组的所有成员依次执行参数函数。但是，forEach方法不返回值，只用来操作数据。

filter() 方法用于过滤数组成员，满足条件的成员组成一个新数组返回。

some() 方法是只要一个成员的返回值是true，则整个some方法的返回值就是true，否则返回false。

every() 方法是所有成员的返回值都是true，整个every方法才返回true，否则返回false。

indexOf() 方法返回给定元素在数组中第一次出现的位置，如果没有出现则返回-1。

lastIndexOf() 方法返回给定元素在数组中最后一次出现的位置，如果没有出现则返回-1。

reduce() 方法和reduceRight() 方法的第一个参数都是一个函数。该函数接受以下四个参数。

* 累积变量，默认为数组的第一个成员 （）
* 当前变量，默认为数组的第二个成员
* 当前位置（从0开始）
* 原数组

```js
[1, 2, 3, 4, 5].reduce(function (a, b) {
  return a + b;
}, 10);
```

### 包装对象

> 所谓“包装对象”，指的是与数值、字符串、布尔值分别相对应的Number、String、Boolean三个原生对象。这三个原生对象可以把原始类型的值变成（包装成）对象。

#### 实例方法

1. valueOf()方法返回包装对象实例对应的原始类型的值。

```js
new Number(123).valueOf()  // 123
new String('abc').valueOf() // "abc"
new Boolean(true).valueOf() // true
```

2. toString()方法返回对应的字符串形式。

```js
new Number(123).toString() // "123"
new String('abc').toString() // "abc"
new Boolean(true).toString() // "true"
```

#### 原始类型与实例对象的自动转换

> 某些场合，原始类型的值会自动当作包装对象调用，即调用包装对象的属性和方法。这时，JavaScript 引擎会自动将原始类型的值转为包装对象实例，并在使用后立刻销毁实例。

### Boolean对象

> Boolean对象是 JavaScript 的三个包装对象之一。作为构造函数，它主要用于生成布尔值的包装对象实例。

> Boolean对象除了可以作为构造函数，还可以单独使用，将任意值转为布尔值

### Number对象

> Number对象是数值对应的包装对象，可以作为构造函数使用，也可以作为工具函数使用。

#### 静态属性

1. Number.POSITIVE_INFINITY：正的无限，指向Infinity。
2. Number.NEGATIVE_INFINITY：负的无限，指向-Infinity。
3. Number.NaN：表示非数值，指向NaN。
4. Number.MIN_VALUE：表示最小的正数（即最接近0的正数，在64位浮点数体系中为5e-324），相应的，最接近0的负数为-Number.MIN_VALUE。
5. Number.MAX_SAFE_INTEGER：表示能够精确表示的最大整数，即9007199254740991。
6. Number.MIN_SAFE_INTEGER：表示能够精确表示的最小整数，即-9007199254740991。

#### 实例方法

1. Number.prototype.toString() Number对象部署了自己的toString方法，用来将一个数值转为字符串形式。
2. Number.prototype.toFixed() 将一个数转为指定位数的小数，然后返回这个小数对应的字符串。

### String对象

> String对象是 JavaScript 原生提供的三个包装对象之一，用来生成字符串对象。

#### 静态方法

1. String.fromCharCode() 该方法的参数是一个或多个数值，代表 Unicode 码点，返回值是这些码点组成的字符串
```js
String.fromCharCode(97) // "a"
```

#### 实例属性

1. String.prototype.length 字符串实例的length属性返回字符串的长度。

#### 实例方法

* String.prototype.charAt(index) 返回指定位置的字符，参数是从0开始编号的位置。
* String.prototype.concat() 连接两个字符串，返回一个新字符串，不改变原字符串。
* String.prototype.slice(start, end) 用于从原字符串取出子字符串并返回，不改变原字符串
* String.prototype.substring(start, end) 用于从原字符串取出子字符串并返回，不改变原字符串
* String.prototype.substr(start, how) 用于从原字符串取出子字符串并返回，不改变原字符串。第一个参数是子字符串的开始位置（从0开始计算），第二个参数是子字符串的长度
* String.prototype.indexOf() 用于确定一个字符串在另一个字符串中第一次出现的位置，返回结果是匹配开始的位置
* String.prototype.trim() 用于去除字符串两端的空格，返回一个新字符串，不改变原字符串。
* String.prototype.toLowerCase() 用于将一个字符串全部转为小写
* String.prototype.toUpperCase() 用于将一个字符串全部转为大写

> 以下四个方法都可以使用正则表达式
* String.prototype.match() 用于确定原字符串是否匹配某个子字符串，返回一个数组，成员为匹配的第一个字符串。
* String.prototype.search() 用法基本等同于match，但是返回值为匹配的第一个位置。如果没有找到匹配，则返回-1。
* String.prototype.replace() 用于替换匹配的子字符串，一般情况下只替换第一个匹配（除非使用带有g修饰符的正则表达式）。
* String.prototype.split() 按照给定规则分割字符串，返回一个由分割出来的子字符串组成的数组。

### Math对象

> Math是 JavaScript 的原生对象，提供各种数学功能。该对象不是构造函数，不能生成实例，所有的属性和方法都必须在Math对象上调用。

* Math.abs()：绝对值
* Math.ceil()：向上取整
* Math.floor()：向下取整
* Math.max()：最大值
* Math.min()：最小值
* Math.pow()：幂运算
* Math.sqrt()：平方根
* Math.log()：自然对数
* Math.exp()：e的指数
* Math.round()：四舍五入
* Math.random()：随机数

### Date对象

### RegExp对象

#### 实例属性

* RegExp.prototype.ignoreCase：返回一个布尔值，表示是否设置了i修饰符。
* RegExp.prototype.global：返回一个布尔值，表示是否设置了g修饰符。
* RegExp.prototype.multiline：返回一个布尔值，表示是否设置了m修饰符。
* RegExp.prototype.flags：返回一个字符串，包含了已经设置的所有修饰符，按字母排序。

#### 实例方法

RegExp.prototype.test() 返回一个布尔值，表示当前模式是否能匹配参数字符串。
```js
/cat/.test('cats and dogs') // true
```

RegExp.prototype.exec() 用来返回匹配结果。如果发现匹配，就返回一个数组，成员是匹配成功的子字符串，否则返回null。
```js
var s = '_x_x';
var r1 = /x/;
var r2 = /y/;

r1.exec(s) // ["x"]
r2.exec(s) // null
```

#### 匹配规则

1. 字面量字符
```js
/dog/.test('old dog') // true
```
2. 点字符（.) 点字符（.）匹配除回车（\r）、换行(\n) 、行分隔符（\u2028）和段分隔符（\u2029）以外的所有字符。
3. ^ 表示字符串的开始位置。
4. $ 表示字符串的结束位置。
5. 选择符（|）竖线符号（|）在正则表达式中表示“或关系”（OR），即cat|dog表示匹配cat或dog。
```js
/11|22|33/.test('911') // true
```
6. 转义符\ 正则表达式中那些有特殊含义的元字符，如果要匹配它们本身，就需要在它们前面要加上反斜杠。
7. [xyz] 表示x、y、z之中任选一个匹配。
8. 方括号内的第一个字符是[^]，则表示除了字符类之中的字符，其他字符都可以匹配。比如，[^xyz]表示除了x、y、z之外都可以匹配。
```js
/[^abc]/.test('bbc news') // true
/[^abc]/.test('bbc') // false
```
9. 连字符（-）用来提供简写形式，表示字符的连续范围。比如，[abc]可以写成[a-c]，[0123456789]可以写成[0-9]，同理[A-Z]表示26个大写字母。
```js
/a-z/.test('b') // false
/[a-z]/.test('b') // true
```

#### 预定义模式

* \d 匹配0-9之间的任一数字，相当于[0-9]。
* \D 匹配所有0-9以外的字符，相当于[^0-9]。
* \w 匹配任意的字母、数字和下划线，相当于[A-Za-z0-9_]。
* \W 除所有字母、数字和下划线以外的字符，相当于[^A-Za-z0-9_]。
* \s 匹配空格（包括换行符、制表符、空格符等），相等于[ \t\r\n\v\f]。
* \S 匹配非空格的字符，相当于[^ \t\r\n\v\f]。
* \b 匹配词的边界。
* \B 匹配非词边界，即在词的内部。

#### 重复类

> 模式的精确匹配次数，使用大括号（{}）表示。{n}表示恰好重复n次，{n,}表示至少重复n次，{n,m}表示重复不少于n次，不多于m次。

#### 量词符

> 量词符用来设定某个模式出现的次数。

* ? 问号表示某个模式出现0次或1次，等同于{0, 1}。
* * 星号表示某个模式出现0次或多次，等同于{0,}。
* + 加号表示某个模式出现1次或多次，等同于{1,}。

#### 贪婪模式

> 默认情况下都是最大可能匹配，即匹配到下一个字符不满足匹配规则为止。
```js
var s = 'aaa';
s.match(/a+/) // ["aaa"]
```

#### 非贪婪模式

> 发现匹配，就返回结果，不要往下检查。如果想将贪婪模式改为非贪婪模式，可以在量词符后面加一个问号。
```js
var s = 'aaa';
s.match(/a+?/) // ["a"]
```

* +?：表示某个模式出现1次或多次，匹配时采用非贪婪模式。
* *?：表示某个模式出现0次或多次，匹配时采用非贪婪模式。
* ??：表格某个模式出现0次或1次，匹配时采用非贪婪模式。

#### 修饰符

> 修饰符（modifier）表示模式的附加规则，放在正则模式的最尾部。修饰符可以单个使用，也可以多个一起使用。

g 修饰符 表示全局匹配（global），加上它以后，正则对象将匹配全部符合条件的结果，主要用于搜索和替换。
i 修饰符以后表示忽略大小写（ignoreCase）

#### 组匹配

> 正则表达式的括号表示分组匹配，括号中的模式可以用来匹配分组的内容。

TODO

### JSON对象

> JSON 格式（JavaScript Object Notation 的缩写）是一种用于数据交换的文本格式。

> JSON 对值的类型和格式有严格的规定。
  > 1. 复合类型的值只能是数组或对象，不能是函数、正则表达式对象、日期对象。
  > 2. 原始类型的值只有四种：字符串、数值（必须以十进制表示）、布尔值和null（不能使用NaN, Infinity, -Infinity和undefined）。
  > 3. 字符串必须使用双引号表示，不能使用单引号。
  > 4. 对象的键名必须放在双引号里面。
  > 5. 数组或对象最后一个成员的后面，不能加逗号。


#### 静态方法

1. JSON.stringify()方法用于将一个值转为 JSON 字符串。该字符串符合 JSON 格式，并且可以被JSON.parse()方法还原。
  > 如果对象的属性是undefined、函数或 XML 对象，该属性会被JSON.stringify()过滤
  > JSON.stringify()方法还可以接受一个数组，作为第二个参数，指定参数对象的哪些属性需要转成字符串
  ```js
    JSON.stringify({0: 'a', 1: 'b'}, ['0']) // "{"0":"a"}"
  ```
  > JSON.stringify()还可以接受第三个参数，用于增加返回的 JSON 字符串的可读性。
  ```js
    // 分行输出
    JSON.stringify({ p1: 1, p2: 2 }, null, '\t')
  ```

2. JSON.parse()方法用于将 JSON 字符串转换成对应的值。

> 如果传入的字符串不是有效的 JSON 格式，JSON.parse()方法将报错。为了处理解析错误，可以将JSON.parse()方法放在try...catch代码块中。