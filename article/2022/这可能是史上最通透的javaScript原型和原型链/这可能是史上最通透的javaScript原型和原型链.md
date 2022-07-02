`javaScript`的原型和原型链是前端程序员的必修课，但真正掌握的人却很少，把`new`操作符讲明白的人就更少了。在大量阅读文章和视频资料后，发现这块知识都讲的太过原理性了，不太能被新手吸收。本文，作者通过最通透，最简单的方式一点点让你吃透`javaScript`的原型和原型链，让你的`js`功底更上一层楼。

#### 一、前置知识

在说正文之前，需要掌握一些前置知识。

`js`有很多数据类型，其中包括了**基本数据类型**和**引用数据类型**：

基本数据类型

```
string | number | boolean | obejct | null | undefined | Symbol | BigInt
```

引用数据类型

```
String | Number | Boolean | Object | Function | Array | Date | RegExp | Error
```

而引用数据类型都可以称为：**对象**，那么创建对象的方式也有很多种，我们可以用**构造函数**的方式创建对象：

```
let a = new String('abc');
let b = new Number(123);
let c = new Object();
c.name = '小东';
```

那好奇的小伙伴肯定会想，我创建个字符串或者数字，不都是直接申明就好了吗？

```
let a = 'abc';
let b = 123;
```

为什么还需要用构造函数的方式创建呢？而且用构造函数创建出来的字符串或者数字好像有点不太一样：

```
let a = new String('abc');
let b = new Number(123);
let c = 'abc';
let d = '123;
console.log(a); //String {'abc'}
console.log(b); //Number {'123'}
console.log(typeof a); //'object'
console.log(typeof b); //'object'
console.log(a === c); //false
console.log(b === d); //false
```

其实，我们平时直接用**字面量**的方式创建的字符串或者数字，都是用这种构造函数的方式`new`出来的，只是`js`帮我们简化了这个过程，你也可以理解为是个**语法糖**，我们可以用`valueOf`方法获取值：

```
let a = new String('abc');
let b = new Number(123);
console.log(a.valueOf); //'abc'
console.log(b.valueOf); //123
console.log(typeof a); //'string'
console.log(typeof b); //'number'
```

为什么变量 `a`可以直接通过`.`就可以拿到`valueOf`方法呢？下面将进入正题，解开这个谜题。

#### 二、原型

##### （1）初识原型

首先，先解答一下为什么变量 `a`可以直接通过`.`就可以拿到`valueOf`方法，在浏览器看一下

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8723485b9b9c43d8af9595cdcdd11e48~tplv-k3u1fbpfcp-zoom-1.image)

这里有个`[[Prototype]]`，这是**新版谷歌浏览器的展示**，如果是旧版，应该显示的是`__proto__`，点击箭头展开，就会发现方法`valueOf`在里面，我们先不管`[[Prototype]]`是什么，也不管为什么这里面有这么多的**方法**（通常在对象里面定义的函数，我们都称之为方法），根据原型链的规则，实例对象可以获取到原型上的属性或者方法（下面会讲）

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f1ca8224824548ed83f637e90d0ee5a3~tplv-k3u1fbpfcp-zoom-1.image)

`a`是由构造函数`new`出来的，即`new String()`，从上面的示例就可以看出来

```
[[Prototype]]: String
```

`Prototype`从字面意思看就是原型的意思，而这里指向`String`，并且前面有个箭头，可以展开，说明是个对象，所以我们也可以把`String`称之为**原型对象**，用一张图说明一下变量 `a`，构造函数`new String`，原型对象`String`的关系

![Untitled-2022-03-21-1502.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eebb388041ad46ba862fbf89cbd19f62~tplv-k3u1fbpfcp-zoom-1.image)

有些人又会说了，你这个`String`是内置的数据类型，那如果是普通函数，会怎么样呢？

```
function User() {
  this.name = '小东';
}
let xd = new User();
console.log(xd) //{'name': '小东'}
```

按照前面的思路，`xd`是变量，是通过构造函`new`出来的，即`new User()`，而`User`的原型对象是？？？

![Untitled-2022-03-21-1502.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/046d44f21a4d4ee09549709fb923259c~tplv-k3u1fbpfcp-zoom-1.image)

其实聪明的小伙伴已经看出来了，`User`的原型对象不就是`User`的`[[Prototype]]`的值嘛，在浏览器上看

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/845658a8877c4c5b9c4d51e98f823929~tplv-k3u1fbpfcp-zoom-1.image)

这是什么鬼？为什么`User`的原型对象是个`Object`呢？不应该是：

```
[[Prototype]]: User
```

Emmm.....，这里肯定不能这样玩，因为前面的**前置知识**有说过，`js`的数据类型只有那几种，肯定不包含`User`的，那为什么是`Object`呢？这是理所当然的，因为函数通过`new`出来的，即构造函数，最终是会生成一个对象的，那对象的原型对象自然是`Object`了。

##### （2）原型的表示

通常我们把`new`出来的对象称为**实例对象**，那实例对象和**原型对象**有什么关联呢？是不是可以这样表示呢？？

```
function User() {
  this.name = '小东';
}
let xd = new User();
​
xd[[Prototype]] ===> Object ???
```

`xd[[Prototype]]`这样语法肯定是会报错的，那`xd.Prototype`可以吗？也不行，因为`xd`身上也没有`Prototype`这个属性，哪里有？构造函数`User`身上有

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f9192311619546c5a8683bfafe975a72~tplv-k3u1fbpfcp-zoom-1.image)

这是可以真正通过`.`的方式获取到的，只不过没有了双括号`[[]]`，首字母也变成了小写，为什么实例对象`xd`没有有`prototype`，而构造函数`User`有呢，**因为`prototype`是函数特有的，而对象只有`[[Prototype]]`，其实这个`[[Prototype]]`就是`__proto__`** ，越说越乱，直接上图：

![Untitled-2022-03-21-1502.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f51900eb3c284b0a9e9f8d8bad3c761c~tplv-k3u1fbpfcp-zoom-1.image)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/91802f8530654f66b1be4fe49d418156~tplv-k3u1fbpfcp-zoom-1.image)

两句话总结：

0.  `prototype`是函数特有的属性，而函数又是引用数据类型，所以是`Object`的一种，只要是对象就有`__proto__`属性，所以函数有`prototype`和`__proto__`
0.  通过构造函数`new`出来的对象，不再是函数了，所以没有`prototype`，只有`__proto__`

##### （3）`prototype`和`__proto__`以及`[[Prototype]]`的区别

0.  `[[Prototype]]`是新版谷歌浏览器关于原型的表示，旧版直接显示为`__proto__`
0.  `prototype`通常称为显示原型；`__proto__`通常称为隐式原型
0.  `prototype`是函数特有的；`__proto__`只要是引用数据类型，都有
0.  `__proto__`是一个对象，它有两个属性，`constructor`和`__proto__`
0.  `prototype`是一个对象，它有一个属性，`constructor`
0.  构造函数通过`prototype`找到原型对象
0.  实例对象通过`__proto__`找到原型对象，原型对象通过`__proto__`找到`Object`
0.  函数的`prototype`一般给实例化对象使用，一般记录实例是由哪个构造函数创建的；函数的`__proto__`一般给函数自身使用

通过下面几张图看看这几者的关系：

![Untitled-2022-03-21-1502.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d66ac5cc0da44306a226a7324fb44050~tplv-k3u1fbpfcp-zoom-1.image)

**实例对象通过`__proto__`找到原型对象，原型对象通过`__proto__`找到`Object`**

![Untitled-2022-03-21-1502.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/425b2313c22543ec91fdaa39a5d1108f~tplv-k3u1fbpfcp-zoom-1.image)

**构造函数通过`prototype`找到原型对象**

![Untitled-2022-03-21-1502.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/79aeabcf80ba462f9238cb7f206cf8bf~tplv-k3u1fbpfcp-zoom-1.image)

**通过代码看一下三者（实例对象，构造函数，原型对象）的关系**

```
function User() {
  this.name = "小东";
}
​
console.log("构造函数：");
console.dir(User);
​
console.log("----------分界线----------");
​
var xd = new User();
​
console.log("实例对象：");
console.dir(xd);
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/48521a7b413542ffbdb4e6de53afe988~tplv-k3u1fbpfcp-zoom-1.image)

总结：

构造函数的显示原型（`prototype`）和实例对象的隐式原型（`__proto__`）相等

```
let a = new String('123');
console.log(String.prototype === a.__proto__); //true
​
let b = new Number(456);
console.log(Number.prototype === b.__proto__); //true
```

```
function User() {
  this.name = '小东';
}
​
let xd = new User();
​
console.log(User.prototype === xd.__proto__); //true
```

##### （4）constructor

上面的有提到：

0.  `__proto__`是一个对象，它有两个属性，`constructor`和`__proto__`
0.  `prototype`是一个对象，它有一个属性，`constructor`

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f2c0b543b6fa4518a10a9f16c2876504~tplv-k3u1fbpfcp-zoom-1.image)

那这个`constructor`是何方神圣呢？

这里就要举个经典的例子了，如果说实例对象是儿子，构造函数是父亲，原型对象是爷爷，这就是祖宗三代。那儿子想要找到爷爷，可以通过`__proto__`联系，如果父亲想找到爷爷，可以通过`prototype`联系，那爷爷想要找到父亲，或者说父亲想要找到儿子，该如何找呢？这里就是通过`constructor`建立联系

![Untitled-2022-03-21-1502.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c6285b8bc0814040b39f9285779a1621~tplv-k3u1fbpfcp-zoom-1.image)

**父亲找儿子**

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2a1ffcaf013a4b02b5cd7200bbe49245~tplv-k3u1fbpfcp-zoom-1.image)

**爷爷找父亲**

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d4076702ee5a4c509e3825bda66c8e21~tplv-k3u1fbpfcp-zoom-1.image)

由上面的两张图可以看出来：

0.  **实例对象**想要找到自己的**父亲**是谁，可以通过隐式原型`__proto__`里的`constructor`
0.  **实例对象**想要找到自己的**爷爷**是谁，可以通过隐式原型`__proto__`里的`constructor`的`__proto__`的`constructor`

```
let a = new Number(123);
​
//儿子找父亲
console.log(a.__proto__.constructor === Number); //true
​
//父亲找爷爷
console.log(Number.__proto__.constructor === Function); //true
​
//儿子找爷爷
console.log(a.__proto__.constructor.__proto__.constructor === Function); //true
```

##### （5）爷爷的父亲

上面的例子都是围绕子，父，爷三代展开的，如果说太爷爷还在世呢？就是爷爷的父亲，那这一家的族谱就更厉害了

太爷爷如何找到？按照这张图可以看出来，太爷爷应该是`Object`原型对象

![Untitled-2022-03-21-1502.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/65fe130b302e45b0b7e75f0d79c396c0~tplv-k3u1fbpfcp-zoom-1.image)

```
function User() {
  this.name = "小东";
}
​
let xd = new User();
​
//儿子找父亲
console.log(xd.__proto__.constructor === User); //true
​
//父亲找爷爷
console.log(User.__proto__.constructor === Function); //true
​
//爷爷找太爷爷
console.log(User.prototype.__proto__ === Object.prototype); //true
```

##### （6）祖宗对象

祖宗对象就是女娲，创建一切对象，对象原型的源头，既然”太爷爷“是`Object`的原型对象，那么我们看下`Object`再往上能找到什么

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/70a6a6ead13143e297401b9e10781ed0~tplv-k3u1fbpfcp-zoom-1.image)

没错，就是`null`，这实际是`js`设计之处的一个”缺陷“，我们用`typeof null` 查看

```
typeof null //'object'
```

##### （7）没有原型的对象

如果一个人，家道中落，没有父亲，爷爷，太爷爷，只有他一个人孤零零的，也是有可能的

```
let xd = Object.create(null, {
  name: {
    value: "小东",
  },
});
​
console.log(xd); //{name: '小东'}
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9e3d74ea1d174d3191e7c86c676eb691~tplv-k3u1fbpfcp-zoom-1.image)

我们通过`Object.create`方法创建一个新对象`xd`，并且指定原型为`null`

具体用法请看：[Object.create()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create)

##### （8）宗谱最终章

![Untitled-2022-03-21-1502.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/327d4e8c14b54bac8bf5bd8f09b389fb~tplv-k3u1fbpfcp-zoom-1.image)

#### 三、原型链

##### （1）原型链的起始

其实看完上面已经大致知道“原型链”的概念了，简单来说就是实例对象被创建后，从自身开始作为链条的起点，向上连接构造函数，原始数据类型，再到`Object`，然后再到`null`的过程，而`null`就是原型链的终点，原型链上的属性和方法都可以被实例对象访问的到，看下面这个例子

```
function User() {
  this.name = "小东";
}
​
User.prototype.show = function () {
  console.log(this.name);
};
​
let xd = new User();
​
xd.show(); //'小东'
```

实例对象`xd`身上没有`show`方法，但是构造函数的`prototype`的属性上有，根据原型链的规则，向上寻找最终找到`show`方法。那必须是构造函数的`prototype`吗？不能是实例对象的`__proto__`吗？

因为我们知道`User.prototype === xd.__proto__`，所以把`show`方法放在实例对象`__proto__`上或者放在构造函数`User`的`prototype`上，效果是一样，这里就不演示了。

那`show`方法可以绑定在构造函数`User`的`__proto__`上吗？答案是不行，`User`的`__proto__`指向的是`Function`，而`Function`的`__proto__`才是`Object`。还记的对象的`__proto__`有两个属性吗，一个是`constructor`，另一个是`__proto__`，所以我们需要这样做才能让实例对象`xd`找到`show`

```
function User() {
  this.name = "小东";
}
​
User.__proto__.__proto__.show = function () {
  console.log(this.name);
};
​
let xd = new User();
​
xd.show(); //'小东'
```

有些小伙伴可能对`User`的`__proto__`指向的是`Function`这里有些疑惑，我这里在在多说几句，因为`User`毕竟是函数，虽然作为构造函数，可以`new`出来一个对象，但自身是个函数，所以`__proto__`当然是`Function`，而`Function`是引用数据类型，所以`Function`的`__proto__`是`Object`，其他引用数据类型也是一样的

```
let a = new String("abc");
let b = new Number(123);
let c = new Function("arg", "console.log(arg)");
let d = new Boolean(true);
​
console.log(a instanceof Object); //true
console.log(b instanceof Object); //true
console.log(c instanceof Object); //true
console.log(d instanceof Object); //true
​
console.log(a.__proto__); //String {...}
console.log(b.__proto__); //Number {...}
console.log(c.__proto__); //f() {...}
console.log(d.__proto__); //Boolean {...}
​
console.log(a.__proto__.__proto__ === Object.prototype); //true
console.log(b.__proto__.__proto__ === Object.prototype); //true
console.log(c.__proto__.__proto__ === Object.prototype); //true
console.log(d.__proto__.__proto__ === Object.prototype); //true
```

所以，我们根据原型链的规则，在往上找，把`show`方法放在`Object.prototype`上也是可以的

```
function User() {
  this.name = "小东";
}
​
Object.prototype.show = function () {
  console.log(this.name);
};
​
let xd = new User();
​
xd.show(); //'小东'
```

那`Object`在往上呢？即`Object.__proto__`，此时就没有办法了，因为`Object.__proto__`是`null`，原型链到头了。

##### （2）原型链的优先级

如果`show`方法在实例对象`xd`上，并且在原型链上游挂载里另外一个`show`方法，但是内容不一样

```
function User() {
  this.name = "小东";
  this.show = function () {
    console.log(`我是${this.name}，今年18岁`);
  };
}
​
User.prototype.show = function () {
  console.log(this.name);
};
​
let xd = new User();
​
xd.show(); //'我是小东，今年18岁'
```

答案当然是使用自己身上的`show`，就好像你自己有钱，就不用管长辈要钱了的道理一样。

#### 四、检测原型

##### （1）instanceof

`instanceof` **运算符**用于检测**构造函数**的 `prototype` 属性是否出现在某个实例对象的原型链上

```
function User() {
  this.name = "小东";
}
​
let xd = new User();
​
console.log(xd instanceof User); //true
console.log(xd instanceof Object); //true
console.log(xd instanceof Number); //false
console.log(xd instanceof null); //false
```

详情看：[instanceof](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/instanceof)

##### （2）Object.prototype.isPrototypeOf

`isPrototypeOf` 方法用于测试一个对象是否存在于另一个对象的原型链上

> `isPrototypeOf()` 与 [`instanceof`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/instanceof) 运算符不同。在表达式 "`object instanceof AFunction`"中，`object` 的原型链是针对 `AFunction.prototype` 进行检查的，而不是针对 `AFunction` 本身。

```
function User() {
  this.name = "小东";
}
​
let xd = new User();
​
console.log(User.prototype.isPrototypeOf(xd)); //true
console.log(Object.prototype.isPrototypeOf(xd)); //true
```

```
let a = {};
let b = {};
​
console.log(Object.prototype.isPrototypeOf(a)); //true
console.log(Object.prototype.isPrototypeOf(b)); //true
console.log(b.isPrototypeOf(a)); //false
```

详情看：[Object.prototype.isPrototypeOf](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/isPrototypeOf)

##### （3）Object.prototype.hasOwnProperty

`hasOwnProperty()` 方法会返回一个布尔值，指示对象自身属性中是否具有指定的属性

```
let a = {
  name: "小东",
};
​
let b = {
  age: 18,
};
​
console.log(a.hasOwnProperty('name')); //true
console.log(a.hasOwnProperty('age')); //false
​
Object.prototype.age = 18;
console.log(a.hasOwnProperty('age')); //false
```

详情看：[Object.prototype.hasOwnProperty](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/hasOwnProperty)

##### （4）in

`in`不仅可以检测属性是否在对象上，还可以检测属性是否在对象的原型链上

```
let a = {
  name: "小东",
};
​
let b = {
  age: 18,
};
​
console.log("name" in a); //true
console.log("age" in a); //false
​
Object.prototype.age = 18;
console.log("age" in a); //true
```


##### （5）Object.getPrototypeOf

`Object.getPrototypeOf()` 方法返回指定对象的原型（内部`[[Prototype]]`属性的值）

```
function User() {
  this.name = "小东";
}
​
let xd = new User();
​
console.log(Object.getPrototypeOf(xd)); //{constructor: f User()}
```

#### 五、修改原型

到这里应该都清楚原型和原型链了，那有没有办法去改变对象上面的原型呢？答案是当然有的

##### （1）直接修改

```
let a = {
  name: "小东",
};
​
let b = {
  age: 18,
};
​
a.__proto__ = b;
​
console.log("age" in a);
```

```
let a = {
  name: "小东",
};
​
a.__proto__ = {
  show() {
    console.log(this.name);
  },
};
​
a.show() //小东
```

```
function User() {
  this.name = "小东";
}
​
User.prototype = {
  show() {
    console.log(this.name);
  },
};
​
let xd = new User();
​
xd.show(); //小东
```

但是这样改，`User`的显示原型`prototype`上就失去了`constructor`了，就失去了原型链的特性，所以我们可以手动补上一个

```
function User() {
  this.name = "小东";
}
​
User.prototype = {
  constructor: User,
  show() {
    console.log(this.name);
  },
};
​
let xd = new User();
​
xd.show(); //小东
```

##### （2）Object.setPrototypeOf

`Object.setPrototypeOf()`方法设置一个指定的对象的原型 ( 即, 内部`[[Prototype]]`属性）到另一个对象或 [`null`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/null)

> **警告：** 由于现代 JavaScript 引擎优化属性访问所带来的特性的关系，更改对象的 `[[Prototype]]`在***各个***浏览器和 JavaScript 引擎上都是一个很慢的操作。其在更改继承的性能上的影响是微妙而又广泛的，这不仅仅限于 `obj.__proto__ = ...` 语句上的时间花费，而且可能会延伸到***任何***代码，那些可以访问***任何***`[[Prototype]]`已被更改的对象的代码。如果你关心性能，你应该避免设置一个对象的 `[[Prototype]]`。相反，你应该使用 [`Object.create()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create)来创建带有你想要的`[[Prototype]]`的新对象

```
let a = {
  name: "小东",
};
​
let b = {
  age: 18,
};
​
Object.setPrototypeOf(a, b);
​
console.log("age" in a); //true
```

```
let b = {
  age: 18,
};
​
Object.setPrototypeOf(b, null); //相当于前面的Object.create(null)的操作
​
console.log(Object.prototype.isPrototypeOf(b)); //false
```




**以上就是本篇文章的全部内容，如果有哪里讲解错误，欢迎指正。**

**如果觉得讲的不错，对你有所帮助，求点赞评论关注收藏，谢谢！！！**

![src=http___5b0988e595225.cdn.sohucs.com_images_20170910_fde97e41003c4568b4e15c4aa7e81da7.jpeg&refer=http___5b0988e595225.cdn.sohucs.webp](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/67c00e33aa244831b1d2d22d516e5653~tplv-k3u1fbpfcp-zoom-1.image)
