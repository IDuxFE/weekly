> 原文链接：[如何借助策略模式优化条件分支](https://juejin.cn/post/7080722969210650660)

## 前言

在日常开发中笔者每天都要碰到条件分支判断的场景，而刚毕业没几年的我在业务里面写了大量的 `if/else` 判断，有时候自己看见了都觉得很不舒服。于是苦恼的我开始思考，该如何优化条件分支代码呢，近期也读了一些文章，它们的文章都不约而同地提到了 `策略模式` ，我对这个东西产生了好奇，于是我花了很多时间去学习它的思想，并且想着用更好地方式给大家分享，而不是一开头就讲述概念。

## 场景引入

此时你皱着眉头正在修 BUG，此时产品经理走了过来跟你说：小伙子，有个很简单的注册功能需要实现一下，该表单需要包括以下用户信息：

- 用户名
- 密码
- 手机号

在提交表单的时候需要校验格式：

- 用户名不能为空。
- 密码长度不能小于 6 位。
- 手机号格式正确。

你大概分析了一下，感觉也还算是简单，你的思路如下：

- 提交的时候先拿到表单实例。
- 对每个表单项进行校验

问题是这个校验怎么写呢，于是你开始实现你的代码。

## 实现

### if/else 实现

首先我们先把 DOM 结构给搭建好，先不考虑样式情况还是很简单的：

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>design-pattern-strategy</title>
  </head>
  <body>
    <form action="/register" method="post" id="registerForm">
      <div class="form-item">
        请输入用户名:<input type="text" name="userName"/ >
      </div>

      <div class="form-item">
        请输入密码:<input type="password" name="password"/ >
      </div>

      <div class="form-item">
        请输入手机号码:<input type="text" name="phoneNumber"/ >
      </div>
      <button>提交</button>
    </form>
  </body>
</html>
```

接着我们需要实现我们的 JS 逻辑，先拿到 form 实例，然后对每个字段进行校验：

```js
const registerForm = document.getElementById('registerForm')

const validatorFn = (form) => {
  if (form.userName.value === '') {
    alert('用户名不能为空')
    return false
  }

  if (form.password.value.length < 6) {
    alert('密码长度不能少于 6 位')
    return false
  }

  if (!/(^1[3|5|8][0-9]{9}$)/.test(form.phoneNumber.value)) {
    alert('手机号码格式不正确')
    return false
  }

  return true
}

registerForm.onsubmit = function (e) {
  const isValid = validatorFn(this)

  if (!isValid) {
    return false
  }

  alert('success submit')

  e.preventDefault()
}
```

实现效果如下：

![Kapture 2022-03-20 at 21.28.04.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4741c6c34e9145968a474e76bffb6ac6~tplv-k3u1fbpfcp-watermark.image?)

你开开心心地提交了代码，然后开始“摸鱼”~

### 策略模式 实现

此时你的另外一个同事又接到需求：

- 用户名只能是字母和数字。
- 密码要改为不少于 8 位，最多 100 位。
- 增加邮箱验证。

它找到了 `validatorFn` 这个方法，看了你的 `if/else` 然后把规则加了进去，这时候这个验证函数越来越大...

而之后你发现这个规则不止注册模块用到，想复用这块代码时，你发现当初为啥要写这么多判断，无奈的你 `CV` 了一模一样的代码过去，最终这个代码成为了别人眼里的 `shit` ...

接下来的一个月时间里你开始修炼设计模式这门 “内功” ，发现策略模式可以很好地优化这部分逻辑，虽然之前的代码确实写得不好，但好在你对代码还有追求，于是你花了很长时间去优化这段代码，你明确了两个要求：

- 让规则可配置化，使用者不必关心具体内部实现，需要符合 “开放-封闭” 原则。
- 规则是可以复用的，方便其他模块使用。

首先需要一个规则对象，这个对象里面封装了所有规则方法处理，而且这些方法需要遵守一些参数规则：

- 第一个参数是要校验的值
- 最后一个参数接收校验失败时显示的文本
- 中间可以传递校验的参数

现在我们单独把这个文件放到一个地方进行维护：

**rules.js**

```js
export const rules = {
  // 非空判断
  isNonEmpty(value, errorMsg) {
    if (value === '') {
      return errorMsg
    }
  },
  // 最小长度限制
  minLength(value, errorMsg, length) {
    if (value.length < length) {
      return errorMsg
    }
  },
  // 是否为手机号
  isMobile(value, errorMsg) {
    if (!/(^1[3|5|8][0-9]{9}$)/.test(value)) {
      return errorMsg
    }
  },
}
```

有了这个对象后，我们可以统一添加各种各样的规则，而且如果规则发生变化也很容易去更改，不用去考虑具体的业务逻辑。

接着我们需要抽象一个 `Validator` 校验器，你可以理解是一个控制器，可以利用它来添加和触发规则。它有以下属性：

- `cache` 数组，里面保存了所有的规则配置。
- `add` 方法，通过该方法来向某个表单项添加规则。
- `validator` 方法，通过该方法触发所有表单项的校验。

我们实现一下这个类：

**validator.js**

```js
import { rules } from './rules.js'

class Validator {
  constructor() {
    this.cache = []
  }

  /**
   * 添加规则
   * @param {HTMLElement} formItem 被校验的表单项
   * @param {Array<{ rule: string; errorMsg: string }>} ruleConfig 校验规则
   */
  add(formItem, ruleConfig) {
    // 遍历所有规则配置，存放到 cache
    ruleConfig.forEach(({ rule, errorMsg }) => {
      // 获取参数 minLength:6 -> ['minLength', '6']
      const params = rule.split(':')

      // 将校验函数存入缓存
      this.cache.push(() => {
        // 校验规则
        const _rule = params.shift()
        // 校验的值
        params.unshift(formItem.value)
        // message
        params.push(errorMsg)

        return rules[_rule].apply(formItem, params)
      })
    })
  }

  validate() {
    // 调用所有校验函数
    for (const fn of this.cache) {
      const errorMsg = fn()

      if (errorMsg) {
        return errorMsg
      }
    }
  }
}

export default Validator
```

接下来我们需要去写业务逻辑：

```js
<script type="module">
  import Validator from './validator.js'

  const validatorFn = (registerForm) => {
    const validator = new Validator()

    validator.add(registerForm.userName, [
      {
        rule: 'isNonEmpty',
        errorMsg: '用户名不能为空',
      },
      {
        rule: 'minLength:6',
        errorMsg: '用户名长度不能小于 10 位',
      },
    ])

    validator.add(registerForm.password, [
      {
        rule: 'minLength:6',
        errorMsg: '密码长度不能小于 6 位',
      },
    ])

    validator.add(registerForm.phoneNumber, [
      {
        rule: 'isMobile',
        errorMsg: '手机号码格式不正确',
      },
    ])

    return validator.validate()
  }

  const registerForm = document.getElementById('registerForm')

  registerForm.onsubmit = function (e) {
    e.preventDefault()

    const errorMsg = validatorFn(this)

    if (errorMsg) {
      alert(errorMsg)
      return false
    }

    alert('success')
  }
</script>
```

此时代码维护性就更好了，之后即使新增了表单项也没有关系，我们只需要添加进去就完事了，而如果还是之前的做法话可能需要写大量的 `if/else` ，很显然代码是不好维护的，这种不用在业务逻辑里关心具体校验方法并以配置的形式添加规则的方式就是策略模式的一种体现，好像还是很模糊对吧...

so，到底什么是策略模式 ？

## 策略模式概念与使用场景

### 概念

用我个人理解的一句话总结就是：**为达到目的，不择手段，不管用什么方式！**

我们先来看第一个词 “策略”。策略从字面上来看就是可以实现目标的所有方案，在上面的例子中 `rules` 对象中所有的规则就是策略，再想想我们是为了什么才有了这些策略呢，再回头看看上面的 `Validator` 类，它提供了校验功能，使用者利用它来管理校验规则，而为了达成校验这个目的，才有了各种各样的策略。引用《JavaScript 设计模式与实践》的一句话：

> 策略模式指的是定义一系列的算法，把它们一个个封装起来。将不变的部分和变化的部分隔开是每个设计模式的主题，策略模式也不例外，策略模式的目的就是将算法的使用与算法的实现分离开来。

策略模式有两部分组成：

- 一组策略类：可以对应 `rules` 对象中所有方法，每一个方法都是一个策略。
- 一个 Context 类：接受使用者请求，把请求委托给某个策略，对应上面的 `Validator`

现在你应该至少了解了它的奥秘之处了吧，那它的使用场景呢？

### 使用场景

策略模式其实比较常用，一般当你发现你业务逻辑为了达到某个目标要进行大量判断的时候就可以考虑了，除了上面的表单校验案例，其实还有很多应用场景。

#### 移动支付

我们目前基本上所有消费都是扫码支付，但是支付的方式多种多样，有的人喜欢支付宝，有的人用微信，还有用银联的...

当你想到做一件事有很多种方式时就可以自然而然可以想到策略模式，“扫码支付” 是一个目标，“支付方式” 就是一组策略的集合。

#### 出行方式

我们日常出行可能有很多种方式，为达到这个目标，我们会选择以下几种：

- 走路
- 骑自行车
- 骑电动车
- 开车
- 坐飞机

这些不同的出行方式正是策略模式的用武之地。

## 策略模式在 Vue 源码中的应用

上面的案例可能实战用得比较少，大家平时肯定都是用框架进行开发，下面我以 `Vue 2.x` 为例，来讲讲它源码中使用策略模式的地方来加深大家的理解。

大家肯定都清楚，`new Vue` 的时候会传入一个 `option` 对象，或者使用 `mixin` 来给某个组件混入一些属性：

```js
new Vue({
  mixins: [],
  data: {
    // ...
  },
  methods: {
    // ...
  },
})
```

每个属性合并的规则其实都不一样，这些不同的规则其实就是策略模式中的策略，又称为 “算法”，使用者根本不需要操心内部的实现，我们只需要安装业去写 `options` 就可以了，下面我们来看看具体的实现。

首先 `new Vue` 的时候内部会调用 `_init` 方法 进行一些初始化：

**src/core/instance/index.js**

```js
function Vue(options) {
  // Vue构造函数只能通过new调用
  if (process.env.NODE_ENV !== 'production' && !(this instanceof Vue)) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  // 调用init方法
  this._init(options)
}
```

该` _init` 方法有一个逻辑是对传入的配置进行处理：

**src/core/instance/init.js**

```js
export function initMixin(Vue: Class<Component>) {
  Vue.prototype._init = function (options?: Object) {
    // 记录vue实例
    const vm: Component = this

    // 将Vue构造函数中options与用户传递的options进行合并
    vm.$options = mergeOptions(
      resolveConstructorOptions(vm.constructor), // 获取 vue 实例内部的配置，在 new 之前已经声明在 Vue.options  里面，不必关心
      options || {}, // 传入的配置
      vm,
    )
  }
}
```

注意这个 `mergeOptions` 它是很关键的一个方法，它会对传入的配置与已有的配置进行合并，使用它内部自己的 “算法”。该方法在 `src/core/util/options.js`, 下面是它的核心逻辑：

```js
export function mergeOptions(
  parent: Object,
  child: Object,
  vm?: Component,
): Object {
  // 从这可以看出不管是在组件实例还是根实例都可以接受函数
  if (typeof child === 'function') {
    child = child.options
  }

  // 将外部传入的 props、inject、directives 统一成内部的格式
  normalizeProps(child, vm)
  normalizeInject(child, vm)
  normalizeDirectives(child)

  // 递归合并 mixins 和 extends 选项到 parent 上
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

  // 创建一个普通对象，这个是最终返回的配置
  const options = {}
  let key

  // 遍历 parent 中所有属性，通过 `mergeField` 方法中的算法把配置合并到 options 对象
  for (key in parent) {
    mergeField(key)
  }
  // 遍历 child 所有不存在 parent 中的属性，通过 `mergeField` 方法中的算法把配置合并到 options 对象
  for (key in child) {
    if (!hasOwn(parent, key)) {
      mergeField(key)
    }
  }
  // 策略模式合并：为每一个配置选择一种不同的方式进行合并
  function mergeField(key) {
    const strat = strats[key] || defaultStrat
    options[key] = strat(parent[key], child[key], vm, key)
  }
  return options
}
```

首先会对传入的配置进行一个标准化，主要是对 `props`、`inject`、`directives` 三个属性进行统一化操作，在内部统一转化成一种格式：

```js
normalizeProps(child, vm)
normalizeInject(child, vm)
normalizeDirectives(child)
```

接下来会把传入配置中的 `extends` 与 `mixins` 递归合并到 `parent` 中：

```js
if (child.extends) {
  parent = mergeOptions(parent, child.extends, vm)
}
if (child.mixins) {
  for (let i = 0, l = child.mixins.length; i < l; i++) {
    parent = mergeOptions(parent, child.mixins[i], vm)
  }
}
```

然后进入最关键的环节，利用策略模式对 `parent` 和 `child` 进行配置合并：

```js
let key
// 下面两个迭代会通过 `mergeField` 方法中的算法把配置合并到 options 对象
for (key in parent) {
  mergeField(key)
}

for (key in child) {
  if (!hasOwn(parent, key)) {
    mergeField(key)
  }
}

// 策略模式合并
function mergeField(key) {
  const strat = strats[key] || defaultStrat // 要合并的属性双方只有一个时，使用 `defaultStrat` 即默认策略
  options[key] = strat(parent[key], child[key], vm, key)
}
```

我们回顾一下策略模式一般有两大类组成：

- 一组策略类：这里对应的就是 `strats` 和 `defaultStrat` （默认策略） 。
- 一个 Context 类：对应上面的  `mergeField`。

`strats` 则定义了每一个配置以什么方式来合并，即具体的算法，这些算法定义在当前文件（**src/core/util/options.js**），而因为篇幅问题，我不会把所有合并的策略列出来，这里挑选几个规则进行介绍。

#### 生命周期函数的合并策略

该策略首先会遍历所有的生命函数钩子，然后都使用 `mergeHook` 这个函数的算法进行合并：

```js
/**
 * 生命周期的合并策略
 * Hooks and props are merged as arrays.
 */
function mergeHook(
  parentVal: ?Array<Function>,
  childVal: ?Function | ?Array<Function>,
): ?Array<Function> {
  const res = childVal
    ? parentVal
      ? parentVal.concat(childVal)
      : Array.isArray(childVal)
      ? childVal
      : [childVal]
    : parentVal
  return res ? dedupeHooks(res) : res
}

LIFECYCLE_HOOKS.forEach((hook) => {
  strats[hook] = mergeHook
})
```

`LIFECYCLE_HOOKS` 定义在 `src/shared/constants.js` ，它列举了 Vue 中所有的生命周期：

```js
export const LIFECYCLE_HOOKS = [
  'beforeCreate',
  'created',
  'beforeMount',
  'mounted',
  'beforeUpdate',
  'updated',
  'beforeDestroy',
  'destroyed',
  'activated',
  'deactivated',
  'errorCaptured',
  'serverPrefetch',
]
```

接下来我们看一下 `mergeHook` 的实现，它的核心逻辑是一个很复杂的三元运算符组成，但是我们可以转换成下面这样去理解：

```js
let res
// child 中这个 hook 值不存在则直接返回 parent 的值
if (!childVal) {
  res = parentVal // 这里 parentVal 初始的时候还是一个函数，但是经过 `dedupeHooks` 的处理会返回一个数组
} else {
  if (parentVal) {
    // 当 parent 和 child 的这个 hook 都有值时，将值合并到一个数组
    res = parentVal.concat(childVal)
  } else {
    // 将 child 转化为数组
    res = Array.isArray(childVal) ? childVal : [childVal]
  }
}

// 放入队列
return res ? dedupeHooks(res) : res
```

总体来看就是把生命周期的 hooks 合并到了一个数组(队列)，这个就能解释了我们在使用 mixin 的时候生命周期被触发的时候，钩子函数都会被执行，甚至我们不用 mixin 也能写出骚操作：

```js
export default {
  created: [
    function () {
      console.log(1)
    },
    function () {
      console.log(2)
    },
  ],
}
```

当然实际开发还是不要乱来啊...

#### data/provide 的合并策略

当你使用 `extend` 或者 `mixin` 的时候，一般都会对配置进行合并，最常见的就是 `data` 了，它的合并策略是什么呢，我们一起来看看。

```js
strats.data = function (
  parentVal: any,
  childVal: any,
  vm?: Component,
): ?Function {
  // 交给 mergeDataOrFn 处理
  return mergeDataOrFn(parentVal, childVal, vm)
}

export function mergeDataOrFn(
  parentVal: any,
  childVal: any,
  vm?: Component,
): ?Function {
  return function mergedInstanceDataFn() {
    // 下面两步目的是得到 data 对象，如果 data 是函数则调用这个函数，返回对象
    const instanceData =
      typeof childVal === 'function' ? childVal.call(vm, vm) : childVal
    const defaultData =
      typeof parentVal === 'function' ? parentVal.call(vm, vm) : parentVal

    // child 返回的值不存在，使用 parent 的值，否则对两个对象进行合并
    if (instanceData) {
      return mergeData(instanceData, defaultData)
    } else {
      return defaultData
    }
  }
}
```

可以看到 `data` 在合并前会进行一些处理，除了根实例，我们 `data` 一般是一个函数，所以需要先调用它，拿到返回的对象，然后再调用 `mergeData` 进行合并：

```js
function mergeData(to: Object, from: ?Object): Object {
  // 不存在直接返回
  if (!from) return to
  // key - 当前要合并的属性, toVal - 原有的值，fromVal - 要合并的值
  let key, toVal, fromVal

  // 遍历待合并对象所有属性，从这可以看出作者更推荐 Reflect.ownKeys 这个方法遍历
  const keys = hasSymbol ? Reflect.ownKeys(from) : Object.keys(from)

  for (let i = 0; i < keys.length; i++) {
    key = keys[i]
    // in case the object is already observed...
    // 防止这个属性已经被设置响应式了，跳过对比（从这可以看出作者是多么严谨）
    if (key === '__ob__') continue
    // 拿到当前遍历的 value
    toVal = to[key]
    fromVal = from[key]

    // 目标对象没有待合并对象的 key，则直接合入并设置响应式
    if (!hasOwn(to, key)) {
      // 等同于 Vue.set / this.$set
      set(to, key, fromVal)
    } else if (
      // 双方的key都存在且对应的值是对象，则递归对比
      toVal !== fromVal &&
      isPlainObject(toVal) &&
      isPlainObject(fromVal)
    ) {
      mergeData(toVal, fromVal)
    }
  }

  return to
}
```

从上面我们可以看到，`data` 合并规则如下：

- 以 `to` 作为合并为目标，将 `from` 对象上的属性合并到 `to`对象上
- `from` 不存在不进行处理，直接返回 `to`
- 遍历 `from` 所有属性
  - `to` 没有这个属性，则使用 `set` 来设置为响应式属性
  - `to` 和 `from` 对应的 value 都是对象，则递归处理
  - 其他情况以 `to` 的值为准，不进行处理（比如 `to` 和 `from` 的值不为对象且不相等的情况）

可见代码是十分严谨的，值得我们学习。

除了上面提到的两个策略，其他属性的合并策略方式也是值得学习的，如果大家感兴趣的话可以继续深究啦，相信到这里你对策略模式有了进一步认识。

## 总结

本文开篇先引入了表单验证的场景，使用了两种不同的方式来解决这个场景问题，一个是比较常见的 `if/else`，另外一个则是策略模式，通过对比来突出策略模式的优点，之后进一步引出策略模式的概念以及它的应用场景。最后通过深入了解 Vue 的源码来了解策略模式的应用。当然 `switch/case` 也是可以优化分支流程的，可是并没有解决本质问题，在涉及到大量分支的时候维护性还是很差。

当然，策略模式也有缺点，比如策略很多的话，这个策略类就很庞大了（JS 比较好一些，用一个普通对象描述这个类就可以），所以需要根据实际场景去使用，希望文章对你有帮助。

## 参考

[1]曾探.JavaScript 设计模式与实践:人民邮电出版社,2015
