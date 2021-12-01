# 使用Node.js和shell实现前端工程的自动初始化

## 一、背景

当我们在一个大的前端工程中，新增模块或者子工程时，总是要先做大量无聊的重复的初始化工作。比如，新增一个页面，需要给新页面新增对应的组件文件，路由，以及依赖引入等。如果有一个工具，可以自动化一键创建所需的页面和代码，那么既可以节省大量的时间，同时又可以避免人工编码时产生的错误，这对我们的开发工作是一个很好的辅助。


## 二、方案对比

自动化脚手架本质上是一段基于某种特定的脚本语言编写的简单程序。执行该程序可以按照工程的骨架结构，代码规范，以及一些组件模板，创建指定新模块或者子工程的代码文件以及填充一些代码片段。理论来说，可以使用任意一种可以调用操作系统API的语言实现，比如C， Java， Bash， Node.js等。考虑到使用的场景，我们需要更加轻便灵巧的方式达到这一目的，那么shell和Node.js是比较好的选择。
我们对这两种方式的实现做了一个对比：


|  对比项   | bash  | Node.js  |
|  ----  | ----  | ----  | 
| 上手难度（对于前端）  | 稍难 | 容易  |
| 与前端工程的融合性  | 一般 | 很好（本质上使用JavaScript语言实现）  |
| 环境兼容性  | Unix / Linux下原生支持，Window平台下需要安装特定的bash环境 | 所有平台原生并不支持，但在装有Node后可以跨平台使用  |

基于以上对比，我们发现在前端工程中，使用Node.js作为项目脚手架的开发语言，要比使用bash更有优势。

## 三、Node.js实现常用的库及函数介绍

### 1.global全局变量

全局变量在所有模块中都可用。例如：require， console，__dirname, process等

- **require(id)**

用于导入模块，JSON或者本地文件。可以使用相对路径导入本地模块和本地文件，路径将根据__dirname命名的目录或者当前工作目录进行解析。本例中，用来导入fs和path模块。

```javascript
let fs = require('fs');
let path = require('path');
```

- **console**

用于提供控制台输出，是一个简单的调试控制台。本例使用console.log()：向标准输出流打印字符，并以换行符结束。本例中，当用户未同时输入模块名，子模块名和页面名时，打印出错误。

```
if (args.length < 3) {
    console.log('模块名，子模块名，页面名必填');
    process.exit(0);
}
```

- **__dirname**

表示当前执行脚本所在目录。本例中，用来获取当前执行脚本所在目录，进行目录拼接。

```javascript
if (type === 'form' || type === 'table') {
    indexTplOrigin = fs.readFileSync(path.join(__dirname, type === 'form' ? './template/form_tpl.vue' : './template/table_tpl.vue'), 'utf8');
    apiTpl = fs.readFileSync(path.join(__dirname, type === 'form' ? './template/api_form.js' : './template/api_table.js'), 'utf8');
    constTpl = fs.readFileSync(path.join(__dirname, type === 'form' ? './template/const_form.js' : './template/const_table.js'), 'utf8');
    if (type === 'table') {
        modalTpl = fs.readFileSync(path.join(__dirname, './template/addEditModal.vue'), 'utf8');
    }
} else {
    indexTplOrigin = fs.readFileSync(path.join(__dirname, './template/template.vue'), 'utf8');
}
```

- **process**

详见process进程

### 2.process进程

process对象提供有关当前Node.js进程的信息，并对其进行控制。全局可用，官网推荐通过require或者import显示的引用它。

```javascript
import process from 'process';
```

process对象提供了很多事件，不在本次讨论中，如有兴趣请移步官网查看。

- **process.argv**

process.argv属性返回数组，可以获取到用户启动进程时传入的参数。第一个元素是启动Node.js进程的可执行文件的绝对路径名，第二个元素是正在执行的Javascript文件路径。其他元素为命令行参数。在本例中，用来获取用户输入的模块名，子模块名和页面名。

```javascript
let args = process.argv.splice(2);
```

假设main.js脚本如下：

```javascript
process.argv.forEach((val, index) => {
  console.log(`${index}: ${val}`);
});
```

启动进程后，输出结果：

![](https://files.mdnice.com/user/22004/5566979c-4562-4d6d-87fc-9fdec8169907.png)


- **process.exit([code])**

process.exit()指Node.js以code退出并同步终止进程。code可省略，此时代表使用成功代码0。调用此方法，将强制进程尽快退出，即使仍有未完全完成的异步操作挂起。
在本例中，用来获取用户输入的模块名，子模块名和页面名。在本例中，当用户未同时输入模块名，子模块名和页面名时，并强制退出。

```javascript
if (args.length < 3) {
    console.log('模块名，子模块名，页面名必填');
    process.exit(0);
}
```

### 3.path路径

path模块提供了用于处理文件和目录的路径的使用工具，使用如下：

```javascript
const path = require('path');
```

- **path.join()**

path.join()使用平台分隔符作为定界符将所有给定的path片段连接在一起，生成路径。本例中，用来根据当前执行脚本所在目录，进行目录拼接。

```javascript
if (type === 'form' || type === 'table') {
    indexTplOrigin = fs.readFileSync(path.join(__dirname, type === 'form' ? './template/form_tpl.vue' : './template/table_tpl.vue'), 'utf8');
    apiTpl = fs.readFileSync(path.join(__dirname, type === 'form' ? './template/api_form.js' : './template/api_table.js'), 'utf8');
    constTpl = fs.readFileSync(path.join(__dirname, type === 'form' ? './template/const_form.js' : './template/const_table.js'), 'utf8');
    if (type === 'table') {
        modalTpl = fs.readFileSync(path.join(__dirname, './template/addEditModal.vue'), 'utf8');
    }
} else {
    indexTplOrigin = fs.readFileSync(path.join(__dirname, './template/template.vue'), 'utf8');
}
```

### 4.fs文件系统分类

Node.js的文件系统操作分同步和异步模式。同步模式是在主进程中直接调用文件系统的API,有可能引起进程阻塞，异步操作是把阻塞进程放到子进程处理，主进程可以继续处理其他操作。

![](https://files.mdnice.com/user/22004/fc987c14-4f99-4d6d-87ce-cca40671719c.png)



创建input.txt文件，创建files.js文件，内容如下：

```javascript
var fs = require("fs");

// read
fs.readFile('input.txt', function (err, data) {
   if (err) {
       return console.error(err);
   }
   console.log("read: " + data.toString());
});

// read sync
var data = fs.readFileSync('input.txt');
console.log("read sync: " + data.toString());

console.log("finish");
```

运行结果如下：

![](https://files.mdnice.com/user/22004/84b747b6-3e47-4940-942d-fe0b2b57e7b6.png)

### 5.fs文件系统

导入文件系统语法：

```javascript
const fs = require("fs")
```

| 方法名称 | **fs.existsSync**                              | **fs.mkdirSync**                       | **fs.readFileSync** | **fs.writeFileSync**               |
| -------- | ---------------------------------------------- | -------------------------------------- | ------------------- | ---------------------------------- |
| 作用     | 检测给定的路径是否存在。                       | 同步创建目录。                         | 读取文件内容        | 同步读取文件内容                   |
| 返回值   | 如果路径存在则返回true，否则返回false。        |                                        | 返回path的内容      | 返回undefined                      |
| 异步方法 | fs.exists                                      | fs.mkdir                               | fs.readFile         | fs.writeFile                       |
| 本例作用 | 用来判断模块，子模块，页面所在文件夹是否存在。 | 用来创建模块，子模块，页面所在文件夹。 | 用来读取模板内容。  | 用来将转换后内容，写入目标文件中。 |



- **fs.readFileSync(path [, options])**

参数：path：文件名或文件描述符；options：对象或者字符串，默认包含 {encoding, flag}。默认编码为 null, flag 为 'r'(以读取模式打开文件。如果文件不存在抛出异常)

- **fs.writeFileSync(file, data[, options])**

参数：file：文件名或文件描述符；data：要写入文件的数据；options：对象或者字符串，默认包含 {encoding, mode, flag}。默认编码为 utf8, 模式为 0666 ， flag 为 'w'(以写入模式打开文件，如果文件不存在则创建)

```javascript
//新建模块文件夹
if (!fs.existsSync(`${controllerPath}/${module1}`)) {
    fs.mkdirSync(`${controllerPath}/${module1}`);
    fs.mkdirSync(`${viewPath}/${module1}`);
}

//新建子模块文件夹
if (!fs.existsSync(`${controllerPath}/${module1}/${submodule}`)) {
    fs.mkdirSync(`${controllerPath}/${module1}/${submodule}`);
    fs.mkdirSync(`${viewPath}/${module1}/${submodule}`);
}

//新建页面文件夹
if (!fs.existsSync(`${viewPath}/${module1}/${submodule}/${page}`)) {
    fs.mkdirSync(`${viewPath}/${module1}/${submodule}/${page}`);
}

//modeEngine文件新增权限
let authOrigin = fs.readFileSync(path.join(__dirname, './template/auth.js'), 'utf8');
let formatAuth1 = authOrigin.replace(/module1/g, module1);
let formatAuth2 = formatAuth1.replace(/submodule/g, submodule);
let auth = formatAuth2.replace(/page/g, page);
let data = fs.readFileSync(path.join(__dirname, './apps/modeEngine.js·'), 'utf8').split(/break;/gm);
data.splice(data.length - 2, 0, auth);
fs.writeFileSync('./apps/modeEngine.js', data.join('break;'));
```

## 四、Node.js实现案例

以下是Ext包裹Vue项目中创建目录和新增模板的实现脚本，实现思路如下：

1. 用户需输入最少三个参数，分别为模块名，子模块名和页面名，如果参数不足3个，则直接报错提示，退出脚本。
2. 允许用户输入第四个参数，如果是空，创建空模板，如果是table，创建table模板，如果是form，创建form模板。
3. 需要创建的为controller文件夹，view文件夹，路径为用户输入的参数，如果不存在则创建，否则直接使用系统原有的文件夹。
4. 从模板读取的内容，需要将用户输入的参数替换对应的值后，写入新创建的模板中。
5. 同时，在路由页面，新增权限

```javascript
/**
 * @fileOverview  自动创建目录
 */

let fs = require('fs');
let path = require('path');

let args = process.argv.splice(2);

if (args.length < 3) {
    console.log('模块名，子模块名，页面名必填');
    process.exit(0);
}

// 定义接收变量
let module1 = args[0];
let submodule = args[1];
let page = args[2];
let type = args[3];

// 定义文件路径变量
let controllerPath = './apps/controller';
let controllerPagePath = `${controllerPath}/${module1}/${submodule}/${page}.js`;
let viewPath = './apps/view';
let viewPanelPath = `${viewPath}/${module1}/${submodule}/${page}/Panel.js`;
let viewIndexPath = `${viewPath}/${module1}/${submodule}/${page}/index.vue`;
let viewConstPath = `${viewPath}/${module1}/${submodule}/${page}/const.js`;
let viewApiPath = `${viewPath}/${module1}/${submodule}/${page}/api.js`;
let viewModalPath = `${viewPath}/${module1}/${submodule}/${page}/addEditModal.vue`;

//新建模块文件夹
if (!fs.existsSync(`${controllerPath}/${module1}`)) {
    fs.mkdirSync(`${controllerPath}/${module1}`);
    fs.mkdirSync(`${viewPath}/${module1}`);
}

//新建子模块文件夹
if (!fs.existsSync(`${controllerPath}/${module1}/${submodule}`)) {
    fs.mkdirSync(`${controllerPath}/${module1}/${submodule}`);
    fs.mkdirSync(`${viewPath}/${module1}/${submodule}`);
}

//新建页面文件夹
if (!fs.existsSync(`${viewPath}/${module1}/${submodule}/${page}`)) {
    fs.mkdirSync(`${viewPath}/${module1}/${submodule}/${page}`);
}

// 读取controller模板文件，并修改内容
let controllerTplOrigin = fs.readFileSync(path.join(__dirname, './template/controller_tpl.js'), 'utf8');
let formatConTpl1 = controllerTplOrigin.replace(/submodule/g, submodule);
let formatConTpl2 = formatConTpl1.replace(/module/g, module1);
let controllerTpl = formatConTpl2.replace(/page/g, page);

// 读取panel模板文件，并修改内容
let panelTplOrigin = fs.readFileSync(path.join(__dirname, './template/panel_tpl.js'), 'utf8');
let formatPanelTpl1 = panelTplOrigin.replace(/submodule/g, submodule);
let formatPanelTpl2 = formatPanelTpl1.replace(/module/g, module1);
let panelTpl = formatPanelTpl2.replace(/page/g, page);

// 读取vue模板文件，并修改内容
let indexTplOrigin = '';
let apiTpl = '';
let constTpl = '';
let modalTpl = '';
if (type === 'form' || type === 'table') {
    indexTplOrigin = fs.readFileSync(path.join(__dirname, type === 'form' ? './template/form_tpl.vue' : './template/table_tpl.vue'), 'utf8');
    apiTpl = fs.readFileSync(path.join(__dirname, type === 'form' ? './template/api_form.js' : './template/api_table.js'), 'utf8');
    constTpl = fs.readFileSync(path.join(__dirname, type === 'form' ? './template/const_form.js' : './template/const_table.js'), 'utf8');
    if (type === 'table') {
        modalTpl = fs.readFileSync(path.join(__dirname, './template/addEditModal.vue'), 'utf8');
    }
} else {
    indexTplOrigin = fs.readFileSync(path.join(__dirname, './template/template.vue'), 'utf8');
}

let indexTpl = indexTplOrigin.replace(/page\b/g, page);

//新建路由和页面文件
if (!fs.existsSync(`${controllerPagePath}`)) {
    fs.writeFileSync(`${controllerPagePath}`, controllerTpl);
    fs.writeFileSync(`${viewPanelPath}`, panelTpl);
    fs.writeFileSync(`${viewIndexPath}`, indexTpl);
    if (type === 'form' || type === 'table') {
        fs.writeFileSync(`${viewApiPath}`, apiTpl);
        fs.writeFileSync(`${viewConstPath}`, constTpl);
        if (type === 'table') {
            fs.writeFileSync(`${viewModalPath}`, modalTpl);
        }
    }
} else {
    console.error('error!\n' + controllerPagePath + ' has already been existed!');
}

//modeEngine文件新增权限
let authOrigin = fs.readFileSync(path.join(__dirname, './template/auth.js'), 'utf8');
let formatAuth1 = authOrigin.replace(/module1/g, module1);
let formatAuth2 = formatAuth1.replace(/submodule/g, submodule);
let auth = formatAuth2.replace(/page/g, page);
let data = fs.readFileSync(path.join(__dirname, './apps/modeEngine.js·'), 'utf8').split(/break;/gm);
data.splice(data.length - 2, 0, auth);
fs.writeFileSync('./apps/modeEngine.js', data.join('break;'));

process.exit(0);
```



空模板:

```vue
<template>
    <div class="page">page</div>
</template>

<script lang="ts">

    /**
     * @fileOverview  page
     */
 import {Component, Mixins, Watch, Prop} from 'vue-property-decorator';

 @Component({
        components: {},
        mixins: []
 })
 export default class page extends Mixins() {
     @Prop({ default: false }) disabled!: boolean; //是否禁用

     private list = [];

     @Watch("list", { deep: true })
     handleListChange() {}

     mounted () {}

     listChange () {}
 }

 </script>
<style lang="less" scoped>
</style>
```

## 五、shell实现常用的命令和语法介绍

### 1.shell的定义和运行

- **定义**

shell定义以开头#!/bin/sh，**#!** 告诉系统这个脚本需要什么解释器来执行，即使用哪种shell。

```shell
#!/bin/bash
echo "Hello World !"
```

- **运行**

添加执行权限：chmod + x ./xxxx.sh

执行脚本：./xxxx.sh

### 2.shell变量

- **定义变量**

定义变量时，变量名不用加$符号。如下

```shell
message = "hello world"
```

定义字符串类型时，可以使用双引号，或者单引号。区别是双引号里面可以有变量和转义符，但是单引号里面内容会原样输出。

变量名的命名规则：

- 命名只能使用英文字母，数字和下划线，首个字符不能以数字开头。
- 中间不能有空格，可以使用下划线 **_**。
- 不能使用标点符号。
- 不能使用bash里的关键字（可用help命令查看保留关键字）。

- **使用变量**

使用变量时，只需在定义过的变量名前，加$符号，如下

```shell
message = 'hello world';
echo ${text};
```

变量名是否加{}是可选的，推荐使用，结构更清晰，还能避免解析错误。

本例中，比如定义和使用输入参数，定义和使用模板：

```shell
# 定义接收变量
module=$1
submodule=$2
page=$3

# 定义路径变量
controller_path="./apps/controller"
controller_page_path=$controller_path/$module/$submodule/$page.js
view_path="./apps/view"
view_panel_path=$view_path/$module/$submodule/$page/Panel.js
view_index_path=$view_path/$module/$submodule/$page/index.vue
```

### 3.shell传递参数

在执行shell脚本时，可以向脚本传递参数，脚本内获取参数的格式为$n。$1为执行脚本的第一个参数，$2为执行脚本的第二个参数，以此类推...  $0 为执行的文件名（包含文件路径）

另外，$#为传递到脚本的参数个数。

举例：脚本文件如下

```shell
#!/bin/bash

echo "filename: $0";
echo "total params: $#"
echo "first params: $1";
echo "second params: $2";
echo "third params: $3";
```

运行后结果：

![](https://files.mdnice.com/user/22004/1e8db56c-a6cc-4bfe-945b-305bffe90ca8.png)

本例中，用来接受用户输入的参数和参数总个数

```shell
# 定义接收变量
module=$1
submodule=$2
page=$3

# 判断输入参数是否小于3
if [ $# -lt 3 ]
then
	echo "Usage: $0 <module> <submodule> <page>"
	exit 0
fi
```

### 4.shell运算符
shell运算符包含关系运算符，文件测试运算符，还有常用的算数运算符，布尔运算符和逻辑运算符，后面三种暂不在本次讨论中。

- **关系运算符**

关系运算符只支持数字，不支持字符串，除非字符串的值是数字。

| 运算符 | 说明                                               |
| ------ | -------------------------------------------------- |
| -eq    | 检测两个数是否相等，相等返回true                   |
| -ne    | 检测两个数是否不相等，不相等返回true               |
| -gt    | 检测左边的数是否大于右边的数，如果是，返回true     |
| -lt    | 检测左边的数是否小于右边的数，如果是，返回true     |
| -ge    | 检测左边的数是否大于等于右边的数，如果是，返回true |
| -le    | 检测左边的数是否小于等于右边的数，如果是，返回true |

本例中，使用-lt检测用户输入的参数个数是否满足要处理的参数个数。

```shell
# 判断输入参数是否小于3
if [ $# -lt 3 ]
then
	echo "Usage: $0 <module> <submodule> <page>"
	exit 0
fi
```

- **文件测试运算符**

文件测试运算符用于检测文件的各种属性

| 操作符  | 说明                                                     |
| ------- | -------------------------------------------------------- |
| -d file | 检测文件是否是目录，如果是，则返回 true                  |
| -r file | 检测文件是否可读，如果是，则返回 true。                  |
| -w file | 检测文件是否可写，如果是，则返回 true。                  |
| -s file | 检测文件是否为空（文件大小是否大于0），不为空返回 true。 |
| -e file | 检测文件（包括目录）是否存在，如果是，则返回 true。      |

等等，表格只列举了部分操作符。

本例中，使用-e file检测要创建的目标文件是否存在

```shell
# 检测目标文件是否存在
if [ -e $controller_page_path ]
then
	echo "Error: $controller_page_path exists"
	exit 1
elif [ -e $view_panel_path ]
then
	echo "Error: $view_pannel_path exists"
	exit 1
elif [ -e $view_index_path ]
then
	echo "Error: $view_index_path exists"
	exit 1
fi
```


### 5.shell echo命令

shell的echo命令用于字符串的输出。可以用来输出普通字符串，变量等。

本例中，用来当目标文件存在时，输出变量提示错误

```shell
# 检测目标文件是否存在
if [ -e $controller_page_path ]
then
	echo "Error: $controller_page_path exists"
	exit 1
elif [ -e $view_panel_path ]
then
	echo "Error: $view_pannel_path exists"
	exit 1
elif [ -e $view_index_path ]
then
	echo "Error: $view_index_path exists"
	exit 1
fi
```

另外，echo还有一些可选选项：-e 等

echo -e: 处理特殊字符，如果出现特殊字符，不会按照一般文字输出，需特别加以处理。如：\n  换行，且光标移至行首，\t 插入tab等

举例：执行脚本如下

```shell
#!/bin/bash

echo "hello \n world";
echo -e "hello \n world";
echo "hello \t world";
echo -e "hello \t world";
```

输出结果：

![](https://files.mdnice.com/user/22004/f20ca1ab-413f-4528-ab49-17ef42c8b36a.png)

本例中，用来将带特殊字符的模板文件输出。

```shell
# 模板中有特殊字符，需要-e处理
touch $controller_page_path
echo -e "$controller_tpl" > $controller_page_path

touch $view_panel_path
echo -e "$view_panel_tpl" > $view_panel_path

touch $view_index_path
echo -e "$view_index_tpl" > $view_index_path
```



### 6.shell流程控制

注意：sh 的流程控制不可为空，如果 else 分支没有语句执行，就不要写这个 else。常见的如if else

if 语法格式：

```shell
if condition
then
    command1 
    command2
    ...
    commandN 
fi
```

**if else**格式语法：

```shell
if condition
then
    command1 
    command2
    ...
    commandN
else
    command
fi
```

**if else-if else** 语法格式：

```shell
if condition1
then
    command1
elif condition2 
then 
    command2
else
    commandN
fi
```

本例中，用来判断目标文件和目标文件夹是否存在。

```shell
if [ $submodule ]
then
	mkdir -p $controller_path/$module/$submodule
	mkdir -p $view_path/$moudle/$submodule
fi

if [ $page ]
then
	mkdir -p $view_path/$module/$submodule/$page
fi


if [ -e $controller_page_path ]
then
	echo "Error: $controller_page_path exists"
	exit 1
elif [ -e $view_panel_path ]
then
	echo "Error: $view_pannel_path exists"
	exit 1
elif [ -e $view_index_path ]
then
	echo "Error: $view_index_path exists"
	exit 1
fi
```



### 7.shell输入/输出重定向

| 命令            | 说明                              | 备注                                                |
| --------------- | --------------------------------- | --------------------------------------------------- |
| command > file  | 将输出重定向到 file。             | 执行command，然后将输出的内容存入file，替换原有内容 |
| command < file  | 将输入重定向到 file。             |                                                     |
| command >> file | 将输出以追加的方式重定向到 file。 | 追加到文件末尾                                      |

本例中，用来将模板内容输出到目标文件。

```shell
touch $controller_page_path
echo -e "$controller_tpl" > $controller_page_path

touch $view_panel_path
echo -e "$view_panel_tpl" > $view_panel_path

touch $view_index_path
echo -e "$view_index_tpl" > $view_index_path
```



### 8.sed命令

sed主要是用来对数据进行选取、替换、删除、新增的命令。sed所做的修改并不会直接改变文件的内容，而是把修改结果只显示到屏幕上，除非使用“-i”选项才会直接修改文件。语法格式如下：

```shell
sed [选项] ‘[动作]’ 文件名
```

- **选项**：

| 选项名        | 说明                                                         | 举例                                                    |
| ------------- | ------------------------------------------------------------ | ------------------------------------------------------- |
| -n            | 一般sed命令会把所有数据都输出到屏幕，如果加入此选择，则只会把经过sed命令处理的行输出到屏幕。 | 指定输出input.txt的第二行<br />sed -n  '2p' student.txt |
| -e            | 允许对输入数据应用多条sed命令编辑                            |                                                         |
| -f 脚本文件名 | 从sed脚本中读入sed操作                                       |                                                         |
| -r            | 在sed中支持扩展正则表达式。                                  |                                                         |
| -i            | 用sed的修改结果直接修改读取数据的文件，而不是由屏幕输出      |                                                         |

- **动作**

| 动作名 | 说明                                                         | 举例                                                |
| ------ | ------------------------------------------------------------ | --------------------------------------------------- |
| s      | 字串替换，用一个字符串替换另外一个字符串。格式为“行范围s/"旧字串/新字串/g” |                                                     |
| p      | 打印，输出指定的行。                                         | 查看下input.txt的第二行<br /> sed  '2p' input.txt   |
| d      | 删除，删除指定的行。                                         | 删除第二行到第四行的数据<br />sed  '2,4d' input.txt |
| c \    | 行替换，用c后面的字符串替换原数据行，替换多行时，除最后一行外，每行末尾需用“”代表数据未完结。 |                                                     |

本例中，用来将新增页面的权限添加到modeEngine文件中。

```shell
# modeEngine文件新增权限
sed -i "s/\([[:space:]]*\)default:/\1\n\1case \'$module.$submodule.$page\':\n\1    import(\/\* webpackChunkName: \"$submodule.$page\" \*\/ '.\/controller\/$module\/$submodule\/$page.js').\n\1    then(function () {\n\1        openMode(module, newTab, params);\n\1    });\n\1    break;\n\1    \n\1default:/" ./apps/modeEngine.js
```



### 9.mkdir命令

mkdir创建指定的目录名，语法为

```shell
mkdir [OPTION]... DIRECTORY...
```

选项

| 选项名 | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| -m     | 对新建目录设置存取权限                                       |
| -p     | 可以是一个路径名称。此时若路径中的某些目录尚不存在,加上此选项后,系统将自动建立好那些尚不存在的目录,即一次可以建立多个目录; |
| -v     | 表示打印每一个创建的目录的信息。                             |
| -help  | 显示帮助信息                                                 |

本例中，用来创建目标文件夹。

```shell
mkdir -p $controller_path/$module
mkdir -p $view_path/$module

if [ $submodule ]
then
	mkdir -p $controller_path/$module/$submodule
	mkdir -p $view_path/$moudle/$submodule
fi

if [ $page ]
then
	mkdir -p $view_path/$module/$submodule/$page
fi
```



### 10.touch命令

touch命令主要用来修改文件时间戳，或者新建一个不存在的文件。语法为：

```shell
touch [OPTION]... FILE...
```

选项

| 选项名 | 说明                   |
| ------ | ---------------------- |
| －a    | 改变档案的读取时间记录 |
| －m    | 改变档案的修改时间记录 |

等等

本例中，用来创建目标文件。

```
touch $controller_page_path
echo -e "$controller_tpl" > $controller_page_path

touch $view_panel_path
echo -e "$view_panel_tpl" > $view_panel_path

touch $view_index_path
echo -e "$view_index_tpl" > $view_index_path
```



### 11.exit命令

exit命令用来退出当前 Shell 进程，并返回一个退出状态。exit 命令可以接受一个整数值作为参数，代表退出状态。如果不指定，默认状态值是 0。

### 六、shell实现案例

以下项目和Node.js项目为同一项目背景和流程：

```shell
#!/bin/bash

# 使用说明
# git bash 执行 ./auto_create_dir.sh + 模块名 + 子模块名 + 页面名（均必填）
# 自动添加controller，view的panel和index，和modeEngine权限

# 定义接收变量
module=$1
submodule=$2
page=$3

# 定义路径变量
controller_path="./apps/controller"
controller_page_path=$controller_path/$module/$submodule/$page.js
view_path="./apps/view"
view_panel_path=$view_path/$module/$submodule/$page/Panel.js
view_index_path=$view_path/$module/$submodule/$page/index.vue

# 定义模板
header_info="/**
 * @Author: $USER || $USERNAME || `git config user.name`
 * @Date: $(date "+%Y-%m-%d %H:%M:%S")
 * @Description: $page
 */
"

controller_tpl="$header_info
import 'components/mgr/AbstractMgr';
import 'view/$module/$submodule/$page/Panel';
Ext.define('app.controller.$module.$submodule.$page', {
    extend: 'app.components.mgr.AbstractMgr',
    title: _('$page'),
    views: ['$module.$submodule.$page.Panel']
});
"

view_panel_tpl="$header_info
import View from './index';
Ext.define('app.view.$module.$submodule.$page.Panel', {
    extend: 'Ext.panel.Panel',
    alias: 'widget.$module.$submodule.$page',
    layout: _('fit'),
    title: _('$page'),
    initComponent () {
        this.createItems();
        this.callParent(arguments);
    },
    createItems () {
        this.items = [
            {
                itemId: 'vueComponent',
                xtype: 'box',
                vueCmp: View,
                enableVue: true
            }
        ];
    }
});
"
view_index_tpl="<template>
  <div class=\"demo\">$page</div>
</template>

<script lang=\"ts\">

$header_info
 import {Component, Mixins, Watch, Prop} from 'vue-property-decorator';

 @Component({
        components: {},
        mixins: []
 })
 export default class $page extends Mixins() {
     @Prop({ default: false }) disabled!: boolean; //是否禁用

     private list = [];

     @Watch(\"list\", { deep: true })
     handleListChange() {}

     mounted () {}

     listChange () {}
 }

 </script>
 <style lang="less" scoped>
 </style>
"

if [ $# -lt 3 ]
then
	echo "Usage: $0 <module> <submodule> <page>"
	exit 0
fi

mkdir -p $controller_path/$module
mkdir -p $view_path/$module

if [ $submodule ]
then
	mkdir -p $controller_path/$module/$submodule
	mkdir -p $view_path/$moudle/$submodule
fi

if [ $page ]
then
	mkdir -p $view_path/$module/$submodule/$page
fi


if [ -e $controller_page_path ]
then
	echo "Error: $controller_page_path exists"
	exit 1
elif [ -e $view_panel_path ]
then
	echo "Error: $view_pannel_path exists"
	exit 1
elif [ -e $view_index_path ]
then
	echo "Error: $view_index_path exists"
	exit 1
fi

touch $controller_page_path
echo -e "$controller_tpl" > $controller_page_path

touch $view_panel_path
echo -e "$view_panel_tpl" > $view_panel_path

touch $view_index_path
echo -e "$view_index_tpl" > $view_index_path

# modeEngine文件新增权限
sed -i "s/\([[:space:]]*\)default:/\1\n\1case \'$module.$submodule.$page\':\n\1    import(\/\* webpackChunkName: \"$submodule.$page\" \*\/ '.\/controller\/$module\/$submodule\/$page.js').\n\1    then(function () {\n\1        openMode(module, newTab, params);\n\1    });\n\1    break;\n\1    \n\1default:/" ./apps/modeEngine.js
exit 0
```


## 七、参考

1. Node.js的底层原理 https://juejin.cn/post/7008504029277847565#heading-25
2. Node.js 中文网 http://nodejs.cn/
3. 菜鸟教程 https://www.runoob.com/nodejs/nodejs-tutorial.html
4. 菜鸟教程 https://www.runoob.com/linux/linux-shell.html
5. shell学习教程 https://blog.csdn.net/w918589859/article/details/108752592
6. linux shell ---mkdir和touch命令详解  https://blog.csdn.net/web_go_run/article/details/46010741
