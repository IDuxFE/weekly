# 什么是 JSON Schema

JSON Schema 是一套用来规范前后端 JSON 数据格式的一种约定方案。

试想一下，如果要确定一个字段是什么类型，用什么来约束这个字段呢？如果这个字段是字符串，那么它又需要符合什么样的正则？字符串长度限制在什么范围？如果这个字段是个数组，那么这个数组每一项应该是单模式，还是个元组？数组的长度在什么范围？这一系列问题都需要我们对数据格式作出一个确定方案，而 JSON Schema 就是来干这个事的。

# Ajv 校验

写好了的 JSON Schema 如果想要验证是否写对了，可以使用 ajv 来做校验，所以可以先写一个简单的 ajv 校验函数。

```js
// ajv.js
const Ajv = require("ajv");
const ajv = new Ajv();

function ajvValidate(schema, data) {
  const valid = ajv.validate(schema, data);
  if (!valid) {
    return ajv.errors;
  }
  return "passed!";
}

module.exports = {
  ajvValidate,
};
```

# 思维导图

这个思维导图列出了以下所写的内容。
![xmind.png](https://files.mdnice.com/user/23672/a0e7fb47-fb91-4cc2-9d58-42b3c032ac37.png)

# JSON Schema 规范

> type 关键字指明了 Schema 的类型， 不同的 type 会有不同的关键字，以下各个类型的例子有举例

## 关键字`type`

type 有`object`、`number（integer）`、`string`、`array`、`null`、`boolean`七种。

type 的值可以是一个字符串，也可以是一个字符串数组。

- type 是字符串

```js
const schema = {
  type: "boolean",
};
const data = false;
```

- type 是字符串数组

```js
// type.js
const { ajvValidate } = require("../ajv");
const schema2 = {
  type: ["string", "number"],
};

const data2 = "happy";
const data3 = 111;

console.log(ajvValidate(schema2, data2));
console.log(ajvValidate(schema2, data3));
```

## object

### 属性

> 使用关键字`properties`

```js
const { ajvValidate } = require("../ajv");

const schema = {
  type: "object",
  properties: {
    background: {
      type: "string",
    },
    width: {
      type: "number",
    },
    height: {
      type: "number",
    },
  },
};

const data1 = {}; // passed
const data2 = {
  // passed
  background: "red",
  width: 100,
  height: 100,
};
```

此外，可以使用`enum`为属性指定可选的属性值“集合”。这样，属性值只能是 enum 集合中的某个值。

```js
// object.js
const { ajvValidate } = require("../ajv");
const schema2 = {
  type: "object",
  properties: {
    color: {
      type: "string",
      enum: ["red", "blue", "pink", "black"],
    },
  },
};

const data3 = {
  color: "#999", // must be equal to one of the allowed values
};
```

### 模式属性

> 使用关键字`patternProperties`

也就是属性名使用正则来匹配，如果某个属性名匹配到了这个正则模式，那便使用这个正则模式属性的约束。

```js
const { ajvValidate } = require("../ajv");
// 正则模式属性
const schema = {
  type: "object",
  patternProperties: {
    "^A_": {
      type: "number",
    },
    Z_: {
      type: "string",
    },
  },
};

const data = {
  // passed
  A_A: 991,
  Z_Z: "hhhh",
};
```

### 额外属性

> 使用关键字`additionalProperties`

在没指定`additionalProperties`的时候，默认情况下，写上`properties`和`patternProperties`以外的属性也是可以的，但是如果`additionalProperties`设置为 false 的话，那便是不能添加额外的属性了。

```js
// object.js
const { ajvValidate } = require("../ajv");
// 额外看属性
const schema = {
  type: "object",
  properties: {
    width: { type: "number" },
    height: { type: "number" },
  },
  additionalProperties: false,
};

const data = { color: "blue" }; // must NOT have additional properties
```

还有一种情况是，可以通过 type 指定`additionalProperties`的类型，只有符合这个 type 的属性才能被加进来。

```js
additionalProperties: {
  type: "string";
}

const data = { color: false };

console.log(ajvValidate(schema, data)); // must be string
```

### 必须属性

> 使用关键字`required`，required 是一个数组

```js
const { ajvValidate } = require("../ajv");

const schema = {
  type: "object",
  properties: {
    width: { type: "number" },
    height: { type: "number" },
  },
  additionalProperties: {
    type: "string",
  },
  required: ["width", "height"],
};

const data = { color: "#991" };

console.log(ajvValidate(schema, data)); // must have required property 'width'
```

这里指定的必须属性有 width、height，但是报错信息只提示了 width 是必须的，而没有两个都一起提示。

### 属性数量

> 使用关键字`minProperties`、`maxProperties`

可以使用属性数量关键字来限制属性个数，`minProperties`、`maxProperties`必须是正整数或 0。

## integer

> 正负整数

有无小数点不能用于区分整数和非整数，比如 3.0 就是一个整数（虽然严格来说不是）。

## number

> 支持的格式包括：正负整数（1、-1 等）、浮点数（5.7、2.333 等）、指数计数法（2.0211208e10 等）

```js
// number.js
const { ajvValidate } = require("../ajv");
const schema = {
  type: "object",
  properties: {
    a: { type: "number" },
    b: { type: "number" },
    c: { type: "number" },
    d: { type: "number" },
    e: { type: "number" },
  },
};

const data = {
  a: 10,
  b: -29,
  c: 5.7,
  d: 2.333,
  e: 2.0211208e10,
};

console.log(ajvValidate(schema, data)); // passed
```

### 倍数

> 使用关键字`multipleOf`

```js
const schema2 = {
  type: "number",
  multipleOf: 3,
};

const data2 = 1;

console.log(ajvValidate(schema2, data2)); // must be multiple of 3
```

### 数值范围

> 使用关键字`minimum`、`maximum`、`exclusiveMinimum`、`exclusiveMaximum`

_帮助记忆 👇_

`minimum`（>=）与`exclusiveMaximum`（<），`maximum`（<=）与`exclusiveMinimum`（>）对应起来就包含了所有的范围。

## string

### 长度约束

> 使用关键字`minLength`、`maxLength`

字符串长度约束有两个关键字，`minLength`、`maxLength`。

```js
const { ajvValidate } = require("../ajv");

const schema1 = {
  type: "string",
  minLength: 5,
  maxLength: 10,
};

const data1 = "1234";
const data2 = "12345";
const data3 = "12345678901";
console.log(ajvValidate(schema1, data1)); // error
console.log(ajvValidate(schema1, data2)); // passed
console.log(ajvValidate(schema1, data3)); // error
```

### 试试给字符串加个正则

> 使用关键字`pattern`

```js
const schemaWithPattern = {
  type: "string",
  pattern: "^[A-J]{2}[0-9]{4}$",
};

const data4 = "AJ0001";
console.log(ajvValidate(schemaWithPattern, data4)); // passed
console.log(ajvValidate(schemaWithPattern, data5)); // error
console.log(ajvValidate(schemaWithPattern, data6)); // error
```

![](https://files.mdnice.com/user/23672/9c2a5087-5780-4a0c-8b47-01961cf8787b.png)

### 内置格式

string 类型的属性常用的内置格式有

| 内置格式类型  | 说明                  | 格式                      |
| ------------- | --------------------- | ------------------------- |
| `'data-time'` | 日期和时间            | 2018-11-13T20:20:39+00:00 |
| `time`        | 时间                  | 20:20:39+00:00            |
| `date`        | 日期                  | 2018-11-13                |
| `email`       | Internet 电子邮件地址 | -                         |
| `hostname`    | 主机                  | -                         |
| `ipv4`        | IPv4                  | -                         |
| `ipv6`        | IPv6                  | -                         |

_PS: 这里没一一列举所有的类型，具体的可以到 [Understanding JSON Schema](https://json-schema.org/understanding-json-schema/reference/string.html#format) 了解。_

## array

### `items`

JSON Schema 中的数组有两种验证形式，一种是列表验证，另一种是元组验证。

- 列表验证
  表示任意长度的序列，每个项目都匹配相同的模式。

```js
const schema = {
  type: "array",
  items: {
    type: "number",
  },
};

const data = [1, 2, 7, 9]; // passed
const data2 = ["1", "2"]; // error
```

- 元组验证
  关键字`items`这时是一个数组，`items`的每个元素对应了待验证的数组的每一项应该是什么数据格式。

```js
const schema = {
  type: "array",
  items: [
    { type: "number" },
    { type: "string" },
    { enum: ["beijing", "shanghai", "shenzhen", "guangzhou"] },
  ],
};

const data = [0001, "zhangsan", "beijing"]; // passed
const data2 = [0001, "zhangsan", "tianjin"]; // error, must be equal to one of the allowed values
```

### `additionalItems`

`additionalItems`表示的是附加元素，可以是一个对象，也可以是一个 boolean 值。

如果`additionalItems`为 false，那么将不允许在 items 指定的项之外再多加其他的元素。

```js
const schema = {
  type: "array",
  items: [
    { type: "number" },
    { type: "string" },
    { enum: ["beijing", "shanghai", "shenzhen", "guangzhou"] },
  ],
  additionalItems: false,
};

const data = [0001, "zhangsan", "beijing"]; // passed
const data2 = [0001, "lisi", "shenzhen", 24]; // error
```

当`addtionalItems`是一个模式对象，表示允许额外添加的数组项只能够是其指定的类型，比如下方这个例子，指明额外添加的数据只能是字符串。

```js
const schema = {
  type: "array",
  items: [
    { type: "number" },
    { type: "string" },
    { enum: ["beijing", "shanghai", "shenzhen", "guangzhou"] },
  ],
  additionalItems: {
    type: "string",
  },
};

const data2 = [0001, "991", "shenzhen", "give up struggle"]; // passed
```

### `contains`

`contains`表示包含的意思，顾名思义，数组中只要有一项符合就通过验证。

```js
const schema = {
  type: "array",
  contains: {
    type: "number",
  },
};

const data = ["Vue", "React"]; // must contain at least 1 valid item(s)
```

### `minItems`、`maxItems`

> 指定数组长度

```js
const schema = {
  type: "array",
  contains: {
    type: "number",
  },
  minItems: 2,
  maxItems: 4,
};

const data = ["Vue", "React", 1, 2, 3]; // must NOT have more than 4 items
const data2 = ["Vue", "React", 1, 2]; // passed
```

### `uniqueItems`

> 规定数组的每一项都是唯一的不可重复

```js
const schema = {
  type: "array",
  uniqueItems: true,
};

const data = [1, 1, 1, 2]; // error, must NOT have duplicate items (items ## 1 and 2 are identical)
const data = []; // passed
```

# 通用关键字

通过上面的例子可以看到，多处用到的`enum`便是其中一个通用的关键字，除此之外，还有一个比较有用的关键字是`const`。

- `enum`是枚举的意思，在这里，就是对数值多了一下限定，值只能是`enum`中的某一个。

- `const`就是常量的意思，把数值限定为某个固定的值。

```js
const schema = {
  type: "array",
  items: [{ type: "number" }, { const: "991" }],
};

const data = [0001, "900"]; // must be equal to constant
const data2 = [0001, "991"]; // passed
```

- 还有其他的通用关键字，比如`title`、`description`有兴趣可以自己去了解下，这里不再一一列举。

# Schema 组合

可以这样理解，通过指定多条规则来对某个 JSON 字段进行约束。

## allOf

> 必须对所有子模式有效

`allOf`必须是一个数组，待验证的字段需要完全符合`allOf`的所有项

```js
const schema = {
  allOf: [
    {
      type: "string",
    },
    {
      maxLength: 5,
    },
  ],
};

const data = "123456"; // must NOT have more than 5 characters
const data2 = "12345"; // passed
const data3 = 12345; // must be string
```

## oneOf

> 必须对恰好一个子模式有效，且是有且只有一个满足才有效，多个满足将视为无效

```js
const schema = {
  oneOf: [
    { type: "number", multipleOf: 4 },
    { type: "number", multipleOf: 7 },
  ],
};

const data = 8; // passed
const data2 = 14; // passed
const data3 = 28; // must match exactly one schema in oneOf
```

## anyOf

> 只要有符合任何一个或多个子模式即有效

```js
const schema = {
  anyOf: [
    { type: "number", multipleOf: 4 },
    { type: "number", multipleOf: 7 },
  ],
};

const data = 8; // passed
const data2 = 14; // passed
const data3 = 28; // passed
```

## not

> 针对 not 指定模式之外的内容做校验

比如针对不是布尔类型的内容做校验

```js
const schema = {
  not: { type: "boolean" },
};

const data = [1, 2, 3, 4, 5, 6]; // passed
const data2 = "a"; // passed
```

## 分解模式

可以把共有的提取出来

```js
const schema = {
  allOf: [
    {
      type: "string",
      maxLength: 10,
    },
    {
      type: "string",
      pattern: "^[A-J]{2}[0-9]{4}$",
    },
  ],
};
// 等价于
const schema = {
  type: "string",
  allOf: [{ maxLength: 10 }, { pattern: "^[A-J]{2}[0-9]{4}$" }],
};
```

# 有条件地应用子模式

## dependencies

> 表示依赖关系，如果是数组，表示属性间依赖，可以单向依赖也可以双向依赖；如果是模式，表示模式依赖

### 属性依赖

```js
const schema = {
  type: "object",
  properties: {
    id: { type: "number" },
    name: { type: "string" },
    age: { type: "number" },
  },
  // 双向依赖
  dependencies: {
    name: ["id"],
    id: ["name"],
  },
};
const data = { id: 00001, name: "991", age: 10 }; // passed
const data2 = { age: 100 }; // passed
const data3 = { name: "zhangsan" }; // must have property id when property name is present
const data4 = { id: 00002 }; // must have property name when property id is present
```

### 模式依赖

```js
const schema = {
  type: "object",
  properties: {
    id: { type: "number" },
    name: { type: "string" },
    age: { type: "number" },
  },
  dependencies: {
    name: {
      properties: {
        address: { enum: ["beijing", "shanghai", "tianjin"] },
      },
      required: ["address"],
    },
  },
};
// name依赖于address，但name不是必须的
const data = { id: 000000003, name: "lisi", age: 1000 }; // must have required property 'address'
const data2 = { id: 000000003 }; // passed
const data3 = { address: "beijing" }; // passed
```

## 条件语句

> `if then else`

if A then B, else C （A 与 B 是队友，C 是自己一个）

| if(A)  | then(B)      | else(C)      | result     |
| ------ | ------------ | ------------ | ---------- |
| T      | T            | 忽略或未定义 | passed     |
| T      | F            | 忽略或未定义 | not passed |
| F      | 忽略或未定义 | T            | passed     |
| F      | 忽略或未定义 | F            | not passed |
| 未定义 | 未定义       | 未定义       | passed     |

上方表格我是这么理解的。

- A passed 了，B 也得 passed，否则 result 将 not passed
- A not passed，就看 C passed 不 passed
- A passed, B passed, C 被忽略或未定义，result passed
- A passed, B not passed, C 被忽略或未定义，result not passed
- A not passed, B 被忽略未定义，C passed, result passed
- A not passed, B 被忽略未定义, C not passsed, result not passed
- A、B、C 均未定义，result passed

以下是一些例子。

### A passed, B passed, C 被忽略或未定义，result passed

```js
// A passed, B passed, C被忽略或未定义，result passed
const schema = {
  type: "object",
  properties: {
    background: { enum: ["red", "blue", "pink", "black"] },
  },
  if: {
    properties: {
      background: { const: "red" },
    },
  },
  then: {
    properties: { code: { type: "number" } },
  },
  else: {
    properties: { code: { type: "string" } },
  },
};
const data = { background: "red", width: 100 }; // passed
```

### A passed, B not passed, C 被忽略或未定义，result not passed

```js
// A passed, B not passed, C被忽略或未定义，result not passed
const schema = {
  type: "object",
  properties: {
    background: { enum: ["red", "blue", "pink", "black"] },
  },
  if: {
    properties: {
      background: { const: "red" },
    },
  },
  then: {
    properties: { code: { type: "number" } },
  },
};
const data2 = { background: "red", code: "300" }; // must be number
```

还有一种特殊情况，没有 background 属性，默认情况下 if 是会 passed 的，这样，then 语句就会被要求也得通过，

### A not passed, B 被忽略或未定义，C passed, result passed

```js
// A not passed, B 被忽略或未定义，C passed, result passed
const schema = {
  type: "object",
  properties: {
    background: { enum: ["red", "blue", "pink", "black"] },
    width: { enum: [100, 300, 500, 700] },
  },
  if: {
    properties: {
      background: { const: "red" },
    },
  },
  else: {
    properties: {
      width: { const: 300 },
    },
  },
};
const data3 = { background: "black", width: 300 }; // passed
```

### A not passed, B 被忽略或未定义, C not passsed, result not passed

```js
// A not passed, B 被忽略或未定义, C not passsed, result not passed
const schema = {
  type: "object",
  properties: {
    background: { enum: ["red", "blue", "pink", "black"] },
  },
  if: {
    properties: {
      background: { const: "red" },
    },
  },
  else: {
    properties: { code: { type: "string" } },
  },
};
const data2 = { background: "pink", code: 300 }; // must be string
```

```js
// A、B、C均未定义，result passed
const schema = {
  type: "object",
  properties: {
    background: { enum: ["red", "blue", "pink", "black"] },
    width: { enum: [100, 300, 500, 700] },
  },
};
const data = { background: "red" }; // passed
```

## 蕴含

蕴含可以理解为`A->B`（若 A 则 B，A 隐含 B）。这是不是就有点像上面写的 if then 了呢？其实是可以这么理解。

| if then | if else | if then else     |
| ------- | ------- | ---------------- |
| `A->B`  | `!A->B` | `A->B AND !A->C` |

非（`!`）即`not`关键字，结合模式组合（`allOf`、`oneOf`、`anyOf`、`not`）和 if then else 可以写出复杂的模式。

# 进一步探索-构建复杂模式

## 声明方言

> 使用关键字`$schema`

关键字的值也是模式的标识符。一个 JSON Schema 的版本便是一个方言。

- Draft 4 的标识符是`http://json-schema.org/draft-04/schema#`
- Draft 6 的标识符是`http://json-schema.org/draft-06/schema#`
- Draft 7 的标识符是`http://json-schema.org/draft-07/schema#`

没有对应的 Draft 5 方言，Draft 5 是 Draft 4 版本的无变化修订版。

## 声明唯一标识符

> 使用关键字`$id`

- `$id`有两个用途

  - 1. 用来唯一标识一个 schema
  - 2. 用来作为`$ref`的 base url 被解析

- `$id`相当于 html 中的 base 标签。

  _PS: HTML \<base> 元素   指定用于一个文档中包含的所有相对 URL 的根 URL。_

- 在 draft 4 中，`$id` 只是 id（没有`$`符号）。

## 一个模式引用另一个模式

> 使用关键字`$ref`

`$ref`的值是一个 JSON 指针。

### 根据`$id`的 base URI 来解析值

```js
{
    $id: 'https://example.com/schema/user_info',
    type: 'object',
    properties: {
        local_address: { $ref: '/schema/address' } // 这里$ref是使用了相对URI
    }
}

// $ref的值实际是https://example.com/schema/address
```

### 结合`$defs`关键字使用

这种是在复杂的模式中常用的使用方法。`$defs`关键字用来存放想在当前模式文档中复用的子模式。

```js
const schema = {
  // 虽然 https://991.com/schemas/user_info 是构造的一个URI，但不妨碍模式的引用
  $id: "https://991.com/schemas/user_info",
  type: "object",
  properties: {
    firstName: { $ref: "#/$defs/username" },
    lastName: { $ref: "#/$defs/username" },
  },
  required: ["firstName", "lastName"],

  $defs: {
    username: { type: "string" },
  },
};

const data = { firstName: "Martin", lastName: "Herry" }; // passed
```

### $ref 实现递归

通常在树形结构的数据中有用。使用符号`#`。

```js
const schema = {
  // 虽然 https://991.com/schemas/user_info 是构造的一个URI，但不妨碍模式的引用
  $id: "https://991.com/schemas/user_info",
  type: "object",
  properties: {
    name: { type: "string" },
    children: {
      type: "array",
      items: { $ref: "#" },
    },
  },
  required: ["name"],
};
const data = {
  name: "Herry Martin",
  children: [
    { name: "A1", children: [{ name: "A2" }] },
    { name: "B1", children: [{ name: "B2" }] },
  ],
};
```

## 锚点指针

> 使用`$anchor`关键字

锚点值必须以字母开头，后跟任意数量的字母、数字、`-`、`_`、`:`、 或`.`。

```js
const schema = {
  $id: "https://991.com/schemas/user_info",
  type: "object",
  properties: {
    firstName: { $ref: "#/$defs/username", $anchor: "#firstName" },
  },
  $defs: {
    username: { type: "string" },
  },
};
// 以上 https://991.com/schemas/user_info#firstName 便是一个锚点
```

以上记录了一些常用的 JSON Schema 规范，还有其他一些不怎么用到的这里就不一一列举了，有需要了解的可以点击参考链接前往查阅。还有在使用过程中，需要注意一些版本问题，不同版本，规范也有一些差异。

# 参考

1. [Understanding JSON Schema](https://json-schema.org/understanding-json-schema/)
2. [JSON Schema 规范（中文版）](https://json-schema.apifox.cn/)
3. [Ajv JSON schema validator](https://ajv.js.org/)
