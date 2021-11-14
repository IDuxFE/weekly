### Vue 响应式原理

[Vue目录结构介绍]: https://juejin.cn/post/7012417885624598564

前文我们主要对 `Vue目录结构` 进行了介绍，接下来主要介绍下 Vue 的核心原理之一: `响应式原理`。文章内容主要包括 

- Vue 的入口文件介绍
- new Vue 的过程
- data 是如何初始成响应式的
- 响应式的处理
- 开发过程的一些心得

### Vue 的入口文件介绍

#### 1. src/core/index.js

```javascript
import Vue from './instance/index'

initGlobalAPI(Vue); // 初始化一些全局API

export default Vue

```

沿着上次分析的入口, 实际上核心的 `Vue` 构造函数入口在 `src/core/index.js` , 在这个入口处, 主要是对 `Vue` 挂了各种静态属性( `Vue.config`, `Vue.options`, `Vue.util` ), 静态方法( `Vue.set`, `Vue.nextTick` ) 等等，静态属性里面主要是集合了影响 `Vue` 所有实例的配置, 例如当定义一个全局组件 `Vue.component`, 就是存在 `Vue.options.components` 配置下, 当后续子组件进行实例化时会通过 `Vue.extend` 进行一个 `mergeOption` 合并配置的操作, 这时候会根据不同配置的 `合并策略` 将子组件配置 `Sub.options` 继承于 `Vue.options`.

#### 2. src/core/instance/index.js

```javascript
import { initMixin } from './init'
import { stateMixin } from './state'
import { renderMixin } from './render'
import { eventsMixin } from './events'
import { lifecycleMixin } from './lifecycle'
import { warn } from '../util/index'

function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  this._init(options)
}

initMixin(Vue)
stateMixin(Vue)
eventsMixin(Vue)
lifecycleMixin(Vue)
renderMixin(Vue)

export default Vue

```

这个模块下主要做了以下操作:

- 定义 `Vue` 构造函数;
- 将 `Vue` 传入不同模块函数, 注入不同的逻辑;

### new Vue 的过程

```javascript
export function initMixin (Vue: Class<Component>) {
  Vue.prototype._init = function (options?: Object) {
    const vm: Component = this
    // a uid
    vm._uid = uid++ // 每个vue实例初始化定义一个唯一id

    let startTag, endTag
    
    // a flag to avoid this being observed
    vm._isVue = true // 用于标识当前是vue实例, 避免vm被进行拦截定义成响应式数据
																						
    initLifecycle(vm) // 初始化生命周期, 定义一些变量, $children, $refs等
    initEvents(vm) // 初始化事件系统, 主要用于$on, $emit实现通信
    initRender(vm) // 初始化和render相关的东西, 如vm.$createElement, 我们平时写的render函数的参数就是这个方法
    callHook(vm, 'beforeCreate') // 触发beforeCreate钩子
    initInjections(vm) // resolve injections before data/props // 初始化注入相关
    initState(vm)// 初始化数据相关, props,methods,data,computed,watch
    initProvide(vm) // resolve provide after data/props
    callHook(vm, 'created')

    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
      vm._name = formatComponentName(vm, false)
      mark(endTag)
      measure(`vue ${vm._name} init`, startTag, endTag)
    }

    if (vm.$options.el) {
      vm.$mount(vm.$options.el)
    }
  }
}
```

通过上面的代码, 能够发现, 我们平时 `new Vue` 时, 实际上就是执行了 `this._init` 方法, 在 `_init` 方法内部会做以下操作:

- 定义 `_isVue` 属性, 用于标识当前是 vue 实例, 避免实例进行拦截定义成响应式数据
- 初始化生命周期, 定义一些变量, $children, $refs 等
- 初始化事件系统, 主要用于$on, $emit 实现通信
- 初始化和 render 相关的东西, 如 vm.$createElement, 我们平时写的 render 函数的参数就是这个方法
- 触发 beforeCreate 钩子
- 初始化 `inject` 注入相关
- 初始化数据相关, props,methods,data,computed,watch 等等;
- 初始化 `provide` 相关
- 出发 `created` 钩子

从上面也不难理解, 在 `Vue` 的生命周期图中为啥说 `created` 生命周期中能够获取到数据, 因为 `initState` 初始化数据操作在 `created` 钩子前已经执行完毕了.

介绍完 `new Vue` 的主要过程后, 接下来着重根据我们平常定义的 `data` , 是如何定义成响应式数据的.

### data 是如何初始成响应式的

```javascript
export function initState (vm: Component) {
  vm._watchers = []
  const opts = vm.$options
  if (opts.props) initProps(vm, opts.props)
  if (opts.methods) initMethods(vm, opts.methods)
  if (opts.data) {
    initData(vm)
  } else {
    observe(vm._data = {}, true /* asRootData */)
  }
  if (opts.computed) initComputed(vm, opts.computed)
  if (opts.watch && opts.watch !== nativeWatch) {
    initWatch(vm, opts.watch)
  }
}
```

上面代码是 `initState` 内部的执行过程, 分别会按顺序对 `props`, `methods`, `data`, `computed`, `watch` 进行初始化操作, 其中对数据的响应式拦截主要根据 `initData` 进行分析

#### initData

```javascript
function initData (vm: Component) {
  let data = vm.$options.data
  data = vm._data = typeof data === 'function'
    ? getData(data, vm)
    : data || {}; // 初始化data, 如果传入的data为function, 则将data执行,并将返回值赋值给_data
  // proxy data on instance
  // 对key进行代理访问
  const keys = Object.keys(data)
  let i = keys.length
  while (i--) {
    
  } else if (!isReserved(key)) {
      proxy(vm, `_data`, key)
    }
  }
  // observe data
  // 对数据进行观测
  observe(data, true /* asRootData */)
}
```

在 `initData` 中, 主要做了如下几步:

- 获取传入的 `data` 配置, 如果传入为 `function`, 则调用 `data.call(vm, vm)`,  并将执行后的返回值赋值到 `vm._data` 中;
- 对数据进行代理, 再第一步处理完后, 我们的数据都是记录在 `_data` 属性中的, 那么如果需要访问时则是通过 `vm._data.xxx` 进行访问, 会比较冗余, 所以 `Vue` 通过代理, 能过使访问 `vm.xxx` 时, 实际上是访问的 `vm._data.xxx` , 简化开发人员编码;
- 对 `_data` 进行 `observe` 观测(响应式数据的核心);

### 响应式的处理

#### observe

```javascript
export function observe (value: any, asRootData: ?boolean): Observer | void {
  if (!isObject(value) || value instanceof VNode) {
    return
  }
  let ob: Observer | void
  ob = new Observer(value)
  return ob
}
```

将代码进行简化, 实际上观测数据就是根据对象生成一个 `observer` 实例, 在 `observer` 实例化时再对数据进行观测;

#### Observer

```javascript
export class Observer {
  value: any;
  dep: Dep;
  vmCount: number; // number of vms that have this object as root $data

  constructor (value: any) {
    this.value = value
    def(value, '__ob__', this)
    if (Array.isArray(value)) {
      if (hasProto) {
        protoAugment(value, arrayMethods)
      } else {
        copyAugment(value, arrayMethods, arrayKeys)
      }
      this.observeArray(value)
    } else {
      this.walk(value)
    }
  }

  /**
   * Walk through all properties and convert them into
   * getter/setters. This method should only be called when
   * value type is Object.
   */
  walk (obj: Object) {
    const keys = Object.keys(obj)
    for (let i = 0; i < keys.length; i++) {
      defineReactive(obj, keys[i])
    }
  }

  /**
   * Observe a list of Array items.
   */
  observeArray (items: Array<any>) {
    for (let i = 0, l = items.length; i < l; i++) {
      observe(items[i])
    }
  }
}
```

根据上述代码, `Observer` 在进行实例化时, 主要做了以下操作:

- 在数据上定义一个不可枚举对象 `__ob__`, `__ob__` 指向当前 `observer` 实例
- 判断是否是数组, 如果是数组, 则修改 `__proto__` 指向拦截过的数组方法上, 并且 `observeArray` 遍历数组再对数组每一项进行递归观测
- 如果是对象, 则通过 `walk` 遍历对象每个属性, 根据 `Vue2.x` 的核心 API `Object.defineProperty` 拦截属性的 `get`, `set` 方法, 后续在属性进行取值时通过 `get` 依赖收集, 而在设值时, 则通过 `set`触发更新.

#### \_\_ob\_\_

初看这个属性, 一直没明白主要的用途. 经过了解后发现这个属性大有用途:

- 可以用于判断当前属性对象是否已经是响应式数据了
- 由于模块化写法的问题, 通过 `__ob__`, 方便在拦截数组方法时获取到当前 `observer` 实例, 对新增数据进行观测

#### 数组响应式

```javascript
import { def } from '../util/index'

const arrayProto = Array.prototype
// 基于Array.prototype创建一个新对象, 即 arrayMethods.__proto__ = arrayProto
export const arrayMethods = Object.create(arrayProto) 

const methodsToPatch = [
  'push',
  'pop',
  'shift',
  'unshift',
  'splice',
  'sort',
  'reverse'
];// 会改变原数组的方法集合

/**
 * Intercept mutating methods and emit events
 */
// 对会改变数组自身的方法进行劫持操作, 在不影响原数组操作执行的同时, 方便对数组新增内容进行观测
methodsToPatch.forEach(function (method) {
  // cache original method
  const original = arrayProto[method]; // 获取数组的
  def(arrayMethods, method, function mutator (...args) {
    const result = original.apply(this, args)
    const ob = this.__ob__
    let inserted
    switch (method) {
      case 'push':
      case 'unshift':
        inserted = args
        break
      case 'splice':
        inserted = args.slice(2)
        break
    }
    if (inserted) ob.observeArray(inserted)
    // notify change
    ob.dep.notify()
    return result
  })
})
```

上面代码主要逻辑过程如下:

- 先获取 `Array.prototype`, 再根据该原型对象创建一个新的对象, 其中该对象的 `__proto__` 指向 `Array.prototype`, 也就意味着当我访问该对象的属性, 如果找不到则会顺着原型链往 `Array.prototype` 上查找

- `methodsToPatch` 意味着需要打补丁的方法, 其中里面的 7 个操作都是会修改数组自身内容的, 所以需要劫持这几个数组操作, 目的是拿到新增的数据, 并对新增的数据进行观测操作;那么代码里面的 `this.__ob__` 怎么理解呢?

  - 举个例子:

    ```javascript
    const vm = new Vue({
    	data () {
            return {
                arr: [1,2,3]
            }
        }
    });
    vm.arr.push(4);
    ```

    上面这个代码, 当对 `vm.arr` 进行 `push` 操作时, 实际上就会走到上面劫持的方法上, 此时函数里的 `this` 指向 `vm.arr`, 即该数组对象, 而我们在前面有对 `value` 定义了 `__ob__` 属性, 所以可以在内部通过 `this.__ob__` 获取到`observer` 实例, 从而通过调用 `observer.observeArray`, 对 `push`, `unshift`, `splice` 新增的属性进行观测, 从而实现数组的观测效果

#### 对象响应式

```javascript
export function defineReactive (
  obj: Object,
  key: string,
  val: any,
  customSetter?: ?Function,
  shallow?: boolean
) {
  const dep = new Dep()
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter () {
      dep.depend();
      return val
    },
    set: function reactiveSetter (newVal) {
      if (newVal === value) {
        return
      }
      val = newVal
      dep.notify();
    }
  })
}
```

对于对象响应式数据的简单实现如上面所示, 即我们在对属性进行取值时, 则会出发 `dep.depend` 进行依赖收集, 而在进行值的设置时, 通过 `dep.notify` 进行通知更新依赖。而更具体的 `依赖收集` 以及 `触发更新` 过程在后续章节再进行介绍。

### 开发过程的一些心得

- 在 `data` 定义数据时, 如果不希望暴露该数据给外部则以 `_`或者 `$` 开头, 这样定义的数据不会定义到实例上进行代理访问;

  ```javascript
  const vm = new Vue({
      data () {
          return {
              _count: 0
          }
      }
  });
  vm._count // undefined;
  vm._data._count // 0
  ```

  之所以这样的原因是

  - 约定以`_`, `$` 开头的属性为私有属性, 外部无法直接访问;
  - 为了避免与 `Vue` 的一些内部实现发生冲突, 如 `_router`, `_route` , 以及内部定义的一些私有变量`_events` 等, 所以 `data`中以`_`, `$` 开头的变量则不会代理到实例上;

- 在开发过程中尽量对属性取值时缓存起来

  ```plain
  const vm = new Vue({
      data () {
          return {
              count: 0
          }
      },
      mounted () {
      	console.time('time');
      	for (let i = 0; i < 1000000; i++) {
      		this.count++;	
      	}
      	console.log(this.count);
   	    console.timeEnd('time'); // time: 160.903076171875 ms
      }
  });
  ```

  ```plain
  const vm = new Vue({
      data () {
          return {
              count: 0
          }
      },
      mounted () {
      	let { count } = this;
      	console.time('time');
      	for (let i = 0; i < 1000000; i++) {
      		count++;	
      	}
      	console.log(count);
   	    console.timeEnd('time'); // time: 2.114990234375 ms
      }
  });
  ```

  之所以造成上述差距的原因主要是, `Vue` 对 `data` 中的属性通过 `Object.defineProperty` 进行了拦截处理, 其中内部做了各种的判断和逻辑处理, 所以在 demo1 中的循环里每次取值都会进入到 `get` 的逻辑中, 造成性能问题. 而 demo2 中仅一开始进行取值操作并缓存起来, 后续操作的都是该缓存值, 所以避免了 demo1 的问题, 自然而然时间差就明显对比开.

### 总结

本文主要对 `Vue的入口文件进行介绍` , 接着介绍了 `new Vue的过程`, 初始化生命周期, 事件系统, render相关, 状态相关(props, methods, data, computed, watch)等, 以及根据 `initData` 介绍了 Vue2.x 是如何基于 `Object.defineProperty` 实现响应式数据的。紧接着根据该章节提供两个开发过程中遇到过的心得分享。 篇幅有限较多内容有所忽略, 如果有哪里不够具体的欢迎指出。
