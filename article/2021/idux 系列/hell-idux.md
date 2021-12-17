# Hello @idux, 一个全新的 vue 3.x 组件库来了

`@idux` 是一个基于 `Vue 3.x` 和 `TypeScript` 开发的开源组件库，它拥有以下特性:

- Monorepo 管理模式：`cdk`, `components`, `pro`
- 开箱即用的高质量组件
- 全面拥抱 Composition API，从源码到文档
- 完全使用 TypeScript 开发，提供完整的类型定义
- 灵活的全局配置
- 深入细节的主题定制能力
- 国际化语言支持

相关链接：

- Github: [IDuxFE/idux](https://github.com/IDuxFE/idux)
- 官方文档: [idux.site](https://idux.site)

## 特性

### Monorepo 模式

`@idux` 总共分成了 3 个包:

- `@idux/cdk`
  - 它是一组用于构建 UI 组件库的开发套件
  - 它主要包含了一些 UI 无关或者轻 UI 的功能
  - 其灵感来自于 [@angular/cdk](https://material.angular.cn/cdk)
- `@idux/components`
  - 它是传统意义上的基础组件库，包含了数十种常规 UI 组件
  - 我们对每一个组件的 API 都进行了精心的设计，保证了 API 一致性
  - 组件设计参考了 [antd](https://ant.design/components/overview-cn/) 和 [ng-zorro](https://ng.ant.design/docs/introduce/zh)
- `@idux/pro`
  - 它是对 `@idux/components` 的进一步封装与组合，使用更少的代码去完成业务开发
  - 它主要来源于我们对自身业务的提炼
  - 它目前数量有限，我们会逐渐补充

### 开箱即用

完全基于 `esm` 构建的包，支持开箱即用的 Tree Shaking 和按需加载，也可以再配合一些自动导入插件，可以让你的编码更加高效

```ts
import { IxButton } from "@idux/components/button";
```

更多内容请参见[快速上手](https://idux.site/docs/getting-started/zh)

我们有覆盖率 `80+` 的单元测试和完善的 CI 流程，来保证我们的组件质量

### Composition API

`Vue 3.x` 的一个重要特性就是 Composition API，我们所有的源代码和示例代码，都使用它来完成

它其实是一种全新的代码组织范式，让开发者可以更好的组织代码，越是复杂的组件，就越能体现它的优雅

![table](https://cdn.jsdelivr.net/gh/danranVm/image-hosting/images/table.png)

### TypeScript

我们所有的源代码都是由 `TypeScript` + `TSX` 编写，为了给用户提供最佳的使用体验，我们为此付出了诸多努力

无论用户使用 `TSX` 还是使用 `SFC` (需要配合 `Vue Language Features` 插件)都可以享受到完整的类型提示与保护

![button](https://cdn.jsdelivr.net/gh/danranVm/image-hosting/images/button.png)

### 国际化

我们针对部分组件的静态文案内置了 `i18n` 的支持，不过目前仅支持中文和英文

PR Welcome, 我们也渴望得到社区力量的支持和帮助，让 `@idux` 变得更加完善

更多内容请参见[国际化](https://idux.site/docs/i18n/zh)

### 全局配置

我们给众多组件添加了全局配置功能，这应该是你见过全局配置最多的组件库之一

用户可以通过全局配置来定义组件的默认行为，从而减少在模板中需要写的代码

支持在运行时修改全局配置项，还支持局部覆盖全局配置项

更多内容请参见[全局配置](https://idux.site/docs/config/zh)

### 主题定制

我们尽最大努力地提供了丰富的主题变量，既有全局变量，也有组件级别的变量
便于用户百变百搭，灵活定制自己想要的主题

注意: 当前该功能并不稳定，我们正在考虑使用 `css` 变量的方案替换掉 `less` 变量
等待方案完善后，我们会在每个组件的文档中声明所支持的主题变量

## 更多特性

### 响应式表单系统

响应式表单系统提供了一种模型驱动的方式来处理表单输入，参考了 `@angular/forms` 的实现

- 显式的表单模型：`FormGroup`, `FormArray`, `FormControl`
- 响应式的表单状态：`value`, `status`
- 统一且易扩展的表单验证：`Validators`

如果你使用过 `@angular/forms`, 那么你对下面的代码应该会感觉到很熟悉

![forms](https://cdn.jsdelivr.net/gh/danranVm/image-hosting/images/forms.png)

### 更加受控的组件

我们为所有组件支持 `v-model` 的 `Props` 都添加了受控模式，参见 [issue #510](https://github.com/IDuxFE/idux/issues/510)

其他的 `Props` 绝大多数也都是受控的，这是来自我们内部低代码平台的需求，他们需要完全受控的组件。

### 数据驱动理念的 API 设计

对于一些复杂组件，例如，`table`, `tree`, `menu`, 我们更加推荐用户使用数据源(`dateSource`)而不是在直接在 `template` 中使用子组件。

在实际的开发过程中，数据源往往是动态的，直接操作数据会比在 `template` 中使用 `v-if`, `v-for` 更加直接，也更利于维护。

```diff
- <IxMenu>
-   <template v-for="item in dataSource">
-     <IxMenuItem v-if="item.type === 'item'" :key="item.key" :icon="item.icon"></IxMenuItem>
-     <IxMenuDivider v-else-if="item.type === 'divider'" :key="item.key"></IxMenuDivider>
-   </template>
- </IxMenu>

+ <IxMenu :dataSource="dataSource"></IxMenu>
```

更多特性等待你的发掘，快来上手体验吧。

## 下一步计划

目前只是 `@idux` 的开始，它还有许多不够完善的地方，接下来我们将在持续的迭代中完善它，这是我们的下一步计划：

- 组件文档的完善与补充
- 视觉优化以及设计指南完善与补充
- 单元测试的完善与补充
- 主题变量方案的完善

另外，我们也在准备一个后台管理系统的模板，以供大家快速的搭建项目原型，敬请期待。

最后，开源不易，希望 `@idux` 能够得到大家的喜欢和支持。我们也将不忘开源初心，积极拥抱社区，依靠社区的力量来让 `@idux` 获得更好的成长。非常欢迎热爱开源的朋友来一起参与 `@idux` 项目，无论是 `star`, `issue` 还是 `PR` 都是对我们的帮助与支持，感谢。

## 关于 IDuxFE

我们是来自深信服科技的前端团队，`@idux` 已经在云计算&安全多场景落地，未来会实现集团内全产品覆盖，所以不用担心我们弃坑，放弃维护。

如果对我们感兴趣的话，欢迎加入我们，可以投递简历到 `uedc@sangfor.com.cn` 或者直接私信我。
