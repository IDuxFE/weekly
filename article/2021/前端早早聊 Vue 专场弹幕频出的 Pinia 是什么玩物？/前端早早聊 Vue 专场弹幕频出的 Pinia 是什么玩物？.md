![](https://files.mdnice.com/user/15734/5f56f778-2d32-4cdd-b30a-7ef1ff0819a5.jpg)

### 初识

在 10 月 23 号的前端早早聊 `Vue` 生态大会上，`Q&A` 环节中 `Pinia` 这个关键词出现的频次可以说是相当高，几乎每个老师分享完，弹幕中都会出现“皮你呀(`Pinia`)”相关的问题。这是何方圣物？本文将带你去认识它。

在认识 `Pinia` 之前，我们先来看 [vue](https://github.com/vuejs/rfcs/pull/271) 上一个关于 `Vuex 5.x` 的 [RFC](https://github.com/kiaking/rfcs/blob/vuex-5/active-rfcs/0000-vuex-5.md) ：

![](https://files.mdnice.com/user/15734/8efafbf1-1d91-4b13-882e-b0d2795a6697.png)

描述中可以看到，`Vue 5.x` 主要改善以下几个特性：

- 同时支持 `composition api` 和 `options api` 的语法；
- 去掉 `mutations`，只有 `state`、`getters` 和 `actions`；
- 不支持嵌套的模块，通过组合 `store` 来代替；
- 更完善的 `Typescript` 支持；
- 清晰、显式的代码拆分；

而 `Pinia` 正是基于 [RFC](https://github.com/kiaking/rfcs/blob/vuex-5/active-rfcs/0000-vuex-5.md) 所生成的一个玩物。

![](https://files.mdnice.com/user/15734/9cfc03fb-acd4-4ab4-812a-034ba1b6a79a.png)

它的定位和特点也很明确：

- 直观，像定义组件一样地定义 `store`，并且能够更好地组合它们；
- 完整的 `Typescript` 支持；
- 关联 `Vue Devtools` 钩子，提供更好地开发体验；
- 模块化设计，能够构建多个 `stores` 并实现自动地代码拆分；
- 极其轻量（1kb），甚至感觉不到它的存在 ；
- 同时支持同步和异步 `actions`；

这么 `niubility` 特性，接下来就 `show you the code` 去逐一学习。

### 接触

光说不练，等于白学。这一小节通过介绍 `Pinia` 的 `API`，感受上一小节讲到的特性。

用 `vite` 快速起一个 `vue` 模板的项目：

```shell
yarn create @vitejs/app pinia-learning --template vue-ts

cd pinia-learning
yarn
yarn dev
```

项目运行起来后，安装 `pinia` 并初始化一个 `store`：

```shell
yarn add pinia
```

在 `src/main.ts` 下定义引用 `pinia` 插件：

```typescript
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import App from './App.vue'

createApp(App).use(createPinia()).mount('#app')
```

### 了解(State)

#### defineStore

之后就可以定义我们的 `store` 并在组件中使用，我们新建 `src/store/index.ts` 文件并定义一个 `store`：

```ts
import { defineStore } from 'pinia'

export default defineStore({
  id: 'app',

  state () {
    return {
      name: '码农小余'
    }
  }
})
```

在 `App.vue` 引入上述文件就可以使用该 `store`：

```html
<template>
  <div>{{ store.name }}</div>
</template>

<script setup lang="ts">
import useAppStore from './store/index'
const store = useAppStore()
console.log(store)
</script>
```

`defineComponent <==> defineStore`、`id <==> name`、 `state <==> setup`，**直观，像定义组件一样地定义 `store`** 到这里是能够体会到该特性的含义了。上述代码是在 composition api 中 setup 的用法，在 options api 中使用跟 Vuex 类似，通过 `mapState` 或者 `mapWritableState` 辅助函数来读写 `state`：

```html
<template>
  <div>{{ this.username }}</div>
  <div>{{ this.interests.join(',') }}</div>
</template>

<script lang="ts">
import { mapState, mapWritableState } from 'pinia'
import infoStore from '../store/info'

export default {
  name: 'HelloWorld',

  computed: {
    // 只读计算属性
    ...mapState(infoStore, ['interests']),
    
    // 读写计算属性
    ...mapWritableState(infoStore, {
      username: 'name'
    })
  },

  mounted () {
    this.interests.splice(1, 1, '足球')
    this.username = 'Jouryjc'
  }
}

</script>
```

#### storeToRefs

那么后半句是啥意思呢？**并且能够更好地组合它们** 。举个例子，马上就 11.11 ，小余当然得往购物车里面塞几本“葵花宝典”，于是乎就需要一个 `cart` 的 `store`：

```typescript
import { defineStore } from 'pinia'

export default defineStore('cart', {
  state () {
    return {
      books: [
        {
          name: '金瓶梅',
          price: 50
        },
        {
          name: '微服务架构设计模式',
          price: 139
        },
        {
          name: '数据密集型应用系统设计',
          price: 128
        }
      ]
    }
  }
})
```

然后在 `AppStore` 组合 `cartStore` :

```typescript
// store/index.ts
import { defineStore } from 'pinia'
import useCartStore from './cart'

export default defineStore('app', {
  state () {
    // 直接使用 cartStore
    const cartStore = useCartStore()

    return {
      name: '码农小余',
      books: cartStore.books
    }
  }
})
```

最终被 `App` 组件消费。⚠️ 直接解构 `store` 会使其失去响应式，为了在保持其响应式的同时从 `store` 中提取属性要使用 `storeToRefs` ，如下述代码所示：

```html
<template>
  <div>{{ name }}</div>
  <p>购物车清单：</p>
  <p v-for="book of books" :key="book.name">
    书名：{{ book.name }} 价格：{{ book.price }}
  </p>
</template>

<script setup lang="ts">
import { storeToRefs } from 'pinia'
import useAppStore from './store/index'

// ⚠️⚠️ 返回的是一个 reactive 对象，不能直接解构哦，使用 pinia 提供的 storeToRefs API
const { name, books } = storeToRefs(useAppStore())
</script>
```

此时的页面效果如图所示：

![](https://files.mdnice.com/user/15734/468b6d96-8af0-4e18-ab58-5116a7332a5b.png)

#### $patch

除了直接修改 `store.xxx` 的值，还可以通过 `$patch` 修改多个字段信息；下面在例子中添加购买数量、总价，并添加付款人：

```html
<template>
    <div>{{ name }}</div>
    <p>购物车清单：</p>
    <p v-for="book of books" :key="book.name">
        书名：{{ book.name }} 价格：{{ book.price }} 数量： {{ book.count }}
        <button @click="add(book)">+</button>
        <button @click="minus(book)">-</button>
        </p>
    <button @click="batchAdd">全部加到10本</button>
    <p>总价：{{ price }}</p>
    <button @click="reset()">重置</button>
</template>

<script setup lang="ts">
import { storeToRefs } from "pinia";
import useAppStore from "./store/index";
import type { BookItem } from "./store/cart";

const store = useAppStore();
const { name, books, price } = storeToRefs(store);

const reset = () => {
  store.$reset();
};

const add = (book: BookItem) => {
  // 直接修改 store.book
  book.count++;
};

const minus = (book: BookItem) => {
  // 直接修改 store.book
  book.count--;
};

const batchAdd = () => {
  // 通过 $patch 方法修改 store 多个字段
  store.$patch({
    name: '小I',
    books: [
      {
        name: "金瓶梅",
        price: 50,
        count: 10,
      },
      {
        name: "微服务架构设计模式",
        price: 139,
        count: 10,
      },
      {
        name: "数据密集型应用系统设计",
        price: 128,
        count: 10,
      },
    ],
  });
};
</script>
```

例子中添加了 “全部加到10本”和 “重置”按钮，点击前者会将全部书籍数量添加到 10 本，点击后者会重置成 0，下面是执行效果：

![](https://files.mdnice.com/user/15734/6bd974ee-7b17-4ae1-809e-f27b863e60b3.gif)

假如你只想将《微服务架构设计模式》的数量修改成10，通过 `$patch` 传对象的方法需要这么操作：

```html
<template>
	<button @click="batchAddMicroService">微服务加到10本</button>
</template>

<script>
	const batchAddMicroService = () => {
      store.$patch({
        name: '小I',
        books: [
          {
            name: "金瓶梅",
            price: 50,
            count: 0,
          },
          {
            name: "微服务架构设计模式",
            price: 139,
            count: 10,
          },
          {
            name: "数据密集型应用系统设计",
            price: 128,
            count: 0,
          },
        ],
      });
    }
</script>
```

可以看到，就算你只是修改数组（集合）的第二项，还是需要将整个 `books` 数组传入，于是就产生了将函数作为 `$patch` 参数的写法：

```html
<script>
const batchAddMicroService = () => {
  store.$patch((state) => {
    state.books[1].count = 10;
  });
}
</script>
```

上述代码重写了 `batchAddMicroService` 方法。

#### $subscribe

该方法跟 `vuex` 的 [subscribe](https://vuex.vuejs.org/api/#subscribe) 类似，用于监听 `state` 及其 `mutation` 动作。上述例子中我们订阅 `appStore` 的状态：

```typescript
const store = useAppStore();

store.$subscribe((mutation, state) => {
  console.log(mutation);
  console.log(state);
});
```

当我们添加一本《金瓶梅》时，会触发 `$subscribe`，`log` 结果如下图：

![](https://files.mdnice.com/user/15734/f38e32ab-526e-4807-8e3f-3f34a22972bd.jpg)

### Getters

`Getters` 就是 `store` 的计算属性(`computed`)。大部分时候，`Getter` 通过 `state` 值去做计算，这种情况下 `TypeScript` 能够正确的推断出类型。例如：

```typescript
export default defineStore('app', {
  state: () => {
    const userInfoStore = useUserInfoStore()
    const cartStore = useCartStore()

    return {
      name: userInfoStore.name,
      books: cartStore.books
    }
  },

  getters: {
    price: (state) => {
      return state.books.reduce((init: number, curValue: BookItem) => {
        return init += curValue.price * curValue.count
      }, 0)
    }
  }
})
```

我们将 `state` 和 `getters` 都改成**箭头函数**，这样就能在 `App.vue` 中正确推断出 `price` 的类型。我们来验证一下：

![](https://files.mdnice.com/user/15734/9334f56c-d12c-479a-9bcb-a7e00c47668c.jpg)

Bingo！如果在 `getters` 中使用 `this` 去访问 `state` 的话，需要显式声明返回值才能正确标记类型，我们来试试：

```typescript
export default defineStore('app', {
  // ...
  getters: {
    price () {
      return this.books.reduce((init: number, curValue: BookItem) => {
        return init += curValue.price * curValue.count
      }, 0)
    }
  }
})
```

结果如下：

![](https://files.mdnice.com/user/15734/547bb450-3ef9-4e41-80fd-7f6d01f847dd.jpg)

我们给 `price` 显示声明返回类型：

```typescript
export default defineStore('app', {
  // ...
  getters: {
    price (): number {
      return this.books.reduce((init: number, curValue: BookItem) => {
        return init += curValue.price * curValue.count
      }, 0)
    }
  }
})
```

此时又能正确地提示 `price` 的类型。

![](https://files.mdnice.com/user/15734/3a607680-4214-4979-a56a-4c7a765743af.jpg)

`Getters` 其他用法比如组合 `Getters`、在 `setup` 或 `options api` 中使用、传参等等都跟 `State` 类似，本节就不展开细述。

### Actions

`Actions` 相当于组件里的 `methods`。双 11 买东西当然免不了折扣，商家也在折扣这环节上设计了活动，能够让顾客自己随机一个折扣比率，于是在 `store` 中的 `actions` 下定义 `changeDiscountRate` 方法：

```typescript
export default defineStore('app', {
  state: () => {
    const userInfoStore = useUserInfoStore()
    const cartStore = useCartStore()
    const discountRate = 1

    return {
      name: userInfoStore.name,
      books: cartStore.books,
      discountRate
    }
  },

  actions: {
    changeDiscountRate () {
      this.discountRate = Math.random() * this.discountRate
    }
  }
})
```

跟 `Getters` 一样，`actions` 中也通过 `this` 去获取整个 `store`。我们通过异步 `actions` 让修改折扣有一个延迟效果：

```typescript
function getNewDiscountRate (rate: number): Promise<number> {
  return new Promise ((resolve) => {
    setTimeout(() => {
      resolve(rate * Math.random())
    }, 1000)
  })
}

export default defineStore('app', {
  // ...
  actions: {
    async changeDiscountRate () {
      this.discountRate = await getNewDiscountRate(this.discountRate)
    }
  }
})
```

#### $onAction

当我们想统计 `actions` 的时间或者记录折扣点击总次数的时候，`$onAction` 订阅器能够很方便地实现，下面是一个官方的示例：

```typescript
// App.vue
const unsubscribe = store.$onAction(
	({
    name,
    store,
    args,
    after,
    onError,
  }) => {
    const startTime = Date.now()
    console.log(`Start "${name}" with params [${args.join(', ')}].`)

    after((result) => {
      console.log(
        `Finished "${name}" after ${
          Date.now() - startTime
        }ms.\nResult: ${result}.`
      )
    })

    onError((error) => {
      console.warn(
        `Failed "${name}" after ${Date.now() - startTime}ms.\nError: ${error}.`
      )
    })
	}
)


function getNewDiscountRate (rate: number): Promise<number> {
  return new Promise ((resolve, reject) => {
    setTimeout(() => {
      // 这里通过reject结束promise
      reject(rate * Math.random())
    }, 1000)
  })
}

export default defineStore('app', {
  // ...

  actions: {
    async changeDiscountRate () {
      try {
        this.discountRate = await getNewDiscountRate(this.discountRate)
      } catch (e) {
        // 示例执行这部分逻辑
        throw Error(e)
      }
    }
  }
})
```

上述代码会以 `rejected` 的状态结束 `promise`，`$onAction` 能够执行到 `onError` 钩子，这个钩子是跟 `vue3` 的 [errorHandler](https://v3.vuejs.org/guide/tooling/deployment.html#tracking-runtime-errors) 挂钩，想知道这部分怎么实现滴可以三连持续关注最新内容哦，下篇文章将会从 `pinia` 源码角度分析。最后执行效果如下图所示：

![](https://files.mdnice.com/user/15734/c225e7d3-0493-4f04-a993-9c930bf7b56f.jpg)

从上图可以看到，第一个 `warn` 是订阅器 `$onAction` 返回的，第二个 `warn` 则是 `errorHandler` 中返回。

最后，`$onAction` 一般是在组件的 `setup` 建立，它会随着组件的 `unmounted` 而自动取消。如果你不想让它取消订阅，可以将第二个参数设置为 `true`：

```typescript
store.$onAction(callback, true)
```

### 深入(Plugins)

通过一些底层 `API`，我们能够各种各样的扩展：

- 给 `store` 添加新的属性；
- 给 `store` 添加新的选项；
- 给 `store` 添加新的方法；
- 包装已经存在的方法；
- 修改或者删除 `actions`；
- 基于特定的 `store` 做扩展；

光说不练，等于白学。下面就来实现一个 `pinia-plugin`，首先在 `store` 下建 `pinia-plugin.ts`：

```typescript
import { PiniaPluginContext } from 'pinia'

export default function myPiniaPlugin(context: PiniaPluginContext) {
  console.log(context)
}
```

然后在 `main.ts` 中引入插件：

```typescript
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import piniaPlugin from './store/pinia-plugin'
import App from './App.vue'

const pinia = createPinia()
pinia.use(piniaPlugin)

createApp(App).use(pinia).mount('#app')
```

`appStore` 打印出来的 `context` 如下：

![](https://files.mdnice.com/user/15734/e34a238e-3184-4875-acf7-9275f6f9c02b.jpg)

我们可以从 `context` 中拿到从 `createApp()` 中创建的 `app` 实例、`defineStore` 中的配置、从 `createPinia()` 中创建的 `pinia` 实例、当前 `store` 对象。有了这些信息，你要完成上述那些扩展无非只是基于 `context` 去发挥。 下面列对两个比较常见的用法举例说明。

#### 给 store 添加新的属性

我们实现一个添加 `state` 的插件：

```typescript
import { PiniaPluginContext } from 'pinia'

export default function myPiniaPlugin(context: PiniaPluginContext) {
  const { store } = context

  store.pluginVar = 'Hello store pluginVar'

  // 可以在 devtools 中使用 
  store.$state.pluginVar = 'Hello store $state pluginVar'

  if (process.env.NODE_ENV === 'development') {
    // 加上自定义属性可以被devtool捕获到
    store._customProperties.add('pluginVar')
  }

  setTimeout(() => {
    store.pluginVar = 'Hello store pluginVar2'
    store.$state.pluginVar = 'Hello store $state pluginVar2'
  }, 1000)
}
```

上述代码中 `store.pluginVar` 的改变不会被 `devtools` 监听到，但是可以通过 `store._customProperties.add('pluginVar')`  `store.$state.pluginVar` 的改变会被 `devtools` 监听到。

#### 基于特定的 store 做扩展

`action` 支持异步，我们在 `action` 中发送请求的需求会比较多，我们就可以将 `axios` 作为扩展加到 `store` 中：

```typescript
import { PiniaPluginContext } from 'pinia'
import { markRaw } from 'vue'
import axios from 'axios'

export default function myPiniaPlugin(context: PiniaPluginContext) {
  const { store } = context

  store.axios = markRaw(axios)
}
```

`store.testAxios` 调用的时候，我们就只需要直接通过 `this` 即可拿到 `axios` 对象：

```typescript
import { defineStore } from 'pinia'
import useCartStore, { BookItem } from './cart'
import useUserInfoStore from './info'

function getNewDiscountRate (rate: number): Promise<number> {
  return new Promise ((resolve, reject) => {
    setTimeout(() => {
      reject(rate * Math.random())
    }, 1000)
  })
}

export default defineStore('app', {
  // ...
  actions: {
    testAxios () {
      // this.testAxios.$get().then().catch(() => {})
      console.log(this.axios)
    }
  }
})
```

### Pinia or Vuex

`Pinia` 作为 `Vue` 状态管理的后起之秀和官方团队维护的 `Vuex`，什么时候该用哪个？这一小节来简单地对比一下。

首先下载量和社区方面：

![](https://files.mdnice.com/user/15734/8592c524-f90f-4906-b180-53938d7094b8.jpg)

![](https://files.mdnice.com/user/15734/0b588cc7-c869-4a47-aec9-0e4b4d606f33.jpg)

`Pinia` 作为后起之秀，不管是 `Stack Overflow` 上的问题解决方案还是下载量上，肯定都不如 `Vue` 核心团队推荐的 `Vuex`。其他方面的对比可以直接阅读 [pinia-vs-vuex](https://blog.logrocket.com/pinia-vs-vuex/) ，这篇文章非常详细地比较了二者。言而总之：小项目可以试水 `Pinia`，大项目还是用成熟的 `Vuex`。(另外，外链的对比时间比较早，目前 `Pinia` 好像是支持了时间旅行功能)

### 总结

本文详细地介绍了 Pinia 的基础使用，从 `state`、`Getters`、`Actions` 到 `Plugins`，都有例子辅助学习。Pinia 的特性也都在文中有涉及，比如`composable store`、`TypeScript` 的支持，`devtool` 插件的集成如下图所示：

![](https://files.mdnice.com/user/15734/89869829-2d1a-4e7e-9297-7d718a7854c8.jpg)

最后简单地跟 `Vuex` 做了比较，详细记得点击链接去查阅引文哦。相信你读完本文，对于 `Pinia` 的基础使用应该是没问题的。下篇文章就让我们从源码的角度去深入理解 `Pinia` 带来的这些特性，关注我，第一时间收获精彩内容。