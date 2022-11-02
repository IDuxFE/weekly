# 为什么需要自动化测试

自动化测试是利用计算机去检查软件是否正常运行的方法，自动化测试一旦被创建，他可以不会吹灰之力的进行无数次重复测试。自动化测试能够预防无意引入的 bug，并鼓励开发者将应用分解为可测试、可维护的函数、模块、类和组件。

在手工测试过程中，我们应该都经历过或者看到过类似的问题：

- 版本发布时需要花好几个小时甚至几天来对我们的应用进行测试，其中老功能的测试占了不少比例
- 应用功能越来越庞大参与人越来越多后，实现一个小的 feature 或者是改一个 BUG，你会变的越来越小心翼翼，总会担心这会不会影响到其他功能
- 代码重构总是伴随着大量的回归测试

通过自动化测试，可以有效帮助团队改善这些问题，让你的团队更快速、自信地构建复杂的应用。

> 注意，并不是所有应用都有这样的特点与对应的问题存在，比如 C 端的各种活动页面，应用的生命周期短且开发周期也非常短，做自动化就是完全没必要的。

# 测试的类型

以下更多的是从前端的视角来对我们可能进行的自动化测试进行分类

- 单元测试：检查给定函数、类或组合式函数的输入是否产生预期的输出或副作用。
- 组件测试：检查你的组件是否正常挂载和渲染、是否可以与之互动，以及表现是否符合预期。这些测试比单元测试导入了更多的代码，更复杂，需要更多时间来执行。
- UI 测试：检查 UI 界面是否符合预期，往往会通过 Mock 解决对于后端的依赖。
- 端到端测试：检查跨越多个页面的功能，并对生产构建的应用进行实际的网络请求。这些测试通常涉及到建立一个数据库或其他后端。（广义上的 UI 测试，也可以认为是端到端测试）

# 组件测试的方式

## 白盒测试

白盒测试知晓一个组件的实现细节和依赖关系。它们更专注于将组件进行更独立的测试。这些测试通常会涉及到模拟一些组件的部分子组件，以及设置插件的状态和依赖性。

通常各种开源 UI 组件库的测试实现(如[ant-design](https://github.com/ant-design/ant-design/blob/master/components/date-picker/__tests__/DatePicker.test.tsx))，就更接近于这里表达的白盒测试类型，他们一般会使用 Jest、Vitest 这样的测试框架。

## 黑盒测试

黑盒测试不知晓一个组件的实现细节。这些测试尽可能少地模拟，以测试组件在页面中工作的真实情况。如果你的组件包含了多个子组件，比如你的业务里面自己封装了一个树状穿梭框 TreeTransfer 组件，它包含了一个 Tree 组件与一个 List 组件，那我们只会选择对把他们集成起来的 TreeTransfer 组件进行测试，而不是单独测试 Tree 与 List 组件。

## 灰盒测试

介于白盒与黑盒之间，不仅关注组件表面的输入输出正确性，同时也关注组件内部的情况。不像白盒测试那样详细完整，但又比黑盒测试更关注内部逻辑，常常通过一些表征性的现象、事件、标志去判断内部状态。

[Playwright](https://playwright.dev/docs/release-notes#version-122)与[Cypress](https://docs.cypress.io/guides/component-testing/writing-your-first-component-test)推出的 Components Testing 功能，在真实的浏览器中去渲染组件然后对它进行自动化测试，就是一种更接近于组件黑盒测试或者灰盒测试的体现。

# 推荐的方案

我们更加推荐使用真实的浏览器去渲染我们的组件进行黑盒或灰盒测试，越接近用户使用方式的测试时是越可信的测试。同时 Jest 等测试框架通过 jsdom 去模拟生成 Dom 的方案也有着很多局限性，比如一些与宽高计算相关的功能测试它们就无法实现。

我们选择当前最热门的两个端到端测试框架`Playwright(41.8k Star)`与`Cypress(40.4K)`针对于 Components Testing 功能去进行对比，他们都能够在真实地浏览器中渲染组件进行组件测试。

# Playwright 组件测试案例

## Playwright 简介

Playwright 是 2020 年微软推出的一个专门用来做 Web 应用的测试与自动化的框架，他们够通过同一套 API 去测试你的 Web 应用运行在 Chromium, Firefox and WebKit 浏览器的情况。支持使用 Jascript/TypeScript/Java/Python/.NET 语言。

更多的特点以及介绍可以查看[官方文档首页](https://playwright.dev/)。

与 Puppeteer（一个使用 NodeJS 操作浏览器的库）的关系与差异：

它的开发团队来自于 Puppeteer，Puppeteer 受到了广泛的欢迎，他们决定把他发扬广大，适用在更多的浏览器上，并且充分借鉴它的优点与踩过的坑，做出大量新的设计，也产成了很多破坏性的变更，于是有了 Playwright。

## playwright 架构图

![playwright架构图](https://tva1.sinaimg.cn/large/008vxvgGgy1h7qs37if16j31020b3jsg.jpg)

@playwright/test 里面实现了测试用例的运行、断言、测试报告生成等功能

@playwright/test 中的自动化测试代码会调用到 Playwright 中各种操作浏览器的 API

而 playwright 通过[Chrome DevTools Protocol](https://chromedevtools.github.io/devtools-protocol/)去控制浏览器，使用 Websocket 进行通信，整个过程中不会出现频繁启动浏览器与建立通信的情况

## BrowserContext

存在于 browser 与 page 之间，每个 browser 可以创建出多个完全独立的 context，context 的创建速度快且资源消耗少，每个 context 可以创建多个 page。

通过这个设计，我们可以去操作多个 session 独立的浏览器上下文，同时在每次运行测试时也可以做到只启动一次浏览器，每一个 test case 都使用一个独立 context 去进行测试。

## 组件测试原理

![组件测试原理](https://tva1.sinaimg.cn/large/008vxvgGgy1h7qs38x0kwj30ev0jvmy5.jpg)

## 开始

参考官方文档中的[How to get started](https://playwright.dev/docs/test-components#how-to-get-started)

在下面两个仓库也提供了通过 Playwright 搭建的组件测试框架，下面所有演示的完整代码都可以在里面找到

- [Vue2 Playwright Components Testing](https://github.com/hangboss1761/front-end-testing-share/tree/master/code/playwright-base)
- [Vue3 Playwright Components Testing](https://github.com/hangboss1761/front-end-testing-share/tree/master/code/playwright-ct-vue3)

## 组件引入

```ts
// playwright/index.ts 在Vue2中引入组件
// Import styles, initialize component theme here.
import ElementUI from 'element-ui';
import 'element-ui/lib/theme-chalk/index.css';

// eslint-disable-next-line @typescript-eslint/ban-ts-comment
// @ts-ignore
Vue.use(ElementUI);

// playwright/index.ts 在Vue3中引入组件
// Import styles, initialize component theme here.
import { beforeMount } from '@playwright/experimental-ct-vue/hooks';
import ElementPlus from 'element-plus';
import 'element-plus/dist/index.css';

beforeMount(async ({ app }) => {
  app.use(ElementPlus);
});
```

除了全局引入，也可以选择在测试文件中按需引入对应的被测试组件。

## 模型封装

类似于端到端测试中的页面对象模型，我们对组件测试过程中的通用行为、逻辑进行封装，来达到简化逻辑与代码复用的目的。

```ts
const useSelect = (ct: Locator, page: Page) => {
  const pickSelectOption = async ({ text, nth }: { text?: string; nth?: number }) => {
    if (text) {
      await page.locator(`.el-select-dropdown:visible .el-select-dropdown__item :text-is("${text}")`).click();
    } else {
      await page.locator(`.el-select-dropdown:visible .el-select-dropdown__item >> nth=${nth}`).click();
    }
  };

  const openPopover = async () => {
    await ct.locator('.el-input').click();
    // 等待popover动画执行完毕
    // eslint-disable-next-line playwright/no-wait-for-timeout
    await page.waitForTimeout(400);
  };

  return {
    pickSelectOption,
    openPopover,
  };
};
```

## 组件渲染测试

我们推荐使用视觉对比去测试组件渲染是否符合预期。

```html
<!-- Select.vue -->
<template>
  <el-select v-bind="propsParams" v-model="value" placeholder="请选择" v-on="eventsParams">
    <el-option
      v-for="item in options"
      :key="item.value"
      :label="item.label"
      :value="item.value"
      :disabled="item.disabled"
    >
    </el-option>
  </el-select>
</template>

<script>
export default {
  props: {
    // component props
    propsParams: {
      type: Object,
      default: () => ({}),
    },
    // component events
    eventsParams: {
      type: Object,
      default: () => ({}),
    },
    // custom props
    defaultValue: {
      type: String,
      default: '',
    },
    options: {
      type: Array,
      default: () => [],
    },
  },
  data() {
    return {
      value: this.defaultValue,
    };
  },
};
</script>

```

```ts
test('mount work', async ({ page, mount }) => {
  const ct = await mount(SelectBase, {
    props: {
      // custom props
      options: baseOptions,
    },
  });
  const { openPopover } = useSelect(ct, page);

  await openPopover();

  // Visual comparisons
  // allow 5% pixe ratio diff
  await expect(page).toHaveScreenshot({ maxDiffPixelRatio: 0.5 });
});
```

## 组件 Props 测试

```ts
test('single select work', async ({ page, mount }) => {
  const ct = await mount(SelectBase, {
    props: {
      options: baseOptions,
      defaultValue: baseOptions[0].value,
    },
  });
  const { pickSelectOption, openPopover } = useSelect(ct, page);

  await openPopover();
  await pickSelectOption({ text: baseOptions[1].label });

  await expect(ct.locator('.el-input input')).toHaveValue(baseOptions[1].label);
});
```

## 组件 Events 测试

```ts
test('event work', async ({ page, mount }) => {
  const messages: string[] = [];

  const ct = await mount(SelectBase, {
    props: {
      propsParams: {
        clearable: true,
      },
      eventsParams: {
        change: () => messages.push('change-trigger'),
        clear: () => messages.push('clear-trigger'),
        'visible-change': () => messages.push('visible-change-trigger'),
      },
      options: baseOptions,
    },
  });
  const { pickSelectOption, openPopover } = useSelect(ct, page);

  await openPopover();
  await pickSelectOption({ text: baseOptions[0].label });

  await ct.locator('.el-input').hover();
  await ct.locator('.el-icon-circle-close').click();

  expect(messages).toContain('change-trigger');
  expect(messages).toContain('clear-trigger');
  expect(messages).toContain('visible-change-trigger');
});
```

## 组件 Slots 测试

```ts
test('slots work', async ({ mount }) => {
  const ct = await mount(Button, {
    slots: {
      default: 'click me',
    },
  });

  await expect(ct).toContainText('click me');
});

test('jsx slots work', async ({ mount }) => {
  const ct = await mount(<el-button>click me</el-button>);

  await expect(ct).toContainText('click me');
});
```

## 快捷键测试

```ts
test('keyboard operations', async ({ page, mount }) => {
  const ct = await mount(SelectBase, {
    props: {
      options: baseOptions,
      defaultValue: baseOptions[0].value,
    },
  });
  const { openPopover } = useSelect(ct, page);

  await openPopover();

  await ct.locator('.el-input').press('ArrowDown');
  await ct.locator('.el-input').press('ArrowDown');
  await ct.locator('.el-input').press('Enter');

  await expect(ct.locator('.el-input input')).toHaveValue(baseOptions[1].label);
});
```

# Cypress 组件测试案例

## Cypress 简介

Cypress 是基于 JavsScript 的前端测试工具，可以对浏览器中运行的任何内容进行更快速简单可靠的测试，它可以用来编写所有类型的测试（端到端测试、接口测试、单元测试）。

更多的特点以及介绍可以查看[Why Cyrpess](https://docs.cypress.io/guides/overview/why-cypress#Features)的 Features 章节。

**Cypress 架构图**
![cypress架构图](https://tva1.sinaimg.cn/large/008vxvgGgy1h7qs3aewcaj30mw0bvmxw.jpg)

Cypress 的测试代码与被测试的 web 应用会直接在同一个浏览器中运行，不需要额外的驱动程序（如 WebDriwer），Cypress 对于被测试的 web 应用很强的控制能力。

在浏览器这个级别，Cypress 能够直接操作来自 web 应用的 Dom、Window、Local Storage、network 等内容。

浏览器之后是一个 Nodejs 进程，通过它启动浏览器后，与浏览器之间会使用
WebSocket 链接进行通信。同时在这里对于网络请求会进行代理控制，可以做到读取和更改网络请求等操作。

在操作系统级别，Cypress 通过 NodeJs 进程可以做到截图、录制视频、文件读写等操作。

## 开始

参考官方文档中的[Quick Start vue](https://docs.cypress.io/guides/component-testing/quickstart-vue)

在下面的仓库也提供了通过 Cypress 搭建的组件测试框架，下面所有演示的完整代码都可以在里面找到

- [Vue3 Cypress Components Testing](https://github.com/hangboss1761/front-end-testing-share/tree/master/code/cypress-base)

## 组件引入

```ts
// cypress/support/component.ts 在Vue3引入全局组件，引入方式不友好，Vue3里面更推荐按需引入
import { mount } from 'cypress/vue'
import Button from '../../src/components/Button.vue'

Cypress.Commands.add('mount', (component, options = {}) => {
  // Setup options object
  options.global = options.global || {}
  options.global.components = options.global.components || {}

  // Register global components
  options.global.components['Button'] = Button

  return mount(component, options)
})

// cypress/support/component.ts 在Vue2引入全局组件
import Vue from 'vue';
import ElementUI from 'element-ui';
import 'element-ui/lib/theme-chalk/index.css';
import { mount } from 'cypress/vue';

Vue.use(ElementUI);

Cypress.Commands.add('mount', mount);
```

## 模型封装

```ts
const selectModal = {
  pickSelectOption: ({ text, nth }: { text?: string; nth?: number }) => {
    if (text) {
      cy.get('.el-select-dropdown:visible .el-select-dropdown__item').contains(text).click();
    } else {
      cy.get(`.el-select-dropdown:visible .el-select-dropdown__item::nth-child(${nth})`).click();
    }
  },
  openPopover: () => {
    // 等待popover动画执行完毕
    cy.get('.el-input').click().wait(400);
  },
};
```

## 组件渲染测试

```html
<template>
  <el-select v-bind="propsParams" v-model="value" placeholder="请选择" v-on="eventsParams">
    <el-option
      v-for="item in options"
      :key="item.value"
      :label="item.label"
      :value="item.value"
      :disabled="item.disabled"
    >
    </el-option>
  </el-select>
</template>

<script lang="ts" setup>
import { ref } from 'vue';
import { ElSelect, ElOption } from 'element-plus';

interface OptionItem {
  value: string;
  label: string;
  disabled?: boolean;
}

const props = withDefaults(
  defineProps<{
    // eslint-disable-next-line @typescript-eslint/no-explicit-any
    propsParams?: Record<string, any>;
    // eslint-disable-next-line @typescript-eslint/no-explicit-any
    eventsParams?: Record<string, any>;
    defaultValue?: string;
    options: OptionItem[];
  }>(),
  {
    propsParams: () => ({}),
    eventsParams: () => ({}),
    defaultValue: '',
    options: () => [],
  },
);

const value = ref(props.defaultValue);
</script>

```

```ts
it('mount work', () => {
  cy.mount(SelectBase, {
    props: {
      options: baseOptions,
    },
  });

  selectModal.openPopover();

  /**
   * 官方文档推荐的cypress-plugin-snapshots插件在cypress10.6.0使用时报错
   * 相关issue见：https://github.com/meinaart/cypress-plugin-snapshots/issues/215
   * 这里使用https://github.com/FRSOURCE/cypress-plugin-visual-regression-diff 来实现视觉对比
   */
  cy.matchImage();
});
```

## 组件 Props 测试

```ts
it('single select work', () => {
  cy.mount(SelectBase, {
    props: {
      options: baseOptions,
      defaultValue: baseOptions[0].value,
    },
  });

  selectModal.openPopover();
  selectModal.pickSelectOption({ text: baseOptions[1].label });
  cy.get('.el-input input').should('have.value', baseOptions[1].label);
});
```

## 组件 Events 测试

```ts
it('event work', () => {
  const messages: string[] = [];

  cy.mount(SelectBase, {
    props: {
      propsParams: {
        clearable: true,
      },
      eventsParams: {
        change: () => messages.push('change-trigger'),
        clear: () => messages.push('clear-trigger'),
        'visible-change': () => messages.push('visible-change-trigger'),
      },
      options: baseOptions,
    },
  });

  selectModal.openPopover();
  selectModal.pickSelectOption({ text: baseOptions[0].label });

  cy.get('.el-input').click();
  cy.get('.el-select__icon:visible').click();

  cy.wrap(messages)
    .should('include', 'clear-trigger')
    .should('include', 'change-trigger')
    .should('include', 'visible-change-trigger');
});
```

## 组件 Slots 测试

```ts
it('slot work', () => {
  cy.mount(ElButton, {
    slots: {
      default: () => <span>click me</span>,
    },
  });

  cy.get('button').should('have.text', 'click me');
});

it('jsx slot work', () => {
  cy.mount(() => <ElButton>click me</ElButton>);

  cy.get('button').should('have.text', 'click me');
});
```

## 快捷键测试

```ts
it('keyboard operations work', () => {
  cy.mount(SelectBase, {
    props: {
      options: baseOptions,
      defaultValue: baseOptions[0].value,
    },
  });

  selectModal.openPopover();

  cy.get('.el-input').type('{downArrow}');
  cy.get('.el-input').type('{downArrow}');
  cy.get('.el-input').type('{enter}');

  cy.get('.el-input input').should('have.value', baseOptions[1].label);
});
```

# 总结

在前端自动化测试中，组件测试是我们认为最具有性价比的测试类型，在对组件进行测试时对于其他服务基本没有依赖，可以独立进行测试，同时各种基础组件、业务组件也广泛的使用在我们的系统中的各个地方，通过自动化测试保障好它们的质量对于系统整体的质量提升有这明显的帮助。

组件测试方面两个测试框架功能层面的差距基本没有，如果你仅仅只考虑组件测试，那么这两个框架都是非常推荐的。如果你同样关注端到端测试以及两个框架更深层次的对比，可以阅读文章[Playwright VS Cypress](https://github.com/hangboss1761/front-end-testing-share/blob/master/docs/playwright-vs-cypress/playwright-vs-cypress.md)。
