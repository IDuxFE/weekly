「时光不负，创作不停，本文正在参加[2022年中总结征文大赛](https://juejin.cn/post/7108989863126368286 "https://juejin.cn/post/7108989863126368286")」 |
| --------------------------------------------------------------------------------------------------------------------------

2022年7月1号，`Vue`正式发布了`2.7`版本 【[官方连接](https://blog.vuejs.org/posts/vue-2-7-naruto.html?continueFlag=6dc6bae7e9ec264bcfa355608f7f223f)】。同时，也宣告了**Vue 2 将在 2023 年底彻底结束生命周期**，此举也是为了照顾部分`Vue2`用户因为无法升级版本，但又想体验`Vue3`新特性的的情况，例如`组合式API`。

在介绍`2.7`版本之前，我们先来回顾一下`Vue`历代版本的代号，请看大图：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b6fef53af6e34f638603af817a094777~tplv-k3u1fbpfcp-watermark.image?)

可以看出来尤大是个老二次元了，以上动漫你们看过哪几部呢？
版本号                                                                                                                                                                                                     | 发布日期       | 相关图片                                                                                                                                       | 官方备注                                                                                                                                                | 相关链接                                                                                                                                                                                                                            |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Vue0.9 Animatrix 黑客帝国](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fvuejs%2Fvue%2Freleases%2Ftag%2Fv0.9.0 "https://github.com/vuejs/vue/releases/tag/v0.9.0")                              | 2014-02-25 | ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/157768fa44364ca7a29273f0534f780f~tplv-k3u1fbpfcp-zoom-1.image) | "...then man made the machine in his own likeness. Thus did man become the architect of his own demise." - The Instructor                           | 黑客帝国动画版 [movie.douban.com/subject/129…](https://link.juejin.cn?target=https%3A%2F%2Fmovie.douban.com%2Fsubject%2F1292347%2F "https://movie.douban.com/subject/1292347/")                                                        |
| [Vue0.10 Blade Runner 银翼杀手](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fvuejs%2Fvue%2Freleases%2Ftag%2Fv0.10.0 "https://github.com/vuejs/vue/releases/tag/v0.10.0")                        | 2014-03-24 | ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6124386ef9404e269f825a4d32769dd1~tplv-k3u1fbpfcp-zoom-1.image) | "A coding sequence cannot be revised once it's been established." -Tyrell                                                                           | 银翼杀手 [movie.douban.com/subject/129…](https://link.juejin.cn?target=https%3A%2F%2Fmovie.douban.com%2Fsubject%2F1291839%2F "https://movie.douban.com/subject/1291839/")                                                           |
| [Vue0.11 Cowboy Bebop 星际牛仔](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fvuejs%2Fvue%2Freleases%2Ftag%2F0.11.0 "https://github.com/vuejs/vue/releases/tag/0.11.0")                          | 2014-11-07 | ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8f98a8ce117544269f41e454a8fe8efc~tplv-k3u1fbpfcp-zoom-1.image) | "There are ends we don't desire, but they're inevitable, we have to face them. It's what being human is all about." - Jet Black                     | 星际牛仔 [movie.douban.com/subject/142…](https://link.juejin.cn?target=https%3A%2F%2Fmovie.douban.com%2Fsubject%2F1424406%2F "https://movie.douban.com/subject/1424406/")                                                           |
| [Vue0.12 Dragon Ball 龙珠](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fvuejs%2Fvue%2Freleases%2Ftag%2F0.12.0 "https://github.com/vuejs/vue/releases/tag/0.12.0")                             | 2015-06-13 | ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/903c5c06fe504ddcbced1f792bee4515~tplv-k3u1fbpfcp-zoom-1.image) | "If you keep calm you'll never be a super saiyan." - Goku                                                                                           | 龙珠 [movie.douban.com/subject/247…](https://link.juejin.cn?target=https%3A%2F%2Fmovie.douban.com%2Fsubject%2F24710121%2F "https://movie.douban.com/subject/24710121/")                                                           |
| [Vue1.0 Evangelion 新福音战士](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fvuejs%2Fvue%2Freleases%2Ftag%2F1.0.0 "https://github.com/vuejs/vue/releases/tag/1.0.0")                              | 2015-10-27 | ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6b7282a6a4ea40f798943eabb0aa1be0~tplv-k3u1fbpfcp-zoom-1.image) | "The fate of destruction is also the joy of rebirth." - SEELE                                                                                       | 新福音战士 [movie.douban.com/subject/104…](https://link.juejin.cn?target=https%3A%2F%2Fmovie.douban.com%2Fsubject%2F10428501%2F "https://movie.douban.com/subject/10428501/")                                                        |
| [Vue2.0 Ghost in the Shell 攻壳机动队](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fvuejs%2Fvue%2Freleases%2Ftag%2Fv2.0.0 "https://github.com/vuejs/vue/releases/tag/v2.0.0")                    | 2016-10-01 | ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/30ad3f1c82334b3b822d6d63721722e3~tplv-k3u1fbpfcp-zoom-1.image) | "Your effort to remain what you are is what limits you." - Puppet Master                                                                            | 攻壳机动队[movie.douban.com/subject/258…](https://link.juejin.cn?target=https%3A%2F%2Fmovie.douban.com%2Fsubject%2F25818101%2F "https://movie.douban.com/subject/25818101/")                                                         |
| [Vue2.1 Hunter X Hunter 全职猎人](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fvuejs%2Fvue%2Freleases%2Ftag%2Fv2.1.0 "https://github.com/vuejs/vue/releases/tag/v2.1.0")                        | 2016-11-23 | ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/47fada26fd4d4d328befd32f8323087e~tplv-k3u1fbpfcp-zoom-1.image) | "You should enjoy the little detours to the fullest. Because that's where you'll find the things more important than what you want." - Ging Freecss | 全职猎人 [movie.douban.com/subject/146…](https://link.juejin.cn?target=https%3A%2F%2Fmovie.douban.com%2Fsubject%2F1460919%2F "https://movie.douban.com/subject/1460919/")                                                           |
| [Vue2.2 Initial D 头文字D](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fvuejs%2Fvue%2Freleases%2Ftag%2Fv2.2.0 "https://github.com/vuejs/vue/releases/tag/v2.2.0")                              | 2017-02-26 | ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9d9436eaf6f14ab9a21dd35ef52d76f2~tplv-k3u1fbpfcp-zoom-1.image) | Fill'er up. High Octane.                                                                                                                            | [movie.douban.com/subject/186…](https://link.juejin.cn?target=https%3A%2F%2Fmovie.douban.com%2Fsubject%2F1867420%2F "https://movie.douban.com/subject/1867420/")                                                                |
| [Vue2.3 JoJo's Bizarre Adventure JOJO的奇妙冒险](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fvuejs%2Fvue%2Freleases%2Ftag%2Fv2.3.0 "https://github.com/vuejs/vue/releases/tag/v2.3.0")          | 2017-04-27 | ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d21b828b616c481cbad4fee3d3bec56f~tplv-k3u1fbpfcp-zoom-1.image) | It was me, Dio!                                                                                                                                     | JOJO的奇妙冒险 [movie.douban.com/subject/114…](https://link.juejin.cn?target=https%3A%2F%2Fmovie.douban.com%2Fsubject%2F11498785%2F "https://movie.douban.com/subject/11498785/")                                                    |
| [Vue2.4 Kill la Kill 斩服少女](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fvuejs%2Fvue%2Freleases%2Ftag%2Fv2.4.0 "https://github.com/vuejs/vue/releases/tag/v2.4.0")                           | 2017-07-13 | ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ec61a92c81524344bf38f5c4c8388357~tplv-k3u1fbpfcp-zoom-1.image) | Fear is freedom! Subjugation is liberation! Contradiction is truth!                                                                                 | [B站](https://link.juejin.cn?target=https%3A%2F%2Fwww.bilibili.com%2Fbangumi%2Fplay%2Fss419%2F%3Ffrom%3Dsearch%26seid%3D5649768096849536506 "https://www.bilibili.com/bangumi/play/ss419/?from=search&seid=5649768096849536506") |
| [Vue2.5 Level E 灵异E接触](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fvuejs%2Fvue%2Freleases%2Ftag%2Fv2.5.0 "https://github.com/vuejs/vue/releases/tag/v2.5.0")                               | 2017-10-03 | ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e54f281d196744058634393a5086f0ab~tplv-k3u1fbpfcp-zoom-1.image) |                                                                                                                                                     | 灵异E接触 [movie.douban.com/subject/247…](https://link.juejin.cn?target=https%3A%2F%2Fmovie.douban.com%2Fsubject%2F24703888%2F "https://movie.douban.com/subject/24703888/")                                                        |
| [Vue2.6 Macross 超时空要塞](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fvuejs%2Fvue%2Freleases%2Ftag%2Fv2.6.0 "https://github.com/vuejs/vue/releases/tag/v2.6.0")                               | 2019-02-04 | ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e294bf527c3a4107b47466c6331469a7~tplv-k3u1fbpfcp-zoom-1.image) |                                                                                                                                                     | [movie.douban.com/subject/205…](https://link.juejin.cn?target=https%3A%2F%2Fmovie.douban.com%2Fsubject%2F2058467%2F "https://movie.douban.com/subject/2058467/")                                                                |
| [Vue2.7 Naruto](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fvuejs%2Fvue%2Freleases%2Ftag%2Fv2.7.0 "https://github.com/vuejs/vue/releases/tag/v2.7.0")                                      | 2022-07-01 | ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cdf88b05b87043c78fd5a9abad7c0f07~tplv-k3u1fbpfcp-zoom-1.image)                                   |                                                                                                                                                     | 火影忍者 [movie.douban.com/subject/142…](https://link.juejin.cn?target=https%3A%2F%2Fmovie.douban.com%2Fsubject%2F1427318%2F "https://movie.douban.com/subject/1427318/")                                                           |
| [Vue3.0 One Piece 海贼王](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fvuejs%2Fvue-next%2Freleases%2Ftag%2Fv3.0.0 "https://github.com/vuejs/vue-next/releases/tag/v3.0.0")                     | 2020-09-18 | ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f77d198ab38a4e5fb41a32c0a02eabf2~tplv-k3u1fbpfcp-zoom-1.image) |                                                                                                                                                     | 海贼王 [book.douban.com/subject/203…](https://link.juejin.cn?target=https%3A%2F%2Fbook.douban.com%2Fsubject%2F2039832%2F "https://book.douban.com/subject/2039832/")                                                               |
| [Vue3.1 Pluto](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fvuejs%2Fvue-next%2Freleases%2Ftag%2Fv3.1.0 "https://github.com/vuejs/vue-next/releases/tag/v3.1.0")                             | 2021-06-08 | ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bf980b0bad2a4d4b92552c2eeb4a7efc~tplv-k3u1fbpfcp-zoom-1.image) |                                                                                                                                                     | [book.douban.com/subject/186…](https://link.juejin.cn?target=https%3A%2F%2Fbook.douban.com%2Fsubject%2F1863031%2F "https://book.douban.com/subject/1863031/")                                                                   |
| [Vue3.2 Quintessential Quintuplets 五等分的花嫁](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fvuejs%2Fvue-next%2Freleases%2Ftag%2Fv3.2.0 "https://github.com/vuejs/vue-next/releases/tag/v3.2.0") | 2021-08-10 | ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/23c2313391b2452e9b103e09499e6cff~tplv-k3u1fbpfcp-zoom-1.image) |                                                                                                                                                     | 五等分的花嫁 [kodansha.us/volume/the-…](https://link.juejin.cn?target=https%3A%2F%2Fkodansha.us%2Fvolume%2Fthe-quintessential-quintuplets-1%2F "https://kodansha.us/volume/the-quintessential-quintuplets-1/")                        |

> 作者：辛宝Otto\
> 链接：https://juejin.cn/post/6998842159835119630


# 好了，言归正传，我们看看`Vue2.7`更新了什么内容：

## 向后移植的功能

-   [Composition API](https://link.juejin.cn?target=https%3A%2F%2Fvuejs.org%2Fguide%2Fextras%2Fcomposition-api-faq.html "https://vuejs.org/guide/extras/composition-api-faq.html")
-   SFC [`<script setup>`](https://link.juejin.cn?target=https%3A%2F%2Fvuejs.org%2Fapi%2Fsfc-script-setup.html "https://vuejs.org/api/sfc-script-setup.html")
-   SFC [CSS v-bind](https://link.juejin.cn?target=https%3A%2F%2Fvuejs.org%2Fapi%2Fsfc-css-features.html%23v-bind-in-css "https://vuejs.org/api/sfc-css-features.html#v-bind-in-css")

此外，还支持以下 API：

-   `defineComponent()`：具有改进的类型推断（与`Vue.extend`相比）；
-   `h()`、`useSlot()`、`useAttrs()`、`useCssModules()`；
-   `set()`、`del()` 和 `nextTick()` 在 ESM 构建中也作为命名导出提供。

Vue 2.7 还支持在模板表达式中使用 ESNext 语法。使用构建系统时，编译后的模板渲染函数将通过为普通 JavaScript 配置的相同 `loaders / plugins`。这意味着如果为`.js`文件配置了 Babel，它也将应用于 SFC 模板中的表达式。


## 注意事项

-   在 ESM 构建中，这些 API 作为命名导出提供（仅限于命名导出）：

```
import Vue, { ref } from 'vue'

Vue.ref // undefined, 改用命名导出
```

-   在 UMD 和 CJS 构建中，这些 API 作为全局 Vue 对象上的属性暴露。

## 与 Vue 3 的行为差异

Composition API 使用 Vue 2 的基于 `getter/setter` 的响应式系统进行反向移植，以确保浏览器兼容性。 这意味着与 Vue 3 的基于 `proxy` 的系统存在一些重要的行为差异：

-   所有 Vue 2 更改检测警告仍然适用；
-   `reactive()`、`ref()` 和 `shallowReactive()` 将直接转换原始对象而不是创建代理：

```
// 在2.7中可行，在3.x中不可行
reactive(foo) === foo
```

-   `readonly()` 确实创建了一个单独的对象，但它不会跟踪新添加的属性并且不适用于数组；
-   避免在 `reactive()` 中使用数组作为 `root` 值，因为如果没有属性访问，则不会跟踪数组的变化（这将导致警告）；
-   Reactivity APIs 忽略带有 `symbol` 键的属性。

此外，以下功能是未移植的：

-   ❌ `createApp()`（Vue 2 没有独立的应用范围）
-   ❌ `<script setup>` 中的顶层 await（Vue 2 不支持异步组件初始化）
-   ❌ 模板表达式中的 TypeScript 语法（与 Vue 2 解析器不兼容）
-   ❌ Reactivity transform（仍处于试验阶段）
-   ❌ `options` 组件不支持 `expose` 选项（但 ）。

## 升级指南
### Vue CLI / webpack

（1）将本地 `@vue/cli-xxx` 依赖项升级到主要版本范围内的最新版本（如果适用）：

-   对于 v4： **~4.5.18**
-   对于 v5： **~5.0.6**

（2）将 Vue 升级到 ^2.7.0。 还可以从依赖项中删除 `vue-template-compiler`，因为在 2.7 中不再需要它。注意：如果正在使用 `@vue/test-utils`，可能需要暂时将它保留在依赖项中，但是这个要求也将在新版本的 Test Utils 中被取消。

（3）检查包管理器 `lock` 文件以确保以下依赖项满足版本要求。 它们可能是 `package.json` 中未列出的传递依赖项：

-   vue-loader: ^15.10.0
-   vue-demi: ^0.13.1

如果没有，需要删除 `node_modules` 和 `lock` 文件并重新安装，以确保它们升级到最新版本。

（4）如果之前使用过 `@vue/composition-api`，请将其导入更新为 vue。 注意，插件导出的一些 API，例如 `createApp`，未在 2.7 中移植。

（5）如果在使用 `<script setup>` 时遇到未使用的变量的 `lint` 错误，请将 `eslint-plugin-vue` 更新到最新版本 (9+)。

（6）Vue 2.7 的 SFC 编译器现在使用 PostCSS 8。 PostCSS 8 应该向后兼容大多数插件，但如果以前使用只能与 PostCSS 7 一起使用的自定义 PostCSS 插件，升级可能会导致问题。在这种情况下，需要将相关插件升级到与 PostCSS 8 兼容的版本。

### Vite

Vue2.7 对 Vite 的支持是通过一个新插件提供的：[@vitejs/plugin-vue2](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fvitejs%2Fvite-plugin-vue2 "https://github.com/vitejs/vite-plugin-vue2")。这个新插件需要 Vue 2.7 或更高版本并取代现有的 [vite-plugin-vue2](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Funderfin%2Fvite-plugin-vue2 "https://github.com/underfin/vite-plugin-vue2")。

注意，新插件不处理特定于 Vue 的 JSX / TSX `transform`，这是有意的。 Vue 2 JSX / TSX`transform`应该在一个单独的专用插件中处理，该插件将很快提供。

### Volar 兼容性

Vue 2.7 提供了改进的类型定义，因此不再需要安装 `@vue/runtime-dom` 来支持 Volar 模板类型推断。 现在只需要在 `tsconfig.json` 中进行以下配置：

```
{
  // ...
  "vueCompilerOptions": {
    "target": 2.7
  }
}
```

### Devtools 支持

Vue Devtools 6.2.0 增加了对检查 2.7 Composition API 状态的支持，但扩展可能仍需要几天时间在各个发布平台上通过审核。

## 2.7 版本的影响

Vue 2.7 是 Vue 2.x 的最终次要版本。 在这个版本之后，Vue 2 进入了 LTS（长期支持），从现在开始持续 18 个月，并且将不再接收新功能。这意味着 Vue 2 将在 2023 年底结束其生命周期。这应该为大多数生态系统迁移到 Vue 3 提供充足的时间。

## 额外细节

在准备此版本时，Vue 团队将 Vue 2 代码库从 Flow 移植到了 TypeScript，这是基于核心团队成员 `@pikax` 的努力。 这样更容易重用 Vue 3 中的代码，并为移植的 API 自动生成类型定义。 除此之外，还将单元测试从 Karma + Jasmine 移至 Vitest，从而大大提高了维护 DX 和 CI 的稳定性。


## 上手

新建个`Vite`项目，初始化后用编辑器打开：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ae3f0b252e2f4758a3d6981380fcc970~tplv-k3u1fbpfcp-watermark.image?)

查看`package.json`，默认装的是`Vue3`版本，这里直接卸载掉，然后我们安装最新的`vue@2.7.0-alpha.6`版本：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ce2e9cc78d7841d086c516fd37125950~tplv-k3u1fbpfcp-watermark.image?)

找到项目入口文件`src/main.js`，删掉无用代码，写一点测试代码：
```js
import Vue, { ref, watch } from "vue/dist/vue";

const App = {
  name: "App",
  template: `
        <div>
            <h2>vue2.7</h2>
            <div>当前数：{{ count }}</div>
            <div>当前数的10倍：{{ bigCount }}</div>
            <button @click="addCount">数字+1</button>
        </div>
    `,
  setup() {
    const count = ref(0);
    const bigCount = ref(0);
    function addCount() {
      count.value++;
    }
    watch(count, (val) => {
      bigCount.value = val * 10;
    });
    return {
      count,
      bigCount,
      addCount,
    };
  },
};

new Vue(App).$mount("#app");

```

执行`npm run dev`，在浏览器中打开：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e32a7135b98b4310aec1ed0fd28795a5~tplv-k3u1fbpfcp-watermark.image?)

点击没有任何问题：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/61ece49ab47246f4a0e282838ae1f813~tplv-k3u1fbpfcp-watermark.image?)

实践得出，在`Vue2.7`版本中使用`组合式API`是没有任何问题的。

## 参考
- 原文：[blog.vuejs.org/posts/vue-2…](https://link.juejin.cn/?target=https%3A%2F%2Fblog.vuejs.org%2Fposts%2Fvue-2-7-naruto.html "https://blog.vuejs.org/posts/vue-2-7-naruto.html")
- [[译]尤雨溪：Vue3将不会支持IE11 精力会投入到Vue2.7](https://juejin.cn/post/6946756821675671566)
- [Vue 2.7 正式发布，代号为 Naruto](https://juejin.cn/post/7115361618774622216)
- [[火速更新]不出所料 Vue2.7 是火影忍者！尤大提到的动漫你看过几部？](https://juejin.cn/post/6998842159835119630)

