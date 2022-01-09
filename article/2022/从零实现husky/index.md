### 从零实现husky

目前的项目配置 `husky+commitlint+prettier`做一些规范化格式校验，但是之前从来没专门去了解过是怎么实现的。最近突然对 `husky` 实现产生好奇，于是自己尝试进行实现一个`myhusky`工具。

### 前置知识点

正式开始前，对可能需要的知识点进行简单介绍。

#### npm是如何处理 scripts 字段的

在项目开发过程中，都会往 `package.json` 添加一些项目使用到的 `scripts` 。不过 `scripts` 中也有一些隐藏的功能。

##### Pre & Post Scripts

```json
{
  "scripts": {
    "precompress": "{{ executes BEFORE the `compress` script }}",
    "compress": "{{ run command to compress files }}",
    "postcompress": "{{ executes AFTER `compress` script }}"
  }
}
```

配置了上述 `scripts`， 当触发`npm run compress`时， 会按顺序触发 `precompress`， `compress`， `postcompress` 脚本。即实际命令类似于`npm run precompress && npm run compress && npm run postcompress`。

##### Lift Cycle Scripts

- 除了上面的 `Pre & Post Scripts`，还有一些特殊的生命周期在指定的情况下触发，这些脚本触发在除 `pre<event>`, `post<event>`, `<event>` 之外。
- `prepare`， `prepublish`， `prepublishOnly`， `prepack`， `postpack` ，以上几个就是内置的生命周期钩子。

以 `prepare` 为例， `prepare` 是从 `npm@4.0.0` 开始支持的钩子，主要触发时机在下面的任意一个流程中：

- 在打包之前运行时
- 在包发布之前运行时
- 在本地不带任何参数运行`npm install`时
- 运行`prepublish`之后，但是在运行`prepublishOnly`之前
- 注意：如果通过 git 安装的包包含`prepare`脚本时，`dependencies`，`devDependencies`将会被安装，此后`prepare`脚本会开始执行。

### 功能点

介绍完涉及到的知识点后，接下来仿照 `husky`, 梳理需要实现的功能点:

- 支持命令行调用 `myhusky`, 
  - 支持安装命令：`myhusky install [dir] (default: .husky)`, 
  - 支持添加 `hook` 命令：`myhusky add <file> [cmd]`
- `install`命令手动触发, 触发后主要逻辑有：
  - 创建对应的`git hooksPath`文件夹；
  - 调用 `git config core.hooksPath` 设置项目 `git hooks`路径【即设置 `git hook` 脚本存放目录， 触发时往此目录上找对应的脚本执行】
- `add` 命令需要两个参数， 一个是传入新增的文件名，另外一个是对应触发的命令。
  -  如 `myhusky add .myhusky/commit-msg 'npx commitlint -e $1' `, 添加 `commit-msg` 钩子, 在进行 `commit` 操作时会触发 `npx commitlint -e $1` 命令。

### 测试先行

结合功能点，先整理需要测试的内容

- 测试 `install` 命令。
  - 执行成功后，工作目录下会生成对应存放 `git hooks` 的文件夹；
  -  `.git/config` 文件内部会新增多一个配置 `hooksPath = .myhusky`
- 测试 `add` 命令。
  - 往 `git hooks` 文件夹新增一个 `commit-msg hook`，用于在进行 `git commit` 时触发本地检验；
  - 往 `commit-msg hook` 写入的检验内容为 `exit 1`， 失败退出；
  - 进行 `git commit` 操作，此时 `git commit` 操作会提交失败，失败状态码为1；

### 测试用例的简单实现

`husky` 库是通过 `shell 脚本`  进行测试的，主要流程是：

- `npm run test` 实际上执行 `sh test/all.sh`
- `all.sh` 的操作有：执行 `build 命令`  ，接着在执行第一个脚本时会再调用 `functions.sh` ，`function.sh` 里面定义了 `setup` 初始化函数，以及自己实现了 `expect`, `expect_hooksPath_to_be` 用于做测试验收的工具函数。

而我们主要通过 `jest` 进行单测实现：

```javascript
const cp = require('child_process')
const fs = require('fs')
const path = require('path')
const { install, add } = require('../lib/index')
const shell = require('shelljs')
const hooksDir = '.myhusky'
const removeFile = p => shell.rm('-rf', p)
const git = args => cp.spawnSync('git', args, { stdio: 'inherit' });
const reset = () => {
  const cwd = process.cwd()
  const absPath = path.resolve(cwd, hooksDir)

  removeFile(absPath)
  git(['config', '--unset', 'core.hooksPath'])
}

beforeAll(() => {
  process.chdir(path.resolve(__dirname, '../'))
})
// 每个单测都会执行的函数
beforeEach(() => {
  reset()
  install(hooksDir) // 执行 install
})
// 测试 install 命令
test('install cmd', async () => {
  // install 逻辑在每个单测开始前都会执行, 所以当前测试用例只需要对结果进行检测
  const pwd = process.cwd()
  const huskyDirP = path.resolve(pwd, hooksDir)
  const hasHuskyDir = fs.existsSync(huskyDirP) && fs.statSync(huskyDirP).isDirectory()
  // 读取 git config 文件信息
  const gitConfigFile = fs.readFileSync(path.resolve(pwd, '.git/config'), 'utf-8')
  // git config 文件需要含有 hooksPath配置
  expect(gitConfigFile).toContain('hooksPath = .myhusky')
  // 期望含有新创建的 git hooks 文件夹
  expect(hasHuskyDir).toBeTruthy()
})
// 测试 add 命令
test('add cmd work', async () => {
  const hookFile = `${hooksDir}/commit-msg`
  const recordCurCommit = git(['rev-parse', '--short', 'HEAD'])
  // 往commit-msg文件写入脚本内容， exit 1
  add(hookFile, 'exit 1')
  git(['add', '.'])
  // 执行 git commit 操作触发钩子
  const std = git(['commit', '-m', 'fail commit msg'])
  // 查看进程返回的状态码
  expect(std.status).toBe(1)
  // 清除当前测试用例的副作用
  git(['reset', '--hard', recordCurCommit])
  removeFile(hookFile)
})
```

### 实现

- 配置 `package.json` , 添加为可执行的命令 `myhusky` 以及对应命令触发的文件 `./lib/bin.js`

  ```json
  {
    "name": "myhusky",
    "version": "1.0.0",
    "description": "",
    "main": "index.js",
    "scripts"
    "bin": {
      "myhusky": "./lib/bin.js"
    }
  }
  ```

- 代码实现：

  ```javascript
  #!/usr/bin/env node
  // lib/bin.js
  const { install, add } = require('./')
  const [cmdType, ...args] = process.argv.slice(2);
  const ln = args.length;
  const cmds = { // 命令集合
  	install,
  	add
  }
  const cmd = cmds[cmdType]
  
  if (cmd) {
  	cmd(...args)
  }
  ```

  ```javascript
  // lib/index.js
  const cp = require('child_process')
  const git = (args) => cp.spawnSync('git', args, { stdio: 'inherit' });
  const fs = require('fs');
  const path = require('path');
  const cwd = process.cwd();
  
  // 初始化安装命令
  exports.install = function install (dir = '.myhusky') {
  	const huskyP = path.resolve(cwd, dir)
  	if (!fs.existsSync(path.join(cwd, '.git'))) {
  		throw new Error('cannot find .git directory');
  	}
  
  	// 创建hooksPath文件夹
  	fs.mkdirSync(huskyP)
  
  	// git config core.hooksPath dir 设置git hooks触发文件目录
  	git(['config', 'core.hooksPath', dir])
  }
  
  // 添加hook命令
  exports.add = function add (file, cmd) {
  	// 追加命令函数
  	// 往git hooks目录下追加 `commit-msg` 脚本
  	fs.writeFileSync(
  		path.join(cwd, file), `#!/bin/sh
  ${cmd}`, { mode: 0o0755 }
  	)
  }
  ```

### 测试

- 项目中 `npm link`， 将 `myhusky` 包添加到npm全局下

- 进入一个测试项目， `npm link myhusky`， 将本地 `myhusky` 依赖安装进项目内

- 测试项目安装 `commitlint` , 并添加 `commitlint.config.js` 配置

  ```javascript
  // commitlint.config.js
  module.exports = {
      rules: {
  		'body-leading-blank': [1, 'always'],
  		'body-max-line-length': [2, 'always', 100],
  		'footer-leading-blank': [1, 'always'],
  		'footer-max-line-length': [2, 'always', 100],
  		'header-max-length': [2, 'always', 100],
  		'subject-case': [
  			2,
  			'never',
  			['sentence-case', 'start-case', 'pascal-case', 'upper-case'],
  		],
  		'subject-empty': [2, 'never'],
  		'subject-full-stop': [2, 'never', '.'],
  		'type-case': [2, 'always', 'lower-case'],
  		'type-empty': [2, 'never'],
  		'type-enum': [
  			2,
  			'always',
  			[ // 支持的类型枚举
  				'build',
  				'chore',
  				'ci',
  				'docs',
  				'feat',
  				'fix',
  				'perf',
  				'refactor',
  				'revert',
  				'style',
  				'test',
  			],
  		],
  	}
  };
  ```

- 测试项目中，调用 `npx myhusky install` ,  `npx myhusky add .myhusky/commit-msg 'npx commitlint -e $1'`

  【这里$1代表什么呢? 】这个是在执行 `shell` 传入的参数,  `$1` 具体的值是 `.git/COMMIT_EDITMSG`, 这个文件记录着**当前的commit信息**, 将这个作为 `commitlint -e`的配置项传入, `commitlint` 会根据这个配置项去读取 `commit信息` 根据配置的规则进行验证, 如果失败则退出导致 `commit` 失败。

- 至此, 进行测试的配置已完成，添加一个文件并进行提交 `git commit -m 'try: add pkg'`， 会发现拦截校验不通过

校验不通过：

<img src="https://files.mdnice.com/user/19436/7063b6a2-9458-44e3-993e-cd8d7e045b33.png" alt="invalid-commit" style="zoom:150%;" />

校验通过：

<img src="https://files.mdnice.com/user/19436/cde53d4a-9d12-468a-8a4d-2cbf17d660eb.png" alt="valid-commit" style="zoom: 150%;" />

### 总结

至此， 一个简单的`git hook`工具大致功能已经完成。通过了解`husky`的内部实现，并且自己手写实现一个仿照的例子，从中能够学习到：

- `git hooks` 是怎么起到作用的；
- `scripts` 一些隐藏的钩子；
- `husky` `commitlint` 是怎么配合实现的；

### 参考资料

- 代码地址：https://github.com/manchixue/mini-husky
- [scripts lift cycle hook](https://docs.npmjs.com/cli/v6/using-npm/scripts)
- [Git Hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks)
