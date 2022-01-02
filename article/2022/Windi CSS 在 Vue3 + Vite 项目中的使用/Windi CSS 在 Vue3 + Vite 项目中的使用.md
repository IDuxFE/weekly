Windi CSS 是一种异于 CSS 预处理器的 CSS 框架。在学 Windi CSS 之前，最好有「其书写习惯有大大的不同」的心理准备，然后才是去接受它，学习它。

# 认识 Windi CSS
Windi CSS 的设计灵感来源于 Tailwindcss，而 Windi CSS 比 Tailwindcss 有更快的加载体验。

Windi CSS 是一款提供功能类的 CSS 框架，最直观的特点就是，你的 CSS 不用离开 HTML 就能够使样式生效，即在 Vue 文件里面，你将不再需要写 style 标签来装载你的 CSS，而是通过 class 工具类，或者「自定义前缀 + 属性模式」来控制 CSS 样式的效果，这些 class 工具类或者属性模式都是 Windi CSS 内部现成的，或者是通过 Windi CSS 配置文件配置的。

使用 Windi CSS 功能类的例子：

## class 工具类
```html
<div class="py-8 px-8 max-w-sm mx-auto bg-white rounded-xl shadow-md space-y-2 sm:(py-4 flex items-center space-y-0 space-x-6)">
  <img class="block mx-auto h-24 rounded-full sm:(mx-0 flex-shrink-0)" src="/img/erin-lindford.jpg" alt="Woman's Face" />
  <div class="text-center space-y-2 sm:text-left">
    <div class="space-y-0.5">
      <p class="text-lg text-black font-semibold">Erin Lindford</p>
      <p class="text-gray-500 font-medium">产品经理</p>
    </div>
    <button class="px-4 py-1 text-sm text-purple-600 font-semibold rounded-full border border-purple-200 hover:(text-white bg-purple-600 border-transparent) focus:(outline-none ring-2 ring-purple-600 ring-offset-2)">
      消息
    </button>
  </div>
</div>
```

## 自定义前缀 + 属性模式
```html
<button 
  w:bg="blue-400 hover:blue-500 dark:blue-500 dark:hover:blue-600"
  w:text="sm white"
  w:font="mono light"
  w:p="y-2 x-4"
  w:border="2 rounded blue-200"
>
  Button
</button>
```
前缀如 `w:` 是可以自定义的，这个不是必须的，但是为了防止命名冲突可以加上最好还是加上。自定义前缀需要手动在配置文件中设置，这个等下会写到。

# 使用 Windi CSS
通过动手在项目中引入然后使用，也许会有一个对 Windi CSS 更清晰的了解。所以接下来，我会写一下如何在项目中引入 Windi CSS 并使用它来实现一些 Demo。

## 使用插件 
Windi CSS 有非常多的类名，所以安装插件配合使用是必要的。建议 VsCode + WindiCSS IntelliSense。

## Vue3 + Vite
这条命令可以快速搭建一个 Vue3 的项目。
```shell
npm init vite@latest my-vue-app --template vue
```

## 安装 Windi CSS
```shell
npm i -D vite-plugin-windicss windicss
```
## 配置
### vite.config.js
```js
// vite.config.js
import { defineConfig } from "vite";
import vue from "@vitejs/plugin-vue";
import WindiCSS from "vite-plugin-windicss";  // <==

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [vue(), WindiCSS()], // <==
});
```
### 新建 Windi CSS 配置文件

这个文件是没有的，需要手动添加，与 vite.config.js 同级。

文件名可以是以下其中的一个，它们都能够被识别。


-   `windi.config.js`
-   `tailwind.config.js`

TypeScript 项目则是：
-   `tailwind.config.ts`
-   `windi.config.ts`

如果你的项目是 monorepo 的代码管理模式，只要把 Windi CSS 配置文件创建到应用的根目录下，就可以生效。

```js
import { defineConfig } from "windicss/helpers";

export default defineConfig({
  /* 配置项... */
});
```

#### 配置项

##### safelist

这种情况无法检测到工具类
```html
<!-- 不会被检测到 -->
<div className={`p-${size}`}>
```

所以需要设置白名单
```js
safelist: "p-1 p-2 p-3 p-4",
```

safelist 也可以是一个数组

```js
function range(size, startAt = 1) {
  return Array.from(Array(size).keys()).map(i => i + startAt)
}
safelist: [
    range(3).map(i => `p-${i}`), // p-1 到 p-3
    range(10).map(i => `mt-${i}`), // mt-1 到 mt-10
]
```

##### attributify

设置为 true，将开启属性模式

```js
attributify: true
```

然后你就可以像这样写样式
```html
  <button
    w:w="100px"
    w:h="40px"
    w:bg="blue-700"
    w:m="10px"
    w:text="color-[#fff]"
  >
    btn
  </button>
```


![](https://files.mdnice.com/user/23672/d148a362-b7b2-42c0-8eaa-daac60189450.png)

但是，为了防止命名冲突，还是建议自定义一个前缀
```js
attributify: {
    prefix: "w:",
},
```

### main.js
```js
import { createApp } from "vue";
import App from "./App.vue";
import "virtual:windi.css"; // <==

createApp(App).mount("#app");
```
`virtual:windi.css`相当于按顺序导入这三个层
```js
import 'virtual:windi-base.css'
import 'virtual:windi-components.css'
import 'virtual:windi-utilities.css'
```
这里可以灵活变动，在中间引入你自己的 CSS 也是可以的。但是会被后面的覆盖。
```js
  import 'virtual:windi-base.css'
  import 'virtual:windi-components.css'
+ import './my-style.css' // <==
  import 'virtual:windi-utilities.css'
```

# 一些有意思的内容

## 引入图片

引入背景图片使用的属性是 `background-image`，但是这个需要自定义来使用。

```js
// windi.config.js
export default defineConfig {
  theme: {
    extend: {
      backgroundImage: theme => ({
        'app-bg': 'url('@app/src/assets/img/bg.svg')',
      }),
    },
  },
}
```

以下是使用`app-bg`这个工具类的例子

```html
<div w:bg="app-bg" w:w="full" w:h="full" w:pos="relative"></div>
```

这样就可以得到一个占满全屏的图片（但前提是`html`、`body`、`#app` 的高度已经为 100%）。

## important 前缀

```html
<div class="text-28px !text-purple-600 text-blue-600 p-110px">
    important class
</div>
```

![](https://files.mdnice.com/user/23672/ea1a1b3f-6dc4-4216-8519-68dca0b1a583.png)

## 响应式处理

会根据断点判断，移动端优先，有默认的断点，也可以自定义断点。

![](https://files.mdnice.com/user/23672/88f3ea1c-9455-4e67-a5bb-44ab70a7ab5a.png)

```js
// 自定义断点
export default defineConfig({
    theme: {
    screens: {
      phone: '320px',
      iPad: '1024px',
      sm: '1280px',
      md: '1366px',
      lg: '1440px',
      xl: '1920px',
    },
  },
})
```

### Demo

```html
<div class="phone:bg-red-400 phone:text-light-100 phone:p-10 iPad:p-50 iPad:bg-blue-700 lg:p-100 lg:bg-cyan-800 lg:text-50px">
    响应式
  </div>
```

以上可以把类写成组的形式
```html
<div class="phone:(bg-red-400 text-light-100 p-10) iPad:(p-50 bg-blue-700) lg:(p-100 bg-cyan-800 text-50px)">
    响应式
</div>
```

也可以写成属性模式，这样就不会把类写得很长
```html
 <div
    w:phone="bg-red-400 text-light-100 p-10"
    w:iPad="bg-blue-700 p-50"
    w:lg="p-100 bg-cyan-800 text-50px"
>
    响应式
</div>
```
#### phone

![](https://files.mdnice.com/user/23672/d663526e-00f5-4c76-9ef7-55ff8f55aac0.png)

#### iPad

![](https://files.mdnice.com/user/23672/b55e04ae-75a7-4f12-b643-c68e36d819d2.png)

#### lg

![](https://files.mdnice.com/user/23672/89d32ca8-c623-4248-96a9-0d7d39080f04.png)


另外，还可以有
```plain
lg  => greater or equal than this breakpoint
<lg => smaller than this breakpoint
@lg => exactly this breakpoint range
```

- lg:p-1
- <lg:p-2
- @lg:p-3

```html
 <div
    w:phone="bg-red-400 text-light-100 p-10"
    w:iPad="bg-blue-700 p-50"
    w:lg="p-100 bg-cyan-800 text-50px"
    class="<phone:bg-black text-light-100"
>
    响应式
</div>
```

![](https://files.mdnice.com/user/23672/7d849f4c-1f24-4813-aa87-e3a1559d427a.png)

## 指令

指令可以用于结合 CSS 来使用。

### @apply
```html
<div class="btn btn-blue">button</div>

<style scoped>
.btn {
  @apply font-bold py-2 px-4 rounded w-100px m-10px;
}
.btn-blue {
  @apply bg-blue-500 hover:bg-blue-700 text-white;
  padding-top: 1rem;
}
</style>
```

![](https://files.mdnice.com/user/23672/6ef3ef31-ca67-4d47-8f59-2e79f35e92d1.png)

### @variants

可以用来生成带有屏幕可变修饰，状态可变修饰，主题可变修饰的工具类
```html
<div class="btn-blue btn">button</div>

<style>
.btn {
  @apply font-bold py-2 px-4 rounded w-100px m-10px;
}
.btn-blue {
  @apply bg-blue-500 hover:bg-blue-700 text-white;
  padding-top: 1rem;
}

@variants focus, hover {   /** <== */
  .btn {
    @apply w-200px h-200px;
  }
}

@variants dark {
    
}
</style>
```

![](https://files.mdnice.com/user/23672/979f5c67-b09f-42a3-a192-8607e9760711.gif)

### @screen

通过名称来引用断点

```css
@screen sm {
  .custom {
    @apply text-lg;
  }
}
```


### theme()

`theme()` 函数允许你通过 `.` 运算符来获取你设置的值

```html
<span class="color-blue">color blue</span>

<style>
.color-blue {
  color: theme("colors.blue.400");
}
</style>
```


![](https://files.mdnice.com/user/23672/03154f37-ccc2-4238-a8ac-66b6f70e64be.png)


Windi CSS 提供了较多的工具类，也允许我们自己按需自定义样式，使用起来还蛮方便的，而且不用我们自己手写 CSS 样式，也不用花时间去思考类的命名、模块区分，但是书写风格与习惯与我们以前写的 less、stylus 很不一样。工具类如果一开始不熟悉，还是得多翻翻文档。还有就是需要有一些约定的规范，比如，class 太多了，是不是得考虑换成写属性模式，或者有些可以组合起来写的，最好组合写，这样才不会把工具类写得太乱。