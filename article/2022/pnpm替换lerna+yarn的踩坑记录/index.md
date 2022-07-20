## pnpm替换lerna+yarn的踩坑记录

如果有使用`monorepo`的需求，`lerna+yarn`会是很多开发者的选择，然而在实际开发中，`lerna`的很多功能我们并不需要，同时它也存在着一定的上手学习成本，而且 `yarn`也会存在一些问题比如多个项目会重复安装依赖、幽灵依赖等，这时候不妨考虑用更加轻便高效的`pnpm`

### pnpm

`pnpm`跟`yarn/npm`一样是一款包管理工具，不同于`yarn/npm`的扁平化的依赖管理机制，`pnpm`采用软硬链接的机制实现依赖的管理和引用，不仅安装速度极快，高效利用磁盘空间，也解决了幽灵依赖的问题，而且`pnpm workplace` 提供了`monorepo`的支持，我们完全可以用`pnpm`替换`lerna+yarn`

#### 如何替换

1. 全局安装`pnpm`
```bash
 npm install -g pnpm    
```
2. 先配置`pnpm workspace`，在项目根目录新建一个`pnpm-workspace`yaml 输入如下代码
```js
packages:

 \- 'packages/*'   
```
3. 删除项目中已有的`lerna.json`, 用`pnpm`的 workspace 来替代`lerna`实现`monorepo`的包管理方式
4. 执行`pnpm install`安装项目需要的依赖（安装过程中可能出现卡住的情况，可能是因为在内网环境下源不稳定重新安装即可） 安装成功后如果是Windows环境会在你当前项目所在硬盘的根目录下新增了一个`.pnpm-store`文件夹，刚刚装的依赖文件都在里面
5. 把项目中的`package-lock.json`和 `yarn.lock`删掉，所有锁都由 `pnpm-lock.yaml`来提供，至此`pnpm`的替换工作结束
6. 替换掉脚本命令，与`yarn`相关的命令替换为: pnpm <command> 或者 pnpm run <command>
7. 调整 `pipeline`、以及`Dockfile`或者其他`CI/CD`配置文件里面的依赖安装命令
#### 遇到的坑和解决方式
##### 1、vite打包失败
```bash
Error:[vite]:Rollup failed to resolve import "vue-property-decorator" from "../shared/xxx/index.ts".This is most likely unintended because it can break your application at runtime.If you do want to externalize this module explicitly add it tobuild.rollupoptions.external'
```

执行打包指令后会遇到一些导入依赖失败的问题，这是因为`pnpm`的依赖引用是根据`package.json`中是否有正确声明依赖来实现的，比如你要在`packages/shared`里声明`vue-property-decorator`,那必须要在`packages/shared/package.json`中有声明这个依赖，而现在`shared`中并没有`package.json`文件，所以无法引入。 所以建议去除每个子项目的共同依赖，比如`vue，lodash`等，然后统一放入顶层的`package.json`内
##### 解决办法
1. 在各子模块的`package.json`搜索导入失败的依赖XXX，确认它们版本号是否统一，如果是的话，直接执行`pnpm add XXX -w`在根目录添加这个依赖再`pnpm rm XXX -F {package}`, 如果版本不一致就不要rm这一步
2. 重复第一步，直到所有有问题的依赖重新安装好 

##### 2、vue页面空白

初步判断是由于`import { Vue } from 'vue-property-decorator'`; 引入异常导致的 在`vite.config`中设置了`vue`的别名, 在引入`vue`的时候没有按`pnpm`规则走软硬链接的方式，而是去直接读取了根目录下`node_modules/vue/dist/vue.js`，所以没有正确取到真正的存在`store`下的`vue`依赖包

```js
resolve: {
    alias: [
        ...
       { find: 'vue', replacement: lernaRootPath('node_modules/vue/dist/vue.js') },
   ],
},
```
##### 解决办法:
* 修改`vue`指向， 把`vue`指向`vue/dist/vue.esm.js`
```js
resolve: {
    alias: [
        ...
        { find: 'vue', replacement: 'vue/dist/vue.esm.js' },
   ],
},
```
##### 3、部分依赖需要预编译

因为`Vite`的`DevServer`是基于浏览器的`Natvie ES Module`实现的，所以对于使用的依赖如果是`CommonJS`或`AMD`的模块，则须要进行模块类型的转化，比如依赖中使用到了`echarts4.6.0`，它属于`CommonJS`的模块，需要进行预编译对它进行模块类型的转化
```js
// vite.config.ts
const { merge } = require('webpack-merge');

export default defineConfig(({ mode }) => {
    return merge(common({ mode }), {
        optimizeDeps: {
            entries: ['./vite/entry.ts'],
            include: ['需要预编译的依赖'],
        },
        ....
    });
});
``` 
