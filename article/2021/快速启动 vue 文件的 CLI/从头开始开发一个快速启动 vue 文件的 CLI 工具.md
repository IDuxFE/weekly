# 从头开始开发一个快速启动 vue 文件的 CLI 工具

## 背景

最近想做个类似 `vue-cli` 中 [vue serve](https://cli.vuejs.org/zh/guide/prototyping.html) 的功能，能够直接指定一个入口文件进行快速预览开发的需求。

简单来说，就是在命令行输入 `vue serve MyComponent.vue` 即可预览组件效果。

功能点：

1. 能够指定一个入口，快速启动一个开发服务器进行预览，且有热更新功能

2. 能够内置一些模板，比如支持内置自定义组件库，做到根据不同场景使用不同的模板启动

![流程图](https://files.mdnice.com/user/18515/6f724e25-06c7-421a-b4b8-3d830dcc86e3.png)

## 初始化项目

```bash
mkdir code-start

cd code-start

npm init -y
```

新建目录初始化，然后这里使用`typescript`配合[tsup](https://www.npmjs.com/package/tsup) 来开发我们的 cli 工具。

```json
// package.json
{
    //...省略一些其他参数以及依赖包
    "scripts": {
        "dev": "tsup-node src/cli.ts --watch src",
        "build": "tsup-node src/cli.ts --format esm,cjs"
    },

    "devDependencies": {
        "tsup": "^4.14.0"
    }
}
```

初始化安装完需要的依赖后，就可以使用 `dev` 命令来进行开发我们需要的功能了。

## cli 工具

`Node.js` 为我们提供了 `process.argv` 来读取命令行参数，如果命令行需要一些复杂的传参，则需要我们自行解析。不过解析参数这块社区已经有相关的库了，这里选择使用 [commander](https://www.npmjs.com/package/commander) 来处理。

```ts
// src/cli.ts

#!/usr/bin/env node

import { program } from 'commander';

program
    .command('serve <entry>', {
        isDefault: true,
    })
    .option('-t, --template <value>', '指定运行的模板', 'vue')
    .option('-p, --port <value>', '指定运行的端口', '2333')
    .action(async (entry, options) => {
        run(options.template, entry, {
            port: options.port,
        });
    });

program.parse(process.argv);
```

第一行的 `#!/usr/bin/env node`，`#!` 表示要指定脚本文件的解析程序，`/usr/bin/env` 表示要去哪里找解析程序，`node` 是解析程序的名字（表示这个要文件由 `Node.js` 来运行）。

这里我们创建了一个`serve`的命令，接收`entry`参数，并且作为默认命令使用。且支持传递`-t`, `-p` 来指定运行的模板与运行的端口。

```json
// package.json
{
    //...
    "bin": {
        "code-start": "./dist/cli.js"
    }
}
```

然后在`package.json`里面添加一项，这样我们就可以使用`npm link`命令来模拟安装本地模块。

这时候输入`code-start -h`我们的命令行程序就生效了。

```bash
code-start -h

Usage: cli <command> [options]

code start

Options:
  -V, --version            output the version number
  -h, --help               display help for command

Commands:
  serve [options] <entry>
  help [command]           display help for command
```

## 开发服务器

开发服务器这里选择了`vite`，对于快速启动开发 `vite` 显然是最合适的。

以一个最基础的 `vue2` 初始化项目为例，在最外层的`templates`目录新建`vue2`目录，目录结构大概如下：

```bash
# templates/vue2
├─src
│  └─main.ts
├─vite.config.ts
└─index.html
```

```ts
// main.ts
import App from '~entry';
import Vue from 'vue';

new Vue({
    render: h => h(App),
}).$mount('#app');
```

然后在`main.ts`中我们先使用一个`~entry`占位，后续替换为真实的入口。

```ts
// src/index.ts
import path from 'path';
import { createServer, InlineConfig } from 'vite';
import { findExisting } from './utils';
import entryPlugin from './plugins/entry';

/**
 *
 * @param {string} template 使用的模板
 * @param {string} entry 入口文件
 * @param {RunOptions} [options] 其他配置
 */
export async function run(
    template: string,
    entry: string,
    options?: {
        port?: number;
    },
) {
    const templateRoot = path.resolve(__dirname, `../templates/${template}`);

    // 查找模板根目录是否有vite配置文件
    let file = findExisting(templateRoot, ['vite.config.ts', 'vite.config.js']);

    const viteConfig: InlineConfig = {
        // 如果模板内有vite配置文件则使用它
        configFile: file ? path.join(templateRoot, `./${file}`) : false,

        // 解析前面`~entry`占位的入口插件
        plugins: [entryPlugin(entry)],
        server: {
            host: true,
            port: options?.port || 2333,
        },
    };

    // 创建开发服务器监听
    const server = await createServer(viteConfig);
    await server.listen();
}
```

然后通过命令行传入的参数，创建一个`viteDevServer`监听。

```ts
// src/plugins/entry.ts
import type { Plugin } from 'vite';
import path from 'path';

const PLACE_HOLDER_ENTRY = '~entry';
export default function entryPlugin(entry: string): Plugin {
    const context = process.cwd();
    const realEntry = path.resolve(context, entry);

    return {
        name: 'entryPlugin',

        resolveId(id) {
            if (id !== PLACE_HOLDER_ENTRY) {
                return;
            }

            return realEntry;
        },
    };
}
```

在`entryPlugin`中将 `~entry` 占位处理为我们命令行传入的真实入口。这一步其实也可以直接用 `alias` 来替换处理，但是后续我们还有其他要做的，所以用插件的方式解决。

这样我们一个简易的处理程序就已经完成了，不过其中模板的一些依赖我们是安装在我们的 cli 程序内的，比如上面 `vue2` 模板，就会依赖一些 `vue` , `vite-plugin-vue2` 等等。

## 依赖处理

在上面的模板目录中，添加一个`package.json`去声明需要的依赖版本。

```bash
# templates/vue

├─src
│  └─main.ts
├─vite.config.ts
├─package.json
└─index.html
```

```ts
program
    .command('serve <entry>', {
        isDefault: true,
    })
    .option('-t, --template <value>', '指定运行的模板', 'sfv')
    .option('-p, --port <value>', '指定运行的端口', '2333')
    .option('--choose-version', '选择依赖版本')
    .action(async (entry, options) => {
        if (options.chooseVersion) {
            await chooseDepsVersion(options.template);
        }
        await installDeps(options.template);
        run(options.template, entry, {
            port: options.port,
        });
    });
```

然后再给 CLI 程序新增一个版本依赖选择的配置功能，如果传递了`--choose-version`参数，提供一个交互操作，让使用者自行指定安装依赖的版本号。

```ts
import { createHash } from 'crypto';
import shell from 'shelljs';
import inquirer from 'inquirer';
import semver from 'semver';
import fs from 'fs-extra';
import path from 'path';

// 获取hash值
export function getDepHash(root: string): string {
    // lookupFile 是依次读取文件内容
    let lock = lookupFile(root, ['package-lock.json', 'yarn.lock', 'pnpm-lock.yaml']) || '';
    let pkg = lookupFile(root, ['package.json']) || '';
    return createHash('sha256')
        .update(pkg + lock)
        .digest('hex')
        .substr(0, 8);
}

// 安装依赖
export async function installDeps(template: string) {
    const root = path.resolve(__dirname, `../templates/${template}`);
    const metaPath = path.join(__dirname, '../_metadata.json');

    // 根据package.json与lock文件生成hash值
    const hash = getDepHash(root);
    let prevData: any = {};
    try {
        prevData = JSON.parse(fs.readFileSync(metaPath, 'utf-8')) || {};
    } catch (e) {}

    // 对比hash是否有变化，没有变化就跳过
    if (prevData[template] !== hash) {
        // 通过shell安装依赖
        const cmd = `yarn install --cwd ${root}`;
        shell.exec(cmd, {
            silent: true,
        });

        prevData[template] = hash;

        // 持久化写入文件，用于后续做hash对比
        fs.writeJSONSync(metaPath, prevData);
    }
}

// 选择依赖版本
export async function chooseDepsVersion(template: string) {
    const root = path.resolve(__dirname, `../templates/${template}`);
    const pkg = lookupFile(root, ['package.json']) || '';
    if (pkg) {
        const pkgObj = JSON.parse(pkg) || {};
        const deps = Object.keys(pkgObj.dependencies || {});

        // 通过inquirer创建一个交互程序
        // 指定每一项依赖的版本，版本号通过semver去校验
        const questions = deps.map((dep): inquirer.Question => {
            const currentVersion = pkgObj.dependencies[dep];
            return {
                type: 'input',
                name: dep,
                message: `输入 ${dep} 版本号（当前版本: ${currentVersion}）：`,
                filter: input => {
                    if (!input) {
                        return input;
                    }

                    return semver.valid(input) || input;
                },
                validate: input => {
                    if (input === '') {
                        return true;
                    }

                    return !!semver.valid(input);
                },
            };
        });
        const answers = await inquirer.prompt(questions);
        const writes: Record<string, string> = {};

        Object.keys(answers).forEach(dep => {
            if (answers[dep]) {
                writes[dep] = answers[dep];
            }
        });

        if (Object.keys(writes).length) {
            const targetFile = path.join(root, 'package.json');
            await fs.writeJson(
                path.join(root, 'package.json'),
                Object.assign(pkgObj, {
                    dependencies: { ...pkgObj.dependencies, ...writes },
                }),
                {
                    spaces: 2,
                },
            );
        }
    }
}
```

1. 选择依赖版本
   通过`inquirer`去创建一个命令行交互程序，输入每个包的版本号，接收到输入后，如果有有效输入，那么写入到`package.json`文件中。

2. 依赖安装检测
   运行模板前检查依赖是否安装好，通过`package.json`与`lock`文件组成的新旧`hash`值来对比是否有变化，如果有变化通过命令行执行`yarn install`安装。

这样子完成了每个模板自己的一些依赖版本，比如给自己自定义模板指定组件库版本，或者是想内置一些工具库等等。

## 扩展多种入口支持

前文说到，目前只支持本地的单一文件，如果我们想实现指定一个目录，使其目录下的所有文件都能够访问或者是指定一个远程地址作为入口，那么只需要扩展一下入口的插件`entryPlugin`即可。

```ts
import type { Plugin } from 'vite';
import path from 'path';
import fs from 'fs-extra';
import glob from 'fast-glob';
import got from 'got';

const PLACE_HOLDER_ENTRY = '~entry';
const ROUTER_MOD = '__router_mod__';
const REMOTE_MOD = '__remote_mod__.vue';

function genRoutes(realEntry: string) {
    const pattern = `${realEntry.replace(/[\/\\]/g, '/').replace(/\/$/, '')}/**/*.vue`;
    const routes = await glob(pattern)
        .map(file => {
            return `
            {
                path: '/${path.relative(realEntry, file).replace(/\.vue$/, '')}',
                component: () => import('${file}')
            }`;
        })
        .join(',');

    return `export default [
                ${routes}
            ]
        `;
}

interface PluginOptions {
    pattern?: string;
}

export default function entryPlugin(entry: string, options?: PluginOptions): Plugin {
    let realEntry = '';
    let isDirectory = false;
    let isRemoteEntry = isRemote(entry);

    if (!isRemoteEntry) {
        const context = process.cwd();
        realEntry = path.resolve(context, entry);
        isDirectory = fs.statSync(realEntry).isDirectory();
    }

    return {
        name: 'entryPlugin',

        resolveId(id) {
            if (id !== PLACE_HOLDER_ENTRY) {
                return;
            }

            // 如果是个远端地址，那么返回一个占位mod
            if (isRemoteEntry) {
                return REMOTE_MOD;
            }

            // 如果是个目录，那么使用路由模式
            if (isDirectory) {
                return ROUTER_MOD;
            }

            return realEntry;
        },

        async load(id) {
            // 如果是路由模式，扫描目录下所有.vue文件，生成导出路由表
            if (id === ROUTER_MOD) {
                return genRoutes(realEntry);
            }

            // 远端地址的话，fetch具体地址的文本内容返回
            if (id === REMOTE_MOD) {
                return got(entry).text();
            }
        },
    };
}
```

如果入口是以`http`或者`https`开头的远端地址，那么通过 vite plugin 的能力，获取到具体的代码返回。

如果入口是一个目录，那么扫描目录下所有`.vue`文件，生成导出路由表，后续在我们模板下`main.ts`处理的时候做下判断，如果返回的是一个数组，那么使用`vue-router`去初始化。

```ts
// templates/vue2/src/main.ts
function initEntry() {
    import('~entry').then(async ({ default: mod }) => {
        const Vue = (await import('vue')).default;

        // 如果entry导出的不是一个数组，那么作为入口正常处理
        if (!Array.isArray(mod)) {
            new Vue({
                render: h => h(mod),
            }).$mount('#app');
            return;
        }

        // 如果导出是一个数组，生成路由导航
        const Router = (await import('vue-router')).default;
        const router = new Router({
            mode: 'hash',
            routes: [
                ...mod,

                // 这里可以简单处理下生成一个列表页面用于导航
                // {
                //     path: '*',
                //     component: () => {},
                // },
            ],
        });

        Vue.use(Router);
        new Vue({
            render: h => h('router-view'),
            router,
        }).$mount('#app');
    });
}
initEntry();
```

## 总结

至此，我们的这个命令行工具基本就算完成了。作为前端开发人员，平时的一些小需求，可以借助 `Node.js` 的强大能力，开发一些适用的功能。社区中有很多实用的命令行工具可以供我们使用。

- [chalk](https://www.npmjs.com/package/chalk)：美化命令行的模块
- [debug](https://www.npmjs.com/package/debug)：日志打印模块
- [shelljs](https://www.npmjs.com/package/shelljs)：执行 shell 命令
- [inquirer](https://www.npmjs.com/package/inquirer)：命令行交互
- [semver](https://www.npmjs.com/package/semver)：npm 语义化版本控制
- [commander](https://www.npmjs.com/package/commander)：命令行工具
- [fast-glob](https://www.npmjs.com/package/fast-glob)：快速高效的 glob 解析库
- [got](https://www.npmjs.com/package/got)：一个 HTTP 请求库
