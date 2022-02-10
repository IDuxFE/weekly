#### 一、前言

编程格式规范已经是目前各大互联网公司都在遵循的一种规范，其中一些大厂有自己出版的规范，如：

- [京东凹凸实验室前端代码规范](https://guide.aotu.io/ )
- [腾讯前端代码规范](http://tgideas.qq.com/doc/index.html)
- [百度前端代码规范文档](https://github.com/ecomfe/spec)



整体规范大体可以分为两类：

##### （1）代码格式规范

		对于代码格式规范，我们通过`Eslint`+`Prettier`+`Vscode`配合进行处理，可以实现在保存代码时，自动规范化代码格式的目的。plainplainplainplainplainplainplainplainplain

##### （2）代码提交规范

		对于代码提交规范，我们通过使用`husky`来监测`git hooks`钩子，通过以下插件完成对应的配置：plainplainplainplainplain

1. `commitlint`：用于检测提交的信息
2. `lint-staged`：检查本次修改更新的代码，并自动修复并且可以添加到暂存区
3. `pre-commit`：`git hooks`的钩子，在代码提交前检查代码是否符合规范，不符合规范将不可被提交
4. `commit-msg`：`git hooks`的钩子，在代码提交前检查`commit`信息是否符合规范
5. `commitizen`：`git`的规范化提交工具，帮助你填写`commit`信息，符合[约定式提交](https://www.conventionalcommits.org/zh-hans/v1.0.0/)要求



#### 二、代码格式规范

##### （1）初始化一个项目

![](https://files.mdnice.com/user/20608/33a35360-6bbc-4b3e-bb67-2352af235bcc.png)



##### （2）安装`ESlint`

```js
npm install eslint --save-dev
```

安装完后，打开终端，执行命令`npx elint --init`根据自己的需求生成配置：

![](https://files.mdnice.com/user/20608/b4945c92-fcb6-45fc-b7e2-5b78d876a4f0.png)



然后会在根目录多了一个`.eslintrc.js`文件：

```js
module.exports = {
    "env": {
        "browser": true,
        "es2021": true,
        "node": true
    },
    "extends": "eslint:recommended",
    "parserOptions": {
        "ecmaVersion": 12,
        "sourceType": "module"
    },
    "rules": {}
};
  
```

测试一下，新增一个`index.js`文件，输入内容：

![](https://files.mdnice.com/user/20608/917297ce-f553-4d01-9515-7a73f3d93f70.png)

控制台执行命令：

```js
npx eslint index.js
```

![](https://files.mdnice.com/user/20608/54cb5b65-62d2-4731-91c4-1fe498f9b820.png)

出现这个报错，说明`eslint`生效了，我们在`.eslintrc.js`文件中添加规则`no-console`：

```plain
module.exports = {
    "env": {
        "browser": true,
        "es2021": true,
        "node": true
    },
    "extends": "eslint:recommended",
    "parserOptions": {
        "ecmaVersion": 12,
        "sourceType": "module"
    },
    "rules": {
    	"no-console": "off"
    }
};
```

> rules 可以配置很多规则，具体可以看[Eslint Rules](https://eslint.bootcss.com/docs/rules/)



然后`index.js`修改为：

![](https://files.mdnice.com/user/20608/81800ee1-f9e6-428a-95ce-855df55e1caa.png)

再次执行:

```js
npx eslint index.js
```

![](https://files.mdnice.com/user/20608/06f24f17-fcb0-4d20-915c-e98bd6d17f61.png)

控制台不再报错，并且可以正常输出变量。



##### （3）Vscode 安装 Eslint 插件

上面每次都需要执行`npx eslint xxx`才能校验，单个文件还好，多个文件就没法玩了。有没有办法在代码保存的时候自动执行`Eslint`，只需要安装`Eslint`插件即可：

![](https://files.mdnice.com/user/20608/ecf724b7-747c-4422-8289-650f25a6f042.png)

在`setting.json`文件中添加以下内容（安装完`Eslint`插件应该会自动添加，如果没有请手动补上）：

```js
"editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
}
```

编辑器就会出现报错提示了：

![](https://files.mdnice.com/user/20608/e93a4e1b-fd98-4988-8941-84808f471db3.png)

在`package.json`脚本中添加`eslint`命令，表示`Eslint`检测当前目录下所有文件：

```js
"scripts": {
    "eslint": "eslint ./**/*"
}
```

在根目录下新增`.eslintignore`文件，表示`Eslint`检测时排除掉以下文件：

```js
node_modules
package.json
package-lock.json
```

新增多一个`test.js`文件：

![](https://files.mdnice.com/user/20608/ba5ac4c1-2b69-4b20-b811-6fe82d8654b9.png)

然后在控制台输入命令：

```js
npm run eslint
```

![](https://files.mdnice.com/user/20608/aedafd23-9938-4c37-840e-1d918c2afce1.png)

多个文件都可以检测得到了。



##### （4）使用 Prettier 自动修复

`Eslint`可以帮助我们检测代码，我们需要使用`Prettier`自动修复代码，让我们的代码符合`Eslint`标准，在`Vscode`中安装`Prettier`插件：

![](https://files.mdnice.com/user/20608/731e9a22-8723-45c8-86f6-1167218a95c8.png)

在项目根目录中新建 `.prettierrc.js` 文件，该文件为 `perttier` 默认配置文件:

```json
modules.exports = {
  // 结尾不使用分号
  "semi": true,
  // 使用单引号
  "singleQuote": true,
  // 多行逗号分割的语法中，最后一行不加逗号
  "trailingComma": "none"
}

```

更多配置，可以查看[Prettier Options](https://prettier.io/docs/en/options.html#print-width)

在设置中，搜索 `save` ，勾选 `Format On Save`：

![](https://files.mdnice.com/user/20608/28fcb025-f344-421f-b09f-5d1acf4c988b.png)

然后在`setting.json`中增加配置：

```js
"[javascript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
},
```

**设置完需要重启，否则不会生效**



然后在`index.js`文件中输入一些不规范的代码：

![](https://files.mdnice.com/user/20608/901a18b6-269f-46ea-8e7d-1fdc5668ff50.gif)

上面的代码，保存后自动修复成了规范的代码，解决了以下问题：

1. 使用了双引号
2. 结尾使用了分号
3. 缩进不一致
4. 多行代码结尾使用了逗号



以上就是代**码格式规范**的全部内容。



#### 三、代码提交规范

完成上述的代**码格式规范**后，接下来当然是要把我们的代码提交上库了，团队中为了规范化提交，通常使用`git hooks`来防止一些不规范的代码被提交到仓库中，简单的列举一些常见的`hooks`钩子：

|                       |                                                              |                                                              |
| :-------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| Git Hook              | 调用时机                                                     | 说明                                                         |
| pre-applypatch        | `git am`执行前                                               |                                                              |
| applypatch-msg        | `git am`执行前                                               |                                                              |
| post-applypatch       | `git am`执行后                                               | 不影响`git am`的结果                                         |
| **pre-commit**        | `git commit`执行前                                           | 可以用`git commit --no-verify`绕过                           |
| **commit-msg**        | `git commit`执行前                                           | 可以用`git commit --no-verify`绕过                           |
| post-commit           | `git commit`执行后                                           | 不影响`git commit`的结果                                     |
| pre-merge-commit      | `git merge`执行前                                            | 可以用`git merge --no-verify`绕过。                          |
| prepare-commit-msg    | `git commit`执行后，编辑器打开之前                           |                                                              |
| pre-rebase            | `git rebase`执行前                                           |                                                              |
| post-checkout         | `git checkout`或`git switch`执行后                           | 如果不使用`--no-checkout`参数，则在`git clone`之后也会执行。 |
| post-merge            | `git commit`执行后                                           | 在执行`git pull`时也会被调用                                 |
| pre-push              | `git push`执行前                                             |                                                              |
| pre-receive           | `git-receive-pack`执行前                                     |                                                              |
| update                |                                                              |                                                              |
| post-receive          | `git-receive-pack`执行后                                     | 不影响`git-receive-pack`的结果                               |
| post-update           | 当 `git-receive-pack`对 `git push` 作出反应并更新仓库中的引用时 |                                                              |
| push-to-checkout      | 当 git-receive-pack`对`git push`做出反应并更新仓库中的引用时，以及当推送试图更新当前被签出的分支且`receive.denyCurrentBranch`配置被设置为`updateInstead`时 |                                                              |
| pre-auto-gc           | `git gc --auto`执行前                                        |                                                              |
| post-rewrite          | 执行`git commit --amend`或`git rebase`时                     |                                                              |
| sendemail-validate    | `git send-email`执行前                                       |                                                              |
| fsmonitor-watchman    | 配置`core.fsmonitor`被设置为`.git/hooks/fsmonitor-watchman`或`.git/hooks/fsmonitor-watchmanv2`时 |                                                              |
| p4-pre-submit         | `git-p4 submit`执行前                                        | 可以用`git-p4 submit --no-verify`绕过                        |
| p4-prepare-changelist | `git-p4 submit`执行后，编辑器启动前                          | 可以用`git-p4 submit --no-verify`绕过                        |
| p4-changelist         | `git-p4 submit`执行并编辑完`changelist message`后            | 可以用`git-p4 submit --no-verify`绕过                        |
| p4-post-changelist    | `git-p4 submit`执行后                                        |                                                              |
| post-index-change     | 索引被写入到`read-cache.c do_write_locked_index`后           |                                                              |

> 详细的 `hooks` 的介绍，可点击[这里](https://git-scm.com/docs/githooks)查看



最常用的其实就是上面表格中标粗的`pre-commit`和`commit-msg`：

1. `commit-msg`：可以用来规范化标准格式，并且可以按需指定是否要拒绝本次提交
2. `pre-commit`：会在提交前被调用，并且可以按需指定是否要拒绝本次提交



##### （1）使用 husky+commitlint 检查提交描述是否符合规范要求

我们使用以下工具帮我们规范化提交：

1. [commitlint](https://github.com/conventional-changelog/commitlint)：用于检查提交信息
2. [husky](https://github.com/typicode/husky)：是`git hooks`工具

首先要保证`npm`在`7.x`版本以上，如果不是，安装最新的`npm`版本：

```js
npm install npm -g
```

安装`commitlint`：

```js
npm install @commitlint/config-conventional @commitlint/cli --save-dev
```

在根目录创建 `commitlint.config.js` 文件：

```js
module.exports = {
  // 继承的规则
  extends: ['@commitlint/config-conventional'],
  // 定义规则类型
  rules: {
    // type 类型定义，表示 git 提交的 type 必须在以下类型范围内
    'type-enum': [
      2,
      'always',
      [
        'feat', // 新功能 feature
        'fix', // 修复 bug
        'docs', // 文档注释
        'style', // 代码格式(不影响代码运行的变动)
        'refactor', // 重构(既不增加新功能，也不是修复bug)
        'perf', // 性能优化
        'test', // 增加测试
        'chore', // 构建过程或辅助工具的变动
        'revert', // 回退
        'build' // 打包
      ]
    ],
    // subject 大小写不做校验
    'subject-case': [0]
  }
}

```

安装`husky`：

```js
npm install husky --save-dev
```

在控制台输入命令，生成`.husky`文件夹：

```js
npx husky install
```

![](https://files.mdnice.com/user/20608/ac5e6ce7-1ac1-44c8-9b1f-62fed0a98a79.png)



##### （2）使用 commit-msg 钩子规范化提交信息

添加 `commitlint` 的 `hook` 到 `husky`中，并指令在 `commit-msg` 的 `hooks` 下执行 `npx --no-install commitlint --edit "$1"` 指令：

```js
npx husky add .husky/commit-msg 'npx --no-install commitlint --edit "$1"'
```

![](https://files.mdnice.com/user/20608/3bf033aa-2947-4047-b730-302e656b20b6.png)

然后测试一下提交：

![](https://files.mdnice.com/user/20608/62ae9aab-e50d-4ad1-a073-df99ada7eafe.png)

发现`git hooks`的钩子`commit-msg`阻止了我们的提交，输入规范的信息后，再次尝试提交：

![](https://files.mdnice.com/user/20608/d20a7e4e-f58c-469e-a2b8-055f47822974.png)

代码成功提交，输出`log`也没有任何问题。



##### （3）使用 pre-commit 检测提交时代码规范

在**代码格式规范**章节中，讲解了如何处理代码格式问题，但是这样的一个格式处理问题，他只能够在本地进行处理，并且我们还需要 手动在 `Vscode` 中配置自动保存才可以。那么这样就会存在一个问题，如果有些人没有配置这个东西怎么办呢？他把一些乱七八糟的代码提交上来怎么办呢？

所以需要使用`pre-commit`来规避这种风险，我们期望通过 `husky` 监测 `pre-commit` 钩子，在该钩子下执行 `npm run eslint` 指令来去进行相关检测：

```js
npx husky add .husky/pre-commit "npm run eslint"
```

![](https://files.mdnice.com/user/20608/257b5a15-1326-4abc-b2c9-da66d9be75ae.png)



我们修改一下`test.js`的内容，然后再次提交：

![](https://files.mdnice.com/user/20608/10ce4ab2-e53a-4db0-9505-9c9ce6a73d94.png)

`pre-commit`阻止了我们的提交，因为存在不规范的代码



修改不规范代码后，再次提交：

![](https://files.mdnice.com/user/20608/68540f25-8beb-46d7-9d8d-eb9f7531d8bb.png)

修改后，代码成功提交。



##### （4）使用 lint-staged 自动修复格式错误

上面我们完成了代码的规范化提交，我们在`pre-commit`检测到我们代码中有不规范的地方，阻止了我们的提交，我们**手动修复**了不规范代码后，才能成功提交。那么这存在几个问题：

1. 我们只修改了个别的文件，没有必要检测所有的文件代码格式
2. 它只能给我们提示出对应的错误，我们还需要手动的进行代码修改
3. 修改后需要手动`git add .`

使用`lint-staged`可以解决这些问题，安装`lint-staged`：

```js
npm install lint-staged --save-dev
```

在`package.json`中添加配置：

```js
"lint-staged": {
    "./**/*.{js}": [
        "eslint --fix",
        "git add"
    ]
}

```

然后修改 `.husky/pre-commit` 文件：

```js
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

npx lint-staged
```

关闭`prettier`保存自动修复（否则无法测试）：

![](https://files.mdnice.com/user/20608/aee7b8f3-f359-49fb-aded-a79d1a429bdb.png)

修改`test.js`文件，增加一些不规范的代码：

![](https://files.mdnice.com/user/20608/f28a37c2-756e-4449-b789-dc3dbb8a7a34.png)

然后执行提交：

![](https://files.mdnice.com/user/20608/753f2e1f-eeea-47b9-b6ce-73908172257f.png)

这个过程`lint-staged`做了两个事情：

1. 如果符合规则：则会提交成功。
2. 如果不符合规则：它会自动执行 `eslint --fix` 尝试帮你自动修复，如果修复成功则会帮你把修复好的代码提交，如果失败，则会提示你错误，让你修好这个错误之后才能允许你提交代码。

如图所示，代码被修复了，并将`test.js`提交到了暂存区。



##### （5）用 commitizen 规范化提交代码

我们每次都需要手动输入`commit`信息很麻烦，并且容易造成不规范，我们可以使用`commitizen`来规范化提交代码

**全局安装**`commitizen`，注意，这里是全局安装，否则无法执行插件的命令：

```js
npm install commitizen -g
```

在项目内安装`cz-customizable`：

```js
npm install cz-customizable --save-dev
```

添加以下配置到 `package.json` 中：

```js
"config": {
    "commitizen": {
        "path": "./node_modules/cz-customizable"
    }
}
```

在根目录创建 `.cz-config.js` 自定义提示文件：

```js
module.exports = {
  // 可选类型
  types: [
    { value: 'feat', name: 'feat:     新功能' },
    { value: 'fix', name: 'fix:      修复' },
    { value: 'docs', name: 'docs:     文档变更' },
    { value: 'style', name: 'style:    代码格式(不影响代码运行的变动)' },
    { value: 'refactor', name: 'refactor: 重构(既不是增加feature，也不是修复bug)'},
    { value: 'perf', name: 'perf:     性能优化' },
    { value: 'test', name: 'test:     增加测试' },
    { value: 'chore', name: 'chore:    构建过程或辅助工具的变动' },
    { value: 'revert', name: 'revert:   回退' },
    { value: 'build', name: 'build:    打包' }
  ],
  // 消息步骤
  messages: {
    type: '请选择提交类型:',
    customScope: '请输入修改范围(可选):',
    subject: '请简要描述提交(必填):',
    body: '请输入详细描述(可选):',
    footer: '请输入要关闭的issue(可选):',
    confirmCommit: '确认使用以上信息提交？(y/n/e/h)'
  },
  // 跳过问题
  skipQuestions: ['body', 'footer'],
  // subject文字长度默认是72
  subjectLimit: 72
}
```

然后使用`git cz`代码`git commit`命令提交代码：

![](https://files.mdnice.com/user/20608/29c697fa-2c6e-4597-8bdf-9cfeb5073ea1.png)

选择对应的类型，依次执行下去，如需跳过，直接回车即可：

![](https://files.mdnice.com/user/20608/ea591e3d-4c39-4ace-83c7-a326923c8204.png)

输出`log`，一切正常：

![](https://files.mdnice.com/user/20608/1ed72e1d-020c-42d6-8fa5-e59db19ae540.png)



以上就是**代码提交规范**的全部内容。



#### 四、总结

标准化规范对于我们的开发流程，代码管理来说具有重要意义，遵循**代码格式规范**和**代码提交规范**，有利于我们更好的维护代码，减少错误，减低代码冲突等；每个公司都有自己的标准化规范，本文从**快速搭建**的角度，带你从`0`到`1`搭建一套符合`Eslint`标准、符合[约定式提交](https://www.conventionalcommits.org/zh-hans/v1.0.0/)要求的项目。其中，像`Eslint`和`Prettier`等工具可根据自己项目的需求自定义配置。



#### 五、参考

- [约定式提交](https://www.conventionalcommits.org/zh-hans/v1.0.0/)
- [Eslint](https://eslint.org/)
- [Prettier](https://prettier.io/)
- [commitlint](https://github.com/conventional-changelog/commitlint)
- [husky](https://github.com/typicode/husky)
- [commitizen](https://github.com/commitizen/cz-cli)
- [git hooks](https://git-scm.com/docs/githooks)