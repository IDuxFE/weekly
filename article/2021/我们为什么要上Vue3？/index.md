---
theme: vue-pro
highlight: a11y-dark
---

## ✨ 开篇

**RT，我们为什么要上 Vue3？使用 Vue3 影响我开法拉利吗？**

最近在 Vue3 发布了`3.2` 大版本之后，掘金上关于 Vue3 的文章越来越多，本来想着让子弹再飞一会，但最近公司上了 Vue3 的项目，自己也跟着学了起来。

昨天还是 Vue2 的 Options API 的忠实信徒，结果今天搞了 Vue3 的 Composition API 之后，直呼 Vue3 真香！

接下来，我们从分析 Vue2 优缺点入手，以及结合图片和一个简单的用例来了解 Vue3 的优势。

## Vue2 ⚔ Vue3

### Vue2

#### 优点

不可否认 Vue2 在国内的流行，以及取得地巨大的成功，占据国内大量的前端框架份额。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7ea2e7c9cbd148d9951da874737a7cdf~tplv-k3u1fbpfcp-watermark.image?)

从我的角度来看，最大的功臣莫过于这三个：

- 响应式数据
- Options API
- SFC

除去这三位大将，不可或缺的包含：Vuex/Vue-Router 这两位良将，以及丰富的周边插件和活跃的社区。

#### 缺点

在一个组件仅承担单一逻辑的时候，使用Options API来书写组件是很清晰的。

但是在我们实际的业务场景中，一个父组件总是要包含多个子组件，父组件需要给子组件传值、处理子组件事件、直接操作子组件以及处理各种各样的调接口的逻辑，这时候我们的父组件的逻辑就会变得复杂。

我们从代码维护者的角度出发，假设这个时候有 10 个函数方法，自然我们要把他们放入到`methods`中，而这个 10 个函数方法又分别操作 10 个数据，这个 10 个数据又分别需要进行`Watch`操作。

这时候，我们根据`Vue2`的`Options API`的写法，就写完了 10 个`method`、10 个`data`、10 个`watch`，我们就将本来 10 个的函数方法，分割在 30 个不同的地方。

> 可能有人说，这么写可以增加代码量，老板夸我牛逼！劝你别闹，哈哈哈哈😂

这时候父组件的代码，对代码维护者来说是很不友好的。

### Vue3

#### 优点

> 自由，自由，还是 TM 的自由！

- 更强的性能
- Composition API + setup
- 更好地支持 TypeScript

#### 可能的缺点

> Vue3: 不再忍心让你受折磨，是我让你自由过了火~

使用`Composition API`在`setup`这个舞台上尽情的表演之后，可能存在一个问题：那就是如何优雅地组织代码？

代码不能优雅的组织，在代码量上去之后，也一样很难维护。

这时候有大佬就说：自定义 Hook 可以解决这个问题。

对，是的。自定义 Hook 提供了我们优雅组织代码的方式，接下来如何组织好，就看我们自身的功力了。

### 图解区别

![Options API对比Composition API.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bd06f155f12f477f95bfd0545afb5b89~tplv-k3u1fbpfcp-watermark.image?)

### SFC的写法对比

Vue2 一个普通的 Vue 文件

```js
<template>
  <div>
    <p>{{ person.name }}</p>
    <p>{{ car.name }}</p>
    <p>{{ animal.name }}</p>
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
      },
      animal: {
        name: "柯基",
        age: 2,
      },
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
    },

    changeAnimalAge() {
        this.animal.age = 3;
    },
  },
};
</script>

```

采用 Vue3 Compisition API 进行重写

```js
<template>
  <p>{{ person.name }}</p>
  <p>{{ car.name }}</p>
  <p>{{ animal.name }}</p>
</template>

<script lang="ts" setup>
import { reactive, watch } from "vue";

// person的逻辑
const person = reactive<{ name: string; sex: string }>({
  name: "sex",
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

// animal的逻辑
const animal = reactive<{ name: string; age: number }>({
  name: "age",
  age: 2,
});
function changeAnimalAge() {
  animal.age = 3;
}
</script>
```

对比完之后，我们会发现，Vue3 可以让我们更好组织代码

`person`、`car`和`animal`的逻辑都被单独放置在一块，代码的逻辑更加清晰

仅仅是代码组织的优势吗？不不不，我们再看看模板中的一些变化

- `<template>`标签中起始便签可以不用`<div>`标签，因为`Vue3`提供了`片段`的能力，使用`Vue-detools`中查看，可以看到有`fragment`的标记  
  ![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e2d30c1c9db648bea38eeecb4c7d75f1~tplv-k3u1fbpfcp-watermark.image?)
- `Watch`也不需要罗列多个了，`Vue3`中支持侦听多个源
- 很自然的使用`TypeScript`，类型约束更加简单，不需要再整`vue-class-component`那套组件类的写法了

## 🎤 总结

Vue3 的优点和新特性还有很多很多，本篇文章主要从`代码组织`和`SFC写法变化`这两个角度介入，给大家介绍我们为什么要上 Vue3。

Vue3 很香，更好地支持 TypeScript，让 Vue3 成为了企业级 Vue 项目的首选，各种新特性也迎合前端技术的发展趋势，让我们主动拥抱 Vue3 吧！

> ie浏览器：那我走？

## 👓 最后

> 翻译翻译什么是 Vue3 ？

如果这篇文章对你有帮助，那麻烦你动动小手点个赞 👍，你的每一个点赞都是我继续创作的莫大鼓励！

> 文章转载自：<https://juejin.cn/post/7040769191393099783><br/>
> 原文作者：我是970
