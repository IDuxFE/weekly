> * 原文地址：[Understanding the ECMAScript spec, part 3](https://v8.dev/blog/understanding-ecmascript-part-3)
> * 原文作者：[marjakh](https://twitter.com/marjakh)
> * 译文出自：[IDuxFE 技术文章翻译](https://juejin.cn/column/7000191408518725662)
> * 本文永久链接：[https://github.com/IDuxFE/weekly/tree/main/translation/ECMAScript/understanding_the_ecmascript_spec_part3.md](https://github.com/IDuxFE/weekly/tree/main/translation/ECMAScript/understanding_the_ecmascript_spec_part3.md)
> * 译者： [Siyii](https://github.com/Siyii)
> * 校对者：

理解 ECMAScript 规范（三）
=========================================

发布于 2020 年 4 月 1 日，标签 [ECMAScript](/blog/tags/ecmascript) [理解 ECMAScript](/blog/tags/understanding-ecmascript)

[全集](/blog/tags/understanding-ecmascript)

在这节内容中，我们将会深入理解 ECMAScript 语言以及文法定义。ECMAScript 规范使用 [上下文无关文法](https://en.wikipedia.org/wiki/Context-free_grammar) 定义语言，如果你这个文法还不了解的话，最好现在去了解一下它的基础文法知识。

ECMAScript 文法 [#](#ecmascript-grammars)
---------------------------------------------

ECMAScript 规范定义了四种文法：

[词法文法](https://tc39.es/ecma262/#sec-ecmascript-language-lexical-grammar)描述了如何将 [Unicode 码点](https://en.wikipedia.org/wiki/Unicode#Architecture_and_terminology) 编译为**输入元素**（标记、换行符、注释、空格）序列。

[语法文法](https://tc39.es/ecma262/#sec-syntactic-grammar)定义了是如何用标记构成文法正确的程序。

[正则文法](https://tc39.es/ecma262/#sec-patterns)描述了是如何将 Unicode 码点编译成正则表达式。

[数值字符串文法](https://tc39.es/ecma262/#sec-patterns)描述了如何将字符串编译转译成数字类型的值。

每一种文法都由上下文无关文法进行定义，其中包含了一组产生式。

四种文法使用的表达式相差无二：语法文法表示为 `LeftHandSideSymbol`，词法文法和正则文法表示为 `LeftHandSideSymbol ::`，而数值字符串文法表示为 `LeftHandSideSymbol :::`。

接下来，我们详细分析一下词法文法和语法文法。

词法文法 [#](#lexical-grammar)
-------------------------------------

 规范将 ECMAScript 源文本定义为一串 Unicode 码点序列。举个例子，变量名称不仅可以是 ASCII 字符，还可以包含其它的 Unicode 字符。规范并没有谈论实际的编码 （比如 UTF-8 或者 UTF-16）。规范已经假定源码已经按照自己的编码方式转化为了 Unicode 码点序列。

很难在初始时对 ECMAScript 源码进行标记，这会让定义词法文法会变得稍显复杂。

比如说，如果不去看 `／` 所在的上下文环境是什么，就不知道 `／` 指的是除法操作符还是正则表达式开头。

    const x = 10 / 5;

这里的 `／` 指的是 `DivPunctuator`｡

    const r = /foo/;

这里第一个 `／` 指的是 `RegularExpressionLiteral` 的开头。

模板引入了类似的歧义 — 比如，``}` `` 的解释取决于它所在的上下文环境。

    const what1 = 'temp';const what2 = 'late';const t = `I am a ${ what1 + what2 }`;

这里的 `` `I am a ${``  是 `TemplateHead`，而 ``}` `` 是 `TemplateTail`。

    if (0 == 1) {}`not very useful`;

这里的 `}` 是 `RightBracePunctuator`，而 `` ` `` 是 `NoSubstitutionTemplate` 的开头。

即使 `/` 和 ``}` ``的解释取决于它们所在的上下文（它们在代码中的语法位置），但是我们接下来谈论的文法仍然是和上下文无关的。

词法文法中使用一些目标符来区分对不同上下文环境中允许哪些输入元素、不允许哪些输入元素。例如，目标符 `InputElementDiv` 被用于 `/` 是除法以及 `/=` 是除法赋值的上下文中。[`InputElementDiv`](https://tc39.es/ecma262/#prod-InputElementDiv) 产生式列出了在次上下文中可能产生的标记：

    InputElementDiv ::  WhiteSpace  LineTerminator  Comment  CommonToken  DivPunctuator  RightBracePunctuator

在这个上下文中，遇到 `/` 会产生 `DivPunctuator` ，并不会产生 `RegularExpressionLiteral`。

同样地，在一个 `/` 是正则开头的上下文中，[`InputElementRegExp`](https://tc39.es/ecma262/#prod-InputElementRegExp) 是对应的目标符。

    InputElementRegExp ::  WhiteSpace  LineTerminator  Comment  CommonToken  RightBracePunctuator  RegularExpressionLiteral

从上面的产生式中可以看出，可能产生 `RegularExpressionLiteral` 输入元素，但是不会产生 `DivPunctuator`。

相似的，这里还有另一种目标符, 在目标符 `InputElementRegExpOrTemplateTail` 所对应的上下文环境中， 除了允许 `RegularExpressionLiteral`，还允许 `TemplateMiddle` 和 `TemplateTail`。最后，`InputElementTemplateTail` 目标符对应的上下文环境只允许 `TemplateMiddle` and `TemplateTail`，不允许`RegularExpressionLiteral` 出现。

在实现过程中，语法文法分析器（解析器），又称为词法文法分析器（标记器或词法器），将目标符作为参数进行传递，并请求下一个符合这个目标符的输入元素。

Syntactic grammar [#](#syntactic-grammar)
语法文法 [#](#syntactic-grammar)
-----------------------------------------

词法文法法定义了如何从 Unicode 码点构建标记。而语法文法在词法文法的基础上，定义了如何将标记组成正确的程序。

### 例子：允许合法的标志符 [#](#example%3A-allowing-legacy-identifiers)

将一种新的关键词引入到文法中可能会造成翻天覆地的变化，要是原有的代码已经将这个关键词作为一个标志符在使用呢？

比如，在 `await` 被定为关键字之前，有人可能写过下面的代码：

    function old() {  var await;}

ECMAScript 文法在不影响代码正常运行下小心地加入了 `await` 关键字。在异步函数中，`await` 是一个关键词，因此下面的代码不能正常运行：

    async function modern() {  var await; // Syntax error}

类似地，在非生成器代码中 `yield` 可以是标志符，而在执行器中禁止使用作为标志符。

要理解 `await` 是如何被允许用作标志符的，需要理解 ECMAScript-specific 语法文法标记。让我们来看一下吧。

### 产生式和缩写 [#](#productions-and-shorthands)

让我们来看看 [`VariableStatement`](https://tc39.es/ecma262/#prod-VariableStatement) 产生式是如何被定义的。乍一看时，下面的文法可能有点可怕。

    VariableStatement[Yield, Await] :  var VariableDeclarationList[+In, ?Yield, ?Await] ;

下标(`[Yield, Await]`)和前缀(`+In` 里面的 `+`，以及 `?Async` 里的 `?`)表示什么意思呢？

这种表示法在[文法表示法](https://tc39.es/ecma262/#sec-grammar-notation)中有解释。

下标是表示一组产生式的缩写，总的来说，是一组产生式左端符号。产生式左端符号有两个参数，可以展开为四个”真正的“产生式左端符号，分别是：`VariableStatement`, `VariableStatement_Yield`，`VariableStatement_Await`, 以及 `VariableStatement_Yield_Await`.

注意这里的 `VariableStatement` 只是表示 `VariableStatement`，不包含 `_Await` 和 `_Yield` 在内的。不要把它和 `VariableStatement[Yield, Await]` 混淆了。

在产生式右端，缩写 `+In` 的意思是”使用带 `_In` 的版本“，`?Await` 意思是”当且仅当产生式左端符号中有 `_Await` 的话使用带 `_Await` 的版本“。（`?Yield` 同理）

第三个缩写，`~Foo` 没有在这个产生式中出现，它的意思是”使用没有的 `_Foo` 的版本“。

了解到上面的信息后，我们可以将上面的产生式展开：

    VariableStatement :  var VariableDeclarationList_In ;VariableStatement_Yield :  var VariableDeclarationList_In_Yield ;VariableStatement_Await :  var VariableDeclarationList_In_Await ;VariableStatement_Yield_Await :  var VariableDeclarationList_In_Yield_Await ;

最终，我们需要知道两件事情：

1. 我们在哪里判断是否处于要使用还是不使用 `_Await`的情况下？
2. `Something_Await` 和 `Something`(没有 `_Await`) 产生式的差异在哪里?

### `_Await` 还是没有 `_Await`? [#](#_await-or-no-_await%3F)

让我们先解决第一个问题。不管我们是否将 `_Await` 放到函数体前，我们都很容易区分异步函数和非异步函数。通过阅读异步函数声明的产生式，我们会发现[这个](https://tc39.es/ecma262/#prod-AsyncFunctionBody)：

    AsyncFunctionBody :  FunctionBody[~Yield, +Await]

注意 `AsyncFunctionBody` 是没有参数的，参数是在产生式右端被添加到 `FunctionBody` 中的。

如果我们展开上面的产生式，我们可以得到：

    AsyncFunctionBody :  FunctionBody_Await

换句话说，异步函数有 `FunctionBody_Await`，这意味着这个函数前的 `await` 会被当做一个关键字。

另一方面，如果是在一个非异步函数里面，对应的[产生式](https://tc39.es/ecma262/#prod-FunctionDeclaration) 是：

    FunctionDeclaration[Yield, Await, Default] :  function BindingIdentifier[?Yield, ?Await] ( FormalParameters[~Yield, ~Await] ) { FunctionBody[~Yield, ~Await] }

（`FunctionDeclaration` 还有一个别的产生式，但是它和我们的代码例子无关。）

为了避免组合式展开，我们会忽略不在这个特别的产生式中使用的 `Default` 参数。

这个产生式的展开形式为：

    FunctionDeclaration :  function BindingIdentifier ( FormalParameters ) { FunctionBody }FunctionDeclaration_Yield :  function BindingIdentifier_Yield ( FormalParameters ) { FunctionBody }FunctionDeclaration_Await :  function BindingIdentifier_Await ( FormalParameters ) { FunctionBody }FunctionDeclaration_Yield_Await :  function BindingIdentifier_Yield_Await ( FormalParameters ) { FunctionBody }

在这个产生式，只有 `FunctionBody` 和 `FormalParameters`(没有 `_Yield` 和 `_Await`) ，因为它们在未展开的产生式中都带有 `[~Yield, ~Await]` 参数。

函数名的处理是不一样的：如果产生式左端符号带上了 `_Await` 和 `_Yield`，则右端的函数名也会带上对应的参数。

总的来说：异步函数中有 `FunctionBody_Await`，非异步函数中有 `FunctionBody` (不含 `_Await`)。因为我们谈论的是非生成器函数，因此我们的异步函数例子和非异步函数都不携带 `_Yield` 参数。

要记住哪个是 `FunctionBody` 和 `FunctionBody_Await` 也许很难。在有 `FunctionBody_Await` 的函数中，`await` 是标志符，还是关键字呢？

你可以认为 `_Await` 参数指的是“`await` 是一个关键字”。这么说未来也不会出错。如果添加一个新的关键字 `blob`，只能在 “blobby” 函数中使用。非 “blobby”  和非异步，非生成器函数跟现在一样，仍然有 `FunctionBody`（不带 `_Await`, `_Yield` 和 `_Blob`）。“blobby” 函数中有 `FunctionBody_Blob`，异步 “blobby” 函数有 `FunctionBody_Await_Blob` 等等。我们虽然要在产生式中添加 `Blob` 下标，但是已存在函数的 `FunctionBody` 扩展形式仍保持原状。

### 不允许 `await` 用作标识符 [#](#disallowing-await-as-an-identifier)

接下来，我们需要探究在 `FunctionBody_Await` 中是如何禁止 `await` 被当作标记符的。

我们可以通过深入探究产生式，发现从 `FunctionBody` 到我们先前谈论的`VariableStatement` 产生式都携带了 `_Await` 参数。

因此，在一个异步函数中，有 `VariableStatement_Await`，而在一个非异步函数中，有 `VariableStatement`。

再更加深入产生式，并且注意到里面的参数。鉴于之前我们已经看到了 [`VariableStatement`](https://tc39.es/ecma262/#prod-VariableStatement) 产生式：

    VariableStatement[Yield, Await] :  var VariableDeclarationList[+In, ?Yield, ?Await] ;

所有的 [`VariableDeclarationList`](https://tc39.es/ecma262/#prod-VariableDeclarationList) 产生式都携带了参数：

    VariableDeclarationList[In, Yield, Await] :  VariableDeclaration[?In, ?Yield, ?Await]

(这里只展示和我们所举例子相关的[产生式](https://tc39.es/ecma262/#prod-VariableDeclaration)。)

    VariableDeclaration[In, Yield, Await] :  BindingIdentifier[?Yield, ?Await] Initializer[?In, ?Yield, ?Await] opt

`opt` 缩写的意思是产生式右端符号是可选的；实际上这里有两种产生式，一个是带可选的符号，另一个不带。

对于和我们所举例子相关的简单实例来说，`VariableStatement` 中包含了关键字 `var`，紧跟着一个没有初始化器的 `BindingIdentifier`，最后以分号收尾。

是否允许将 `await` 视为 `BindingIdentifier`，我们希望结果是下面这样：

    BindingIdentifier_Await :  Identifier  yieldBindingIdentifier :  Identifier  yield  await

这表示不允许在异步函数中把 `await` 作为标志符，但是在非异步函数中允许作为标志符。

规范中并没有给出这样的定义，但是我们找到了这个[产生式](https://tc39.es/ecma262/#prod-BindingIdentifier):

    BindingIdentifier[Yield, Await] :  Identifier  yield  await

将其扩展，会得到下面的产生式：

    BindingIdentifier_Await :  Identifier  yield  awaitBindingIdentifier :  Identifier  yield  await

(这里省略了例子里不需要的 `BindingIdentifier_Yield` 和 `BindingIdentifier_Yield_Await` 产生式。)

看起来似乎 `await` 和 `yield` 在任何场景下都可以当作标志符。为什么呢？是不是这篇文章白费功夫？

### 静态语义来拯救

事实证明，禁止在异步函数中将 `await` 作为一个标识符使用， 需要用到**静态语义**。

静态语义描述了一些在程序运行之前进行校验的静态规则。

在这里，[`BindingIdentifier` 的静态语义](https://tc39.es/ecma262/#sec-identifiers-static-semantics-early-errors) 定义了以下语法文法导向的规则：

>     BindingIdentifier[Yield, Await] : await
> 
> 如果这个产生式有一个 `[Await]` 参数，就是一个语法错误．

事实上，这会禁止 `BindingIdentifier_Await : await` 产生式。

规范中解释，之所以存在这个产生式但是又会被静态语义定义为文法错误，是由于与自动插入分号（ASI）有冲突。

当根据文法产生式无法将一行代码进行解析时，ASI 就会介入。为了满足语句和声明必须以分号结尾的要求，ASI 会尝试添加分号。（在下集我们会深入介绍 ASI）

思考以下的代码（摘自规范中）：

    async function too_few_semicolons() {  let  await 0;}

如果文法不允许将 `await`作为一个标识符使用，ASI 此时就会介入，把代码转化为下面所示文法正确的代码， `let` 会被当成一个标识符：

    async function too_few_semicolons() {  let;  await 0;}

ASI 的这种介入干涉会让人觉得困惑，因此静态语义才会用来会禁止将 `await` 当成标识符。

### 不允许标识符的 `StringValues`  [#](#disallowed-stringvalues-of-identifiers)

这里还有另外一条相关的规则：

>     BindingIdentifier : Identifier
> 
> 如果产生式有 `[Await]` 参数并且 `Identifier` 的 `StringValue` 是 `"await"`， 就是一个语法错误。

开始可能让人有点困惑。[标识符](https://tc39.es/ecma262/#prod-Identifier) 的定义是这样的：

    Identifier :  IdentifierName but not ReservedWord

`await` 是一个 `ReservedWord`，所以 `await` 怎么可以作为 `Identifier` 呢？

事实证明，`await` 不能作为 `Identifier`，但是它可以 `StringValue` 为 `"await"`（字符序列 `awat` 一种不同的表示方式） 的其他值。

[标志符名称的静态语义](https://tc39.es/ecma262/#sec-identifier-names-static-semantics-stringvalue)定义了如何计算标志符名称的 `StringValue`。举个例子，`a` 的 Unicode 转义序列是 `\u0061`，因此 `\u0061wait` 的 `StringValue` 是 `"await"`。词法文法不会把 `\u0061wait` 当作关键字，而是当作一个 `Identifier`。静态语义禁止在异步函数中将 `\u0061wait` 作为变量名。

因此，这样的代码可以运行：

    function old() {  var \u0061wait;}

这样的代码无法运行：

    async function modern() {  var \u0061wait; // Syntax error}

总结 [#](#summary)
---------------------

在这集内容中，我们了解了词法文法，语法文法，以及用于定义语法文法所用到的缩写。在示例中，我们了解到在异步方法中，`await` 不能作为标志符使用，但是在非异步方法中可以被当作标志符。

在下集内容中，会讲解关于语法文法其他有趣的部分，比如自动化插入逗号以及包含文法，敬请关注！