## Vue选项的合并

上面的代码都是对`Vue`选项的规范化，接下来的代码才是真正的合并阶段。继续看剩下的一段代码：

```js
  const options = {}
  let key
  for (key in parent) {
    mergeField(key)
  }
  for (key in child) {
    if (!hasOwn(parent, key)) {
      mergeField(key)
    }
  }
  function mergeField (key) {
    const strat = strats[key] || defaultStrat
    options[key] = strat(parent[key], child[key], vm, key)
  }
  return options
```

这段代码的第一句和最后一句说明了`mergeOptions`函数返回了一个新的对象，第一句代码定义了一个`options`常量，最后一句代码则把`options`返回，可以预估是中间的一段代码就是在充实`options`，`options`常量就应该是最终合并之后的选项。

明确一下代码结构，有两个`for...in`循环以及一个名字叫`mergeField`函数，先看第一段`for...in`代码：

```js
for (key in parent) {
    mergeField(key)
}
```

遍历`parent`，然后把`parent`的`key`作为参数传递给`mergeField`函数，假如`parent`就是`Vue.options`：

```js
Vue.options = {
  components: {
      KeepAlive,
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

那么`key`就是`components`，`directives`，`filters`，`_base`，除了`_base`其他的字段都可以理解为`Vue`提供的选项的名字。

第二段`for...in`循环：

```js
  for (key in child) {
    if (!hasOwn(parent, key)) {
      mergeField(key)
    }
  }
```

其遍历的就是`child`对象，并且多了一个判断：

```js
if (!hasOwn(parent, key))
```

`hasOwn`来自于`platforms/shared/util.js`，用来判断一个属性是否是对象自身的属性（不包括原型上的），源代码如下：

```js
/**
 * Check whether an object has the property.
 */
const hasOwnProperty = Object.prototype.hasOwnProperty
export function hasOwn (obj: Object | Array<*>, key: string): boolean {
  return hasOwnProperty.call(obj, key)
}
```

所以上句代码的意思就是，遍历`child`，如果`key`不在`parent`身上出现，则会调用`mergeField`，否则不会调用，因为上面已经遍历过了，所以这里避免了重复调用。

总之这两个`for`循环就是使用在`parent`或者`child`对象中出现的`key(即选项的名字)`作为参数调用`mergeField`，所以真正合并的操作是在`mergeField`函数中。

`mergeField`函数代码如下：

```js
  function mergeField (key) {
    const strat = strats[key] || defaultStrat
    options[key] = strat(parent[key], child[key], vm, key)
  }
```

`mergeField`只有两句代码，第一句定义了一个`strat`变量，变量的值通过`key`访问`starts`对象得来的，当访问的对象不存在，则使用`defaultStrat`作为值。

`strat`是什么呢？在`src/core/util/options.js`文件的顶部有一句代码：

```js
/**
 * Option overwriting strategies are functions that handle
 * how to merge a parent option value and a child option
 * value into the final value.
 */
const strats = config.optionMergeStrategies
```

这句代码定义了`strats`变量，且是一个常量，`config`对象是全局配置对象，来自于`src/core/config.js`文件，此时`config.optionMergeStrategied`还是一个空对象。上面注释的大概意思是：选项覆盖策略函数用来处理如何合并父选项值和子选项值合并到最终值的函数。也就是说 `config.optionMergeStrategies` 是一个合并选项的策略对象，这个对象下包含很多函数，这些函数就可以认为是合并特定选项的策略。这样不同的选项使用不同的合并策略，如果你使用自定义选项，那么你也可以自定义该选项的合并策略，只需要在 `Vue.config.optionMergeStrategies` 对象上添加与自定义选项同名的函数就行。而这就是 `Vue` 文档中提过的全局配置：[optionMergeStrategies](https://vuejs.org/v2/api/#optionMergeStrategies)



### 选项el，propsData的合并策略

接下来查看一下这个选项合并策略对象都有哪些策略，接下来的代码：

```js
/**
 * Options with restrictions
 */
if (process.env.NODE_ENV !== 'production') {
  strats.el = strats.propsData = function (parent, child, vm, key) {
    if (!vm) {
      warn(
        `option "${key}" can only be used during instance ` +
        'creation with the `new` keyword.'
      )
    }
    return defaultStrat(parent, child)
  }
}
```

非生成环境下在`strats`策略对象上添加了两个策略`el`和`propsData`，且这两个属性的值是一个函数，通过这两个属性的名字可知，这两个策略是用来处理`el`选项和`propsData`选项的。其实本质上没做什么合并工作。

首先是一段`if`分支判断语句，判断是否有传递`vm`参数：

```js
if (!vm) {
    warn(
        `option "${key}" can only be used during instance ` +
        'creation with the `new` keyword.'
    )
}
```

判断如果没有传递了`vm`参数，就会给你一个提示。`el`选项或者`propsData`选项只能在使用`new`操作符创建实例的时候可用，比如下面的代码：

```js
// 子组件
var ChildComponent = {
  el: '#app2',
  created: function () {
    console.log('child component created')
  }
}

// 父组件
new Vue({
  el: '#app',
  data: {
    test: 1
  },
  components: {
    ChildComponent
  }
})
```

上面的代码中在父组件中使用`el`选项，这没有什么问题，但是在子组件中也使用了`el`选项，就会得到上面的警告信息，这说明了，在策略函数中如果拿不到`vm`参数，那说明处理的就是子组件选项，那么为什么可以通过`vm`来判断是不是子组件呢？先看一下`vm`参数从哪里来的，看一下`mergeField`函数：

```js
function mergeField (key) {
    const strat = strats[key] || defaultStrat
    options[key] = strat(parent[key], child[key], vm, key)
}
```

函数体的第二句代码在调用策略函数的时候，第三个参数`vm`就是我们在策略函数中使用的那个`vm`，`vm`是从`mergeOptions`函数透传过来的，因为`mergeOptions`第三个参数就是`vm`，我们知道在`_init`方法中调用`mergeOptions`函数时候第三个参数就是当前`Vue`实例：

```js
// _init 方法中调用 mergeOptions 函数，第三个参数是 Vue 实例
vm.$options = mergeOptions(
  resolveConstructorOptions(vm.constructor),
  options || {},
  vm
)
```

所以我们可以理解为：策略函数中的`vm`来源于`mergeOptions`函数的第三个参数。所以当我们调用`mergeOptions`函数且不传递第三个参数的时候，那么在策略函数中就拿不到`vm`参数。所以我们可以猜测一件事，`mergeOptions`函数除了在`_init`方法中被调用了之外，还在其他地方调用了，且没有传递第三个参数。那么到底是在哪里被调用的呢？这里可以先明确地告诉大家，就是在 `Vue.extend` 方法中被调用的，大家可以打开 `core/global-api/extend.js` 文件找到 `Vue.extend` 方法，其中有这么一段代码：

```js
Sub.options = mergeOptions(
  Super.options,
  extendOptions
)
```

可以发现，此时调用`mergeOptions`函数没有传递第三个参数，也就是说通过`Vue.extend`创建子类的时候`mergeOptions`会被调用，此时策略函数就拿不到第三个参数。

所以现在就很明朗，在策略函数中通过判断是否存在`vm`参数就能够得知`mergeOptions`是在实例化时调用（使用`new`操作符走`_init`方法），还是在继承时调用（`Vue.extend`），而子组件的实现方式就是通过实例化子类完成的，子类又是通过`Vue.extend`创造出来的，所以我们就能够通过对`vm`的判断而得知是否是子组件。

所以最终结论就是：如果策略函数中拿不到`vm`参数，那么处理的就是子组件的选项。

接着看`strats.el`和`strats.propsData`策略函数的代码，在`if`判断分支下面，直接调用`defaultStrat`函数并返回：

```js
return defaultStrat(parent, child)
```

`defaultStrat`函数定义源码如下：

```js
/**
 * Default strategy.
 */
const defaultStrat = function (parentVal: any, childVal: any): any {
  return childVal === undefined
    ? parentVal
    : childVal
}
```

`defaultStrat`函数就和他名字一样，是默认策略，逻辑很简单：只要子选项不是`undefined`那么就使用子选项，否则使用父选项。

`strats.el` 和 `strats.propsData` 这两个策略函数是只有在非生产环境才有的，在生产环境下访问这两个函数将会得到 `undefined`，那这个时候 `mergeField` 函数的第一句代码就起作用了：

```js
// 当一个选项没有对应的策略函数时，使用默认策略
const strat = strats[key] || defaultStrat
```

所以在生产环境下就会直接使用默认的策略函数`defaultStrat`来处理`el`和`propsData`这两个选项。



### 选项data的合并策略

跳过`mergeData`以及`mergeDataOrFn`，继续看下面的代码，如下：

```js
strats.data = function (
  parentVal: any,
  childVal: any,
  vm?: Component
): ?Function {
  if (!vm) {
    if (childVal && typeof childVal !== 'function') {
      process.env.NODE_ENV !== 'production' && warn(
        'The "data" option should be a function ' +
        'that returns a per-instance value in component ' +
        'definitions.',
        vm
      )

      return parentVal
    }
    return mergeDataOrFn(parentVal, childVal)
  }

  return mergeDataOrFn(parentVal, childVal, vm)
}
```

这段代码的作用就是在`strats`策略对象上添加`data`策略函数，用来处理`data`选项。首先是一个`if`判断分支：

```js
if (!vm) {
    // 省略
}
```

与`el`和`propsData`处理策略相同，先判断是否传递了`vm`参数，我们知道如果没有传递`vm`参数，说明处理的是子组件的选项。先看如何处理子组件的选项的，`if`判断语句块内的代码如下：

```js
if (childVal && typeof childVal !== 'function') {
    process.env.NODE_ENV !== 'production' && warn(
        'The "data" option should be a function ' +
        'that returns a per-instance value in component ' +
        'definitions.',
        vm
    )

    return parentVal
}
return mergeDataOrFn(parentVal, childVal)
```

首先判断是否传递了子组件的`data`选项（`childVal`），并且检测`childVal`的类型是不是一个`function`，如果`childVal`的类型不是一个`function`类型，则会给一个警告，也就是说`childVal`应该是一个函数。这就是我们知道的：子组件中的`data`必须是一个返回对象的函数。如果不是函数，则除了会给你一段警告之外，会直接返回`parentVal`。

如果`childVal`是函数类型，那说明满足子组件的`data`选项需要是一个函数的要求，那么就直接返回`mergeDataOrFn`函数的执行结果：

```js
return mergeDataOrFn(parentVal, childVal)
```

上面的情况是在`strats.data`函数策略拿不到`vm`参数时的情况，如果拿到了`vm`参数，那说明处理的就不是子组件的选项，而是正常使用`new`操作符创建实例时的选项，这个时候则直接返回`mergeDataOrFn`的函数的执行结果，但是会多透传一个参数`vm`，

```js
return mergeDataOrFn(parentVal, childVal, vm)
```

通过上面的分析可知一件事，即`strats.data`函数策略无论合并处理的是子组件的选项还是非子组件的选项，其最终都是调用`mergeDataOrFn`函数进行处理的，并且以`mergeDataOrFn`函数的返回值作为策略函数的最终返回值。但有一点不同的是处理非子组件选项的时候所调用的`mergeDataOrFn`函数多传递了一个`vm`参数，所以让我们看一下`mergeDataOrFn`的代码，看看它的返回值是什么，因为它的返回值就等价于`strats.data`策略函数的返回值。`mergeDataOrFn`函数的源码如下：

```js
/**
 * Data
 */
export function mergeDataOrFn (
  parentVal: any,
  childVal: any,
  vm?: Component
): ?Function {
  if (!vm) {
    // in a Vue.extend merge, both should be functions
    if (!childVal) {
      return parentVal
    }
    if (!parentVal) {
      return childVal
    }
    // when parentVal & childVal are both present,
    // we need to return a function that returns the
    // merged result of both functions... no need to
    // check if parentVal is a function here because
    // it has to be a function to pass previous merges.
    return function mergedDataFn () {
      return mergeData(
        typeof childVal === 'function' ? childVal.call(this, this) : childVal,
        typeof parentVal === 'function' ? parentVal.call(this, this) : parentVal
      )
    }
  } else {
    return function mergedInstanceDataFn () {
      // instance merge
      const instanceData = typeof childVal === 'function'
        ? childVal.call(vm, vm)
        : childVal
      const defaultData = typeof parentVal === 'function'
        ? parentVal.call(vm, vm)
        : parentVal
      if (instanceData) {
        return mergeData(instanceData, defaultData)
      } else {
        return defaultData
      }
    }
  }
}
```



