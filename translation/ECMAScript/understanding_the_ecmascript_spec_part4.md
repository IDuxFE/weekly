> * 原文地址：[Understanding the ECMAScript spec, part 4](https://v8.dev/blog/understanding-ecmascript-part-4)
> * 原文作者：[marjakh](https://twitter.com/marjakh)
> * 译文出自：[IDuxFE 技术文章翻译](https://juejin.cn/column/7000191408518725662)
> * 本文永久链接：[https://github.com/IDuxFE/weekly/tree/main/translation/ECMAScript/understanding_the_ecmascript_spec_part4.md](https://github.com/IDuxFE/weekly/tree/main/translation/ECMAScript/understanding_the_ecmascript_spec_part4.md)
> * 译者：BertramXue
> * 校对者：

理解 ECMAScript 规范（四）
=========================================

发布于 2020 年 5 月 19 日， 标签 [ECMAScript](/blog/tags/ecmascript) [理解 ECMAScript](/blog/tags/understanding-ecmascript)

[全集](/blog/tags/understanding-ecmascript)

Web共性问题 [#](#meanwhile-in-other-parts-of-the-web)
-----------------------------------------------------------------------------

来自Mozilla的[Jason Orendorff](https://github.com/jorendorff) 发表了 [深入分析 JS 诡异语法](https://github.com/mozilla-spidermonkey/jsparagus/blob/master/js-quirks.md#readme) 一文。 尽管在实现细节上有差异，但每个 JS 引擎在这些诡异的细节上都面对同样的问题。

包含文法 [#](#cover-grammars)
-----------------------------------

这篇文章我们将深入研究包含文法 _cover grammars_。包含文法是为那些一开始看起来模棱两可的语法构造规定语法的一种方式。

同样为简单起见，我们跳过下标 `[In, Yield, Await]` ，因为它们对本文并不重要。可以参考 [第三篇文章](/blog/understanding-ecmascript-part-3) 了解它们的含义和用法。

有限前查 [#](#finite-lookaheads)
-----------------------------------------

通常，解析器在有限前查 _finite lookahead_ (根据固定个数的标记) 的基础上决定使用哪个产生式。

在某些情况下，下一个标记可以明确地确定要使用的产生式。[例如](https://tc39.es/ecma262/#prod-UpdateExpression):

    UpdateExpression :  
        LeftHandSideExpression
        LeftHandSideExpression ++  
        LeftHandSideExpression --  
        ++ UnaryExpression  
        -- UnaryExpression

如果我们正在解析 `UpdateExpression` 且下一个标记是 `++` 或者是 `--`，那我们就可以马上知道要使用哪个产生式。如果下一个标记不是它们两个，那也问题不大：我们可以从所在位置开始解析 `LeftHandSideExpression`，解析完成后再决定下一步干什么。

如果 `LeftHandSideExpression` 后面的标记是 `++`，那么使用的产生式是`UpdateExpression : LeftHandSideExpression ++`。后面是 `--` 的情况类似。如果 `LeftHandSideExpression` 后面的标记既不是 `++` 也不是 `--`, 那么使用的产生式是 `UpdateExpression : LeftHandSideExpression`。

### 箭头函数形参列表还是括号表达式？ [#](#arrow-function-parameter-list-or-a-parenthesized-expression%3F)

将箭头函数形参列表与带括号的表达式区分开来更为复杂。

例如：

    let x = (a,

这是箭头函数的开头吗，像这样?

    let x = (a, b) => { return a + b };

或者它是一个括号表达式，像这样?

    let x = (a, 3);

不管括号中的内容是什么，但它们可能是任意长度的。因此不能根据有限标记确定它是什么。

想象一下，假设我们有下列直观的产生式：

    AssignmentExpression :  
        ...
        ArrowFunction  
        ParenthesizedExpression

    ArrowFunction :  
        ArrowParameterList => ConciseBody

那我们就不能使用有限前查来选择产生式。如果解析了 `AssignmentExpression` 的下一个标记是 `(`，那怎么确定接下来解析什么？我们可以解析 `ArrowParameterList`，也可以解析 `ParenthesizedExpression`， 但猜测可能会出错。

### 非常宽松的新符号：`CPEAAPL` [#](#the-very-permissive-new-symbol%3A-cpeaapl)

规范通过引入符号 `CoverParenthesizedExpressionAndArrowParameterList` (简写成`CPEAAPL`) 来解决这个问题。`CPEAAPL` 表示既可能是 `ParenthesizedExpression` 也可能是 `ArrowParameterList`，但目前还不知道选哪个。

[`CPEAAPL` 的产生式](https://tc39.es/ecma262/#prod-CoverParenthesizedExpressionAndArrowParameterList)非常宽松，允许所有可以出现在 `ParenthesizedExpression` 和 `ArrowParameterList` 中的构造：

    CPEAAPL :  
        ( Expression )  
        ( Expression , )  
        ( )  
        ( ... BindingIdentifier )
        ( ... BindingPattern )  
        ( Expression , ... BindingIdentifier )  
        ( Expression , ... BindingPattern )

例如, 下列表达式都是有效的 `CPEAAPL`：

    // 有效的ParenthesizedExpression and ArrowParameterList:
    (a, b)
    (a, b = 1)

    // 有效的ParenthesizedExpression:
    (1, 2, 3)
    (function foo() { })
    
    // 有效的ArrowParameterList:
    ()
    (a, b,)
    (a, ...b)
    (a = 1, ...b)
    
    // 两个都是无效的，但仍是 CPEAAPL:
    (1, ...b)
    (1, )

末尾的逗号和 `...` 只能出现在 `ArrowParameterList`。有些构造， 如 `b = 1` 两种情况下都有可能出现，只是含义不同: 出现在 `ParenthesizedExpression` 中是赋值，出现在 `ArrowParameterList` 中是带默认值的参数。 数值和其他不是有效参数名称（或参数解构模式）的`PrimaryExpressions` 只可能出现在 `ParenthesizedExpression`。但它们都可能出现在 `CPEAAPL`。

### 在产生式中使用 `CPEAAPL` [#](#using-cpeaapl-in-productions)

现在我们可以在[`AssignmentExpression` 产生式](https://tc39.es/ecma262/#prod-AssignmentExpression)中使用这个非常宽松的 `CPEAAPL`。(注意： `ConditionalExpression` 通过一条很长的生产链通向 `PrimaryExpression`，只是这里没有显示。)

    AssignmentExpression :  
        ConditionalExpression  
        ArrowFunction  
        ...

    ArrowFunction :  
        ArrowParameters => ConciseBody
    
    ArrowParameters :  
        BindingIdentifier  
        CPEAAPL

    PrimaryExpression :  
        ...  
        CPEAAPL

假设我们又回到了需要解析 `AssignmentExpression` 的情况，`AssignmentExpression` 的下一个标记是 `(`。现在我们可以解析 `CPEAAPL`，然后再考虑使用哪个产生式。此时是解析 `ArrowFunction` 还是解析 `ConditionalExpression` 并不重要，无论解析哪一个，下一个要解析的符号都是 `CPEAAPL`！

解析完 `CPEAAPL` 后, 就能决定最开始的（包含 `CPEAAPL` 的那个）`AssignmentExpression` 使用哪个产生式了。这是由 `CPEAAPL` 后面跟着的标记决定的。

如果这个标记是 `=>`，就使用下面的产生式：

    AssignmentExpression :  
        ArrowFunction

如果这个标记是其他什么，就使用下面的产生式：

    AssignmentExpression :  
        ConditionalExpression

例如：

    let x = (a, b) => { return a + b; };
    //      ^^^^^^
    //     CPEAAPL
    //             ^^
    //             跟在CPEAAPL后面的标记

    let x = (a, 3);
    //      ^^^^^^
    //     CPEAAPL
    //            ^
    //            跟在CPEAAPL后面的标记

此时，我们可以保持 `CPEAAPL` 不变，并继续解析程序的其余部分。例如，如果这个 `CPEAAPL` 在  `ArrowFunction` 中，我们还不需要看它是否是一个有效的箭头函数参数列表 —— 这可以在以后做。（实际的解析器可能会选择立即进行有效性检查，但从规范的角度来看，我们不需要这样做。）

### 限制 CPEAAPLs [#](#restricting-cpeaapls)

正如我们之前看到的，`CPEAAPL` 的产生式是非常宽松的，允许根本不合法的构造（例如 `(1, ...a)`）。 在按照文法解析完程序后，需要驳回其中不合法的构造。

规范通过添加以下限制来实现这一点：

> [静态语义：前期错误](https://tc39.es/ecma262/#sec-grouping-operator-static-semantics-early-errors)
> 
> `PrimaryExpression : CPEAAPL`
> 
> 如果 `CPEAAPL` 未包含 `ParenthesizedExpression` 就是一个语法错误。

> [补充语法](https://tc39.es/ecma262/#sec-primary-expression)
> 
> 在处理以下产生式的实例时：
> 
> `PrimaryExpression : CPEAAPL`
> 
> 对 `CPEAAPL` 的解释使用以下语法进行了精炼：
> 
> `ParenthesizedExpression : ( Expression )`

这意味着：如果 `CPEAAPL` 出现在语法树的 `PrimaryExpression` 的位置，那它实际上是 `ParenthesizedExpression`，且这是它唯一有效的产生式。

上面的 `Expression` 永远不能为空，因此 `( )` 不是有效的 `ParenthesizedExpression`。像 `(1, 2, 3)` 这样由逗号分隔的列表由[逗号操作符](https://tc39.es/ecma262/#sec-comma-operator)创建：

    Expression :  
        AssignmentExpression  
        Expression , AssignmentExpression

类似地，如果 `CPEAAPL` 出现在 `ArrowParameters` 中，则适用如下限制：

> [静态语义：前期错误](https://tc39.es/ecma262/#sec-arrow-function-definitions-static-semantics-early-errors)
> 
> `ArrowParameters : CPEAAPL`
> 
> 如果 `CPEAAPL` 未包含 `ArrowFormalParameters` 就是一个语法错误。

> [补充语法](https://tc39.es/ecma262/#sec-arrow-function-definitions)
> 
> 在处理以下产生式的实例时：
> 
> `ArrowParameters` : `CPEAAPL`
> 
> 对 `CPEAAPL` 的解释使用以下语法进行了精炼:
> 
> `ArrowFormalParameters :`  
> `( UniqueFormalParameters )`

### 其他包含文法 [#](#other-cover-grammars)

除了 `CPEAAPL` 之外, 规范还对其他模棱两可的构造使用了包含文法。

`ObjectLiteral` 用作 `ObjectAssignmentPattern` 的包含文法，它出现在箭头函数参数列表中。这意味着 `ObjectLiteral` 允许不能在实际的对象字面量中出现的构造。

    ObjectLiteral :  
        ...  
        { PropertyDefinitionList }

    PropertyDefinition :  
        ...  
        CoverInitializedName
    
    CoverInitializedName :  
        IdentifierReference Initializer

    Initializer :  
        = AssignmentExpression

例如：

    let o = { a = 1 }; // 语法错误
    
    // 箭头函数使用了带默认值的解构参数：
    let f = ({ a = 1 }) => { return a; };
    f({}); // 返回 1
    f({a : 6}); // 返回 6

异步箭头函数在使用有限前查时同样有歧义：

    let x = async(a,

调用 `async` 函数呢，还是一个异步箭头函数？

    let x1 = async(a, b);
    let x2 = async();
    function async() { }

    let x3 = async(a, b) => {};
    let x4 = async();

为此，文法定义了一个原理与 `CPEAAPL` 类似的包含文法符号 `CoverCallExpressionAndAsyncArrowHead`。

总结 [#](#summary)
---------------------

在本文中，我们研究了规范如何定义包含文法，以及如何在不能基于有限前查识别当前语法结构的情况下使用它们。

特别是，我们研究了箭头函数参数列表与括号表达式之间的区别，以及规范在碰到模棱两可的构造时怎么宽松地使用包含文法，然后使用静态语义规则限制它们。