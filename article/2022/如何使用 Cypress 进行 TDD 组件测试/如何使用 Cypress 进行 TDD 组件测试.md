# 如何使用 Cypress 进行 TDD 组件测试

作为前端，我们经常有听到 TDD、写单测等等，那么跟 UI 强关联的组件的测试应该怎么做？

本文使用 `Cypress` 框架，通过一个组件示例，一步步进行实践，尝试把 TDD 在前端落地。



## 环境搭建

`Cypress` 是一个完整易用的测试框架，我们可以使用 `Cypress` 进行 `e2e`测试、集成测试、单元测试

用来实现组件测试有着几大优势

- 支持真实的浏览器运行环境，直接使用web浏览器上的开发工具直接调试

- 在运行测试的时候，会获取快照，记录了测试执行过程的每一步细节

- 运行速度非常快，基本可以与浏览器内容实时同步

  

在 `Cypress` 官方的组件测试示例仓库 [cypress-component-examples](https://github.com/cypress-io/cypress-component-examples) 中，选择 [vite-vue](https://github.com/cypress-io/cypress-component-examples/tree/main/vite-vue) 作为初始化的模板

> 实际业务中已有的项目可以参考 [Vite Based Projects (Vue, React)](https://docs.cypress.io/guides/component-testing/framework-configuration#Vite-Based-Projects-Vue-React) 中说明进行接入



![](https://raw.githubusercontent.com/miomio-xiao/picture/main/images/2022/cypressimage-20220327170450796.png)

可以看到，有一个 `HelloWorld.spec.js` 的测试文件

首先开始安装、运行，看看是什么效果

```bash
pnpm install

# 在浏览器打开测试用例集的界面
pnpm cypress open-ct
```

![](https://raw.githubusercontent.com/miomio-xiao/picture/main/images/2022/cypressimage-20220327170823183.png)

从 `Cypress` 自带的测试用例预览界面可以看到，用例已经正常运行通过了，接下来进入正文



> 由于后续示例代码使用 `ts` 编写，这里先添加 `@vitejs/plugin-vue-jsx` 插件
>
> ```js
> // vite.config.js
> import { defineConfig } from 'vite';
> import vue from '@vitejs/plugin-vue';
> import vueJsx from '@vitejs/plugin-vue-jsx';
> 
> // https://vitejs.dev/config/
> export default defineConfig({
>   plugins: [vue(), vueJsx()],
> });
> ```
>
> ```json
> // cypress.json
> {
>   "testFiles": "**/*.spec.[jt]s",
>   "componentFolder": "src"
> }
> ```



## 组件测试

以一个 Rate 评分组件为例

### 功能需求

![](https://raw.githubusercontent.com/miomio-xiao/picture/main/images/2022/cypressimage-20220328000911955.png)

1. 基础显示
2. 支持点击选中
3. 支持传入初始值选中
4. 支持hover高亮

受限篇幅，本文只支持以上功能

### 测试驱动开发

测试驱动开发，即 TDD，它的规则很简单，可以归纳为下面三条：

- 先编写一个因为缺乏实现代码而运行失败的测试，然后编写实现代码。
- 只允许编写一个**刚好失败**的测试 - 编译失败也算失败。
- 只允许编写**刚好能使当前失败测试通过**的实现代码。

遵循 TDD 三原则，意味着你的每一行实现代码都是有测试保证的，先有的测试，才有的你那一行**恰好**可以通过的实现代码。你的测试是完备的，你有信心部署你测试全过的代码，这些测试告诉我们，我们的系统是可靠、可部署的。

通过上述的三原则，从第一个测试用例开始

#### 编写第一个测试用例

在 `components` 下新建 `rate` 目录存放相关代码实现以及测试用例

![](https://raw.githubusercontent.com/miomio-xiao/picture/main/images/2022/cypressimage-20220327172841807.png)

如上图所示，由于我们还没开始写 `Rate` 组件的实现，现在导入组件是编译报错的状态

假设初始化会渲染 5 个类为 `mio-rate-item` 的元素，那么此时写下第一个测试用例代码

```ts
import { mount } from '@cypress/vue';
import Rate from './index';

describe('rate component', () => {
  it('should render 5 item elements', () => {
    mount(Rate);

    cy.get('.mio-rate-item').should('have.length', 5);
  });
});

```

![](https://raw.githubusercontent.com/miomio-xiao/picture/main/images/2022/cypressimage-20220327173449313.png)

打开浏览器用例运行界面，可以看到左侧的用例列表多出来了 `src/component/rate/rate.spec.ts` ，且编译摆错了。

#### 针对第一个用例编写实现代码

为了使刚才写的第一个用例通过，回想之前提到的三原则，这次只针对性的写这个用例对应的实现代码

创建一个容器，然后渲染 5 个类名为 `mio-rate-item` 的子元素

```tsx
// rate/index.tsx
import { defineComponent } from 'vue';
import { renderIcon } from './icon';
import './index.css';

const getNumberList = (num: number) => {
  return Array.from({ length: num }).map((_, i) => i + 1);
};

const useRateClasses = () => {
  return ['mio-rate-item'];
};

export default defineComponent({
  name: 'Rate',

  setup() {
    const renderRateItem = () => (
      <div class={useRateClasses()}>{renderIcon()}</div>
    );
    const renderRate = () => (
      <div class='mio-rate'>{getNumberList(5).map(() => renderRateItem())}</div>
    );

    return () => renderRate();
  },
});
```



![](https://raw.githubusercontent.com/miomio-xiao/picture/main/images/2022/cypressimage-20220327174940290.png)

查看用例运行界面可以发现用例已经运行通过了，右侧的界面成功的渲染了 5 个 ⭐️



#### 第二个测试用例

根据上述的步骤继续，这次需要支持点击选择功能

假设我们点击后会给选中的子元素加上 `.is-active` ，那么自然而然写下测试用例代码

```ts
import { mount } from '@cypress/vue';
import Rate from './index';

describe('rate component', () => {
  it('should render 5 item elements', () => {
    mount(Rate);

    cy.get('.mio-rate-item').should('have.length', 5);
  });

  it('should be highlighted when the element is clicked', () => {
    mount(Rate);

    cy.get('.mio-rate-item.is-active').should('have.length', 0);

    // 点击第 3 个子元素
    cy.get('.mio-rate-item:nth-of-type(3)').click();
	
    // 带有 .is-active 的子元素应该有 3 个
    cy.get('.mio-rate-item.is-active').should('have.length', 3);
  });
});
```



![](https://raw.githubusercontent.com/miomio-xiao/picture/main/images/2022/cypressimage-20220327180001067.png)

查看运行界面，可以看到用例运行状态是失败的，右侧渲染的组件中也没有高亮

然后需要在对应的组件文件进行实现

```tsx
import { defineComponent, ref, Ref } from 'vue';
import { renderIcon } from './icon';
import './index.css';

const getNumberList = (num: number) => {
  return Array.from({ length: num }).map((_, i) => i + 1);
};
function useRateClasses({
  currentValue,
  index,
}: {
  currentValue: Ref<number>;
  index: number;
}) {
  return ['mio-rate-item', currentValue.value >= index ? 'is-active' : ''];
}

export default defineComponent({
  name: 'Rate',

  setup() {
    const currentValue = ref(0);
    const onClickItem = (index: number) => (currentValue.value = index);

    const renderRateItem = (index: number) => (
      <div
        class={useRateClasses({ currentValue, index })}
        onClick={() => onClickItem(index)}
      >
        {renderIcon()}
      </div>
    );
    const renderRate = () => (
      <div class='mio-rate'>
        {getNumberList(5).map((i) => renderRateItem(i))}
      </div>
    );

    return () => renderRate();
  },
});
```



![](https://raw.githubusercontent.com/miomio-xiao/picture/main/images/2022/cypresscypress-rate.gif)

切换到用例运行的面板，可以看到用例已经执行成功了

点击步骤可查看组件的中间状态，如果中间出现问题，还可以打开 chrome devtools 去调试



#### 重复上述步骤

根据之前提到的 TDD 三原则，重复的进行 **写用例 -> 写实现代码 -> 调试通过 -> 重构/优化设计 -> 写用例 -> ...** 的过程。

- `v-model` 功能

  支持传入初始值选中显示

  - `props` 用例

    ```ts
    import { mount, mountCallback } from '@cypress/vue';
    import Rate from './index';
    
    describe('rate component', () => {
      // ...
      describe('v-model value', () => {
        
        // 当前 describe 作用域下每个用例执行前进行 mount
        beforeEach(
          mountCallback(Rate, {
            propsData: {
              modelValue: 3,
            },
          }),
        );
    
        it('should work when set props value', () => {
          cy.get('.mio-rate-item.is-active')
            .should('have.length', 3)
            .then(() => {
            
              // 参考 @vue/test-utils 的 wrapper api
              Cypress.vueWrapper.setProps({
                modelValue: 4,
              });
              cy.get('.mio-rate-item.is-active').should('have.length', 4);
            });
        });
      });
    });	
    ```

  - `Props` 逻辑实现

    ```ts
    export default defineComponent({
      name: 'Rate',
      props: {
        modelValue: {
          type: Number,
          default: 0,
        },
      },
    
      setup(props) {
        const currentValue = ref(props.modelValue);
    
        watch(
          () => props.modelValue,
          (value) => {
            currentValue.value = value;
          },
        );
        // ...
      },
    });
    ```

  - `emit` 用例

    ```ts
    it('should be emit input when the element is clicked', () => {
      cy.get('.mio-rate-item.is-active').should('have.length', 3);
    
      cy.get('.mio-rate-item:nth-of-type(2)')
        .click()
        .then(() => {
        expect(Cypress.vueWrapper.emitted()['update:modelValue'].length).to.eq(1);
        expect(Cypress.vueWrapper.emitted()['update:modelValue'][0][0]).to.eq(2);
      });
    });
    ```

  - `emit` 实现

    ```ts
    setup(props, { emit }) {
      const currentValue = ref(props.modelValue);
    
      watch(currentValue, (value) => {
        emit('update:modelValue', value);
      });
      // ...
    })
    ```

- 悬浮高亮功能

  - 用例

    ```ts
    it('should be highlighted when hover', () => {
      mount(Rate);
    
      cy.get('.mio-rate-item:nth-of-type(3)').trigger('mouseenter');
    
      cy.get('.mio-rate-item.is-active').should('have.length', 3);
    
      cy.get('.mio-rate-item:nth-of-type(3)').trigger('mouseleave');
    
      cy.get('.mio-rate-item.is-active').should('have.length', 0);
    });
    ```

  - 实现

    ```ts
    // ...
    function useRateClasses({
      currentValue,
      currentOverValue,
      index,
    }: {
      currentValue: Ref<number>;
      currentOverValue: Ref<number>;
      index: number;
    }) {
      return [
        'mio-rate-item',
        (currentOverValue.value || currentValue.value) >= index ? 'is-active' : '',
      ];
    }
    
    		// setup ...
        const currentOverValue = ref(0);
        const renderRateItem = (index: number) => (
          <div
            class={useRateClasses({ currentValue, currentOverValue, index })}
            onClick={() => onClickItem(index)}
            onMouseenter={() => (currentOverValue.value = index)}
            onMouseleave={() => (currentOverValue.value = 0)}
          >
            {renderIcon()}
          </div>
        );
    
    ```

#### 运行效果

![cypress-rate-final](https://raw.githubusercontent.com/miomio-xiao/picture/main/images/2022/cypresscypress-rate-final.gif)

可以看到，用例已经全部运行通过了

#### 重构

完成了上述的过程是否就已经结束了呢？其实还漏了一个重要的步骤，那就是重构。

如果有任何重复的逻辑、比较冗余的代码，重构可以消除重复并提高表达能力（减少耦合，增加内聚力）。

再次运行测试验证重构是否引入新的错误。如果没有通过，很可能是在重构时犯了一些错误，需要立即修复并重新运行，直到所有测试通过。

以上述实现的 `v-model` 功能为例，在封装组件的时候，这类功能是比较常见的，那么这部分是否可以抽离出一个单独的函数来维护？先简单来实践一下

首先封装一个名为 `useVModel` 的函数，将 `v-model` 所涉及到的关联逻辑放进来

```ts
const useVModel = <T extends { modelValue: T['modelValue'] }>(props: T) => {
  const { emit } = getCurrentInstance();

  const proxy = ref(props.modelValue);

  watch(proxy, (value) => {
    emit('update:modelValue', value);
  });

  watch(
    () => props.modelValue,
    (value) => {
      proxy.value = value as UnwrapRef<T['modelValue']>;
    },
  );

  return proxy;
};
```

在业务中替换使用

```diff
- const currentValue = ref(props.modelValue);
- 
- watch(currentValue, (value) => {
-   emit('update:modelValue', value);
- });
- 
- watch(
-   () => props.modelValue,
-   (value) => {
-     currentValue.value = value;
-   },
- );

+ const currentValue = useVModel(props);
```

替换完成后，再次执行刚才写的测试用例，正常通过。

通过这个重构操作，上述封装的 `useVModel` 就可以在其他地方进行复用，也简化了在业务上的调用逻辑。



总的来说，有了之前的测试用例基础，重构也有对应的质量保障，且重构能够 **消除重复设计，优化设计结构** ，对于整体的代码质量，可维护性与可扩展性都有了提升。



## 总结

**测试驱动开发** 要求每次只添加一个行为，先写一个失败的测试，然后写出恰好能使这个测试通过的实现代码。

你写出的每一个测试都是一份代码示例，如何调用 API，如何创建某个对象。测试已经有了超过 90% 使用场景覆盖，而且这可以立即发现错误，去调试、修复它。如果先写一大堆实现代码，再来补测试，这时候已经先入为主，你很难发现自己的代码有什么问题。

在此过程中穿插的重构，也会让我们不断的思考如何实现好的代码，提升整体的代码质量。



另外，在工具层面，`Cypress` 新的[组件测试器](https://on.cypress.io/component)对测试组件有着很好的支持，而且对于 `vite` 项目来说，也有比较好的集成，是测试在浏览器中呈现的任何内容理想选择。



## 参考链接

> https://docs.cypress.io/  官方文档
>
> https://github.com/cypress-io/cypress-component-examples  官方代码模板仓库
>
> https://www.npmjs.com/package/@cypress/vue vue组件测试配套工具库
>
> [vite-vue](https://github.com/miomio-xiao/cypress-component-examples/tree/main/vite-vue) 本文示例代码仓库
>
> https://teobler.com/posts/20210221-agile-technology-practices-tdd