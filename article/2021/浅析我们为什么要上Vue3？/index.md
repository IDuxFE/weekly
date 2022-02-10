---
theme: vue-pro
highlight: a11y-dark
---

## ✨ 开篇

**RT，我们为什么要上 Vue3？使用 Vue3 影响我开法拉利吗？**

最近在 Vue3 发布了`3.2` 大版本之后，掘金上关于 Vue3 的文章越来越多，本来想着让子弹再飞一会，但最近公司上了 Vue3 的项目，自己也跟着学了起来。

昨天还是 Vue2 的 Options API 的忠实信徒，结果今天搞了 Vue3 的 Composition API 之后，直呼 Vue3 真香！

接下来，我们从分析 Vue2 优缺点入手，以及结合图片和用例来了解 Vue3 的优势。

## Vue2 ⚔ Vue3

### Vue2

#### 优点

不可否认 Vue2 在取得的成功，在 Github 的 Front-End 分类的排行榜上也能看到 Vue 仓库的排名是第一，粉丝足够多，活跃用户足够多

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a8dace12e30945d181ec50023345c6cb~tplv-k3u1fbpfcp-watermark.image?)

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/46751d216b4847e48714d649f02cd97a~tplv-k3u1fbpfcp-watermark.image?)

比 React 足足多了 12k+ star

从我的角度来看，最大的功臣莫过于这三个：

- 响应式数据
- Options API
- SFC

除去这三位大将，不可或缺 `Vuex`/`Vue-Router` 这两位功臣，以及丰富的周边插件和活跃的社区。

#### 缺点

在一个组件仅承担单一逻辑的时候，使用 Options API 来书写组件是很清晰的。

但是在我们实际的业务场景中，一个父组件总是要包含多个子组件，父组件需要给子组件传值、处理子组件事件、直接操作子组件以及处理各种各样的调接口的逻辑，这时候我们的父组件的逻辑就会变得复杂。

我们从代码维护者的角度出发，假设这个时候有 10 个函数方法，自然我们要把他们放入到`methods`中，而这个 10 个函数方法又分别操作 10 个数据，这个 10 个数据又分别需要进行`Watch`操作。

这时候，我们根据`Vue2`的`Options API`的写法，就写完了 10 个`method`、10 个`data`、10 个`watch`，我们就将本来 10 个的函数方法，分割在 30 个不同的地方。

这时候父组件的代码，对代码维护者来说是很不友好的。

> 可能有人说，这么写可以增加代码量，老板夸我牛逼！哈哈哈哈 😂

### Vue3

#### 优点

> 自由，自由，还是 TM 的自由！

- 更强的性能，更好的 tree shaking
- Composition API + setup
- 更好地支持 TypeScript

#### 可能的缺点

> Vue3: 让你自由过了火

使用`Composition API`在`setup`这个舞台上尽情的表演之后，可能存在一个问题：那就是如何优雅地组织代码？

代码不能优雅的组织，在代码量上去之后，也一样很难维护。

### SFC 写法变化

![Options API对比Composition API.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/19a5449cbc1b45f4b6bb8941fe940180~tplv-k3u1fbpfcp-watermark.image?)

Vue2 一个普通的 Vue 文件

```js
<template>
  <div>
    <p>{{ person.name }}</p>
    <p>{{ car.name }}</p>
  </div>
</template>

<script>
export default {
  name: "Person",

  data() {
    return {
      person: {
        name: "小明",
        sex: "male",
      },
      car: {
        name: "宝马",
        price: "40w",
      }
    };
  },

  watch:{
      'person.name': (value) => {
          console.log(`名字被修改了, 修改为 ${value}`)
      },
      'person.sex': (value) => {
          console.log(`性别被修改了, 修改为 ${value}`)
      }
  },

  methods: {
    changePersonName() {
      this.person.name = "小浪";
    },

    changeCarPrice() {
      this.car.price = "80w";
    }
  },
};
</script>
```

采用 Vue3 Composition API 进行重写

```js
<template>
  <p>{{ person.name }}</p>
  <p>{{ car.name }}</p>
</template>

<script lang="ts" setup>
import { reactive, watch } from "vue";

// person的逻辑
const person = reactive<{ name: string; sex: string }>({
  name: "小明",
  sex: "male",
});
watch(
  () => [person.name, person.sex],
  ([nameVal, sexVal]) => {
    console.log(`名字被修改了, 修改为 ${nameVal}`);
    console.log(`名字被修改了, 修改为 ${sexVal}`);
  }
);
function changePersonName() {
  person.name = "小浪";
}

// car的逻辑
const car = reactive<{ name: string; price: string }>({
  name: "宝马",
  price: "40w",
});
function changeCarPrice() {
  car.price = "80w";
}
</script>
```

采用 Vue3 自定义 Hook 的方式，进一步拆分

```js
<template>
  <p>{{ person.name }}</p>
  <p>{{ car.name }}</p>
  <p>{{ animal.name }}</p>
</template>

<script lang="ts" setup>
import { usePerson, useCar, useAnimal } from "./hooks";

const { person, changePersonName } = usePerson();

const { car } = useCar();
</script>
```

```ts
// usePerson.ts
import { reactive, watch } from "vue";

export default function usePerson() {
  const person = reactive<{ name: string; sex: string }>({
    name: "小明",
    sex: "male",
  });
  watch(
    () => [person.name, person.sex],
    ([nameVal, sexVal]) => {
      console.log(`名字被修改了, 修改为 ${nameVal}`);
      console.log(`名字被修改了, 修改为 ${sexVal}`);
    }
  );
  function changePersonName() {
    person.name = "小浪";
  }
  return {
    person,
    changePersonName,
  };
}
```

```ts
// useCar.ts
import { reactive } from "vue";

export default function useCar() {
  const car = reactive<{ name: string; price: string }>({
    name: "宝马",
    price: "40w",
  });
  function changeCarPrice() {
    car.price = "80w";
  }
  return {
    car,
    changeCarPrice,
  };
}
```

对比完之后，我们会发现，Vue3 可以让我们更好组织代码。`person`和`car`的逻辑都被单独放置在一块

仅仅是代码组织的优势吗？不不不，我们再看看模板中的一些变化

- `<template>`标签中起始便签可以不用`<div>`标签，因为`Vue3`提供了`片段`的能力，使用`Vue-detools`中查看，可以看到有`fragment`的标记  
  ![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e2d30c1c9db648bea38eeecb4c7d75f1~tplv-k3u1fbpfcp-watermark.image?)
- `Watch`也不需要罗列多个了，`Vue3`中支持侦听多个源
- 很自然的使用`TypeScript`，类型约束更加简单

### 性能方面

Vue3 主要在这几个方面进行了提升

1. 编译阶段。对 diff 算法优化、静态提升等等
2. 响应式系统。`Proxy()`替代`Object.defineProperty()`监听对象。监听一个对象，不需要再深度遍历，`Proxy()`就可以劫持整个对象
3. 体积包减少。Compostion API 的写法，可以更好的进行 tree shaking，减少上下文没有引入的代码，减少打包后的文件体积
4. 新增`片段`特性。Vue 文件的`<template>`标签内，不再需要强制声明一个的`<div>`标签，节省额外的节点开销

## 🎤 总结

Vue3 的优点和新特性还有很多很多，本篇文章主要从`Vue2/3版本的优缺点`和`SFC写法变化`这两个角度介入，给大家介绍我们为什么要上 Vue3。

Vue3 很香，更好地支持 TypeScript，让 Vue3 成为了企业级 Vue 项目的首选，各种新特性也迎合前端技术的发展趋势。

> ie 浏览器：那我走？

## 👓 最后

> 翻译翻译什么是 Vue3 ？

如果这篇文章对你有帮助，那麻烦你动动小手点个赞 👍，你的每一个点赞都是我继续创作的莫大鼓励！

## 参考资料

[面试官：Vue3.0 性能提升主要是通过哪几方面体现的？](https://vue3js.cn/interview/vue3/performance.html#%E4%B8%80%E3%80%81%E7%BC%96%E8%AF%91%E9%98%B6%E6%AE%B5 "面试官：Vue3.0性能提升主要是通过哪几方面体现的？")

[Vue3 教程：Vue 3.x 快在哪里？](https://juejin.cn/post/6903171037211557895 "Vue3教程：Vue 3.x快在哪里？")

> 投稿作者：我是970  
> 原文链接：<https://juejin.cn/post/7040769191393099783>
