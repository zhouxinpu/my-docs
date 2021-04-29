## core/util目录下的方法



### debug.js文件代码说明

该文件主要导出了4个函数：

```js
export let warn = noop
export let tip = noop
export let generateComponentTrace = (noop: any) // work around flow check
export let formatComponentName = (noop: any)
```

这4个变量都被初始化为空函数。

接下来是一个判断语句

```js
if (process.env.NODE_ENV !== 'production') {
    // 省略
}
```

通过这段代码能看出来这些代码只在非生产环境下才会生效。

接下来定义了三个变量：

```js
  const hasConsole = typeof console !== 'undefined'
  const classifyRE = /(?:^|[-_])(\w)/g
  const classify = str => str
    .replace(classifyRE, c => c.toUpperCase())
    .replace(/[-_]/g, '')
```

其中 `hasConsole` 用来检测宿主环境的 `console` 是否可用，`classifyRE` 是一个正则表达式：`/(?:^|[-_])(\w)/g`，用于 `classify` 函数，`classify` 函数的作用是将一个字符串的首字母转为驼峰并把`-` 或者`_`去除，classify` 的使用如下：

```js
console.log(classify('aa-bb__cc')) // AaBbCc
```

warn

tip

formatComponentName

generateComponentTrace



### env.js文件代码说明



```js
export const inBrowser = typeof window !== 'undefined'
```

通过判断window对象是否存在来判断是否是浏览器环境

```js
export const UA = inBrowser && window.navigator.userAgent.toLowerCase()
```

在浏览器环境下，返回浏览器的`userAgent`，并将该字符串小写化赋值给`UA`常量

```js
export const isIE = UA && /msie|trident/.test(UA)
```

通过浏览器的`userAgent`是否包含`msie|trident`字段来判断是否是`IE`浏览器。

```js
export const isIE9 = UA && UA.indexOf('msie 9.0') > 0
```

通过浏览器的`userAgent`是否包含`msie 9.0`字段来判断是否是`IE9`浏览器。

```js
export const isEdge = UA && UA.indexOf('edge/') > 0
```

通过浏览器的`userAgent`是否包含`edge/`字段来判断是否是`IE/edge版本`浏览器。

```js
export const inWeex = typeof WXEnvironment !== 'undefined' && !!WXEnvironment.platform
```

通过判断`WXEnvironment`对象是否存在且`WXEnvironment.platform`来判断是否是在`WX`环境

```js
export const weexPlatform = inWeex && WXEnvironment.platform.toLowerCase()
```

在`WX`环境下，返回`WXEnvironment.platform`，并将该字符串小写化赋值给`weexPlatform`变量

```js
export const isAndroid = (UA && UA.indexOf('android') > 0) || (weexPlatform === 'android')
```

判断是否是在`android`环境下

```js
export const isIOS = (UA && /iphone|ipad|ipod|ios/.test(UA)) || (weexPlatform === 'ios')
```

判断是否是在`IOS`环境下

```js
export const isChrome = UA && /chrome\/\d+/.test(UA) && !isEdge
```

判断是否是在`chrome`浏览器

```js
export const isPhantomJS = UA && /phantomjs/.test(UA)
```

判断是否是在`phantom`浏览器

```js
export const isFF = UA && UA.match(/firefox\/(\d+)/)
```

判断是否是在`firefox`浏览器

```js
// Firefox has a "watch" function on Object.prototype...
export const nativeWatch = ({}).watch
```

在 `Firefox` 中原生提供了 `Object.prototype.watch` 函数，所以当运行在 `Firefox` 中时 `nativeWatch` 为原生提供的函数，在其他浏览器中 `nativeWatch` 为 `undefined`。这个变量主要用于 `Vue` 处理 `watch` 选项时与其冲突。

```js
// can we use __proto__?
export const hasProto = '__proto__' in {}
```

用来检查当前环境是否可以使用对象的 `__proto__` 属性。

```js
  var supportsPassive = false;
  if (inBrowser) {
    try {
      var opts = {};
      Object.defineProperty(opts, 'passive', ({
        get: function get () {
          /* istanbul ignore next */
          supportsPassive = true;
        }
      })); // https://github.com/facebook/flow/issues/285
      window.addEventListener('test-passive', null, opts);
    } catch (e) {}
  }
```

检查在浏览器环境下是否支持`passive`

`passive`是`chrome`提出的新特性，用来告诉浏览器，当前页面内注册的事件监听器是否会调用`preventDefault`函数来阻止事件的默认行为。

```js
// this needs to be lazy-evaled because vue may be required before
// vue-server-renderer can set VUE_ENV
let _isServer
export const isServerRendering = () => {
  if (_isServer === undefined) {
    /* istanbul ignore if */
    if (!inBrowser && !inWeex && typeof global !== 'undefined') {
      // detect presence of vue-server-renderer and avoid
      // Webpack shimming the process
      _isServer = global['process'] && global['process'].env.VUE_ENV === 'server'
    } else {
      _isServer = false
    }
  }
  return _isServer
}
```

`isServerRendering` 函数的执行结果是一个布尔值，用来判断是否是服务端渲染。

判断不在浏览器环境`!inBrowser`也不是weex`!inWeex`，同时`global`存在，则再判断`global['process'].env.VUE_ENV`是否成立。

```js
// detect devtools
export const devtools = inBrowser && window.__VUE_DEVTOOLS_GLOBAL_HOOK__
```

判断在浏览器环境中是否有`vue_devtools`这个插件

```js
export function isNative (Ctor: any): boolean {
  return typeof Ctor === 'function' && /native code/.test(Ctor.toString())
}
```

判断是否是系统函数，比如`Date | Math | Function`等系统函数

```js
export const hasSymbol =
  typeof Symbol !== 'undefined' && isNative(Symbol) &&
  typeof Reflect !== 'undefined' && isNative(Reflect.ownKeys)
```

判断当前宿主环境是否支持原生 `Symbol` 和 `Reflect.ownKeys` 的可用性。

```js
let _Set
/* istanbul ignore if */ // $flow-disable-line
if (typeof Set !== 'undefined' && isNative(Set)) {
  // use native Set when available.
  _Set = Set
} else {
  // a non-standard Set polyfill that only works with primitive keys.
  _Set = class Set implements SimpleSet {
    set: Object;
    constructor () {
      this.set = Object.create(null)
    }
    has (key: string | number) {
      return this.set[key] === true
    }
    add (key: string | number) {
      this.set[key] = true
    }
    clear () {
      this.set = Object.create(null)
    }
  }
}

export interface SimpleSet {
  has(key: string | number): boolean;
  add(key: string | number): mixed;
  clear(): void;
}

export { _Set }
```

判断是否支持`Set`，支持就直接导出，不支持就自己基于`SimpleSet`接口实现`set`然后导出。

