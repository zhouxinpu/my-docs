## `Vue`选项的规范化



### 弄清楚传递给`mergeOptions`函数的三个参数

通过查看引用关系发现`mergeOptions`函数来自于`core/util/options`文件，在深入了解`core/util/options`之前，有必要搞清楚下面代码中具体再做什么：

```js
vm.$options = mergeOptions(
    resolveConstructorOptions(vm.constructor),
    options || {},
    vm
)
```

传递给`mergeOptions`函数的3个参数到底都是什么呢？

第一个参数是通过调用一个函数得到的，将`vm.constructor`作为参数传递给`resolveConstructorOptions`函数，第二个参数就是我们调用`Vue`构造函数透传进来的对象，第三个参数是当前的`Vue`实例。

`resolveConstructorOptions`函数就声明在`core/instance/init.js`文件中，代码如下：

```js
export function resolveConstructorOptions (Ctor: Class<Component>) {
  let options = Ctor.options
  if (Ctor.super) {
    const superOptions = resolveConstructorOptions(Ctor.super)
    const cachedSuperOptions = Ctor.superOptions
    if (superOptions !== cachedSuperOptions) {
      // super option changed,
      // need to resolve new options.
      Ctor.superOptions = superOptions
      // check if there are any late-modified/attached options (#4976)
      const modifiedOptions = resolveModifiedOptions(Ctor)
      // update base extend options
      if (modifiedOptions) {
        extend(Ctor.extendOptions, modifiedOptions)
      }
      options = Ctor.options = mergeOptions(superOptions, Ctor.extendOptions)
      if (options.name) {
        options.components[options.name] = Ctor
      }
    }
  }
  return options
}
```

首先函数的第一句代码：

```js
let options = Ctor.options
```

其中`Ctor`就是传递过来的`vm.constructor`，在本例中就是`Vue`构造函数，所以`Ctor.options`就是`Vue.options`，然后是一大段嵌套在`if`中的代码，跳过`if`语句，直接查看返回值：

```js
return options
```

也就是把`Vue.options`返回出去，所以这段代码的作用和它的名字那样，就是用来获取构造者的`options`的，`resolveConstructorOptions` 函数的第一句和最后一句代码中间还有一坨包裹在 `if` 语句块中的代码，那么这坨代码是干什么的呢？

我可以很明确地告诉大家，这里水稍微有那么点深，比如 `if` 语句的判断条件 `Ctor.super`，`super` 这是子类才有的属性，如下：

```js
const Sub = Vue.extend()
console.log(Sub.super)  // Vue
```

也就是说，`super` 这个属性是与 `Vue.extend` 有关系的，事实也的确如此。除此之外判断分支内的第一句代码：

```js
const superOptions = resolveConstructorOptions(Ctor.super)
```

我们发现，又递归地调用了 `resolveConstructorOptions` 函数，只不过此时的参数是构造者的父类，之后的代码中，还有一些关于父类的 `options` 属性是否被改变过的判断和操作，并且大家注意这句代码：

```js
// check if there are any late-modified/attached options (#4976)
const modifiedOptions = resolveModifiedOptions(Ctor)
```

我们要注意的是注释，有兴趣的同学可以根据注释中括号内的 `issue` 索引去搜一下相关的问题，这句代码是用来解决使用 `vue-hot-reload-api` 或者 `vue-loader` 时产生的一个 `bug` 的。

现在大家知道这里的水有多深了吗？关于这些问题，我们在讲 `Vue.extend` 时都会给大家一一解答，不过有一个因素从来没有变，那就是 `resolveConstructorOptions` 这个函数的作用永远都是用来获取当前实例构造者的 `options` 属性的，即使 `if` 判断分支内也不例外，因为 `if` 分支只不过是处理了 `options`，最终返回的永远都是 `options`。

所以根据我们的例子，`resolveConstructorOptions` 函数目前并不会走 `if` 判断分支，即此时这个函数相当于：

```js
export function resolveConstructorOptions (Ctor: Class<Component>) {
  let options = Ctor.options
  return options
}
```

根据我们的示例，此时`mergeOptions`函数的第一个参数就是`Vue.options`，也就是：

```js
Vue.options = {
	components: {
		KeepAlive
		Transition,
    	TransitionGroup
	},
	directives:{
	    model,
        show
	},
	filters: Object.create(null),
	_base: Vue
}
```

此时，第二个参数`options`，这个参数就是我们调用`Vue`构造函数透传进来的选项，所以根据我们的例子`options`的值如下：

```js
{
  el: '#app',
  data: {
    test: 1
  }
}
```

第三个参数`vm`也就是`Vue`实例对象本身，综上所述，最终代码如下：

```js
vm.$options = mergeOptions(
  resolveConstructorOptions(vm.constructor),
  options || {},
  vm
)
```

相当于：

```js
vm.$options = mergeOptions(
  // resolveConstructorOptions(vm.constructor)
  {
    components: {
      KeepAlive
      Transition,
      TransitionGroup
    },
    directives:{
      model,
      show
    },
    filters: Object.create(null),
    _base: Vue
  },
  // options || {}
  {
    el: '#app',
    data: {
      test: 1
    }
  },
  vm
)
```

现在我们已经搞清楚了传递给`mergeOptions`的三个参数分别是什么了，接下来就看看`mergeOptions`函数具体做了什么。



#### `mergeOptions`函数

打开`core/util/options.js`文件，找到`mergeOptions`方法，这个方法上面有一段注释：

```js
/**
 * Merge two option objects into a new one.
 * Core utility used in both instantiation and inheritance.
 */
```

合并两个选项成一个新的对象，这个函数在实例化和继承的时候都有用到。这里要注意两点：第一，这个函数将会产生一个新的对象；第二，这个函数不仅仅在实例化对象(即`_init`方法中)的时候用到，在继承(`Vue.extend`)中也有用到，所以这个函数应该是一个用来合并两个选项对象为一个新对象的通用程序。

`mergeOptions`函数的第一段代码：

```js
export function mergeOptions (
  parent: Object,
  child: Object,
  vm?: Component
): Object {
  if (process.env.NODE_ENV !== 'production') {
    checkComponents(child)
  }
 // 省略
}
```

在非生产环境下，把`child`作为参数传递给`checkComponents`方法，查看此方法：

```js
/**
 * Validate component names
 */
function checkComponents (options: Object) {
  for (const key in options.components) {
    validateComponentName(key)
  }
}
```

由注释可知，这个方法是用来校验组件的名称是否符合要求。`checkComponents`方法使用`for...in...`循环遍历`options.components`选项，所以`validateComponentName`才是真正校验名字的函数，查看`validateComponentName`函数定义如下：

```js
export function validateComponentName (name: string) {
  if (!new RegExp(`^[a-zA-Z][\\-\\.0-9_${unicodeRegExp.source}]*$`).test(name)) {
    warn(
      'Invalid component name: "' + name + '". Component names ' +
      'should conform to valid custom element name in html5 specification.'
    )
  }
  if (isBuiltInTag(name) || config.isReservedTag(name)) {
    warn(
      'Do not use built-in or reserved HTML elements as component ' +
      'id: ' + name
    )
  }
}
```

`validateComponentName`由两个`if`语句块组成，所以，对于组件的名字要满足两条规则：

1. new RegExp(`^[a-zA-Z][\\-\\.0-9_${unicodeRegExp.source}]*$`)
2. 要满足条件 isBuiltInTag(name) || config.isReservedTag(name)

第一条规则，限定组件的名称由普通的字符开头，[\\-\\.0-9_${unicodeRegExp.source}]* 出现任意次

第二条规则，首先将`options.component`对象的`key`作为参数分别调用两个方法：`isBuiltInTag`和`config.isReservedTag`，其中`isBuiltInTag`方法的作用是用来检测所注册的组件是否是内置的标签，`isBuiltInTag`来自于`src/shared/util.js`，代码实现如下：

```js
export const isBuiltInTag = makeMap('slot,component', true)
```

函数又调用了`makeMap`函数，`makeMap`函数代码实现如下：

```js
export function makeMap (
  str: string,
  expectsLowerCase?: boolean
): (key: string) => true | void {
  const map = Object.create(null)
  // map = {}
  const list: Array<string> = str.split(',')
  // list = ['slot','component']
  map = {
    slot:true,
    component: true
  }
  for (let i = 0; i < list.length; i++) {
    map[list[i]] = true
  }
  // map = {slot: true, component: true}
  return expectsLowerCase
    ? val => map[val.toLowerCase()]
    : val => map[val]
}
```

由此可知，`slot`和`component`这两个名字被`Vue`作为内置标签而存在，是不能够使用的。

除了检测注册的组件名字是否为内置的标签之外，还会通过`config.isReservedTag`方法来进行检测，该方法是在`platforms/web/runtime/index.js`文件中进行赋值：

```js
Vue.config.isReservedTag = isReservedTag
```

其值`isReservedTag`来自于`platforms/web/util/element.js`文件的`isReservedTag`函数，代码如下：

```js
export const isReservedTag = (tag: string): ?boolean => {
  return isHTMLTag(tag) || isSVG(tag)
}
```

函数返回值是调用了`isHTMLTag`或`isSVG`函数，查看这两个函数：

```js
export const isHTMLTag = makeMap(
  'html,body,base,head,link,meta,style,title,' +
  'address,article,aside,footer,header,h1,h2,h3,h4,h5,h6,hgroup,nav,section,' +
  'div,dd,dl,dt,figcaption,figure,picture,hr,img,li,main,ol,p,pre,ul,' +
  'a,b,abbr,bdi,bdo,br,cite,code,data,dfn,em,i,kbd,mark,q,rp,rt,rtc,ruby,' +
  's,samp,small,span,strong,sub,sup,time,u,var,wbr,area,audio,map,track,video,' +
  'embed,object,param,source,canvas,script,noscript,del,ins,' +
  'caption,col,colgroup,table,thead,tbody,td,th,tr,' +
  'button,datalist,fieldset,form,input,label,legend,meter,optgroup,option,' +
  'output,progress,select,textarea,' +
  'details,dialog,menu,menuitem,summary,' +
  'content,element,shadow,template,blockquote,iframe,tfoot'
)
```

```js
// this map is intentionally selective, only covering SVG elements that may
// contain child elements.
export const isSVG = makeMap(
  'svg,animate,circle,clippath,cursor,defs,desc,ellipse,filter,font-face,' +
  'foreignObject,g,glyph,image,line,marker,mask,missing-glyph,path,pattern,' +
  'polygon,polyline,rect,switch,symbol,text,textpath,tspan,use,view',
  true
)
```

由此可知，`options.component`的`key`不能使用`html`和`svg`的内置标签。

接下来的一段代码同样是一个`if`语句块：

```js
  if (typeof child === 'function') {
    child = child.options
  }
```

这说明`child`参数除了可以是普通的选项对象外，还可以是一个函数，如果是函数的话就取函数的`options`静态属性作为新的`child`，我们想一想什么样的函数具有 `options` 静态属性呢？现在我们知道 `Vue` 构造函数本身就拥有这个属性，其实通过 `Vue.extend` 创造出来的子类也是拥有这个属性的。所以这就允许我们在进行选项合并的时候，去合并一个 `Vue` 实例构造者的选项了。



### 规范化props`（normalizeProps，normalizeInject，normalizeDirectives）`

接下来的一段代码：

```js
  normalizeProps(child, vm)
  normalizeInject(child, vm)
  normalizeDirectives(child)
```

这三个函数是用来规范选项的。

先看第一个函数`normalizeProps`：

```js
/**
 * Ensure all props option syntax are normalized into the
 * Object-based format.
 */
function normalizeProps (options: Object, vm: ?Component) {
  const props = options.props
  if (!props) return
  const res = {}
  let i, val, name
  if (Array.isArray(props)) {
    i = props.length
    while (i--) {
      val = props[i]
      if (typeof val === 'string') {
        name = camelize(val)
        res[name] = { type: null }
      } else if (process.env.NODE_ENV !== 'production') {
        warn('props must be strings when using array syntax.')
      }
    }
  } else if (isPlainObject(props)) {
    for (const key in props) {
      val = props[key]
      name = camelize(key)
      res[name] = isPlainObject(val)
        ? val
        : { type: val }
    }
  } else if (process.env.NODE_ENV !== 'production') {
    warn(
      `Invalid value for option "props": expected an Array or an Object, ` +
      `but got ${toRawType(props)}.`,
      vm
    )
  }
  options.props = res
}
```

根据注释可知，`normalizeProps`的作用就是确保所有props配置语法规范化到基于对象的格式。

以props为例，我们在使用props的时候，可以有两种写法，一种是字符串：

```js
const ChildComponent = {
    props: ['propName']
}
```

另外一种是使用对象语法：

```js
const ChildComponent = {
    props: {
        propName: {
            type: propType,
            default: propDefault
        }
    }
}
```

这两种写法给开发者提供了非常灵活且便利的写法，但是对于`Vue`要对选项进行处理，这种情况下就需要在内部将其规范成同一种方式，这样在选项合并的时候就能统一处理，这就是上面3个函数的作用。

```js
const props = options.props
```

接下来定义了一个变量`res`用来接收规范化的props:

```js
const res = {}
```

把传递进来的`options.props`赋值给一个变量，

```js
if (Array.isArray(props)) {
    i = props.length
    while (i--) {
        val = props[i]
        if (typeof val === 'string') {
            name = camelize(val)
            res[name] = { type: null }
        } else if (process.env.NODE_ENV !== 'production') {
            warn('props must be strings when using array syntax.')
        }
    }
}
```

判断props如果是一个数组，遍历props，判断每一项如果不是字符串，则给个友好的提示，如果是字符串，如果是连字符，就变为驼峰作为key，值为{type: null}，比如：

```js
props: ['name']
```

经过上述代码处理之后就会变成：

```js
props:[
	name: {
    	type: null
    }
]
```

接下来的一段代码判断props如果是一个对象，遍历对象每一项，对象的key如果是连字符就变驼峰表示，value的值如果是对象则直接返回，否则返回{type: props[key]}：

```js
else if (isPlainObject(props)) {
    for (const key in props) {
        val = props[key]
        name = camelize(key)
        res[name] = isPlainObject(val)
            ? val
        : { type: val }
    }
}
```

如果你的props是对象如下：

```js
props:{
    someData1: Number,
 	someData2: {
        type: string,
        default: ''
    }
}
```

经过上述代码处理之后：

```js
props:{
    someData1:{
        type: Number
    },
    someData2:{
        type: String,
        default: ''
    }
}
```

代码最后还有一个判断分支，即当你传递了props，但其值既不是对象也不是字符串数组的时候，就会给你一个友好的提示：

```js
if (process.env.NODE_ENV !== 'production') {
    warn(
        `Invalid value for option "props": expected an Array or an Object, ` +
        `but got ${toRawType(props)}.`,
        vm
    )
}
```

代码的最后使用`res`覆盖了`options.props`。



在看第二个函数`normalizeInject`：

```js
function normalizeInject (options: Object, vm: ?Component) {
  const inject = options.inject
  if (!inject) return
  const normalized = options.inject = {}
  if (Array.isArray(inject)) {
    for (let i = 0; i < inject.length; i++) {
      normalized[inject[i]] = { from: inject[i] }
    }
  } else if (isPlainObject(inject)) {
    for (const key in inject) {
      const val = inject[key]
      normalized[key] = isPlainObject(val)
        ? extend({ from: key }, val)
        : { from: val }
    }
  } else if (process.env.NODE_ENV !== 'production') {
    warn(
      `Invalid value for option "inject": expected an Array or an Object, ` +
      `but got ${toRawType(inject)}.`,
      vm
    )
  }
}
```

首先是这三句代码：

```js
  const inject = options.inject
  if (!inject) return
  const normalized = options.inject = {}
```

第一句话用变量`inject`缓存了`options.inject`，第二句判断了有没有传递`inject`，没有的话就直接返回，第三句重写了`options.inject`为空对象，然后定义了变量`normalized`为空对象，现在变量`normalized`和`options.inject`引用了同一个对象，也就是修改`normalized`，`options.inject`也会受到影响。



之后同样是判断分支语句，判断`inject`选项是数组或者是纯对象，`inject`选项是`v2.2.0`版本之后新增的，它需要配合`provide`选项一起使用，比如：

```js
// 子组件
const ChildComponent = {
  template: '<div>child component</div>',
  created: function () {
    // 这里的 data 是父组件注入进来的
    console.log(this.data)
  },
  inject: ['data']
}

// 父组件
var vm = new Vue({
  el: '#app',
  // 向子组件提供数据
  provide: {
    data: 'test provide'
  },
  components: {
    ChildComponent
  }
})
```

上面的代码中，在子组件的`created`生命周期里打印`this.data`，但是我们没有定义这个数据，之所以可以使用是因为我们使用了`inject`选项注入了这个数据，数据来源于父组件通过`provide`提供，父组件通过`provide`选项向子组件提供数据，子组件使用`inject`注入数据，`inject`不仅可以上例的字符串数据，也可以写成对象形式，比如：

```js
// 子组件
const ChildComponent = {
  template: '<div>child component</div>',
  created: function () {
    // 这里的 data 是父组件注入进来的
    console.log(this.d)
  },
  // 对象的语法类似于给数据声明了一个别名
  inject: {d: 'data'}
}
```

所以回头再去看`normalizeInject`函数所做的无非就是把两种格式的代码给规范化成一种写法罢了。

接下来的代码：

```js
if (Array.isArray(inject)) {
    for (let i = 0; i < inject.length; i++) {
        normalized[inject[i]] = { from: inject[i] }
    }
}
```

判断`inject`是不是数组，如果是数组，遍历`inject`，如果你的`inject`选项是这样写的：

```js
inject: ['data']
```

经过上面的代码规范化就会变成这样：

```js
inject:{
    data:{from: data}
}
```

当`inject`不是数组，是一个纯对象的时候，就走`else if`分支：

```js
else if (isPlainObject(inject)) {
    for (const key in inject) {
        const val = inject[key]
        normalized[key] = isPlainObject(val)
            ? extend({ from: key }, val)
        : { from: val }
    }
}
```

遍历`inject`，如果`inject`的`val`是一个纯对象，则使用`extend`来进行混合，否则就直接使用`val`作为字段`from`字段的值。

比如：

```js
inject: {
  data1,
  'd2': 'data2',
  'data3': { someProperty: 'someValue' }
}
```

经过上述函数转换之后就会变成这样：

```js
inject: {
  'data1': { from: 'data1' },
  'd2': { from: 'data2' },
  'data3': { from: 'data3', someProperty: 'someValue' }
}
```



代码最后的`else if`判断分支，同样是用来判断如果传递了`inject`，但其值既不是对象也不是字符串数组的时候，非生产环境下就会给你一个友好的提示：

```js
else if (process.env.NODE_ENV !== 'production') {
    warn(
        `Invalid value for option "inject": expected an Array or an Object, ` +
        `but got ${toRawType(inject)}.`,
        vm
    )
}
```

第三个函数`normalizeDirectives`：

```js
/**
 * Normalize raw function directives into object format.
 */
function normalizeDirectives (options: Object) {
  const dirs = options.directives
  if (dirs) {
    for (const key in dirs) {
      const def = dirs[key]
      if (typeof def === 'function') {
        dirs[key] = { bind: def, update: def }
      }
    }
  }
}
```

由注释可知，这个函数的作用是把一个函数类型的指令转换为对象格式。遍历所有的指令，当发现注册的指令是函数的时候，则将该函数作为对象形式的`bind`属性和`update`属性的值，也就是可以把函数语法注册指令的方式理解为一种简写。



比如我们注册了两个局部组件分别是`v-test1`和`v-test2`：

```js
<div id="app" v-test1 v-test2>{{test}}</div>

var vm = new Vue({
    el: '#app',
    data: {
        test: 1
    },
    // 注册两个局部指令
    directives: {
        test1: {
            bind: function () {
                console.log('v-test1')
            }
        },
        test2: function () {
            console.log('v-test2')
        }
    }
})
```

上面的代码中我们注册了两个局部组件，但是注册的方法不同，`test1`使用了对象语法，`test2`使用了函数语法，经过上面规范化处理之后就会变成这样：

```js
<div id="app" v-test1 v-test2>{{test}}</div>

var vm = new Vue({
    el: '#app',
    data: {
        test: 1
    },
    // 注册两个局部指令
    directives: {
        test1: {
            bind: function () {
                console.log('v-test1')
            }
        },
        test2: { 
            bind: function () {
            	console.log('v-test2')
        	},
            update: function () {
            	console.log('v-test2')
        	}
        }
    }
})
```



通过上面的3个规范化函数处理之后，`props`，`inject`，`directive`就会被处理成统一的格式了。

看完了`mergeOptions`函数的三个规范化函数之后，看下一段代码：

```js
// Apply extends and mixins on the child options,
// but only if it is a raw options object that isn't
// the result of another mergeOptions call.
// Only merged options has the _base property.
if (!child._base) {
    if (child.extends) {
        parent = mergeOptions(parent, child.extends, vm)
    }
    if (child.mixins) {
        for (let i = 0, l = child.mixins.length; i < l; i++) {
            parent = mergeOptions(parent, child.mixins[i], vm)
        }
    }
}
```

这段代码先判断`child.extends`如果为真，则递归调用`mergeOptions`函数将`parent`和`child.extends`进行合并，然后判断`child.mixins`如果为真，则`for`循环`child.mixins`，然后递归调用将`parent`和`children.mixins[i]`进行合并。并将结果作为新的`parent`。要注意的是，之前说过`mergeOptions`函数将产生一个新的对象，所以此时的`parent`已经被新的对象重新赋值了。

经过上面的两个判断分支，此时的`parent`很可能已经不是之前的`parent`，而是经过合并后的新对象。

至此看的`mergeOptions`的所有代码都是对选项的规范化，或者说，现在所做的事还是对`parent`已经`child`进行预处理，而这是接下来合并选项的必要步骤。

