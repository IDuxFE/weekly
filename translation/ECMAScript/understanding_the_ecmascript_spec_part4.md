> * 原文地址：[Understanding the ECMAScript spec, part 4](https://v8.dev/blog/understanding-ecmascript-part-4)
> * 原文作者：[marjakh](https://twitter.com/marjakh)
> * 译文出自：[IDuxFE 技术文章翻译](https://juejin.cn/column/7000191408518725662)
> * 本文永久链接：[https://github.com/IDuxFE/weekly/tree/main/translation/ECMAScript/understanding_the_ecmascript_spec_part4.md](https://github.com/IDuxFE/weekly/tree/main/translation/ECMAScript/understanding_the_ecmascript_spec_part4.md)
> * 译者：
> * 校对者：

Understanding the ECMAScript spec, part 4
=========================================

Published 19 May 2020 · Tagged with [ECMAScript](/blog/tags/ecmascript) [Understanding ECMAScript](/blog/tags/understanding-ecmascript)

[All episodes](/blog/tags/understanding-ecmascript)

Meanwhile in other parts of the Web [#](#meanwhile-in-other-parts-of-the-web)
-----------------------------------------------------------------------------

[Jason Orendorff](https://github.com/jorendorff) from Mozilla published [a great in-depth analysis of JS syntactic quirks](https://github.com/mozilla-spidermonkey/jsparagus/blob/master/js-quirks.md#readme). Even though the implementation details differ, every JS engine faces the same problems with these quirks.

Cover grammars [#](#cover-grammars)
-----------------------------------

In this episode, we take a deeper look into _cover grammars_. They are a way to specify the grammar for syntactic constructs which look ambiguous at first.

Again, we'll skip the subscripts for `[In, Yield, Await]` for brevity, as they aren't important for this blog post. See [part 3](/blog/understanding-ecmascript-part-3) for an explanation of their meaning and usage.

Finite lookaheads [#](#finite-lookaheads)
-----------------------------------------

Typically, parsers decide which production to use based on a finite lookahead (a fixed amount of following tokens).

In some cases, the next token determines the production to use unambiguously. [For example](https://tc39.es/ecma262/#prod-UpdateExpression):

    UpdateExpression :  LeftHandSideExpression  LeftHandSideExpression ++  LeftHandSideExpression --  ++ UnaryExpression  -- UnaryExpression

If we're parsing an `UpdateExpression` and the next token is `++` or `--`, we know the production to use right away. If the next token is neither, it's still not too bad: we can parse a `LeftHandSideExpression` starting from the position we're at, and figure out what to do after we've parsed it.

If the token following the `LeftHandSideExpression` is `++`, the production to use is `UpdateExpression : LeftHandSideExpression ++`. The case for `--` is similar. And if the token following the `LeftHandSideExpression` is neither `++` nor `--`, we use the production `UpdateExpression : LeftHandSideExpression`.

### Arrow function parameter list or a parenthesized expression? [#](#arrow-function-parameter-list-or-a-parenthesized-expression%3F)

Distinguishing arrow function parameter lists from parenthesized expressions is more complicated.

For example:

    let x = (a,

Is this the start of an arrow function, like this?

    let x = (a, b) => { return a + b };

Or maybe it's a parenthesized expression, like this?

    let x = (a, 3);

The parenthesized whatever-it-is can be arbitrarily long - we cannot know what it is based on a finite amount of tokens.

Let's imagine for a moment that we had the following straightforward productions:

    AssignmentExpression :  ...  ArrowFunction  ParenthesizedExpressionArrowFunction :  ArrowParameterList => ConciseBody

Now we can't choose the production to use with a finite lookahead. If we had to parse a `AssignmentExpression` and the next token was `(`, how would we decide what to parse next? We could either parse an `ArrowParameterList` or a `ParenthesizedExpression`, but our guess could go wrong.

### The very permissive new symbol: `CPEAAPL` [#](#the-very-permissive-new-symbol%3A-cpeaapl)

The spec solves this problem by introducing the symbol `CoverParenthesizedExpressionAndArrowParameterList` (`CPEAAPL` for short). `CPEAAPL` is a symbol that is actually an `ParenthesizedExpression` or an `ArrowParameterList` behind the scenes, but we don't yet know which one.

The [productions](https://tc39.es/ecma262/#prod-CoverParenthesizedExpressionAndArrowParameterList) for `CPEAAPL` are very permissive, allowing all constructs that can occur in `ParenthesizedExpression`s and in `ArrowParameterList`s:

    CPEAAPL :  ( Expression )  ( Expression , )  ( )  ( ... BindingIdentifier )  ( ... BindingPattern )  ( Expression , ... BindingIdentifier )  ( Expression , ... BindingPattern )

For example, the following expressions are valid `CPEAAPL`s:

    // Valid ParenthesizedExpression and ArrowParameterList:(a, b)(a, b = 1)// Valid ParenthesizedExpression:(1, 2, 3)(function foo() { })// Valid ArrowParameterList:()(a, b,)(a, ...b)(a = 1, ...b)// Not valid either, but still a CPEAAPL:(1, ...b)(1, )

Trailing comma and the `...` can occur only in `ArrowParameterList`. Some constructs, like `b = 1` can occur in both, but they have different meanings: Inside `ParenthesizedExpression` it's an assignment, inside `ArrowParameterList` it's a parameter with a default value. Numbers and other `PrimaryExpressions` which are not valid parameter names (or parameter destructuring patterns) can only occur in `ParenthesizedExpression`. But they all can occur inside a `CPEAAPL`.

### Using `CPEAAPL` in productions [#](#using-cpeaapl-in-productions)

Now we can use the very permissive `CPEAAPL` in [`AssignmentExpression` productions](https://tc39.es/ecma262/#prod-AssignmentExpression). (Note: `ConditionalExpression` leads to `PrimaryExpression` via a long production chain which is not shown here.)

    AssignmentExpression :  ConditionalExpression  ArrowFunction  ...ArrowFunction :  ArrowParameters => ConciseBodyArrowParameters :  BindingIdentifier  CPEAAPLPrimaryExpression :  ...  CPEAAPL

Imagine we're again in the situation that we need to parse an `AssignmentExpression` and the next token is `(`. Now we can parse a `CPEAAPL` and figure out later what production to use. It doesn't matter whether we're parsing an `ArrowFunction` or a `ConditionalExpression`, the next symbol to parse is `CPEAAPL` in any case!

After we've parsed the `CPEAAPL`, we can decide which production to use for the original `AssignmentExpression` (the one containing the `CPEAAPL`). This decision is made based on the token following the `CPEAAPL`.

If the token is `=>`, we use the production:

    AssignmentExpression :  ArrowFunction

If the token is something else, we use the production:

    AssignmentExpression :  ConditionalExpression

For example:

    let x = (a, b) => { return a + b; };//      ^^^^^^//     CPEAAPL//             ^^//             The token following the CPEAAPLlet x = (a, 3);//      ^^^^^^//     CPEAAPL//            ^//            The token following the CPEAAPL

At that point we can keep the `CPEAAPL` as is and continue parsing the rest of the program. For example, if the `CPEAAPL` is inside an `ArrowFunction`, we don't yet need to look at whether it's a valid arrow function parameter list or not - that can be done later. (Real-world parsers might choose to do the validity check right away, but from the spec point of view, we don't need to.)

### Restricting CPEAAPLs [#](#restricting-cpeaapls)

As we saw before, the grammar productions for `CPEAAPL` are very permissive and allow constructs (such as `(1, ...a)`) which are never valid. After we've done parsing the program according to the grammar, we need to disallow the corresponding illegal constructs.

The spec does this by adding the following restrictions:

> [Static Semantics: Early Errors](https://tc39.es/ecma262/#sec-grouping-operator-static-semantics-early-errors)
> 
> `PrimaryExpression : CPEAAPL`
> 
> It is a Syntax Error if `CPEAAPL` is not covering a `ParenthesizedExpression`.

> [Supplemental Syntax](https://tc39.es/ecma262/#sec-primary-expression)
> 
> When processing an instance of the production
> 
> `PrimaryExpression : CPEAAPL`
> 
> the interpretation of the `CPEAAPL` is refined using the following grammar:
> 
> `ParenthesizedExpression : ( Expression )`

This means: if a `CPEAAPL` occurs in the place of `PrimaryExpression` in the syntax tree, it is actually an `ParenthesizedExpression` and this is its only valid production.

`Expression` can never be empty, so `( )` is not a valid `ParenthesizedExpression`. Comma separated lists like `(1, 2, 3)` are created by [the comma operator](https://tc39.es/ecma262/#sec-comma-operator):

    Expression :  AssignmentExpression  Expression , AssignmentExpression

Similarly, if a `CPEAAPL` occurs in the place of `ArrowParameters`, the following restrictions apply:

> [Static Semantics: Early Errors](https://tc39.es/ecma262/#sec-arrow-function-definitions-static-semantics-early-errors)
> 
> `ArrowParameters : CPEAAPL`
> 
> It is a Syntax Error if `CPEAAPL` is not covering an `ArrowFormalParameters`.

> [Supplemental Syntax](https://tc39.es/ecma262/#sec-arrow-function-definitions)
> 
> When the production
> 
> `ArrowParameters` : `CPEAAPL`
> 
> is recognized the following grammar is used to refine the interpretation of `CPEAAPL`:
> 
> `ArrowFormalParameters :`  
> `( UniqueFormalParameters )`

### Other cover grammars [#](#other-cover-grammars)

In addition to `CPEAAPL`, the spec uses cover grammars for other ambiguous-looking constructs.

`ObjectLiteral` is used as a cover grammar for `ObjectAssignmentPattern` which occurs inside arrow function parameter lists. This means that `ObjectLiteral` allows constructs which cannot occur inside actual object literals.

    ObjectLiteral :  ...  { PropertyDefinitionList }PropertyDefinition :  ...  CoverInitializedNameCoverInitializedName :  IdentifierReference InitializerInitializer :  = AssignmentExpression

For example:

    let o = { a = 1 }; // syntax error// Arrow function with a destructuring parameter with a default// value:let f = ({ a = 1 }) => { return a; };f({}); // returns 1f({a : 6}); // returns 6

Async arrow functions also look ambiguous with a finite lookahead:

    let x = async(a,

Is this a call to a function called `async` or an async arrow function?

    let x1 = async(a, b);let x2 = async();function async() { }let x3 = async(a, b) => {};let x4 = async();

To this end, the grammar defines a cover grammar symbol `CoverCallExpressionAndAsyncArrowHead` which works similarly to `CPEAAPL`.

Summary [#](#summary)
---------------------

In this episode we looked into how the spec defines cover grammars and uses them in cases where we cannot identify the current syntactic construct based on a finite lookahead.

In particular, we looked into distinguishing arrow function parameter lists from parenthesized expressions and how the spec uses a cover grammar for first parsing ambiguous-looking constructs permissively and restricting them with static semantic rules later.