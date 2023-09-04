---
theme: channing-cyan
---

> `@idux`是我司是基于`vue3.x`和`Typescript`开发的开源组件库，它拥有以下特性:
>
> - `Monorepo` 管理模式：`cdk`, `components`, `pro`
> - 开箱即用的高质量组件
> - 全面拥抱 `composition api`，从源码到文档
> - 完全使用 `TypeScript` 开发，提供完整的类型定义
> - 灵活的全局配置
> - 深入细节的主题定制能力
> - 国际化语言支持
>
> 相关链接：
>
> - Github: **[IDuxFE/idux](https://github.com/IDuxFE/idux)**
> - 官方文档: **[idux.site](https://idux.site/)**
>
> 感兴趣的朋友可以尝试使用，我们会一直维护 🌹~

## 前言

本篇文章介绍如何基于`vue3.3`+`vite4.x`+`ts`+`@idux`开发一套符合企业级规范的项目，相关技术栈：

- 包管理模式：`pnpm Monorepo`
- 框架：`vue3.3`
- 工程化：`vite4.x` + `rollup`
- 语言：`ts` + `tsx`
- 组件库：`@idux`
- css：`less`+`postcss`
- 请求库：`axios`
- 状态管理：`pinia`
- 单侧：`vitest`
- 代码规范：`eslint` + `prettier` + `stylelint`
- 提交规范：`husky`+`lint-staged`+`commitlint`

废话不多说，下面直接开始吧

## 环境准备

在开始之前，推荐以下开发环境：

1. `vscode`
2. `pnpm>=8.x`
3. `node>=12.x`
4. `chrome浏览器`
5. `vue-devtools`

## 初始化

使用`pnpm`创建项目

```javascript
pnpm create vite
```

输入完指令后会提示输入项目的名称，选择`vue`和`ts`之后，项目创建成功：
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/acfd682404c3463b929313513a436d67~tplv-k3u1fbpfcp-watermark.image?)
进入项目目录，执行`pnpm install`安装依赖：
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9bd3b584e72d4cf8b8281a6ba9c5855a~tplv-k3u1fbpfcp-watermark.image?)
`pnpm run dev`进入项目:
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1dd2a0cddd8745aeb954596f52b6c353~tplv-k3u1fbpfcp-watermark.image?)
看到这个界面说明项目初始化成功

## CSS 工程化

我们把项目中无用的代码删掉，只保留一个`App.vue`，这样保证我们的项目干净：
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d1c18818ca4c48fcb94a2af8a6c693e8~tplv-k3u1fbpfcp-watermark.image?)
添加`less`脚本：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e5d6065af8c6452790d32ee9691bed62~tplv-k3u1fbpfcp-watermark.image?)
这个时候会报错，这是很正常的，因为我们还没有装`less`：
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/72400424423b45e089e2d11c36de7c35~tplv-k3u1fbpfcp-watermark.image?)
安装`less`依赖：

```javascript
pnpm add less
```

再次启动，样式生效：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ec65d74732c34578800ec6f5573debbf~tplv-k3u1fbpfcp-watermark.image?)

试一下变量和嵌套样式，一样没有问题：
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7a4293bf918b4d44ab9bdbf98c10a987~tplv-k3u1fbpfcp-watermark.image?)

如果我们想引入全局的样式文件呢？例如把`@color`放在`style.less`文件中

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/070603e3c50d44a89fc3b280df06ff25~tplv-k3u1fbpfcp-watermark.image?)

解决方案有两种：

### 1、在 style 标签中引入 style.less 文件

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fc31006649294d04bec12b7388406f7d~tplv-k3u1fbpfcp-watermark.image?)

如果是某个`vue`组件特定的变量没有问题，但是对于全局性的变量或者样式，总不能每个都通过`@import`导入一次吧？

### 2、配置 preprocessorOptions

`vite`为我们提供了`preprocessorOptions`配置，可以在`style`标签或者`.less`文件中自动引入`style.less`文件：

```javascript
//vite.config.ts
import { defineConfig, normalizePath } from "vite";
import vue from "@vitejs/plugin-vue";
// 可以安装@types/node解决类型报错
import path from "path";

// 用 normalizePath 解决 window 下的路径问题
const variablePath = normalizePath(path.resolve("./src/style.less"));

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [vue()],
  css: {
    preprocessorOptions: {
      less: {
        additionalData: `@import "${variablePath}";`,
      },
    },
  },
});
```

我们安装`@types/node`解决`path`模块的类型错误

```javascript
pnpm add @types/node -D
```

使用`postcss`一般用来解析和处理`css`代码，通常通过`postcss.config.js`文件来配置`postcss`，不过`vite`已经提供了相关配置入口，可以直接在`vite.config.ts`进行操作，我们安装一个常见的插件，用来解决浏览器兼容性问题，那就是`autoprefixer`
安装`autoprefixer`:

```javascript
pnpm add autoprefixer -D
```

在`vite.config.ts`中配置：

```javascript
// vite.config.ts
import autoprefixer from "autoprefixer";

export default {
  css: {
    postcss: {
      plugins: [
        autoprefixer({
          // 指定目标浏览器
          overrideBrowserslist: ["> 1%", "last 2 versions"],
        }),
      ],
    },
  },
};
```

我们尝试使用一些较新的语法：

```less
.demo {
  &-text {
    color: @color;
    text-decoration: dashed;
  }
}
```

执行`pnpm run build`，可以看到产物中，已经在自动在样式前加上了前缀：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a320c1699e49480c8aabddc8829bef70~tplv-k3u1fbpfcp-watermark.image?)

OK，那`css工程化`相关的内容就讲到这。

## 代码规范

### 1、Eslint

安装`eslint`

```javascript
pnpm add eslint -D
```

执行`npx eslint --init`初始化

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4dadc54d034d4c939e2e901350704486~tplv-k3u1fbpfcp-watermark.image?)
步骤最后会问你是否要立即安装以上依赖，这里建议不要，因为默认用的是`npm`安装，我们更希望用`pnpm`进行安装

```javascript
pnpm add @typescript-eslint/eslint-plugin@latest eslint-plugin-vue@latest @typescript-eslint/parser@latest -D
```

可以看到初始化之后，项目自动新建了`.eslintrc.cjs`文件

```javascript
// .eslintrc.cjs
module.exports = {
  env: {
    browser: true,
    es2021: true,
  },
  extends: [
    "eslint:recommended",
    "plugin:@typescript-eslint/recommended",
    "plugin:vue/vue3-essential",
  ],
  overrides: [
    {
      env: {
        node: true,
      },
      files: [".eslintrc.{js,cjs}"],
      parserOptions: {
        sourceType: "script",
      },
    },
  ],
  parserOptions: {
    ecmaVersion: "latest",
    parser: "@typescript-eslint/parser",
    sourceType: "module",
  },
  plugins: ["@typescript-eslint", "vue"],
  rules: {
    "linebreak-style": ["error", "unix"],
    quotes: ["error", "double"],
    semi: ["error", "always"],
  },
};
```

解释一下上面几个关键的内容：

`parserOptions`是专门对解析器进行能力定制的`ecmaVersion: latest`表示启用最新的`es语法`；`sourceType: module`表示使用`ES Module`；`parser`解析器：`@typescript-eslint/parser`

ESLint 底层默认使用  [Espree](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Feslint%2Fespree "https://github.com/eslint/espree")来进行 AST 解析，这个解析器目前已经基于  `Acron`  来实现，虽然说  `Acron`  目前能够解析绝大多数的  [ECMAScript 规范的语法](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Facornjs%2Facorn%2Ftree%2Fmaster%2Facorn "https://github.com/acornjs/acorn/tree/master/acorn")，但还是不支持 TypeScript ，因此需要引入其他的解析器完成 TS 的解析。

社区提供了`@typescript-eslint/parser`这个解决方案，专门为了 TypeScript 的解析而诞生，将  `TS`  代码转换为  `Espree`  能够识别的格式(即  [**Estree 格式**](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Festree%2Festree "https://github.com/estree/estree"))，然后在 Eslint 下通过`Espree`进行格式检查， 以此兼容了 TypeScript 语法。

`plugins`插件中使用了`@typescript-eslint/eslint-plugin`(可以简写成`@typescript-eslint`)来对`TS代码`规则进行一些拓展，`vue`插件则专门针对`vue代码`

### 2、Prettier

搞定`eslint`之后，我们需要安装`Prettier`来做代码格式化，虽然`eslint`也可以做这件事，但是术业有专攻，`eslint`主要优势在于代码的风格检查并给出提示，所以企业中常常使用`Eslint`+`Prettier`的组合来约束我们的代码规范

安装`prettier`

```javascript
pnpm add prettier -D
```

在根目录新建`.prettierrc.cjs`配置文件，并填写如下配置内容：

```javascript
// .prettierrc.cjs
module.exports = {
  printWidth: 80, //一行的字符数，如果超过会进行换行，默认为80
  tabWidth: 2, // 一个 tab 代表几个空格数，默认为 2 个
  useTabs: false, //是否使用 tab 进行缩进，默认为false，表示用空格进行缩减
  singleQuote: true, // 字符串是否使用单引号，默认为 false
  semi: true, // 行尾是否使用分号，默认为true
  trailingComma: "none", // 是否使用尾逗号
  bracketSpacing: true // 对象大括号直接是否有空格，默认为 true
```

接下来需要把`Prettier`集成到现有的`Eslint`工具中，安装这两个包

```javascript
pnpm i eslint-config-prettier eslint-plugin-prettier -D
```

其中`eslint-config-prettier`用来覆盖 ESLint 本身的规则配置，而`eslint-plugin-prettier`则是用于让 Prettier 来接管`eslint --fix`即修复代码的能力。在  `.eslintrc.js`  配置文件中接入 prettier 的相关工具链

```javascript
// .eslintrc.cjs
module.exports = {
  env: {
    browser: true,
    es2021: true,
  },
  extends: [
    "eslint:recommended",
    "plugin:@typescript-eslint/recommended",
    "plugin:vue/vue3-essential",
    // 1. 接入 prettier 的规则
    "prettier",
    "plugin:prettier/recommended",
  ],
  overrides: [
    {
      env: {
        node: true,
      },
      files: [".eslintrc.{js,cjs}"],
      parserOptions: {
        sourceType: "script",
      },
    },
  ],
  parserOptions: {
    ecmaVersion: "latest",
    parser: "@typescript-eslint/parser",
    sourceType: "module",
  },
  plugins: [
    "@typescript-eslint",
    "vue",
    // 2. 加入 prettier 的 eslint 插件
    "prettier",
  ],
  rules: {
    // 3. 注意要加上这一句，开启 prettier 自动修复的功能
    "prettier/prettier": "error",
    "linebreak-style": ["error", "unix"],
    quotes: ["error", "double"],
    semi: ["error", "always"],
  },
};
```

在`package.json`文件中定义脚本

```json
// package.json
{
  "scripts": {
    // 省略已有 script
    "lint:script": "eslint --ext .js,.ts,.vue --fix --quiet ./src"
  }
}
```

然后我们执行命令试一下效果

```javascript
pnpm run lint:script
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/148766fc9ffa4477bd944205ee1f596a~tplv-k3u1fbpfcp-watermark.image?)

可以看到，校验已经生效了，根据报错提示不能使用单引号，这是因为我们在`.prettierrc.cjs`中配置了允许单引号，而初始化`eslint`时选择了双引号。我们修改需要一下规则

```javascript
// .eslintrc.cjs
{
    "rules": {
        "quotes": [
            "error",
            "single"
        ],
    }
}
```

再次运行命令，此时校验通过，不再报错

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/aef331b6da60428b92aa8f9aa42a2413~tplv-k3u1fbpfcp-watermark.image?)

关于如何配置`rules`，可以参考[Eslint rules 文档](https://eslint.org/docs/latest/rules/)，这里不过多赘述。
不过每次执行这个命令未免会有些繁琐，我们可以在`VSCode`中安装`ESLint`和`Prettier`这两个插件，并且在设置区中开启`Format On Save`:

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fdf2bdd06a374722b26a3f2ac117773d~tplv-k3u1fbpfcp-watermark.image?)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c2757b0d554b4f508319545f3208a4a5~tplv-k3u1fbpfcp-watermark.image?)

我们修改`main.ts`中的单引号变成双引号

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c32163fbcc3c4b6fb1589f84675f977f~tplv-k3u1fbpfcp-watermark.image?)

此时按`Ctrl+S`保存，自动格式化成单引号，则说明`Vscode`插件生效了，它会根据我们的`.eslintrc.cjs`和`.prettierrc.cjs`来自动格式化代码

![屏幕录制2023-08-19 13.59.19.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b063c6ee477443e4974f74e12d9141bd~tplv-k3u1fbpfcp-watermark.image?)

> 有一些同学说`Ctrl+S`无法自动格式化，可以在`settings.json`配置文件中加上这个配置:
> "editor.codeActionsOnSave": {
> "source.fixAll.eslint": true
> }

除了安装编辑器插件，我们也可以通过 `Vite` 插件的方式在开发阶段进行 `ESLint` 扫描，以命令行的方式展示出代码中的规范问题，并能够直接定位到原文件。

```javascript
pnpm add vite-plugin-eslint -D
```

在`vite.config.ts`中配置

```javascript
// vite.config.ts
import viteEslint from "vite-plugin-eslint";

{
  plugins: [viteEslint()];
}
```

### 3、Stylelint

`Stylelint`是专门对样式代码规范检查的，安装相关的依赖

```javascript
pnpm add stylelint stylelint-prettier stylelint-config-prettier stylelint-config-recess-order stylelint-config-standard stylelint-config-standard-less -D
```

在根目录下创建`.stylelintrc.cjs`

```javascript
// .stylelintrc.js
module.exports = {
  // 注册 stylelint 的 prettier 插件
  plugins: ["stylelint-prettier"],
  // 继承一系列规则集合
  extends: [
    // standard 规则集合
    "stylelint-config-standard",
    // standard 规则集合的 less 版本
    "stylelint-config-standard-less",
    // 样式属性顺序规则
    "stylelint-config-recess-order",
    // 接入 Prettier 规则
    "stylelint-config-prettier",
    "stylelint-prettier/recommended",
  ],
  // 配置 rules
  rules: {
    // 开启 Prettier 自动格式化功能
    "prettier/prettier": true,
  },
};
```

然后在`package.json`文件中添加对应的脚本

```json
// package.json
{
  "scripts": {
    // 整合 lint 命令
    "lint": "npm run lint:script && npm run lint:style",
    // stylelint 命令
    "lint:style": "stylelint --fix \"src/**/*.{css,less,vue}\""
  }
}
```

执行`pnpm run lint:style`会有一个警告提示

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9880e6cd4321449a84e5b4404c011986~tplv-k3u1fbpfcp-watermark.image?)

这是因为`stylelint`默认只会对`.css`文件进行校验，但是我们还想校验`.vue`文件中`<style lang="less"></style>`的样式代码的话，需要额外处理

安装支持`vue`的插件

```javascript
pnpm add stylelint-config-recommended-vue postcss-html postcss-less -D
```

在`.stylelintrc.cjs`中配置

```javascript
// .stylelintrc.cjs
{
    extends: [
        //忽略其他
        // vue 规则
        'stylelint-config-recommended-vue'
    ]，
    overrides: [
        {
          files: ['**/*.{html,vue}'],
          customSyntax: 'postcss-html'
        },
        {
          files: ['**/*.less'],
          customSyntax: 'postcss-less'
        }
    ]
}
```

再次执行`pnpn run lint:style`，此时不再报错或警告

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a2361270ae774da9bb76aa4445d0c2c8~tplv-k3u1fbpfcp-watermark.image?)

和`eslint`类似，`stylelint`也有相同得`vite`插件

```javascript
pnpm add vite-plugin-stylelint -D
```

在`vite.config.ts`中添加

```javascript
// vite.config.ts
import viteStylelint from "vite-plugin-stylelint";

{
  plugins: [
    // 省略其它插件
    viteStylelint(),
  ];
}
```

> 在`vite.config.ts`添加的`eslint`和`prettier`插件可以保证我们在`dev`或者`build`都会去校验代码

`eslint`和`stylelint`都可以添加`ignore`文件用于忽略某个目录或者文件的校验，在根目录下新建`.eslintignore`和`.stylelintignore`

```javascript
// .eslintignore
node_modules;
dist;
public;
publish;
pnpm - lock.yaml;
```

```javascript
// .stylelintignore
node_modules;
dist;
public;
publish;
pnpm - lock.yaml;
```

这里根据你们具体的项目情况填写即可

最后，输入`pnpm run lint`可以同时校验`js`和`css`了，那么代码规范的内容就讲到这。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fd4dfc10ea6f4dc1bdc6dd1724e211de~tplv-k3u1fbpfcp-watermark.image?)

> 正常来说，`Ctrl+S`是可以自动格式化样式代码的，如果不生效，和`eslint`的操作一样，可以在`settings.json`配置文件中加上这个配置:
>
> "editor.codeActionsOnSave": {
> "source.fixAll.stylelint": true
> }

OK，我们`代码规范`的内容就讲到这。

## 提交规范

上面提到的代码规范只能说是在开发阶段提前暴露问题，并不能保证把不规范的代码带到线上，因此我们需要在代码推送到远端时就要对代码进行校验，如果不符合规范，则不允许提交。社区中已经有了对应的工具——`Husky`来完成这件事情

### 1、初始化 git 仓库

首先需要先初始化一个`git`仓库，由于之前我把`vite`默认创建的`.git`目录给删掉了，所以重新再手动初始化一次

```javascript
git init
```

### 2、使用 husky 来校验提交的代码

安装依赖

```javascript
pnpm add husky -D
```

初始化`npx husky install`，并将  `husky install`作为项目启动前脚本

```json
// package.json
{
  "scripts": {
    // 会在安装 npm 依赖后自动执行
    "prepare": "husky install"
  }
}
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/beb4c7192f8f470692bf68b9b9bdfe0e~tplv-k3u1fbpfcp-watermark.image?)

添加 `husky` 钩子，在终端执行如下命令

```javascript
npx husky add .husky/pre-commit "npm run lint"
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8105d116a98b4689a51be76190ec0deb~tplv-k3u1fbpfcp-watermark.image?)

现在，当你执行  `git commit`  的时候，会首先执行  `npm run lint`脚本，通过`lint`检查后才会正式提交代码记录。不过呢，这样操作会对我们整个项目进行全量检测，也就是说，即使没有任何修改，也会走一次`lint`校验，这是没有必要的，而`lint-staged`就是用来解决上述全量扫描问题的，可以实现只对存入`暂存区`的文件进行`lint`检查，大大提高了提交代码的效率。

### 3、使用`lint-staged`来扫描暂存区的代码

```javascript
pnpm add lint-staged -D
```

然后在`package.json`中添加如下配置

```json
// package.json
{
  "script": {
     //省略其它
     "lint-staged": "lint-staged"
  },
  "lint-staged": {
    "**/*.{vue,js,ts}": [
      "eslint --fix"
    ],
    "**/*.{vue,css,less}": [
      "stylelint --fix
    ]
  }
}
```

接下来我们需要在`husky`中应用`lint-stage`，回到`.husky/pre-commit`脚本中，将原来的`npm run lint`换成如下脚本

```javascript
npx --no-install -- lint-staged
```

如此一来，我们便实现了提交代码时的`增量lint检查`

### 4、使用 commitlint 来规范化 git commit 信息

项目中规范`commit`信息也是非常有必要的，规范的 commit 信息能够方便团队协作和问题定位

安装依赖

```javascript
pnpm add commitlint @commitlint/cli @commitlint/config-conventional -D
```

项目根目录下新建`commitlint.config.cjs`

```javascript
// commitlint.config.cjs
module.exports = {
  extends: ["@commitlint/config-conventional"],
};
```

`@commitlint/config-conventional`规定了`commit`信息的一般有两个部分:`type`和`subject`

```javascript
// type 指提交的类型
// subject 指提交的摘要信息
<type>: <subject>
```

常用的  `type`  值包括如下:

- `feat`: 添加新功能。
- `fix`: 修复 Bug。
- `chore`: 一些不影响功能的更改。
- `docs`: 专指文档的修改。
- `perf`: 性能方面的优化。
- `refactor`: 代码重构。
- `test`: 添加一些测试代码等等。

将`commitlint`的功能集成到`husky`的钩子中

```javascript
npx husky add .husky/commit-msg "npx --no-install commitlint --edit $1"
```

会发现，在`.husky`目录中会多一个`commit-msg`文件

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6bded2d281bb4b9caa0556bb8114e4a0~tplv-k3u1fbpfcp-watermark.image?)

我们尝试一下提交文件，输入一个不符合`commitlint`规范的信息，可以提示我们需要按照规范输入
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d002d2402d51484e87f75c702ee76dbd~tplv-k3u1fbpfcp-watermark.image?)

需要输入正确的`commit`后可以提交成功

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7a68cff31fb04ee084e954ee0b7dd0db~tplv-k3u1fbpfcp-watermark.image?)

> 还可以通过`commitizen`来帮助我们提交`git commit`信息，可以看一下我另外一篇文章[《手把手教你如何使用 Commitizen 规范化提交代码》](https://juejin.cn/post/7098882036529643550?searchId=20230820224045A062C0C972E2604B3291)，这里就不过多赘述了，根据自己项目需要选择即可。

OK，我们`提交规范`的内容就讲到这。

## 处理静态资源

这一小节主要介绍如何在项目引入静态资源，例如图片、视频等。拿图片来说

在模板中引入

```javascript
<template>
  <div>
    <img src="./assets/imgs/vite.svg" />
  </div>
</template>
```

在`style`中引入

```less
.logo-img {
  background: url("./assets/imgs/vite.svg") no-repeat;
}
```

以上相对路径的方式，我们更多是通过`配置别名`，这样不仅可以在`script`中作为资源引入，更方便了我们的书写，不然你会看到很多地狱路径

```less
background: url("../../../../../assets/imgs/vite.svg");
```

#### 1、在 vite 中配置别名

```javascript
// vite.config.ts
import path from 'path';

{
  resolve: {
    // 别名配置
    alias: {
      '@assets': path.join(__dirname, './src/assets')
    }
  }
}
```

在`script`中引入

```javascript
// App.vue
<template>
  <div class="demo">
    <div class="demo-text">hello idux</div>
    <img :src="logoImg" alt="logo" width="300" />
  </div>
</template>

<script setup lang="ts">
import logoImg from '@assets/imgs/vite.svg';
</script>

<style lang="less" scoped>
.demo {
  &-text {
    color: @color;
    text-decoration: dashed;
  }
}
</style>
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/341d74f0112a4f599ffb7c197e8f53e4~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1080&h=850&e=png&b=ffffff)

在`style`中同样使用别名，注意这里就不能继续用`img`标签了

```javascript
// App.vue
<template>
  <div class="demo">
    <div class="demo-text">hello idux</div>
    <div class="logo-img"></div>
  </div>
</template>

<style lang="less" scoped>
.demo {
  &-text {
    color: @color;
    text-decoration: dashed;
  }

  .logo-img {
    width: 300px;
    height: 300px;
    background: url('@assets/imgs/vite.svg') no-repeat;
    background-size: cover;
  }
}
</style>
```

效果是一样的，这里就不放图了

#### 2、SVG 组件方式加载

上面案例用到了`svg`，业界有很多关于`svg`的最佳实践，这里介绍一种，把`svg`作为组件的方式引入。社区中已经有了对应的插件支持

Vue3 项目中可以引入  [vite-svg-loader](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fjpkleemans%2Fvite-svg-loader "https://github.com/jpkleemans/vite-svg-loader")

```javascript
pnpm add vite-svg-loader -D
```

在`vite`中添加插件

```javascript
// vite.config.ts
import svgLoader from "vite-svg-loader";

{
  plugins: [
    // 其它插件省略
    svgLoader(),
  ];
}
```

直接作为组件引入

```javascript
<template>
  <div class="demo">
    <div class="demo-text">hello idux</div>
    <logoImg />
  </div>
</template>

<script setup lang="ts">
import logoImg from '@assets/imgs/vite.svg';
</script>

<style lang="less" scoped>
.demo {
  &-text {
    color: @color;
    text-decoration: dashed;
  }
}
</style>
```

可以看到，目前组件渲染出来的就是一个`svg`标签

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/db30e34b82c34ebca739d88b3b422be2~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=2714&h=826&e=png&b=212225)

#### 3、生产环境处理

在前面的内容中，我们围绕着如何加载静态资源这个问题，在 `vite` 中进行具体的编码实践，相信对于 `vite` 中各种静态资源的使用你已经比较熟悉了。但另一方面，在生产环境下，我们又面临着一些新的问题。

- 部署域名怎么配置？
- 资源打包成单文件还是作为 `Base64` 格式内联?
- 图片太大了怎么压缩？

##### 3.1、自定义部署域名

一般在我们访问线上的站点时，站点里面一些静态资源的地址都包含了相应域名的前缀，如:

```html
<video
  src="https://idux-cdn.sangfor.com.cn/medias/home-banner.mp4"
  autoplay
  loop
>
  您的浏览器不支持 video 标签。
</video>
```

以上面这个地址例子，`https://idux-cdn.sangfor.com.cn`是 CDN 地址前缀，`/medias/home-banner.mp4`则是我们开发阶段使用的路径。那么，我们是不是需要在上线前把图片先上传到 CDN，然后将代码中的地址手动替换成线上地址呢？这样就太麻烦了！

在 Vite 中我们可以有更加自动化的方式来实现地址的替换，只需要在配置文件中指定`base`参数即可:

```javascript
// vite.config.ts
// 是否为生产环境，在生产环境一般会注入 NODE_ENV 这个环境变量，见下面的环境变量文件配置
const isProduction = process.env.NODE_ENV === "production";
// 填入项目的 CDN 域名地址
const CDN_URL = "https://idux-cdn.sangfor.com.cn";

{
  base: isProduction ? CDN_URL : "/";
}
```

根目录下新增`.env.development`和`.env.production`

```javascript
// .env.development
NODE_ENV = development;

// .env.production
NODE_ENV = production;
```

顾名思义，即分别在开发环境和生产环境注入一些环境变量，这里为了区分不同环境我们加上了`NODE_ENV`，你也可以根据需要添加别的环境变量。

> 打包的时候 `vite` 会自动将这些环境变量替换为相应的字符串。

在项目中引入资源

```javascript
<template>
  <div class="demo">
    <div class="demo-text">hello idux</div>
    <logoImg />
    <div>
      <video width="500" :src="bannerVideo" autoplay loop>
        您的浏览器不支持 video 标签。
      </video>
    </div>
  </div>
</template>

<script setup lang="ts">
import logoImg from '@assets/imgs/vite.svg';
import bannerVideo from '@assets/medias/home-banner.mp4';
</script>

<style lang="less" scoped>
.demo {
  &-text {
    color: @color;
    text-decoration: dashed;
  }
}
</style>
```

可以正常展示
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9921abcd64ef4d47b77bcdbb19b7beec~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=2562&h=824&s=368685&e=png&b=fbfbfb)

我们执行`pnpm run build`，可以看到产物中已经自动加上了`CDN`地址前缀了

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/93be7a0b96b3496fa55f7b1962f5b771~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=3066&h=696&s=342673&e=png&b=1e1e1e)

当然，`HTML` 中的一些 `JS`、`CSS` 资源链接也一起加上了 `CDN` 地址前缀

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0cb97f3610d74cad83312d70595bb035~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=3430&h=1066&s=310490&e=png&b=1e1e1e)
除了`CDN`之外，有时候可能项目中的某些图片需要存放到另外的存储服务，一种直接的方案是将完整地址写死到 `src属性` 中，如:

```html
<img src="https://my-image-cdn.com/logo.png" />
```

这样做显然是不太优雅的，我们可以通过定义环境变量的方式来解决这个问题，在项目根目录新增`.env`文件:

```javascript
// .env
VITE_IMG_BASE_URL=https://idux.site
```

> 开发环境优先级: `.env.development` > `.env`
>
> 生产环境优先级: `.env.production` > `.env`

然后进入  `src/vite-env.d.ts`增加类型声明:

```typescript
/// <reference types="vite/client" />

interface ImportMetaEnv {
  // 自定义的环境变量
  readonly VITE_IMG_BASE_URL: string;
}

interface ImportMeta {
  readonly env: ImportMetaEnv;
}
```

> 如果某个环境变量要在 `vite` 中通过  `import.meta.env`  访问，那么它必须以`VITE_`开头，如`VITE_IMG_BASE_URL`

我们在项目中增加一个`img`

```javascript
<template>
  <div class="demo">
    <div class="demo-text">hello idux</div>
    <logoImg />
    <div>
      <video width="500" :src="bannerVideo" autoplay loop>
        您的浏览器不支持 video 标签。
      </video>
    </div>
    <img :src="iduxLogoUrl" alt="idux logo" width="200" />
  </div>
</template>

<script setup lang="ts">
import logoImg from '@assets/imgs/vite.svg';
import bannerVideo from '@assets/medias/home-banner.mp4';
// 通过import.meta.env获取env环境变量
const iduxLogoUrl = `${import.meta.env.VITE_IMG_BASE_URL}/icons/logo.svg`;
</script>

<style lang="less" scoped>
.demo {
  &-text {
    color: @color;
    text-decoration: dashed;
  }
}
</style>
```

可以看到，`img`路径已经成功被替换成`env`中的路径

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ba855eb92110449e80d38dc8f57a73af~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=2934&h=970&s=397003&e=png&b=212225)

至此，我们就解决了生成环境下域名替换的问题。

##### 3.2、单文件 or 内联

`vite` 中内置的优化方案是下面这样的:

- 如果静态资源体积 `>= 4KB`，则提取成单独的文件
- 如果静态资源体积 `< 4KB`，则作为 `base64` 格式的字符串内联

上述的`4 KB`即为提取成单文件的临界值，当然，这个临界值你可以通过`build.assetsInlineLimit`自行配置，如下代码所示:

```javascript
// vite.config.ts
{
  build: {
    // 8 KB
    assetsInlineLimit: 8 * 1024;
  }
}
```

> `svg` 格式的文件不受这个临时值的影响，始终会打包成单独的文件

##### 3.3、图片压缩

社区中已经有了成熟的图片压缩方案：`vite-plugin-imagemin`

安装依赖

```javascript
pnpm add vite-plugin-imagemin -D
```

> 如果安装不上可以参考这篇文章：https://blog.csdn.net/qq_43806604/article/details/124732352

然后在`vite.config.ts`中配置

```javascript
//vite.config.ts
import viteImagemin from "vite-plugin-imagemin";
{
  plugins: [
    // 忽略前面的插件
    viteImagemin({
      // 无损压缩配置，无损压缩下图片质量不会变差
      optipng: {
        optimizationLevel: 7,
      },
      // 有损压缩配置，有损压缩下图片质量可能会变差
      pngquant: {
        quality: [0.8, 0.9],
      },
      // svg 优化
      svgo: {
        plugins: [
          {
            name: "removeViewBox",
          },
          {
            name: "removeEmptyAttrs",
            active: false,
          },
        ],
      },
    }),
  ];
}
```

为了测试我特意找了张`.png`格式的图片

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ad62aab75a9b4f639a968e714df29279~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=3640&h=1912&s=584309&e=png&b=1e1e1e)

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/528d6e4df5fb47ad8c4c1da28e4737b4~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=2698&h=1214&s=452082&e=png&b=fdfdfd)

这张图片大小是`10.4KB`，按照之前配置的应该会打包成`单文件`

使用之前`vite-plugin-imagemin`进行压缩之前：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c90747bff2b24c26b92755898d880cd8~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=3004&h=840&s=203258&e=png&b=181818)

使用之前`vite-plugin-imagemin`进行压缩之后：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/25d9a1bb1a3e4c9a836f5d80668a8980~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=2944&h=1038&s=234991&e=png&b=181818)

可以看到效果非常明显，这对我们的性能有很大提升

OK，我们`处理静态资源`的内容就讲到这。

### TSX

写过组件库的都知道，`tsx`在语法上会比较有优势，我们把它接入进来

安装依赖

```javascript
pnpm add @vitejs/plugin-vue-jsx -D
```

使用插件

```javascript
// vite.config.ts
import vueJsx from '@vitejs/plugin-vue-jsx';

{
    "plugins": [
        //其他配置
        vueJsx()
    ]
}
```

补充`lint`命令支持后缀

```json
{
  "script": {
    "lint:script": "eslint --ext .js,.ts,.tsx,.vue --fix --quiet ./src"
  }
}
```

编写组件需要以`.tsx`结尾

```javascript
// src/componens/Demo.tsx
import { defineComponent } from "vue";

export default defineComponent({
  setup() {
    return () => <div>hello idux</div>;
  },
});
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3e502708de8a4b3bb762fb028e370dfe~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=400&h=196&s=3645&e=png&b=ffffff)

OK, 这里可以顺利执行。

### Pinia 状态管理

大型项目中一般有状态管理的需求，我们首当其冲的使用：[pinia](https://pinia.vuejs.org/zh/)

安装依赖

```javascript
pnpm add pinia pinia-plugin-persist
```

> 插件`pinia-plugin-persist`支持数据持久化（将`store`的数据缓存到`storage`中）

创建别名

```javascript
// vite.config.ts

{
    resolve: {
        alias: {
          // 其他配置
          '@store/*': path.resolve(__dirname, './src/store')
        }
    }
}
```

配置`tsconfig.json`防止类型报错

```json
{
  "compilerOptions": {
    // 其他配置
    "types": ["pinia-plugin-persist"],
    "baseUrl": ".",
    "paths": {
      "@store/*": ["./src/store/*"]
    },
    "strict": false
  }
}
```

创建`store`实例

```javascript
// src/store/index.ts
import { createPinia } from "pinia";
import { App } from "vue";
import piniaPersist from "pinia-plugin-persist"; //数据持久化

const store = createPinia();

const install = (app: App): void => {
  store.use(piniaPersist);

  app.use(store);
};
export default { install };
```

注册`store`

```javascript
// src/main.ts
import { createApp } from "vue";
import store from "./store";
import App from "./App.vue";

createApp(App).use(store).mount("#app");
```

创建`useCountStore`

```javascript
// src/store/use_count_store.ts
import { defineStore } from "pinia";
import { ref } from "vue";

export const useCountStore = defineStore("count", () => {
  const count = ref(0);
  const increment = () => {
    count.value++;
  };
  return {
    count,
    increment,
  };
});
```

在`src/components`文件夹下创建`Demo.vue`并在`App.vue`中引入，用来测试状态是否共享

```javascript
// App.vue
<template>
  <div class="demo">
    <div class="demo-text">hello idux</div>
    <br />
    <logoImg />
    <br />
    <video width="500" :src="bannerVideo" autoplay loop>
      您的浏览器不支持 video 标签。
    </video>
    <br />
    <img :src="iduxLogoUrl" alt="idux logo" width="200" />
    <br />
    <img src="@assets/imgs/vue3.png" alt="vue3.x" />
    <br />
    <div>
      App Count: {{ count }}
      <button @click="increment">新增</button>
    </div>
    <br />
    <DemoCmp />
  </div>
</template>

<script setup lang="ts">
import logoImg from '@assets/imgs/vite.svg';
import bannerVideo from '@assets/medias/home-banner.mp4';
import { useCountStore } from '@store/use_count_store';
import { storeToRefs } from 'pinia';
import DemoCmp from './components/Demo.vue';
// 通过import.meta.env获取env环境变量
const iduxLogoUrl = `${import.meta.env.VITE_IMG_BASE_URL}/icons/logo.svg`;

const store = useCountStore();
const { increment } = store;
const { count } = storeToRefs(store);
</script>

<style lang="less" scoped>
.demo {
  &-text {
    color: @color;
    text-decoration: dashed;
  }
}
</style>

// Demo.vue
<template>
  Demo Count: {{ count }}
  <button @click="increment">新增</button>
</template>

<script setup lang="ts">
import { useCountStore } from '@store/use_count_store';
import { storeToRefs } from 'pinia';
const store = useCountStore();
const { increment } = store;
const { count } = storeToRefs(store);
</script>
```

可以看到`Demo.vue`组件会有一个报错：组件文件名必须驼峰命名，解决方案：

- 修改组件名为驼峰格式，例如：`DemoComponent.vue`或者配置`name: DemoComponent`
- 配置`eslint`规则，增加：`vue/multi-word-component-names: off`的规则

执行项目，可以看到状态已经同步，配合`vue-devtools`效果更佳

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ed4f97d4c4f74ca59c6ea08fbf82a6d2~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=3824&h=1474&s=518577&e=png&b=0b1016)

`pinia`持久化的配置这里就不过多介绍了，网上已经有很多详细的介绍方案。

OK，`pinia`的内容就讲到这。

### Vue-Router

接下来，我们要接入`router`路由

安装依赖

```javascript
pnpm add vue-router@latest
```

创建路由实例

```javascript
// src/router/index.ts
import type { App } from "vue";
import { createRouter, createWebHashHistory } from "vue-router";
import routes from "./routes";

const router = createRouter({
  routes,
  history: createWebHashHistory(),
});

const install = (app: App): void => {
  app.use(router);
};

export default { install };
```

创建路由，这里我们随意创建个首页和登录页的路由作为案例

```javascript
// src/router/routes.ts
import type { RouteRecordRaw } from "vue-router";

const routes: RouteRecordRaw[] = [
  {
    path: "/",
    redirect: "/home",
  },
  {
    path: "/home",
    component: () => import("@views/Home/Index.vue"), //在vite.config.ts中配置@views别名
    meta: {
      title: "首页",
    },
  },
  {
    path: "/login",
    component: () => import("@views/Login/Index.vue"),
    meta: {
      title: "登录",
    },
  },
];

export default routes;
```

创建`Home`和`Login`组件

```javascript
// src/components/Home/Index.vue
<template>
  <div>首页</div>
</template>

// src/components/Login/Index.vue
<template>
  <div>登录页</div>
</template>
```

注册路由

```javascript
// src/main.ts
import { createApp } from "vue";
import store from "./store";
import router from "./router";
import App from "./App.vue";

createApp(App).use(store).use(router).mount("#app");
```

修改`tsconfig.json`防止类型报错

```json
{
  "compilerOptions": {
    // 其他配置
    "moduleResolution": "node"
  }
}
```

在`App.vue`添加`<RouterView />`组件，并添加跳转按钮测试

```javascript
<template>
  <div class="demo">
    <div class="demo-text">hello idux</div>
    <br />
    <logoImg />
    <br />
    <video width="500" :src="bannerVideo" autoplay loop>
      您的浏览器不支持 video 标签。
    </video>
    <br />
    <img :src="iduxLogoUrl" alt="idux logo" width="200" />
    <br />
    <img src="@assets/imgs/vue3.png" alt="vue3.x" />
    <br />
    <div>
      App Count: {{ count }}
      <button @click="increment">新增</button>
    </div>
    <br />
    <DemoCmp />
    <br />
    <div style="margin-top: 30px">
      测试路由
      <button @click="goHome">去首页</button>
      <button @click="goLogin">去登录页</button>
      <RouterView />
    </div>
  </div>
</template>

<script setup lang="ts">
import { RouterView, useRouter } from 'vue-router';
import logoImg from '@assets/imgs/vite.svg';
import bannerVideo from '@assets/medias/home-banner.mp4';
import { useCountStore } from '@store/use_count_store';
import { storeToRefs } from 'pinia';
import DemoCmp from './components/DemoComponents.vue';
// 通过import.meta.env获取env环境变量
const iduxLogoUrl = `${import.meta.env.VITE_IMG_BASE_URL}/icons/logo.svg`;

const store = useCountStore();
const { increment } = store;
const { count } = storeToRefs(store);

const router = useRouter();

const goHome = () => {
  router.push({
    path: '/home'
  });
};
const goLogin = () => {
  router.push({
    path: '/login'
  });
};
</script>

<style lang="less" scoped>
.demo {
  &-text {
    color: @color;
    text-decoration: dashed;
  }
}
</style>
```

运行项目看一下效果，可以看到已经可以实现路由的跳转了
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/16db9e4e899c4d639a5b9287a4c2da9e~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=2914&h=1626&s=370825&e=png&b=ffffff)

OK, 关于路由的介绍就到这里。

### Idux 组件库

前期基础工作都准备的差不多了，我们可以开始接入组件库了

#### 1、初始化 Idux

安装依赖

```javascript
pnpm add @idux/cdk @idux/components @idux/pro
```

引入`idux`

```javascript
// src/plugins/idux.ts
import type { App } from "vue";

import "@idux/components/default.full.css";
import "@idux/pro/default.css";

// 如果不需要 reset 全局样式和滚动条样式，移除下面 2 行代码
import "@idux/components/style/core/reset.default.css";
import "@idux/components/style/core/reset-scroll.default.css";

import IduxCdk from "@idux/cdk";
import IduxComponents from "@idux/components";
import IduxPro from "@idux/pro";

import { createGlobalConfig } from "@idux/components/config";

// 动态加载图标：不会被打包，可以减小包体积，需要加载的时候时候 http 请求加载
const loadIconDynamically = (iconName: string) => {
  return fetch(`/idux-icons/${iconName}.svg`).then((res) => res.text());
};

const globalConfig = createGlobalConfig({
  // 默认为中文，可以打开注释设置为其他语言
  // locale: enUS,
  icon: { loadIconDynamically },
});

const install = (app: App): void => {
  app.use(IduxCdk).use(IduxComponents).use(IduxPro).use(globalConfig);
};

export default { install };
```

动态加载图标，需要安装`vite-plugin-static-copy`

```javascript
pnpm add vite-plugin-static-copy -D
```

使用插件

```javascript
// vite.config.ts
import { viteStaticCopy } from 'vite-plugin-static-copy';

{
    "plugins": [
        //其他配置
        viteStaticCopy({
          targets: [
            {
              src: './node_modules/@idux/components/icon/assets/*.svg',
              dest: 'idux-icons'
            }
          ]
        })
    ]
}
```

这样打包的时候，`svg`就作为单独的资源文件放在`idux-icons`目录，而不是打包进`js`文件中

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/00e6846707d74d43a1378ebcfba9ee5c~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=722&h=218&s=14978&e=png&b=1e1e1e)

导出`idux`

```javascript
// src/plugins/index.ts
import idux from "./idux";

export { idux };
```

注册`idux`

```javascript
// src/main.ts
import { createApp } from "vue";
import store from "./store";
import { idux } from "./plugins";
import router from "./router";
import App from "./App.vue";

createApp(App).use(store).use(router).use(idux).mount("#app");
```

处理组件类型，创建`types`目录并新建`idux.d.ts`，同时把原来的`vite-env.d.ts`移动到该目录下

```typescript
// idux.d.ts
/// <reference types="@idux/cdk/types" />
/// <reference types="@idux/components/types" />
/// <reference types="@idux/pro/types" />
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c716f9bca62342a58161f5466aac952e~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=2570&h=950&s=157066&e=png&b=1d1d1d)

#### 2、创建`IduxProvider`组件和 utils

`IduxProvider`方便我们全局注入`provider`，可以使用`idux`提供的`hooks`函数

```javascript
// src/components/idux-provider/IduxProvider.vue
<template>
  <IxDrawerProvider ref="drawerProviderRef">
    <IxNotificationProvider>
      <IxModalProvider ref="modalProviderRef">
        <IxMessageProvider>
          <IduxProviderRegister></IduxProviderRegister>
          <slot></slot>
        </IxMessageProvider>
      </IxModalProvider>
    </IxNotificationProvider>
  </IxDrawerProvider>
</template>

<script setup lang="ts">
import { type DrawerProviderInstance } from '@idux/components/drawer';
import { type ModalProviderInstance } from '@idux/components/modal';

import { onMounted, ref } from 'vue';
import { useRouter } from 'vue-router';
import IduxProviderRegister from './IduxProviderRegister.vue';

const drawerProviderRef = ref<DrawerProviderInstance>();
const modalProviderRef = ref<ModalProviderInstance>();

const router = useRouter();

onMounted(() => {
  // 每次路由切换时销毁当前的抽屉和弹窗（仅对通过 useDrawer/useModal 创建的生效）
  router.afterEach(() => {
    drawerProviderRef.value!.destroyAll();
    modalProviderRef.value!.destroyAll();
  });
});
</script>
```

```javascript
<script setup lang="ts">
// src/components/idux-provider/IduxProviderRegister.vue
import { useDrawer } from '@idux/components/drawer';
import { useMessage } from '@idux/components/message';
import { useModal } from '@idux/components/modal';
import { useNotification } from '@idux/components/notification';

import { registerProviders } from '@utils'; //需要在vite.config.ts中配置@utils别名

const drawer = useDrawer();
const notification = useNotification();
const modal = useModal();
const message = useMessage();

registerProviders({ drawer, notification, modal, message });
</script>

<!-- eslint-disable-next-line vue/valid-template-root -->
<template></template>
```

因为上面的`useDrawer`等`hooks`需要在`setup`中才能使用，所以我们可以把实例存在全局的变量中，这样在`ts`文件也可以使用

```javascript
// src/utils/iduxProviders.ts
import { DrawerProviderRef } from "@idux/components/drawer";
import { NotificationProviderRef } from "@idux/components/notification";
import { ModalProviderRef } from "@idux/components/modal";
import { MessageProviderRef } from "@idux/components/message";

let Drawer: DrawerProviderRef | undefined;
let Notification: NotificationProviderRef | undefined;
let Modal: ModalProviderRef | undefined;
let Message: MessageProviderRef | undefined;

// 方便在 ts 中直接调用
export function registerProviders(option: {
  drawer: DrawerProviderRef,
  notification: NotificationProviderRef,
  modal: ModalProviderRef,
  message: MessageProviderRef,
}): void {
  Drawer = option.drawer;
  Notification = option.notification;
  Modal = option.modal;
  Message = option.message;
}

export { Drawer, Notification, Modal, Message };
```

导出`组件`和`utils`

```javascript
// src/components/idux-provider/index.ts
import IduxProvider from "./IduxProvider.vue";

export { IduxProvider };
```

```javascript
// src/utils/index.ts
export * from "./iduxProviders";
```

在`App.vue`中引入
我们把之前的测试代码全都删掉，保持项目的干净，并引入`IduxProvider`组件

```javascript
<template>
  <IduxProvider>
    <router-view />
  </IduxProvider>
</template>

<script setup lang="ts">
//需要在vite.config.ts中配置@componrnts别名
import { IduxProvider } from '@components/idux-provider';
import { RouterView } from 'vue-router';
</script>
```

我们去`Home组件`中引入`idux`组件，试一下效果

```javascript
// src/components/Home/Index.vue
<template>
  <div>首页</div>
  <IxButton mode="primary">Hello Idux</IxButton>
</template>

<script setup lang="ts">
import { defineComponent } from 'vue';
import { IxButton } from '@idux/components/button';
</script>
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/597efe9dc2604dfcb30dfab353fafdb4~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=2098&h=1014&s=177777&e=png&b=0c1218)

可以看到已经可以引入组件了，我们再试试`useMessage`等`hooks`的使用

```javascript
// src/components/Home/Index.vue
<template>
  <div>首页</div>
  <IxButton mode="primary">Hello Idux</IxButton>
</template>

<script setup lang="ts">
import { IxButton } from '@idux/components/button';
import { useMessage } from '@idux/components/message';

const { info, success, warning, error, loading } = useMessage();
info('hello idux');
success('hello idux');
warning('hello idux');
error('hello idux');
loading('hello idux');
</script>
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b1d6388c3bc1415a8bec1e13ba2078c2~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1902&h=1024&s=63701&e=png&b=fefefe)

可以成功使用了，我们在试试直接在`ts`文件中使用

```javascript
// src/components/Home/hooks.ts
import { Message } from "@utils";

const useIduxMessage = (): void => {
  Message?.success("hello, idux");
};

export { useIduxMessage };
```

在`Home组件`引入

```javascript
<template>
  <div>首页</div>
  <IxButton mode="primary">Hello Idux</IxButton>
</template>

<script setup lang="ts">
import { IxButton } from '@idux/components/button';
import { useIduxMessage } from './hooks';

useIduxMessage();
</script>
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e44963a75fb9400bb39d22941b0aed10~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1898&h=986&s=35551&e=png&b=ffffff)

看一下效果，也是可以提示的

> 在`setup`中可以使用`useMessage`等`hooks`函数，因为这是依赖于`provider/inject`的注入，这就是`IduxProvider`组件的作用，而在`ts`文件中使用的话会报错，所以我们通过挂载全局变量的方式使用，达到同样的效果

#### 3、通过 unplugin-vue-components 按需加载

在上面例子中，我们可以看到组件是通过`手动引入`的方式

```javascript
import { IxButton } from "@idux/components/button";
```

如果使用的组件一多，那需要写特别多的`import`。而且我们前面注册`idux`时，直接`use`了整个组件库，为了降低打包的文件体积，我们使用`按需加载`的方式使用组件

安装依赖

```javascript
pnpm add unplugin-vue-components -D
```

添加插件

```javascript
// vite.config.ts
import { IduxResolver } from 'unplugin-vue-components/resolvers';
import Components from 'unplugin-vue-components/vite';

{
    "plugins": [
        //其他配置
        Components({
          resolvers: [
            // 可以通过指定 `importStyle` 来按需加载 css 或 less 代码, 也支持不同的主题
            IduxResolver({ importStyle: 'css', importStyleTheme: 'default' })
          ],
          dts: false //不生成d.ts文件
        })
    ]
}
```

移除注册代码

```javascript
// src/plugins/idux.ts
// 移除
- import "@idux/components/default.full.css";
- import "@idux/pro/default.css";
// 新增
+ import '@idux/cdk/index.css'
// 移除
- import IduxCdk from "@idux/cdk";
- import IduxComponents from "@idux/components";
- import IduxPro from "@idux/pro";

const install = (app: App): void => {
    // 移除
    - app.use(IduxCdk).use(IduxComponents).use(IduxPro).use(globalConfig);
    // 新增
    + app.use(globalConfig)
};
```

完整代码如下

```javascript
// src/plugins/idux.ts
import type { App } from "vue";

import "@idux/cdk/index.css";
// 如果不需要 reset 全局样式和滚动条样式，移除下面 2 行代码
import "@idux/components/style/core/reset.default.css";
import "@idux/components/style/core/reset-scroll.default.css";

import { createGlobalConfig } from "@idux/components/config";
import {
  IDUX_ICON_DEPENDENCIES,
  addIconDefinitions,
} from "@idux/components/icon";
// import { enUS } from "@idux/components/locales";

// 静态加载: `IDUX_ICON_DEPENDENCIES` 是 `@idux` 的部分组件默认所使用到图标，建议在此时静态引入。
addIconDefinitions(IDUX_ICON_DEPENDENCIES);

// 动态加载：不会被打包，可以减小包体积，需要加载的时候时候 http 请求加载
// 注意：请确认图标的 svg 资源被正确放入到 `public/idux-icons` 目录中
const loadIconDynamically = (iconName: string) => {
  return fetch(`/idux-icons/${iconName}.svg`).then((res) => res.text());
};

const globalConfig = createGlobalConfig({
  // 默认为中文，可以打开注释设置为其他语言
  // locale: enUS,
  icon: { loadIconDynamically },
});

const install = (app: App): void => {
  app.use(globalConfig);
};

export default { install };
```

`首页组件`也移除`import`的相关代码

```javascript
<template>
  <div>首页</div>
  <IxButton mode="primary">Hello Idux</IxButton>
</template>

<script setup lang="ts">
import { useIduxMessage } from './hooks';

useIduxMessage();
</script>
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/debe5dc11a674f40a13876a5b2bc739c~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1750&h=862&s=32677&e=png&b=ffffff)

可以看到，已经可以成功实现按需加载了

OK，`idux`组件库的内容就讲到这。

### Axios

接下来，我们准备接入`axios`，用来发请求
安装依赖

```javascript
pnpm add axios
```

业界有很多二次封装`axios`的案例，这里就不费尽口舌了，仅介绍一些整体框架和思路

添加`VITE_BASE_API_URL`环境变量

```javascript
// .env
VITE_IMG_BASE_URL=https://idux.site
VITE_BASE_API_URL=http://xxxxx //填写你要请求的url地址
```

添加到类型中

```javascript
// src/types/vite.env.d.ts

/// <reference types="vite/client" />

interface ImportMetaEnv {
  // 自定义的环境变量
  readonly VITE_IMG_BASE_URL: string;
  readonly VITE_BASE_API_URL: string;
}

interface ImportMeta {
  readonly env: ImportMetaEnv;
}
```

添加`showErrorMessage`方法，当请求错误时用来提示

```javascript
// src/utils/requeset.ts

// 请求错误时消息提示
import type { AxiosError } from "axios";
import { Message } from "./iduxProviders";

const showErrorMessgae = (error: AxiosError): void => {
  const { response } = error;
  if (!response) {
    return;
  }
  const { status } = response;
  switch (status) {
    case 401:
      Message?.warning("账号没有权限，无法访问资源");
      break;
    case 403:
      Message?.warning("请求的资源无法访问");
      break;
    case 404:
      Message?.error("请求的资源不存在");
      break;
    case 500:
      Message?.error("网络错误，请重试");
      break;
    default:
      Message?.error("网络错误，请重试");
      break;
  }
};

export { showErrorMessgae };
```

创建`axios`实例

```javascript
// src/request/index.ts
import axios, {
  type AxiosRequestConfig,
  type AxiosError,
  type AxiosResponse,
} from "axios";
import { requestInterceptors, responseiInterceptors } from "./interceptors";

const request = axios.create({
  baseURL: import.meta.env.VITE_BASE_API_URL,
  timeout: 1000 * 10, //10s超时
});

// 请求拦截器
requestInterceptors?.forEach((interceptors) => {
  request.interceptors.request.use(
    (config: AxiosRequestConfig) => {
      return interceptors?.resolve(config);
    },
    (error: AxiosError) => {
      return interceptors?.reject(error);
    }
  );
});

// 响应拦截器
responseiInterceptors?.forEach((interceptors) => {
  request.interceptors.response.use(
    (response: AxiosResponse) => {
      return interceptors?.resolve(response);
    },
    (error: AxiosError) => {
      return interceptors?.reject(error);
    }
  );
});

export default request;
```

创建`errorStatus`拦截器，专门处理错误状态码的

```javascript
// src/request/interceptors/response/errorStatus.ts

// 专门处理失败状态码的拦截器
import type { AxiosError } from "axios";
import { showErrorMessgae } from "@utils";

const errorStatusReject = (error: AxiosError): Promise<AxiosError> => {
  showErrorMessgae(error);
  return Promise.reject(error);
};

export default errorStatusReject;
```

整体层级结构

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/532a7f362d1b4014b18ac34c5e1f5ef8~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=3424&h=1170&s=302475&e=png&b=1d1d1d)

创建`mock`

```json
// src/components/Home/mock.json

{
  "data": {
    "list": [
      {
        "key": 1,
        "name": "张三",
        "age": 18
      },
      {
        "key": 2,
        "name": "李四",
        "age": 30
      }
    ]
  },
  "success": true
}
```

在`首页组件`中发请求

```javascript
// src/components/Home/Index.vue
<template>
  <div>首页</div>
  <IxButton mode="primary" @click="fetch">Hello Idux</IxButton>
</template>

<script setup lang="ts">
import { useIduxMessage } from './hooks';
import request from '@/request';

useIduxMessage();

const fetch = () => {
  request.get('./mock.json').then((res) => {
    console.log(res);
  });
};
</script>
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2a0c7aa747334c23b8481b68b5f034f8~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=3158&h=1330&s=282062&e=png&b=222326)

可以看到，已经可以成功提示错误消息了，但是为什么会`404`呢？我们需要设置代理等，社区有成熟的`mock`方案，那就是[vite-plugin-mock](https://github.com/vbenjs/vite-plugin-mock/tree/main)

安装依赖

```javascript
pnpm add vite-plugin-mock@2.9.8 mockjs -D
```

> `vite-plugin-mock`已经升级到`3.0`版本，但是好像会报错，所以建议安装`2.9.8`

根目录下创建`mock`文件夹和`demo.js`

```javascript
// mock/demo.js
export default [
  {
    url: "/api/demo",
    method: "get",
    response: () => {
      return {
        data: {
          "list|0-10": [
            {
              key: "@id",
              name: "@cname",
              age: "@integer(18, 60)",
            },
          ],
          total: "@integer(0, 100)",
        },
        success: true,
      };
    },
  },
];
```

使用插件

```javascript
// vite.config.ts
import { viteMockServe } from 'vite-plugin-mock';
{
    "plugins": [
        //其他配置
        viteMockServe({
          mockPath: 'mock',
          localEnabled: !isProduction, //生产环境下不启用
          watchFiles: true
        }),
    ]
}
```

修改请求路径

```javascript
// src/components/Home/Index.vue
<template>
  <div>首页</div>
  <IxButton mode="primary" @click="fetch">Hello Idux</IxButton>
</template>

<script setup lang="ts">
import { useIduxMessage } from './hooks';
import request from '@/request';

useIduxMessage();

const fetch = () => {
  request.get('/api/demo').then((res) => {
    console.log(res);
  });
};
</script>
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1f323c1251d344ab8ee3637f5482de59~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=2606&h=1188&s=330453&e=png&b=212225)

可以看到，已经可以成功请求`mock`了

模拟`403`，也可以正确提示

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/167420f3c6d34968ba18bbe5da15b2de~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=3414&h=1714&s=590073&e=png&b=ffffff)

但是，这里还有个小问题，我们每次请求都去引入一次`request`

```javascript
import request from "@/request";
```

那这样每次都会重复去创建一次实例

```javascript
const request = axios.create();

export default request;
```

这个过程似乎是没有必要的，提供个思路，我们同样可以放在全局变量上，只初始化一次，每次引入`request`的全局变量即可

```javascript
// src/request/index.ts
import type { AxiosInstance } from "axios";

let request: AxiosInstance;

const initAxios = (curRequest: AxiosInstance) => {
  request = curRequest;
};

export { request, initAxios };
```

在`main.ts`中注入

```javascript
// src/main.ts
import { createApp } from "vue";
import store from "./store";
import { idux } from "./plugins";
import router from "./router";
import request from "./request/create"; //修改原来的request.ts变为create.ts
import { initAxios } from "./request";
import App from "./App.vue";

initAxios(request);

createApp(App).use(store).use(router).use(idux).mount("#app");
```

请求时直接引入即可，避免多次创建

```javascript
import { request } from "@/request";
```

OK, `axios`相关的内容就讲到这。

### 使用 Vitest 进行测试

安装依赖

```javascript
pnpm add vitest -D
```

我们直接使用官方的[案例](案例)

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b927255462064109b4a25ac0c1308a80~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=3450&h=1450&s=245463&e=png&b=1d1d1d)

添加命令

```json
// package.json
{
  "script": {
    //其他配置
    "test": "vitest"
  }
}
```

运行`pnpm run test`

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f515d5b83607444d996466b401b21bb9~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=2726&h=1592&s=272105&e=png&b=1b1b1b)

看到输出结果说明已经成功接入`vitest`

### 总结

经过上面的流程，我们已经把整个架构都搭好了，可以根据自己需求继续完善即可。

我们专门建立了`@idux`的企微群，如果有任何组件库相关的疑问都可以扫描进群询问

![IMG_1534.JPG](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2645156f88334fe19ccd602e2274023c~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1125&h=1846&s=181675&e=jpg&b=ffffff)

> 项目代码位置：https://github.com/fengxiaodong28/idux-vue-setup
