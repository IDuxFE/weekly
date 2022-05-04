# Node.js知识体系概览

![](https://files.mdnice.com/user/22004/2441ab5e-2e4e-4534-8209-d5cacf965d4d.jpeg)

## 一、背景

本文从《Node.js实战（第2版）》出发，旨在总结使用Node.js创建Web应用程序的知识点，构建Node.js相关知识体系。

## 二、什么是Node.js？

Node.js(以下简称Node)是一个开源的、跨平台的运行时环境。通过Node.js，开发人员可以使用 JavaScript 创建各种服务器端工具和应用程序。

## 三、Node优势

Node显著特征是它的异步和事件驱动机制，以及小巧精悍的标准库。

**非阻塞I/O**

在进行速度较慢的处理时让Node能做其他事情，是使用带非阻塞I/O的异步API真正的好处。举例来说，程序在做其他事情时发起一个请求来获取网络资源，然后当网络操作完成时，将会运行一个回调函数来处理这个操作的结果。Node使用libuv的库来访问操作系统的非阻塞网络调用。

**事件轮询**

事件轮询是单向运行的先入先出队列，经过以下几个阶段：首先是计时器开始执行，这些计时器都是用JavaScript函数setTimeout和setInterval安排好的。接下来是运行I/O回调，即触发回调函数。轮询阶段会去获取新的I/O事件，最后是用setImmediate安排回调。

![](https://files.mdnice.com/user/22004/2a143c48-c1f7-4855-a6e5-bf0286f4e45b.png)

**Node自带的工具**

Node自带了npm包管理器，以及从文件的fs、path和网络I/O的http到zlib压缩等无所不包的核心JavaScript模块，还有一个调试器debug。

## 四、Hello World

创建一个新的Web程序流程如下：首先创建一个目录，然后运行npm init，添加依赖项，起一个服务器，添加npm脚本。

### 4.1 创建目录

    mkdir hello-world
    cd hello-world
    npm init -y

项目创建完后，使用npm来的模块来降低开发难度。Node自带来一个http模块，它有个服务器。但使用http模块需有很多套路化开发工作，所以一般选择更便捷的Express。

### 4.2 添加依赖项

要添加项目依赖项，可以用npm install。

比如安装Express命令，安装完后可在package.json的dependencies项中看到已经加上。

    npm install --save express

卸载Express命令,此命令会把它从node_modules/中删除，还会更新package.json。

    npm rm express --save

### 4.3 一个简单的服务器

Express以Node自带的http模块为基础，在HTTP请求和响应上来建模Web程序：首先需要用express创建一个程序实例，添加路由处理器，然后将这个程序实例绑定到一个TCP端口上。

    const express = require('express');
    const app = express();

    app.get('/', (req, res) => {
        res.send('Hello World!');
    });

    app.listen(3000, () => {
        console.log('示例应用正在监听 3000 端口!');
    }); 

将以上代码放到index.js文件中，用node index.js运行起来，访问http://localhost:3000 。 可以看到左上角有 “Hello World" 字样的网页。

### 4.4 npm脚本

将启动服务器的命令可以保存为npm脚本，打开package.json文件，在script里添加属性，只要运行添加的命令即可启动程序。如上例：

    "scripts": {
        "start": "node index.js",
        "test": "echo \"Error: no test specified\" && exit 1"
    }

## 五、服务器端框架

### 5.1 框架分类

+ API框架：用于搭建Web API的库，有协助组织程序结构的框架支持。比如：LoopBack。
+ HTTP服务器库：所有基于Express的项目都可以归为这一类。这些库围绕HTTP动词和路由搭建程序。比如：Koa。
+ HTTP服务器框架：用来搭建模块化HTTP服务器的框架。比如：hapi。
+ Web MVC框架：模型-视图-控制器框架。比如：Sail.js。
+ 全栈框架：这种框架在服务器端和浏览器上用的都是JavaScript，并且两端可以共享代码。这被称为同构代码。比如：DerbyJS。

以下介绍为结合Node官网列出的几个值得学习的框架。

### 5.2 框架简介

+ koa：由 Express 背后的同一个团队构建，旨在更简单、更小，建立在多年知识的基础上。新项目的诞生是为了在不破坏现有社区的情况下创建不兼容的更改。此框架轻便，极简，适合依赖外部Web API的单页Web程序。
+ hapi：用于构建应用程序和服务的富框架，使开发者能够专注于编写可重用的应用程序逻辑，而不是花时间搭建基础设施。此框架重点是HTTP服务器和路由。适合由很多小服务器组成的轻便后台。
+ Loopback.io：使构建需要复杂集成的现代应用程序变得容易。此框架省掉了写套路化代码的工作。它可以快速生成带有数据库支持的REST API，并有个API管理界面。

### 5.3 框架对比

以下为框架特性对比：

|  | koa | hapi | LoopBack |
| ----------- | ----------- | ----------- | ----------- |
| 库类型 | HTTP服务器库 | HTTP服务器框架 | API框架 |
| 功能特性 | 基于生成器的中间件，请求/响应模型 | 高层服务器容器抽象，安全的头部信息 | ORM、API用户界面、WebSocket、客户端SDK |
| 建议应用 | 轻型Web程序、不严格的HTTP API、单页Web程序 | 单页Web程序、HTTP API | 支持多客户端的API |
| 插件架构 | 中间件  | hapi插件 | Express中间件 |
| 文档 | https://koajs.com/  | https://hapi.dev/ | https://loopback.io/ |
| 热门程度 | GitHub 32.6K 星 | GitHub 13.8K 星 | GitHub 13.3K 星 |

以下为框架具体使用对比：

|  | koa | hapi | LoopBack |
| ----------- | ----------- | ----------- | ----------- |
| 设置 | 安装模块和定义中间件 | 创建目录，安装hapi，创建实例，服务器连接，服务器启动 | 使用StrongLoop命令行工具，通过slc命令使用 |
| 定义路由 | koa router是一个流行的路由器中间件组成 | 有创建路由的API，要创建路由，必须提供一个包含请求方法、URL和回调函数的对象，其中的回调函数就是路由处理器 | 在Express层面添加 |
| REST API | 没有提供实现RESTful API所必须的工具，只能借助某种路由处理中间件 | 支持HTTP动词和URL参数化，允许用标准的hapi路由API实现REST API | 模型生成器 |
| 优点 | 精简，还有一些非常棒的第三方模块，语法优雅，能根据项目的具体需求量身定制  | hapi插件：扩展服务器，添加各种功能；基于HTTP服务器，适合部署某些场景中 | 免除了繁琐的套路化代码；对前端代码没有太多限制 |
| 缺点 | 可配置水平低，低下的代码复用率  | 极简，对项目结构没有把控。过于依赖插件可能会造成难以维护。 | 使用成本较高：如果映射到已有的数据库模型；受限于Express所支持的功能；没有特定插件API |

## 六、Express

### 6.1 什么是Express？

Express 是最流行的 Node 框架，是许多其它流行 Node 框架 的底层库。它提供了以下机制：

+ 为不同 URL 路径中使用不同 HTTP 动词的请求（路由）编写处理程序。
+ 集成了“视图”渲染引擎，以便通过将数据插入模板来生成响应。
+ 设置常见 web 应用设置，比如用于连接的端口，以及渲染响应模板的位置。
+ 在请求处理管道的任何位置添加额外的请求处理“中间件”。

虽然 Express 本身是极简风格的，但是开发人员通过创建各类兼容的中间件包解决了几乎所有的 web 开发问题。这些库可以实现 cookie、会话、用户登录、URL 参数、POST 数据、安全头等功能。

### 6.2 生成程序框架

**使用应用生成器**

首先使用npm安装express-generator，生成器有许多选项，可以使用 --help（或 -h）命令进行查看：

    $ express --help

    用法：express [选项] [目录]

    选项：

            --version        打印版本号
        -e, --ejs            添加 ejs 引擎支持
            --pug            添加 pug 引擎支持
            --hbs            添加 handlebars 引擎支持
        -H, --hogan          添加 hogan.js 引擎支持
        -v, --view <engine>  添加 <engine> 试图引擎支持 (ejs|hbs|hjs|jade|pug|twig|vash) (默认为 jade)
        -c, --css <engine>   添加 <engine> 样式表引擎支持 (less|stylus|compass|sass) (默认为纯 css)
            --git            添加 .gitignore
        -f, --force          对非空文件夹强制执行
        -h, --help           打印帮助信息

**创建目录**

运行 Express 应用生成器，生成器将创建项目的文件。目录结构（使用 Pug 模板库，不使用 CSS 引擎）如下（不带 “/” 前缀的是文件）：

    /projectName
        app.js
        /bin
            www
        package.json
        /node_modules
        /public
            /images
            /javascripts
            /stylesheets
                style.css
        /routes
            index.js
            users.js
        /views
            error.pug
            index.pug
            layout.pug

package.json 文件定义依赖项和其他信息，以及一个调用应用入口（/bin/www，一个 JavaScript 文件）的启动脚本，脚本中还设置了一些应用的错误处理，加载 app.js 来完成其余工作。/routes 目录中用不同模块保存应用路由。/views 目录保存模板。

**生成的项目**

执行npm install，然后执行npm start启动项目。在浏览器中访问 http://localhost:3000/ ， 默认程序如下：

![](https://files.mdnice.com/user/22004/b904d59f-0a32-4fcd-9554-5ae68044fc58.png)

然后根据存储数据章节，进行数据库连接。

### 6.3 Express MVC

下图展示了 HTTP 请求/响应处理的主数据流和需要实现的行为。图中除视图（View）和路由（Route）外，还展示了控制器（Controller），它们是实际的请求处理函数，与路由请求代码是分开的。

模型已经创建，现在要创建的主要是：

+ 路由：把需要支持的请求（以及请求 URL 中包含的任何信息）转发到适当的控制器函数。
+ 控制器：从模型中获取请求的数据，创建一个 HTML 页面显示出数据，并将页面返回给用户，以便在浏览器中查看。
+ 视图（模板）：供控制器用来渲染数据。

![](https://files.mdnice.com/user/22004/2d512ab6-1f12-4f3b-8535-dcc686df7e04.png)

### 6.4 中间件

Express是在Connect基础上，通过添加高层糖衣扩展和搭建出来的。本节将简单介绍下Connect中间件。

Connect中间件就是JavaScript函数。这个函数一般会有三个参数：请求对象，响应对象，以及一个名为next的回调函数。一个中间件完成自己的工作，要执行后续的中间件时，可以调用这个回调函数。

Connect中的use方法就是用来组合中间件的。举例：

    const connect = require('connect');

    //输出HTTP请求的方法和URL并调用next()：把控制权交还给分派器
    function logger(req, res, next) {
        next();
    }
    //用‘hello’响应HTTP请求
    function hello(req, res) {
        res.end('hello')
    }

    connect()
        .use(logger)
        .use(hello)
        .listen(3000)

中间件的顺序会对程序的行为产生显著影响。漏掉next()能停止执行，也可以通过组合中间件实现用户认证之类的功能。

Express中间件脚本会放在middleware/目录下。

### 6.5 路由

Express路由的主要任务是将特地模式的URL匹配到响应逻辑上。但也可以将URL模式匹配到中间件上，以便用中间件实现某些路由上的可重用功能。

创建路由有几种方法。本文将使 express.Router 中间件，因为使用它可以将站点特定部分的路由处理程序打包，并使用通用路由前缀访问它们。

以下代码举例说明了如何创建路由模块，以及如何在 Express 应用中使用它。

    // wiki.js - 维基路由模块

    const express = require('express'); //导入 Express 应用对象
    const router = express.Router(); //取得一个 Router 对象

    // 主页路由： 用 get() 方法向其添加具体的路由
    router.get('/', (req, res) => { 
    res.send('维基主页');
    });

    // “关于页面”路由：该回调有三个参数（通常命名为：req、res、next），分别是：HTTP 请求对象、HTTP 响应、中间件链中的下一个函数。
    router.get('/about', (req, res) => {
    res.send('关于此维基');
    });

    module.exports = router; //导出该 Router 对象

在主应用中使用该路由模块：

    const wiki = require('./wiki.js');
    // ...
    app.use('/wiki', wiki); //对 Express 应用对象调用 use()，即可将其添加到中间件处理路径。

### 6.5 其他

+ [从数据库获取记录以及使用模版的实战经验](https://developer.mozilla.org/zh-CN/docs/Learn/Server-side/Express_Nodejs/Displaying_data)
+ [如何在Express使用HTML表单，Pug，以及从数据库中创建，更新，删除文件](https://developer.mozilla.org/zh-CN/docs/learn/Server-side/Express_Nodejs/forms)

## 七、Web程序的模版 

模版引擎可以把程序逻辑和展示组织好。EJS、Hogan.js和Pug都是Node中比较流行的模版引擎。

### 7.1 模版引擎对比

以下是三种模版引擎的简单对比：

|  | EJS | Hogan.js | Pug |
| ----------- | ----------- | ----------- | ----------- |
| 复杂程度 | 简单 | 简单 | 较复杂 |
| 创建模版 | const template = '<%= message %>' | const template = '{{ message }}' | 用缩紧表示嵌入关系，不使用关闭标签 |
| 字符转义 | <%= 自动转义；<%- 不做转义； | {{}} 自动转义；{{{}}}/{{& }} 不做转义； |  |
| 调整分隔符 | ejs.delimiter = ”$“ | hogan.compile(text, { delimiters: '<% %>' }) |  |
| 是否支持流程控制 | 支持 | 不支持，但支持Mustache标准 | 支持 |
| 特性 | 嵌入JavaScript，可以用在服务器端或客户端 | 通过子模版在其他模版中重用模版 | 嵌入JavaScript，可以用在服务器端或客户端；模版继承和mixins |

### 7.2 EJS创建模版

    const ejs = require('ejs')；
    const template = '<% message %>';
    const context = { message: 'Hello template' };
    ejs.render(template, context)

### 7.3 Hogan创建模版

    const hogan = require('hogan.js')；
    const templateSource = '{{ message }}';
    const context = { message: 'Hello template' };
    const template = hogan.compile(templateSource);
    template.render(context)

### 7.4 Pug创建模版

    const pug = require('pug')；
    const template = '#{message}';
    const context = { message: 'Hello template' };
    const fn = pug.compile(template)
    fn(context)

总体来说，数据传给模版引擎方式是一样的：模版先被编译成函数，然后带着上下文调用它，以便渲染HTML输出。



## 八、存储数据

### 8.1 数据库分类

+ 关系型数据库：以关系代数和集合论为理论基础，用模版指定各种数据类型的歌是和这些数据类型之间的关系。比如：PostgreSQL（Postgres）。
+ NoSQL数据库：非关系模型的数据存储统称为NoSQL，此类数据库没有模式定义、重复的数据、松散的强制性约束。比如：键值存储的Redis和文档存储的MongoDB。

Node既能用关系型数据库，也能用NoSQL数据库。

以下内容简单介绍数据库在Node中使用

### 8.2 PostgreSQL

### 8.2.1 安装及配置

不同平台上的安装是不同的。安装完成后，启动Postgres进程。

### 8.2.2 创建数据库

Postgres守护进程跑起来后就可以创建数据库了：

    createdb articles //创建了名为articles的库

要删掉已有数据库中的全部数据，使用以下命令：

    dropdb articles

### 8.2.3 从Node中连接Postgres

在Node中与Postgres交换，使用的是npm的pg包。

    const pg = require('pg');
    const db = new pg.Client({database: 'articles'}); //配置连接参数
    db.connect((err, client)=>{
        if(err) throw err;
        console.log('连接数据库', db.database)
        db.end(); //关闭数据库连接，Node进程可以退出
    })

### 8.2.4 表的增删改查

使用CREATE定义表，使用INSERT插入数据，使用UPDATE更新数据，使用SELECT查询数据。

使用sql进行操作，操作模版为：

    db.query(sql, (err, result) => {
        if(err) throw err;
        // 使用result做些事情
        db.end();
    })

### 8.3 MongoDB

#### 8.3.1 安装和配置

不同系统上的MongoDB安装是不一样的。MongoDB服务器是用可执行文件mongod启动的。MongoDB驱动为官方的mongodb包。

#### 8.3.2 连接MongoDB

装好mongodb包，启动了mongod服务器后，可以从Node中作为客户端连接它。

    const { MongoClient, ObjectId } = require('mongodb');

    const url = 'mongodb://localhost:27017';
    const client = new MongoClient(url);
    const dbName = 'myProject';

    async function main() {
        await client.connect();
        console.log('Connected successfully to server');
        const db = client.db(dbName);
        const collection = db.collection('documents');
        return 'done.';
    }

    main()
    .then(console.log)
    .catch(console.error)
    .finally(() => client.close());

#### 8.3.3 增删改查

大部分数据库交互都是通过collection API完成的。举例如下：

    const insertResult = await collection.insertMany([{ a: 1 }, { a: 2 }, { a: 3 }]); //插入多个文档

    const article = {
        title: 'i like cake',
        content: 'it is quite good'
    }
    const result = await collection.insertOne(article) //插入单个文档

    const findResult = await collection.find({title: 'i like cake'}).toArray(); //找到跟查询匹配的文档

    const filteredDocs = await collection.findOne({ _id: new ObjectId("624af5a356444cc80f69cef7")}); //找到一个跟查询匹配的文档

### 8.4 Redis

#### 8.4.1 安装和配置

可以用系统上的包管理工具安装Redis。用可执行文件redis-server启动服务器.

#### 8.4.2 初始化

Redis客户端实力使用redis npm包的createClient函数创建：

    const { createClient } = require('redis');

    (async () => {
        const client = createClient({
        url: 'redis://:root@127.0.0.1:6379'
        });

        client.on('error', (err) => console.log('Redis客户端错误', err));

        await client.connect();
    })()


#### 8.4.3 增删改查

**处理键值对**

Redis可以当作普通的键值存储用，支持字符串和任何二进制数据。分别用get和set方法读写键值对：

    await client.set('key', 'value');
    const value = await client.get('key');
    console.log(value, 'value') //value

**处理键**

exists可以检查某个键是否存在，它能接受任何数据类型：

    const isExist = await client.exists('users')
    console.log('users exist', isExist) //0

**使用散列表**

散列表是键值对的数据集。举例如下：

    await client.HSET('camping', { //设定散列表键值对
        shelter: '2-person tent',
        cooking: 'campstove'
    })

    const cooking  = await client.HGET('camping', 'cooking') //获取值
    console.log(cooking, 'cooking') //campstove

    const hkeys = await client.HKEYS('camping') //以数组形式获取散列键
    console.log(hkeys, 'hkeys') //['shelter', 'cooking']

**使用列表**

列表是包含字符串值的有序数据集，可以存在同一值的多个副本。举例如下：

    await client.LPUSH('tasks', 'red') //将值存到列表中
    await client.LPUSH('tasks', 'green')
    const tasks = await client.LRANGE('tasks', 0, -1) //读取值
    console.log(tasks, 'tasks') //['green', 'red']

**使用集合**

集合是无序数据集，其中不允许有重复值。举例如下：

    await client.SADD('admins', 'alice');
    await client.SADD('admins', 'bob');
    await client.SADD('admins', 'alice');
    const members = await client.SMEMBERS('admins')
    console.log(members, 'members') //['alice', 'bob']

### 8.5 客户端存储

请查看我的另外一篇文章[《客户端存储》](https://github.com/jiaoyang2014uk/weekly/blob/feat-docs-client-side-storage/article/2022/%E5%AE%A2%E6%88%B7%E7%AB%AF%E5%AD%98%E5%82%A8/index.md)

## 九、测试Node程序

### 9.1 测试分类

+ 单元测试：直接测试代码逻辑，通常是在函数或方法层面，适用于所有类型的程序。分为两大形态：测试驱动开发TDD和行为驱动开发BDD。比如：Node自带的assert模块（TDD）和Mocha（TDD或BDD）
+ 验收测试：一般是对Web程序进行的额外测试，需要用脚本控制浏览器来触发Web程序的功能。

### 9.2 assert模块

assert模块可以测试一个条件，如果条件未满足，则抛出错误。

#### 9.2.1 用equal检查变量的值

    assert.equal(actual, expected[, message])

使用 == 运算符测试 actual 和 expected 参数之间的浅层强制相等。 NaN 是特殊处理的，如果双方都是 NaN，则视为相同。

如果值不相等，则抛出 AssertionError，其 message 属性设置为等于 message 参数的值。 如果未定义 message 参数，则分配默认错误消息。 如果 message 参数是 Error 的实例，则将抛出错误而不是 AssertionError。

#### 9.2.2 用notEqual找出逻辑中的问题

    assert.notEqual(actual, expected[, message])

使用 != 运算符测试浅层强制不相等。 NaN和结果同equal处理。

#### 9.2.3 用ok测试异步值是否为true

    assert.ok(value[, message])

测试 value 是否为真。 相当于 assert.equal(!!value, true, message)。

如果 value 不是真值，则抛出 AssertionError，其 message 属性设置为等于 message 参数的值。 如果 message 参数为 undefined，则分配默认错误消息。 如果 message 参数是 Error 的实例，则将抛出错误而不是 AssertionError。 如果根本没有传入任何参数，则 message 将设置为字符串：'No value argument passed to `assert.ok()`'。

#### 9.2.4 测试能否正常抛出错误

    assert.throws(fn[, error][, message])

期望函数 fn 抛出错误。

如果指定，且如果 fn 调用失败或错误验证失败，则 message 将附加到 AssertionError 提供的消息。

### 9.3 Mocha

Mocha测试默认使用BDD风格的函数定义和设置，包含describe、it、before、after、beforeEach、afterEach。另外Mocha也有TDD接口，用suite代替describe，test代替it，setup代替before，teardown代替after。本文使用默认风格。

    const assert = require('assert')
    describe('memdb', ()=>{
        describe('sync .saveSync(doc)', ()=>{
            it('should save the doc', ()=>{
                const pet = {name: 'tom'}
                memdb.saveSync(pet)
                const ret = memdb.first({name: 'tom'})
                assert(ret == pet)
            })
        })
    })

## 十、参考

1. [Express Web Framework](https://developer.mozilla.org/zh-CN/docs/learn/Server-side/Express_Nodejs)

2. [Node.js中文网](http://nodejs.cn/)

3. 《Node.js实战（第2版）》
