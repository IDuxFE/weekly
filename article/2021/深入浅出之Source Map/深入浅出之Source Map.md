#### 一、什么是 Source Map

通俗的来说， `Source Map` 就是一个信息文件，里面存储了代码打包转换后的位置信息，实质是一个 `json` 描述文件，维护了打包前后的代码映射关系。关于 `Source Map` 的解释可以看下[Introduction to JavaScript Source Maps](https://www.html5rocks.com/en/tutorials/developertools/sourcemaps/)。

我们线上的代码一般都是经过打包的，如果线上代码报错了，想要调试起来，那真是很费劲了，比如下面这个例子：

使用打包工具 `Webpack` ，编译这一段代码

```js
console.log('source map!!!')
console.log(a); //这一行肯定会报错
```

浏览器打开后的效果：

![](https://files.mdnice.com/user/20608/6d6bda5e-99ba-4d10-8cad-4d65fb83b271.png)

点击进入报错文件之后：

![](https://files.mdnice.com/user/20608/f7ec77ce-6cc6-4af1-a84e-cb00d13b1723.png)

这根本没法找到具体位置以及原因，所以这个时候， `Source Map` 的作用就来了， `Webpack` 构建代码中，开启 `Source Map` ：

![](https://files.mdnice.com/user/20608/da809f08-d4f1-4990-baea-004c8ab50166.png)

然后重新执行构建，再次打开浏览器：

![](https://files.mdnice.com/user/20608/1ca87741-054e-48e4-b0ab-f47f2209e5e8.png)

可以发现，可以成功定位到具体的报错位置了，这就是 `Source Map` 的作用。需要注意一点的是， `Source Map` 并不是 `Webpack` 特有的，其他打包工具同样支持 `Source Map` ，打包工具只是将 `Source Map` 这项技术通过配置化的方式引入进来。关于打包工具，下文会有介绍。

#### 二、Source Map 的作用

上面的案例只是 `Source Map` 的初体验，现在来说一下它的作用，我们为什么需要 `Source Map` ?

阮一峰老师的[JavaScript Source Map 详解](http://www.ruanyifeng.com/blog/2013/01/javascript_source_map.html)指出，JavaScript 脚本正变得越来越复杂。大部分源码（尤其是各种函数库和框架）都要经过转换，才能投入生产环境。

常见的源码转换，主要是以下三种情况：

* 压缩，减小体积
* 多个文件合并，减少 HTTP 请求数
* 其他语言编译成 JavaScript

这三种情况，都使得实际运行的代码不同于开发代码，除错（ `debug` ）变得困难重重，所以才需要 `Source Map` 。结合上面的例子，即使打包过后的代码，也可以找到具体的报错位置，这使得我们 `debug` 代码变得轻松简单，这就是 `Source Map` 想要解决的问题。

#### 三、如何生成 Source Map

各种主流前端任务管理工具，打包工具都支持生成 `Source Map` 。

##### 3.1 UglifyJS

`UglifyJS` 是命令行工具，用于压缩 `JavaScript` 代码

安装 `UglifyJS` ：

```js
npm install uglify - js - g
```

压缩代码的同时生成 `Source Map` ：

```js
uglifyjs app.js - o app.min.js--source - map app.min.js.map
```

`Source Map` 相关选项：

```js
--source - map Source Map的文件的路径和名称
    --source - map - root 源文件的路径
    --source - map - url //#sourceMappingURL的路径。 默认为--source-map指定的值。
    --source - map - include - sources 是否将源代码的内容添加到sourcesContent数组
    --source - map - inline 是否将Source Map写到压缩代码的最后一行
    -- in -source - map 输入Source Map， 当源文件已经经过变换时使用
```

##### 3.2 Grunt

`Grunt` 是 `JavaScript` 项目构建工具

配置 `grunt-contrib-uglify` 插件以生成 `Source Map` ：

```js
grunt.initConfig({
    uglify: {
        options: {
            sourceMap: true
        }
    }
});
```

使用 `grunt-usemin` 打包源码时， `grunt-usemin` 会依次调用[grunt-contrib-concat](https://github.com/gruntjs/grunt-contrib-concat)与[grunt-contrib-uglify](https://github.com/gruntjs/grunt-contrib-uglify)对源码进行打包和压缩。因此都需要进行配置：

```js
grunt.initConfig({
    concat: {
        options: {
            sourceMap: true
        }
    },
    uglify: {
        options: {
            sourceMap: true,
            sourceMapIn: function(uglifySource) {
                return uglifySource + '.map';
            },
        }
    }
});
```

##### 3.3 Gulp

`Gulp` 是 `JavaScript` 项目构建工具

使用[gulp-sourcemaps](https://github.com/floridoo/gulp-sourcemaps)生成 `Source Map` :

```js
var gulp = require('gulp');
var plugin1 = require('gulp-plugin1');
var plugin2 = require('gulp-plugin2');
var sourcemaps = require('gulp-sourcemaps');

gulp.task('javascript', function() {
    gulp.src('src/**/*.js')
        .pipe(sourcemaps.init())
        .pipe(plugin1())
        .pipe(plugin2())
        .pipe(sourcemaps.write('../maps'))
        .pipe(gulp.dest('dist'));
});
```

##### 3.4 SystemJS

`SystemJS` 是模块加载器

使用[SystemJS Build Tool](https://github.com/systemjs/builder)生成 `Source Map` :

```js
builder.bundle('myModule.js', 'outfile.js', {
    minify: true,
    sourceMaps: true
});
```

* `sourceMapContents`选项可以指定是否将源码写入`Source Map`文件

##### 3.5 Webpack

`Webpack` 是前端打包工具（本文案例都会使用该打包工具）。在其配置文件 `webpack.config.js` 中设置**[devtool](https://webpack.js.org/configuration/devtool/)**即可生成 `Source Map` 文件：

```js
const path = require('path');

module.exports = {
    entry: './src/index.js',
    output: {
        filename: 'bundle.js',
        path: path.resolve(__dirname, 'dist')
    },
    devtool: "source-map"
};
```

* **devtool**有 20 多种不同取值，分别生成不同类型的`Source Map`，可以根据需要进行配置。下文会详细介绍，这里不再赘述。

##### 3.6 Closure Compiler

利用 [Closure Compiler](https://github.com/google/closure-compiler) 生成

#### 四、如何使用 Source Map

生成 `Source Map` 之后，一般在浏览器中调试使用，前提是需要开启该功能，以 `Chrome` 为例：

打开开发者工具，找到 `Settins` ：

![](https://files.mdnice.com/user/20608/d42c0b9c-013d-4ad4-96cf-5511b9782f61.png)

勾选以下两个选项：

![](https://files.mdnice.com/user/20608/70cdc8e3-0e78-4281-be8d-5dc8afd1e596.png)

再回到上面的案例中，源代码文件变成了 `index.js` ，点击进入后显示真实的源代码，即说明成功开启并使用了 `Source Map`

![](https://files.mdnice.com/user/20608/667e7f1c-d279-4de8-83ea-4d21ae1a9db1.png)

#### 五、Source Map 的工作原理

还是上面这个案例，执行打包后，生成 `dist` 文件夹，打开 `dist/bundld.js` ：

![](https://files.mdnice.com/user/20608/acd2323b-a604-4f24-862c-431df22b0e53.png)

可以看到尾部有这句注释：

```js
//# sourceMappingURL=bundle.js.map
```

正是因为这句注释，标记了该文件的 `Source Map` 地址，浏览器才可以正确的找到源代码的位置。 `sourceMappingURL` 指向 `Source Map` 文件的 `URL` 。

除了这种方式之外，[MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/SourceMap)中指出，可以通过 `response header` 的 `SourceMap: <url>` 字段来表明。

>

`

```
> SourceMap: /path/to/file.js.map
> ```

`dist` 文件夹中，除了 `bundle.js` 还有 `bundle.js.map` ，这个文件才是 `Source Map` 文件，也是 `sourceMappingURL` 指向的 `URL`

![](https://files.mdnice.com/user/20608/9124506c-2e1d-410e-b141-97062b36fc3f.png)

* `version`：`Source map`的版本，目前为`v3`。
* `sources`：转换前的文件。该项是一个数组，表示可能存在多个文件合并。
* `names`：转换前的所有变量名和属性名。
* `mappings`：记录位置信息的字符串，下文会介绍。
* `file`：转换后的文件名。
* `sourceRoot`：转换前的文件所在的目录。如果与转换前的文件在同一目录，该项为空。
* `sourcesContent`：转换前文件的原始内容。

##### 5.1 关于Source map的版本

在2009年 `Google` 的一篇文章中，在介绍 `Cloure Compiler` 时， `Google` 也趁便推出了一款调试东西： `Firefox` 插件 `Closure Inspector` ，以便利调试编译后代码。这便是 `Source Map` 的初步代啦！

> You can use the compiler with Closure Inspector , a Firebug extension that makes debugging the obfuscated code almost as easy as debugging the human-readable source.

2010年，在第二代即 `Closure Compiler Source Map 2.0` 中， `Source Map` 招认了共同的 `JSON` 格式及其他标准，已几乎具有现在的雏形。最大的差异在于 `mapping` 算法，也是 `Source Map` 的要害地址。第二代中的 `mapping` 已决定运用 `base 64` 编码，可是算法同现在有收支，所以生成的 `.map` 比较现在要大许多。
2011年，第三代即[**Source Map Revision 3 Proposal**](https://docs.google.com/document/d/1U1RGAehQwRypUTovF1KRlpiOFze0b-_2gc6fAH0KY0k/edit#)出炉了，这也是咱们现在运用的 `Source Map` 版别。从文档的命名看来，此刻的 `Source Map` 已脱离 `Clousre Compiler` ，演化成了一款独立东西，也得到了浏览器的支撑。这一版相较于二代最大的改动是 `mapping` 算法的紧缩换代，运用[VLQ](https://en.wikipedia.org/wiki/Variable-length_quantity)编码生成[base64](https://zh.wikipedia.org/zh-cn/Base64)前的 `mapping` ，大大缩小了 `.map` 文件的体积。

`Source Map` 发展史的诙谐之处在于，它作为一款辅佐东西被开发出来。毕竟它辅佐的方针日渐式微，而它却成为了技能主体，被写进了浏览器中。

> Source Map V1最初步生成的Source Map文件大概有转化后文件的10倍大。Source Map V2将之减少了50%，V3又在V2的基础上减少了50%。所以现在133k的文件对应的Source Map文件巨细大概在300k左右。

##### 5.2 关于mappings属性

为了避免干扰，将案例改成如下不报错的情况：

```js
var a = 1;
console.log(a);
`
```

打包编译的后 `bundle.js` 文件：

```js
/******/
(() => { // webpackBootstrap
    var __webpack_exports__ = {};
    /*!**********************!*\
      !*** ./src/index.js ***!
      \**********************/
    var a = 1;
    console.log(a);
    /******/
})();
//# sourceMappingURL=bundle.js.map
```

打包编译后的 `bundle.js.map` 文件：

```json
{
    "version": 3,
    "sources": [
        "webpack://learn-source-map/./src/index.js"
    ],
    "names": [],
    "mappings": "AAAA;AACA,c",
    "file": "bundle.js",
    "sourcesContent": [
        "var a = 1;\r\nconsole.log(a);"
    ],
    "sourceRoot": ""
}
```

可以看到 `mappings` 属性的值是： `AAAA; AACA, c` ，要想说清楚这个东西，需要先解释一下它的组成结构。这是一个字符串，它分成三层：

* 第一层是**行对应**，以分号（; ）表示，每个分号对应转换后源码的一行。所以，第一个分号前的内容，就对应源码的第一行，以此类推。
* 第二层是**位置对应**，以逗号（, ）表示，每个逗号对应转换后源码的一个位置。所以，第一个逗号前的内容，就对应该行源码的第一个位置，以此类推。
* 第三层是**位置转换**，以[VLQ 编码](https://en.wikipedia.org/wiki/Variable-length_quantity)表示，代表该位置对应的转换前的源码位置。

在回到源代码，就可以分析出：

1. 因为源代码中有两行，所以有一个分号，分号前后表示了第一行和第二行。即`mappings`中的`AAAA`和`AACA,c`。
2. 分号后面表示第二行，也就是代码`console.log(a);`可以拆分出两个位置，分别是`console`和`log(a)`，所以存在一个逗号。即`AACA,c`中的`AACA`和`c`。

总结，就是**转换后的源码分成两行，第一行有一个位置，第二行有两个位置**。

至于这个 `AAAA` ， `AAcA` 等字母是怎么来的，可以参考阮一峰老师的[JavaScript Source Map 详解](http://www.ruanyifeng.com/blog/2013/01/javascript_source_map.html)有作详细的介绍。笔者自己的理解是：

`AAAA` 和 `AAcA` 以及 `c` 都是代表了位置，正常来说，每个位置最多由 5 个字母组成，5 个字母的含义分别是：

* 第一位，表示这个位置在（转换后的代码的）的第几列。
* 第二位，表示这个位置属于 sources 属性中的哪一个文件。
* 第三位，表示这个位置属于转换前代码的第几行。
* 第四位，表示这个位置属于转换前代码的第几列。
* 第五位，表示这个位置属于 names 属性中的哪一个变量。

**这里转换后最多只有 4 个字母，是因为没有 `names` 属性。**

每一个位置都可以用[VLQ 编码](https://en.wikipedia.org/wiki/Variable-length_quantity)转换，形成一种映射关系。可以在[这个网站](https://www.murzwin.com/base64vlq.html)自己转换测试，将 `AAAA; AACA, c` 转换后的结果：

![](https://files.mdnice.com/user/20608/77083640-8f4f-467c-92ba-9367e20b633c.png)

可以得到两组数据：

```js
[0, 0, 0, 0]
[0, 0, 1, 0], [14]
```

数字都是从 `0` 开始的，拿位置 `AAAA` 举例，转换后得到 `[0, 0, 0, 0]` ，所以代表的含义分别是；

1. 压缩代码的第一列。
2. 第一个源代码文件，即`index.js`。
3. 源代码的第一行。
4. 源代码第一列

通过以上解析，我们就能知道源代码中 `var a = 1;` 在打包后文件中，即 `bundle.js` 的具体位置了。

#### 六、Webpack 中的 Source Map

上文介绍了 `Source Map` 的作用，原理等。现在说一下打包工具 `WebPack` 中对 `Source Map` 的应用，毕竟我们在开发中，都离不开它。

上文有说道，只需要在 `webpack.config.js` 文件中配置 `devtool` 就可以使用 `Source Map` ，这个 `devtool` 具体的值有哪些，可以参考**[webpack devtool](https://webpack.docschina.org/configuration/devtool/#root)**

的介绍，官方罗列了 20 几种类型，我们当然不能全部都记住，可以记住几个关键的：

![](https://files.mdnice.com/user/20608/405ff757-af11-4f76-947f-de80c5cd3453.png)

建议以下 7 种可选方案：

* **source-map**：外部。可以查看错误代码准确信息和源代码的错误位置。
* **inline-source-map**：内联。只生成一个内联 `Source Map`，可以查看错误代码准确信息和源代码的错误位置
* **hidden-source-map**：外部。可以查看错误代码准确信息，但不能追踪源代码错误，只能提示到构建后代码的错误位置。
* **eval-source-map**：内联。每一个文件都生成对应的 `Source Map`，都在 `eval` 中，可以查看错误代码准确信息 和 源代码的错误位置。
* **nosources-source-map**：外部。可以查看错误代码错误原因，但不能查看错误代码准确信息，并且没有任何源代码信息。
* **cheap-source-map**：外部。可以查看错误代码准确信息和源代码的错误位置，只能把错误精确到整行，忽略列。
* **cheap-module-source-map**：外部。可以错误代码准确信息和源代码的错误位置，`module` 会加入 `loader` 的 `Source Map`。

内联和外部的区别：

1. 外部生成了文件（`.map`），内联没有。
2. 内联构建速度更快。

以下通过具体的案例演示上面的 7 种类型：

首先，将案例改成报错状态，为了体现列的情况，将源代码修改成如下：

```js
console.log('source map!!!')
var a = 1;
console.log(a, b); //这一行肯定会报错
```

##### 6.1 source-map

```js
devtool: 'source-map'
```

编译后，可以查看**错误代码准确信息和源代码的错误位置**：

![](https://files.mdnice.com/user/20608/00ae616b-41a6-4541-9579-d7a3197e418c.png)

生成了 `.map` 文件：

![](https://files.mdnice.com/user/20608/ca91f358-2f5e-4dd5-8ee4-0c5d6fab0e38.png)

##### 6.2 inline-source-map

```js
devtool: 'inline-source-map'
```

编译后，可以查看**错误代码准确信息和源代码的错误位置**：

![](https://files.mdnice.com/user/20608/00ae616b-41a6-4541-9579-d7a3197e418c.png)

但是没有生成 `.map文件` ，而是以 `base64` 的形式插入到 `sourceMappingURL` 中：

![](https://files.mdnice.com/user/20608/d3ca007d-a842-4336-8191-e129d7099d32.png)

##### 6.3 hidden-source-map

```js
devtool: 'hidden-source-map'
```

编译后，可以查看**错误代码准确信息，但是无法查看源代码的位置**：

![](https://files.mdnice.com/user/20608/5cb53ae1-2bce-4ac9-a497-419e23b33afc.png)

生成了 `.map` 文件：

![](https://files.mdnice.com/user/20608/ca91f358-2f5e-4dd5-8ee4-0c5d6fab0e38.png)

##### 6.4 eval-source-map

```js
devtool: 'eval-source-map'
```

编译后，可以查看**错误代码准确信息和源代码的错误位置**：

![](https://files.mdnice.com/user/20608/d9e29c67-9ce6-4ccd-8ba3-a0865ae4ee77.png)

但是没有生成 `.map文件` ，而是在 `eval函数` 中，包括 `sourceMappingURL` :

![](https://files.mdnice.com/user/20608/dd281d5a-5495-4039-a74c-0c8543ed93b5.png)

![](https://files.mdnice.com/user/20608/5159f157-2b40-4536-82ff-23d81d101ca3.png)

##### 6.5 nosources-source-map

```js
devtool: 'nosources-source-map'
```

编译后，可以查看**无法查看错误代码的准确位置和源代码的错误位置，只能提示错误原因**：

![](https://files.mdnice.com/user/20608/3fced6da-e5a9-4747-90a7-1790827a1759.png)

生成了 `.map` 文件：

![](https://files.mdnice.com/user/20608/ca91f358-2f5e-4dd5-8ee4-0c5d6fab0e38.png)

##### 6.6 cheap-source-map

```js
devtool: 'cheap-source-map'
```

编译后，可以查看**错误代码准确信息和源代码的错误位置，但是忽略了具体的列（ `因为是b导致报错` ）**：

![](https://files.mdnice.com/user/20608/00ae616b-41a6-4541-9579-d7a3197e418c.png)

生成了 `.map` 文件：

![](https://files.mdnice.com/user/20608/ca91f358-2f5e-4dd5-8ee4-0c5d6fab0e38.png)

##### 6.7 cheap-module-source-map

因为需要 `module` ，所以案例中增加 `loader` ：

```js
module: {
    rules: [{
        test: /\.css$/,
        use: [
            // style-loader：创建style标签，将js中的样式资源插入进去，添加到head中生效
            'style-loader',
            // css-loader：将css文件变成commonjs模块加载到js中，里面内容是样式字符串
            'css-loader'
        ]
    }]
}
```

在 `src` 目录下新建 `index.css` 文件，添加样式代码：

```css
body {
    margin: 0;
    padding: 0;
    height: 100%;
    background-color: pink;
}
```

然后在 `src/index.js` 中引入 `index.css` ：

```js
//引入index.css
import './index.css';

console.log('source map!!!')
var a = 1;
console.log(a, b); //这一行肯定会报错
```

修改 `devtool` ：

```js
devtool: 'cheap-module-source-map'
```

打包后，打开浏览器，样式生效，说明 `loader` 引入成功。可以查看**错误代码准确信息和源代码的错误位置，但是忽略了具体的列（ `因为是b导致报错` ）**：

![](https://files.mdnice.com/user/20608/f4c707a0-ffd8-45d4-9c23-d7921d75b741.png)

生成了 `.map` 文件，同时，将 `loader` 的信息也一起打包进来:

![](https://files.mdnice.com/user/20608/27e26292-426c-4ba3-b0c9-4840e4d6852f.png)

![](https://files.mdnice.com/user/20608/a4825a9d-bbee-4cb0-a322-1b603d097e97.png)

##### 6.8 总结

（1）**开发环境**：需要考虑速度快，调试更友好

* 速度快( `eval` > `inline` > `cheap` >... )
  1. `eval-cheap-souce-map`
  2. `eval-source-map`
* 调试更友好
  1. `souce-map`
  2. `cheap-module-souce-map`
  3. `cheap-souce-map`

**最终得出最好的两种方案 --> eval-source-map（完整度高，内联速度快） / eval-cheap-module-souce-map（错误提示忽略列但是包含其他信息，内联速度快）**

（2）**生产环境**：需要考虑源代码要不要隐藏，调试要不要更友好

* 内联会让代码体积变大，所以在生产环境不用内联
* 隐藏源代码
  1. `nosources-source-map` 全部隐藏（打包后的代码与源代码）
  2. `hidden-source-map` 只隐藏源代码，会提示构建后代码错误信息

**最终得出最好的两种方案 --> source-map（最完整） / cheap-module-souce-map（错误提示一整行忽略列）**

#### 七、总结

`Source Map` 是我们日常开发过程中必不可少的，它可以帮助我们调试，定位错误。尽管它涉及非常多的知识点，例如：[VLQ](https://en.wikipedia.org/wiki/Variable-length_quantity)、[base64](https://zh.wikipedia.org/zh-cn/Base64)等，但是我们核心关注的是它的工作原理，以及在打包工具中，如 `webpack` 等对 `Source Map` 的应用。

`Source Map` 非常强大，不仅在应用于日常开发，还可以做更多的事情，如 `性能异常监控平台` 。比如[FunDebug](https://www.fundebug.com/)这个网站就是通过 `Source Map` 还原生产环境中的压缩代码，提供完整的堆栈信息，准确定位出错误源码，帮助用户快速修复 `Bug` ，像这样的案例还有许多。

总之，学习 `Source Map` 是非常有必要的。

#### 八、参考q

* [Introduction to JavaScript Source Maps](https://www.html5rocks.com/en/tutorials/developertools/sourcemaps/)
* [MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/SourceMap)
* [JavaScript Source Map 详解](http://www.ruanyifeng.com/blog/2013/01/javascript_source_map.html)
* [VLQ](https://en.wikipedia.org/wiki/Variable-length_quantity)
* [base64](https://zh.wikipedia.org/zh-cn/Base64)
* [base64vlq](https://www.murzwin.com/base64vlq.html)
* [FunDebug](https://www.fundebug.com/)
* [绝了，没想到一个 source map 居然涉及到那么多知识盲区](https://juejin.cn/post/6963076475020902436)
* [谈谈我是如何获得知乎的前端源码的](https://zhuanlan.zhihu.com/p/26033573)
