# TS 类型基本用法



## TS简介

- TypeScript，简称 TS， 是一种由微软开发的编程语言，它是对 JavaScript 的一个增强
- 让我们更加方便地进行类型检查和代码重构，提高代码的可靠性和可维护性
- 同时，TypeScript 还支持 ECMAScript 的最新特性

## 搭建学习环境
进入 Node 官网安装
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1676787464593-021e7fff-9076-4619-aa71-0b8b401baf61.png#averageHue=%23f9f9f8&clientId=u3c81feae-bd68-4&from=paste&height=345&id=ue3495a2a&name=image.png&originHeight=778&originWidth=1578&originalType=binary&ratio=2&rotation=0&showTitle=false&size=131065&status=done&style=none&taskId=u1299fbc0-24ac-4ee6-977a-a7577903420&title=&width=699)
安装完成后使用以下命令查看是否安装完成：
安装完成后使用以下命令查看是否安装完成：

- `node -v`
- `npm -v`

继续安装 nrm 管理包源：

- `npm i nrm -g`
- `nrm ls`
- `nrm use taobao`

全局安装 typescript：

- `npm i typescript -g`

全局安装 ts 的编译工具，使用 ts-node 可以将 ts 文件执行

- `npm i ts-node -g`
   - 使用：`ts-node index.ts`
- 安装 ts-node 依赖包：`npm install tslib @types/node -g`

使用 TS 可以有良好的提示，使代码可读性变强，更提前发现问题

---

## TS 类型

- TS 出现弥补的 JS 的类型缺失
- 众所周知，代码错误越早发现越好，`代码编写 > 代码编译 > 代码运行`   `开发 > 测试 > 上线`
- Vue2 使用 `Flow` 进行类型检查，后续 Vue3 也使用 `Typescript` 重写
- TS 代码要运行在浏览器，需要进行类型擦除，转换为 JS 代码
- TS 类型包含所有 JS 类型 null、undefined、string、number、boolean、bigInt、Symbol、object（数组，对象，函数，日期）
- 还包含 void、never、enum、unknown、any 以及 自定义的 type 和 interface

### 变量声明

- `var/let/const 标识符: 数据类型 = 赋值`

手动指定数据的类型（类型注解），不要写成大写的 `String` ，因为这是 JS 的一个内置类
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678120624537-4c3dec7c-d5cb-4c95-a450-f48bfebe3c6b.png#averageHue=%2339322c&clientId=u95f70254-8dcd-4&from=paste&height=28&id=ub525eeff&name=image.png&originHeight=56&originWidth=694&originalType=binary&ratio=2&rotation=0&showTitle=false&size=9294&status=done&style=none&taskId=u7eae5e21-8153-4fb8-abc8-4e5e9159294&title=&width=347)
变量类型定义的时候已经决定
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678458540049-8f91f233-929a-45c4-9651-4c6c68d61765.png#averageHue=%239b6d38&clientId=ud54645fa-aac9-4&from=paste&height=129&id=uf80949dd&name=image.png&originHeight=258&originWidth=672&originalType=binary&ratio=2&rotation=0&showTitle=false&size=22659&status=done&style=none&taskId=ude0bd6cc-9734-47db-b0e3-2f7e4492bbc&title=&width=336)

---


### 类型推导

- 如果没有明确指定类型，TS 会隐式的推导出一个类型
- 这类型根据赋值的类型推断，没有赋值则为 `any` 类型，能自动推导出类型，没必要手动指定

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678121138325-52d0c9f9-0a1e-49f9-8292-8a13840887c1.png#averageHue=%235e7551&clientId=u95f70254-8dcd-4&from=paste&height=90&id=u7860c1d1&name=image.png&originHeight=180&originWidth=596&originalType=binary&ratio=2&rotation=0&showTitle=false&size=11913&status=done&style=none&taskId=u5dfa05d8-2eb7-4f5f-b51f-82dcff76eb6&title=&width=298)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678121112379-d0b731f3-9cd1-4720-ba1a-8371d86d73d9.png#averageHue=%23343231&clientId=u95f70254-8dcd-4&from=paste&height=67&id=u2c073e2e&name=image.png&originHeight=134&originWidth=510&originalType=binary&ratio=2&rotation=0&showTitle=false&size=9303&status=done&style=none&taskId=u020ec99d-fd22-42a5-8ad9-f0bb57468ca&title=&width=255)

---

### 基础类型
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678458653766-7bdd17b6-74c7-448a-a9af-903e05e7d207.png#averageHue=%23302f2d&clientId=ud54645fa-aac9-4&from=paste&height=665&id=u79990ec0&name=image.png&originHeight=1330&originWidth=1212&originalType=binary&ratio=2&rotation=0&showTitle=false&size=165062&status=done&style=none&taskId=u2337f67a-58ec-4309-88a4-3ec2aa027ce&title=&width=606)

---


### 数组和元组
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678458920138-3f461dd7-9d60-4054-ac1f-9a6b7fb6401e.png#averageHue=%23353331&clientId=ud54645fa-aac9-4&from=paste&height=268&id=ub5e4deeb&name=image.png&originHeight=536&originWidth=1106&originalType=binary&ratio=2&rotation=0&showTitle=false&size=120642&status=done&style=none&taskId=ube23bafc-51e5-4194-87db-5b8b4fda60b&title=&width=553)

- tuple可以作为函数返回的值，React 的 useState 就是个元组，类似于

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678459012405-984a224e-53fc-485e-a7fa-73db4e4930f6.png#averageHue=%232e2d2b&clientId=ud54645fa-aac9-4&from=paste&height=327&id=udd3c6c92&name=image.png&originHeight=654&originWidth=1404&originalType=binary&ratio=2&rotation=0&showTitle=false&size=87187&status=done&style=none&taskId=ua6e9d3ed-d25a-4424-ad9e-b1be3944625&title=&width=702)

---


### 对象类型

- TS 中的 object 类型泛指所有的的非原始类型，如对象、数组、函数
- 下面我们使用 object 声明了这个对象，但是这个对象既不能设置新数据，也不能修改老数据

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678122199150-0fa5fee4-c196-414d-96e6-c5210052ac88.png#averageHue=%232f2d2b&clientId=u95f70254-8dcd-4&from=paste&height=186&id=ud40ff06f&name=image.png&originHeight=372&originWidth=532&originalType=binary&ratio=2&rotation=0&showTitle=false&size=26614&status=done&style=none&taskId=ud0088020-24d5-472a-9aa3-c7f4575af77&title=&width=266)

- 下面这种对象类型的限制才更为精确
- 可限制对象每个属性的类型

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678459243605-7f08b94e-5d6e-4ed1-8ab4-0ded638a8b56.png#averageHue=%2334312d&clientId=ud54645fa-aac9-4&from=paste&height=90&id=ue8e1a552&name=image.png&originHeight=180&originWidth=1548&originalType=binary&ratio=2&rotation=0&showTitle=false&size=53469&status=done&style=none&taskId=u21ad0689-2aab-4128-810c-193a95eb643&title=&width=774)

---

### any、unknown、never

- 无法确定一个变量的类型，可使用 `any`，此时在其身上做任何操作都是合法的，即使访问了一个不存在的属性
- 如果某些情况处理类型过于繁琐，或者在引入一些第三方库时，缺失了类型注解，这个时候 我们可以使用 any，更多是为了兼容老代码

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678459436344-33a2088c-32be-46b6-b840-ef561540b97c.png#averageHue=%2334312c&clientId=ud54645fa-aac9-4&from=paste&height=85&id=u56f8607f&name=image.png&originHeight=170&originWidth=672&originalType=binary&ratio=2&rotation=0&showTitle=false&size=20622&status=done&style=none&taskId=u15ed23ec-4918-4149-a395-555435ba553&title=&width=336)
如果想要 msg 不标注 any，默认也是 any 类型，但如果我们不想这种隐式的 any，可以新建 `tsconfig.json`，书写以下配置：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678459500758-07b4afba-b099-4381-822d-7b0dc84c9144.png#averageHue=%232c2c2b&clientId=ud54645fa-aac9-4&from=paste&height=148&id=u2aca917c&name=image.png&originHeight=296&originWidth=598&originalType=binary&ratio=2&rotation=0&showTitle=false&size=17839&status=done&style=none&taskId=u5ebb71e4-34d7-4ed8-94d8-9d8bf5cfd4d&title=&width=299)

---

- `unknown` 类型表示一个值可以是任何类型，它是所有类型的父类型，任何类型都可以赋值给 unknown 类型，但是 unknown 类型只能赋值给 any 类型和 unknown 类型本身
- 类似 `any`，与 `any` 类型不同的是，`unknown` 类型的变量不能直接赋值给其他类型的变量，也不能调用其上的任何方法或属性，除非先进行类型检查或类型断言，这样确保运行时的类型安全
- 默认在其操作都是不合法的，主要是在编写通用代码时，例如编写库或框架时，需要处理来自不同来源的数据，但又不确定数据的类型

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678891829111-ee89b3f9-87ae-4a0f-80c1-d48f84c60a19.png#averageHue=%2333312d&clientId=u966ae342-cb49-4&from=paste&height=404&id=u1d846f66&name=image.png&originHeight=808&originWidth=912&originalType=binary&ratio=2&rotation=0&showTitle=false&size=117224&status=done&style=none&taskId=uc416b537-d34c-4c78-8f2b-c1704c9515c&title=&width=456)

---

- 假如一个函数的返回结果是死循环或者异常，我们可以使用 `never` 类型表示这种永不存在值的类型
- 它是一个底层类型，不是任何类型的子类型，也没有任何子类型	
- 更多情况是封装工具库时候可以使用，比如下面这段代码，如果单纯在函数参数的类型多加一个参数，而没有对应 `case` 处理，则会报错

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678460700298-571eeb9d-312e-4a50-9092-c6c262754266.png#averageHue=%232f2d2b&clientId=u16232f78-f54c-4&from=paste&height=473&id=u3acee9d9&name=image.png&originHeight=946&originWidth=988&originalType=binary&ratio=2&rotation=0&showTitle=false&size=76965&status=done&style=none&taskId=u05986df7-11c2-4e5e-8901-8b339d5aef1&title=&width=494)
`never` 会在联合类型被直接移除
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678808745311-d0950277-ba73-4fe2-84e4-3d338bd8d94b.png#averageHue=%23b46c31&clientId=u2d00128e-d8db-4&from=paste&height=88&id=u8df2bb49&name=image.png&originHeight=176&originWidth=722&originalType=binary&ratio=2&rotation=0&showTitle=false&size=15210&status=done&style=none&taskId=udfa0128d-382b-4a94-9ae4-1845ac59ea7&title=&width=361)

---

### 函数类型

- 声明函数时，可以在每个参数后添加类型注解，声明其参数类型
- 同样也可以声明返回值的类型，不过也可以不写让 TS 自动推导
- 函数参数的一般顺序 必传参数 - 有默认值的参数 - 可选参数

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678503441194-1d7ab1fd-54bf-4714-be7d-793822df6966.png#averageHue=%23302e2d&clientId=ua79daa53-6c9d-4&from=paste&height=437&id=ua0187bee&name=image.png&originHeight=874&originWidth=1882&originalType=binary&ratio=2&rotation=0&showTitle=false&size=145766&status=done&style=none&taskId=ua2ff1757-fc5e-4304-864e-75ed1d951ca&title=&width=941)

---


### 枚举类型

- 枚举类型将一组可能出现的值，一个个列举，定义在一个类型中，这个类型就是枚举

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678463942803-baecbb30-3b5a-4756-ad9d-50ea18776738.png#averageHue=%232d2d2c&clientId=uc72cbd69-3adc-4&from=paste&height=838&id=ue6d11e35&name=image.png&originHeight=1676&originWidth=1604&originalType=binary&ratio=2&rotation=0&showTitle=false&size=252868&status=done&style=none&taskId=u29403ce7-eed6-48c9-9cdb-59591a87e72&title=&width=802)
这种字符串的枚举可能使用 `type Direction = 'LEFT' | 'RIGHT' | 'TOP' | 'RIGHT'`可能会更好点
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678640964383-79764051-5292-479e-b5ed-a22271544cb6.png#averageHue=%23302e2c&clientId=u7ca94bf8-2f3c-4&from=paste&height=146&id=u7d129127&name=image.png&originHeight=292&originWidth=1212&originalType=binary&ratio=2&rotation=0&showTitle=false&size=36599&status=done&style=none&taskId=u3ff40427-4011-48f9-ba31-820af530f1f&title=&width=606)
---



## interface 和 type

### 基本使用

- 使用 interface 定义接口，使用 type 定义类型别名
- 都可以约束对象的结构

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678503849681-efbfb5b4-35d5-4e8c-a3f1-f858329af70c.png#averageHue=%23302e2d&clientId=u201382e8-0dbe-4&from=paste&height=352&id=u737f0dc0&name=image.png&originHeight=704&originWidth=918&originalType=binary&ratio=2&rotation=0&showTitle=false&size=64292&status=done&style=none&taskId=u1fe731fe-5d68-40c6-a339-ad2ae402cd4&title=&width=459)

---

### 区别

1. interface 只描述对象，type 则可以描述所有数据
2. interface 使用 extends 来实现继承，type 使用 & 来实现交叉类型
3. interface 会创建新的类型名，type 只是创建类型别名，并没有创建新类型
4. interface 可以重复声明扩展，type 则不行（别名是不能重复的）

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678507322844-46d38b8d-f310-4fea-9145-bce1c74f4854.png#averageHue=%23eaeaea&clientId=u37a9fd67-5167-4&from=paste&height=351&id=ub6d315c1&name=image.png&originHeight=1176&originWidth=1918&originalType=binary&ratio=2&rotation=0&showTitle=false&size=274571&status=done&style=none&taskId=u146e34a8-1b29-45f1-9154-55747fa2190&title=&width=573)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678509522531-3d6ee0d9-cfa9-462d-abc1-e8fe2ce0ce83.png#averageHue=%232f2d2b&clientId=ub8af351f-d6d5-4&from=paste&height=469&id=uffc2e854&name=image.png&originHeight=938&originWidth=978&originalType=binary&ratio=2&rotation=0&showTitle=false&size=97491&status=done&style=none&taskId=uf0c4c0d0-9a05-424c-9dce-25c2601bbde&title=&width=489)

---

### 索引签名（Index Signatures）

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678507454285-c0846ebc-3ec5-4ade-af91-32df2da9e13b.png#averageHue=%232e2d2c&clientId=u37a9fd67-5167-4&from=paste&height=324&id=u9c343f7f&name=image.png&originHeight=820&originWidth=1438&originalType=binary&ratio=2&rotation=0&showTitle=false&size=96712&status=done&style=none&taskId=u429d9db0-b7df-4b46-b216-77b72757510&title=&width=568)

---

### 接口继承

- 接口和类继承相同，都是使用 `extends` 关键字
- 接口是支持多继承的（类不支持多继承）

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678517280068-97754686-97bb-47da-82f1-388ae5019d0e.png#averageHue=%232d2c2b&clientId=u184d7bb6-3faa-4&from=paste&height=443&id=u4a572e75&name=image.png&originHeight=1162&originWidth=1794&originalType=binary&ratio=2&rotation=0&showTitle=false&size=121407&status=done&style=none&taskId=ucf16e553-69c9-4bc3-8a56-81bf5bbaaaa&title=&width=684)

---

### 接口实现

- 定义的接口可以被类实现
- 之后如果需要传入接口的地方，同样也可以将类实例传入
- 这就是面向接口开发

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678508269561-503ae4e2-883c-4743-af1c-08a6f44510d8.png#averageHue=%232f2d2b&clientId=u1ff926c3-8644-4&from=paste&height=493&id=u801eb19c&name=image.png&originHeight=1104&originWidth=928&originalType=binary&ratio=2&rotation=0&showTitle=false&size=83781&status=done&style=none&taskId=u78f8578f-35b7-4ea8-88b4-c0a44b6e98e&title=&width=414)

## 函数

### 基本使用

-  我们可以编写**函数类型的表达式（Function Type Expressions）**，来表示函数类型

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678589417969-2e59a128-065e-43d7-ac04-bb9096be1fe8.png#averageHue=%23302e2d&clientId=u5375e113-462c-4&from=paste&height=472&id=u3430569d&name=image.png&originHeight=1058&originWidth=1228&originalType=binary&ratio=2&rotation=0&showTitle=false&size=143920&status=done&style=none&taskId=u7fc13415-8cb0-4ccb-952f-6ff8e95c80b&title=&width=548)

---

更多细节使用
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678544663407-8eacb5e9-13c7-40d3-953b-17876cb0eb32.png#averageHue=%232d2d2c&clientId=ud83c2f35-ca1c-4&from=paste&height=229&id=u1f3bb5c2&name=image.png&originHeight=586&originWidth=2108&originalType=binary&ratio=2&rotation=0&showTitle=false&size=105487&status=done&style=none&taskId=ufe9d7169-1efe-47a7-8d36-ebc5403d142&title=&width=822)

---

### 调用签名（Call Signatures）

- 函数除了被调用，也可以有自己的属性值

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678517603448-d517ba90-3398-40a0-a6be-8337494849eb.png#averageHue=%232f2e2c&clientId=u184d7bb6-3faa-4&from=paste&height=294&id=u41540c27&name=image.png&originHeight=588&originWidth=898&originalType=binary&ratio=2&rotation=0&showTitle=false&size=68055&status=done&style=none&taskId=u93d8f942-fa54-4611-b482-76bda795e28&title=&width=449)

---

### 构造签名 （Construct Signatures）

- 函数也可以使用 new 操作符去当作构造函数
- 使用构造签名，即在调用签名前面加 new 关键词

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678517957630-7a71b20c-02cb-46b6-bf98-b5e81c52d066.png#averageHue=%232e2d2b&clientId=u184d7bb6-3faa-4&from=paste&height=462&id=u1401f87f&name=image.png&originHeight=998&originWidth=1222&originalType=binary&ratio=2&rotation=0&showTitle=false&size=107168&status=done&style=none&taskId=u2476b3f5-04a7-4594-8a96-6f5329d79f5&title=&width=566)

### this 

- TS 中默认情况下如果没有指定 this 的类型，this 是  any 类型
- 我们可以在函数第一个参数声明 this 的类型，函数调用传入的参数从第二个开始接收

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678518417650-2e1f2363-c862-483b-85fd-a9c94f90651f.png#averageHue=%23302e2d&clientId=u184d7bb6-3faa-4&from=paste&height=174&id=u5c572600&name=image.png&originHeight=348&originWidth=1122&originalType=binary&ratio=2&rotation=0&showTitle=false&size=52032&status=done&style=none&taskId=u581b7950-8f57-40de-81db-7b3aa767d64&title=&width=561)

---

- `ThisParameterType` 提取函数类型的 this 参数类型，如果没有 this 参数则返回 unknown
- `OmitThisParameter` 移除函数类型 this 参数类型，返回当前函数类型

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678529473891-1f0c7376-985d-4918-9ed2-e845441ab613.png#averageHue=%23312f2d&clientId=uc9ff726f-8d82-4&from=paste&height=262&id=ue96a7f71&name=image.png&originHeight=640&originWidth=1808&originalType=binary&ratio=2&rotation=0&showTitle=false&size=114093&status=done&style=none&taskId=u38ce8e0c-797f-4a2b-b0a4-5b1b73cf2d8&title=&width=741)

---

- `ThisType` 指定所在对象的所有方法里面 this 的类型

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678531965445-a0201826-f0f7-4d29-8478-5a2b82581f58.png#averageHue=%23212121&clientId=ua805e6e9-6110-4&from=paste&height=599&id=ubc0e7393&name=image.png&originHeight=1198&originWidth=1010&originalType=binary&ratio=2&rotation=0&showTitle=false&size=120024&status=done&style=none&taskId=ufd3112c4-f2c6-4cb2-8b7a-eab7c67d312&title=&width=505)

## 联合类型、交叉类型、函数重载

### 联合类型（Union Type）和重载

- TS 允许我们使用多种运算符，从**现有类型中构建新类型**
- 联合类型就是一种组合类型的方式，**多种类型满足一个即可**，使用 | 符号，其中每个联合的类型被称之为**联合成员（union's members）**
- 函数重载则是我们可以去编写不同的**重载签名（overload signatures）**表示函数可以不同的方式调用，一般写两个及以上的重载签名，再编写一个通用函数的实现

假如现在有个函数，可以传入字符串或数组，以获取长度
方式一：联合类型
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678501859631-4e854170-e064-476c-9339-08c28e4a5936.png#averageHue=%23312f2c&clientId=ude4d6d26-3fb0-4&from=paste&height=189&id=u51286552&name=image.png&originHeight=378&originWidth=1016&originalType=binary&ratio=2&rotation=0&showTitle=false&size=44597&status=done&style=none&taskId=u5e9a22f9-49a0-4c05-afa4-ad9e7f0cef1&title=&width=508)

方式二：函数重载
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678501815696-cb9f4d56-9a76-45e3-9257-97c5eca47f34.png#averageHue=%2335312c&clientId=ude4d6d26-3fb0-4&from=paste&height=266&id=u165ab0c0&name=image.png&originHeight=532&originWidth=1298&originalType=binary&ratio=2&rotation=0&showTitle=false&size=87833&status=done&style=none&taskId=u43b9d2f9-8beb-4869-a294-8ab9a1b6604&title=&width=649)
开发中，尽量使用联合类型，更易阅读

---

- 交叉类型则是**满足多个类型**的条件，使用 & 符号
- 例如 `type MyType = number & string`，满足一个既要是 number 类型，也要是 string 类型的值，显然没有值满足，则会交叉成 never 类型
- 进行交叉时，通常是使用**对象类型交叉**

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678502806117-5dcbcde9-b8e6-4705-87e5-dd9489b81dd4.png#averageHue=%232f2d2b&clientId=u27c9673f-2957-4&from=paste&height=265&id=u534d579d&name=image.png&originHeight=530&originWidth=738&originalType=binary&ratio=2&rotation=0&showTitle=false&size=39760&status=done&style=none&taskId=ub722bb88-b2eb-473a-aeff-3d70d480349&title=&width=369)

---


## 类型、非空、常量断言

- 类型断言 `as`，当 TS 无法获取到具体的类型信息，就需要使用**类型断言（Type Assertions）**
- 它可以允许我们断言成更具体或者不太具体的（比如any）的类型

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678539172875-10da159a-e7b6-4235-b575-59b696783f4d.png#averageHue=%23302f2e&clientId=u03bf352b-c00c-4&from=paste&height=288&id=uc9579815&name=image.png&originHeight=576&originWidth=1578&originalType=binary&ratio=2&rotation=0&showTitle=false&size=122325&status=done&style=none&taskId=ue5bde99b-c559-4823-9db1-aaf900bf3af&title=&width=789)

---

- 非空类型断言 `!`，当我们确定参数有值，需要跳过 TS 对它的检测的时候可以使用

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678539583884-80cfaf0f-d264-4dc3-a0c1-314f1fccd630.png#averageHue=%2334302b&clientId=u03bf352b-c00c-4&from=paste&height=85&id=u4d8a12bb&name=image.png&originHeight=170&originWidth=644&originalType=binary&ratio=2&rotation=0&showTitle=false&size=21343&status=done&style=none&taskId=uc3f6092f-747d-435a-8062-897df318b69&title=&width=322)

---

- 常量断言 `as const`，将类型尽量收窄到字面量类型，如果用在对象后面，相当于给对象里面每个成员加上 `readonly`并收窄

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678541879940-5535a4fb-827e-4d02-ac3d-eb997408e921.png#averageHue=%23ac6f31&clientId=ua28a33ae-7c55-4&from=paste&height=291&id=u1e0d3dda&name=image.png&originHeight=582&originWidth=778&originalType=binary&ratio=2&rotation=0&showTitle=false&size=40430&status=done&style=none&taskId=u30fc282d-9b30-46f5-a0f8-3a6faaa4d65&title=&width=389)

---


## 字面量类型

- 其实使用 JS 定义的值不仅可以做值，还可以当做 TS 的类型

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678541275712-a2f0a9ab-0bc8-4135-8208-bde0e3f11fd3.png#averageHue=%232e2e2e&clientId=ua28a33ae-7c55-4&from=paste&height=203&id=u88ec9c28&name=image.png&originHeight=406&originWidth=1346&originalType=binary&ratio=2&rotation=0&showTitle=false&size=66026&status=done&style=none&taskId=ubd88f558-15c1-46ca-87f3-9b09f4a8636&title=&width=673)

---


## 字面量推理

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678541815327-4516bbd3-f369-4a24-83cd-4a2224c72045.png#averageHue=%232f2e2e&clientId=ua28a33ae-7c55-4&from=paste&height=436&id=uc8abc9a8&name=image.png&originHeight=872&originWidth=1660&originalType=binary&ratio=2&rotation=0&showTitle=false&size=155575&status=done&style=none&taskId=u30a829a8-47b8-4825-a9c6-0ad1d8aacb4&title=&width=830)

---


## 类型收窄（Type Narrowing）

- 由一个更宽泛的类型变为更小的类型，缩小声明时的类型路径（Type Narrowing），比如 `number | string -> number`
- 而我们可以通过**类型保护（type guards）**来收窄类型
- 常见的类型保护有
  - `typeof`
  - `Switch` 或者一些相等运算符（`=== 、 !==`）来表达相等性
  - `instanceof`
  - `in`

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678543009365-6845b33f-84fe-4c52-8cda-74f83c806512.png#averageHue=%232f2d2c&clientId=ua28a33ae-7c55-4&from=paste&height=503&id=u90f08d40&name=image.png&originHeight=1272&originWidth=1266&originalType=binary&ratio=2&rotation=0&showTitle=false&size=122826&status=done&style=none&taskId=u9d88c07c-73e9-49e9-a361-e69b4cb5590&title=&width=501)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678589708361-5fe98bd1-b44a-49e3-a801-0907fa0cd4f4.png#averageHue=%232d2c2b&clientId=ua39188b9-9abc-4&from=paste&height=561&id=u1e8d5236&name=image.png&originHeight=1524&originWidth=1356&originalType=binary&ratio=2&rotation=0&showTitle=false&size=149019&status=done&style=none&taskId=uc72c8e3c-a328-496c-a2b9-2c5ffd5b640&title=&width=499)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678544725855-540fa4f9-f16f-4dff-9ef5-94957aa39a22.png#averageHue=%2334302c&clientId=ud83c2f35-ca1c-4&from=paste&height=205&id=u7f6b80da&name=image.png&originHeight=410&originWidth=818&originalType=binary&ratio=2&rotation=0&showTitle=false&size=50970&status=done&style=none&taskId=u82ab4c38-18ff-44c5-90fc-414755a2039&title=&width=409)

---



# 类型兼容

- TS 的类型兼容性是指当一个类型的值可以被另一个类型的变量所接受时，这两种类型就是兼容的
- 比如当一个类型 A 可以被赋值给另一个类型 B 时，我们就说类型 A 兼容于类型 B，，那么类型 A 就是类型 B 的子类型，类型 B 就是类型 A 的父类型

## 基本类型和普通对象

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678587680235-910d9c06-e01b-465f-a215-ec2b51fa727b.png#averageHue=%232d2d2c&clientId=u25cb17e1-f759-4&from=paste&height=675&id=u6b5d314d&name=image.png&originHeight=1632&originWidth=1516&originalType=binary&ratio=2&rotation=0&showTitle=false&size=184428&status=done&style=none&taskId=u46f4cd5e-b577-4b2e-9b3e-48bbfb5f6fc&title=&width=627)

---

## 接口兼容

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678585402968-a408ca35-2ff5-4133-acba-6ce00ab5ce09.png#averageHue=%232f2d2c&clientId=u4a5dbf1e-ddbd-4&from=paste&height=411&id=ufe50ac81&name=image.png&originHeight=822&originWidth=812&originalType=binary&ratio=2&rotation=0&showTitle=false&size=57198&status=done&style=none&taskId=ue065efc1-5f64-4583-af3d-58e1e641b3a&title=&width=406)

---

## 函数兼容

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678586171723-d7ae24bb-73e2-4658-9377-5c7c8a9786b5.png#averageHue=%232e2d2c&clientId=u4a5dbf1e-ddbd-4&from=paste&height=761&id=uc364f555&name=image.png&originHeight=1522&originWidth=1450&originalType=binary&ratio=2&rotation=0&showTitle=false&size=206968&status=done&style=none&taskId=ub1390c50-95fa-4571-88b2-843a6a01905&title=&width=725)

---

## 类的兼容

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678980450305-c9f7d80c-bc50-4b72-bc64-0bb8b1472ab2.png#averageHue=%23302e2c&clientId=u1049bf45-9e35-4&from=paste&height=416&id=u71f45294&name=image.png&originHeight=1048&originWidth=994&originalType=binary&ratio=2&rotation=0&showTitle=false&size=116513&status=done&style=none&taskId=ucba5485c-ad98-4fd5-a7a2-c02ecb55ef2&title=&width=395)

---



# 类

- ES6 之前使用函数实现类，ES6可以使用 class 关键字声明一个类
- 在默认的 `strictPropertyInitialization` 模式下面我们的属性是必须在`constructor` 初始化的，如果没有初始化，那么编译时就会报错，如希望此模式下不报错，可以使用 `name!: string` 的语法
- 类拥有自己的构造器 `constructor` ，当我们通过 `new` 关键字创建一个实例时会被调用，类中定义的函数叫方法

## 基本使用

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678594186316-92889b69-9ac4-49f3-a0f9-bdbaaa3a5a7e.png#averageHue=%232f2d2b&clientId=ue91f6808-6dfb-4&from=paste&height=365&id=ub79d69a0&name=image.png&originHeight=992&originWidth=1000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=105052&status=done&style=none&taskId=ua09f4fbb-b357-4a6e-b3ff-587a5534401&title=&width=368)

---

## 类的继承

- 使用 `extends` 关键字来实现继承，子类中使用 `super` 来访问父类

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678594248876-bb06afa7-0cde-4409-a87a-eae2db107582.png#averageHue=%232e2d2c&clientId=ue91f6808-6dfb-4&from=paste&height=515&id=u7fb11e17&name=image.png&originHeight=1338&originWidth=1362&originalType=binary&ratio=2&rotation=0&showTitle=false&size=184245&status=done&style=none&taskId=u1444a716-e83b-448d-ac48-a03d562c67e&title=&width=524)

---

## 类的成员修饰符

- 在 TS 中，类的属性和方法支持三种修饰符： `public`、`private`、`protected`
- **public**：类外可见，默认编写的属性就是 public 的
- **protected**：类和子类中可见
- **private**：仅自身类可见
- **#属性**：实现私有属性，并且类型擦除之后还有效

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678595302588-0cd05eb8-2953-4bbb-b7a8-606d4bf56ef3.png#averageHue=%23302e2c&clientId=u65f53549-0edf-4&from=paste&height=715&id=u5c6ccd9b&name=image.png&originHeight=1688&originWidth=1128&originalType=binary&ratio=2&rotation=0&showTitle=false&size=237371&status=done&style=none&taskId=ub425d703-c3dc-43b4-a91d-0e04ca07a75&title=&width=478)

---


## 只读属性

- 如果一个值不希望外界随意修改，可以使用 readonly 变得只读

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678595785661-3fb84d1c-3f53-4d09-b9d8-cf0728664d3b.png#averageHue=%23302e2d&clientId=u65f53549-0edf-4&from=paste&height=472&id=u35d5a16a&name=image.png&originHeight=944&originWidth=1580&originalType=binary&ratio=2&rotation=0&showTitle=false&size=185450&status=done&style=none&taskId=u4cf8f6dd-9ad4-4afe-a7b0-8c3763d72ad&title=&width=790)

---


## 访问器 getters/setters

- 之前有一些私有属性我们不能直接访问，我们就可以使用存取器监听他的获取（getter）和设置（setter）

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678596083617-c6ebdf67-b875-456b-848b-900da36a49a0.png#averageHue=%23302e2b&clientId=u65f53549-0edf-4&from=paste&height=480&id=u401106c2&name=image.png&originHeight=1056&originWidth=686&originalType=binary&ratio=2&rotation=0&showTitle=false&size=88354&status=done&style=none&taskId=u0bb564f4-16db-4ee1-8430-e53b6ad5127&title=&width=312)

---


## 静态成员

- 通过  static 可以定义类级别的成员和方法，通过 `类.属性或方法` 就可访问

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678596302814-4019adc4-9cd7-4580-be7d-83216bdea918.png#averageHue=%23302e2c&clientId=u65f53549-0edf-4&from=paste&height=294&id=u1605f352&name=image.png&originHeight=588&originWidth=780&originalType=binary&ratio=2&rotation=0&showTitle=false&size=68142&status=done&style=none&taskId=uf6d32c8c-eddd-4d67-a288-485185a7a15&title=&width=390)

---


## 抽象类

- 抽象方法，必须存在于抽象类（使用 `abstract` 声明）中
- 抽象类不能被 new 实例化，且内部抽象方法和属性必须被子类实现

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678603708965-b4204bb3-ac2e-4c3e-81d4-e65e54066f12.png#averageHue=%232e2c2b&clientId=u6dc6c8c9-f2db-4&from=paste&height=852&id=u01595193&name=image.png&originHeight=1704&originWidth=1918&originalType=binary&ratio=2&rotation=0&showTitle=false&size=228403&status=done&style=none&taskId=uc387ce00-2678-4189-bd34-a847299b68e&title=&width=959)

---


## 类的类型

- 类本身也可以当做一种类型

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678601338988-212d1360-d3ee-4c59-9b61-770e0487ee0b.png#averageHue=%23302e2d&clientId=ua021c256-93ae-4&from=paste&height=477&id=u8f552322&name=image.png&originHeight=1118&originWidth=748&originalType=binary&ratio=2&rotation=0&showTitle=false&size=108361&status=done&style=none&taskId=ubd018250-cd1e-4443-8019-e1ab0e1cfb9&title=&width=319)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678605410078-92816334-857f-4d93-a8f7-8d41d7e44298.png#averageHue=%23312e2c&clientId=u61fe6553-4325-4&from=paste&height=481&id=u4bba1830&name=image.png&originHeight=1106&originWidth=978&originalType=binary&ratio=2&rotation=0&showTitle=false&size=137610&status=done&style=none&taskId=u969ac6c6-118d-480a-915c-b5dc7724de8&title=&width=425)

---


## 参数属性（Parameter Properties）

- 参数属性将 `constructor` 参数转换为同名同值的实例属性，相当于帮我们做 `this.name = name` 的操作
- 我们可以在其 `constructor` 参数前面添加可见修饰符（public private protected 或者 readonly）创建参数属性

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1679212158032-78af21bd-d4b5-4462-a22e-1d87350cee56.png#averageHue=%232d2c2b&clientId=u531fb2b5-e32f-4&from=paste&height=436&id=u0577e92c&name=image.png&originHeight=1042&originWidth=1652&originalType=binary&ratio=2&rotation=0&showTitle=false&size=139529&status=done&style=none&taskId=uf0c93c1b-8c81-4c57-b505-efdf52c60e3&title=&width=691)

---

# 泛型编程

函数的本质个人认为是

- 推后部分待定的代码

想象一下 JS 没有函数会怎么样，其实 TS 的泛型就类似于 JS 函数，不过它是**推后执行部分待定的类型**

## 泛型实现类型参数化

- 定义函数的时候不决定参数的类型
- 而是让调用者使用尖括号形式传入对应函数

比如我们实现一个函数，传入一个参数并返回它，保证这个参数和返回值类型一致
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678284086790-ed25bfdd-3fb9-45b9-a7ad-9b1a3a3e1cc4.png#averageHue=%23302e2b&clientId=uc236713c-6c28-4&from=paste&height=88&id=u5f5bdafe&name=image.png&originHeight=176&originWidth=942&originalType=binary&ratio=2&rotation=0&showTitle=false&size=19754&status=done&style=none&taskId=u5c900e8f-9fd7-43c7-b836-13be61fc986&title=&width=471)
此函数我们使用的话

1. 通过** <类型> **的方式将类型传递给函数；
2. 通过类型推导，自动推导出传入参数的类型

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678284250493-6a317779-67a7-4df2-8008-0ac1d312b3b3.png#averageHue=%23977f42&clientId=uc236713c-6c28-4&from=paste&height=124&id=u22e72191&name=image.png&originHeight=248&originWidth=1120&originalType=binary&ratio=2&rotation=0&showTitle=false&size=24366&status=done&style=none&taskId=u110aeebc-08e7-4cc5-9d9f-cf45a91d16a&title=&width=560)

当然我们可以传入多个类型
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678284307353-c1676153-bf61-4ed1-9f1a-25c9f01a832f.png#averageHue=%2337322b&clientId=uc236713c-6c28-4&from=paste&height=37&id=udb1dacb0&name=image.png&originHeight=74&originWidth=840&originalType=binary&ratio=2&rotation=0&showTitle=false&size=9762&status=done&style=none&taskId=ue9f8eff9-b82a-4029-a228-1a24d3e403b&title=&width=420)
其中，T，E 这些都是我们可以自定义的，它们代表的意义是

- T（Type）：类型
- K（key）、V（value）：，键值对
- E（Element）：元素
- O（Object）：对象

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678810373158-e8e23f6c-10d1-4f49-9a65-9620bf88bc64.png#averageHue=%23302f2d&clientId=u0a4f6df7-a934-4&from=paste&height=292&id=u885feeaf&name=image.png&originHeight=584&originWidth=1168&originalType=binary&ratio=2&rotation=0&showTitle=false&size=91881&status=done&style=none&taskId=u65cd3150-40e7-45bd-bb99-cc3296f81f2&title=&width=584)

---

## 泛型接口

泛型接口是一种具有泛型类型参数的接口，它可以在接口的定义中使用这些类型参数，从而使得接口的属性和方法能够适用于多种类型

- 定义接口的时候使用泛型

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678284572032-e7096dc8-7879-453e-8019-5057c48ae646.png#averageHue=%232f2d2b&clientId=uc236713c-6c28-4&from=paste&height=337&id=u6859a1d3&name=image.png&originHeight=674&originWidth=704&originalType=binary&ratio=2&rotation=0&showTitle=false&size=57221&status=done&style=none&taskId=u346d3e26-721b-4b6d-9c03-d5f5268c7aa&title=&width=352)

- 指定类型默认值

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678284587756-e331ea65-2615-4792-9289-3d2e4fced8a4.png#averageHue=%23302e2b&clientId=uc236713c-6c28-4&from=paste&height=157&id=uc32d1987&name=image.png&originHeight=314&originWidth=780&originalType=binary&ratio=2&rotation=0&showTitle=false&size=28302&status=done&style=none&taskId=u3a2c30ba-1961-41fd-8687-9c0fe80b5ad&title=&width=390)


## 泛型类

- 泛型类是一种具有泛型类型参数的类，它可以在类的定义中使用这些类型参数，从而使得类的属性和方法能够适用于多种类型

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678284800785-9480b0af-e251-42d9-abeb-a4609bcf383f.png#averageHue=%232f2d2b&clientId=uc236713c-6c28-4&from=paste&height=347&id=u3ecbd171&name=image.png&originHeight=784&originWidth=1058&originalType=binary&ratio=2&rotation=0&showTitle=false&size=73243&status=done&style=none&taskId=udd546b4c-2cbe-4d28-bbcd-76f063b38d6&title=&width=468)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678602530862-f6d790e4-c063-498e-a9bc-a447bdf25bd0.png#averageHue=%232f2d2c&clientId=ua358c274-b322-4&from=paste&height=369&id=u700349b0&name=image.png&originHeight=944&originWidth=1260&originalType=binary&ratio=2&rotation=0&showTitle=false&size=122687&status=done&style=none&taskId=u3d6b71d5-0971-445c-95b7-b18b6d8f76d&title=&width=492)


## 泛型约束（Generic Constraints）

### 泛型中使用 extends

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678810517536-b6000360-dd9b-4622-9dd7-dc73d9490d2c.png#averageHue=%23302f2d&clientId=u0a4f6df7-a934-4&from=paste&height=457&id=u4025ec58&name=image.png&originHeight=1288&originWidth=1296&originalType=binary&ratio=2&rotation=0&showTitle=false&size=211008&status=done&style=none&taskId=ufcdeaa2a-8b16-4838-868e-cb40302ce90&title=&width=460)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678892266917-03949e89-da83-4351-9063-fd73cdf6e878.png#averageHue=%2335312d&clientId=ua9d9d28d-2486-4&from=paste&height=116&id=u6dba92d5&name=image.png&originHeight=232&originWidth=1550&originalType=binary&ratio=2&rotation=0&showTitle=false&size=79336&status=done&style=none&taskId=uf47dcee9-bbc6-461f-87ed-f980af44c7d&title=&width=775)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678809100646-0a55df1d-0370-48a9-8231-3ebe912800fe.png#averageHue=%23c5c5c5&clientId=uf3ab9fb8-dab7-4&from=paste&height=354&id=ucb6cf2fb&name=image.png&originHeight=708&originWidth=590&originalType=binary&ratio=2&rotation=0&showTitle=false&size=56647&status=done&style=none&taskId=ubf4cf234-f1cf-4cfa-8637-d3e48f66f51&title=&width=295)

---

### 泛型中使用 keyof

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678810618581-c4349686-b1fa-430d-8a35-76539d679400.png#averageHue=%232f2d2c&clientId=u0a4f6df7-a934-4&from=paste&height=350&id=u3e1bc103&name=image.png&originHeight=700&originWidth=1092&originalType=binary&ratio=2&rotation=0&showTitle=false&size=86629&status=done&style=none&taskId=u694e241b-b34e-4426-8319-97ff1ad82db&title=&width=546)

---


### 泛型中使用 extends 和 keyof

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678811477031-4380c5a6-8208-43fb-89d9-dcfdfb0f624a.png#averageHue=%232f2e2d&clientId=u569e8349-029e-4&from=paste&height=446&id=u8daf2888&name=image.png&originHeight=1054&originWidth=1512&originalType=binary&ratio=2&rotation=0&showTitle=false&size=177239&status=done&style=none&taskId=u35b5c826-b2d1-4b41-bda0-a24d72f0ca1&title=&width=640)

---


## 映射类型（Mapped Types）

- 映射类型是 TS 中的一种高级类型，它可以用来从一个现有类型中生成一个新的类型
- TS 大部分的内置工具和类型体操都是基于映射类型实现
- 映射类型的语法形式是 `{ [K in keyof T]: U }`
- 其中 `K` 是 `T` 的所有属性名的联合类型，`keyof` 是一个索引类型查询操作符，用来获取一个类型的所有属性名的联合类型。
- `U` 是一个类型变换函数，它用来将 `T` 中的每个属性类型变成另一个类型

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678892877524-edd8b88f-b35b-4bb4-a458-17b2549de60a.png#averageHue=%23302e2b&clientId=ua9d9d28d-2486-4&from=paste&height=399&id=ud60c2afa&name=image.png&originHeight=1116&originWidth=1072&originalType=binary&ratio=2&rotation=0&showTitle=false&size=122433&status=done&style=none&taskId=u5a4e04e8-ddc8-43f9-a13a-cb02e3a55e7&title=&width=383)

在使用映射类型时，有两个额外的修饰符

- `readonly`，用于设置属性只读
- `? `，用于设置属性可选

`-?` 去掉可选，如果变成 `+?` 则都变可选  `-readonly` 代表去除 `readonly`
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678724482648-1a50007b-9b6a-4c3a-bad7-88cccac7916c.png#averageHue=%23b3702e&clientId=ue21edfb6-25d2-4&from=paste&height=355&id=u325bb2eb&name=image.png&originHeight=762&originWidth=808&originalType=binary&ratio=2&rotation=0&showTitle=false&size=72656&status=done&style=none&taskId=u5ac6b58d-6ab7-4173-b7d8-097550a0395&title=&width=376)

更高级的一些用法
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678896979540-c67bf934-ab96-469b-bf5d-70ad8b5465fe.png#averageHue=%232d2c2b&clientId=uc9e23b2f-ae12-4&from=paste&height=405&id=u4fd39432&name=image.png&originHeight=1456&originWidth=1934&originalType=binary&ratio=2&rotation=0&showTitle=false&size=288128&status=done&style=none&taskId=u48051665-51f1-471e-9182-c794a779f46&title=&width=538)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678897439896-b277f01f-555d-4d8f-96b7-d9313abb1b90.png#averageHue=%232e2d2b&clientId=u81f34c54-61da-4&from=paste&height=461&id=ue57de94e&name=image.png&originHeight=1160&originWidth=1360&originalType=binary&ratio=2&rotation=0&showTitle=false&size=132082&status=done&style=none&taskId=u25d6ef19-82dc-4308-8333-a7652a027c3&title=&width=540)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678898284884-7f27a0d5-3ee3-44c9-b8a3-3e08d5789ea6.png#averageHue=%232f2d2d&clientId=u25d51ac1-2f49-4&from=paste&height=495&id=u9af187dd&name=image.png&originHeight=1340&originWidth=734&originalType=binary&ratio=2&rotation=0&showTitle=false&size=111159&status=done&style=none&taskId=u0c1b0b55-12be-440b-b610-48ebcfa5db1&title=&width=271)

---


## 内置工具和类型体操

- TS 的类型系统增加了很多功能以适配 JS 的灵活性，导致 TS 是一门**支持类型编程的类型系统**
- 通常我们为代码加上类型约束，不太需要过多类型编程的能力
- 但是在开发一些通用框架，库的时候，考虑各种适配就需要更多考虑类型编程


### 条件类型（Conditional Types）

- 条件类型可以根据某个特定的条件，从两个类型中选择一个作为最终类型
- 写法类似于 JS 三元 ：`SomeType extends OtherType ? TrueType : FalseType`

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678894830411-518ac6b6-a0bd-48ae-b3f5-4e7ae8ce807b.png#averageHue=%232d2c2b&clientId=u310971a4-65eb-4&from=paste&height=533&id=uc16b7d75&name=image.png&originHeight=1066&originWidth=2242&originalType=binary&ratio=2&rotation=0&showTitle=false&size=232353&status=done&style=none&taskId=uf1414ba0-4526-44b7-b50e-f0a88841c15&title=&width=1121)

---


### 条件类型中推断（infer）和 ReturnType<Type>

- 条件类型提供了 `infer` 关键词，可以从正在比较的类型中推断类型，然后在 true 分支里引用该推断结果
- 比如目前有一个数组类型，想要获取函数参数和返回值类型

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678287414689-5355a421-e35d-4a21-b2ab-b5b0d39dee0e.png#averageHue=%232f2d2b&clientId=uc236713c-6c28-4&from=paste&height=180&id=ud194c294&name=image.png&originHeight=360&originWidth=2404&originalType=binary&ratio=2&rotation=0&showTitle=false&size=69808&status=done&style=none&taskId=u606d7822-2757-4c20-aeba-937730d4fcd&title=&width=1202)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678373027993-ee2dd84c-d9aa-4dcc-8fd4-df302f5841f3.png#averageHue=%23332f2b&clientId=u3fdef9f6-3b73-4&from=paste&height=156&id=ucf13e8a5&name=image.png&originHeight=312&originWidth=798&originalType=binary&ratio=2&rotation=0&showTitle=false&size=28326&status=done&style=none&taskId=u7783118b-1e72-4c59-bdf4-fd4ce0335ac&title=&width=399)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678287472042-5c066eda-c38b-4f2c-ba4f-02453da4b3e2.png#averageHue=%23312e2b&clientId=uc236713c-6c28-4&from=paste&height=119&id=u7cae2d31&name=image.png&originHeight=238&originWidth=2352&originalType=binary&ratio=2&rotation=0&showTitle=false&size=58429&status=done&style=none&taskId=u4837db91-138c-46c6-877c-b079dec465b&title=&width=1176)


### 分发条件类型（Distributive Conditional Types）

- 泛型中使用条件类型，如果传入联合类型，就会变成 分发的（`distributive`）

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678287806973-f5bda5b4-39ca-4ea7-90bb-568f7ac92a1d.png#averageHue=%2335312e&clientId=ud4431150-957f-4&from=paste&height=138&id=uabaa91e5&name=image.png&originHeight=312&originWidth=1648&originalType=binary&ratio=2&rotation=0&showTitle=false&size=74129&status=done&style=none&taskId=udea2dbae-d314-4eac-9676-da9205f2d96&title=&width=731)
如果我们希望是 `(string | number)[]` 这种类型，给 T 加个方括号就行
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678723779573-ba795f9f-63b5-438c-ac60-a69e33fe76b1.png#averageHue=%23b3712e&clientId=u8a416de1-428d-4&from=paste&height=117&id=u359a87cb&name=image.png&originHeight=234&originWidth=1100&originalType=binary&ratio=2&rotation=0&showTitle=false&size=35509&status=done&style=none&taskId=u3c9f4eae-5eab-458c-8d36-4d7ae665c56&title=&width=550)
注意：若传入 `never` ，则返回的类型始终为 `never`

---

### Partial\<Type>

- 所有属性变为可选的类型

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678287940354-bec69fce-e6b1-40b0-ac45-9e2575f7f80c.png#averageHue=%23b7722e&clientId=ud4431150-957f-4&from=paste&height=350&id=u0b422b6d&name=image.png&originHeight=798&originWidth=908&originalType=binary&ratio=2&rotation=0&showTitle=false&size=70722&status=done&style=none&taskId=ub46827bc-53f2-4953-99f3-a6fcf778433&title=&width=398)

---


### Required\<Type>

- 所有属性变为必填的类型

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678288050144-780987e5-a697-4128-b9bc-d3715296bc06.png#averageHue=%23b7732e&clientId=ud4431150-957f-4&from=paste&height=391&id=u76ef31db&name=image.png&originHeight=782&originWidth=720&originalType=binary&ratio=2&rotation=0&showTitle=false&size=64889&status=done&style=none&taskId=u8df9ff2a-d740-4ff8-b10b-6ead1b59032&title=&width=360)

---


### Readonly\<Type>

- 所有属性变为只读的类型

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678288131914-eb233a5c-bc89-490d-8983-4e456c5cd521.png#averageHue=%23b8722f&clientId=ud4431150-957f-4&from=paste&height=395&id=u1f2e7635&name=image.png&originHeight=790&originWidth=1104&originalType=binary&ratio=2&rotation=0&showTitle=false&size=80645&status=done&style=none&taskId=u36399f37-f771-4e6e-84ef-d789be8aec3&title=&width=552)

---


### Record<Keys, Type>

- 构造一个对象类型，所有key(键)都是 keys 类型， 所有 value(值)都是 Type 类型

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678288327952-faf32ef1-1d56-4e04-910d-9b958dc09335.png#averageHue=%232f2d2b&clientId=ud4431150-957f-4&from=paste&height=449&id=uc386775a&name=image.png&originHeight=898&originWidth=1016&originalType=binary&ratio=2&rotation=0&showTitle=false&size=88725&status=done&style=none&taskId=u02e0bd23-f722-4b6d-81aa-e5ac73d78e3&title=&width=508)

---


### Pick<Type, Keys>

- 构造一个类型，从 Type 类型里面挑选一些类型 Keys

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678288473139-36b3cc1c-e341-4bdc-be06-3efca257f78b.png#averageHue=%232f2d2b&clientId=ud4431150-957f-4&from=paste&height=420&id=ud421003c&name=image.png&originHeight=840&originWidth=1080&originalType=binary&ratio=2&rotation=0&showTitle=false&size=79109&status=done&style=none&taskId=uec96dbd0-8ebb-4adc-aec4-ac7d5378893&title=&width=540)

---


### Omit<Type, Keys>

- 构造一个类型，从 Type 类型里面过滤掉一些类型 Keys

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678288587336-28b0ef48-b3e9-4017-bf88-f3078dbde287.png#averageHue=%232f2d2b&clientId=ud4431150-957f-4&from=paste&height=423&id=u73731d77&name=image.png&originHeight=846&originWidth=1204&originalType=binary&ratio=2&rotation=0&showTitle=false&size=82317&status=done&style=none&taskId=u4decbaca-6f5f-4827-8404-6d17f514f32&title=&width=602)

---


### Exclude<UnionType, ExcludeMembers>

- 构造一个类型，它是从 UnionType 联合类型里面排除了所有可以赋给 ExcludedMembers 的类型
- 可以使用它帮助实现 Omit

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678290088130-275fb68c-267a-412f-8fd4-b7e0416f9484.png#averageHue=%232f2d2b&clientId=ua727c8bb-21fb-4&from=paste&height=423&id=ue4a8e5b2&name=image.png&originHeight=846&originWidth=1294&originalType=binary&ratio=2&rotation=0&showTitle=false&size=103709&status=done&style=none&taskId=u69092957-e5df-4b9f-8975-e0271ad6a6e&title=&width=647)

---


### Extract<Type, Union>

- 构造一个类型，从 Type 类型里面提取了所有可以赋给 Union 的类型

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678290187696-3ebb3491-5b57-4c80-bdd2-a0a760076d13.png#averageHue=%23302e2b&clientId=ua727c8bb-21fb-4&from=paste&height=125&id=u959c2af4&name=image.png&originHeight=250&originWidth=1260&originalType=binary&ratio=2&rotation=0&showTitle=false&size=37016&status=done&style=none&taskId=u16008e53-824a-49dd-9a5c-90ae5d66194&title=&width=630)

---


### NonNullable\<Type>

- 构造一个类型，这个类型从 Type 中排除了所有的 null、undefined 的类型

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678290309308-1376754f-9135-4b50-838c-873f178250f2.png#averageHue=%23312e2b&clientId=ua727c8bb-21fb-4&from=paste&height=293&id=u7c6445bc&name=image.png&originHeight=586&originWidth=1448&originalType=binary&ratio=2&rotation=0&showTitle=false&size=71110&status=done&style=none&taskId=u3a0a2593-abeb-4f78-88e8-e6893853fac&title=&width=724)

---


### InstanceType\<Type>

- 构造一个由所有 Type 的构造函数的实例类型组成的类型

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678293730062-61fcb878-fe76-40e8-8b1c-5d6f0ed974c0.png#averageHue=%232e2d2c&clientId=ue1f4f9e0-3e08-4&from=paste&height=550&id=uc917ed7a&name=image.png&originHeight=1100&originWidth=2358&originalType=binary&ratio=2&rotation=0&showTitle=false&size=203882&status=done&style=none&taskId=ud7b68925-fcf7-4375-a8ca-698f23dba56&title=&width=1179)

下面通过泛型结合工厂函数，更灵活获取实例的类型
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678293818815-45899513-f345-4b2c-8045-a45343a01350.png#averageHue=%232f2e2c&clientId=ue1f4f9e0-3e08-4&from=paste&height=321&id=ue1a94e11&name=image.png&originHeight=642&originWidth=1998&originalType=binary&ratio=2&rotation=0&showTitle=false&size=116541&status=done&style=none&taskId=u07fa9e02-e01f-467b-b738-344aa54cd44&title=&width=999)

---



# 扩展知识

## TS 模块化和 .d.ts

- TS 支持两种方式来控制我们的作用域
- 模块化：每个文件可以是一个独立的模块，支持 `ESModule`，也支持 `CommonJS`

如果文件内没有任何 `export` 或 `import` ，而你又希望将它作为模块使用，即使它没有导出任何内容，添加下面这行代码
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678622128511-fa0fe1de-5351-4979-8a55-b276b85ae858.png#averageHue=%2339322b&clientId=u49d4a21c-c2af-4&from=paste&height=39&id=u6093d167&name=image.png&originHeight=78&originWidth=242&originalType=binary&ratio=2&rotation=0&showTitle=false&size=4204&status=done&style=none&taskId=uf47a96b6-930b-4b7d-bf76-f030296b528&title=&width=121)

- 内置类型导入（Inline type imports），使用 type 前缀 ，下面两种方式都表示导入一个类型

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678890366363-958166f7-b6fd-46ff-b2ba-f1aa40d16a5e.png#averageHue=%23332f2b&clientId=u9e0f34e2-811a-4&from=paste&height=29&id=u903c4c8b&name=image.png&originHeight=58&originWidth=1062&originalType=binary&ratio=2&rotation=0&showTitle=false&size=14816&status=done&style=none&taskId=ub73b3fab-e6df-4928-befb-3c1caa01f4c&title=&width=531)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678890397479-58a6c5db-0f37-46d3-81e6-8d06a47af2b6.png#averageHue=%2335302b&clientId=u9e0f34e2-811a-4&from=paste&height=28&id=u7d124f3d&name=image.png&originHeight=56&originWidth=1160&originalType=binary&ratio=2&rotation=0&showTitle=false&size=15671&status=done&style=none&taskId=u35468b62-cb9c-46b9-879c-92bd958beba&title=&width=580)

---

- 命名空间：通过 `namespace` 来声明一个命名空间，主要是在早期将模块内部，再进行作用域的划分，防止命名冲突

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678622327227-bd7fa004-f80b-44aa-9aa1-501f2dd57fce.png#averageHue=%23332f2b&clientId=u49d4a21c-c2af-4&from=paste&height=162&id=u62e0cbce&name=image.png&originHeight=354&originWidth=944&originalType=binary&ratio=2&rotation=0&showTitle=false&size=41211&status=done&style=none&taskId=uecdb3a21-9776-4a83-a630-556c7a32fc6&title=&width=432)

---


## 类型声明和查找

- 之前我们写的 `const imageEl = document.getElementById ("image") as HTMLImageElement`，这个 HTMLImageElement 类型来自于哪里呢？
- 这就涉及到对类型的管理和查找规则了，除了我们写的 .ts 的代码文件，其实还有个 .d.ts 文件，它是用来做**类型声明（declare）**或者 **类型定义（Type Definition）**文件
- 当我们写一个类型时候，会在**内置类型声明、外部定义类型声明、自己定义类型声明**里查找

### 内置类型声明

- TS 帮我们内置了一些运行时标准化 API 的声明文件
- 比如 `Function`、`String`、`Math`、`Date`、`RegExp`、`Error` 等内置类型
- 也包含运行环境中的 DOM 、BOM API，比如 `Window`、`Document`、`HTMLElement`、`Event`、`NodeList`等
- 很多常用方法其实 TS  已经帮你声明好了，具体地址：

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678807514137-653881b3-4c0c-49e9-81e0-6367cd5bb715.png#averageHue=%2338332d&clientId=u1c56c86a-1a3f-4&from=paste&height=177&id=u1d3e2e54&name=image.png&originHeight=354&originWidth=860&originalType=binary&ratio=2&rotation=0&showTitle=false&size=69607&status=done&style=none&taskId=ue2ba786e-e8aa-4812-b122-d5edc8d2c54&title=&width=430)

---


### 内置声明的环境

- 我们可以通过 `target` 和 `lib` 决定哪些内置类型声明可以使用
- 例如，`startsWith` 字符串方法 ECMAScript 6 版本才开始使用，我们就可以修改选项以获取新 API 的类型
- 


### 外部定义类型声明（第三方库）

- 第三方库使用的时候也需要类型声明
- 方式一：库自带 d.ts 的类型声明文件，比如 axios
- 方式二：通过社区公有库 `DefinitelyTyped` 存放类型声明文件，比如我们安装 react 类型声明 `npm i @types/react --save-dev`
  - GitHub地址：


### 外部定义类型声明（自定义声明）

- 情况一：纯 JS 第三方库，比如 lodash，如果也没有类型声明库安装，我们就需要手动为其添加类型声明
- 情况二：自己项目声明一些公共的类型，方便复用


### 声明文件 d.ts

- .d.ts 文件是声明文件（Declaration File），用于描述 JavaScript 模块、类、函数、变量等的类型信息
- 如果需要为第三方库或者自己库编写全局通用的声明，就可以创建 .d.ts

```typescript
declare var 声明全局变量
declare const 声明全局常量
declare function 声明全局方法
declare class 声明全局类
declare enum 声明全局枚举类型
declare namespace 声明（含有子属性的）全局对象
interface 和 type 声明全局类型
```

`xxx.d.ts`
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678721317955-3f528499-a2f1-489b-8cfe-15f69ae04574.png#averageHue=%23312e2c&clientId=u0020c49b-0494-4&from=paste&height=594&id=ucbe111f6&name=image.png&originHeight=1684&originWidth=1270&originalType=binary&ratio=2&rotation=0&showTitle=false&size=227370&status=done&style=none&taskId=u626b25a7-86e1-4d6a-a0a1-1f4783683b6&title=&width=448)
使用的地方就可以放心使用，因为 .d.ts 结尾的文件声明都是全局
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21596389/1678721378078-96e6499f-fa99-4a12-bb88-d99bd2e6748c.png#averageHue=%232d2c2b&clientId=u0020c49b-0494-4&from=paste&height=413&id=u947e56e4&name=image.png&originHeight=982&originWidth=1158&originalType=binary&ratio=2&rotation=0&showTitle=false&size=96313&status=done&style=none&taskId=uf2d009e9-b1ad-45de-9110-ad3707d44b1&title=&width=487)

---



## tsconfig.json

- 通过 `tsc --init` 命令可以生成 `tsconfig.json`
- 它是 TS 的配置文件，用于配置 TS 编译器的行为

```json
"compilerOptions": {
  "incremental": true, // TS编译器在第一次编译之后会生成一个存储编译信息的文件，第二次编译会在第一次的基础上进行增量编译，可以提高编译的速度
  "tsBuildInfoFile": "./buildFile", // 增量编译文件的存储位置
  "diagnostics": true, // 打印诊断信息 
  "target": "ES5", // 编译后的 JavaScript 代码的目标版本。例如："es5"、"es6" 等
  "module": "CommonJS", // 编译后的 JavaScript 代码的模块化方案。例如："commonjs"、"es6" 等
  "outFile": "./app.js", // 将多个相互依赖的文件生成一个文件，可以用在AMD模块中，即开启时应设置"module": "AMD",
  "lib": ["DOM", "ES2015", "ScriptHost", "ES2019.Array"], // 编译器需要引入的库文件。例如："es5"、"es6"、"dom" 等
  "allowJS": true, // 允许编译器编译JS，JSX文件
  "checkJs": true, // 允许在JS文件中报错，通常与allowJS一起使用
  "outDir": "./dist", // 编译后的 JavaScript 文件输出目录
  "rootDir": "./", // 指定输出文件目录(用于输出)，用于控制输出目录结构
  "declaration": true, // 是否生成声明文件
  "declarationDir": "./file", // 声明文件的输出目录
  "emitDeclarationOnly": true, // 只生成声明文件，而不会生成js文件
  "sourceMap": true, // 是否生成源代码与编译后代码的映射文件
  "inlineSourceMap": true, // 生成目标文件的inline SourceMap，inline SourceMap会包含在生成的js文件中
  "declarationMap": true, // 为声明文件生成sourceMap
  "typeRoots": [], // 声明文件目录，默认时node_modules/@types
  "types": [], // 加载的声明文件包
  "removeComments":true, // 删除注释 
  "noEmit": true, // 不输出文件,即编译后不会生成任何js文件
  "noEmitOnError": true, // 发送错误时不输出任何文件
  "noEmitHelpers": true, // 不生成helper函数，减小体积，需要额外安装，常配合importHelpers一起使用
  "importHelpers": true, // 通过tslib引入helper函数，文件必须是模块
  "downlevelIteration": true, // 降级遍历器实现，如果目标源是es3/5，那么遍历器会有降级的实现
  "strict": true, // 是否启用严格模式
  "alwaysStrict": true, // 在代码中注入'use strict'
  "noImplicitAny": true, // 不允许隐式的any类型
  "strictNullChecks": true, // 不允许把null、undefined赋值给其他类型的变量
  "strictFunctionTypes": true, // 不允许函数参数双向协变
  "strictPropertyInitialization": true, // 类的实例属性必须初始化
  "strictBindCallApply": true, // 严格的bind/call/apply检查
  "noImplicitThis": true, // 不允许 this 有隐式的 any 类型
  "noImplicitAny": true, 是否禁止隐式的 any 类型。
  "noUnusedLocals": true, // 检查只声明、未使用的局部变量(只提示不报错)
  "noUnusedParameters": true, // 检查未使用的函数参数(只提示不报错)
  "noFallthroughCasesInSwitch": true, // 防止switch语句贯穿(即如果没有break语句后面不会执行)
  "noImplicitReturns": true, //每个分支都会有返回值
  "esModuleInterop": true, // 允许 esmoudle 和 commonjs 相互调用
  "allowUmdGlobalAccess": true, // 允许在模块中全局变量的方式访问umd模块
  "moduleResolution": "node", // 模块解析策略，ts默认用node的解析策略，即相对的方式导入
  "baseUrl": "./", // 解析非相对模块的基地址，默认是当前目录
  "paths": { // 路径映射，相对于baseUrl
    // 如使用jq时不想使用默认版本，而需要手动指定版本，可进行如下配置
    "jquery": ["node_modules/jquery/dist/jquery.min.js"]
  },
  "rootDirs": ["src","out"], // 将多个目录放在一个虚拟目录下，用于运行时，即编译后引入文件的位置可能发生变化，这也设置可以虚拟src和out在同一个目录下，不用再去改变路径也不会报错
  "listEmittedFiles": true, // 打印输出文件
  "listFiles": true// 打印编译的文件(包括引用的声明文件)
}
 
//  指定需要编译的文件或目录。可以使用通配符 * 匹配多个文件或目录（属于自动指定该路径下的所有ts相关文件）
"include": [
   "src/**/*"
],
//  指定不需要编译的文件或目录。可以使用通配符 * 匹配多个文件或目录（include的反向操作）
 "exclude": [
   "demo.ts"
],
// 指定需要编译的文件列表（属于手动一个个指定文件）
 "files": [
   "demo.ts"
]
```

选项过多，讲几个常用点的：

- target: 编译后的 JavaScript 代码的目标版本。例如："es5"、"es6" 等
- module: 编译后的 JavaScript 代码的模块化方案。例如："commonjs"、"es6" 等
- lib: 编译器需要引入的库文件。例如："es5"、"es6"、"dom" 等
- allowJs:  允许编译器编译 JS，JSX 文件
- strict: 启用所有严格类型检查选项。
- esModuleInterop: 允许 esmoudle 和 commonjs 相互调用
- include: 要编译的文件路径，可以是文件或文件夹的相对路径或绝对路径。
- exclude: 不需要编译的文件路径，可以是文件或文件夹的相对路径或绝对路径
- extends: 继承其他的 tsconfig.json 文件
- skipLibCheck：跳过对引入的库文件的类型检查
- sourceMap：是否生成源代码与编译后代码的映射文件
- "removeComments"：编译文件后删除所有注释

---



# 实战



## 封装 axios

- 支持请求和响应拦截器
- 支持取消请求和取消全部请求的功能
- 提供了 GET、POST、PUT、DELETE 四种请求方法

```typescript
import axios from 'axios'
import type { AxiosInstance, AxiosResponse, InternalAxiosRequestConfig, AxiosRequestConfig } from 'axios'

class Request {
	// Axios 实例
	instance: AxiosInstance
	// 存放取消请求控制器
	abortControllerMap: Map<string, AbortController>

	constructor(config: AxiosRequestConfig) {
		// 创建 Axios 实例
		this.instance = axios.create(config)
		// 存储取消请求的控制器
		this.abortControllerMap = new Map()
		// 添加请求拦截器
		this.instance.interceptors.request.use(
			(res: InternalAxiosRequestConfig) => {
				// 创建取消请求的控制器
				const controller = new AbortController()
				// 获取请求的 url
				const url = res.url || ''
				// 将控制器存储到 Map 中
				res.signal = controller.signal
				this.abortControllerMap.set(url, controller)
				return res
			},
			(err: any) => err
		)

		// 添加响应拦截器
		this.instance.interceptors.response.use(
			(res: AxiosResponse) => {
				// 获取响应的 url
				const url = res.config.url || ''
				// 从 Map 中删除对应的控制器
				this.abortControllerMap.delete(url)
				return res.data
			},
			(err: any) => err // 响应拦截器错误处理函数
		)
	}

	// 发送请求的方法，返回 Promise 对象
	request<T>(config: AxiosRequestConfig<T>): Promise<T> {
		return new Promise((resolve, reject) => {
			this.instance
				.request(config)
				.then(res => {
					resolve(res as T)
				})
				.catch((err: any) => {
					reject(err)
				})
		})
	}

	// 取消全部请求
	cancelAllRequest() {
		for (const controller of this.abortControllerMap.values()) {
			controller.abort()
		}
		// 清空 Map
		this.abortControllerMap.clear()
	}

	// 取消指定的请求
	cancelRequest(url: string | string[]) {
		// 将参数转换为数组
		const urlList = Array.isArray(url) ? url : [url]
		urlList.forEach(_url => {
			// 根据 url 获取对应的控制器并取消请求
			this.abortControllerMap.get(_url)?.abort()
			// 从 Map 中删除对应的控制器
			this.abortControllerMap.delete(_url)
		})
	}

	// 发送 GET 请求的方法，返回 Promise 对象
	async get<T = any>(url: string, config?: AxiosRequestConfig): Promise<T> {
		return this.request<T>({ ...config, method: 'get', url })
	}

	// 发送 POST 请求的方法，返回 Promise 对象
	async post<T = any>(url: string, data?: any, config?: AxiosRequestConfig): Promise<T> {
		return this.request<T>({ ...config, method: 'post', url, data })
	}

	// 发送 PUT 请求的方法，返回 Promise 对象
	async put<T = any>(url: string, data?: any, config?: AxiosRequestConfig): Promise<T> {
		return this.request<T>({ ...config, method: 'put', url, data })
	}
	// 发送 DELETE 请求的方法，返回 Promise 对象
	async delete<T = any>(url: string, config?: AxiosRequestConfig): Promise<T> {
		return this.request<T>({ ...config, method: 'delete', url })
	}
}

const myRequest = new Request({
	baseURL: 'https://www.fastmock.site/mock/13089f924ad68903046c5a61371475c4',
	timeout: 10000,
})

export default myRequest
```

使用：

```vue
<script setup lang="ts">
import Axios from "./axios"
import { onMounted } from "vue"

const myRequest = new Axios({
  baseURL: "https://www.fastmock.site/mock/13089f924ad68903046c5a61371475c4",
  timeout: 10000
})

interface Req {
  name: string
}
interface Res {
  code: string
  data: {
    userName: string
  }
}

const getData = (data: Req) => {
  return myRequest.request<Res>({
    url: "/api/user/login",
    method: "POST",
    data
  })
}

onMounted(async () => {
  const res = await getData({
    name: "云牧"
  })
  console.log(res)
})
</script>
```

---



## 封装 LocalStorage

```typescript
interface LocalStorageItem<T> {
  value: T // 存储的值
  expire: number | null // 过期时间，如果为 null 则表示永不过期
}

class LocalStorage {
  private static instance: LocalStorage // 单例模式，保证只有一个实例
  private storage: Storage // localStorage 对象

  private constructor() {
    this.storage = window.localStorage // 获取 localStorage 对象
  }

  public static getInstance(): LocalStorage {
    if (!LocalStorage.instance) {
      // 如果实例不存在，则创建一个新实例
      LocalStorage.instance = new LocalStorage()
    }
    return LocalStorage.instance // 返回实例
  }

  public setItem<T>(key: string, value: T, expire?: number): void {
    const item: LocalStorageItem<T> = {
      value: value, // 存储的值
      expire: expire ? new Date().getTime() + expire : null, // 过期时间
    }
    this.storage.setItem(key, JSON.stringify(item)) // 将对象序列化为字符串并存储到 localStorage 中
  }

  public getItem<T>(key: string): T | null {
    const itemStr = this.storage.getItem(key) // 获取存储的字符串
    if (itemStr) {
      // 如果字符串存在
      const item: LocalStorageItem<T> = JSON.parse(itemStr) // 将字符串反序列化为对象
      if (!item.expire || new Date().getTime() < item.expire) {
        // 如果没有过期或者还没有过期
        return item.value // 返回存储的值
      } else {
        this.storage.removeItem(key) // 如果已经过期，则删除该项
      }
    }
    return null // 如果不存在或者已经过期，则返回 null
  }

  public removeItem(key: string): void {
    this.storage.removeItem(key) // 删除指定的项
  }

  public clear(): void {
    this.storage.clear() // 清空 localStorage
  }
}

export default LocalStorage.getInstance() // 导出 LocalStorage 实例
```

使用：

```typescript
<script lang="ts" setup>
import storage from "./storage"

// 存储数据
storage.setItem("name", "黛玉", 60 * 60 * 1000) // 存储一个过期时间为 1 小时的数据
storage.setItem("person", { name: "云牧" }, 60 * 60 * 1000) // 存储一个过期时间为 1 小时的数据

interface Person {
  name: string
}

// 获取数据
const name = storage.getItem<string>("name")
const person = storage.getItem<Person>("person")
console.log(name) // 黛玉
console.log(person.name) // 云牧

// 删除数据
storage.removeItem("name")

// 清空所有数据
storage.clear()
</script>
```



