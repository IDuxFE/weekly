> * 原文地址：[Understanding the ECMAScript spec, part 2](https://v8.dev/blog/understanding-ecmascript-part-2)
> * 原文作者：[marjakh](https://twitter.com/marjakh)
> * 译文出自：[IDuxFE 技术文章翻译](https://juejin.cn/column/7000191408518725662)
> * 本文永久链接：[https://github.com/IDuxFE/weekly/tree/main/translation/ECMAScript/understanding_the_ecmascript_spec_part2.md](https://github.com/IDuxFE/weekly/tree/main/translation/ECMAScript/understanding_the_ecmascript_spec_part2.md)
> * 译者：Levix
> * 校对者：

第二部分，了解 ECMAScript 规范
=========================================

2020 年 3 月 2 日发布 · 标签：[ECMAScript](/blog/tags/ecmascript) [了解 ECMAScript](/blog/tags/understanding-ecmascript)

让我们再多练习一下阅读规范的技巧，如果你还没有看过上一集，现在正是学习它的好时机！

[全集](/blog/tags/understanding-ecmascript)

准备好第二部分的学习了吗？[#](#ready-for-part-2%3F)
-------------------------------------------

一个了解规范的好方法是从我们熟知的 JavaScript 特性开始，并弄清楚它是如何被定义的。

> 警告! 截止至 2020 年 2 月，本集包含的算法是从 [ECMAScript 规范](https://tc39.es/ecma262/) 中复制粘贴的，它们最终会被废弃的。

我们知道，属性是通过原型链查找得到的：如果一个对象没有我们要读取的属性，我们会沿着原型链向上查找，直到找到它（或者找到一个不再有原型的对象）。

例如：

```js
const o1 = { foo: 99 };
const o2 = {};
Object.setPrototypeOf(o2, o1);
o2.foo;
// → 99
```

原型路径的定义在哪里？[#](#prototype-walk)
--------------------------------------------------------

我们一起尝试找出这种行为是在哪里定义的，可以从[对象的内部方法](https://tc39.es/ecma262/#sec-object-internal-methods-and-internal-slots)列表开始。

（每个对象）都包含 `[[GetOwnProperty]]` 和 `[[Get]]` （两个基本内部方法） —— 我们感兴趣的是不限制于 _own_ 属性的版本，所以我们将选择 `[[Get]]` 。

遗憾的是，[Property Descriptor 规范类型](https://tc39.es/ecma262/#sec-property-descriptor-specification-type)也有一个叫做 `[[Get]]` 的字段，所以在浏览 `[[Get]]` 的规范时，我们需要仔细区分这两种各自的使用方式。

`[[Get]]` 是一个**基本的内部方法**，**普通对象**实现了基本内部方法的默认行为。**外来对象**可以定义自己的内部方法 `[[Get]]` ，但它偏离了默认行为。在这篇文章中，我们专注于讨论普通对象。

对 `[[Get]]` 的默认实现委托给了 `OrdinaryGet` ：

> **[`[[Get]] ( P, Receiver )`](https://tc39.es/ecma262/#sec-ordinary-object-internal-methods-and-internal-slots-get-p-receiver)**
> 
> 当 `O` 的内部方法 `[[Get]]` 被属性键为 `P` 以及 ECMAScript 语言值为 `Receiver` 调用时，将执行以下步骤：
> 
> 1.  返回 `? OrdinaryGet(O, P, Receiver)`.

我们很快就会看到， `Receiver` 是在调用访问器属性的 getter 函数时，被用作 **this 值**。

`OrdinaryGet` 定义如下：

> **[`OrdinaryGet ( O, P, Receiver )`](https://tc39.es/ecma262/#sec-ordinaryget)**
> 
> 在以对象 `O` 、属性键 `P` 和 ECMAScript 语言值 `Receiver` 调用抽象操作 `OrdinaryGet` 时，将执行以下步骤：
> 
> 1.  断言： `IsPropertyKey(P)` 为 `true` ；
> 2.  令 `desc` 为 `? O.[[GetOwnProperty]](P)` ；
> 3.  若 `desc` 为 `undefined` ，则
>     a.  令 `parent` 为 `? O.[[GetPrototypeOf]]()` ；
>     b.  若 `parent` 为 `null` ，返回 `undefined` ；
>     c.  返回 `? parent.[[Get]](P, Receiver)` ；
> 4.  若 `IsDataDescriptor(desc)` 为 `true` ， 返回 `desc.[[Value]]` ；
> 5.  断言： `IsAccessorDescriptor(desc)` 为 `true` ；
> 6.  令 `getter` 为 `desc.[[Get]]` ；
> 7.  若 `getter` 为 `undefined` ，返回 `undefined` ；
> 8.  返回 `? Call(getter, Receiver)` 。

原型链的查找行为在步骤 3 里面：如果我们没找到自有属性（`desc`），调用原型的 `[[Get]]` 方法，并再次委托（抽象操作） `OrdinaryGet` 。假如我们依然没有找到该属性，继续调用原型的 `[[Get]]` 方法，又会再次委托 `OrdinaryGet` ，以此类推，直到找到该属性或者遇到一个没有原型的对象为止。

让我们看看当我们访问 `o2.foo` 时，该算法是如何工作的。首先我们令 `O` 为 `o2` ，`P` 为 `"foo"` 去调用 `OrdinaryGet` ， `O.[[GetOwnProperty]]("foo")` 返回 `undefined` ，因为 `o2` 没有名为 `"foo"` 的自有属性，所以我们在步骤 3 采用 if 分支。在步骤 3.a 中，我们将 `parent` 设置为 `o2` 的原型，即 `o1` ，`parent` 不为 `null`，因此在步骤 3.b 中不会直接返回。在步骤 3.c 中，我们调用 parent 的 `[[Get]]` 方法，传入 `"foo"` ，并返回最终结果。

parent（`o1`）是一个普通对象，所以它的 `[[Get]]` 方法会再次调用 `OrdinaryGet`，这一次 `O` 为 `o1` ， `P` 为 `"foo"` `o1` 有一个叫 `"foo"` 的自有属性。因此在步骤 2 中， `O.[[GetOwnProperty]]("foo")` 返回相应的 Property Descriptor （属性描述符），并将其保存在 `desc` 中。

[Property Descriptor](https://tc39.es/ecma262/#sec-property-descriptor-specification-type)是一种规范类型，数据属性描述符把属性的值直接保存在 `[[Value]]` 字段中，访问器属性描述符把访问器函数保存在字段 `[[Get]]` 和/或 `[Set]` 中，在这种情况下，与 `"foo"` 关联的属性描述符是一个数据属性描述符。

步骤 2 中保存在 `desc` 中的数据属性描述符不是 `undefined` ，因此我们在步骤 3 中没有使用 `if` 分支，接下来执行步骤 4 ，该属性描述符是一个数据属性描述符，因此我们返回其 `[[Value]]` 字段值为 `99` ，步骤 4 到此结束。

`Receiver` 是什么，它来自哪里？[#](#receiver)
-------------------------------------------------------------

`Receiver` 参数只在步骤 8 的访问器属性中使用，当调用访问器属性的 `getter` 函数时，它被用作 `this 值` 传递。

`OrdinaryGet` 在整个递归过程中传递初始的 `Receiver` ，没有变化（步骤 3.c），让我们来看看 `Receiver` 最初来源于哪里！

在搜索调用 `[[Get]]` 的地方时，我们发现了一个抽象操作 `GetValue` ，它对引用进行操作。引用是一种规范类型，由一个基础值、一个引用名和一个严格的引用标志组成。以 `o2.foo` 为例，基础值是对象 `o2` ，引用名是字符串 `"foo"` ，且严格的引用标志是 `false` ，原因是示例代码没有启用严格模式。

### 题外话：为什么是引用而非记录？ [#](#side-track%3A-why-is-reference-not-a-record%3F)

题外话：引用并非记录，尽管它听起来可能是，引用包含 3 个组成部分，同样可以被表达为 3 个命名字段。引用并非记录，这是一个历史遗留问题。

### 回到 `GetValue` [#](#back-to-getvalue)

让我们看看 `GetValue` 是如何定义的：

> **[`GetValue ( V )`](https://tc39.es/ecma262/#sec-getvalue)**
> 
> 1.  `ReturnIfAbrupt(V)` ；
> 2.  若 `Type(V)` 为非 `Reference` ，则返回 `V` ；
> 3.  令 `base` 为 `GetBase(V)` ；
> 4.  若 `IsUnresolvableReference(V)` 为 `true` ， 抛出 `ReferenceError` 异常；
> 5.  若 `IsPropertyReference(V)` 为 `true` ， 则
>     a.  若 `HasPrimitiveBase(V)` 为 `true` ， 则
>         i.  断言： 在这种情况下， `base` 永远不会是 `undefined` 或者 `null` ；
>         ii.  设置 `base` 为 `! ToObject(base)` ；
>     b.  返回 `? base.[[Get]](GetReferencedName(V), GetThisValue(V))` 。
> 6.  否则，
>     a.  断言： `base` 是一个 Environment Record.
>     b.  返回 `? base.GetBindingValue(GetReferencedName(V), IsStrictReference(V))`

我们例子中的引用是 `o2.foo` ，是一个属性引用，因此我们选择分支 5 ，我们不会进入 5.a 分支，因为基础对象（`o2`）不是[一个原始值](/blog/react-cliff#javascript-types)（Number、String、Symbol、BigInt、Boolean、Undefined 或者 Null）。

接着我们在步骤 5.b 中调用 `[[Get]]` ，而 `Receiver` 是通过 `GetThisValue(V)` 传入的，此时，它就是引用的基础值。

> **[`GetThisValue( V )`](https://tc39.es/ecma262/#sec-getthisvalue)**
> 
> 1.  断言： `IsPropertyReference(V)` 为 `true` ；
> 2.  若 `IsSuperReference(V)` 为 `true` ，则
>     a.  返回引用 `V` 的 `thisValue` 组成部分的值；
> 3.  返回 `GetBase(V)` 。

对于 `o2.foo` ，我们在第 2 步不使用分支，因为它不是一个父类（Super Reference）引用（如 `super.foo`），但我们会进入第 3 步并返回引用的基础值 `o2` 。

综上所述，我们发现我们把 `Receiver` 设置为原始引用的基础值，然后在原型链查找过程中保持其不变。最终，如果我们要找的是一个访问器属性，则在调用它的时候使用 `Receiver` 作为 `this 值` 。 

尤其是，getter 中的 **this 值**指向我们试图从中获取属性的原始对象，而不是在原型链遍历期间发现该属性的对象。

我们来试试看！

```js
const o1 = { x: 10, get foo() { return this.x; } };
const o2 = { x: 50 };
Object.setPrototypeOf(o2, o1);
o2.foo;
// → 50
```

在该例中，我们有一个名为 `foo` 的访问器属性，并为它定义了一个 getter 方法，getter 返回 `this.x` 。

接着我们访问 `o2.Foo` —— getter 函数会返回什么？

我们发现，调用 getter 函数时， **this 值**是我们最初尝试获取属性的对象，而不是我们发现该属性的对象。在这种情况下， **this 值**是 `o2` ，而不是 `o1` ，我们可以通过检查 getter 是返回 `o2.x` 还是 `o1.x` 来验证，的确，它返回 `o2.x` 。

行得通！我们能够根据阅读规范来预测这段代码的行为。

访问属性 —— 它为什么会调用 `[[Get]]` ？[#] (# property-access-get)
------------------------------------------------------------------------------

规范中哪里说过当访问 `o2.foo` 这样的属性时， Object 的内部方法 `[[Get]]` 会被调用？事实上它肯定在某个地方被定义了，切记不要听风就是雨！

我们发现对象内部方法 `[[Get]]` 是在抽象操作 `GetValue` 中调用的，该操作对引用进行操作，但这又是哪里调用的 `GetValue` 呢？

### MemberExpression` 的运行时语义 [#](#memberexpression)

规范中的文法规则定义了语言的语法。[运行时语义](https://tc39.es/ecma262/#sec-runtime-semantics)定义了语法结构的“含义”（如何在运行时求出它们的值）。

如果你不熟悉[上下文无关文法](https://en.wikipedia.org/wiki/Context-free_grammar)，现在可以去了解一下！

我们将在以后的章节中更深入地了解文法规则，现在我们大致了解一下即可！特别是，我们可以忽略本章中产生式的下标(`Yield` ， `Await` 等)。

以下的产生式描述了什么是 [`MemberExpression`](https://tc39.es/ecma262/#prod-MemberExpression) ：

```grammar
MemberExpression :
  PrimaryExpression
  MemberExpression [ Expression ]
  MemberExpression . IdentifierName
  MemberExpression TemplateLiteral
  SuperProperty
  MetaProperty
  new MemberExpression Arguments
```

这里有 7 个 `MemberExpression` 的产生式。一个 `MemberExpression` 可以只是一个 `PrimaryExpression` ，另外，一个 `MemberExpression` 可以由另一个 `MemberExpression` 和 `Expression` 构造得到： `MemberExpression [ Expression ]` ，例如 `o2['foo']` 。或者它也可以是 `MemberExpression . IdentifierName` ，如 `o2.foo` —— 这是与我们例子相关的产生式。

产生式 `MemberExpression : MemberExpression . IdentifierName` 的运行时语义定义了对它求值时的步骤：

> **[运行时语义： `MemberExpression : MemberExpression . IdentifierName` 求值](https://tc39.es/ecma262/#sec-property-accessors-runtime-semantics-evaluation)**
> 
> 1.  令 `baseReference` 为 `MemberExpression` 的求值结果；
> 2.  令 `baseValue` 为 `? GetValue(baseReference)` ；
> 3.  若 `MemberExpression` 匹配的代码为严格模式代码，令 `strict` 为 `true`；否则令 `strict` 为 `false` ；
> 4.  返回 `? EvaluatePropertyAccessWithIdentifierKey(baseValue, IdentifierName, strict)` 。

该算法委托给抽象操作 `EvaluatePropertyAccessWithIdentifierKey` ，所以我们也需要读取它：

> **[`EvaluatePropertyAccessWithIdentifierKey( baseValue, identifierName, strict )`](https://tc39.es/ecma262/#sec-evaluate-property-access-with-identifier-key)**
> 
> 抽象操作 `EvaluatePropertyAccessWithIdentifierKey` 将值 `baseValue`、解析节点 `identifierName` 和布尔参数 `strict` 作为参数，它执行以下步骤：
> 
> 1.  断言： `identifierName` 作为 `IdentifierName`
> 2.  令 `bv` 为 `? RequireObjectCoercible(baseValue)` ；
> 3.  令 `propertyNameString` 作为 `identifierName` 的 `StringValue` ；
> 4.  返回引用类型值，其基础值为 `bv` ，其引用名称为 `propertyNameString` ，其严格引用标志为 `strict` 。

也就是说： `EvaluatePropertyAccessWithIdentifierKey` 构造了一个引用，它使用提供的 `baseValue` 作为基础，字符串 `identifierName` 的值作为属性名，`strict` 作为严格模式标志。

最终，这个引用被传递给 `GetValue` ，在规范中它有几个地方进行了定义，具体取决于最终如何使用该引用。

### `MemberExpression` 作为一个参数 [#](#memberexpression-as-a-parameter)

在我们的例子中，我们使用属性访问作为一个参数：

```js
console.log(o2.foo);
```

在这种情况下，该行为被定义在 `ArgumentList` 运行时语义的产生式中，它在参数上调用了 `GetValue` 。

> **[运行时语义：`ArgumentListEvaluation` （赋值表达式）](https://tc39.es/ecma262/#sec-argument-lists-runtime-semantics-argumentlistevaluation)**
> 
> `ArgumentList : AssignmentExpression`
> 
> 1.  令 `ref` 为 `AssignmentExpression` 的求值结果；
> 2.  令说 `arg` 为 `? GetValue(ref)` ；
> 3.  返回只包含 `arg` 的列表。

`o2.foo` 看起来不像是一个 `AssignmentExpression` ，但它确实是，所以适用于该产生式。

步骤 1 中的 `AssignmentExpression` 是 `o2.foo` ，而 `o2.foo` 的求值结果 `ref` 就是上面提到的引用，在步骤 2 中我们基于它调用 `GetValue` 。因此，我们就知道了 Object 的内部方法 `[[Get]]` 会被调用，而原型链遍历也将触发。

总结 [#](#summary)
---------------------

在本文中我们看到了规范是如何定义一个语言特性的，如原型查找，跨越了所有不同的层级：触发该特性的语法结构和定义该特性的算法。