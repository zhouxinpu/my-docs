## javascript 基础语法

### var 变量的赋值

> 使用 var 重新声明一个已经存在的变量，是无效的
```js
var x = 1;
var x;
x // 1
```

> 如果第二次声明的时候还进行了赋值，则会覆盖掉前面的值
```js
var x = 1;
var x = 2;

// 等同于
var x = 1;
var x;
x = 2;

x // 2
```

### 变量提升
> javascript引擎的工作方式，先解析代码，获取被声明的变量，然后再一行一行的运行。导致的结果，就是所有变量的声明语句，都会被提升到代码的头部，这叫做变量提升。

> var 定义的变量存在变量提升,

```js
console.log(x) // undefined
var x = 1;

// 等同于
var x;
console.log(x);
x = 1;
```

### 区块

> JavaScript 使用大括号，将多个相关的语句组合在一起，称为“区块”（block）

> 对于var命令来说，JavaScript 的区块不构成单独的作用域（scope）。

```js
{
  var x = 1;
}
a // 1
```

### switch

> switch语句后面的表达式，与case语句后面的表示式比较运行结果时，采用的是严格相等运算符（===），而不是相等运算符（==），这意味着比较时不会发生类型转换。

### break语句和continue语句

> break语句和continue语句都具有跳转作用，可以让代码不按既有的顺序执行。

> break语句用于跳出代码块或循环(for语句可以使用break语句)，continue语句用于终止本次循环，返回循环结构的头部，开始下一轮循环。
```js
for(var i=0;i<5;i++>){
  console.log(i);
  if(i === 3) break;
}
// 0
// 1
// 2
// 3
```