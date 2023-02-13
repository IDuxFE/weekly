> * 原文地址：[Understanding the ECMAScript spec, part 1](https://v8.dev/blog/understanding-ecmascript-part-1)
> * 原文作者：[marjakh](https://twitter.com/marjakh)
> * 译文出自：[IDuxFE 技术文章翻译](https://juejin.cn/column/7000191408518725662)
> * 本文永久链接：[https://github.com/IDuxFE/weekly/tree/main/translation/ECMAScript/understanding_the_ecmascript_spec_part1.md](https://github.com/IDuxFE/weekly/tree/main/translation/ECMAScript/understanding_the_ecmascript_spec_part1.md)
> * 译者：TaylenaWu
> * 校对者：

理解 ECMAScript 规范（一）
=========================================

发布于 2020 年 2 月 3 日，标签 [ECMAScript](/blog/tags/ecmascript) [理解 ECMAScript](/blog/tags/understanding-ecmascript)

[全集](/blog/tags/understanding-ecmascript)

在这篇文章中，我们会从规范中找一个简单的功能，并试着理解其中的符号。让我们开始吧！

前言 [#](#preface)
---------------------

即使你了解 JavaScript ，阅读它的语言规范 [ECMAScript Language specification, or the ECMAScript spec for short](https://tc39.es/ecma262/) 可能也是令人望而生畏的。至少这是我第一次开始阅读时的感受。

让我们以一个具体例子作为开始，然后通过规范去理解它。下面的代码演示了 `Object.prototype.hasOwnProperty` 的使用：

    const o = { foo: 1 };o.hasOwnProperty('foo'); // trueo.hasOwnProperty('bar'); // false

在这个例子中，`o` 并没有一个 `hasOwnProperty` 属性，所以我们通过原型链寻找。我们发现它在 `o` 的原型对象，也就是 `Object.prototype` 上。

为了描述 `Object.prototype.hasOwnProperty` 是怎样生效的，规范使用了类似伪代码的说明：

> **[`Object.prototype.hasOwnProperty(V)`](https://tc39.es/ecma262#sec-object.prototype.hasownproperty)**
> 
> 当用参数 `V` 调用 `hasOwnProperty` 方法时，以下步骤将会执行：
> 
> 1.  令 `P` 为 `? ToPropertyKey(V)`；
> 2.  令 `O` 为 `? ToObject(this value)`；
> 3.  返回 `? HasOwnProperty(O, P)`。

以及

> **[`HasOwnProperty(O, P)`](https://tc39.es/ecma262#sec-hasownproperty)**
> 
> 抽象操作 `HasOwnProperty` 被用于确定是否一个对象拥有一个带有特定键的属性。会返回一个布尔值。这个操作以参数 `O` 和 `P` 调用，`O` 是对象，`P` 是属性键。这个抽象操作执行以下步骤：
> 
> 1.  断言： `Type(O)` 为 `Object`；
> 2.  断言： `IsPropertyKey(P)` 为 `true`；
> 3.  令 `desc` 为 `? O.[[GetOwnProperty]](P)`；
> 4.  若 `desc` 为 `undefined`， 返回 `false`；
> 5.  返回 `true`。

但是什么是“抽象操作”呢？在 `[[ ]]` 中的又是什么东西？为什么在函数前面有一个 `?` ？断言又是什么意思？

让我们找出答案吧！

语言类型和规范类型 [#](#language-types-and-specification-types)
-----------------------------------------------------------------------------------

让我们以一些看起来很眼熟的东西作为开始。 规范使用了一些值例如 `undefined`, `true`, 和 `false`, 我们早已从 JavaScript 中了解它们了。它们都是 [**语言值**](https://tc39.es/ecma262/#sec-ecmascript-language-types), 即规范定义的 **语言类型** 的值。

规范内部也使用了语言值，举个例子，一个内部的数据类型可能包含一个值是 `true` 和 `false` 的字段。与此相反，JavaScript 引擎内部通常不会使用语言值。例如：如果 JavaScript 引擎是用 C++写的，它通常会使用 C++的 `true` 和 `false` （而不是其 JavaScript 的 `true` 和 `false` 的内部表示形式）。

除了语言类型之外，规范还使用了 [**规范类型**](https://tc39.es/ecma262/#sec-ecmascript-specification-types)，规范类型是只存在在规范中的类型，而不在 JavaScript 语言中。JavaScript 引擎不需要（但完全可以）去实现它们。在这篇文章中，我们将会去了解规范类型记录（和它的子类型完成记录）。

抽象操作 [#](#abstract-operations)
---------------------------------------------

[**抽象操作**](https://tc39.es/ecma262/#sec-abstract-operations) 是定义在 ECMAScript 规范中的函数；定义它们的目的是为了更简洁的书写规范。JavaScript 引擎内部不需要将它们作为单独的函数来实现。这些函数不能直接被 JavaScript 调用。

内部槽位和内部方法 [#](#internal-slots-and-internal-methods)
-----------------------------------------------------------------------------

[**内部槽位** 和 **内部方法**](https://tc39.es/ecma262/#sec-object-internal-methods-and-internal-slots) 使用包含在 `[[ ]]` 中的名称。

内部槽位是 JavaScript 对象或者规范类型的数据成员。它们被用来存储对象的状态。内部方法是 JavaScript 对象的成员方法。

例如，每个 JavaScript 都有一个内部槽位 `[[Prototype]]` 和内部方法 `[[GetOwnProperty]]`。

内部槽位和方法是无法从 JavaScript 进行访问的。比如，你不能访问 `o.[[Prototype]]` 或者调用 `o.[[GetOwnProperty]]()` 。为了自己内部的的使用，JavaScript 引擎可以实现此功能，但是没有必要。

有时候内部方法委托给相似名称的抽象操作，例如普通对象的 `[[GetOwnProperty]]:` 

> **[`[[GetOwnProperty]](P)`](https://tc39.es/ecma262/#sec-ordinary-object-internal-methods-and-internal-slots-getownproperty-p)**
> 
> 当 `O` 的内部方法 `[[GetOwnProperty]]` 被属性键 `P` 调用时, 将会执行以下步骤：
> 
> 1.  返回 `! OrdinaryGetOwnProperty(O, P)`。

（我们将在下一章中了解感叹号的含义。）

`OrdinaryGetOwnProperty` 不是内部方法，因为它不与任何对象相关联；相反的，它将所操作的对象作为一个参数传递。

`OrdinaryGetOwnProperty` 被叫做 “ordinary”（普通） 是因为它只操作普通对象。ECMAScript 对象可以是**普通对象** 或者 **异质对象**。普通对象必须对一组被称为 **基本内部方法** 的方法具有默认行为。如果一个对象偏离了默认行为，那它就是异质对象。

最著名的异质对象就是 `Array`，因为它的 length 属性表现出了非默认的行为：设置 `length` 属性可能会从 `Array` 删除元素。

基本内部方法在[此处](https://tc39.es/ecma262/#table-5)列举 。

完成记录 [#](#completion-records)
-------------------------------------------

问号和感叹号又是什么意思呢？为了理解它们，我们需要去浏览 [**完成记录**](https://tc39.es/ecma262/#sec-completion-record-specification-type)!

完成记录是一种规范类型（仅为了规范目的而定义）。JavaScript 引擎不需要有一个相关的内部数据类型。

完成记录是一种记录 —— 有一组固定的命名字段的数据类型。一个完成记录包含三个字段：

名字

描述

`[[Type]]`

`normal`, `break`, `continue`, `return`, 或 `throw` 其中之一。除了 `normal` 之外的其它类型被称为 **硬性完成**。

`[[Value]]`

完成时生成的值，例如函数的返回值或者异常（如果有异常抛出）。

`[[Target]]`

用于定向控制转移（与本文无关）。

每个抽象操作会隐式返回一个完成记录。即使抽象操作看起来像是会返回一个基本数据类型比如布尔值，这个值也是会被包裹在一个 `normal` 类型的完成记录里面（见 [隐式完成值](https://tc39.es/ecma262/#sec-implicit-completion-values)）。

注 1 ：规范在这一方面也不是完全一致的；有一些辅助函数会返回裸值，并且其返回值按原样使用，而不需要从完成记录里提取值。这在上下文中通常是很清楚的。

注 2 ：规范编辑正在考虑更加显式地处理完成记录。

如果一个算法抛出异常，那意味着返回了一个 `[[Type]]` 为 `throw`， `[[Value]]` 为异常对象的完成记录。在这里，我们先忽略 `break`, `continue` 和 `return` 。

[`ReturnIfAbrupt(argument)`](https://tc39.es/ecma262/#sec-returnifabrupt) 表示执行以下步骤：

> 1.  若 `argument` 为硬性完成，返回 `argument`；
> 2.  设置 `argument` 为 `argument.[[Value]]`。

也就是说，我们会检查一个完成记录，如果是硬性完成，则立即返回。否则，我们将会从完成记录中取值。

`ReturnIfAbrupt` 可能看起来像是一个函数调用，但其实不然。它导致了 `ReturnIfAbrupt()` 生成位置的函数的返回，而不是 `ReturnIfAbrupt()` 函数本身。这种行为更像是 C 语言中的宏。

`ReturnIfAbrupt` 可以像这样使用：

> 1.  令 `obj` 为 `Foo()`；(`obj` 是一个完成记录。)
> 2.  `ReturnIfAbrupt(obj)`。
> 3.  `Bar(obj)`。 (如果我们到了这一步， `obj` 已经是一个从完成记录里面提取的值了。)

现在该说到[问号](https://tc39.es/ecma262/#sec-returnifabrupt-shorthands)了: `? Foo()` 等价于 `ReturnIfAbrupt(Foo())`。 使用简写是实用的: 我们不需要每一次都显式地编写错误处理代码。

类似地， `令 val 为 ! Foo()` 等价于:

> 1.  令 `val` 为 `Foo()`;
> 2.  断言： `val` 不是一个硬性完成;
> 3.  设 `val` 为 `val.[[Value]]`;

知道了这些之后，我们可以像这样重写 `Object.prototype.hasOwnProperty` ：

> **`Object.prototype.hasOwnProperty(V)`**
> 
> 1.  令 `P` 为 `ToPropertyKey(V)`；
> 2.  若 `P` 是一个硬性完成，返回 `P`。
> 3.  设 `P` 为 `P.[[Value]]`；
> 4.  令 `O` 为 `ToObject(this value)`；
> 5.  若 `O` 是一个硬性完成。返回 `O`。
> 6.  设 `O` 为 `O.[[Value]]`；
> 7.  令 `temp` 为 `HasOwnProperty(O, P)`；
> 8.  若 `temp` 是硬性完成，返回 `temp`；
> 9.  令 `temp` 为 `temp.[[Value]]`；
> 10.  返回 `NormalCompletion(temp)`。

我们也可以像这样重写 `HasOwnProperty` ：

> **`HasOwnProperty(O, P)`**
> 
> 1.  断言： `Type(O)` 为 `Object`；
> 2.  断言： `IsPropertyKey(P)` 为 `true`；
> 3.  令 `desc` 为 `O.[[GetOwnProperty]](P)`；
> 4.  若 `desc` 不是一个硬性完成，返回 `desc`。
> 5.  设 `desc` 为 `desc.[[Value]]`；
> 6.  若 `desc` 为 `undefined`, 返回 `NormalCompletion(false)`；
> 7.  返回 `NormalCompletion(true)`。

我们也可以重写 `[[GetOwnProperty]]` 内部方法且不带感叹号：

> **`O.[[GetOwnProperty]]`**
> 
> 1.  令 `temp` 为 `OrdinaryGetOwnProperty(O, P)`；
> 2.  断言： `temp` 是一个硬性完成；
> 3.  令 `temp` 为 `temp.[[Value]]`；
> 4.  返回 `NormalCompletion(temp)`。

这里我们假设 `temp` 是一个新的临时变量，它不与其它任何变量产生冲突。

我们还使用了这个知识：当返回语句返回的不是完成记录时，返回值会被隐式地包裹在一个 `NormalCompletion` 中。

### 拓展知识: `Return ? Foo()` [#](#side-track%3A-return-%3F-foo())

规范使用了 `Return ? Foo()` ，为什么这里要写问号呢？

`Return ? Foo()` 展开写是:

> 1.  令 `temp` 为 `Foo()`；
> 2.  若 `temp` 是硬性完成，返回 `temp`；
> 3.  设 `temp` 为 `temp.[[Value]]`；
> 4.  返回 `NormalCompletion(temp)`。

这和 `Return Foo()` 是一样的；它在硬性完成和正常完成中表现出一样的行为。

`Return ? Foo()` 仅仅只用于编辑，来使得 `Foo` 更显式地返回一个完成记录。

断言 [#](#asserts)
---------------------

断言在规范中断言了算法的不变条件。它们是为了明确起见而添加的， 但是并不要求实现它们，也就是说，实现并不要求检查这些条件。

继续 [#](#moving-on)
-------------------------

抽象操作委托给其它抽象操作（见下图），但是基于这篇文章我们应该能够弄清楚它们做了什么。我们会遇到属性操作符，这是另一种规范类型。

![](/_img/understanding-ecmascript-part-1/call-graph.svg)

函数调用图从 `Object.prototype.hasOwnProperty` 开始

小结 [#](#summary)
---------------------

我们通读了一个简单的方法 `Object.prototype.hasOwnProperty` ，和它所调用的**抽象操作** 。熟悉了与错误处理有关的简写 `?` 和 `!` 。我们也了解了 **语言类型**, **规范类型**, **内部槽位**, 和 **内部方法**。

相关链接 [#](#useful-links)
-------------------------------

[如何阅读 ECMAScript 规范](https://timothygu.me/es-howto/): 此教程从一个稍微不同的角度涵盖了这篇文章中的大部分素材。