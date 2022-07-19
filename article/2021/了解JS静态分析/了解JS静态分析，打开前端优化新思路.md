## 前言
我们经常能在webpack和ES6相关知识中看到静态分析（静态优化）这个词，今天我们就来给静态分析做个分析，希望看完这篇文章，能让你知道以下三个内容：
1. 什么是静态分析？
1. 为什么要做静态分析？
1. JS如何做静态分析？

## 1. 什么是静态分析

静态分析通俗来讲就是在程序没有运行的时候，对它的语法、词法等进行代码扫描分析，发现可能存在的问题，并针对问题做优化改进。
对于JS来讲，`eslint` `tree-shaking` 可以说是我们经常听到的静态代码分析工具了，本篇也就是重点介绍这两个工具是如何对代码做静态分析和优化的。


## 2.为什么要做静态分析？
设想一下，如果我们写的代码没有经过工具的检测就放到服务器使用，出现错误的概率几乎是100%！再设想一下，一个项目经过年复一年的开发迭代，经过多少的人，多少的变更，里面有多少不被使用却依然被保留，或者引入了却没被调用的代码，要解决这些问题就需要做静态分析。


## 3.JS如何做静态分析？
接下来是本篇文章的重点，我们将介绍`eslint`和`tree-shaking`是如何做静态分析的，在开始前，我们就不得不先了解下JS是怎么运行的。
### 3.1 JS运行过程
我们都知道JS可以在浏览器中运行，我们就拿Chrome的V8引擎来讲JS的执行过程，在这之前先了解几个概念：
- 编译器（Compiler）：将源代码在运行之前编译成计算机能执行的机器码，由于要编译完所有源代码后在执行，所以编译器需要更多的内存存储机器码，但执行快；
- 解释器（Interpreter）：将源代码在运行时逐行解释执行，由于是一边解释一边执行，故启动快，执行慢；
- 抽象语法树（AST）：解析器（Parser)将源代码进行词法分析、语法分析后生成的抽象语法树，想要看生成的结果请戳：https://astexplorer.net/
- 字节码（Bytecode）：又称作中间代码，在JS解析中就是从`AST -> 字节码 -> 机器码`，字节码是后面才被V8引擎引入的，主要目的是为了解决机器码带来的内存占用问题；
- 即时编译器（JIT）：简单的理解就是一段代码被解释器执行多次之后就会变成热点代码（HotSpot），热点代码会被编译器直接编译成机器码，当代码再次执行时直接运行机器码，从而达到提高性能的目的，这种编译器和解释器混合使用的技术被叫做即时编译。


**了解了几个基本概念后我们再来看看V8执行一段JS代码的过程图：**

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d5c63cf4c71e49ad8b31e1bd1c037fe7~tplv-k3u1fbpfcp-watermark.image?)

**关于即时编译器的运行过程可以看下图：**

![662413313149f66fe0880113cb6ab98a.webp](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9162d69b4d8241199e8d2f68ee43ab6a~tplv-k3u1fbpfcp-watermark.image?)

*（图片来自极客时间 --《浏览器工作原理与实践》）*

有同学想要再深入了解运行原理可以看看[浏览器工作原理与实践](https://time.geekbang.org/column/intro/100033601?tab=catalog "liull") 课程（要钱）或者看看其他文章或[官网](https://v8.dev/docs)。

### 3.2 JS的静态分析阶段

从上面的图中可以看出执行一段JS代码需要经历3次重要的代码转换即：`源码->AST->字节码->机器码`，最常见的静态分析阶段就是在 `源码->AST` 这个过程，比如`eslint`,`tree-shaking`,`babel`,`uglify`等都是把源码转换成`AST`后去做分析处理的。

### 3.3 eslint静态分析
>这里我们来实际跑一跑`eslint`的检验过程，看一看它是怎么把源码转换成AST后去做分析处理的。

##### Step1：拉仓库
拉取[源码](https://github.com/eslint/eslint)，安装完毕后，我们选一条比较简单的规则 `lib/rules/default-case.js` 来看效果，本条规则是对`switch`语句是否有`default`做检测。

##### Step2：准备调试环境
我们使用vscode去跑`eslint`的规则测试，`eslint`单测用的是`mocha`，配置vscode调试如下：
```json
{
    // 在.vscode目录下的launch.json文件
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Run mocha",
            "type": "node",
            "request": "launch",
            "program": "${workspaceRoot}/node_modules/mocha/bin/_mocha", //启用mocha脚本
            "stopOnEntry": false,
            "args": [
                "tests/lib/rules/default-case.js", //这个就是我们等下要调试的规则
                "--no-timeouts"
            ],
            "cwd": "${workspaceRoot}",
            "runtimeExecutable": null,
            "env": {
                "NODE_ENV": "testing"
            }
        }
    ]
}
```
##### Step3：跑起来
> 前置知识：关于eslint的规则怎么写，AST节点类型介绍都可以看我们的另一篇文章[深入浅出之ESLint](https://juejin.cn/post/7028754877312401444)

看下图的4个重要文件：
![eslint.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/739fc5aeba0e4f0e9a490f9405af5bc0~tplv-k3u1fbpfcp-watermark.image?)

调试进入`linter.js`文件可以看到`AST`的转化结果：
![截屏2021-11-20 下午3.48.12.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ca78c9c0a0314b3ab734996c7cd70480~tplv-k3u1fbpfcp-watermark.image?)

如原始语句：`switch (a) { case 1: break; default: break; }` 先转化成AST如下：
```json
{
    "type": "Program",
    "loc": {
        "start": {
            "line": 1,
            "column": 0
        },
        "end": {
            "line": 1,
            "column": 45
        }
    },
    "range": [0, 45],
    "body": [{
        "type": "SwitchStatement", //这个就是要判断的switch节点
        "loc": {
            "start": {
                "line": 1,
                "column": 0
            },
            "end": {
                "line": 1,
                "column": 45
            }
        },
        "range": [0, 45],
        "discriminant": {
            "type": "Identifier",
            "loc": {
                "start": {
                    "line": 1,
                    "column": 8
                },
                "end": {
                    "line": 1,
                    "column": 9
                }
            },
            "range": [8, 9],
            "name": "a"
        },
        "cases": [{
            "type": "SwitchCase",
            "loc": {
                "start": {
                    "line": 1,
                    "column": 13
                },
                "end": {
                    "line": 1,
                    "column": 27
                }
            },
            "range": [13, 27],
            "consequent": [{
                "type": "BreakStatement",
                "loc": {
                    "start": {
                        "line": 1,
                        "column": 21
                    },
                    "end": {
                        "line": 1,
                        "column": 27
                    }
                },
                "range": [21, 27],
                "label": null
            }],
            "test": {
                "type": "Literal",
                "loc": {
                    "start": {
                        "line": 1,
                        "column": 18
                    },
                    "end": {
                        "line": 1,
                        "column": 19
                    }
                },
                "range": [18, 19],
                "value": 1,
                "raw": "1"
            }
        }, {
            "type": "SwitchCase",
            "loc": {
                "start": {
                    "line": 1,
                    "column": 28
                },
                "end": {
                    "line": 1,
                    "column": 43
                }
            },
            "range": [28, 43],
            "consequent": [{
                "type": "BreakStatement",
                "loc": {
                    "start": {
                        "line": 1,
                        "column": 37
                    },
                    "end": {
                        "line": 1,
                        "column": 43
                    }
                },
                "range": [37, 43],
                "label": null
            }],
            "test": null
        }]
    }],
    "sourceType": "script",
    "comments": [],
    "tokens": [{
        "type": "Keyword",
        "value": "switch",
        "loc": {
            "start": {
                "line": 1,
                "column": 0
            },
            "end": {
                "line": 1,
                "column": 6
            }
        },
        "range": [0, 6]
    }, {
        "type": "Punctuator",
        "value": "(",
        "loc": {
            "start": {
                "line": 1,
                "column": 7
            },
            "end": {
                "line": 1,
                "column": 8
            }
        },
        "range": [7, 8]
    }, {
        "type": "Identifier",
        "value": "a",
        "loc": {
            "start": {
                "line": 1,
                "column": 8
            },
            "end": {
                "line": 1,
                "column": 9
            }
        },
        "range": [8, 9]
    }, {
        "type": "Punctuator",
        "value": ")",
        "loc": {
            "start": {
                "line": 1,
                "column": 9
            },
            "end": {
                "line": 1,
                "column": 10
            }
        },
        "range": [9, 10]
    }, {
        "type": "Punctuator",
        "value": "{",
        "loc": {
            "start": {
                "line": 1,
                "column": 11
            },
            "end": {
                "line": 1,
                "column": 12
            }
        },
        "range": [11, 12]
    }, {
        "type": "Keyword",
        "value": "case",
        "loc": {
            "start": {
                "line": 1,
                "column": 13
            },
            "end": {
                "line": 1,
                "column": 17
            }
        },
        "range": [13, 17]
    }, {
        "type": "Numeric",
        "value": "1",
        "loc": {
            "start": {
                "line": 1,
                "column": 18
            },
            "end": {
                "line": 1,
                "column": 19
            }
        },
        "range": [18, 19]
    }, {
        "type": "Punctuator",
        "value": ":",
        "loc": {
            "start": {
                "line": 1,
                "column": 19
            },
            "end": {
                "line": 1,
                "column": 20
            }
        },
        "range": [19, 20]
    }, {
        "type": "Keyword",
        "value": "break",
        "loc": {
            "start": {
                "line": 1,
                "column": 21
            },
            "end": {
                "line": 1,
                "column": 26
            }
        },
        "range": [21, 26]
    }, {
        "type": "Punctuator",
        "value": ";",
        "loc": {
            "start": {
                "line": 1,
                "column": 26
            },
            "end": {
                "line": 1,
                "column": 27
            }
        },
        "range": [26, 27]
    }, {
        "type": "Keyword",
        "value": "default",
        "loc": {
            "start": {
                "line": 1,
                "column": 28
            },
            "end": {
                "line": 1,
                "column": 35
            }
        },
        "range": [28, 35]
    }, {
        "type": "Punctuator",
        "value": ":",
        "loc": {
            "start": {
                "line": 1,
                "column": 35
            },
            "end": {
                "line": 1,
                "column": 36
            }
        },
        "range": [35, 36]
    }, {
        "type": "Keyword",
        "value": "break",
        "loc": {
            "start": {
                "line": 1,
                "column": 37
            },
            "end": {
                "line": 1,
                "column": 42
            }
        },
        "range": [37, 42]
    }, {
        "type": "Punctuator",
        "value": ";",
        "loc": {
            "start": {
                "line": 1,
                "column": 42
            },
            "end": {
                "line": 1,
                "column": 43
            }
        },
        "range": [42, 43]
    }, {
        "type": "Punctuator",
        "value": "}",
        "loc": {
            "start": {
                "line": 1,
                "column": 44
            },
            "end": {
                "line": 1,
                "column": 45
            }
        },
        "range": [44, 45]
    }]
}
```
内容很长，但是需要处理匹配的节点只有`SwitchStatement`，到`lib/rules/default-case.js`中去查看运行结果：
![截屏2021-11-23 下午11.14.10.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/12aaf2e96ec04a17b867de1a8a306271~tplv-k3u1fbpfcp-watermark.image?)
符合规则则通过，否则不通过。


小结：到这里就已经跑完`eslint`校验的整个过程，看起来很简单，里面的工程却是很大，在此不详细讲里面的细节，这里主要就是让大家了解它是怎么把 `源码 转 AST` 后去做分析处理的，感兴趣的同学可以自己去拉源码跑一跑。

### 3.4 tree-shaking静态优化
`tree-shaking`中文摇树，意思就是把挂树上没用的东西（模块）摇掉。在此之前，我们需要先知道：`tree-shaking`的模块消除原理是基于`ES6`的模块特性。

阮一峰老师说的：`ES6` 模块的设计思想是尽量的**静态化**，使得编译时就能确定模块的依赖关系，以及输入和输出的变量。`CommonJS` 和 `AMD` 模块，都只能在运行时确定这些东西。比如，`CommonJS` 模块就是对象，输入时必须查找对象属性。[【Module 的语法】](https://es6.ruanyifeng.com/#docs/module)

所以只有在`ESM`的模块设计之下`tree-shaking`才有它的用武之地。和传统的代码优化工具如`uglify`不同，`tree-shaking`关注于消除没有用到的代码，而`uglify`关注于消除不会执行的代码。

**来看看`tree-shaking`的效果**
>如下演示效果是用webpack举例，webpack2.0之后已引入tree-shaking，开启tree-shaking方式需要满足：
>1. 使用 ESM 规范编写模块代码
>2. 启用optimization代码优化功能

简单写个变量导入：
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b7d4e3d2ea2e43bd98f0edca7cdc624a~tplv-k3u1fbpfcp-watermark.image?)

打包后的结果，**成功把没用到的`test`变量干掉**：
![webpack.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/242bd648ee564a3fa0d44be2f770663d~tplv-k3u1fbpfcp-watermark.image?)

如上可以看出模块的引入确实可以得到优化，但是许多代码副作用带来的问题，使`Tree Shaking`的优化效果并不能达到预期，比如下面这番无意义的赋值操作：

![截屏2021-11-20 上午10.37.59.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/69f8d1b32307429db8b597e067e92c1e~tplv-k3u1fbpfcp-watermark.image?)
**并没有把`test`变量干掉：**
![截屏2021-11-20 上午10.45.24.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b99ed020fbff4f5cabcacf3a8ebd1b1b~tplv-k3u1fbpfcp-watermark.image?)


>所以前端的优化靠工具是完全不够的，还需要靠智人们（聪明的你们）良好的编码习惯和规范，方能打造出人见人爱，花见花开的优秀代码。


顺带验证一下上面说的如果用`CommonJS`的方式引入则没法做到`tree-shaking`的效果：
![截屏2021-11-19 下午11.25.26.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/824f9d68b55346b19c71cd656890f743~tplv-k3u1fbpfcp-watermark.image?)

恩！果不其然：
![截屏2021-11-19 下午11.25.50.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1704d9db51b44c9285aafc45f627fc50~tplv-k3u1fbpfcp-watermark.image?)

有想要自己玩玩的可以拉我的demo项目快速跑一跑：[webpack-demo](https://github.com/kenicya/webpack-demo)

>`tree-shaking`的静态优化就解释到这里，看官想要继续深入理解里面的原理可以看看[【Tree-Shaking性能优化实践 - 原理篇】](https://zhuanlan.zhihu.com/p/32554436)和[ 【Webpack 原理系列九】](https://juejin.cn/post/7002410645316436004)这里不多做赘述。


## 总结
本篇目的就是让大家了解什么是静态分析，前端是如何做到静态分析并且有哪些比较常见的方式，当下次你再看到关于`静态分析`或`静态优化`亦或者`静态化`时，能知道他们想要说的是什么。

前端静态优化的路还很长，`TypeScript`的崛起、`ES6`模块的静态化设计、越来越多的分析工具，未来还会有更多的新科技新技能出现，向为我们创造更好的IT时代的巨人们致敬。

