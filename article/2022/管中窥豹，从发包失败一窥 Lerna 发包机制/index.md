# 管中窥豹

## 场景复现

事情是这样子的，林哈哈 前端同学修改了组件库里的某两个 pkg（我们暂且把它称为 A、B），想要往 NPM 仓库发个包，如往常一样，本应 version、publish 一气呵成，结果就出现了如下这个问题。

![](https://files.mdnice.com/user/28073/a51485cf-14a9-4119-bc87-e1201b446b42.png)

哈哈 同学怀疑是否自己本地环境有问题，于是让我也试了一波，结果问题还是一样。虽然 version 失败了，但我还是抱着侥幸心理，想看看能不能继续往下走了 publish 流程。

![](https://files.mdnice.com/user/28073/2ad68be1-cb2e-416a-bd14-1a29353ec4cc.png)

结果显示发布成功了 A pkg，然后屁颠屁颠在项目上更新了 A pkg，npm install 一顿操作，显示 NPM 不存在 B pkg 的 1.5.0 版本？？？

![](https://files.mdnice.com/user/28073/d0b15450-0d9c-4a15-a306-f7d2332e02e2.png)

## 提出疑问&大胆假设

经过上述的场景复现，可能懂王老铁们一眼就看出来了问题，那就可以忽略这一环节的内容了。不明觉厉的小可爱可以继续把你的手指头往下滑起来。

![](https://files.mdnice.com/user/28073/25ae6548-d6a9-408a-b87d-1fd328481c1d.png)

看到这里，我们应该会有几个疑问：

> 1. Lerna version 操作为啥会出现 tag xxx already exists 的提示？
> 2. Lerna version 失败了为啥还能继续 publish?

顺着问题走，既然提示存在了 1.5.0 版本的 tag，那么就到组件库的 Gitlab 上查了下 B pkg 的 CHANGLOG 标签，神奇的发现了 B pkg 的这个 tag 居然在 8 个月之前就已经存在了。

![](https://files.mdnice.com/user/28073/c281535b-8198-4039-8a6e-3ed88bdf11a5.png)

那么问题来了，既然 8 个月之前就存在了这个 tag，根据 lerna 的发包机制，pkg 版本号理应是跟 tag 一一对应的。

然而现在我们这个包的版本为何是 1.4.5，它是如何降了版本的呢？是人性的扭曲，还是道德的沦丧？不妨大胆提出以下假设：

> 这个包之前就有人升级过这个版本，在仓库上面已经打上了对应的 tag，然后因为某些原因，revert 了这个提交，然后重新执行了 Lerna 的发包流程，重新给这个包打上了相应的 tag，并覆盖掉了 NPM 仓库上这个包的版本。

## 小心求证

说到 Git 的版本问题，那就不得不继续在 Gitlab 瞄一眼这个 pkg 的提交历史了，果然，的确有人在很久之前就已经发过这个版本。

![](https://files.mdnice.com/user/28073/cd3ff7c0-33d1-48ad-99b0-4dd0563ccfcd.png)

但这个还没发解释为何现在的版本降回到了 1.4.5，顺着提交历史继续，惊喜出现了，果然这个提交是被 revert 了。

![](https://files.mdnice.com/user/28073/459dae66-db63-4343-b9e6-010220b23519.png)

到这里，已经解释了为何我们现在的 B pkg 的版本号（1.4.5）会低于 8 个月之前的 1.5.0 了。那么回到最初的问题，执行 Lerna version 出现 tag already exists 也就可以解释得通了。

我们这次要升级的版本是基于一个 feature 提交，版本号就从 1.4.5 升级到了 1.5.0，Lerna version 在处理版本信息需要在 Gitlab 上打 1.5.0 的 tag，但是由于前面执行 revert 操作的同学，没有顺手把以前对应的 tab 清除掉，所以 Gitlab 上会存在无用的 tag 信息，刚好跟后面要发布版本打的 tag 重复了。

![](https://files.mdnice.com/user/28073/b9c544ee-d2b6-4aad-860c-e92ceac5d78b.png)

## 问题闭环

既然知道了问题的来龙去脉，解决问题也就是动动手指的事了。

> 方案 1. 大胆的把 Gitlab 仓库中的无用 tag 删除掉（推荐使用，无用的信息就干掉吧）。
> 方案 2. 手动修改这个包的 pkg.json 版本号，跳过已存在的 tag 的版本继续升级。

# 具体的豹子，问题的延伸

Lerna 作为风靡前端社区的 Monorepo 包管理的工具，管中窥豹，从上面的一个实际场景出发，我们来打开这个黑盒，看看它具体做了神马我们看不见的东西：

咱们不贴详细的 API ，就挑两个关键的来简单了解一下大概的工作流程（version & publish）。

## lerna version

### 它做了什么（流程）

这是官网给的 lerna version 指令干的活：

![](https://files.mdnice.com/user/28073/4251fce0-64fb-455d-9cbf-11bffd2b3ecb.png)

英文好的老铁依然可以忽略这一段，我按自己的理解给简单翻译翻译：

![](https://files.mdnice.com/user/28073/064a0f19-4efa-4429-8aa5-0ffcc1db904f.png)

> 1. 识别出上一个 tag 版本以来发生了变更的 pkg 包。
> 2. 给识别出来的变更包做出新的版本提示，让用户选择是否创建对应的版本包（可使用 --conventional-commits 参数自动化跳过选择）。

![](https://files.mdnice.com/user/28073/0c6eddcf-f50a-45d4-9824-1b5602bd871b.png)

> 3. 修改对应包的 pkg 元数据版本，来对应新的版本号，然后执行 Lerna 项目中根目录 pkg.json 定义的 script 生命周期中的指令。

![](https://files.mdnice.com/user/28073/8f00731f-e0c2-4b6b-812d-125cf34ff97d.png)

> 4. 然后把变更版本后的一些变更文件，提交 git commit 并 push 到远程仓库中。

![](https://files.mdnice.com/user/28073/106d9257-7fd8-4b08-82ea-a5b33eb2ef53.png)

### 它是怎么实现的（源码）

咱们先看一下 Lerna 里面 version 模块的目录结构，它本质上其实是一个 Node.js 的脚手架。

```
├── CHANGELOG.md
├── README.md
├── __tests__  // 单元测试相关
├── command.js // cli 脚手架文件，定义了 version 操作的一堆 Options 选项
├── index.js   // version 入口文件，执行逻辑相关的函数文件
├── lib        // version 具体使用到的一些工具函数
│   ├── __mocks__
│   ├── create-release.js
│   ├── get-current-branch.js
│   ├── git-add.js
│   ├── git-commit.js
│   ├── git-push.js
│   ├── git-tag.js
│   ├── is-anything-committed.js
│   ├── is-behind-upstream.js
│   ├── is-breaking-change.js
│   ├── prompt-version.js
│   ├── remote-branch-exists.js
│   └── update-lockfile-version.js
└── pkg.json
```

从入口 index.js 开始，看里面具体做了什么，这里面是一个 VersionCommand 类，继承自 command 模块（咱们今天主要讨论的是 versionm 模块，command 模块先忽略，后面有时间再详细分析这块内容），核心逻辑逻辑主要是：属性配置、初始化、指令执行。
```
class VersionCommand extends Command {
  configureProperties() {
    // version 参数配置项的校验和整合
  }

  // 初始化
  initialize() {
    // 根据参数配置，进行一些初始化配置
  }

  execute() {
    // 调用其他方法执行对应的指令
  }

  // 其他一些执行操作函数，如：
     更新版本号、Git 的 commit 提交和打 tag 、远程 push 推送等
  ...
}

```

#### 参数配置项初始化
参数配置比较简单，咱们直接贴源码，把一些影响阅读的非核心代码干掉，就剩这么点：
```
 configureProperties() {
    // 1、从父类中继承参数，并设置默认值
    const {
      amend,
      commitHooks = true,
      gitRemote = "origin",
      gitTagVersion = true,
      granularPathspec = true,
      push = true,
      signGitCommit,
      signGitTag,
      forceGitTag,
      tagVersionPrefix = "v",
    } = this.options;

    // 2、设置当前类的全局配置参数
    this.gitOpts = {
      amend,
      commitHooks,
      granularPathspec,
      signGitCommit,
      signGitTag,
      forceGitTag,
    };

    // 3、判断用户执行指令时是否带 --exact 参数，设置版本前缀
    this.savePrefix = this.options.exact ? "" : "^";
  }

```

#### 版本信息初始化
同理，咱们继续看初始化做了哪些东西：

```
initialize() {
    if (!this.project.isIndependent()) {
      // 1、判断当前包是否存在依赖其他包的情况
      this.logger.info("current version", this.project.version);
    }

    if (this.requiresGit) {
      // 2、对检查 git 操作相关配置参数，没有则给出 info 提示
    } else {

    }
    
    // 3、收集需要更细的包信息
    this.updates = collectUpdates();
    
    // 4、定义需要执行 script 生命周期函数（包括包的和全局的 script 生命周期），供后续执行
    this.runPackageLifecycle = createRunner(this.options);
    this.runRootLifecycle = /^(pre|post)?version$/.test(process.env.npm_lifecycle_event)
      ? (stage) => {
          this.logger.warn("lifecycle", "Skipping root %j because it has already been called", stage);
        }
      : (stage) => this.runPackageLifecycle(this.project.manifest, stage);
    
    // 5、收集需要更新包信息的版本，定义一个任务数组，供后续执行
    const tasks = [
      () => this.getVersionsForUpdates(),
      (versions) => this.setUpdatesForVersions(versions),
      () => this.confirmVersions(),
    ];
    
    // 6、将前面定义的任务数组创建一个依赖上一个任务执行结果的异步队列导出
    return pWaterfall(tasks);
  }
```

#### 执行版本更新指令
这里要做的其实就是我们前面流程提到的，收集包版本更新信息、给变更文件添加到本地缓存区并打上 tag，最后 push 到远端仓库的过程，至于具体怎么提交、打 tag、更新版本等，感兴趣的老铁可以去 Github 上看对应的实现。

```
execute() {
    // 指定执行任务队列，把更新的包版本信息推到任务队列中
    const tasks = [() => this.updatePackageVersions()];
    
    // 下面的几个判断也比较好理解，就不展开说明了
    if (this.commitAndTag) {
      tasks.push(() => this.commitAndTagUpdates());
    } else {
      this.logger.info("execute", "Skipping git tag/commit");
    }

    if (this.pushToRemote) {
      tasks.push(() => this.gitPushToRemote());
    }

    if (this.releaseClient) {
      this.logger.info("execute", "Creating releases...");
      tasks.push(() =>
        createRelease(
          this.releaseClient,
          { tags: this.tags, releaseNotes: this.releaseNotes },
          { gitRemote: this.options.gitRemote, execOpts: this.execOpts }
        )
      );
    } else {
      this.logger.info("execute", "Skipping releases");
    }

    return pWaterfall(tasks).then(() => {
      if (!this.composed) {
        this.logger.success("version", "finished");
      }

      return {
        updates: this.updates,
        updatesVersions: this.updatesVersions,
      };
    });
  }
```

## lerna publish

囿于篇幅，咱们下次再来一篇文章专门的介绍 publish 的内容吧。

# 结语

本次分享只是源于日常开发当中遇到的一个小问题，初心只是想要弄明白为何会发包失败，然后顺带瞄了下 Lerna 的源码，可能理解得不够准确，还请各位看官不吝赐教！！！

![](https://files.mdnice.com/user/28073/245c5f28-4348-4b64-9d8b-d0ac15bfb875.png)


#### *参考资料*：

*[Lerna version 源码](https://github.com/lerna/lerna/tree/main/commands/version#readme)*

*[约定式提交规范](https://www.conventionalcommits.org/zh-hans/v1.0.0 "约定式提交规范")*

*[NPM 版本号规范](https://docs.npmjs.com/misc/config#save-prefix "NPM 版本前缀规范")*
