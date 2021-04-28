## vue源码



### 源码目录

先打开文件夹 看一下文件目录：

```js
├── scripts ------------------------------- 构建相关的文件，一般情况下我们不需要动
│   ├── git-hooks ------------------------- 存放git钩子的目录
│   ├── alias.js -------------------------- 别名配置
│   ├── config.js ------------------------- 生成rollup配置的文件
│   ├── build.js -------------------------- 对 config.js 中所有的rollup配置进行构建
│   ├── ci.sh ----------------------------- 持续集成运行的脚本
│   ├── release.sh ------------------------ 用于自动发布新版本的脚本
├── dist ---------------------------------- 构建后文件的输出目录
├── examples ------------------------------ 存放一些使用Vue开发的应用案例
├── flow ---------------------------------- 类型声明，使用开源项目 [Flow](https://flowtype.org/)
├── packages ------------------------------ 存放独立发布的包的目录
├── test ---------------------------------- 包含所有测试文件
├── src ----------------------------------- 这个是我们最应该关注的目录，包含了源码
│   ├── compiler -------------------------- 编译器代码的存放目录，将 template 编译为 render 函数
│   ├── core ------------------------------ 存放通用的，与平台无关的代码
│   │   ├── observer ---------------------- 响应系统，包含数据观测的核心代码
│   │   ├── vdom -------------------------- 包含虚拟DOM创建(creation)和打补丁(patching)的代码
│   │   ├── instance ---------------------- 包含Vue构造函数设计相关的代码
│   │   ├── global-api -------------------- 包含给Vue构造函数挂载全局方法(静态方法)或属性的代码
│   │   ├── components -------------------- 包含抽象出来的通用组件
│   ├── server ---------------------------- 包含服务端渲染(server-side rendering)的相关代码
│   ├── platforms ------------------------- 包含平台特有的相关代码，不同平台的不同构建的入口文件也在这里
│   │   ├── web --------------------------- web平台
│   │   │   ├── entry-runtime.js ---------- 运行时构建的入口，不包含模板(template)到render函数的编译器，所以不支持 `template` 选项，我们使用vue默认导出的就是这个运行时的版本。大家使用的时候要注意
│   │   │   ├── entry-runtime-with-compiler.js -- 独立构建版本的入口，它在 entry-runtime 的基础上添加了模板(template)到render函数的编译器
│   │   │   ├── entry-compiler.js --------- vue-template-compiler 包的入口文件
│   │   │   ├── entry-server-renderer.js -- vue-server-renderer 包的入口文件
│   │   │   ├── entry-server-basic-renderer.js -- 输出 packages/vue-server-renderer/basic.js 文件
│   │   ├── weex -------------------------- 混合应用
│   ├── sfc ------------------------------- 包含单文件组件(.vue文件)的解析逻辑，用于vue-template-compiler包
│   ├── shared ---------------------------- 包含整个代码库通用的代码
├── package.json -------------------------- 不解释
├── yarn.lock ----------------------------- yarn 锁定文件
├── .editorconfig ------------------------- 针对编辑器的编码风格配置文件
├── .flowconfig --------------------------- flow 的配置文件
├── .babelrc ------------------------------ babel 配置文件
├── .eslintrc ----------------------------- eslint 配置文件
├── .eslintignore ------------------------- eslint 忽略配置
├── .gitignore ---------------------------- git 忽略配置
```



### package.json

去掉了测试，weex相关的脚本配置

```js
"scripts": {
	  // 构建完整版 umd 模块的 Vue
    "dev": "rollup -w -c scripts/config.js --environment TARGET:web-full-dev",
    // 构建运行时 cjs 模块的 Vue
    "dev:cjs": "rollup -w -c scripts/config.js --environment TARGET:web-runtime-cjs",
    // 构建运行时 es 模块的 Vue
    "dev:esm": "rollup -w -c scripts/config.js --environment TARGET:web-runtime-esm",
    // 构建 web-server-renderer 包
    "dev:ssr": "rollup -w -c scripts/config.js --environment TARGET:web-server-renderer",
    // 构建 Compiler 包
    "dev:compiler": "rollup -w -c scripts/config.js --environment TARGET:web-compiler ",
    "build": "node scripts/build.js",
    "build:ssr": "npm run build -- vue.runtime.common.js,vue-server-renderer",
    "lint": "eslint src build test",
    "flow": "flow check",
    "release": "bash scripts/release.sh",
    "release:note": "node scripts/gen-release-note.js",
    "commit": "git-cz"
  },
```

*源码分析的文章，大多数时候是基于 dev 脚本的（即：`npm run dev`），也就是完整版的 umd 模块的 Vue*。原因是方便我们直接引用并使用，且完整版带了 `Compiler` 我们就不用单独去分析了。



### vue的构造函数

我们在使用`Vue`的时候都需要用到`new`操作符进行调用，这说明`Vue`是一个构造函数，所以我们要做的第一件事就是：把`Vue`构造函数搞清楚。

在`package.json`文件中，`dev`对应的命令是：

```js
"dev": "rollup -w -c scripts/config.js --environment TARGET:web-full-dev",
```

当我们在执行`npm run dev`的时候，回去`scripts/config.js`文件中根据`TARGET:web-full-dev`找对应的配置：

```js
// Runtime+compiler development build (Browser)
  'web-full-dev': {
    entry: resolve('web/entry-runtime-with-compiler.js'),
    dest: resolve('dist/vue.js'),
    format: 'umd',
    env: 'development',
    alias: { he: './entity-decoder' },
    banner
  },
```

从上面配置可知，入口是`web/entry-runtime-with-compiler.js`,打包的出口是`dist/vue.js`。`web`是别名配置，在`script/alias.js`中进行配置：

```js
module.exports = {
  vue: resolve('src/platforms/web/entry-runtime-with-compiler'),
  compiler: resolve('src/compiler'),
  core: resolve('src/core'),
  shared: resolve('src/shared'),
  web: resolve('src/platforms/web'),
  weex: resolve('src/platforms/weex'),
  server: resolve('src/server'),
  sfc: resolve('src/sfc')
}
```

所以`Vue`真正的入口文件是`src/platforms/web/entry-runtime-with-compiler.js`。



直接打开`src/platforms/web/entry-runtime-with-compiler.js`，可以看到这样一句代码：

```js
import Vue from './runtime/index'
```

说明这里不是`Vue`构造函数的出生地，可以看到`Vue`是从`src/platforms/web/runtime/index`文件导入的。

打开`src/platforms/web/runtime/index`，也可以看到这样一句代码：

```js
import Vue from 'core/index'
```

说明这里也不是`Vue`构造函数的出生地，可以看到`Vue`是从`src/core/index`文件导入的。

打开`src/core/index`文件，也可以看到这样一句代码：

```js
import Vue from './instance/index'
```

说明这里也不是`Vue`构造函数的出生地，`Vue`是从`core/instance/index`文件导入的。

继续打开`./instance/index`文件：

```js
// 导入了5个函数
import { initMixin } from './init'
import { stateMixin } from './state'
import { renderMixin } from './render'
import { eventsMixin } from './events'
import { lifecycleMixin } from './lifecycle'
import { warn } from '../util/index'

// 定义了Vue的构造函数
function Vue (options) {
  // 如果不是生产环境且不是通过new实例的 就给一个提示
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  this._init(options)
}
// 将Vue作为参数传入下面的方法中
initMixin(Vue)
stateMixin(Vue)
eventsMixin(Vue)
lifecycleMixin(Vue)
renderMixin(Vue)

export default Vue
```

可以发现这里才是`Vue`构造函数的出生地。分别从`core/instance/index`导入`initMixin`，从`core/instance/state`导入`stateMixin`，从`core/instance/render`导入`renderMixin`，从`core/instance/events`导入`eventsMixin`，从`core/instance/lifecycle`导入`lifecycleMixin`，然后将`Vue`构造函数作为参数传入这5个函数中，最后再导出`Vue`。



先看`initMixin`这个方法：

```js
export function initMixin (Vue: Class<Component>) {
  Vue.prototype._init = function (options?: Object) {
      // 省略具体逻辑
  }
}
```

`initMixin`这个方法就是在`Vue`的原型上添加了一个`_init`内部初始化的方法，在`core/instance/index.js`文件中`Vue`的构造函数中就见到过这个方法：

```js
// 定义 Vue 构造函数
function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  // 在这里调用了 _init 方法
  this._init(options)
}
```

在`Vue`的构造函数中有这么一句`this._init(options)`,说明在我们执行`new Vue`的时候，`this._init(options)`将会被执行。



打开`core/instance/state.js`,找到`stateMixin`方法：

```js
export function stateMixin (Vue: Class<Component>) {
  // flow somehow has problems with directly declared definition object
  // when using Object.defineProperty, so we have to procedurally build up
  // the object here.
  const dataDef = {}
  dataDef.get = function () { return this._data }
  const propsDef = {}
  propsDef.get = function () { return this._props }
  if (process.env.NODE_ENV !== 'production') {
    dataDef.set = function () {
      warn(
        'Avoid replacing instance root $data. ' +
        'Use nested data properties instead.',
        this
      )
    }
    propsDef.set = function () {
      warn(`$props is readonly.`, this)
    }
  }
  Object.defineProperty(Vue.prototype, '$data', dataDef)
  Object.defineProperty(Vue.prototype, '$props', propsDef)

  Vue.prototype.$set = set
  Vue.prototype.$delete = del
	// 省略以下逻辑
}
```

` Object.defineProperty(Vue.prototype, '$data', dataDef)
  Object.defineProperty(Vue.prototype, '$props', propsDef)`这两句可以看出在`Vue.prototype`上定义了两个属性，分别是`$date`和`$props`，这两个属性的定义分别写在`dataDef`和`propsDef`中。

```js
const dataDef = {}
dataDef.get = function () { return this._data }
const propsDef = {}
propsDef.get = function () { return this._props }
if (process.env.NODE_ENV !== 'production') {
    dataDef.set = function () {
        warn(
            'Avoid replacing instance root $data. ' +
            'Use nested data properties instead.',
            this
        )
    }
    propsDef.set = function () {
        warn(`$props is readonly.`, this)
    }
}
```

可以看出`$date`代理的是`this._data`，`$props`代理的是`this._props`，然后就是一个判断，如果不是生产环境的话，如果给`dataDef`或`propsDef`设置一下`set`，给用户一个不允许修改的友好的提醒。

接下来又在`Vue.prototype`上定义了三个方法

```js
Vue.prototype.$set = set
  Vue.prototype.$delete = del

  Vue.prototype.$watch = function (
    expOrFn: string | Function,
    cb: any,
    options?: Object
  ): Function {
      // 省略
  }
```



然后是`eventsMixin`方法，打开`core/instance/event.js`，发现`eventsMixin`函数给`Vue.prototype`原型上添加了`$on`,`$once`,`$off`,`$emit`4个方法。

```js
export function eventsMixin (Vue: Class<Component>) {
    Vue.prototype.$on = function (event: string | Array<string>, fn: Function): Component {}
	Vue.prototype.$once = function (event: string, fn: Function): Component {}
	Vue.prototype.$off = function (event?: string | Array<string>, fn?: Function): Component {}
	Vue.prototype.$emit = function (event: string): Component {}    
}
```



然后是`lifecycleMixin`方法，打开`core/instance/lifecycle`，发现`lifecycleMixin`函数给`Vue.prototype`原型上添加了`_update`，`$forceUpdate`，`$destroy`3个方法。

```js
export function lifecycleMixin (Vue: Class<Component>) {
 	Vue.prototype._update = function (vnode: VNode, hydrating?: boolean) {}
	Vue.prototype.$forceUpdate = function () {}
    Vue.prototype.$destroy = function () {}
}
```



然后是`renderMixin`方法，打开`core/instance/render`，

```js
export function renderMixin (Vue: Class<Component>) {
    installRenderHelpers(Vue.prototype)
    Vue.prototype.$nextTick = function (fn: Function) {}
    Vue.prototype._render = function (): VNode {}
}
```

函数开头把`Vue.prototype`作为参数传入到`installRenderHelpers`这个函数中。

```js
export function installRenderHelpers (target: any) {
  target._o = markOnce
  target._n = toNumber
  target._s = toString
  target._l = renderList
  target._t = renderSlot
  target._q = looseEqual
  target._i = looseIndexOf
  target._m = renderStatic
  target._f = resolveFilter
  target._k = checkKeyCodes
  target._b = bindObjectProps
  target._v = createTextVNode
  target._e = createEmptyVNode
  target._u = resolveScopedSlots
  target._g = bindObjectListeners
  target._d = bindDynamicKeys
  target._p = prependModifier
}
```

这个函数方法来自于`core/instance/render-helpers/index`中，这个方法的作用就是给`Vue.prototype`中添加了一系列的方法。

然后又在`Vue.prototype`中添加了`$nextTick`，`_render`2个方法。



至此，`instance/index.js`文件中的代码运行完毕，大概可以理解为每个`*Mixin`方法的作用其实就是为了包装`Vue.prototype`，在其上挂载一些属性和方法。

```js
// initMixin(Vue)    src/core/instance/init.js **************************************************
Vue.prototype._init = function (options?: Object) {}

// stateMixin(Vue)    src/core/instance/state.js **************************************************
Vue.prototype.$data
Vue.prototype.$props
Vue.prototype.$set = set
Vue.prototype.$delete = del
Vue.prototype.$watch = function (
  expOrFn: string | Function,
  cb: any,
  options?: Object
): Function {}

// eventsMixin(Vue)    src/core/instance/events.js **************************************************
Vue.prototype.$on = function (event: string | Array<string>, fn: Function): Component {}
Vue.prototype.$once = function (event: string, fn: Function): Component {}
Vue.prototype.$off = function (event?: string | Array<string>, fn?: Function): Component {}
Vue.prototype.$emit = function (event: string): Component {}

// lifecycleMixin(Vue)    src/core/instance/lifecycle.js **************************************************
Vue.prototype._update = function (vnode: VNode, hydrating?: boolean) {}
Vue.prototype.$forceUpdate = function () {}
Vue.prototype.$destroy = function () {}

// renderMixin(Vue)    src/core/instance/render.js **************************************************
// installRenderHelpers 函数中
Vue.prototype._o = markOnce
Vue.prototype._n = toNumber
Vue.prototype._s = toString
Vue.prototype._l = renderList
Vue.prototype._t = renderSlot
Vue.prototype._q = looseEqual
Vue.prototype._i = looseIndexOf
Vue.prototype._m = renderStatic
Vue.prototype._f = resolveFilter
Vue.prototype._k = checkKeyCodes
Vue.prototype._b = bindObjectProps
Vue.prototype._v = createTextVNode
Vue.prototype._e = createEmptyVNode
Vue.prototype._u = resolveScopedSlots
Vue.prototype._g = bindObjectListeners
Vue.prototype.$nextTick = function (fn: Function) {}
Vue.prototype._render = function (): VNode {}
```

上面为`instance/index.js`执行完毕之后，`Vue.prototype`上挂载的所有的方法和属性。



### Vue构造函数的静态属性和方法（全局API）

`instance/index.js`文件已经看完了，按照之前的文件路径往前回溯，接下来该打开`core/index.js`文件，`core/index`所有代码如下：

```js
import Vue from './instance/index'
import { initGlobalAPI } from './global-api/index'
import { isServerRendering } from 'core/util/env'
import { FunctionalRenderContext } from 'core/vdom/create-functional-component'

// 将Vue作为入参传递给initGlobalAPI函数，该函数来自于 './global-api/index'
initGlobalAPI(Vue)

// 在Vue.prototype 原型上挂载 $isServer 属性
Object.defineProperty(Vue.prototype, '$isServer', {
  get: isServerRendering
})

// 在Vue.prototype 原型上挂载 $ssrContext 属性
Object.defineProperty(Vue.prototype, '$ssrContext', {
  get () {
    /* istanbul ignore next */
    return this.$vnode && this.$vnode.ssrContext
  }
})

// 在Vue构造函数本身挂载了一个 FunctionalRenderContext 方法
// expose FunctionalRenderContext for ssr runtime helper installation
Object.defineProperty(Vue, 'FunctionalRenderContext', {
  value: FunctionalRenderContext
})

// Vue.version 存储了当前Vue的版本信息
// script/config.js文件，genConfig方法， __VERSION__: version 插件会把__VERSION__ 给替换成版本信息。
Vue.version = '__VERSION__'

export default Vue
```

文件开头导入了3个函数`initGlobalAPI`，`isServerRendering`，`FunctionalRenderContext`

```js
import { initGlobalAPI } from './global-api/index'
import { isServerRendering } from 'core/util/env'
import { FunctionalRenderContext } from 'core/vdom/create-functional-component'
```

然后在`Vue.prototype`原型上挂载了两个只读属性`$isServer`，`$ssrContext`， 接着又在`Vue`构造函数上定义了`FunctionalRenderContext`静态属性，这个属性用于`ssr服务端渲染`.

最后在`Vue`构造函数上添加了一个静态属性`version`，存储当前`Vue`的版本信息。



### initGlobalAPI

`initGlobalAPI`是一个函数，并且把`Vue`构造函数作为入参进行调用。

```js
initGlobalAPI(Vue)
```

打开`./global-api/index`查看`initGlobalAPI`这个方法：

```js
import config from '../config'
export function initGlobalAPI (Vue: GlobalAPI) {
  // config
  const configDef = {}
  configDef.get = () => config
  if (process.env.NODE_ENV !== 'production') {
    configDef.set = () => {
      warn(
        'Do not replace the Vue.config object, set individual fields instead.'
      )
    }
  }
  Object.defineProperty(Vue, 'config', configDef)
	// 省略
}
```

这段代码的主要作用就是在`Vue`构造函数上添加`config`属性，该属性是只读属性，当试图修改的时候就会给一个友好的只读提示。

`config`的值来自于`core/config.js`文件

```js
import config from '../config'
```

所以`Vue.config`属性所代理的是`core/config.js`文件中导出的对象。



然后是这样一段代码：

```js
import {
  warn,
  extend,
  nextTick,
  mergeOptions,
  defineReactive
} from '../util/index'

// exposed util methods.
// NOTE: these are not considered part of the public API - avoid relying on
// them unless you are aware of the risk.
Vue.util = {
    warn,
    extend,
    mergeOptions,
    defineReactive
}
```

这段代码是在`Vue`上添加了`util`对象属性，这个对象拥有4个属性，分别是`warn`，`extend`，`mergeOptions`,`defineReactive`，这4个属性来自于`core/util/index`文件。

一大段注释的作用就是告诉我们`Vue.util`和它下面的四个属性不是公共API的一部分。要避免使用，如果要使用的话，风险需要自己控制。所以，尽量不要去依赖使用它们。



接下来是这样的一段代码：

```js
Vue.set = set
Vue.delete = del
Vue.nextTick = nextTick

// 2.6 explicit observable API
Vue.observable = <T>(obj: T): T => {
    observe(obj)
    return obj
}

Vue.options = Object.create(null)
```

这段代码是给`Vue`的构造函数上添加了`set`，`delete`，`nextTick`，`observable`，`options`5个属性，这里的`options`还是一个空对象，通过`Object.create(null)`来创建。

`Vue.observable`在2.6新增的属性，作用是让一个对象可响应，`Vue`内部会用它来处理`data`函数返回的对象。

接下来的代码：

```js
ASSET_TYPES.forEach(type => {
   Vue.options[type + 's'] = Object.create(null)
})

// this is used to identify the "base" constructor to extend all plain-object
// components with in Weex's multi-instance scenarios.
Vue.options._base = Vue

extend(Vue.options.components, builtInComponents)
```

上面的代码中，`ASSET_TYPES`来自于`shared/constants.js`文件中，查看该文件发现：

```js
export const ASSET_TYPES = [
  'component',
  'directive',
  'filter'
]
```

所以上面的代码执行之后，`Vue.options`变成了这样：

```js
Vue.options = {
	component: null,
    directive: null,
    filter: null,
    _base: Vue
}
```

紧接着，是这句代码：

```js
extend(Vue.options.components, builtInComponents) // extend方法来自于 shared/util.js
```

```js
// extend 源代码
export function extend (to: Object, _from: ?Object): Object {
  for (const key in _from) {
    to[key] = _from[key]
  }
  return to
}
```

可以发现该方法的作用是将`from`的属性混合到`to`中，也就是把`builtInComponents`的属性混合到`Vue.options.components`中。

`builtInComponents` 来自于`core/components.js`,代码如下：

```js
import KeepAlive from './keep-alive'

export default {
  KeepAlive
}
```

所以，目前为止`Vue.options`的结构如下：

```js
Vue.options = {
	component: {
    	KeepAlive
    },
    directive: null,
    filter: null,
    _base: Vue,
}
```

在`initGlobalAPI`方法的最后部分，以`Vue`为参数调用了4个`init*`方法：

```js
// import { initUse } from './use'
// import { initMixin } from './mixin'
// import { initExtend } from './extend'
// import { initAssetRegisters } from './assets'  

initUse(Vue)
initMixin(Vue)
initExtend(Vue)
initAssetRegisters(Vue)
```

打开`core/global-api/use.js`找到`initUse`方法：

```js
export function initUse (Vue: GlobalAPI) {
  Vue.use = function (plugin: Function | Object) {
      // 省略
  }
}
```

`initUse`方法的作用就是在`Vue`构造函数上添加`Vue.use`方法，用来安装`Vue`的插件。



打开`core/global-api/mixin.js`找到`initMixin`方法：

```js
export function initMixin (Vue: GlobalAPI) {
  Vue.mixin = function (mixin: Object) {
    this.options = mergeOptions(this.options, mixin)
    return this
  }
}
```

`initMixin`方法的作用就是在`Vue`上添加`mixin`这个`全局API`。



打开`core/global-api/extend.js`找到`initExtend`方法：

```js
export function initExtend (Vue: GlobalAPI) {
  /**
   * Each instance constructor, including Vue, has a unique
   * cid. This enables us to create wrapped "child
   * constructors" for prototypal inheritance and cache them.
   */
  Vue.cid = 0
  let cid = 1

  /**
   * Class inheritance
   */
  Vue.extend = function (extendOptions: Object): Function {
      // 省略
  }
}
```

`initExtend` 方法在 `Vue` 上添加了 `Vue.cid` 静态属性，和 `Vue.extend` 静态方法。



打开`core/global-api/assets.js`找到`initAssetRegisters`方法：

```js
export function initAssetRegisters (Vue: GlobalAPI) {
  ASSET_TYPES.forEach(type => {
    Vue[type] = function (
      id: string,
      definition: Function | Object
    ): Function | Object | void {
        // 省略
    }
  }
}  
```

`initAssetRegisters`方法的作用就是给`Vue`增加了3个静态方法。`Vue.component`，`Vue.filter`，`Vue.directive`。



至此，`initGlobalAPI`方法的全部功能也全部看了一遍，作用是在`Vue`构造参数上面添加全局的`API`。

```js
// initGlobalAPI
Vue.config
Vue.util = {
	warn,
	extend,
	mergeOptions,
	defineReactive
}
Vue.set = set
Vue.delete = del
Vue.nextTick = nextTick
Vue.options = {
	components: {
		KeepAlive
		// Transition 和 TransitionGroup 组件在 runtime/index.js 文件中被添加
		// Transition,
    	// TransitionGroup
	},
	directives: Object.create(null),
	// 在 runtime/index.js 文件中，为 directives 添加了两个平台化的指令 model 和 show
	// directives:{
	//	model,
    //	show
	// },
	filters: Object.create(null),
	_base: Vue
}

// initUse ***************** global-api/use.js
Vue.use = function (plugin: Function | Object) {}

// initMixin ***************** global-api/mixin.js
Vue.mixin = function (mixin: Object) {}

// initExtend ***************** global-api/extend.js
Vue.cid = 0
Vue.extend = function (extendOptions: Object): Function {}

// initAssetRegisters ***************** global-api/assets.js
Vue.component =
Vue.directive =
Vue.filter = function (
  id: string,
  definition: Function | Object
): Function | Object | void {}

// expose FunctionalRenderContext for ssr runtime helper installation
Object.defineProperty(Vue, 'FunctionalRenderContext', {
  value: FunctionalRenderContext
})

Vue.version = '__VERSION__'
```



### Vue平台化的包装

现在，在我们弄清`Vue`构造函数的过程中已经看了两个重要的文件，`core/instance/index`和`core/index`文件，前者是`Vue`构造函数的定义文件，主要作用是定义`Vue`构造函数，并对其`Vue.prototype`原型添加属性和方法。即实例属性和实例方法。后者的作用是主要添加`全局API`，也就是静态属性和静态方法。两个文件都在`core`文件夹下，`core`存放的是和平台无关的代码，无论是`core/instance/index`还是`core/index`文件，目的都是为了包装核心的`Vue`，且这些包装都是和平台无关的。但是，`Vue` 是一个 `Multi-platform` 的项目（platform目录下包含web和weex），不同平台可能会内置不同的组件、指令，或者一些平台特有的功能等等，那么这就需要对 `Vue` 根据不同的平台进行平台化地包装，这就是接下来我们要看的文件，这个文件也出现在我们寻找 `Vue` 构造函数的路线上，它就是：`platforms/web/runtime/index.js` 文件。

打开`platforms/web/runtime/index.js` 

```js
// install platform specific utils
Vue.config.mustUseProp = mustUseProp
Vue.config.isReservedTag = isReservedTag
Vue.config.isReservedAttr = isReservedAttr
Vue.config.getTagNamespace = getTagNamespace
Vue.config.isUnknownElement = isUnknownElement
```

这就是在覆盖默认导出的 `config` 对象的属性，注释已经写得很清楚了，安装平台特定的工具方法，至于这些东西的作用这里我们暂且不说，你只要知道它在干嘛即可。



然后：

```js
//import platformDirectives from './directives/index'
//import platformComponents from './components/index'
extend(Vue.options.directives, platformDirectives)
extend(Vue.options.components, platformComponents)
```

安装特定平台运行时的指令和组件，在此之前的`Vue.options`长这样：

```js
Vue.options = {
    components: {
		KeepAlive
	},
	directives: Object.create(null),
	filters: Object.create(null),
	_base: Vue
}
```

查看`web/runtime/directives/index`，代码如下：

```js
import model from './model'
import show from './show'

export default {
  model,
  show
}
```

查看`web/runtime/components/index`，代码如下：

```js
import Transition from './transition'
import TransitionGroup from './transition-group'

export default {
  Transition,
  TransitionGroup
}
```

所以经过`extend`之后，`Vue.options`将变成：

```js
Vue.options = {
    components: {
		KeepAlive,
        Transition,
	    TransitionGroup,
	},
	directives: {
    	model,
		show
    },
	filters: Object.create(null),
	_base: Vue
}
```



接下来的代码如下：

```js
// install platform patch function
Vue.prototype.__patch__ = inBrowser ? patch : noop

// public mount method
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  el = el && inBrowser ? query(el) : undefined
  return mountComponent(this, el, hydrating)
}
```

在`Vue.prototype`上添加`__patch__`方法，如果运行环境是浏览器的话，`__patch__`方法为`patch`，否则为`noop`。然后又在`Vue.prototype`上添加了`$mount`方法。

在往下的一段代码是`vue-devtools`的全局钩子，被包裹在`setTimeout`中，判断如果是浏览器环境下，如果`devtools`为`true`则初始化`vue-devtools`插件，最后导出`Vue`

```js
// devtools global hook
/* istanbul ignore next */
if (inBrowser) {
  setTimeout(() => {
    if (config.devtools) {
      if (devtools) {
        devtools.emit('init', Vue)
      }
      // 省略
  },0)
}
```

看完`platform/web/runtime/index.js`的代码，该文件的作用是对`Vue`进行平台化的包装。

+ 设置平台化的`Vue.config`
+ 在`Vue.options`上混合了两个指令（directives），分别是`model` 和`show`
+ 在`Vue.options`上混合了两个组件（components），分别是`Transition` 和`TransitionGroup`。
+ 在`Vue.prototype`上添加了两个方法，`__patch__`和`$mount`。

在经过这个文件之后，`Vue.options` 以及 `Vue.config` 和 `Vue.prototype` 都有所变化，我们把这些变化更新到对应的 `附录` 文件里，都可以查看的到。



`Vue.prototype`：

```js
// initMixin(Vue)    src/core/instance/init.js **************************************************
Vue.prototype._init = function (options?: Object) {}

// stateMixin(Vue)    src/core/instance/state.js **************************************************
Vue.prototype.$data
Vue.prototype.$props
Vue.prototype.$set = set
Vue.prototype.$delete = del
Vue.prototype.$watch = function (
  expOrFn: string | Function,
  cb: any,
  options?: Object
): Function {}

// eventsMixin(Vue)    src/core/instance/events.js **************************************************
Vue.prototype.$on = function (event: string | Array<string>, fn: Function): Component {}
Vue.prototype.$once = function (event: string, fn: Function): Component {}
Vue.prototype.$off = function (event?: string | Array<string>, fn?: Function): Component {}
Vue.prototype.$emit = function (event: string): Component {}

// lifecycleMixin(Vue)    src/core/instance/lifecycle.js **************************************************
Vue.prototype._update = function (vnode: VNode, hydrating?: boolean) {}
Vue.prototype.$forceUpdate = function () {}
Vue.prototype.$destroy = function () {}

// renderMixin(Vue)    src/core/instance/render.js **************************************************
// installRenderHelpers 函数中
Vue.prototype._o = markOnce
Vue.prototype._n = toNumber
Vue.prototype._s = toString
Vue.prototype._l = renderList
Vue.prototype._t = renderSlot
Vue.prototype._q = looseEqual
Vue.prototype._i = looseIndexOf
Vue.prototype._m = renderStatic
Vue.prototype._f = resolveFilter
Vue.prototype._k = checkKeyCodes
Vue.prototype._b = bindObjectProps
Vue.prototype._v = createTextVNode
Vue.prototype._e = createEmptyVNode
Vue.prototype._u = resolveScopedSlots
Vue.prototype._g = bindObjectListeners
Vue.prototype.$nextTick = function (fn: Function) {}
Vue.prototype._render = function (): VNode {}

// platforms/web/runtime/index ***********
Vue.prototype.__patch__ = inBrowser ? patch : noop
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  el = el && inBrowser ? query(el) : undefined
  return mountComponent(this, el, hydrating)
}
```

`Vue.options`：

```js
Vue.options = {
    components: {
		KeepAlive,
        Transition,
	    TransitionGroup,
	},
	directives: {
    	model,
		show
    },
	filters: Object.create(null),
	_base: Vue
}
```

`Vue.config`：

```js
// core/config.js 里的属性
Vue.config.mustUseProp = mustUseProp
Vue.config.isReservedTag = isReservedTag
Vue.config.isReservedAttr = isReservedAttr
Vue.config.getTagNamespace = getTagNamespace
Vue.config.isUnknownElement = isUnknownElement
```





### with compiler

看完`platform/web/runtime/index.js`文件后，运行时版本的`Vue`构造函数已经成型，