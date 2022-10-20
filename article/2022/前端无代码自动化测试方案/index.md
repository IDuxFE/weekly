# 前端无代码E2E自动化测试方案

## 背景
### 1. 为什么要做前端自动化测试：
- 前端自动化测试能力不足，绝大部分的没有自动化测试覆盖，改动引发高，版本上线没有质量保障，每次发版都提心吊胆
- 绝大部分UI用例都以来测试人员手工执行，甚至存在同个用例多个迭代多次手工执行的情况，测试人力成本高
- 大规模的组件库升级、代码重构无法做到可靠测试，人工测试不可能全量覆盖

### 1. 为什么是无代码自动化测试
- 由开发人员手工编写E2E自动化测试脚本耗时长，编码：写自动化脚本时间在1:2左右，如果是新手，可能需要1:1。在敏捷团队中，每个迭代只有2周的时间，根本没有时间手写自动化测试脚本
- 成本高，手写E2E自动化测试脚本需要投入大量的人力时间，随着业务的增长，需要持续增加投入，性价比是个瓶颈，需要一个业务量指数增加，低自动化工作量的方案
- 针对老项目的老模块，人工补充自动化测试脚本成本也是非常高
- 无代码自动化测试不面向脚本代码，没有语言门槛，前端、后端、测试人员、pm都可以进行UI E2E测试
- 基于以上原因，E2E自动化测试能真正发挥的价值受限

**为此我们急需新的方案解决以上问题，带来业务质量和研发效能的提升**

## 先聊聊自动化测试原理
### WebDriver
WebDriver是W3C标准中的浏览器远程控制协议（Protocol）
[WebDriver介绍](https://link.juejin.cn/?target=https%3A%2F%2Fwww.w3.org%2FTR%2Fwebdriver%2F)
![](https://files.mdnice.com/user/36775/1bceb47a-724d-47cf-9e3e-183dcbd94825.png)
Client就是我们的测试代码，Browser就是我们要控制的浏览器，中间会有个WebDriver协议的实现，就是Browser Driver。

在Client端通过Language Binding，可以实现多语言的测试脚本（Java、Ruby、Python、C#、Javascript etc...）

在Browser Driver端，是各个浏览器基于WebDriver的实现，用来远程控制浏览器

当执行测试脚本时：发起http请求，基于JSON Wire Protocol，生成返回JSON格式的请求，发送给Browser Driver

Browser Driver通过http服务器接受请求，再通过http server决定执行在浏览器的指令

执行状态将发送返回给http server，再将其发送回自动化脚本

### CDP
全称：Chrome Devtools Protocol

也就是：Chrome浏览器开发工具协议

[CDP介绍](https://link.juejin.cn/?target=https%3A%2F%2Fchromedevtools.github.io%2Fdevtools-protocol%2F)

![](https://files.mdnice.com/user/36775/5c5c9137-f2e5-47de-94a6-4a3802f2da44.png)

在 Selenium 中 chrome driver的实现中也是基于CDP二次开发去控制浏览器。

类似的还有FDP（FireFox Devtools Protocol）等

Puppeteer、playwright就是基于CDP的实现

**相比于Webdriver的协议，CDP可以做更多底层的操作，比如可以检查/调试/监听网络流量，我们更倾向于选择基于CDP的自动化框架**

## 无代码自动化测试方案

### 无代码方案面临的需求
| 需求       | 描述 |
| :--------- | :--: |
| 适合自己 |  适合自己公司的前端项目才是最重要  |
| 可靠     |  自动化测试的要求，要保证测试是准确的  |
| 效率高   |  不需要人工编写自动化脚本，效率相比人工编写效率至少提升3倍以上  |
| 成本低   |  不需要投入很多人就可短时间完成一个大项目自动化覆盖，支持不改造项目本身  |
| 开箱即用 |  不需要过多的环境配置，开箱即用  |
| 可扩展 |  针对不同的项目，可以自定义一些扩展配置  |
| 兼容性测试 |  支持至少chrome、Firefox、webkit三种浏览器内核的测试  |


### 方案对比（非人工编码自动化测试方案之间的对比）

| 方案名       | 优点 |缺点 |
| :--------- | :--: | :--: |
| ui-recorder  |  报告直观<br>浏览器兼容多（chrome、IE、FF等）| 环境配置麻烦<br>可靠性低，诸如元素等待、请求等待等细节无处理<br>坑多<br>基于WebDriver |
| playwright-test | 微软背书功能强大<br>基于CDP<br>可靠性高<br>自动等待<br>社区活跃  | 无代码能力较弱，生成脚本还需要人工加工，成本不低<br>高度封装难以扩展，无法适配公司业务<br>报告简单 |
| testIM | 开箱即用，使用简单<br>基于CDP<br>集成化高<br>完全无代码 | 需要付费使用<br>可靠性一般，不支持自动等待<br>高度封装，无法适配公司业务 |
| sft（自研） | 可扩展，更适合自己<br>开箱即用<br>效率高<br>使用成本低<br>支持无埋点<br>自动等待<br>完全无代码  | 自研需要成本<br>功能广度目前还不如开源框架，需要花时间人力迭代演进<br> |


**最终采取基于playwright自研sft无代码自动化测试方案**

> 接下来先讲讲sft方案的效果，再接着讲sft的技术实现

## sft无代码测试方案效果

1.开箱即用：
```shell
npm i -g @sxf/sft   // 全局安装sft-cli

sft init   // 在执行目录初始化sft配置文件
```

2.录制用例：
```shell
sft record [pageURL] -o [caseOutputDir] -c [configDir]
```

3.回放用例：
```shell
sft run [caseDir] -c [configDir]
```

4.查看报告：
![](https://files.mdnice.com/user/36775/34ad4bdf-1b40-47b3-9eb9-866c7ff27d54.png)

> 以上就是一次用例生成->回放的流程，整个过程不需要面对自动化脚本代码，也不需要改造原业务项目，甚至可以不埋点（当然sft也支持配置埋点，能更加准确和持久维护，但对于sft而言，自动生成的选择器已经经过算法的处理，属于最佳选择器，后面会介绍sft选择器算法）

5.CI集成
![](https://files.mdnice.com/user/36775/42846c2f-7249-4c71-81ab-4b600ad06acc.jpg)

## sft无代码测试方案实现

### sft整体规划地图

![](https://files.mdnice.com/user/36775/acbfccbe-104c-4088-82eb-85e95ec3c0be.png)

> sft作为一个项目，规划了多个迭代和其中的功能，当前处于迭代二发布，支持sangfor前端项目的无代码自动化测试基本需求，后续会继续开展迭代三及后面的迭代，不断增强sft能力

### sft整体架构图

![](https://files.mdnice.com/user/36775/5c7e1e2c-8e40-4daf-abb2-95d976d2805c.png)

> sft是基于playwright自研的上层框架，包含了sft-client、sft-cli、sft-web、sft-ai-selector、sft-engine、sft-inject多个模块


#### sft-cli
sft工具端模块，包含sft record（录制命令）、sft run（回放命令）、sft-init（初始化配置命令）。sft-cli是整个sft的功能调用入口

#### sft-engine
sft的核心引擎模块。sft的用例录制、回放与浏览器操作管理、用例管理、断言管理、web请求管理、报告收集等都在sft-engine中，是整个sft的心脏。下面聊聊这些如何实现上面的功能

##### 录制用例流程

![](https://files.mdnice.com/user/36775/c2434b29-64fa-4ad1-8000-9b4dfc76556b.png)

录制的最终目的是生成无代码自动化用例数据（产物是json以及一些断言资源如截图、元素快照等）
![](https://files.mdnice.com/user/36775/7f1d46be-6b77-4b7b-9fc8-94f38ee9f94f.png)

>录制的产出是json而不是自动化脚本代码，也是无代码自动化的标志（不在面向代码做自动化测试）。而json中包含了录制过程中每个浏览器动作和断言的信息；json具有强大扩展能力，为未来扩展增量录制、用例可视化管理提供了可能

##### 回放用例流程

![](https://files.mdnice.com/user/36775/b4f24315-d6b9-48f1-b090-8c6eac9fe5df.png)

##### 断言管理
sft基于expect进行断言，当前已经支持的断言类型有：url断言、元素截图断言、页面截图断言、元素快照断言。未来将进一步扩充断言类型

##### 请求管理
页面一些诸如提交、刷新后的动作需要等待请求后执行，所以sft封装了一个请求管理类（其原理是一个请求队列，不断塞入新请求，剔出已完成的请求），当页面请求完成后才会执行下一步action，保证了操作的可靠性，使线上回放用例成为可能

##### 报告管理
sft的报告包含用例每一个action的执行结果，即需要对浏览器时间和断言动作进行监听生成对应action的报告。

**1.浏览器事件报告收集：**
浏览器事件sft是交给playwright进行操作浏览器，所以我们可以通过监听playwright api获得浏览器事件action的结果：

```javascript
initListener (testInfoImpl: TestInfoImpl) {
    let stepIndex = 0
    const createInstrumentationListener = context => {
      return {
        onApiCallBegin: (apiCall, stackTrace, userData) => {
          // playwright api开始监听器
          const stepId = `pw:api@${apiCall}@${stepIndex++}`

          const step :Step = {
            stepId,
            startTime: Date.now(),
            title: `操作浏览器api：${apiCall}`,
            status: 'pass'
          }

          testInfoImpl.addStep(step)

          userData.userObject = step

        },
        onApiCallEnd: (userData, error) => {
          // playwright api结束监听器

          const step = userData.userObject

          if (step) {
            step.endTime = Date.now()
          }

          if (error) {
            step.error = error.toString()
            step.status = 'fail'
          }
        }
      };
    };

    const onDidCreateBrowserContext = async context => {
      const listener = createInstrumentationListener(context);

      context._instrumentation.addListener(listener);
    };

    for (const browserType of [chromium, firefox, webkit]) {

      // 为个浏览器注册playwright api监听器
      (browserType as any)._onDidCreateContext = onDidCreateBrowserContext;
    }
  }
```

**2.断言事件报告收集：**
expect断言库没有提供api监听注册功能，但我们可以通过Proxy，完成对断言action 的结果收集

```Javascript
function createExpect(actual: unknown) {
    return new Proxy(expectLib(actual), {
        get(target: any, matcherName: any, receiver: any) {
            let matcher = Reflect.get(target, matcherName, receiver)

            return (...args: any[]) => {
                const testInfoImpl = getTestInfoImplState();
                if (!testInfoImpl) {
                    return matcher.call(target, ...args);
                }

                const step = testInfoImpl.addStep({
                    stepId: `expect:api@expect: ${actual || ''}  ${matcherName}  ${args[0] || ''}`,
                    startTime: Date.now(),
                    title: `断言:api@expect: ${actual || ''}  ${matcherName}  ${args[0] || ''}`,
                    status: 'pass'
                })

                const reportStepError = (error: Error) => {
                    step.error = error.toString()
                    step.status = 'fail'
                }

                try {
                    matcher.call(target, ...args);
                    step.endTime = Date.now()
                  } catch (e: any) {
                    reportStepError(e);
                    throw e
                  }
            }
        }
    });
}

export const expect = new Proxy(expectLib as any, {
    apply: function(target: any, thisArg: any, argumentsList: [actual: unknown]) {
      const [actual] = argumentsList;
      return createExpect(actual);
    }
  });
```


#### sft-inject
sft在录制用例时，插入到浏览器页面的脚本，用来设置页面事件监听器；同时提供一个toolbar菜单，菜单包含断言触发按钮，支持使用者触发断言动作：

![](https://files.mdnice.com/user/36775/e9ff9892-d86a-415a-b4e8-6d49a064d30b.png)


#### sft-ai-selector
sft在录制用例过程中，sft-ai-selector生成web事件元素、断言元素的选择器信息。里面包含一套选择器信息生成算法，用于生成最佳选择器（准确、可持久）

选择器是web自动化测试成功的关键因素，其中准确性、持久性尤为重要。一个好的选择器应该满足以下几个最佳实践：
- 优先考虑直接面向用户属性的（例如text、placeholder等）
- 优先考虑埋点属性
- 避免依赖dom层级

**sft-ai-selector结合以上原则，制定了实现策略：**
- 定义选择器权重，根据权重生成一系列选择器。
- 对用一个元素生成一个优先选择器集，避免独苗选择器因为页面改动失效导致元素命中失败
- 一套可在优先选择器集为空或者都失效情况下的兜底算法
- 一套持续自主训练算法，选择器具备自主学习能力，能在回放用例时针对已经失效的选择器信息进行自动更新，保持选择器持久有效性


**选择器生成流程概览**

![](https://files.mdnice.com/user/36775/98b33d80-7d9d-475c-9bed-ce14945f618f.png)


**兜底流程概览**

![](https://files.mdnice.com/user/36775/0c90c96a-09d1-43a6-84b3-518dc63f96e3.png)


**选择器自主训练流程概览**


![](https://files.mdnice.com/user/36775/33a78187-73b6-467e-af70-a073ea74bd74.png)

>自主训练算法会在sft后续迭代合入

#### sft-web
sft报告模块，输出用例集测试报告，用于排查用例错误

#### sft-client
sft客户端，用于可视化管理用例，并提供增量录制等能力增强功能
















