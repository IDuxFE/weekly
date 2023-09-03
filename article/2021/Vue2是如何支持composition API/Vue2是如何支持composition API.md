
### 前言
自从 `Vue3` 发布之后，`composition API` 这个词走入写 `Vue` 同学的视野之中，相信大家也一直听到 `composition API` 比之前的 `options API` 有多好多强，如今由于 `@vue/composition-api` 插件的发布，`Vue2` 的同学也可以上车咯，接下来我们主要以响应式的 `ref` 和 `reactive` 来深入分析一下，这个插件是怎么实现此功能的。

### 如何使用
```
// 入口文件引入并注册
import Vue from 'vue'
import VueCompositionAPI from '@vue/composition-api'

Vue.use(VueCompositionAPI)
```

```
// vue文件使用
import { defineComponent, ref, reactive } from '@vue/composition-api'

export default defineComponent({
  setup () {
     const foo = ref('foo');
     const obj = reactive({ bar: 'bar' });
     
     return {
        foo,
        obj
     }
  }
})
```

怎么样，看完是不是感觉和 `vue3` 一模一样，你可能会想：

- 这是 `vue2` 啊，我之前的 `data` 、`methods` 里面也有变量和方法，怎么做到跟 `setup` 返回值打通合并在一起的。

- `vue2` 不是只有定义在 `data` 里面的数据才会被处理成响应式的吗？ `ref` 和 `reactive` 是怎么做到的呢？

- `vue2` 响应式数据定义的约束（添加赋值原对象没有的属性，数组下标修改等），改用 `ref` 和 `reactive` 就没问题吗？

当然还有很多的疑惑，因为插件提供的 `API` 相当多，覆盖了绝大部分 `Vue3` 所拥有的，这里主要从这几个问题来分析一下是如何做到的。

### 原理解析


得益于 `Vue` 的插件系统，`@vue/composition-api` 像 `vue-router` 、`vuex` 一样也是通过官方提供的插件式来注入。

```
// 这里只贴跟本章要讲的相关代码

funciton mixin (Vue) {
    Vue.mixin({
      beforeCreate: functionApiInit
   }
}

function install (Vue) {
    mixin(Vue);
}

export const Plugin = {
  install: (Vue: VueConstructor) => install(Vue),
}
```
`Vue` 插件就是向外面暴露一个 `install` 的方法，当调用 `use` 的时候会调用该方法，并把 `Vue` 构造函数作为参数传入，然后调用 `Vue.mixin` 混入对应钩子时要处理的函数。

接下来主要看下 `functionApiInit` 做了什么

```
function functionApiInit(this: ComponentInstance) {
  const vm = this
  const $options = vm.$options
  const { setup, render } = $options
  
  // render 相关
  
  
  const { data } = $options
  
  $options.data = function wrappedData() {
    initSetup(vm, vm.$props)
    return isFunction(data)
     ? (
        data as (this: ComponentInstance, x: ComponentInstance) => object
      ).call(vm, vm)
     : data || {}
  }
  
```
因为 `Vue` 在 `beforeCreated` 和 `created` 生命周期之间，会 `initState` 对数据进行处理，其中对 `data`的处理时就会调用 `$options.data`拿到定义的数据，所以这里重新对该函数其包裹一层，这也是为什么要选择 `beforeCreate` 钩子注入的一个原因，必须在该函数调用前进行包裹。
接下来看 `initSetup`都做了什么

```
function initSetup(vm: ComponentInstance, props: Record<any, any> = {}) {
  const setup = vm.$options.setup!
  const ctx = createSetupContext(vm)
  const instance = toVue3ComponentInstance(vm)
  instance.setupContext = ctx
  
  def(props, '__ob__', createObserver())
  resolveScopedSlots(vm, ctx.slots)

  let binding: ReturnType<SetupFunction<Data, Data>> | undefined | null
  activateCurrentInstance(instance, () => {
    binding = setup(props, ctx)
  })

   // setup返回是函数的情况 需要重写render函数

  const bindingObj = binding

  Object.keys(bindingObj).forEach((name) => {
    let bindingValue: any = bindingObj[name]

    // 数据处理
    
    asVmProperty(vm, name, bindingValue)
  })
  return
  }
}
```

这个函数比较长，不在本次要讲解的主线上代码逻辑都删除了，这个函数主要是创建了 `ctx` 和把 `vm` 实例转换成 `Vue3` 数据类型定义的 `instance` ，然后执行 `setup` 函数得到返回值，然后遍历每个属性，调用 `asVmProperty` 挂载到 `vm` 上面，当然这里的挂载不是直接通过把属性和值添加到 `vm` 上面，这么做会有一个问题，就是后续对该属性的修改不能同步到 `vm` 中，这里采用的还是 `Vue` 最常见的数据代理。

```
export function asVmProperty(
  vm: ComponentInstance,
  propName: string,
  propValue: Ref<unknown>
) {
  const props = vm.$options.props
  if (!(propName in vm) && !(props && hasOwn(props, propName))) {
    if (isRef(propValue)) {
      proxy(vm, propName, {
        get: () => propValue.value,
        set: (val: unknown) => {
          propValue.value = val
        },
      })
    } else {
      proxy(vm, propName, {
        get: () => {
          if (isReactive(propValue)) {
            ;(propValue as any).__ob__.dep.depend()
          }
          return propValue
        },
        set: (val: any) => {
          propValue = val
        },
      })
    }
}    
```

看到这里，相信你已经明白了在 `setup` 中定义返回的为什么能够在 `template` 、 `data` 、 `methods` 等之中去使用了，因为返回的东西都已经被代理到 `vm` 之上了。


#### 响应式（ `ref`  `reactive` 的实现）

接下来我们来说说响应式相关的，为什么 `ref` 和 `reactive` 也可以让数据成为响应式的。


`ref` 的实现其实是对 `reactive` 再次封装，主要用来给基本类型使用。

```
function ref(raw?: unknown) {
  if (isRef(raw)) {
    return raw
  }

  const value = reactive({ [RefKey]: raw })
  return createRef({
    get: () => value[RefKey] as any,
    set: (v) => ((value[RefKey] as any) = v),
  })
}

```
因为 `reactive` 接受的必须是一个对象，所有这里使用了一个常量作为 `ref` 的 `key`, 也就是 
```
const value = reactive({
  "composition-api.refKey": row
})
```
``` 
export function createRef<T>(
  options: RefOption<T>,
  isReadonly = false,
  isComputed = false
): RefImpl<T> {
  const r = new RefImpl<T>(options)

  const sealed = Object.seal(r)
  if (isReadonly) readonlySet.set(sealed, true)
  return sealed
}

export class RefImpl<T> implements Ref<T> {
  readonly [_refBrand]!: true
  public value!: T
  constructor({ get, set }: RefOption<T>) {
    proxy(this, 'value', {
      get,
      set,
    })
  }
}
```
通过 `new RefImpl` 实例，该实例上有一个 `value` 的属性，对 `value` 做代理，当取值的时候返回 `value[RefKey]`，赋值的时候赋值给 `value[RefKey]`, 这就是为什么 `ref` 可以用在基本类型，然后对返回值的 `.value` 进行操作。调用 `object.seal` 是把对象密封起来（会让这个对象变的不能添加新属性，且所有已有属性会变的不可配置。属性不可配置的效果就是属性变的不可删除，以及一个数据属性不能被重新定义成为访问器属性，或者反之。但属性的值仍然可以修改。）


我们主要看下 `reactive` 的实现

```
export function reactive<T extends object>(obj: T): UnwrapRef<T> {
    const observed = observe(obj)
    setupAccessControl(observed)
    return observed as UnwrapRef<T>
}


export function observe<T>(obj: T): T {
  const Vue = getRegisteredVueOrDefault()
  let observed: T
  if (Vue.observable) {
    observed = Vue.observable(obj)
  } else {
    const vm = defineComponentInstance(Vue, {
      data: {
        $$state: obj,
      },
    })
    observed = vm._data.$$state
  }

  return observed
}
```
我们通过 `ref` 或者 `reactive` 定义的数据，最终还是通过了变成了一个 `observed` 实例对象，也就是 `Vue2` 在对 `data` 进行处理时，会调用 `observe` 返回的一样，这里在 `Vue2.6+` 把
`observe` 函数向外暴露为 `Vue.observable`，如果是低版本的话，可以通过重新 `new` 一个 `vue` 实例，借助 `data` 也可以返回一个 `observed` 实例，如上述代码。

因为在 `reactive` 中定义的数据，就如你在 `data` 中定义的数据一样，都是在操作返回的 `observed` ，当你取值的时候，会触发 `getter` 进行依赖收集，赋值时会调用 `setter` 去派发更新，
只是定义在 `setup` 中，结合之前讲到的 `setup` 部分，比如当我们在 `template` 中访问一个变量的值时，`vm.foo` -> `proxy` 到 `setup` 里面的 `foo` -> `observed` 的 `foo` ，完成取值的流程，这会比直接在 `data` 上多代理了一层，因此整个过程也会有额外的性能开销。

因此使用该 `API` 也不会让你可以直接规避掉 `vue2` 响应式数据定义的约束，因为最终还是用 `Object.defineProperty` 去做对象拦截，插件同样也提供了 `set API` 让你去操作对象新增属性等操作。

#### 总结
通过上面的了解，相信你一定对于 `Vue2` 如何使用 `composition API` 有了一定的了解，因为 `API` 相当多， 响应式相关的就还有 `toRefs、toRef、unref、shallowRef、triggerRef` 等等，这里就不一一分析，有兴趣的可以继续看源码的实现。

写 `Vue2` 的同学也可以不用羡慕写 `Vue3` 的同学了，直接引入到项目就可以使用起来，虽然没有 `vue3` 那么好的体验，但是绝大部分场景还是相同的，使用时注意 `README` 文档最后的限制章节，里面讲了一些使用限制。
