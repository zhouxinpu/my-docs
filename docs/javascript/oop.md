## 面向对象编程（OOP）

### “对象”（object）到底是什么？

1. 对象是单个实物的抽象。
2. 对象是一个容器，封装了属性（property）和方法（method）。

### 构造函数

> JavaScript 语言的对象体系，不是基于“类”的，而是基于构造函数（constructor）和原型链（prototype）。

> 构造函数就是普通的函数，但是具有自己的特征和用法。

```js
var Vehicle = function () {
  this.price = 1000;
};
```

上面代码中，Vehicle就是构造函数。为了与普通函数区别，构造函数名字的第一个字母通常大写。

构造函数的特点有两个。
1. 函数体内部使用了this关键字，代表了所要生成的对象实例。
2. 生成对象的时候，必须使用new命令。


### new 命令

> new命令的作用，就是执行构造函数，返回一个实例对象。

```js
var Vehicle = function (p) {
  this.price = p;
};

var v = new Vehicle(1000);
v.price // 1000
```

#### new 原理

1. 创建一个空对象，作为将要返回的对象实例。
2. 将这个空对象的原型，指向构造函数的prototype属性。
3. 将这个空对象赋值给函数内部的this关键字。
4. 开始执行构造函数内部的代码。

构造函数内部，this指的是一个新生成的空对象，所有针对this的操作，都会发生在这个空对象上。构造函数之所以叫“构造函数”，就是说这个函数的目的，就是操作一个空对象（即this对象），将其“构造”为需要的样子。

如果构造函数内部有return语句，而且return后面跟着一个对象，new命令会返回return语句指定的对象；否则，就会不管return语句，返回this对象。

#### new target

> 函数内部可以使用new.target属性。如果当前函数是new命令调用，new.target指向当前函数，否则为undefined。

```js
function f() {
  console.log(new.target === f);
}

f() // false
new f() // true
```

使用这个属性，可以判断函数调用的时候，是否使用new命令。

```js
function f() {
  if (!new.target) {
    throw new Error('请使用 new 命令调用！');
  }
  // ...
}

f() // Uncaught Error: 请使用 new 命令调用！
```

#### Object.create() 创建实例对象

> 构造函数作为模板，可以生成实例对象。但是，有时拿不到构造函数，只能拿到一个现有的对象。我们希望以这个现有的对象作为模板，生成新的实例对象，这时就可以使用Object.create()方法。

```js
var person1 = {
  name: '张三',
  age: 38,
  greeting: function() {
    console.log('Hi! I\'m ' + this.name + '.');
  }
};

var person2 = Object.create(person1);

person2.name // 张三
person2.greeting() // Hi! I'm 张三.
```

**对象person1是person2的模板，后者继承了前者的属性和方法。**


### this关键字

> this就是属性或方法“当前”所在的对象。它总是返回一个对象。

对象的属性可以赋给另一个对象，所以属性所在的当前对象是可变的，即this的指向是可变的。

```js
var A = {
  name: '张三',
  describe: function () {
    return '姓名：'+ this.name;
  }
};

var B = {
  name: '李四'
};

B.describe = A.describe;
B.describe()
// "姓名：李四"
```
面代码中，A.describe属性被赋给B，于是B.describe就表示describe方法所在的当前对象是B，所以this.name就指向B.name。

```js
var A = {
  name: '张三',
  describe: function () {
    return '姓名：'+ this.name;
  }
};

var name = '李四';
var f = A.describe;
f() // "姓名：李四"
```

只要函数被赋给另一个变量，this的指向就会变。(指向顶层对象window)

#### 实质

```js
var obj = { foo:  5 };
```
上面的代码将一个对象赋值给变量obj。JavaScript 引擎会先在内存里面，生成一个对象{ foo: 5 }，然后把这个对象的内存地址赋值给变量obj。也就是说，变量obj是一个地址（reference）。后面如果要读取obj.foo，引擎先从obj拿到内存地址，然后再从该地址读出原始的对象，返回它的foo属性。

****

```js
var obj = { foo: function () {} };
```

等同于

```js
var f = function () {};
var obj = { f: f };

// 单独执行
f()

// obj 环境执行
obj.f()
```

属性的值可能是一个函数。引擎会将函数单独保存在内存中，然后再将函数的地址赋值给foo属性的value属性.由于函数是一个单独的值，所以它可以在不同的环境（上下文）执行。

```js
var f = function () {
  console.log(x);
};```

上面代码中，函数体里面使用了变量x。该变量由运行环境提供。


**函数可以在不同的运行环境执行，所以需要有一种机制，能够在函数体内部获得当前的运行环境（context）。所以，this就出现了，它的设计目的就是在函数体内部，指代函数当前的运行环境。**

```js
var f = function () {
  console.log(this.x);
}

var x = 1;
var obj = {
  f: f,
  x: 2,
};

// 单独执行
f() // 1

// obj 环境执行
obj.f() // 2
```

#### this的使用场景

1. 全局环境，全局环境使用this，它指的就是顶层对象window。

```js
this === window // true

function f() {
  console.log(this === window);
}
f() // true
```

上面代码说明，不管是不是在函数内部，只要是在全局环境下运行，this就是指顶层对象window。

2. 构造函数中的this，指的是实例对象。

```js
var Obj = function (p) {
  this.p = p;   // this指向当前Obj实例
};
```

3. 对象的方法里面包含this，this的指向就是方法运行时所在的对象。该方法赋值给另一个对象，就会改变this的指向。

```js
var obj ={
  foo: function () {
    console.log(this);
  }
};

obj.foo() // obj
```

#### 绑定this的方法

##### Function.prototype.call() 

> 如果参数为空、null和undefined，则默认传入全局对象。call的第一个参数就是this所要指向的那个对象，后面的参数则是函数调用时所需的参数。

```js
var obj = {};

var f = function () {
  return this;
};

f() === window // true
f.call(obj) === obj // true
```

##### Function.prototype.apply()

> apply方法的作用与call方法类似，也是改变this指向，然后再调用该函数。唯一的区别就是，它接收一个数组作为函数执行时的参数。第一个参数也是this所要指向的那个对象，如果设为null或undefined，则等同于指定全局对象。

```js
func.apply(thisValue, [arg1, arg2, ...])
```

##### Function.prototype.bind()

> bind()方法用于将函数体内的this绑定到某个对象，然后返回一个新函数。

```js
var d = new Date();
d.getTime() // 1481869925657

var print = d.getTime;
print() // Uncaught TypeError: this is not a Date object.
```


### 对象的继承

> JavaScript 语言的继承不通过 class，而是通过“原型对象”（prototype）实现，本章介绍 JavaScript 的原型链继承。(ES6 引入了 class 语法)

#### 构造函数的缺点

```js
function Cat (name, color) {
  this.name = name;
  this.color = color;
}

var cat1 = new Cat('大毛', '白色');

cat1.name // '大毛'
cat1.color // '白色'
```

**Cat函数是一个构造函数，函数内部定义了name属性和color属性，所有实例对象（上例是cat1）都会生成这两个属性，即这两个属性会定义在实例对象上面。虽然很方便，但是有一个缺点。同一个构造函数的多个实例之间，无法共享属性，从而造成对系统资源的浪费。**

**这个问题的解决方法，就是 JavaScript 的原型对象（prototype）。**

#### prototype 属性的作用

1. JavaScript 规定，每个函数都有一个prototype属性，指向一个对象。

```js
function f() {}
typeof f.prototype // "object"
```

2. 属性和方法定义在原型上，那么所有实例对象就能共享，不仅节省了内存，还体现了实例对象之间的联系。

```js
function Animal(name) {
  this.name = name;
}
Animal.prototype.color = 'white';

var cat1 = new Animal('大毛');
var cat2 = new Animal('二毛');

cat1.color // 'white'
cat2.color // 'white'
```

3. 实例对象自身就有某个属性或方法，它就不会再去原型对象寻找这个属性或方法。

```js
cat1.color = 'black';

cat1.color // 'black'
cat2.color // 'yellow'
Animal.prototype.color // 'yellow';
```

**原型对象的作用，就是定义所有实例对象共享的属性和方法。这也是它被称为原型对象的原因，而实例对象可以视作从原型对象衍生出来的子对象。**

#### 原型链

> JavaScript 规定，所有对象都有自己的原型对象（prototype）。一方面，任何一个对象，都可以充当其他对象的原型；另一方面，由于原型对象也是对象，所以它也有自己的原型。因此，就会形成一个“原型链”（prototype chain）

如果一层层地上溯，所有对象的原型最终都可以上溯到Object.prototype，即Object构造函数的prototype属性。也就是说，所有对象都继承了Object.prototype的属性。这就是所有对象都有valueOf和toString方法的原因，因为这是从Object.prototype继承的。

Object.prototype对象有没有它的原型呢？回答是Object.prototype的原型是null。null没有任何属性和方法，也没有自己的原型。因此，原型链的尽头就是null。

```js
Object.getPrototypeOf(Object.prototype) // null
```

#### constructor 属性 # 

> prototype对象有一个constructor属性，默认指向prototype对象所在的构造函数。

```js
function P() {}
var p = new P();

p.constructor === P // true
p.constructor === P.prototype.constructor // true
p.hasOwnProperty('constructor') // false
```

上面代码中，p是构造函数P的实例对象，但是p自身没有constructor属性，该属性其实是读取原型链上面的P.prototype.constructor属性

constructor属性的作用是，可以得知某个实例对象，到底是哪一个构造函数产生的。

#### instanceof 运算符 # 

> instanceof运算符返回一个布尔值，表示对象是否为某个构造函数的实例。

```js
var v = new Vehicle();
v instanceof Vehicle // true

// 等同于
Vehicle.prototype.isPrototypeOf(v)
```

#### 构造函数的继承 # 

