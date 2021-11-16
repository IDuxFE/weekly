#### 一、前言

一般团队中都是使用 `git` 来做版本管理，结合 `Eslint` + `Prettier` 这样的工具可以使得代码按照约定好的规则规范起来，最后在提交到 `git` 仓库中。提交到仓库的 `commit` 信息最好也要规范起来，不然其他人在阅读提交记录时也会非常头疼，就像下面这种情况一样：

![](https://files.mdnice.com/user/20608/f27808bc-f5b6-4342-aeb2-33bb4b542a61.png)

对于 `git` 提交规范来说，不同的团队可能会有不同的标准，推荐目前比较流行的 [Angular 团队规范](https://github.com/angular/angular.js/blob/master/DEVELOPERS.md#-git-commit-guidelines)延伸出的[Conventional Commits specification（约定式提交）](https://www.conventionalcommits.org/zh-hans/v1.0.0/) 。

约定式提交规范要求如下：

```js
< type > [optional scope]: < description >

    [optional body]

[optional footer(s)]
```

翻译过来就是：

```js
< 类型 > [可选 范围]: < 描述 >

    [可选 正文]

[可选 脚注]
```

举个栗子：

![](https://files.mdnice.com/user/20608/eda79b28-d27b-4b4f-bd4f-0353a714c399.png)

但是有个问题是，如果每次 `commit` 都这么写，着实有点痛苦，比较麻烦。所以就诞生了[Commitizen](<[github.com/commitizen/cz-cli](https://github.com/commitizen/cz-cli)>)这样的工具，只需要使用 `git cz` 命令代替 `git commit` 就可以帮我们书写 `commit` 信息，非常强大！

#### 二、准备

开始前，需要做一些准备，使用 `git init` 和 `npm init -y` 初始化一个 `git` 管理以及 `npm` 的项目：

![](https://files.mdnice.com/user/20608/b3cd84a3-a9db-419a-aa66-20046e3ea64b.png)

然后全局安装 `commitizen`

```js
npm install - g commitizen
```

在 `commitizen-demo` 文件夹中安装 `cz-customizable`

```js
npm install cz - customizable--save - dev
```

新建 `.gitignore` 文件，防止将 `node_modules` 文件提交：

![](https://files.mdnice.com/user/20608/f4dcc534-e4b0-443c-b34d-f175d14c4b34.png)

最后，项目目录结构如下：

![](https://files.mdnice.com/user/20608/d34d79b7-b1f8-4667-8be0-ff94a1b7c970.png)

#### 三、Commitizen 提交配置

和 `Eslint` 、 `Prettier` 工具一样， `commitizen` 在工作前，也需要进行配置。

添加以下配置到 `package.json` 中：

![](https://files.mdnice.com/user/20608/17e4cdaf-69bf-432f-8fed-69a2c28bd1f6.png)

在根目录下创建 `.cz-config.js` 文件，因为 `commitizen` 是符合 `CommonJS` 规范的，所以需要通过 `module.exports` 导出配置：

![](https://files.mdnice.com/user/20608/1b6b485d-8fd4-4c61-b217-bbf7328715f3.png)

根据**约定式规范**提交要求，我们先配置 `types` ，该属性是我们提交 `commit` 时的**类型**：

![](https://files.mdnice.com/user/20608/f89dbc9f-607b-4b87-830c-e35bcfbf6cb0.png)

然后配置步骤，步骤是为了告诉 `commitizen` 如何提交，简单来说就是第一步要什么，第二步要输入什么……：

![](https://files.mdnice.com/user/20608/a51f7a69-381f-4b66-a0f2-934a41d7dc17.png)

最后，可以约束一下文字的长度（看团队需求），比如 `subject` 约束文字长度为： `72` :

![](https://files.mdnice.com/user/20608/9d619659-694b-470f-8464-574cf0e264e0.png)

更多配置项可以参考[官网](https://github.com/commitizen/cz-cli)，这里不过多赘述。

#### 四、实战

完成配置后，我们来试下效果，创建一个 `README.md` 文件，然后使用 `git cz` 命令提交代码：

![](https://files.mdnice.com/user/20608/3145a318-1a3d-4767-a2ff-e5b71b7daa91.png)

可以看到，控制台输出了对应的类型选择，这里选择**新功能**，回车下一步：

输入修改范围，回车下一步：

![](https://files.mdnice.com/user/20608/f89ba31f-6bac-449c-b3aa-610505339640.png)

下面的步骤依次输入完，如果没有可以按回车跳过，最后输入 `yes` 确认：

![](https://files.mdnice.com/user/20608/a477e6ee-4aa9-4c38-947a-d619a7463e86.png)

输入 `git log` 查看提交记录：

![](https://files.mdnice.com/user/20608/cda2d7e4-eed2-4f51-afc8-e3f5c3be39ee.png)

这样，一个标准的 `commit` 信息就提交了。

#### 五、查看 changeLog

有时候，我们想要查看 `commit` 的 `change log` 信息，可以使用工具 `conventional-changelog`

在项目安装 `cz-conventional-changelog` 和 `conventional-changelog-cli`

```js
npm install conventional - changelog conventional - changelog - cli--save - dev
```

在 `package.json` 增加脚本信息：

![](https://files.mdnice.com/user/20608/79363948-b1eb-47f4-9c84-d3494d6c9353.png)

运行脚本：

```js
npm run git: log
```

![](https://files.mdnice.com/user/20608/0c1ae36b-6d1b-4ce1-a15e-30948a843011.png)

就可以看到刚才提交的 `commit` 信息了

#### 六、总结

`commitizen` 已经是目前作为 `git` 提交规范中最优秀的工具之一了。在团队开发中，一般结合 `Eslint` 和 `Prettier` 工具一起使用，养成良好的代码规范是每个程序员的必修课！

#### 七、参考

* [Angular 团队规范](https://github.com/angular/angular.js/blob/master/DEVELOPERS.md#-git-commit-guidelines)
* [Conventional Commits specification（约定式提交）](https://www.conventionalcommits.org/zh-hans/v1.0.0/)
