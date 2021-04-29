## 以一个例子为线索

我们有如下模板：

```js
<div id="app">{{test}}</div>
```

和这样一段`js`代码：

```js
var vm = new Vue({
    el: '#app',
    data: {
        test: 1
    }
})
```

这段 `js` 代码很简单，只是简单地调用了 `Vue`，传递了两个选项 `el` 以及 `data`。这段代码的最终效果就是在页面中渲染为如下 `DOM`：

```html
<div id="app">1</div>
```

其中 `{{ test }}` 被替换成了 `1`，并且当我们尝试修改 `data.test` 的值的时候

```js
vm.$data.test = 2
// 或
vm.test = 2
```

那么页面的 `DOM` 也会随之变化为：

```html
<div id="app">2</div>
```

当我们执行`new Vue`的时候，第一句执行的代码是`core/instance/index.js`：

```js
function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  this._init(options)
}
```

一旦开始使用`new Vue`构造函数的，第一句执行的代码就是`this._init(options)`方法，其中`options`就是我们调用`Vue`构造函数的时候透传过来的，也就是说：

```js
options = {
    el: '#app',
    data: {
        test: 1
    }
}
```

打开`core/instance/init.js`里的`initMixin`方法，方法的一开始：

```js
Vue.prototype._init = function (options?: Object) {
    const vm: Component = this
    // a uid
    vm._uid = uid++
	// 省略
}
```

定义`vm`常量，将`this`也就是`Vue`实例赋值给`vm`，在`vm`上定义了一个`_uid`属性，初始化值为0，可以看到的是每次初始化一个`Vue`实例的话，`vm._uid`都会`++`。

接下来的代码：

```js
let startTag, endTag
/* istanbul ignore if */
if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
    startTag = `vue-perf-start:${vm._uid}`
    endTag = `vue-perf-end:${vm._uid}`
    mark(startTag)
}

// 中间的代码省略...

/* istanbul ignore if */
if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
    vm._name = formatComponentName(vm, false)
    mark(endTag)
    measure(`vue ${vm._name} init`, startTag, endTag)
}
```

首先声明两个变量 `startTag` 和 `endTag`，然后这两段代码有一个共同点，即拥有相同的判断语句：

```js
if (process.env.NODE_ENV !== 'production' && config.performance && mark)
```

意思是：在非生产环境下，并且 `config.performance` 和 `mark` 都为真，那么才执行里面的代码，其中 `config.performance` 来自于 `core/config.js` 文件，我们知道，`Vue.config` 同样引用了这个对象，在 `Vue` 的官方文档中可以看到如下内容：

`Vue` 提供了全局配置 `Vue.config.performance`，我们通过将其设置为 `true`，即可开启性能追踪，你可以追踪四个场景的性能：

- 1、组件初始化(`component init`)
- 2、编译(`compile`)，将模板(`template`)编译成渲染函数
- 3、渲染(`render`)，其实就是渲染函数的性能，或者说渲染函数执行且生成虚拟DOM(`vnode`)的性能
- 4、打补丁(`patch`)，将虚拟DOM渲染为真实DOM的性能

其中*组件初始化*的性能追踪就是我们在 `_init` 方法中看到的那样去实现的，其实现的方式就是在初始化的代码的开头和结尾分别使用 `mark` 函数打上两个标记，然后通过 `measure` 函数对这两个标记点进行性能计算。`mark`和`measure`都是从`core/util/perf.js`文件导出的，查看代码可知，只有在非生产环境下，且浏览器必须支持`window.performance`API的情况下才会导出有用的`remark`和`measure`，否则导出的就是`undefined`， 也就不会执行`if`语句里的代码了。

两段代码中间省略部分的代码，也就是被追踪的代码：

```js
// a flag to avoid this being observed
vm._isVue = true
// merge options
if (options && options._isComponent) {
    // optimize internal component instantiation
    // since dynamic options merging is pretty slow, and none of the
    // internal component options needs special treatment.
    initInternalComponent(vm, options)
} else {
    vm.$options = mergeOptions(
    resolveConstructorOptions(vm.constructor),
    options || {},
    vm
    )
}
/* istanbul ignore else */
if (process.env.NODE_ENV !== 'production') {
    initProxy(vm)
} else {
    vm._renderProxy = vm
}
// expose real self
vm._self = vm
initLifecycle(vm)
initEvents(vm)
initRender(vm)
callHook(vm, 'beforeCreate')
initInjections(vm) // resolve injections before data/props
initState(vm)
initProvide(vm) // resolve provide after data/props
callHook(vm, 'created')
```

首先在`Vue`实例上添加了`isVue`属性，并设置值为`true`，目的是用来标识一个对象是`Vue`实例，即如果发现一个对象拥有`isVue`属性并且值为`true`，就代表该对象是`Vue`实例。这样就可以避免该对象被响应系统给监测到。

再往下是这一段代码：

```js
// merge options
if (options && options._isComponent) {
    // optimize internal component instantiation
    // since dynamic options merging is pretty slow, and none of the
    // internal component options needs special treatment.
    initInternalComponent(vm, options)
} else {
    vm.$options = mergeOptions(
        resolveConstructorOptions(vm.constructor),
        options || {},
        vm
    )
}
```

上面的代码是一段`if`分支语句，条件是`options && options._isComponent`，其实`options`就是我们调用`Vue`时候传递的参数选项，`options._isComponent`是一个内部选项，这个内部选项是在`Vue`创建组件的时候才会有的。

根据本节的例子，代码肯定会走`else`分支，也就是：

```js
vm.$options = mergeOptions(
    resolveConstructorOptions(vm.constructor),
    options || {},
    vm
)
```

这段代码在`Vue`实例上添加了`$options`属性，查看`$options`属性的作用，这个属性用于当前`Vue`的初始化。`vm.$options`的属性在下面的`init*`方法中都使用到了，代码如下：

```js
initLifecycle(vm)
initEvents(vm)
initRender(vm)
callHook(vm, 'beforeCreate')
initInjections(vm) // resolve injections before data/props
initState(vm)
initProvide(vm) // resolve provide after data/props
callHook(vm, 'created')
```

这些方法都是真正起作用的一些初始化方法，在这些初始化方法中，都使用到了`$options`，这个属性的生成都依赖于`mergeOptions`这个函数的作用，接下来就看`mergeOptions`做了什么。又有什么意义。