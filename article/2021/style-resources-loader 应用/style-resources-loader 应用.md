# style-resources-loader 应用

> 此loader将样式资源（例如变量、mixin）注入到sass、scss、less、stylus等模块中

---
通过在工作中的应用场景，来介绍此loader用法和一些发散的点

- 场景需求
- 解决方案
- 发散点
    - loader匹配规则
    - loader顺序
    - 调试loader
    - style-resources-loader的injector属性



## 场景需求    
项目结构如下：
```                          
  ├─views             
  ├─componets              组件目录    
  |     ├─brand1           组件1
  |            ├─cmp.vue
  |            ├─...
  |     └─brand2           组件2
  |            ├─cmp.vue
  |            ├─...
  ├─theme                  主题目录
  (每个主题中颜色变量可能会重复)
  |     ├─brand1.less      组件1主题颜色变量
  |     └─brand2.less      组件2主题颜色变量
  |     └─other.less       其他主题颜色变量
  └─...     
```

需求：

- 组件目录使用对应的主题颜色变量，其他目录使用主题3颜色变量
- 使用主题颜色变量时不需要每次单独@import  '@/theme/brandx.less'


## 解决方案
```javascript
module: {
        rules: [
            {
                test: /\.less$/i,
                use: [
                    MiniCssExtractPlugin.loader, 
                    "css-loader", 
                    'less-loader'
                ],
            },
            {
                test: /components\\brand1.*\.less$/i,
                use: [{
                    loader: 'style-resources-loader',
                    options: {
                        patterns: [path.resolve(__dirname, 'src/theme/brand1.less')]
                    },
                }],
            },
            {
                test: /components\\brand2.*\.less$/i,
                use: [{
                    loader: 'style-resources-loader',
                    options: {
                        patterns: [path.resolve(__dirname, 'src/theme/brand2.less')]
                    },
                }],
            },
            {
                test: /[^components\\].*\.less$/i,
                use: [{
                    loader: 'style-resources-loader',
                    options: {
                        patterns: [path.resolve(__dirname, 'src/theme/brand3.less')]
                    },
                }],
            },
        ]
    },

```

## 发散点
### loader匹配规则
对于loader不熟悉的时候会有一个疑问，为什么样式明明写在.vue文件中，但是我们loader的test是.less后缀呢？

因为loader会把非js的文件转换为js文件，不同类型的文件根据定义的rules中不同test去匹配相应的loader进行处理，这里对vue组件中样式文件的处理其实是先经过了vue-loader将.vue文件中的`<style lang="less">`部分转换为less文件后，再由其他匹配此规则的loader进行处理。

### loader顺序
在之前使用loader的过程中，我只知道use中的loader顺序是从右向左(或者说是从下到上，这个是看书写的习惯)，例如下面，执行顺序是less-loader->css-loader->MiniCssExtractPlugin.loader
```javascript
{
    test: /\.less$/i,
    use: [
        MiniCssExtractPlugin.loader, 
        "css-loader", 
        'less-loader'
    ],
},
```
其实在rules中对于相同类型文件的匹配顺序也是一样；例如我们把rules顺序换一下，此时所有的less文件在第一个规则中已经全部转换为css，无法匹配第二个规则执行style-resources-loader，导致其less变量并未注入从而引起报错
```javascript
// error code
module: {
        rules: [
            {
                test: /components\\brand1.*\.less$/i,
                use: [{
                    loader: 'style-resources-loader',
                    options: {
                        patterns: [path.resolve(__dirname, 'src/theme/brand1.less')]
                    },
                }],
            },
            {
                test: /\.less$/i,
                use: [
                    MiniCssExtractPlugin.loader, 
                    "css-loader", 
                    'less-loader'],
            },
        ]
      }
```

同时也可以根据打印信息看到具体的loader执行顺序
![](https://files.mdnice.com/user/17293/f13803e1-a028-4903-8139-8644ff214bb2.png)


> 扩展：在很多资料中有介绍loader的pitch概念
[https://webpack.docschina.org/api/loaders/#pitching-loader](https://webpack.docschina.org/api/loaders/#pitching-loader)

这边可以自己写一个例子去感受一下
```javascript
//src/loaders/a.js
module.exports = function(source) {
      // this.data为在 pitch 阶段和 normal 阶段之间共享的 data 对象。
      console.log('a-loader'， this.data);
      return source;
};

module.exports.pitch = function(remainingRequest, precedingRequest, data) {
      console.log('a-pitch**************************');
      console.log(remainingRequest);
      console.log(precedingRequest);
      console.log(data);
      data.value = 'a-pitch-value';
}

//src/loaders/b.js
module.exports = function(source) {
      console.log('b-loader', this.data);
      return source;
};

module.exports.pitch = function(remainingRequest, precedingRequest, data) {
      console.log('b-pitch**************************');
      console.log(remainingRequest);
      console.log(precedingRequest);
      console.log(data);
      data.value = 'b-pitch-value';
}

//src/loaders/c.js
module.exports = function(source) {
      console.log('c-loader'， this.data);
      return source;
};

module.exports.pitch = function(remainingRequest, precedingRequest, data) {
      console.log('c-pitch**************************');
      console.log(remainingRequest);
      console.log(precedingRequest);
      console.log(data);
      data.value = 'c-pitch-value';
}

//webpack.config.js
module: {
        rules: [
            {
                use: [
                    path.join(__dirname, 'src/loaders/a.js'),
                    path.join(__dirname, 'src/loaders/b.js'),
                    path.join(__dirname, 'src/loaders/c.js'),
                ],
            },
        ]
    },
```

结果如下：
![](https://files.mdnice.com/user/17293/081645fa-380c-4318-b9cb-0e67c7b64371.png)

这个方法好像用的不是很多，目前就在`mini-css-extract-plugin`中看到


### 调试loader

![](https://files.mdnice.com/user/17293/8da541fb-2d11-455c-9c6f-350657c033e9.png)
然后选择`node.js`，项目里面会多一个`.vscode`的文件夹
![](https://files.mdnice.com/user/17293/65d7b599-a698-4fd7-ab15-d17dd4de6da9.png)
配置`launch.json`文件
```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "type": "pwa-node",
            "request": "launch",
            "name": "webpack",
            "program": "D:\\webpack-test\\node_modules\\webpack-cli\\bin\\cli.js",
            //"args": [
            //    "dev"
            //]
            // 或者
            // "runtimeExecutable": "npm",
            // "runtimeArgs": [
            //     "run-script", 
            //     "dev"
            // ]
        }
    ]
}
```
主要是配置`program`，此为你执行文件的位置，`args`为你执行命令时的参数，我这边只是用webpack打包，所以不需要此参数，这里等价于让vscode自带的node调试功能去帮你执行webpack这条命令，或者是直接使用调试npm脚本命令的方式，这样更简单，前提是你需要在package.json中定义好script

配置好了之后，就可以直接在node_modules里面找到相应的loader代码，然后打上断点，按F5开始进行调试

### style-resources-loader的injector属性

第一次看这个属性的时候可能会看不懂，但是如果会调试了loader的话，直接去源码里面去打个断点就知道怎么用的了

|   Name  |  Type   | Default    |    Description |
| --- | --- | --- | --- |
|  injector   | Function  <br>'prepend' <br>'append'    |  'prepend'   |   Controls the resources injection precisely  |


此属性为`精确控制资源注入`

例如:
```css
//a.less
@primary-color: red;
.text-color {
    color: @primary-color;
}
//var.less
@primary-color: black;
```
当`injector`为`prepend`时
```css
//a.css
.text-color {
  color: red;
}
```
当`injector`为`append`时
```css
//a.css
.text-color {
  color: black;
}
```
从例子可以看出来，控制我们的变量是否会覆盖原文件中的变量

当`injector`为回调函数时，官方给的例子是
```
module.exports = {
    // ...
    module: {
        rules: [{
            test: /\.styl$/,
            use: ['style-loader', 'css-loader', 'stylus-loader', {
                loader: 'style-resources-loader',
                options: {
                    patterns: [
                        path.resolve(__dirname, 'path/to/stylus/variables/*.styl'),
                        path.resolve(__dirname, 'path/to/stylus/mixins/*.styl')
                    ],
                    injector: (source, resources) => {
                        const combineAll = type => resources
                            .filter(({ file }) => file.includes(type))
                            .map(({ content }) => content)
                            .join('');
 
                        return combineAll('variables') + combineAll('mixins') + source;
                    }
                }
            }]
        }]
    },
    // ...
}
```
意思就是让我们自己控制注入的变量内容是来自于哪个文件（variables、mixins）

source为源文件内容字符串

resources为注入文件的对象数组，对象包含file和content属性
  - file为该文件的绝对路径
  - content为该文件的文件内容字符串

源码：
```typescript
const normalizeInjector = (injector: StyleResourcesLoaderOptions['injector']): StyleResourcesNormalizedInjector => {
    if (typeof injector === 'undefined' || injector === 'prepend') {
        return (source, resources) => resources.map(getResourceContent).join('') + source;
    }

    if (injector === 'append') {
        return (source, resources) => source + resources.map(getResourceContent).join('');
    }

    return injector;
};
```