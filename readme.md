# Smart pipelines
ECMAScript Stage-0 Proposal. Living Document. J. S. Choi, 2018-02.

<nav><details>
<summary>

📖 **Table of Contents**

</summary>

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [Motivation](#motivation)
  - [Core Proposal](#core-proposal)
    - [WHATWG Fetch Standard (Core Proposal only)](#whatwg-fetch-standard-core-proposal-only)
    - [jQuery (Core Proposal only)](#jquery-core-proposal-only)
    - [Underscore.js (Core Proposal only)](#underscorejs-core-proposal-only)
    - [Lodash (Core Proposal only)](#lodash-core-proposal-only)
  - [Additional Feature BC](#additional-feature-bc)
  - [Additional Feature BA](#additional-feature-ba)
  - [Additional Feature BP](#additional-feature-bp)
    - [WHATWG Fetch Standard (Core Proposal + Additional Feature BP)](#whatwg-fetch-standard-core-proposal--additional-feature-bp)
    - [jQuery (Core Proposal + Additional Feature BP)](#jquery-core-proposal--additional-feature-bp)
    - [Lodash (Core Proposal + Additional Feature BP)](#lodash-core-proposal--additional-feature-bp)
  - [Additional Feature PP](#additional-feature-pp)
    - [jQuery (Core Proposal + Additional Feature BP+PP)](#jquery-core-proposal--additional-feature-bppp)
    - [Underscore.js (Core Proposal + Additional Feature BP+PP)](#underscorejs-core-proposal--additional-feature-bppp)
  - [Additional Feature TS](#additional-feature-ts)
  - [Additional Feature PF](#additional-feature-pf)
    - [Ramda (Core Proposal + Additional Feature BP+PF)](#ramda-core-proposal--additional-feature-bppf)
    - [WHATWG Streams Standard (Core Proposal + Additional Features BP+PP+PF)](#whatwg-streams-standard-core-proposal--additional-features-bppppf)
  - [Additional Feature NP](#additional-feature-np)
    - [Lodash (Core Proposal + Additional Features BP+PP+PF+NP)](#lodash-core-proposal--additional-features-bppppfnp)
    - [Ramda (Core Proposal + Additional Features BP+PF+NP)](#ramda-core-proposal--additional-features-bppfnp)
    - [WHATWG Streams Standard (Core Proposal + Additional Features BP+PP+PF+NP)](#whatwg-streams-standard-core-proposal--additional-features-bppppfnp)
  - [Additional Feature FS](#additional-feature-fs)
- [Goals](#goals)
  - [“Don’t break my code.”](#dont-break-my-code)
    - [Backward compatibility](#backward-compatibility)
    - [Zero runtime cost](#zero-runtime-cost)
    - [Forward compatibility](#forward-compatibility)
  - [“Don’t shoot me in the foot.”](#dont-shoot-me-in-the-foot)
    - [Opt-in behavior](#opt-in-behavior)
    - [Simple scoping](#simple-scoping)
    - [Static analyzability](#static-analyzability)
  - [“Don’t make me overthink.”](#dont-make-me-overthink)
    - [Syntactic locality](#syntactic-locality)
    - [Semantic clarity](#semantic-clarity)
    - [Expressive versatility](#expressive-versatility)
    - [Cyclomatic simplicity](#cyclomatic-simplicity)
  - [“Make my code easier to read.”](#make-my-code-easier-to-read)
    - [Untangled flow](#untangled-flow)
    - [Distinguishable punctuators](#distinguishable-punctuators)
    - [Terse parentheses](#terse-parentheses)
    - [Terse variables](#terse-variables)
    - [Terse function calls](#terse-function-calls)
    - [Terse composition](#terse-composition)
    - [Terse partial application](#terse-partial-application)
  - [Other Goals](#other-goals)
    - [Conceptual generality](#conceptual-generality)
    - [Human writability](#human-writability)
    - [Novice learnability](#novice-learnability)
- [Relations to other work](#relations-to-other-work)
  - [Pipelines in other programming languages](#pipelines-in-other-programming-languages)
  - [Topic references in other programming languages](#topic-references-in-other-programming-languages)
  - [`do` expressions](#do-expressions)
  - [Function binding](#function-binding)
  - [Function composition](#function-composition)
  - [Partial function application](#partial-function-application)
  - [Optional `catch` binding](#optional-catch-binding)
  - [Pattern matching](#pattern-matching)
  - [Block parameters](#block-parameters)
    - [Topic metaprogramming references](#topic-metaprogramming-references)
  - [`do` expressions](#do-expressions-1)
  - [Private class fields, class decorators, nullish coalescing, and optional chaining](#private-class-fields-class-decorators-nullish-coalescing-and-optional-chaining)
  - [Alternative pipeline Babel plugin](#alternative-pipeline-babel-plugin)
  - [Alternative pipeline proposals](#alternative-pipeline-proposals)
- [Appendices](#appendices)
  - [Smart body syntax](#smart-body-syntax)
    - [Bare style](#bare-style)
    - [Topic style](#topic-style)
    - [Practical consequences](#practical-consequences)
  - [Operator precedence and associativity](#operator-precedence-and-associativity)
  - [Explanation of nomenclature](#explanation-of-nomenclature)
  - [Term rewriting](#term-rewriting)
    - [Core Proposal](#core-proposal-1)
    - [Additional Feature BP](#additional-feature-bp-1)
    - [Additional Feature PP](#additional-feature-pp-1)
    - [Additional Feature NP](#additional-feature-np-1)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->
</details></nav>

***

This document is an **explainer for** the [**formal specification** of a proposed
**smart pipeline operator `|>`**][formal pipeline specification] in
**JavaScript**, along with several other additional features. The specification
is divided into **one Stage-0 Core Proposal** plus **five** mutually
independent-but-compatible **Additional Features**:

|Name                     | Status  | Features                                                               | Purpose                                                                                                         |
| ----------------------- | ------- | ---------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
|[Core Proposal][]        | Stage 0 | Infix pipelines `… \|> …`<br>Lexical topic `#`                         | **Unary** function/expression **application**                                                                   |
|[Additional Feature BC][]| None    | Bare constructor calls `… \|> new …`                                   | Tacit application of **constructors**                                                                           |
|[Additional Feature BA][]| None    | Bare awaited calls `… \|> await …`                                     | Tacit application of **async functions**                                                                        |
|[Additional Feature BP][]| None    | Block pipeline bodies `… \|> {…}`                                      | Application of **statement blocks**                                                                             |
|[Additional Feature PP][]| None    | Prefix pipelines `\|> …`                                               | Tacit application **within blocks**                                                                             |
|[Additional Feature PF][]| None    | Pipeline functions `+>  `                                              | **Partial** function/expression **application**<br>Function/expression **composition**<br>**Method extraction** |
|[Additional Feature TS][]| None    | Pipeline `try` statements                                              | Tacit application to **caught errors**                                                                          |
|[Additional Feature NP][]| None    | N-ary pipelines `(…, …) \|> …`<br>Lexical topics `##`, `###`, and `...`| **N-ary** function/expression **application**                                                                   |
|[Additional Feature FS][]| None    | Pipeline `for` / `for await` statements                                | Tacit application within **iteration loops**                                                                    |

The **Core Proposal** is currently at **Stage 0** of the [TC39 process][TC39
process] and is planned to be presented, along with a [competing
proposal][Pipeline Proposal 1], to TC39 by [Daniel “**littledan**” Ehrenberg of
Igalia][littledan]. The Core Proposal is a **variant** of the [first
pipeline-operator proposal][] also championed by Ehrenberg; this variant is
listed as [**Proposal 4: Smart Mix** in the pipe-proposal wiki][Pipeline
Proposal 4]. The variant resulted from [previous discussions in the previous
pipeline-operator proposal][previous pipeline-placeholder discussions],
discussions which culminated in an [invitation by Ehrenberg to try writing a
specification draft][littledan invitation].

The **additional features** are **not part of the Stage-0 Core Proposal**. They
are included to illustrate possible **separate follow-up proposals** for the case
in which the Core Proposal advances past Stage 1. Together, the Core Proposal
and the additional features demonstrate a **unified vision** of a future in
which composition, partial application, method extraction, and error handling
are all tersely expressible with the same simple pipeline/topic concept.

An [**update** to the existing pipeline **Babel plugin**][Babel plugin] is also
being developed jointly between the author of this proposal and [James
DiGioia][mAAdhaTTah], the author of the [competing proposal][Pipeline
Proposal 1]. The [update will support both this proposal and the other proposal,
configurable with a flag][Babel update summary].

You can take part in discussions on the **[GitHub issue tracker][]**. When you
file an issue, please note in it that you are talking **specifically** about
**[“Proposal 4: Smart Mix”][Pipeline Proposal 4]**.

**This specification uses `#`** as its [topic reference][nomenclature]. However,
this is **not set** in stone. In particular, **`@` or `?`** could also be used.
**Bikeshedding discussions** over what characters to use for the topic token has
been occurring on GitHub at [tc39/proposal-pipeline-operator
issue #91][topic-token bikeshedding].

# Motivation
This section gives a brief overview of the motivations behind the smart pipeline
operator’s Core Proposal, as well the additional features listed above.
**Examples from real-world libraries** are juxtaposed with their original
versions. The original versions have been lightly edited (e.g., breaking up
lines, removing semicolons), in order to fit their horizontal widths into this
table. **Examples that use additional features** are included **only to
illustrate** the power of the pipeline/topic concept and are always simply
**rewritable** into forms that use **only the Core Proposal**.

## Core Proposal
The Core Proposal is [**formally specified in in the draft
specification**][formal CP].

<table>
<thead>
<tr>
<th>With smart pipelines
<th>Status quo

<tbody>
<tr>
<td>

The infix “smart” pipeline operator `|>` proposed here would provide a
**[backwards- and forwards-compatible][don’t break my code]** style of
**chaining nested expressions** into a readable, **left-to-right** manner.

Using a **[zero-cost abstraction][zero runtime cost]**, **nested** data
transformations become [**untangled** into **short steps**][untangled flow].

<td>

```js
console.log(
  await stream.write(
    new User.Message(
      capitalize(
        doubledSay(
          await promise
            || throw new TypeError(
              `Invalid value from ${promise}`)
        ), ', '
      ) + '!'
    )
  )
);
```
**Nested, deeply composed** expressions occur often in JavaScript. They occur
whenever any single value must be processed by a **series of data
transformations**, whether they be **operations, functions, or constructors**.
Unfortunately, these deeply nested expressions often result in **messy
spaghetti** code, due to their mixing of **prefix, infix, and postfix**
expressions together. Writing such code requires many nested **levels of
indentation** and parentheses. Reading such code requires checking **both the
left and right of each subexpression** to understand its data flow.

<tr>
<td>

```js
promise
|> await #
|> # || throw new TypeError(
  `Invalid value from ${promise}`)
|> doubleSay(#, ', ')
|> capitalize
|> # + '!'
|> new User.Message(#)
|> await stream.write(#)
|> console.log;
```
With smart pipelines, the code above becomes **terser** and, literally, more
**straightforward**. Prefix, infix, and postfix expressions would be less
tangled together in threads of spaghetti. Instead, data values would be **piped
from left to right** through a **single flat thread of postfix expressions**,
with a [**single** level of **indentation**][untangled flow] and [**four fewer**
pairs of **parentheses**][terse parentheses]  – essentially forming a [reverse
Polish notation][].

The resulting code’s [**terseness** and **flatness**][make my code easier to
read] may be both easier for the JavaScript developer to **read** and to
**edit**. This uniform postfix notation preserves locality between related code;
the reader may **follow the flow** of data more easily through this [single
flattened thread of postfix operations][untangled flow]. And the developer may
[more easily **add or remove operations**][human writability] at the beginning,
end, or middle of the thread, **without changing** the **indentation** of
unrelated lines.

<td>

```js
console.log(
  await stream.write(
    new User.Message(
      capitalize(
        doubledSay(
          await promise
            || throw new TypeError(
              `Invalid value from ${promise}`)
        ), ', '
      ) + '!'
    )
  )
);
```
Compared with the pipeline version, the original code requires **additional
indentation and grouping** on each step. This requires four more levels of
indentation and four more pairs of parentheses.

In addition, much related code is here separated by unrelated code. Rather than
a **uniform** postfix chain, operations appear **either before** the previous
step’s expression (`await stream.write(…)`,`new User.Message(…)`,
`capitalize(…)`, `doubledSay(…)`, `await …`) but also **after** (`… || throw
new TypeError()`, `… + '!'`). An additional argument to function calls (such as
`, ` in `doubledSay(…, ', ')`) is also separated from its function calls,
forming another easy-to-miss “postfix” argument.


<tr>
<td>

Each postfix expression in a pipeline (called a **[pipeline body][]**) is in its
own **inner lexical scope**, within which a special topic reference `#` is
defined. This `#` is a reference to the **[lexical topic][]** of the pipeline
(`#` itself is a **topic reference**). When the [pipeline’s **head**][pipeline
head] (the expression at its left-hand side) is **evaluated**, it then becomes
the pipeline’s lexical topic. A **new lexical environment** is created, within
which `#` is immutably **bound to the topic**, and with which the pipeline’s
body is then evaluated, using that **topic binding**. In the end, the whole
pipeline expression’s value is the end result into which the pipeline body
evaluated with the topic binding.
```js
input |> (# = 50);
// 🚫 Reference Error:
// Cannot assign to topic reference.
```
The topic binding is immutable, established only once per lexical environment.
It is an error to attempt to assign a value to it using `=`, whether inside or
outside a pipeline body.

<td>

<tr>
<td>

For instance, the chained pipeline:
```js
input
|> # - 3
|> -#
|> # * 2
|> Math.max(#, 0)
|> console.log;
```

<td>

…is equivalent to the tangled nested expression:
```js
console.log(
  Math.max(
    -(input - 3) * 2,
    0
  )
);
```

<tr>
<td>

The syntax is [**statically term rewritable** into already valid code][term
rewriting] in this way, with [theoretically **zero runtime cost**][zero runtime
cost].

Similar use cases appear **numerous times** in JavaScript code, whenever any input
is transformed by **[expressions of any type][expressive versatility]**:
function calls, property calls, method calls, object constructions, arithmetic
operations, logical operations, bitwise operations, `typeof`, `instanceof`,
`await`, `yield` and `yield *`, and `throw` expressions.

<tr>
<td>

```js
promise
|> await #
|> # || throw new TypeError()
|> doubleSay(#, ', ')
|> capitalize
|> # + '!'
|> new User.Message(#)
|> await stream.write(#)
|> console.log;
```
Note that, in the example above, it is **not necessary** to include the
**parenthesized argument (`#`)** for `capitalize` and `console.log`. They were
**tacitly implied**, forming a **tacit unary function call**. In other words,
the example above is equivalent to the version in the next row.

<td>

```js
console.log(
  await stream.write(
    new User.Message(
      capitalize(
        doubledSay(
          await promise
            || throw new TypeError(
              `Invalid value from ${promise}`)
        ), ', '
      ) + '!'
    )
  )
);
```

<tr>
<td>

```js
promise
|> await #
|> # || throw new TypeError(
    `Invalid value from ${#}`)
|> doubleSay(#, ', ')
|> capitalize(#)
|> # + '!'
|> new User.Message(#)
|> await stream.write(#)
|> console.log(#);
```
This version is equivalent to the version above, except that the `capitalize`
and `console.log` pipeline bodies explicitly include optional topic
references `#`, making the expressions slightly wordier than necessary.

<td>

```js
console.log(
  await stream.write(
    new User.Message(
      capitalize(
        doubledSay(
          await promise
            || throw new TypeError(
              `Invalid value from ${promise}`)
        ), ', '
      ) + '!'
    )
  )
);
```

<tr>
<td>

Being able to automatically detect this **“[bare style][]”** is the [**smart**
part of the “smart pipeline operator”][smart body syntax]. The styles of
[**functional** programming][functional programming], [**dataflow**
programming][dataflow programming], and [**tacit** programming][tacit
programming] may particularly benefit from bare pipelines and their [terse
function application][].

<td>

<tr>
<td>

```js
const object = input
|> f
|> # + 2
|> # * 3
|> -#
|> g(#, x)
|> o.unaryMethod
|> await asyncFunction(#)
|> await o.asyncMethod(#)
|> new Constructor(#);
```
This pipeline is a very flat expression, with only one level of indentation, and
with each transformation step on its own line.

Note that `… |> f` is a bare unary function call. This is the same as `… |> f(#)`,
but the topic reference `#` is unnecessary; it is invisibly, tacitly implied. The
same goes for `o.unaryMethod`, which is a unary function call on `o.unaryMethod`.

This is the [**smart** part of the smart pipeline operator][smart body syntax],
which can distinguish between two syntax styles (**[bare style][]** vs. **[topic
style][]**) by using a simple rule: **bare** style uses only **identifiers and
dots** – and **never parentheses, brackets, braces**, or **other operators**. And
**topic** style **always** contains at least one **topic reference**. For more
information, see the reference below about the **[smart body syntax][]**.

<td>

```js
const object =
  new Constructor(
    await o.asyncMethod(
      await asyncFunction(
        o.unaryMethod(
          g(
            -(f(input) + 2)
              * 3,
            x
          )
        )
      )
    )
  );
```
In contrast to the version with pipes, this code is deeply nested, not flat.

The expression has two levels of indentation instead of one.
Reading its data flow requires checking both the beginning and end of each
expression, and each step expression gradually increases in size.

Inserting or removing any step of the data flow also requires changes to the
indentation of any previous steps’ lines.

<tr>
<td>

```js
input |> x + 50 |> f |> g(x, 2);
// 🚫 Syntax Error:
// Topic-style pipeline body
// `|> x + 50`
// binds topic but contains
// no topic reference.
// 🚫 Syntax Error:
// Topic-style pipeline body
// `|> g(x, 2)`
// binds topic but contains
// no topic reference.
```
In order to fulfill the [goal][goals] of [“don’t shoot me in the foot”][],
when a **pipeline is in [topic style][]** but its **body has no topic reference**,
that is an **[early error][]**. Such a degenerate pipeline has a very good
chance of actually being an accidental bug. (Note that the bare-style pipeline
body `|> f` is *not* an error. The [bare style][] is not supposed to contain any
topic references `#`.)

<td>

<tr>
<td>

For instance, this code may be clear enough:
```js
input |> object.method;
```
It is a valid [bare-style pipeline][bare style]. Bare style is designed to be
strictly simple: it must either be a simple reference or it is not in bare style.

<td>

It means:
```js
object.method(input);
```

<tr>
<td>

But this code would be less clear. That is why it is an [early Syntax
Error][early errors]:
```js
input |> object.method();
// 🚫 Syntax Error:
// Topic-style pipeline body
// `|> object.method()`
// binds topic but contains
// no topic reference.
```
It is an invalid [topic-style pipeline][topic style]. It is in topic style
because it is not a simple reference; it has parentheses. And it is invalid
because it is in topic style yet it does not have a topic reference.

<td>

Had that code not been an error, it could reasonably mean either of these lines:
```js
object.method(input);
object.method()(input);
```

<tr>
<td>

Instead, the developer must clarify what they mean, using a topic reference,
into either of these two valid topic-style pipelines:
```js
input |> object.method(#);
input |> object.method()(#);
```
The reading developer benefits from explicitness and clarity, without
sacrificing the benefits of [untangled flow][] that pipelines bring.

<td>

```js
object.method(input);
object.method()(input);
```

<tr>
<td>

Adding other arguments:
```js
input |> object.method(x, y);
// 🚫 Syntax Error:
// Topic-style pipeline body
// `|> object.method(x, y)`
// binds topic but contains
// no topic reference.
```
…would make this problem of semantic ambiguity worse. But the reader is
protected from this ambiguity by the same early error.

<td>

That code could have any of these reasonable interpretations:
```js
object.method(input, x, y);
object.method(x, y, input);
object.method(x, y)(input);
```
Both inserting the input as the first argument and inserting it as the last
argument are reasonable interpretations, as evidenced by how [other programming
languages’ pipeline operators variously do either][topic references in other
programming languages]. Or it could be a factory method that creates a function
that is in turn to be called with a unary input argument.

<tr>
<td>

The writer must clarify which of these reasonable interpretations is correct:
```js
input |> object.method(#, x, y);
input |> object.method(x, y, #);
input |> object.method(x, y)(#);
```

<td>

```js
object.method(input, x, y);
object.method(x, y, input);
object.method(x, y)(input);
```

<tr>
<td>

And this example’s ambiguity would be even worse:
```js
input |> await object.method(x, y);
// 🚫 Syntax Error:
// Topic-style pipeline body
// `|> await object.method(x, y)`
// binds topic but contains
// no topic reference.
```
…were it not an invalid topic-style pipeline.

<td>

It could reasonably mean any of these lines:
```js
await object.method(input, x, y);
await object.method(x, y, input);
await object.method(x, y)(input);
(await object.method(x, y))(input);
```

<tr>
<td>

So the developer must clarify their intent using one of these lines:
```js
input |> await object.method(#, x, y);
input |> await object.method(x, y, #);
input |> await object.method(x, y)(#);
input |> (await object.method(x, y))(#);
```

<td>

```js
await object.method(input, x, y);
await object.method(x, y, input);
await object.method(x, y)(input);
(await object.method(x, y))(input);
```

<tr>
<td>

```js
function doubleSay (str, separator) {
  return `${str}${separator}${string}`;
}

function capitalize (str) {
  return str[0].toUpperCase()
    + str.substring(1);
}

promise
|> await #
|> # || throw new TypeError()
|> doubleSay(#, ', ')
|> capitalize
|> # + '!'
|> new User.Message(#)
|> await stream.write(#)
|> console.log;
```
This pipeline is also relatively flat, with only one level of indentation, and
with each transformation step on its own line.

`… |> capitalize` is a bare unary function call equivalent to `… |> capitalize(#)`.

<td>

```js
function doubleSay (str, separator) {
  return `${str}${separator}${str}`;
}

function capitalize (str) {
  return str[0].toUpperCase()
    + str.substring(1);
}

console.log(
  await stream.write(
    new User.Message(
      capitalize(
        doubledSay(
          await promise
            || throw new TypeError(
              `Invalid value from ${promise}`)
        ), ', '
      ) + '!'
    )
  )
);
```
This deeply nested expression has four levels of indentation instead of two.
Reading its data flow requires checking both the beginning of each expression
(`new User.Message`, `capitalizedString`, `doubledSay`, `await promise` and
end of each expression (`|| throw new TypeError()`, `, ', '`, ` + '!'`)).

<tr>
<td>

```js
x = … |> f(#, #);
```
```js
x = … |> [#, # * 2, # * 3];
```
The topic reference may be used multiple times in a pipeline body. Each use
refers to the same value (wherever the topic reference is not overridden by
another, inner pipeline’s topic scope). Because it is bound to the result of the
topic, the topic is still only ever evaluated once.

<td>

```js
{
  const $ = …;
  x = f($, $);
}
```
```js
{
  const $ = …;
  x = [$, $ * 2, $ * 3];
}
```
This is equivalent to assigning the topic value to a [unique variable][lexically
hygienic], then using that variable multiple times in an expression.

<tr>
<td>

```js
promise
|> await #
|> # || throw new TypeError()
|> `${#}, ${#}`
|> #[0].toUpperCase() + #.substring(1)
|> # + '!'
|> new User.Message(#)
|> stream.write
|> console.log;
```
When tiny functions are only used once, and when their bodies would be obvious and
self-documenting in meaning, then they might be ritual boilerplate that a developer
may prefer to inline: trading off self-documentation for localization of code.

<td>

```js
{
  const promiseValue = await promise
    || throw new TypeError();
  const doubledValue =
    `${promiseValue}, ${promiseValue}`;
  const capitalizedValue
    = doubledValue[0].toUpperCase()
      + doubledValue.substring(1);
  const exclaimedValue
    = capitalizedValue + '!';
  const userMessage =
    new User.Message(exclaimedValue);
  const writeValue =
    stream.write(userMessage);
  console.log(writeValue);
}
```
Using a sequence of variables instead has both advantages and disadvantages. The
variable names may be self-documenting. But they also are verbose. They visually
distract from the crucial data transformations (overemphasizing the expressions’
nouns over their verbs), and it is easy to typo their names.

<tr>
<td>

```js
promise
|> await #
|> # || throw new TypeError()
|> normalize
|> `${#}, ${#}`
|> #[0].toUpperCase() + #.substring(1)
|> # + '!'
|> new User.Message(#)
|> await stream.write(#)
|> console.log;
```
With a pipeline, there are no unnecessary variable identifiers. Inserting a new
step in between two steps (or deleting a step) only touches one new line. Here,
a call of a function `normalize` was inserted between the second and third steps.

<td>

```js
{
  const promiseValue = await promise
    || throw new TypeError();
  const normalizedValue = normalize();
  const doubledValue =
    `${normalizedValue}, ${normalizedValue}`;
  const capitalizedValue =
    doubledValue[0].toUpperCase()
      + doubledValue.substring(1);
  const exclaimedValue =
    capitalizedValue + '!';
  const userMessage =
    new User.Message(exclaimedValue);
  const writeValue =
    stream.write(userMessage);
  console.log(writeValue);
}
```
This code underwent a similar insertion of `normalize`. With a series of
variables, inserting a new step in between two other steps (or deleting a step)
requires editing the variable names in the following step.

<tr>
<td>

```js
input |> f |> [0, 1, 2, ...#] |> g;
```
The body of a pipeline in topic style may contain array literals. These may be
flattened, just like any other sort of expression.

<td>

```js
g([0, 1, 2, ...f(input)]);
```

<tr>
<td>

The body of a pipeline in topic style may also contain object literals. However,
pipeline bodies that are entirely object literals must be parenthesized. It is
similar to how arrow functions distinguish between object literals and blocks.
```js
input |> f |> ({ x: #, y: # }) |> g;
```
This fulfills the goal of [forward compatibility][] with [Additional
Feature BP][], which introduces block pipeline bodies. (It is expected that
block pipelines would eventually be much more common than pipelines with object
literals.)
```js
input |> f |> { x: #, y: # } |> g;
// 🚫 Syntax Error:
// Unexpected token `{`.
// Cannot parse base expression.
```

<td>

```js
{
  const $ = f(input);
  g({ x, $: y: f($) });
}
```

<tr>
<td>

```js
f = input
|> f
|> (x => # + x);
```
The body of a pipeline in topic style may contain an inner arrow function. Both
versions of this example result in an arrow function in a closure on the
previous pipeline’s result `input |> f`.

<td>

```js
{
  const $ = f(input);
  x => $ + x;
}
```
The arrow function lexically closes over the topic value, takes one parameter,
and returns the sum of the topic value and the parameter.

<tr>
<td>

```js
input
|> f
|> settimeout(() => # * 5)
|> processIntervalID;
```
This ability to create arrow functions, which do not lexically shadow the topic,
can be useful for using callbacks in a pipeline.

<td>

```js
{
  const $ = f(input);
  const intervalID = settimeout(() => $ * 5);
  processIntervalID(intervalID);
}
```
The topic value of the second pipeline (here represented by a normal variable
`$`) is still lexically accessible within its body, an arrow function, in both
examples.

<tr>
<td>

```js
input
|> f
|> (() => # * 5)
|> settimeout
|> processIntervalID;
```
The arrow function can also be created on a separate pipeline step.

<td>

```js
{
  const $ = f(input);
  const callback = () => $ * 5;
  const intervalID = settimeout(callback);
  processIntervalID(intervalID);
}
```
The result here is the same.

<tr>
<td>

```js
input
|> f
|> () => # * 5
|> settimeout
|> processIntervalID;
// 🚫 Syntax Error:
// Unexpected token `=>`.
// Cannot parse base expression.
```
Note, however, that arrow functions have looser precedence than the pipe
operator. This means that if a pipeline creates an arrow function alone in one
of its steps’ bodies, then the arrow-function expression must be parenthesized.
(The same applies to assignment and yield operators, which are also looser than
the pipeline operator.) The example above is being parsed as if it were:
```js
(input |> f |> ()) =>
  (# * 5 |> settimeout |> processIntervalID);
// 🚫 Syntax Error:
// Unexpected token `=>`.
// Cannot parse base expression.
```
The arrow function must be parenthesized, as with any other looser-precedence
expression:
```js
input
|> (f, g)
|> (() => # * 5)
|> settimeout
|> processIntervalID;
```

<td>

<tr>
<td>

Both the head and the body of a pipeline may contain nested inner pipelines.
```js
x = input
|> f(x =>
  # + x |> g |> # * 2)
|> #.toString();
```

<td>

A nested pipeline works consistently. It merely shadows the outer context’s
topic with the topic within its own body’s inner context.
```js
{
  const $ = input;
  x = f(x =>
    g($ + x) * 2
  ).toString();
}
```

<tr>
<td>

```js
x = input
|> # ** 2
|> f(x => #
  |> g(#, x)
  |> [# * 3, # * 5]);
```

<td>

```js
{
  const $ = input ** 2;
  x = f(x => {
    const _$ = g($, x);
    return [_$ * 3, _$ * 5];
  });
}
```

<tr>
<td>

**Four kinds of statements cannot** use an **outside context’s topic** in their
expressions. These are:

* `function` definitions (including those for async functions, generators, and
  async generators; but not arrow functions, as explained above),
* `class` definitions,
* `for` and `while` statements (but see [Additional Feature FS][]),
* `catch` clauses (but see [Additional Feature TS][]), and
* `with` statements.

This behavior is in order to fulfill the [goals][] of [simple scoping][] and of
[“don’t shoot me in the foot”][]: it prevents the origin of any topic from being
difficult to find. It also fulfills the goal of [forward compatibility][] with
future [additional features][].

```js
x = input |> function () { return #; };
// 🚫 Syntax Error:
// Lexical context `function () { return #; }`
// contains a topic reference
// but has no topic binding.
// 🚫 Syntax Error:
// Pipeline body `|> function () { … }`
// binds topic but contains
// no topic reference.
```
<td>

<tr>
<td>

```js
x = input |> class { m: () { return #; } };
// 🚫 Syntax Error:
// Pipeline body `|> class { … }`
// binds topic but contains
// no topic reference.
```

<td>

<tr>
<td>

```js
x = input
|> await f(#, 5)
|> () => {
  if (#)
    return # + 30;
  else
    return #;
}
|> g;
```
Any other nested blocks **may** contain topic references from outer lexical
environments. These include **arrow functions, `if` statements**, `try`
statements and their `finally` clauses (though not their `catch` clauses),
`switch` statements, and bare block statements.

<td>

```js
x = g(await f(input, 5) + 30);
```

<tr>
<td>

A function definition that is the body of a pipeline may contain topic
references in its default parameters’ expressions, because their scoping is
similar to that of the outside context’s: similar enough such that also allowing
topic references in them would fulfill the goal of [simple scoping][]. However,
as stated above, the function body itself still may not contain topic references.

```js
value
|> processing
|> function (x = #) { return x; }
```
<td>

```js
function (x = processing(value)) {
  return x;
}
```

<tr>
<td>

The same applies to the parenthesized antecedents of `for` and `while` loops.

```js
input
|> process
|> (x, y) => {
  for (const element of #)
    …
}
```
```js
input
|> process
|> (x, y) => {
  let element;
  while (element = getNextFrom(#))
    …
}
```
<td>

```js
(x, y) => {
  for (const element
    of process(input))
    …
}
```
```js
(x, y) {
  let element;
  while (element =
    getNextFrom(input))
    …
}
```

</table>

### WHATWG Fetch Standard (Core Proposal only)
The [WHATWG Fetch Standard][] contains several examples of using the DOM `fetch`
function, resolving its promises into values, then processing the values in
various ways. These examples may become more easily readable with smart pipelines.

<table>
<thead>
<tr>
<th>With smart pipelines
<th>Status quo

<tbody>
<tr>
<td>

```js
'/music/pk/altes-kamuffel'
|> await fetch(#)
|> await #.blob()
|> playBlob;
```

<td>

```js
fetch('/music/pk/altes-kamuffel')
  .then(res => res.blob())
  .then(playBlob);
```
```js
playBlob(
  await (
    await fetch('/music/pk/altes-kamuffel')
  ).blob()
);
```

<tr>
<td>

```js
'https://example.com/'
|> await fetch(#, { method: 'HEAD' })
|> #.headers.get('content-type')
|> console.log;
```

<td>

```js
fetch('https://example.com/',
  { method: 'HEAD' }
).then(response =>
  console.log(
    response.headers.get('content-type'))
);
```

<tr>
<td>

```js
'https://example.com/'
|> await fetch(#, { method: 'HEAD' })
|> #.headers.get('content-type')
|> console.log;
```

<td>

```js
console.log(
  (await
    fetch('https://example.com/',
      { method: 'HEAD' }
    )
  ).headers.get('content-type')
);
```

<tr>
<td>

```js
'https://example.com/'
|> await fetch(#, { method: 'HEAD' })
|> #.headers.get('content-type')
|> console.log;
```

<td>

```js
{
  const url = 'https://example.com/';
  const response =
    await fetch(url, { method: 'HEAD' });
  const contentType =
    response.headers.get('content-type');
  console.log(contentType);
}
```

<tr>
<td>

```js
'https://pk.example/berlin-calling'
|> await fetch(#, { mode: 'cors' })
|> (
  #.headers.get('content-type')
  |> # && #
    .toLowerCase()
    .indexOf('application/json')
    >= 0
  )
  ? #
  : throw new TypeError()
|> await #.json()
|> processJSON;
```

<td>

```js
fetch('https://pk.example/berlin-calling',
  { mode: 'cors' }
).then(response => {
  if (response.headers.get('content-type')
    && response.headers.get('content-type')
      .toLowerCase()
      .indexOf('application/json') >= 0
  )
    return response.json();
  else
    throw new TypeError();
}).then(processJSON);
```

</table>

### jQuery (Core Proposal only)
As the single most-used JavaScript library in the world, [jQuery][] has provided
an alternative human-ergonomic API to the DOM since 2006. jQuery is under the
stewardship of the [JS Foundation][], a member organization of TC39 through which
jQuery’s developers are represented in TC39. jQuery’s API requires complex data
processing that becomes more readable with smart pipelines.

<table>
<thead>
<tr>
<th>With smart pipelines
<th>Status quo

<tbody>
<tr>
<td>

```js
return data
|> buildFragment([#], context, scripts)
|> #.childNodes
|> jQuery.merge([], #);
```
The path that a reader’s eyes must trace while reading this pipeline moves
straight down, with some movement toward the right then back: from `data` to
`buildFragment` (and its arguments), then `.childNodes`, then `jQuery.merge`.
Here, no one-off-variable assignment is necessary.

<td>

```js
parsed = buildFragment(
  [ data ], context, scripts
);
return jQuery.merge(
  [], parsed.childNodes
);
```
From [jquery/src/core/parseHTML.js][]. In this code, the eyes first must look
for `data` – then upwards to `parsed = buildFragment` (and then back for
`buildFragment`’s other arguments) – then down, searching for the location of
the `parsed` variable in the next statement – then right when noticing its
`.childNodes` postfix – then back upward to `return jQuery.merge`.

<tr>
<td>

```js
(key |> toType) === 'object';
```
```js
key |> toType |> # === 'object';
```
`|>` has a looser precedence than most operators, including `===`. (Only
assignment operators, arrow function `=>`, yield operators, and the comma
operator are any looser.)

<td>

```js
toType(key) === 'object';
```
From [jquery/src/core/access.js][].

<tr>
<td>

```js
context = context
|> # instanceof jQuery
    ? #[0] : #;
```

<td>

```js
context =
  context instanceof jQuery
    ? context[0] : context;
```
From [jquery/src/core/access.js][].

<tr>
<td>

```js
context
|> # && #.nodeType
  ? #.ownerDocument || #
  : document
|> jQuery.parseHTML(match[1], #, true)
|> jQuery.merge;
```

<td>

```js
jQuery.merge(
  this, jQuery.parseHTML(
    match[1],
    context && context.nodeType
      ? context.ownerDocument
        || context
      : document,
    true
  )
);
```
From [jquery/src/core/init.js][].

<tr>
<td>

```js
match
|> context[#]
|> (this[match] |> isFunction)
  ? this[match](#);
  : this.attr(match, #);
```
Note how, in this version, the parallelism between the two clauses is very
clear: they both share the form `match |> context[#] |> something(match, #)`.

<td>

```js
if (isFunction(this[match])) {
  this[match](context[match]);
} else
  this.attr(match, context[match]);
}
```
From [jquery/src/core/init.js][]. Here, the parallelism between the clauses
is somewhat less clear: the common expression `context[match]` is at the end
of both clauses, at a different offset from the margin.

<tr>
<td>

```js
elem = match[2]
|> document.getElementById;
```

<td>

```js
elem = document.getElementById(match[2]);
```
From [jquery/src/core/init.js][].

<tr>
<td>

```js
// Handle HTML strings
if (…)
  …
// Handle $(expr, $(...))
else if (!# || #.jquery)
  return context
  |> # || root
  |> #.find(selector);
// Handle $(expr, context)
else
  return context
  |> this.constructor
  |> #.find(selector);
```
The parallelism between the final two clauses becomes clearer here too.
They both are of the form `return context |> something |> #.find(selector)`.

<td>

```js
// Handle HTML strings
if (…)
  …
// Handle $(expr, $(...))
else if (!context || context.jquery)
  return (context || root).find(selector);
// Handle $(expr, context)
else
  return this.constructor(context)
    .find(selector);
```
From [jquery/src/core/init.js][]. The parallelism is much less clear here.

</table>

### Underscore.js (Core Proposal only)
[Underscore.js][] is another utility library very widely used since 2009,
providing numerous functions that manipulate arrays, objects, and other
functions. It too has a codebase that transforms values through many expressions
– a codebase whose readability would therefore benefit from smart pipelines.

<table>
<thead>
<tr>
<th>With smart pipelines
<th>Status quo

<tbody>
<tr>
<td>

```js
function (obj, pred, context) {
  return obj
  |> isArrayLike
  |> # ? _.findIndex : _.findKey
  |> #(obj, pred, context)
  |> (# !== void 0 && # !== -1)
      ? obj[#] : undefined;
}
```

<td>

```js
function (obj, pred, context) {
  var key;
  if (isArrayLike(obj)) {
    key = _.findIndex(obj, pred, context);
  } else {
    key = _.findKey(obj, pred, context);
  }
  if (key !== void 0 && key !== -1)
    return obj[key];
}
```

<tr>
<td>

```js
function (obj, pred, context) {
  return pred
  |> cb
  |> _.negate
  |> _.filter(obj, #, context);
}
```

<td>

```js
function (obj, pred, context) {
  return _.filter(obj,
    _.negate(cb(pred)),
    context
  );
}
```

<tr>
<td>

```js
function (
  srcFn, boundFn, ctxt, callingCtxt, args
) {
  if (!(callingCtxt instanceof boundFn))
    return srcFn.apply(ctxt, args);
  var self = srcFn
  |> #.prototype |> baseCreate;
  return self
  |> srcFn.apply(#, args)
  |> _.isObject(#) ? # : self;
}
```

<td>

```js
function (
  srcFn, boundFn,
  ctxt, callingCtxt, args
) {
  if (!(callingCtxt instanceof boundFn))
    return srcFn.apply(ctxt, args);
  var self = baseCreate(srcFn.prototype);
  var result = srcFn.apply(self, args);
  if (_.isObject(result)) return result;
  return self;
}
```

<tr>
<td>

```js
function (obj) {
  return obj
  |>  # == null
    ? 0
    : # |> isArrayLike
    ? # |> #.length
    : # |> _.keys |> #.length;
  };
}
```
Smart pipelines make parallelism between all three clauses becomes clearer:\
`0` if it is nullish,\
`# |> #.length` if it is array-like, and\
`# |> something |> #.length` otherwise.\
(Technically, `# |> #.length` could simply be `#.length`, but it is written in
this redundant form in order to emphasis its parallelism with the other branch.)

[This particular example becomes even clearer][Underscore.js + CP + BP + PP]
when paired with [Additional Feature BP][] and [Additional Feature PP][].

<td>

```js
function (obj) {
  if (obj == null) return 0;
  return isArrayLike(obj)
    ? obj.length
    : _.keys(obj).length;
}
```

</table>

### Lodash (Core Proposal only)
[Lodash][] is a fork of [Underscore.js][] that remains under rapid active
development. Along with Underscore.js’ other utility functions, Lodash provides
many other high-order functions that attempt to make [functional programming][]
more ergonomic. Like [jQuery][], Lodash is under the stewardship of the
[JS Foundation][], a member organization of TC39, through which Lodash’s developers
also have TC39 representation. And like jQuery and Underscore.js, Lodash’s API
involves complex data processing that becomes more readable with smart pipelines.

<table>
<thead>
<tr>
<th>With smart pipelines
<th>Status quo

<tbody>
<tr>
<td>

```js
function hashGet (key) {
  return this.__data__
  |> nativeCreate
    ? (#[key] === HASH_UNDEFINED
      ? undefined : #)
    : hashOwnProperty.call(#, key)
    ? #[key]
    : undefined;
}
```

<td>

```js
function hashGet (key) {
  var data = this.__data__;
  if (nativeCreate) {
    var result = data[key];
    return result === HASH_UNDEFINED
      ? undefined : result;
  }
  return hasOwnProperty.call(data, key)
    ? data[key] : undefined;
}
```

<tr>
<td>

```js
function listCacheHas (key) {
  return this.__data__
  |> assocIndexOf(#, key)
  |> # > -1;
}
```

<td>

```js
function listCacheHas (key) {
  return assocIndexOf(this.__data__, key)
    > -1;
}
```

<tr>
<td>

```js
function mapCacheDelete (key) {
  const result = key
  |> getMapData(this, #)
  |> #['delete']
  |> #(key);
  this.size -= result ? 1 : 0;
  return result;
}
```

<td>

```js
function mapCacheDelete (key) {
  var result =
    getMapData(this, key)['delete'](key);
  this.size -= result ? 1 : 0;
  return result;
}
```

<tr>
<td>

```js
function castPath (value, object) {
  return value |>
    # |> isArray
    ? #
    : (# |> isKey(#, object))
    ? [#]
    : # |> toString |> stringToPath;
}
```

<td>

```js
function castPath (value, object) {
  if (isArray(value)) {
    return value;
  }
  return isKey(value, object)
    ? [value]
    : stringToPath(toString(value));
}
```

</table>

## Additional Feature BC
An additional feature – **bare constructor calls** – makes constructor calls
terser. It adds a mode to bare style: if a bare-style pipeline body is preceded
by a `new`, then instead of a function call, it is a constructor call. `value |>
object.Constructor` is equivalent to `object.Constructor(value)`. This is
backwards compatible with the [Core Proposal][] as well as all other [additional
features][].

[Additional Feature BC is **formally specified in in the draft
specification**][formal BC].

<table>
<thead>
<tr>
<th>With smart pipelines
<th>Status quo

<tbody>
<tr>
<td>

```js
value
|> # + '!'
|> new User.Message
|> await stream.write(#)
|> console.log;
```

<td>

```js
console.log(
  await stream.write(
    new User.Message(
      value + '!'
    )
  )
);
```

</table>

## Additional Feature BA
Another additional feature – **bare awaited calls** – makes async function calls
terser. It adds another mode to bare style: if a bare-style pipeline body is
preceded by a `await`, then instead of a mere function call, it is an awaited
function call. `value |> await object.asyncFunction` is equivalent to `await
object.asyncFunction(value)`. This is backwards compatible with the [Core
Proposal][] as well as all other [additional features][].

[Additional Feature BA is **formally specified in in the draft
specification**][formal BC].

<table>
<thead>
<tr>
<th>With smart pipelines
<th>Status quo

<tbody>
<tr>
<td>

```js
value
|> # + '!'
|> new User.Message(#)
|> await stream.write
|> console.log;
```

<td>

```js
console.log(
  await stream.write(
    new User.Message(
      value + '!'
    )
  )
);
```

</table>

## Additional Feature BP
There is a TC39 proposal for [`do` expressions][] at Stage 1. Smart pipelines do
**not** require `do` expressions. However, if [`do` expressions][] also become
part of JavaScript, then, as with **any** other type of expression, a pipeline
in [topic style][] may use a `do` as its body, as long as the `do` expression
contains the topic reference `#`. The topic reference `#` is bound to the
pipeline head’s value, the `do` expression is evaluated, then the result of the
`do` block becomes the final result of that pipeline, and the lexical
environment is reset – all as usual.

In this manner, pipelines with `do` expressions act as a way to create a
“topic-context block”, similarly to [Perl 6’s given block][]. Within this block,
statements may use the topic reference may be used as an abbreviation for the
same value. This can be useful for embedding side effects, `if` `else`
statements, `try` statements, and `switch` statements within pipeline chains.
They may be made even pithier with [Additional Feature BP][], explained later.

[`do` expressions][] as [topic-style][topic style] pipeline bodies might be so
useful, in fact, that it might be worth building them into the pipeline operator
`|>` itself as an add-on feature. This additional feature – **block pipelines**
– adds an additional [topic-style pipeline body syntax][smart body syntax],
using blocks to stand for `do` expressions.

[Additional Feature BP is **formally specified in in the draft
specification**][formal BP].

<table>
<thead>
<tr>
<th>With smart pipelines
<th>Status quo

<tbody>
<tr>
<td>

```js
x = input
|> f
|> { sideEffect(); #; }
|> g;
```
Side effects may easily be embedded within block pipeline bodies.

<td>

```js
const $ = f(input);
sideEffect();
x = g($);
```

<tr>
<td>

```js
x = input
|> f
|> {
  if (typeof # === 'number')
    # + 1;
  else
    { data: # };
}
|> g;
```
`if` `else` statements may also be used within block pipeline bodies, as an
alternative to the ternary conditional operator `?` `:`.

<td>

```js
const _1 = f(input);
let _2;
if (typeof $ === 'number')
  _2 = $ + 1;
else
  _2 = { data: $ };
x = g(_2);
```

<tr>
<td>

```js
x = input
|> f
|> {
  try {
    # |> JSON.parse;
    catch (error) {
      { message: error.message };
    }
  }
}
|> g;
```
`try` statements would also be useful to embed in pipelines with block bodies.
This example becomes even pithier with [Additional Feature PP][] and [Additional
Feature TS][].

<td>

```js
const _1 = f(input);
let _2;
try {
  _2 = JSON.parse(_1);
  catch (error) {
    _2 = { message: error.message };
  }
}
x = g(_2);
```

<tr>
<td>

```js
input
|> await f(#, 5)
|> {
  if (x > 20)
    x + 30;
  else
    x - 10;
}
|> g;
// 🚫 Syntax Error:
// Pipeline body `|> { if (…) … else … }`
// binds topic but contains
// no topic reference.
```
The same [early error rules][] that apply to any topical pipeline body apply
also to topical bodies that are `do` expressions.

<tr>
<td>

As with all other [additional features][], Additional Feature BP is [forward
compatible][] with the [Core Proposal][]. This compatibility includes pipeline
bodies that are object literals, which must be parenthesized.
```js
input |> f |> { x: #, y: # } |> g;
input |> f |> { if (#) { x: #, y: # }; } |> g;
```
Of course, object literals do not have to be parenthesized inside blocks.

<td>

```j

{
  const _1 = f(input);
  let _2;
  if ($) _2 = { x: $, y: $ };
  g(_2);
}
```

</table>

### WHATWG Fetch Standard (Core Proposal + Additional Feature BP)
Revisiting an [example above from the WHATWG Fetch Standard][WHATWG Fetch + CP]
shows how human comprehensibility could be further improved with [Additional
Feature BP][].

<table>
<thead>
<tr>
<th>With smart pipelines
<th>Status quo

<tbody>

<tr>
<td>

```js
'https://pk.example/berlin-calling'
|> await fetch(#, { mode: 'cors' })
|> (
  #.headers.get('content-type')
  |> # && #
    .toLowerCase()
    .indexOf('application/json')
    >= 0
  )
  ? #
  : throw new TypeError()
|> await #.json()
|> processJSON;
```
This pipeline version uses [Core Proposal][] syntax only.

<td>

```js
fetch('https://pk.example/berlin-calling',
  { mode: 'cors' }
).then(response => {
  if (response.headers.get('content-type')
    && response.headers.get('content-type')
      .toLowerCase()
      .indexOf('application/json') >= 0
  )
    return response.json();
  else
    throw new TypeError();
}).then(processJSON);
```

<tr>
<td>

And this pipeline version also uses [Additional Feature BP][]. This allows the
use of an `if` `else` statement instead of a ternary `?` `:` expression.
```js
'https://pk.example/berlin-calling'
|> await fetch(#, { mode: 'cors' })
|> {
  const contentTypeIsJSON =
    #.headers.get('content-type')
    |> # && #
      .toLowerCase()
      .indexOf('application/json')
      >= 0;
  if (contentTypeIsJSON) #;
  else throw new TypeError();
}
|> await #.json()
|> processJSON;
```
It also allows the judicious use of variable/constant assignment where it would
make the meaning of code clearer, rather than [requiring unnecessary variables
redundant with the names of functions][terse variables].

<td>

```js
fetch('https://pk.example/berlin-calling',
  { mode: 'cors' }
).then(response => {
  if (response.headers.get('content-type')
    && response.headers.get('content-type')
      .toLowerCase()
      .indexOf('application/json') >= 0
  )
    return response.json();
  else
    throw new TypeError();
}).then(processJSON);
```

</table>

### jQuery (Core Proposal + Additional Feature BP)
Revisiting the [examples above from jQuery][jQuery + CP] with [Additional
Feature BP][] and [Additional Feature PP][] shows how terseness could be further
improved.

<table>
<thead>
<tr>
<th>With smart pipelines
<th>Status quo

<tbody>
<tr>
<td>

```js
match
|> context[#]
|> (this[match] |> isFunction)
  ? this[match](#);
  : this.attr(match, #);
```
This pipeline version uses [Core Proposal][] syntax.

<td>

```js
if (isFunction(this[match])) {
  this[match](context[match]);
} else
  this.attr(match, context[match]);
}
```
From [jquery/src/core/init.js][].

<tr>
<td>

```js
match
|> context[#]
|> {
  if (this[match] |> isFunction)
    this[match](#);
  else
    this.attr(match, #);
}
```
With [Additional Feature BP][], an `if` `else` statement can be used instead of
a ternary `?` `:` expression.

<td>

```js
if (isFunction(this[match])) {
  this[match](context[match]);
} else
  this.attr(match, context[match]);
}
```
From [jquery/src/core/init.js][].

<tr>
<td>

```js
// Handle HTML strings
if (…)
  …
// Handle $(expr, $(...))
else if (!# || #.jquery)
  return context
  |> # || root
  |> #.find(selector);
// Handle $(expr, context)
else
  return context
  |> this.constructor
  |> #.find(selector);
```
This pipeline version uses [Core Proposal][] syntax only. Note that both
statements are of the form `return context |> something |> #.find(selector)`.

<td>

```js
// Handle HTML strings
if (…) {
  …
// Handle $(expr, $(...))
} else if (!context || context.jquery) {
  return (context || root).find(selector);
// Handle $(expr, context)
} else {
  return this.constructor(context)
    .find(selector);
}
```
From [jquery/src/core/init.js][].

<tr>
<td>

```js
return context
|> {
  // Handle HTML strings
  if (…)
    …
  // Handle $(expr, $(...))
  else if (!# || #.jquery)
    # || root;
  // Handle $(expr, context)
  else
    # |> this.constructor;
}
|> #.find(selector);
```
This pipeline version uses [Additional Feature BP][]. The common phrases `return
context |>` and `|> #.find(selector)` have moved out of the `if` `else if` `else`,
into its own statement. The `if` `else if` `else` itself was moved into a block in
the middle of the new unified pipeline. This emphasizes the unity of the common
path through which content data flow in this code.

<td>

```js
// Handle HTML strings
if (…) {
  …
// Handle $(expr, $(...))
} else if (!context || context.jquery) {
  return (context || root).find(selector);
// Handle $(expr, context)
} else {
  return this.constructor(context)
    .find(selector);
}
```
From [jquery/src/core/init.js][].

<tr>
<td>

```js
return selector |> {
  if (typeof # === 'string')
    …
  else if (# |> isFunction)
    root.ready !== undefined
      ? root.ready(#)
      : #(jQuery);
  else
    jQuery.makeArray(#, this);
};
```
This is a example from jQuery’s codebase on which pipelines would not have been
worth using without [Additional Feature BP][].

<td>

```js
if (typeof selector === 'string') {
  …
} else if (isFunction(selector)) {
  return root.ready !== undefined
    ? root.ready(selector)
    : selector(jQuery);
}
return jQuery.makeArray(selector, this);
```
From [jquery/src/core/access.js][].

</table>

### Lodash (Core Proposal + Additional Feature BP)

<table>
<thead>
<tr>
<th>With smart pipelines
<th>Status quo

<tbody>
<tr>
<td>

```js
function hashGet (key) {
  return this.__data__
  |> nativeCreate
    ? (#[key]
      |> # === HASH_UNDEFINED
      ? undefined : #)
    : hashOwnProperty.call(#, key)
    ? #[key]
    : undefined;
}
```
This pipeline version uses [Core Proposal][] syntax only.

<td>

```js
function hashGet (key) {
  var data = this.__data__;
  if (nativeCreate) {
    var result = data[key];
    return result === HASH_UNDEFINED
      ? undefined : result;
  }
  return hasOwnProperty.call(data, key)
    ? data[key] : undefined;
}
```

<tr>
<td>

```js
function hashGet (key) {
  return this.__data__ |> {
    if (nativeCreate) {
      if (#[key] === HASH_UNDEFINED)
        undefined;
      else
        #;
    } else if (hashOwnProperty.call(#, key))
      #[key];
  };
}
```
This pipeline version also uses [Additional Feature BP][].

<td>

```js
function hashGet (key) {
  var data = this.__data__;
  if (nativeCreate) {
    var result = data[key];
    return result === HASH_UNDEFINED
      ? undefined : result;
  }
  return hasOwnProperty.call(data, key)
    ? data[key] : undefined;
}
```

<tr>
<td>

```js
function mapCacheDelete (key) {
  const result = key
  |> getMapData(this, #)
  |> #['delete']
  |> #(key);
  this.size -= result ? 1 : 0;
  return result;
}
```
This pipeline version uses [Core Proposal][] syntax only.

<td>

```js
function mapCacheDelete (key) {
  var result =
    getMapData(this, key)['delete'](key);
  this.size -= result ? 1 : 0;
  return result;
}
```

<tr>
<td>

```js
function mapCacheDelete (key) {
  return key
  |> getMapData(this, #)
  |> #['delete']
  |> #(key)
  |> {
    this.size -= # ? 1 : 0;
    #;
  };
}
```
This pipeline version also uses [Additional Feature BP][].

<td>

```js
function mapCacheDelete (key) {
  var result =
    getMapData(this, key)['delete'](key);
  this.size -= result ? 1 : 0;
  return result;
}
```

<tr>
<td>

```js
function castPath (value, object) {
  return value |>
    # |> isArray
    ? #
    : (# |> isKey(#, object))
    ? [#]
    : # |> toString |> stringToPath;
}
```
This pipeline version uses [Core Proposal][] syntax only.

<td>

```js
function castPath (value, object) {
  if (isArray(value)) {
    return value;
  }
  return isKey(value, object)
    ? [value]
    : stringToPath(toString(value));
}
```

<tr>
<td>

```js
function castPath (value, object) {
  return value |> {
    if (# |> isArray)
      #;
    else if (# |> isKey(#, object))
      [#];
    else
      # |> toString |> stringToPath;
  };
}
```
This pipeline version also uses [Additional Feature BP][].

<td>

```js
function castPath (value, object) {
  if (isArray(value)) {
    return value;
  }
  return isKey(value, object)
    ? [value]
    : stringToPath(toString(value));
}
```

</table>

## Additional Feature PP
The next additional feature – **Prefix Pipelines** – adds a “headless” tacit
prefix form of the pipeline operator. The tacit, default head is the topic
reference `#` itself, which must be resolvable within the outer lexical
environment.

This feature is especially useful in ternary-conditional `?` `:`
expressions and (with [Additional Feature BP][]) `if` `else` statements, `try`
statements, and `switch` statements.

(If [Additional Feature PF][] or [Additional Feature NP][] are active, then a
pipeline might not have only one pipeline head. In those cases, a prefix
pipeline’s tacit, default head is whatever topic references their pipeline
bodies use. See those other two features for more information.)

[Additional Feature PP is **formally specified in in the draft
specification**][formal PP].

<table>
<thead>
<tr>
<th>With smart pipelines
<th>Status quo

<tbody>
<tr>
<td>

```js
input
|> (# |> predicate)
  ? (# |> f |> # ** 2;)
  : (# |> g |> # ** 3)
|> console.log;
```
In this version, which uses Core Proposal syntax only, several pipelines start
with the phrase `# |>`.

<td>

```js
{
  let temp;
  if (predicate(input))
    temp = f(input) ** 2;
  else
    temp = g(input) ** 3;
  console.log(temp);
}
```
Note that the topic reference in the repeated `# |>` here all refer to the same
topic value – `input` – into `predicate`, `f`, and `g`.

<tr>
<td>

```js
input
|> (|> predicate)
  ? (|> f |> # ** 2;)
  : (|> g |> # ** 3)
|> console.log;
```
In this version, which also uses Additional Feature PP, those pipelines omit the
phrase `# |>`, using a tacit prefix pipeline `|>`, which is implied to use `#`
as the value of their topics.

<td>

```js
{
  let temp;
  if (predicate(input))
    temp = f(input) ** 2;
  else
    temp = g(input) ** 3;
  console.log(temp);
}
```
The prefix pipeline `|>` still piped in the same tacit topic from the same
lexical environment – `x` – into `predicate`, `f`, and `g`. The result is still
the same as before.

<tr>
<td>

```js
x |> {
  if (# |> predicate)
    # |> f |> # ** 2;
  else
    # |> g |> # ** 3;
}
```
In this version, which also uses [Additional Feature BP][] but not Additional
Feature PP, several pipelines also start with the phrase `# |>`.

<td>

```js
if (predicate(x))
  f(x) ** 2;
else
  g(x) ** 3;
```

<tr>
<td>

```js
x |> {
  if (|> predicate)
  |> f |> # ** 2;
  else
  |> g |> # ** 3;
}
```
In this version, which also uses Additional Feature PP, those pipelines omit the
phrase `# |>`, using a tacit prefix pipeline `|>`, which is implied to use `#`
as the value of their topics.

<td>

```js
if (predicate(x))
  f(x) ** 2;
else
  g(x) ** 3;
```
The prefix pipeline `|>` still piped in the same tacit topic from the same
lexical environment – `x` – into `predicate`, `f`, and `g`. The result is still
the same as before.

<tr>
<td>

```js
value
|> f
|> {
  try {
  |> JSON.parse;
    catch (error) {
      { message: error.message };
    }
  }
}
|> g;
```
This example becomes even pithier with [Additional Feature TS][].

<td>

```js
const _1 = f(value);
let _2;
try {
  _2 = JSON.parse(#);
  catch (error) {
    _2 = { message: error.message };
  }
}
g(_2);
```

<tr>
<td>

```js
function () {
|> f |> g;
}
// 🚫 Syntax Error:
// Lexical context `function () { |> f |> g }`
// contains a prefix pipeline `|> f`
// but has no topic binding.
```
If a prefix pipeline is used within a context in which the topic is not
resolvable, then this is an [early error][]. This is just like how it is an
error to use explicit topic references within a context without a topic:

```js
function () {
  # |> f |> g;
}
// 🚫 Syntax Error:
// Lexical context `function () { # |> f |> g }`
// contains a topic reference
// but has no topic binding.
```

<td>

</table>

### jQuery (Core Proposal + Additional Feature BP+PP)

<table>
<thead>
<tr>
<th>With smart pipelines
<th>Status quo

<tbody>
<tr>
<td>

```js
return context |> {
  // Handle HTML strings
  if (…)
    …
  // Handle $(expr, $(...))
  else if (!# || #.jquery)
    # || root;
  // Handle $(expr, context)
  else
    # |> this.constructor;
}
|> #.find(selector);
```
This pipeline version uses [Core Proposal][] syntax plus [Additional Feature BP][].
With [Additional Feature PP][], the `#` in `# |> this.constructor` can be elided.

<td>

```js
// Handle HTML strings
if (…) {
  …
// Handle $(expr, $(...))
} else if (!context || context.jquery) {
  return (context || root).find(selector);
// Handle $(expr, context)
} else {
  return this.constructor(context)
    .find(selector);
}
```
From [jquery/src/core/init.js][].

<tr>
<td>

```js
return context |> {
  // Handle HTML strings
  if (…)
    …
  // Handle $(expr, $(...))
  else if (!# || #.jquery)
    # || root;
  // Handle $(expr, context)
  else
  |> this.constructor;
}
|> #.find(selector);
```
This pipeline version uses [Additional Feature BP][] and [Additional
Feature PP][]. The `#` in `# |> this.constructor` has been elided, but it is
still tacitly there.

<td>

```js
// Handle HTML strings
if (…) {
  …
// Handle $(expr, $(...))
} else if (!context || context.jquery) {
  return (context || root).find(selector);
// Handle $(expr, context)
} else {
  return this.constructor(context)
    .find(selector);
}
```
From [jquery/src/core/init.js][].

<tr>
<td>

```js
return selector |> {
  if (typeof # === 'string')
    …
  else if (# |> isFunction)
    root.ready !== undefined
      ? root.ready(#)
      : #(jQuery);
  else
    jQuery.makeArray(#, this);
};
```
This pipeline version uses [Core Proposal][] syntax plus [Additional Feature BP][].
With [Additional Feature PP][], the `#` in `# |> isFunction` can be elided.

<td>

```js
if (typeof selector === 'string') {
  …
} else if (isFunction(selector)) {
  return root.ready !== undefined
    ? root.ready(selector)
    : selector(jQuery);
}
return jQuery.makeArray(selector, this);
```
From [jquery/src/core/access.js][].

<tr>
<td>

```js
return selector |> {
  if (typeof # === 'string')
    …
  else if (|> isFunction)
    root.ready !== undefined
      ? root.ready(#)
      : #(jQuery);
  else
    jQuery.makeArray(#, this);
};
```
This pipeline version uses [Additional Feature BP][] and [Additional
Feature PP][]. The `#` in `# |> isFunction` has been elided, but it is still
tacitly there.

<td>

```js
if (typeof selector === 'string') {
  …
} else if (isFunction(selector)) {
  return root.ready !== undefined
    ? root.ready(selector)
    : selector(jQuery);
}
return jQuery.makeArray(selector, this);
```
From [jquery/src/core/access.js][].

</table>

### Underscore.js (Core Proposal + Additional Feature BP+PP)
One of the [examples above from Underscore.js][Underscore.js + CP]
with [Additional Feature PP][] improves the visual parallelism of its code.

<table>
<thead>
<tr>
<th>With smart pipelines
<th>Status quo

<tbody>
<tr>
<td>

```js
function (obj) {
  return obj
  |> # == null
    ? 0
    : # |> isArrayLike
    ? # |> #.length
    : # |> _.keys |> #.length;
  };
}
```
Smart pipelines make parallelism between all three clauses becomes clearer. This
pipeline version uses [Core Proposal][] syntax only. Note that several
expressions start with `# |>`.

<td>

```js
function (obj) {
  if (obj == null) return 0;
  return isArrayLike(obj)
    ? obj.length
    : _.keys(obj).length;
}
```

<tr>
<td>

```js
function (obj) {
  return obj
  |>  # == null
    ? 0
    : |> isArrayLike
    ? |> #.length
    : |> _.keys |> #.length;
  };
}
```
By removing `# |>` clutter, [Additional Feature PP][] makes this parallelism even
clearer for the human reader:\
`0` if it is nullish,\
`|> #.length` if it is array-like, and\
`|> something |> #.length` otherwise.\
(Technically, `|> #.length` could simply be `#.length`, but it is written in
this redundant form in order to emphasis its parallelism with the other branch.)

<td>

```js
function (obj) {
  if (obj == null) return 0;
  return isArrayLike(obj)
    ? obj.length
    : _.keys(obj).length;
}
```
The behavior of the code remains the same. All tacit prefix pipelines `|>` here
use the outer environment’s topic: `obj`.

</table>

## Additional Feature TS
With the [Core Proposal][] only, all `try` statements’ `catch` clauses would
prohibit the use of the topic reference within their bodies, except where the
topic reference `#` is inside an inner pipeline inside the `catch` clause: this
is one of the Core Proposal’s [early errors][] mentioned above.

The next additional feature – **Pipeline `try` Statements** – adds new forms of
the `try` statement, the `catch` clause, and the `finally` clause, in the form
of `try |> …`, `catch |> …`, and `finally |> …`, each followed by a [pipeline
body with the same smart body syntax][smart body syntax].

The developer must **[opt into this behavior][opt-in behavior]** by using a
pipeline token `|>`, followed by the pipeline body. No existing code would be
affected. Any, some, or none of the three clauses in a `try` statement may be in
a pipeline form versus the regular block form.

The pipeline `try |> …` statement and the `finally |> …` clause would both apply
the outer context’s topic to their pipeline bodies. As per the usual [smart body
syntax][], if a pipeline body is in bare mode, then it will be called as a
function call, constructor call, or awaited function call on the outer topic.
If the pipeline body is in topic style, then the body is evaluated as an
expression with a new lexical environment, in which the topic reference is bound
to the outer topic.

The pipeline `catch |> …` clause would treat its caught error as if it were the
head of a pipeline whose body is the expression following the `|>`. As per the
usual [smart body syntax][], if the pipeline body is in bare mode, then it will
be called as a function call, constructor call, or awaited function call on
the error. If the pipeline body is in topic style, then the body is evaluated as
an expression with a new lexical environment, in which the topic reference is
bound to the caught error.

With [Additional Feature BP][], this syntax which would naturally allow the form
`catch |> { … }`, except, within the block, the error would be `#`.

In addition, a bare `catch` form, completely lacking a parenthesized antecedent,
has already been proposed as [optional `catch` binding][]. This bare form is
mutually compatible with Additional Feature TS.

[Additional Feature TS is **formally specified in in the draft
specification**][formal TS].

<table>
<thead>
<tr>
<th>With smart pipelines
<th>Status quo

<tbody>
<tr>
<td>

The example below also uses [Additional Feature BP][].

The `try` clause is in the pipeline form using the [topic style][]. It applies
the expression `1 / #` to the outer context’s topic (in this case, `f(value)`).

The `catch` clause is also in the pipeline form using the [bare style][]. It
applies the function `processError` to the caught error.

```js
value
|> f
|> {
  try |> 1 / #;
  catch |> processError;
}
|> g;
```
The semicolons after `1 / #` and after `processError` are optional. There is no
ASI hazard here because pipeline bodies may never contain a `catch` or `finally`
clause, unless the clause is inside a block.

<td>

```js
let _1;
try {
  _1 = 1 / f(value);
}
catch (error) {
  _1 = processError(error);
}
g (_1, 1);
```

<tr>
<td>

```js
value
|> f
|> {
  try |> 1 / #;
  catch |> #.message;
}
|> g(#, 1);
```
Now the `catch` clause is also in topic style, applying apply `console.error` as
a method call to the caught error.

<td>

```js
let _1;
try {
  _1 = 1 / f(value);
}
catch (error) {
  _1 = error.message;
}
g (_1, 1);
```

<tr>
<td>

```js
value
|> f
|> {
  try
  |> 1 / #;
  catch
  |> #.message |> capitalize;
}
|> g(#, 1);
```
Pipeline `try` statements and their clauses may be chained as usual. This
pipeline `catch` clause is in [topic style][] (`|> #.message`) followed by [bare
style][] (`|> capitalize`).

<td>

```js
let _1;
try {
  _1 = 1 / f(value);
}
catch (error) {
  _1 = capitalize(error.message);
}
g (_1, 1);
```

<tr>
<td>

This pipeline `try` statement’s `catch` clause is using the topic-block style
from [Additional Feature BP][], as well as [Additional Feature PP][] to
abbreviate both `# |> 1 / #` and `# |> #.message`.
```js
value
|> f
|> {
  try {
    |> 1 / #;
  }
  catch |> {
    |> #.message
    |> capitalize;
  }
}
|> g;
```
A `|>` between `try` and its block `{ |> 1 / # }` is unnecessary, because the
outer topic does not need to be rebound. However, it is necessary between
`catch` and its block in order to opt into binding the topic reference to the
caught errors.

<td>

```js
let _1;
try {
  _1 = 1 / f(value);
}
catch (error) {
  _1 = capitalize(error.message);
}
g (_1, 1);
```

<tr>
<td>

If the developer includes the parenthesized parameter (like `(error)` in this
example) or if they leave out the `|>` after the `catch`, then no topic binding
is established. As per the [early error rules][] in [Core Proposal][], topic
references are not allowed in regular `catch` blocks.
```js
value
|> f
|> {
  try { 1 / #; }
  catch (error) {
    #.message |> capitalize;
  }
}
|> g(#, 1);
// 🚫 Syntax Error:
// Lexical context `catch { … }`
// contains a topic reference
// but has no topic binding.
```
This sort of [opt-in behavior][] is a goal of this proposal and helps ensure
that the developer does not [shoot themselves in the foot][“don’t shoot me in
the foot”] by accidentally using the topic value from an unexpected outer
environment.

<tr>
<td>

```js
value
|> f
|> {
  try { 1 / #; }
  catch {
    #.message |> capitalize;
  }
}
|> g(#, 1);
// 🚫 Syntax Error:
// Lexical context `catch { … }`
// contains a topic reference
// but has no topic binding.
```
This opt-in behavior is mutually compatible with the proposal for [optional
`catch` binding][].

<td>

<tr>
<td>

```js
value
|> f
|> {
  try |> JSON.parse;
  catch |> { message: #.message };
}
|> g(#, 1);
```

<td>

```js
let _1;
try {
  _1 = 1 / f(value);
}
catch (error) {
  _1 = { message: error.message };
}
g (_1, 1);
```

</table>

## Additional Feature PF
**More than any other** possible extension in this table, this additional
feature – **Pipeline Functions** – would dramatically increase the usefulness of
pipelines. It introduces just one additional operator that solves:\
tacit unary **functional composition**,\
tacit unary functional **partial application**,\
and many kinds of tacit **method extraction**,\
…all at the same time.

And with [Additional Feature NP][], this additional feature would *also* solve\
tacit **N-ary** functional partial application\
and **N-ary** functional composition.

The new operator is a **prefix operator `+> …`**, which creates **pipeline
functions**, which are just **arrow functions**. `+> …` interprets its inner
expression as a **pipeline body** but wraps it in an arrow function that applies
its pipeline body to its arguments.

A pipe function takes **no** a parameter list; no such list is needed.
Just like with regular pipelines, a pipeline function may be in **[bare style][]
or [topic style][]**.

If the pipeline function starts with [bare style][] (like `+> f |> # + 1`), then
the function is **variadic** and applies **all** its arguments to the function
reference to which the bare-style body evaluates (that is,
`(...$) => f(...$) + 1`), where `$` is a [hygienically unique
variable][lexically hygienic]. (This is [forward compatible][] with [Additional
Feature NP][].)

If the pipeline function starts with [topic style][] (like `+> # + 1 |> # + 1`),
then the function is unary and applies its first argument (that is,
`$ => # + 1`, where `$` is a hygienically unique variable).

As an aside, topic style can also handle multiple parameters with [Additional
Feature NP][], such that `+> # + ##` would be a binary arrow function equivalent
to `($, $$) => $ + $$`, and `+> [...].length` would be a variadic arrow function
equivalent to `(...$rest) => [...$rest].length` – where `$`, `$$`, and `$rest`
are all [hygienically unique variables][lexically hygienic].

In general, Additional Feature NP would explain “`+>` _PipelineExpression_” as
equivalent to “`(...$rest) => ...$rest |>` _PipelineExpression_”.

`+>` was chosen because of its similarity both to `|>` and to `=>`. The precise
appearance of the pipeline-function operator does not have to be `+>`. It could
also be `~>`, `->`, `=|`, `=|>` or something else to be decided after future
bikeshedding discussion.

[Additional Feature PF is **formally specified in in the draft
specification**][formal PF].

<table>
<thead>
<tr>
<th>With smart pipelines
<th>Status quo

<tbody>
<tr>
<td>

```js
array.map($ => $ |> #);
```
```js
array.map($ => $);
```
These functions are the same. They both pipe a unary parameter into a
topic-style pipeline whose bodies evaluate simply to the topic, unmodified.

<td>

```js
array.map($ => $);
```
In other words, they are both [identity function][]s.

<tr>
<td>

```js
array.map($ => $ |> # + 2);
```
```js
array.map(+> # + 2);
```
These functions are also the same with each other. They both pipe a unary
parameter into a topic-style pipeline whose bodies are the topic plus two.

<td>

```js
array.map($ => $ + 2);
```

<tr>
<td>

```js
array.map(+> f |> # + 2);
```
This pipeline function starts in **bare mode**. This means it is a variadic function.
(As an aside, with [Additional Feature NP][], this would also be expressible as:
`array.map((...$) => ...$ |> f |> # + 2)`.)

<td>

```js
array.map((...$) => f(...$));
```

<tr>
<td>

Pipelines may be chained within a pipeline function.
```js
array.map(+> f |> g |> h |> # * 2);
```
The prefix pipeline-function operator `+>` has looser precedence than the infix
pipeline operator `|>`.

<td>

```js
array.map((...$) => h(g(f(...$))) * 2);
```

<tr>
<td>

When coupled with [Additional Feature PP][], the phrase `+> |>` (that is, the
prefix pipeline-function operator `+>` immediately followed by the prefix
pipeline operator `|>`) cancels out into simply the prefix pipeline-function
operator `+>`.
```js
array.map(+> f |> g |> h |> # * 2);
array.map(+> |> f |> g |> h |> # * 2);
```
Both of these expressions here are equivalent, both for bare style and topic style,
and with or without [Additional Feature NP][].

<td>

```js
array.map((...$) => h(g(f(...$))) * 2);
```

<tr>
<td>

```js
+> x + 2;
// 🚫 Syntax Error:
// Pipeline body `+> x + 2`
// binds topic but contains
// no topic reference.
```

This is an [early error][], as usual. The topic is not used anywhere
in the pipeline function’s body – just like with `… |> x + 2`.

<td>

<tr>
<td>

```js
=> # + 2;
// 🚫 Syntax Error:
// Unexpected token `=>`.
```
If the pipeline-function operator `+>` is typoed as an arrow function `=>`
instead, then this is another syntax error, because the arrow function `=>`
expects to always have a parameter antecedent as its head.

<td>

<tr>
<td>

```js
() => # + 2;
// 🚫 Syntax Error:
// Lexical context `() => # + 2`
// contains a topic reference
// but has no topic binding.
```
But even if that typo also includes a parameter head for the arrow function
`=>`, this is would still be an [early error][]…unless the outer lexical
environment does have its own topic binding.

<td>

<tr>
<td>

**[Terse composition][]** of unary functions is a goal of smart pipelines. It is
equivalent to piping a value through several function calls, within a unary
function, starting with the outer function’s tacit unary parameter.
```js
array.map(+> f |> g |> h(2, #) |> # + 2);
```
There are [several existing proposals for unary functional composition][function
composition], which Additional Feature PF would all subsume. And with
[Additional Feature NP][], even n-ary functional composition would be supported,
which no current proposal yet addresses.

<td>

```js
array.map((...$) => h(2, g(f(...$))) + 2);
```

<tr>
<td>

```js
const doubleThenSquareThenHalfAsync =
  async $ => $
    |> double |> await squareAsync |> half;
```
```js
const doubleThenSquareThenHalfAsync =
  async +> double |> await squareAsync |> half;
```
When compared to the proposal for [syntactic functional composition by
TheNavigateur][TheNavigateur functional composition], this syntax does not need
to give implicit special treatment to async functions. There is instead an async
version of the pipe-function operator, within which `await` may be used, simply
as usual.

<td>

```js
const doubleThenSquareThenHalfAsync =
  async $ =>
    half(await squareAsync(double($)));
```
```js
const doubleThenSquareThenHalfAsync =
  double +> squareAsync +> half;
```
From the proposal for [syntactic functional composition by
TheNavigateur][TheNavigateur functional composition].

<tr>
<td>

```js
const toSlug =
  $ => $
  |> #.split(' ')
  |> #.map($ => $.toLowerCase())
  |> #.join('-')
  |> encodeURIComponent;
```
This example also uses [Additional Feature PP][] for its second line:
```js
const toSlug = +>
|> #.split(' ')
|> #.map(+> #.toLowerCase())
|> #.join('-')
|> encodeURIComponent;
```
When compared to the proposal for [syntactic functional composition by Isiah
Meadows][isiahmeadows functional composition], this syntax does not need to
surround each non-function expression with an arrow function. The [smart body
syntax][] has more powerful [expressive versatility][], improving the
readability of the code.

<td>

```js
const toSlug = $ =>
  encodeURIComponent(
    $.split(' ')
      .map(str =>
        str.toLowerCase())
      .join('-'));
```
```js
const toSlug =
  _ => _.split(" ")
  :> _ => _.map(str =>
    str.toLowerCase())
  :> _ => _.join("-")
  :> encodeURIComponent;
```
From the proposal for [syntactic functional composition by Isiah
Meadows][isiahmeadows functional composition].

<tr>
<td>

```js
const getTemperatureFromServerInLocalUnits =
  async +>
  |> await getTemperatureKelvinFromServerAsync
  |> convertTemperatureKelvinToLocalUnits;
```
Lifting of non-sync-function expressions into function expressions is
unnecessary for composition with Additional Feature PF. [Additional
Feature BA][] is also useful here.

<td>

```js
Promise.prototype[Symbol.lift] =
  f => x => x.then(f)
const getTemperatureFromServerInLocalUnits =
  getTemperatureKelvinFromServerAsync
  :> convertTemperatureKelvinToLocalUnits;
```
From the proposal for [syntactic functional composition by Isiah
Meadows][isiahmeadows functional composition].

<tr>
<td>

```js
// Functional Building Blocks
const car = +>
|> startMotor
|> useFuel
|> turnKey;
const electricCar = +>
|> startMotor
|> usePower
|> turnKey;

// Control Flow Management
const getData = +>
|> truncate
|> sort
|> filter
|> request;

// Argument Assignment
const sortBy = 'date';
const getData = +>
  |> truncate
  |> sort
  |> #::filter(sortBy)
  |> request;
```
This example also uses [function binding][].

<td>

```js
// Functional Building Blocks
const car = startMotor.compose(
  useFuel, turnKey);
const electricCar = startMotor.compose(
  usePower, turnKey);

// Control Flow Management
const getData = truncate.compose(
  sort, filter, request);

// Argument Assignment
const sortBy = 'date';
const getData = truncate.compose(
  sort,
  $ => filter.bind($, sortBy),
  request);
```
From the proposal for [syntactic functional composition by Simon
Staton][simonstaton functional composition].

<tr>
<td>

```js
const pluck = +> map |> prop;
```

<td>

```js
const pluck = compose(map)(prop);
```
From a [comment about syntactic functional composition by Tom Harding][i-am-tom
functional composition].

<tr>
<td>

**[Terse partial application][] into a unary function** is equivalent to piping
a tacit parameter into a function-call expression, within which the one
parameter is resolvable.
```js
array.map($ => $ |> f(2, #));
array.map(+> f(2, #));
```

<td>

Pipeline functions look similar to the proposal for [partial function
application][] by [Ron Buckton][], except that partial-application expressions
are simply pipeline bodies that are prefixed by the pipeline-function operator.
```js
array.map(f(2, ?));
array.map($ => f(2, $));
```

<tr>
<td>

```js
const addOne = +> add(1, #);
addOne(2); // 3
```

<td>

```js
const addOne = add(1, ?);
addOne(2); // 3
```

<tr>
<td>

```js
const addTen = +> add(#, 10);
addTen(2); // 12
```

<td>

```js
const addTen = add(?, 10);
addTen(2); // 12
```

<tr>
<td>

```js
let newScore = player.score
|> add(7, #)
|> clamp(0, 100, #);
```

<td>

```js
let newScore = player.score
|> add(7, ?)
|> clamp(0, 100, ?);
```

<tr>
<td>

```js
const toSlug = +>
|> encodeURIComponent
|> _.split(#, " ")
|> _.map(#, _.toLower)
|> _.join(#, "-");
```
Additional Feature PF simultaneously handles function composition and
partial application into unary functions.

<td>

```js
const toSlug =
  encodeURIComponent
  :> _.split(?, " ")
  :> _.map(?, _.toLower)
  :> _.join(?, "-");
```
From the proposal for [syntactic functional composition by Isiah
Meadows][isiahmeadows functional composition].

<tr>
<td>

Many kinds of **method extraction** can be addressed by pipeline functions
alone, as a natural result of their pipeline-operator-like semantics.\
`+> console.log` is equivalent to `(...$) => console.log(...$)`…
```js
Promise.resolve(123)
  .then(+> console.log);
```

<td>

```js
Promise.resolve(123)
  .then(console.log.bind(console));
Promise.resolve(123)
  .then(::console.log);
```

<tr>
<td>

```js
$('.some-link').on('click', +> view.reset);
```

<td>

```js
$('.some-link').on('click', ::view.reset);
```

<tr>
<td>

Note that this is *not* the same as `console.log.bind(console.log)`, which
creates an exotic function that always uses whatever value `console.log`
evaluates into – even if `console.log` is reassigned later.
```js
const consoleLog =
  console.log.bind(console.log);
const arrayFrom =
  Array.from.bind(Array.from);
const arrayMap =
  Function.bind.call(Function.call,
    Array.prototype.map);
…
input
|> process
|> consoleLog;
input
|> arrayFrom
|> arrayMap(#, $ => $ + 1)
|> consoleLog;
```
This [robust method extraction][] is a use case that this proposal leaves to
another operator, such as prefix `::` or prefix `&`.

<td>

```js
const consoleLog =
  console.log.bind(console.log);
const arrayFrom =
  Array.from.bind(Array.from);
const arrayMap =
  Function.bind.call(Function.call, Array.prototype.map);
…
consoleLog(
  process(input));
consoleLog(
  arrayMap(arrayFrom(input), $ => $ + 1));
```

<tr>
<td>

```js
…
input
|> process
|> &console.log;
input
|> &Array.from
|> #::&Array.prototype.map($ => $ + 1)
|> &console.log;
```
Pipeline functions would not preclude adding another operator that addresses
[robust method extraction][] with [inline caching][method-extraction inline
caching], such as the hypothetical prefix `&` operators (for cached method
extraction) and infix `::` operators (for `this` binding) shown here. Such
hypothetical notations could even be eventually accommodated by a new [bare
style][] notation, shown here with `… |> &console.log` and `… |> &Array.from`.

<td>

```js
…
consoleLog(
  process(input));
consoleLog(
  &Array.from(input)
  ::&Array.prototype.map($ => $ + 1));
```

<tr>
<td>

```js
const { hasOwnProperty } =
  Object.prototype;
const x = { key: 5 };
x::hasOwnProperty;
x::hasOwnProperty('key');
```
For terse **method calling/binding**, the infix `::` operator would also still
be required.

<td>

```js
const { hasOwnProperty } =
  Object.prototype;
const x = { key: 5 };
x::hasOwnProperty;
x::hasOwnProperty('key');
```

</table>

### Ramda (Core Proposal + Additional Feature BP+PF)
[Ramda][] is a utility library focused on [functional programming][] with [pure
functions][] and [immutable objects][]. Its functions are automatically
[curried][currying]. Smart pipelines with [Additional Feature PF][]
would address many of Rambda’s use cases. The examples below were taken from the [Ramda wiki
cookbook][]. They use smart pipelines with vanilla JavaScript APIs when possible
(such as `Array.prototype.map` instead of `R.map`), but they also use Ramda
functions wherever no terse JavaScript equivalent yet exists (such as with
`R.zipWith` and `R.adjust`).

[Even more of Ramda’s use cases are covered][Ramda + CP + BP + PF + NP] when
[Additional Feature NP][] syntax is supported.

<table>
<thead>
<tr>
<th>With smart pipelines
<th>Status quo

<tbody>
<tr>
<td>

```js
const pickIndexes = +> R.values |> R.pickAll;
['a', 'b', 'c'] |> pickIndexes([0, 2], #);
// ['a', 'c']
```

<td>

```js
const pickIndexes = R.compose(
  R.values, R.pickAll);
pickIndexes([0, 2], ['a', 'b', 'c']);
// ['a', 'c']
```

<tr>
<td>

```js
const list = +> [...];
list(1, 2, 3);
// [1, 2, 3]
```

<td>

```js
const list = R.unapply(R.identity);
list(1, 2, 3);
// [1, 2, 3]
```

<tr>
<td>

```js
const getNewTitles = async +>
|> await fetch
|> parseJSON
|> #.flatten()
|> #.map(+> #.items)
|> #.map(+> #.filter(+> #))
|> #.map(+> #.title);

try {
  '/products.json'
  |> getNewTitles
  |> console.log;
}
catch
|> console.error;

const fetchDependent = async +>
|> await fetch
|> JSON.parse
|> #.flatten()
|> #.map(+> #.url)
|> #.map(fetch)
|> #.flatten();

try {
  'urls.json'
  |> fetchDependent
  |> console.log;
}
catch
|> console.error;
```
This example also uses [Additional Feature TS][] for terse `catch` clauses
and [Additional Feature BA][] for terse awaited function calls.

<td>

```js
const getNewTitles = R.compose(
  R.map(R.pluck('title')),
  R.map(R.filter(R.prop('new'))),
  R.pluck('items'),
  R.chain(JSON.parse),
  fetch
);

getNewTitles('/products.json')
  .fork(console.error, console.log);

const fetchDependent = R.compose(
  R.chain(fetch),
  R.pluck('url'),
  R.chain(parseJSON),
  fetch
);

fetchDependent('urls.json')
  .fork(console.error, console.log);
```

<tr>
<td>

```js
number
|> R.repeat(Math.random, #)
|> #.map(+> #());
```

<td>

```js
R.map(R.call,
  R.repeat(Math.random, number));
```

<tr>
<td>

```js
const renameBy = (fn, obj) =>
  [...obj]
  |> #.map(R.adjust(fn, 0)),
  |> ({...#});
{ A: 1, B: 2, C: 3 };
|> renameBy(+> `a${#}`));
// { aA: 1, aB: 2, aC: 3 }
```

<td>

```js
const renameBy = R.curry((fn, obj) =>
  R.pipe(
    R.toPairs,
    R.map(R.adjust(fn, 0)),
    R.fromPairs
  )(obj)
);
renameBy(R.concat('a'), { A: 1, B: 2, C: 3 });
// { aA: 1, aB: 2, aC: 3 }
```

</table>

### WHATWG Streams Standard (Core Proposal + Additional Features BP+PP+PF)
The [WHATWG Streams Standard][] provides an efficient, standardized stream API,
inspired by Node.js’s Streams API, but also applicable to the DOM. The
specification contains numerous usage examples that would become more readable
with smart pipelines. The Core Proposal alone would untangle much of this code,
and the additional features would further improve its terseness.

<table>
<thead>
<tr>
<th>With smart pipelines
<th>Status quo

<tbody>
<tr>
<td>

```js
class LipFuzzTransformer {
  constructor(substitutions) {
    this.substitutions = substitutions;
    this.partialChunk = "";
    this.lastIndex = undefined;
  }

  transform (chunk, controller) {
    this.partialChunk = ""
    this.lastIndex = 0
    const partialAtEndRegexp =
      /\{(\{([a-zA-Z0-9_-]+(\})?)?)?$/g
    partialAtEndRegexp.lastIndex =
      this.lastIndex
    this.lastIndex = undefined
    chunk
    |> this.partialChunk + #
    |> #.replace(
        /\{\{([a-zA-Z0-9_-]+)\}\}/g,
        +> this.replaceTag)
    |> partialAtEndRegexp.exec
    |> {
      if (#) {
        this.partialChunk =
        |> #.index
        |> chunk.substring;
        #
        |> #.index
        |> chunk.substring(0, #);
      }
      else
        chunk;
    }
    |> controller.enqueue;
  }

  flush (controller) {
    this.partialChunk |> {
      if (#.length > 0) {
      |> controller.enqueue;
      }
    };
  }

  replaceTag (match, p1, offset) {
    return this.substitutions
    |> #[p1]
    |> # === undefined ? '' : #
    |> {
        this.lastIndex =
        |> #.length
        |> offset + #;
        #;
    };
  }
}
```

<td>

```js
class LipFuzzTransformer {
  constructor (substitutions) {
    this.substitutions = substitutions;
    this.partialChunk = "";
    this.lastIndex = undefined;
  }

  transform (chunk, controller) {
    chunk = this.partialChunk + chunk;
    this.partialChunk = "";
    this.lastIndex = 0;
    chunk = chunk.replace(
      /\{\{([a-zA-Z0-9_-]+)\}\}/g,
      this.replaceTag.bind(this));
    const partialAtEndRegexp =
      /\{(\{([a-zA-Z0-9_-]+(\})?)?)?$/g;
    partialAtEndRegexp.lastIndex =
      this.lastIndex;
    this.lastIndex = undefined;
    const match =
      partialAtEndRegexp.exec(chunk);
    if (match) {
      this.partialChunk =
        chunk.substring(match.index);
      chunk =
        chunk.substring(0, match.index);
    }
    controller.enqueue(chunk);
  }

  flush (controller) {
    if (this.partialChunk.length > 0) {
      controller.enqueue(
        this.partialChunk);
    }
  }

  replaceTag (match, p1, offset) {
    let replacement = this.substitutions[p1];
    if (replacement === undefined) {
      replacement = "";
    }
    this.lastIndex =
      offset + replacement.length;
    return replacement;
  }
}
```

</table>

## Additional Feature NP
Another Additional Feature – **n-ary pipelines** – enables the passing of
multiple arguments from each pipeline’s head into its body. `(a, b) |> f` is
equivalent to `f(a, b)`.

For [topic style][], Additional Feature NP introduces **multiple lexical
topics**: not only the **primary** topic reference `#`, but also **secondary**
`##`, **tertiary** `###`, and **rest** `...` **topic references**. It also
enables both **n-ary application** and **n-ary partial application**.
This is somewhat akin to [Clojure’s compact anonymous functions][Clojure compact
function], which use `%` aka `%1`, then `%2`, `%3`, … for its parameters within
the compact functions’ bodies.

When combined with [Additional Feature PF][], Additional Feature NP would
complete the subsumption of partial function application, addressing all its use
cases, including using partial application to create new binary, trinary, and
variadic functions.

This explainer limits this Additional Feature to three topic references plus a
rest topic reference. This limit could theoretically be lifted, but readability
would rapidly suffer with five, six, seven different topics at once. Arrow
functions could always be used instead for such many-parameter functions.\
The precise appearances of the secondary, tertiary, and rest topic references do
not have to be `##`, `###`, and `...`. For instance, they could instead be `#1`,
`#2`, and `#...`; this is yet to be bikeshedded.

[Additional Feature NP is **formally specified in in the draft
specification**][formal NP].

<table>
<thead>
<tr>
<th>With smart pipelines
<th>Status quo

<tbody>
<tr>
<td>

```js
(a, b) |> f;
```
Pipeline heads using commas would be interpreted as argument lists, which would
then be applied to the pipeline bodies.

<td>

```js
f(a, b);
```

<tr>
<td>

```js
(a, b, ...c, d) |> f |> g;
```
Spread elements are permitted within pipeline heads, with the same meaning as in
regular argument lists.

<td>

```js
g(f(a, b, ...c, d));
```

<tr>
<td>

```js
...a |> f |> g;
```
When a pipeline head only consists of one item, its parentheses may be omitted,
which is the usual syntax from the [Core Proposal][]. But this now goes for
spread elements too.

<td>

```js
g(f(...a));
```

<tr>
<td>

```js
(a, b) |> f(#, x, ##) |> g;
```
When a pipeline’s body is in [topic style][], the first element in the argument
list is bound to the primary topic reference `#`, the second element is bound to
the secondary topic reference `##`, and the third element is bound to the
tertiary topic reference `###`. These are resolvable as usual within the
pipeline body.

<td>

```js
g(f(a, x, b));
```

<tr>
<td>

```js
(a, b, ...c, d) |> f(#, x, ...) |> g;
```
The pipeline also binds an array to a rest topic reference `...` within the
pipeline body. The array contains the arguments of the pipeline head that were
not bound to any other topic reference.

<td>

```js
g(f(a, x, ...[b, ...c, d]));
```

<tr>
<td>

```js
(a, b, c, d, e) |> f(##, x, ...) |> g;
```
The rest topic reference `...` starts from beyond the furthest topic reference
that is used within the pipeline body. Here, the furthest topic reference is the
secondary topic reference `##`: the second argument item. So `[c, d, e]` is
bound to the rest topic reference. The rest topic reference `...` may only be
used where the spread operator `...expression` would also be valid (that is,
argument lists, array literals, and object literals), and it automatically
spreads its elements into whatever expression surrounds it.

<td>

```js
{
  const [_primary, _secondary, _tertiary, ..._rest]
    = [a, b, c, d, e];
  g(f(a, _secondary, x, ..._rest));
}
```

<tr>
<td>

```js
(a, b, c, ...d, e) |> f(#, ###, x, ...) |> g;
```
Here, the furthest topic reference is the tertiary topic reference `###`: the
third argument item. So only the rest topic reference `...` contains `d`’s
spread elements as well as `e`. The second argument, `b`, is skipped entirely,
because `##` is not used at all in the pipeline body.

<td>

```js
{
  const _rest = [...d, e];
  g(f(a, _tertiary, x, ..._rest));
}
```

<tr>
<td>

```js
(a, ...b, c, ...d, e)
|> f(#, ##, ###, x, ...)
|> g;
```

<td>

```js
{
  const [_secondary, _tertiary, ..._rest] =
    [...b, c, ...d, e];
  g(f(a, _secondary, _tertiary, x, ..._rest));
}
```

<tr>
<td>

```js
(a, ...b, c, ...d, e)
|> f(#, ##, x, ...)
|> g;
```

<td>

```js
{
  const [_secondary, ..._rest] =
    [...b, c, ...d, e];
  g(f(a, _secondary, x, ..._rest));
}
```

<tr>
<td>

```js
(a, b) |> # - ## |> g;
```

<td>

```js
g(a - b);
```

<tr>
<td>

N-ary pipelines may be chained by using comma expressions to make their pipeline
bodies also n-ary.
```js
(a, b) |> (f, g) |> h;
```
Each element in an N-ary pipeline body is independently applied to each
consecutive argument from the pipeline head.

<td>

```js
h(f(a), g(b));
```

<tr>
<td>

```js
(a, b)
|> (f, # ** c + ##)
|> # - ##;
```
The elements in an N-ary pipeline body may be either in bare style (like the `f`
here) or in topic style (like the `# ** c + ##` here).

<td>

```js
f(a) - (a ** c + b);
```

<tr>
<td>

```js
(a, b)
|> (f, g)
|> h
|> (i, # + 1, k)
|> l;
```


<td>

```js
{
  const $ = h(f(a), g(b));
  l(i($), $ + 1, k($));
}
```

<tr>
<td>

```js
(a, b)
|> (f, g)
|> (h, i);
// 🚫 Syntax Error:
// A pipeline chain terminates
// with a 2-ary pipeline body
// but must terminate with a
// unary pipeline body.
```
It is an [early error][] for a pipeline head to end with an n-ary pipeline body,
where n > 1. Such a comma expression would almost certainly be an accidental
mistake by the developer.

<td>

<tr>
<td>

```js
value
|> (f, g)
|> (x, y) => # * x + ## * y
|> settimeout
// 🚫 Syntax Error:
// Unexpected token `=>`.
// Cannot parse base expression.
```
Because arrow functions have looser precedence than the pipeline operator `|>`,
it is never ambiguous with the parenthesized-list syntax for N-ary pipelines.
The above invalid code is being interpreted as if it were the below:
```js
(value |> (f, g) |> (x, y)) =>
  (# * 5 |> settimeout);
// 🚫 Syntax Error:
// Unexpected token `=>`.
// Cannot parse base expression.
```
The arrow function must be parenthesized, simply as with any other
looser-precedence expression:
```js
value
|> (f, g)
|> ((x, y) => # * x + ## * y)
|> settimeout;
```

<td>

<tr>
<td>

```js
number
|> ...createRange
|> [#, ###, ...];
```

<td>

```js
{
  const [_primary, , _tertiary, ..._rest] =
    createRange(number);
  [_primary, _tertiary, _rest];
}
```

<tr>
<td>

```js
input |> f |> [0, 1, 2, ...#] |> g;
```
```js
input |> f |> ...# |> [0, 1, 2, ...] |> g;
```
This is an adapted example from the [Core Proposal section above][Core
Proposal]. It is equivalent to the original example; it is shown only for
illustrative purposes.

<td>

```js
g([0, 1, 2, ...f(input)]);
```
```js
{
  const [..._rest] = f(input);
  g([0, 1, 2, ..._rest]);
}
```
All these code blocks are equivalent.

<tr>
<td>

```js
x
|> (f, ...g, h)
|> [...].length;
```
As a result of the rules, `… |> [...]` collects its pipeline head’s n-ary
arguments into a single flattened array, to which the rest topic reference `...`
is then bound.

<td>

```js
[f(x), ...g(x), h(x)].length;
```

<tr>
<td>

```js
[ { x: 22 }, { x: 42 } ]
  .map(+> #.x)
  .reduce(+> # - ##, 0);
```

<td>

```js
[ { x: 22 }, { x: 42 } ]
  .map(el => el.x)
  .reduce((_0, _1) => _0 - _1, 0);
```

<tr>
<td>

```js
array.sort(+> # - ##);
```
Additional Feature NP, when coupled with [Additional Feature PF][], would
enable very terse callback functions.

<td>

```js
array.sort((_0, _1) => _0 - _1);
```

<tr>
<td>

```js
[ { x: 22 }, { x: 42 } ]
  .map(+> #.x)
  .reduce(+> # - ##, 0);
```

<td>

```js
[ { x: 22 }, { x: 42 } ]
  .map(el => el.x)
  .reduce((_0, _1) => _0 - _1, 0);
```

<tr>
<td>

```js
const f = (x, y, z) => [x, y, z];
const g = +> f(#, 4, ##);
g(1, 2); // [1, 4, 2]
```
Additional Feature NP, when coupled with [Additional Feature PF][], would also
solve **partial application into n-ary functions**. (Additional Feature PF would
only address partial application into unary functions.)

<td>

```js
const f = (x, y, z) => [x, y, z];
const g = f(?, 4, ?);
g(1, 2); // [1, 4, 2]
```
The current proposal for [partial function application][] assumes that each use
of the same `?` placeholder token represents a different parameter. In contrast,
each use of `#` within the same scope always refers to the same value. This is
why additional topic parameters are required.

<tr>
<td>

The resulting model is more flexible: with Additional Feature NP with
[Additional Feature PF][], `+> f(#, 4, ##)` is different from `+> f(#, 4, #)`.
The former refers to a **binary** function: a function with two parameters,
essentially `(x, y) => f(x, 4, y)`. The latter refers to a **unary** function
that passes the same one argument into both the first and third parameters of
the original function `f`: `x => f(x, 4, x)`. The same symbol refers to the same
value in the same lexical environment.

<tr>
<td>

```js
const maxGreaterThanZero =
  +> Math.max(0, ...);
maxGreaterThanZero(1, 2); // 2
maxGreaterThanZero(-1, -2); // 0
```
Partial application into a variadic function is also naturally handled by
Additional Feature NP with [Additional Feature PF][].

<td>

```js
const maxGreaterThanZero =
  Math.max(0, ...);
maxGreaterThanZero(1, 2); // 2
maxGreaterThanZero(-1, -2); // 0
```
In this case, the topic function version looks once again nearly identical to
the other proposal’s code.

<tr>
<td>

Additional Feature NP would explain bare style in [Additional Feature PF][]
“`+>` _PipelineExpression_” as equivalent to a function with a variadic pipeline
“`(...$rest) => ...$rest |>` _PipelineExpression_”.
```js
+> g |> f |> # + 1;
(...$rest) => ...$rest |> g |> f |> # + 1;
```
These two lines of code are equivalent. The first is taken from an example in
the [Additional Feature PF section above][Additional Feature PF].

<td>

```js
(...$rest) =>
  f([...$].length) + 1;
```

<tr>
<td>

```js
+> [...].length |> f |> # + 1;
(...$rest) =>
  ...$rest |> [...].length |> f |> # + 1;
```
These two lines of code are also equivalent.

<td>

```js
(...$rest) => f([...$].length) + 1;
```

<tr>
<td colspan=2>

When coupled with Additional Feature NP, the prefix pipeline operator `|>` of
[Additional Feature PP][], would be equivalent to a pipeline whose head is
simply the argument list of all the topic references that its body uses. The
code statements in each of the table rows below are mutually equivalent.

<tr>
<td>

```js
// Unary topic style.
input
|> # && ([#] |> f |> #+1)
|> console.log;
input
|> # && (|> [#] |> f |> #+1)
|> console.log;
input
|> # && ((#) |> [#] |> f |> #+1)
|> console.log;
```

<td>

```js
// Unary topic style.
{
  const $0 = input;
  let $1;
  if ($0) {
    const $2 = [$0];
    const $3 = f($2);
    const $4 = $3 + 1;
    $1 = $4;
  }
  console.log($1);
}
```

<tr>
<td>

```js
// Binary topic style.
inputs
|> # && ([#,##] |> f |> #+1)
|> console.log;
inputs
|> # && (|> [#,##] |> f |> #+1)
|> console.log;
inputs
|> # && ((#,##)|> [#,##] |> f |> #+1)
|> console.log;
```

<td>

```js
// Binary topic style.
{
  const [$0, $$0] = inputs;
  let $1;
  if ($0) {
    const $2 = [$0, $$0];
    const $3 = f($2);
    const $4 = $3 + 1;
    $1 = $4;
  }
  console.log($1);
}
```

<tr>
<td>

```js
// Variadic topic style (1 positional topic).
inputs
|> # && ([#,...] |> f |> #+1)
|> console.log;
inputs
|> # && (|> [#,...] |> f |> #+1)
|> console.log;
inputs
|> # && ((#,...) |> [#,...] |> f |> #+1)
|> console.log;
```

<td>

```js
// Variadic topic style (1 positional topic).
{
  const [$0, ...$r0] = inputs;
  let $1;
  if ($0) {
    const $2 = [$0, ...$r0];
    const $3 = f($2);
    const $4 = $3 + 1;
    $1 = $4;
  }
  console.log($1);
}
```

<tr>
<td>

```js
// Variadic bare style.
inputs
|> true && ([...] |> f |> #+1)
|> console.log;
inputs
|> true && (|> [...] |> f |> #+1)
|> console.log;
inputs
|> true && (... |> [...] |> f |> #+1)
|> console.log;
```

<td>

```js
{
  const [...$r0] = inputs;
  let $1;
  if ($0) {
    const $2 = [...$r0];
    const $3 = f($2);
    const $4 = $3 + 1;
    $1 = $4;
  }
  console.log($1);
}
```

<tr>
<td colspan=2>

Additional Feature NP would also explain how the cancellation of [Additional
Feature PP][] by [Additional Feature PF][] (“`+> |>` _PipelineExpression_” is
equivalent to “`+>` _PipelineExpression_”). Each of the following table rows’
code statements are equivalent.

<tr>
<td>

```js
// Unary topic style.
+> [#] |> f |> #+1;
($) =>
  $ |> [#] |> f |> #+1;
+> |> [#] |> f |> #+1;
($) =>
  $ |> # |> [#] |> f |> #+1;
```

<td>

```js
// Unary topic style.
($0) => {
  const $1 = $0;
  return f([$1]) + 1;
};
($0) => {
  const $1 = $0,
        $2 = $1;
  return f([$2]) + 1;
};
```

<tr>
<td>

```js
// Binary topic style.
+> [#,##] |> f |> #+1;
($,$$) =>
  ($,$$) |> [#,##] |> f |> #+1;
+> |> [#,##] |> f |> #+1;
($,$$) =>
  ($,$$) |> (#,##) |> [#,##] |> f |> #+1;
```

<td>

```js
// Binary topic style.
($0, $$0) => {
  const $1 = [$0, $$0],
        $2 = f($1);
  return $2 + 1;
};
($0, $$0) => {
  const [$1, $$1] = [$0, $$0],
        $2 = [$1, $$1],
        $3 = f($1);
  return $3 + 1;
};
```

<tr>
<td>

```js
// Variadic topic style (1 positional topic).
+> [#,...] |> f |> #+1;
($,...$r) =>
  ($,...$r) |> [#,...] |> f |> #+1;
+> |> [#,...] |> f |> #+1;
($,...$r) =>
  ($,...$r) |> (#,...) |> [#,...] |> f |> #+1;
```

<td>

```js
// Variadic topic style (1 positional topic).
($0, ...$r) => {
  const $1 = [$0, ...$0],
        $2 = f($1);
  return $2 + 1;
};
($0, ...$r) => {
  const $1 = [$0, ...$0],
        $2 = [$1, ...$1],
        $3 = f($2);
  return $3 + 1;
};
```

<tr>
<td>

```js
// Variadic bare style.
+> f;
(...$rest) =>
  ...$rest |> f |> #+1;
+> |> f;
(...$rest) =>
  ...$rest |> ... |> f |> #+1;
```

<td>

```js
(...$r0) => {
  const $1 = [...$r0],
        $2 = f($1);
  return $2 + 1;
};
(...$r0) => {
  const [...$r1] = [...$r0],
        $2 = [...$r1],
        $3 = f($1);
  return $3 + 1;
};
```

</table>

### Lodash (Core Proposal + Additional Features BP+PP+PF+NP)

<table>
<thead>
<tr>
<th>With smart pipelines
<th>Status quo

<tbody>
<tr>
<td>

```js
function createRound (methodName) {
  var func = Math[methodName];
  return function (number, precision) {
    number = number |> toNumber;
    precision = precision |> {
      if (# == null)
        0;
      else
      |> toInteger |> nativeMin(#, 292);
    }
    return number |> {
      if (precision)
        // Shift with exponential notation
        // to avoid floating-point issues.
        // See https://mdn.io/round#Examples.
      |> `${#}e`
      |> ...#.split('e')
      |> `${#}e${+## + precision}`
      |> func
      |> `${#}e`
      |> ...#.split('e')
      |> `${#}e${+## - precision}`
      |> +#;
      else
      |> func;
    };
  };
}
```
The parallelism between the `if` clause’s `|> shift |> func |> shiftBack` and
the `else` clause’s `|> func` becomes visually clearer with smart pipelines.

<td>

```js
function createRound (methodName) {
  var func = Math[methodName];
  return function (number, precision) {
    number = toNumber(number)
    precision = precision == null
      ? 0
      : nativeMin(toInteger(precision), 292)
    if (precision) {
      // Shift with exponential notation
      // to avoid floating-point issues.
      // See https://mdn.io/round#Examples.
      var
        pair = (toString(number) + 'e')
          .split('e'),
        value = func(
          pair[0] + 'e' + (
            +pair[1] + precision));

      pair = (toString(value) + 'e')
        .split('e');
      return +(
        pair[0] + 'e' + (
          +pair[1] - precision));
    }
    return func(number);
  }
}
```

</table>

### Ramda (Core Proposal + Additional Features BP+PF+NP)
[Many examples above using Ramda][Ramda + CP + BP + PF] benefited from pipeline
functions with Additional Feature PF. Even more use cases are covered by
pipeline functions when [Additional Feature NP][] syntax is supported.

<table>
<thead>
<tr>
<th>With smart pipelines
<th>Status quo

<tbody>
<tr>
<td>

```js
const cssQuery = +> ##.querySelectorAll(#);
const setStyle = +> { ##.style = # };
document
|> cssQuery('a, p', #)
|> #.map(+> setStyle({ color: 'red' }));
```

<td>

```js
const cssQuery = R.invoker(1,
  'querySelectorAll');
const setStyle = R.assoc('style');
R.pipe(
  cssQuery('a, p'),
  R.map(setStyle({ color: 'red' }))
)(document);
```

<tr>
<td>

```js
const disco = +>
|> R.zipWith(+> #(##),
    [ red, green, blue ])
|> #.join(' ');
[ 'foo', 'bar', 'xyz' ]
|> disco
|> console.log;
```

<td>

```js
const disco = R.pipe(
  R.zipWith(
    R.call,
    [ red, green, blue ]),
  R.join(' '));
console.log(
  disco([ 'foo', 'bar', 'xyz' ]));
```

<tr>
<td>

```js
const dotPath = +>
|> (#.split('.'), ##)
|> R.path(#, ##);
const propsDotPath = +>
|> (R.map(dotPath), [##])
|> R.ap;
const obj = {
  a: { b: { c: 1 } },
  x: 2
};
propsDotPath(['a.b.c', 'x'], obj);
// [ 1, 2 ]
```

<td>

```js
const dotPath = R.useWith(
  R.path,
  [R.split('.')]);
const propsDotPath = R.useWith(
  R.ap,
  [R.map(dotPath), R.of]);
const obj = {
  a: { b: { c: 1 } },
  x: 2
};
propsDotPath(['a.b.c', 'x'], obj);
// [ 1, 2 ]
```

</table>

### WHATWG Streams Standard (Core Proposal + Additional Features BP+PP+PF+NP)
[Many examples above using WHATWG Streams][WHATWG Streams + CP + BP + PF]
benefited from pipeline functions with Additional Features CP + PF. Even more
use cases are covered by pipeline functions with [Additional Feature NP][].

<table>
<thead>
<tr>
<th>With smart pipelines
<th>Status quo

<tbody>
<tr>
<td>

```js
try {
  readableStream
  |> await #.pipeTo(writableStream);

  "Success"
  |> console.log;
}
catch
|> ("Error", #)
|> console.error;
```
This example also uses [Additional Feature TS][] for terse `catch` clauses.

<td>

```js
readableStream.pipeTo(writableStream)
  .then(() => console.log("Success"))
  .catch(e => console.error("Error", e));
```

<tr>
<td>

```js
const reader = readableStream
  .getReader({ mode: "byob" });

try {
  new ArrayBuffer(1024)
  |> await readInto
  |> ("The first 1024 bytes:", #)
  |> console.log;
}
catch
|> ("Something went wrong!", #)
|> console.error;

async function readInto(buffer, offset = 0) {
  return buffer |> {
    if (#.byteLength === offset)
      #;
    else
    |> (#, offset, #.byteLength - offset)
    |> new Uint8Array
    |> await reader.read
    |> (#.buffer, #.byteLength)
    |> readInto(#, offset + ##);
  };
}
```
This example also uses [Additional Feature TS][] for terse `catch` clauses,
[Additional Feature BC][] for a terse constructor call on `Uint8Array`, and
[Additional Feature BA][] for a terse async function call on `readInto`.

<td>

```js
const reader = readableStream
  .getReader({ mode: "byob" });

let startingAB = new ArrayBuffer(1024);
readInto(startingAB)
  .then(buffer =>
    console.log("The first 1024 bytes:", buffer))
  .catch(e =>
    console.error("Something went wrong!", e));

function readInto(buffer, offset = 0) {
  if (offset === buffer.byteLength) {
    return Promise.resolve(buffer);
  }
  const view = new Uint8Array(
    buffer, offset, buffer.byteLength - offset)
  return reader.read(view).then(newView => {
    return readInto(newView.buffer,
      offset + newView.byteLength);
  });
}
```

</table>

## Additional Feature FS
With the [Core Proposal][] only, `for`–`of` statements would prohibit the use
of `#` within their bodies, except where `#` is inside an inner pipeline inside
the `for` loop. But this could be changed afterward with Additional Feature FS –
**Topical `for`s** – which would cause `for` loops to bind the topic to useful
values, which in turn would make `for` loops terser, emphasizing what happens to
each item rather than the items’ variables themselves.

[Additional Feature FS is **formally specified in in the draft
specification**][formal FS].

<table>
<thead>
<tr>
<th>With smart pipelines
<th>Status quo

<tbody>
<tr>
<td>

With Additional Feature FT, a tacit pipeline `for` loop would be added, which
would tacitly apply a pipeline body to each iterator value.
```js
for (range(0, 50)) |> {
  log(# ** 2);
  log(|> Math.sqrt);
}
```
This `for` loop form, completely lacking a parenthesized antecedent, would also
be added. This example uses that tacit form, along with [Additional
Feature PP][].

<td>

```js
for (const i of range(0, 50)) {
  log(i ** 2);
  log(Math.sqrt(i));
}
```

<tr>
<td>

```js
for (const i of range(0, 50)) {
  log(# ** 2);
  log(|> Math.sqrt);
}
// 🚫 Syntax Error:
// Lexical context `for (…) { … }`
// contains a topic reference
// but has no topic binding.
```
This implicit binding would only occur [when the developer opts into this
behavior][opt-in behavior] with `|>`. It would never occur when a normal,
explicit parenthesized binding exists. Attempting to use a topic reference
in such a regular `for` block would trigger an [early error rule][].

<td>

<tr>
<td>

Similar additions would be made to the asynchronous `for` loop. A pipeline
`for`–`await`–`of` loop would implicitly bind each iterator value to `#`.
```js
for await (stream) |> {
  yield |>
  |> await f(#, 1)
  |> #.length
  |> # + 3
  |> g;
}
```
Note that in this case, a `|>` (or a `#`) must be included after `yield` because
– as usual – a newline after a `yield` causes [automatic semicolon
insertion][ASI] after the `yield`.

<td>

```js
for await (const c of stream) {
  yield g(
    (await f(c, 1))
      .length
      + 3
  );
}
```

</table>

# Goals

There are seventeen ordered goals that the smart body syntax tries to fulfill,
which may be summarized,\
“Don’t break my code,”\
“Don’t make me overthink,”\
“Don’t shoot me in the foot,”\
“Make my code easier to read,”\
and a few other goals.

<table>
<tr>
<td>

**“Don’t break my code.”**

 1. [Backward compatibility](#backward-compatibility)
 2. [Zero runtime cost](#zero-runtime-cost)
 3. [Forward compatibility](#forward-compatibility)

<td>

**“Don’t make me overthink.”**

 4. [Syntactic locality](#syntactic-locality)
 5. [Semantic clarity](#semantic-clarity)
 6. [Expressive versatility](#expressive-versatility)
 7. [Cyclomatic simplicity](#cyclomatic-simplicity)

<td>

**“Don’t shoot me in the foot.”**

 8. [Simple scoping](#simple-scoping)
 9. [Opt-in behavior](#opt-in-behavior)
10. [Static analyzability](#static-analyzability)

<tr>
<td>

**“Make my code easier to read.”**

11. [Untangled flow](#untangled-flow)
12. [Distinguishability](#distinguishability)
13. [Terse parentheses](#terse-parentheses)
14. [Terse variables](#terse-variables)
15. [Terse function calls](#terse-function-calls)

<td>

**Other**

15. [Conceptual generality](#conceptual-generality)
16. [Human writability](#human-writability)
17. [Novice learnability](#novice-learnability)

</table>

## “Don’t break my code.”
The syntax should not break any existing code; it should also be forward
compatible with future code.

### Backward compatibility
The syntax must avoid stepping on the toes of existing code, including but not
limited to JavaScript libraries such as [jQuery][] and [Lodash][]. In particular,
the topic reference should not be an existing identifier such as `$` or `_`,
which both may cause surprising results to a developer who adopts pipelines
while also using a globally bound convenience variable. It is a common pattern
to do this even without a library: `var $ = document.querySelectorAll`”. The
silent shadowing of such outer-context variables may silently cause bugs, which
may also be difficult to debug (see [Expressive Versatility][]).

Nor can it cause previously valid code to become invalid. This includes, to a
lesser extent, common nonstandard extensions to JavaScript: for instance, using
`<>` for the topic reference would retroactively invalidate existing E4X and JSX
code.

This proposal uses `#` for its topic reference. This is compatible with all
known previous JavaScript code. `?` and `@` could be chosen instead, which are
each also backwards compatible.

### Zero runtime cost
This could be considered a specific type of backward compatibility. When
translating old code into the new syntax, doing so should not cause unexpected
performance regression. For instance, the new syntax should not require memory
allocation for newly created functions that were not necessary in the old code.
Instead, it should, at least theoretically, perform as well the old code did for
both memory and CPU. And it should be able to do this without dramatically
rearranging code logic or relying on hidden, uncontrollable compiler
optimization.

For instance, in order to apply the syntax to the logic of an async functions, a
hypothetical new pipeline syntax might not support using `await` in the same
async context as the pipeline itself. Such a syntax would, for each of its
pipelines’ steps, require inner async functions that would return wrapper
promises and pass them between consecutive steps. Such an approach would be be
unnecessarily expensive to naively evaluate for both CPU and memory. But
inlining these async functions may be internally complicated, and such
optimizations would be difficult for the developer to correctly predict and
might differ widely between JavaScript engines.

Instead, this proposal’s use of a topic reference enables the zero-cost
rewriting of any expression within the current environmental context, including
`await` operations in async functions, without having to create unnecessary
inner async functions, and without having to wrap values in unnecessary promises.

### Forward compatibility
The syntax should not preclude other proposals: both [already-proposed
ECMAScript proposals][other ECMAScript proposals], such as [partial function
application][] and [private class fields][] – as well as the [Additional
Features][intro] of this proposal. The Core Proposal is forward compatible with
all of these, especially because of its [early errors][].

## “Don’t shoot me in the foot.”
The syntax should not be a footgun: it should not easy for a developer to
accidentally shoot themselves in the foot with it. Lessons from the [topic
variables of other programming languages][topic references in other programming
languages] may be instructive in formulating these goals.

### Opt-in behavior
The lexical topic is implicit, hidden state. State is intrinsically dangerous in
that it may induce the developer to commit [mode errors][], later “surprising”
the developer with unpredicted behavior. It should therefore not be easy for a
developer to accidentally bind or use the topic. If the developer accidentally
binds or uses the topic, any use of that reference could result in subtle,
pernicious bugs.

The larger the probability that a developer will accidentally clobber or
overwrite the topic, the less predictable their code becomes. It should not be
easy to accidentally shadow a reference from an outer lexical scope.

The larger the probability that a developer accidentally uses the topic, the
less predictable their code becomes. It should not be easy to accidentally use
the current lexical scope’s topic. In particular, bare/tacit function calls that
use the topic should not be easy to accidentally perform.

In this proposal, the developer therefore **must explicitly opt into**
topic-using behavior, whether binding or using, by using the pipeline operator
`|>`. This includes [Additional Feature TS][] and [Additional Feature FS][],
which both require the use of `|>`.

This is quite different than [much prior art in other programming
languages][topic references in other programming languages]. Other languages
frequently bind their topic references using numerous syntactic structures, with
no way for the developer to opt out. In addition, bare/tacit function calls are
easier to accidentally perform in some programming languages – making it more
difficult to tell whether a bare identifier `print` is meant to be a simple
variable reference or a bare function call on the topic value.

### Simple scoping
It should not be easy to accidentally shadow a reference from an outer lexical
scope. When the developer does so, any use of that reference could result in
subtle, pernicious bugs.

The rules for where the topic is bound should be simple and consistent. It should
be clear and obvious when a topic is bound and in what scope it exists. And
forgetting these rules should result in [early, compile-time errors][early
errors], not subtle runtime bugs.

It should always be easy to find the origin of a topic binding, without looking
deeply into the stack. Topic references are therefore bound only in the bodies
of pipelines, and they cannot be used within `function`, `class`, `for`,
`while`, `catch`, and `with` statements (see [Core Proposal][]). When the
developer wishes to trace the origin of a topic binding, they may be certain
that if they find any of such statements during their search, they have moved
too far and should retrace their path for the topic binding.

This proposal’s topic references are different than [much prior art in other
programming languages][topic references in other programming languages]. Other
languages frequently use dynamic binding rather than lexical binding for their
topic references.

### Static analyzability
[Early errors][] help the editing JavaScript developer avoid common [footguns][]
at compile time, such as preventing them from accidentally omitting a topic
reference where they meant to put one. For instance, if `x |> 3` were not an
error, then it would be a useless operation and almost certainly not what the
developer intended. Situations like these should be statically detectable and
cause compile-time [early errors][].

The same preference for strict early errors is used by the class decorators
proposal: see [tc39/proposal-decorators#30][], [tc39/proposal-decorators#42][],
and [tc39/proposal-decorators#60][]. Early errors also assist with [forward
compatibility][], as changing a behavior from “throws” to “does something” is
generally web compatible, though the reverse is not true.

## “Don’t make me overthink.”
The syntax should not make a developer overthink about the syntax, rather than
their product.

### Syntactic locality
The syntax should minimize the parsing lookahead that the compiler must check.
If the grammar makes [garden-path syntax][] common, then this increases the
dependency that pieces of code have on other code. This long lookahead in turn
makes it more likely that the code will exhibit developer-unintended behavior.

This is true particularly for [distinguishing between different styles of
pipeline body syntax][smart body syntax]. A pipeline’s meaning would often be
ambiguous between these styles – at least without checking the pipeline’s body
carefully to see in which style it is written. And the pipeline body may be a
very long expression.

By restricting the space of valid bare-style pipeline bodies (that is, without
topic references), the rule minimizes garden-path syntax that would otherwise be
possible – such as `value |> compose(f, g, h, i, j, k, #)`. Syntax becomes more
locally readable. It becomes easier to reason about code without thinking about
code elsewhere.

### Semantic clarity

<table>
<tbody>
<tr>
<td colspan=2>

The syntax might be unambiguous, but its semantics can be more difficult to interpret.

<tr>
<td>

For instance:
```js
input |> object.method;
```
…may be clear enough.

<td>

It means:
```js
object.method(input);
```

<tr>
<td>

But:
```js
input |> object.method(); 🚫
```
…is less clear.

<td>

It could reasonably mean either of these lines:
```js
object.method(input);
object.method()(input);
```

<tr>
<td>

Adding other arguments:
```js
input |> object.method(x, y); 🚫
```
…makes it worse.

<td>

It could reasonably mean any of these lines:
```js
object.method(input, x, y);
object.method(x, y, input);
object.method(x, y)(input);
```

<tr>
<td>

And this is even worse:
```js
input |> await object.method(x, y); 🚫
```

<td>

It could reasonably mean any of these lines:
```js
await object.method(input, x, y);
await object.method(x, y, input);
await object.method(x, y)(input);
(await object.method(x, y))(input);
```

<tr>
<td colspan=2>

It is undesirable for the human reader to be uncertain which of multiple
interpretations of a pipeline – all which are reasonable – is correct. It is
both a distracting incidental cognitive burden and a potential source of
developer error. [The Zen of Python][PEP 20] famously says, “Explicit is better
than implicit,” for reasons such as these. And it is for these reasons that this
proposal makes the unclear pipelines above [early errors][].

<tr>
<td>

This pipeline:
```js
input |> object.method;
```
…is a valid [bare-style pipeline][bare style]. Bare style is designed to be
strictly simple: it must either be a simple reference or it is not in bare style.

<td>

```js
object.method(input);
```

<tr>
<td>

This:
```js
input |> object.method(); 🚫
```
…is an invalid [topic-style pipeline][topic style]. It is in topic style because
it is not a simple reference; it has parentheses. And it is invalid because it is
in topic style yet it does not have a topic reference.

<td>

The writing developer is forced by the compiler to clarify their intended
meaning, using a topic reference:
```js
input |> object.method(#);
input |> object.method()(#);
```
The reading developer benefits from explicitness and clarity, without
sacrificing the benefits of [untangled flow][] that pipelines bring.

<tr>
<td>

Adding other arguments:
```js
input |> object.method(x, y); 🚫
```
…is the same. This is an invalid topic-style pipeline.

<td>

The writer must clarify which of these reasonable interpretations is correct:
```js
input |> object.method(#, x, y);
input |> object.method(x, y, #);
input |> object.method(x, y)(#);
```
Both inserting the input as the first argument and inserting it as the last
argument are reasonable interpretations, as evidenced by how [other programming
languages’ pipeline operators variously do either][topic references in other
programming languages]. Or it could be a factory method that creates a function
that is in turn to be called with a unary input argument.

<tr>
<td>

Invalid topic-style pipeline:
```js
input |> await object.method(x, y); 🚫
```

<td>

Valid topic-style pipelines:
```js
input |> await object.method(#, x, y);
input |> await object.method(x, y, #);
input |> await object.method(x, y)(#);
input |> (await object.method(x, y))(#);
```

</table>

### Expressive versatility
JavaScript is a language rich with [expressions of numerous kinds][MDN
operator precedence], each of which may usefully transform data from one
form to another. There is **no single type** of expression that forms a
**majority of used expressions**.

<table>
<tr>
<td>

 1. `undefined` and `null`.
 2. Boolean literals.
 3. String literals.
 4. Regular-expression literals.

<td>

 5. Template literals.
 6. Array literals.
 7. Object literals.

<tr>
<td>

 8. Variable references.
 9. Property accessors.
10. `this`.
11. `new.target`.

<td>

13. Arithmetic operations.
14. Bitwise operations.
15. Logical operations.

<tr>
<td>

16. Equality operations.
17. `instanceof` and `in` operations.
18. Conditional operations.

<td>

19. Unary function calls.
20. Unary constructor calls.
21. N-ary function calls.
22. N-ary constructor calls.
23. `super` calls.

<tr>
<td>

23. Arrow functions.
24. Function definitions.
25. Generator definitions.

<td>

26. Async-function definitions.
27. Async-generator definitions.
28. Class definitions.

<tr>
<td>

29. `typeof` operations.
30. `void` expressions.
31. `await` expressions.
32. `yield` expressions.

<td>

33. [Function binding?][Function binding]

</table>

The goal of the pipeline operator is to untangle deeply nested expressions into flat
threads of postfix expressions. To limit it to only one type of expression, even
a common type, truncates its benefits to that one type only and compromises its
expressivity and versatility.

In particular, relying on immediately invoked function expressions ([IIFEs][])
to accomodate non-unary function is insufficient for idiomatic JavaScript code.
JavaScript functions have never fulfilled the [Tennent correspondence
principle][]. Several common types of expressions cannot be equivalently used
within inner functions, particularly `await` and `yield`. In these frequent
cases, attempting to replacing code with “equivalent” IIFEs may cause different
behavior, may cause different performance behavior (see example in [zero runtime
cost][]), or may require dramatic rearrangement of logic to conserve the old
code’s behavior.

It would be possible to add ad-hoc handling, for selected other expression
types, to the operator’s grammar. This would expand its benefits to that type.
However, this conflicts with the goal of [cyclomatic simplicity][], by adding
complexity to the parsing process, proportional to the number of ad-hoc handled
cases. It also does not fulfill this goal well either: excluding, perhaps
arbitrarily, whatever classes its grammar’s branches do not handle.

Such new [incidental complexity][] makes code less readable and distracts the
developer from the program’s [essential logic][essential complexity]. A pipeline
operator that improves readability should be versatile (this goal) but
[conceptually and cyclomatically simple][cyclomatic simplicity]. Such an
operator should be able to handle **all** expressions, in a **single** manner
**uniformly** **universally** applicable to **all** expressions. It is the hope
of this proposal’s authors that its [smart body syntax][] fulfills both criteria.

### Cyclomatic simplicity
Each edge case of the grammar increases the [cyclomatic complexity][] of parsing
the new syntax, increasing cognitive burden on both machine compiler and human
reader in writing and reading code without error. If edge cases and branching
are minimized, then the resulting syntax will be uniform and consistent. The
reduced complexity would hopefully reduce the probability that the developer
will misunderstand the code they read or write.

Similarly, reducing edge cases reduces the amount of trivia that a developer
must learn and remember in order to use the syntax. The more uniform and
simple the syntax’s rules, the more the developer may focus on the actual
meaning of their code.

Both [expressive versatility][] and simplicity are important components of
[“don’t make me overthink”][], but they sometimes conflict with one another.
When this happens, expressive versatility often wins: simplicity is important,
but sometimes it may be traded off for increased expressiveness. For instance,
[terse function calls][] are important for [tacit functional programming][tacit
programming], one of the impetuses for the [first pipeline-operator proposal][].

The pipeline operator could be designed to support only [topic style][]: that
would require `x |> f` to be `x |> f(#)`. But adding a [bare style][] brings
many expressive benefits for tacit functional programming: not just [terse
function calls][] but also the possibility of [terse composition][] with
[Additional Feature PF][].

But even with this tradeoff, not too much simplicity should be given up. The
sacrifice of simplicity for bare style’s alternate mode can be minimized by
ensuring that [its parsing rules are very simple][smart body syntax].

## “Make my code easier to read.”
The new syntax should increase the human readability and writability of much
common code. It should be simpler to read and comprehend. And it should be
easier to compose and update. Otherwise, the new syntax would be useless.

Making JavaScript expressions more ergonomic for humans is the prime, original
purpose of this proposal. To a computer, the form of complex expressions –
whether as deeply nested groups or as flat threads of postfix steps – should not
matter. But to a human, it can make a significant difference.

### Untangled flow
When a human reads deeply nested groups of expressions – which are very common
in JavaScript code – their attention must switch between the start and end of
each nested expression. And these expressions will dramatically differ in
length, depending on their level in the syntactic tree. To use the example above:
```js
console.log(
  await stream.write(
    new User.Message(
      capitalize(
        doubledSay(
          (await promise)
            || throw new TypeError(`Invalid value from ${promise}`)
        )
      ) + '!')));
```
…the deep inner expression `await promise` is relatively short. In
contrast, the shallow outer expression
`` capitalize(doubledSay((await promise) || throw new TypeError(`Invalid value from ${promise}`))) + '!'`) ``
is very long. Yet both are
quite similar: they are transformations of a string into another. This
insight is lost in the deeply nested noise.

With pipelines, the code forms a flat thread of postfix steps. It is much
easier for a human to read and comprehend. Each of its steps are roughly the
same length. In order to understand what occurs before a given step, one
only need to scan left, rather than in both directions as the deeply nested
tree would require. To read the whole thing, a reader may simply follow
along left to right, not back and forth.
```js
promise
|> await #
|> # || throw new TypeError()
|> doubleSay(#, ', ')
|> capitalize
|> # + '!'
|> new User.Message(#)
|> await stream.write(#)
|> console.log;
```

The introduction to this [motivation][] section already explained much of
the readability rationale.

### Distinguishable punctuators
Another important aspect of code readability is the visual distinguishability of
its most important words or symbols. Visually similar punctuators can distract
or even mislead the human reader, as they attempt to figure out the true meaning
of their code.

Any new punctuator should be easily distinguishable from existing symbols and should
not be visually confusable with unrelated syntax. This is particularly true for
choosing the topic-reference token, which would appear often in a wide variety
of expressions. If the topic reference hypothetically were `?` (and `??` and
`???` with [Additional Feature NP][]), and if the topic reference were used
anywhere near the visually similar [optional-chaining syntax proposal][] and
[nullish coalescing proposal][], then the topic reference might be lost or
unnoticed by the developer: for example, `(?)??.m(??)` is much less readable
than `#??.m(##)`.

### Terse parentheses
Terseness also aids distinguishability by obviating the need for boilerplate
syntactic noise. Parentheses are a prominent example: as long as operator
precedence is clear, then reducing parentheses always would JavaScript code more
visually terse and less cluttered.

The example above demonstrates how numerous verbose parentheses could become
unnecessary with pipelines. In these cases the [“data-to-ink” visual ratio][]
would significantly increase, emphasizing the program’s essential information.
The developer’s cognitive burden – of ignoring unimportant incidental symbols as
they read – has hopefully lightened.

### Terse variables
Similarly, terseness of code may also be increased by removing variables where
possible. This in turn would increase the data-to-ink visual ratio of the text
and the distinguishability of important symbols. This style of programming is
known as [tacit or point-free programming][tacit programming] (where “point”
refers to function arguments). Jeremy Gibbons, a computer scientist, expressed
its claimed benefits in a 1970 paper as such:

> Our calculations got completely bogged down using [function arguments]. In
> attempting to rephrase [function] definitions […] in particular, eliminating
> as many variables as possible and performing point-free (or ‘pointless‘)
> calculations at the level of functino compisition instead of point-wise
> calculations at the level of application, suddenly the calculations became
> almost trivial. This is the point of [point-free] calculations: when you
> travel light – discarding variables that do not contribute to the calculation
> – you can sometimes step lightly across the surface of the quagmire.

This sort of terseness, in which the explicit is made tacit and implicit, must
be balanced with [syntactic locality][] and [cyclomatic simplicity][]. Excessive
implicitness compromises comprehensibility, at least without low-level tracing
of tacit arguments’ invisible paths, rather than the actual, high-level meaning
of the code. Yet at the same time, excessive explicitness generates ritual,
verbose boilerplate that also interferes with reading comprehension. Therefore,
[untangled flow][] must be balanced with [backward compatibility][], [syntactic
locality][], and [cyclomatic simplicity][].

[The Zen of Python][PEP 20] famously says, “Explicit is better than implicit,”
but it also says, “Flat is better than nested,” and, “Sparse is better than
dense.”

### Terse function calls
Unary function / constructor calls are a particularly frequent type of
expression and a good target for especial human optimization. However, such
extra shortening might dramatically reduce the verbosity of unary function
calls, but again this must be balanced with [backward compatibility][],
[syntactic locality][], and [cyclomatic simplicity][].

It is the hope of this proposal’s authors that its [smart body syntax][] reaches
a good balance between this goal and [syntactic locality][] and [cyclomatic
simplicity][], in the same manner that [Huffman coding][] optimizes textual
symbols’ length for their frequency of use: more commonly used symbols are
shorter in written length.

Furthermore, calls are not only unary; they may also be n-ary. [Additional
Feature NP][] adds support for terse N-ary function calls within pipelines.

### Terse composition
Terse composition of all expressions – [not only unary functions][expressive
versatility] but also n-ary functions, object methods, async functions,
generators, `if` `else` statements, and so forth – is a goal of smart pipelines.
[Several alternative proposals also address function composition][function
composition], but [Additional Feature PF][] holistically addresses it with
application, partial application, and some forms of method extraction, and not
only for unary functions but also for expressions of any type.

### Terse partial application
Terse partial application of all expressions – [not only functions][expressive
versatility] but also object methods, async functions, generators, `if` `else`
statements, and so forth – is a goal of smart pipelines. [An existing
alternative proposal also addresses partial function application][partial
function application], but [Additional Feature PF][] holistically addresses it
with application, partial application, and some forms of method extraction, and
not only for unary functions but also for expressions of any type. [Additional
Feature NP][] extends this ability to N-ary expressions, including variadic
expressions.

## Other Goals
Although these have been prioritized last, they are still important.

### Conceptual generality
If a concept is uniformly generalizable to many other cases, then this
multiplies its usefulness. The more versatile its concepts, the more it may be
applied to other syntax, including existing syntax and future syntax (compare
with [forward compatibility][]).

This proposal’s concept of a **topic reference does not need to be coupled only
to pipelines**. The topic concept is **generalizable to many syntactic forms**,
as the [additional features][] demonstrate. They together form one unified vision
of a future in which composition, partial application, and error handling are
all tersely expressible with the same simple concepts.

### Human writability
Writability of code is less important a priority than readability of code. Code
is usually written a few days, perhaps by a few authors – but code will be read
dozens or hundreds of times, perhaps by many more people. However, ease of
writing and editing is still a good goal, and it often naturally increases when
code also becomes more readable. A useful heuristic for writability is assessing
the probability that a single edit to one piece of code will necessitate changes
to other parts of code that are not directly related to the edit.

The simple addition or removal of a deeply nested expression may necessitate the
indentation, de-indentation, parenthetical grouping, and parenthetical
flattening of many lines of code; the tedium of these incidental changes is a
major factor in the general popularity of automatic code formatters.

Achieving [static analyzability][] therefore also improves the ease of composing
and editing code. By flattening deeply nested expression trees into single
threads of postfix steps, a step may be added oredited in isolation on a single
line, it may be rearranged up or down, it may be removed – all without affecting
the pipeline’s other steps in the lines above or below it.

### Novice learnability
Learnability of the syntax is a desirable goal: the more intuitive the syntax
is, the more rapidly it might be adopted by developers. However, learnability in
of itself is not more desirable than the [other goals above][goals]. Most
JavaScript developers would be novices to this syntax at most once, during which
the intuitiveness of the syntax will dominate their experience. But after that
honeymoon period, the syntax’s usability in workaday programming will instead
affect their reading and writing most.

So instead, readability, comprehensibility, locality, simplicity,
expressiveness, and terseness are prioritized first, where they would conflict
with learnability itself. However, a syntax that is simple but expressive – and,
most of all, readable – could well be easier to learn. Its up-front cost in
learning could be small, particularly in comparison to the large gains in
readability and comprehensibility that it might bring to code in general.

# Relations to other work
## Pipelines in other programming languages
The concept of a pipeline operator appears in numerous other languages, variously
called “pipeline”, “threading”, and “feed” operators. This is because developers
find the concept useful.

<table>
<tr>
<th>

`|`

<td>

tacit only parameter

<td>

[Unix shells, PowerShell][Unix pipe]

<tr>
<th>

`|>`

<td>

tacit first parameter

<td>

[Elixir and Erlang][Elixir pipe], [Julia][Julia pipe], [LiveScript][LiveScript pipe]

<tr>
<th>

`|>`

<td>

tacit last parameter

<td>

[Elm][Elm pipe], [OCaml][OCaml pipe]

<tr>
<th>

`|>`

<td>

tacit only parameter

<td>

[Julia][Julia pipe], [F# / F-sharp][F# pipe]

<tr>
<th>

`|>`

<td>

`$$` placeholder

<td>

[Hack][Hack pipe]

<tr>
<th>

`%>%`

<td>

tacit first parameter or `.` placeholder

<td>

[R with magrittr][R pipe]

<tr>
<th>

`==>`

<td>

tacit last parameter

<td>

[Perl 6][Perl 6 pipe]

<tr>
<th>

`->` `->>`\
`as->` `as->>`\
`some->` `some->>`\
`cond->` `cond->>`

<td>

tacit first or last parameters or arbitrary placeholder

<td>

[Clojure][Clojure pipe]

<tr>
<th>

[Term concatenation][concatenative programming]

<td>

tacit only parameter

<td>

Factor, Forth, Joy, Onyx, PostScript, RPL

</table>

Pipeline operators are also conceptually similar to [WHATWG-stream piping][] and
[Node-stream piping][].

## Topic references in other programming languages
The [concept of the “topic variable” already exists in many other programming
languages][topic variables in other languages], commonly named with an
underscore `_` or `$_`. These languages often integrate their topic variables
into their function-call control-flow syntaxes, with [Perl 6 as perhaps the most
extensive, unified example][Perl 6 topicization]. Integration of topic with
syntax enables especially pithy, terse [tacit programming][].

In addition, many JavaScript console [REPLs][], [such as the WebKit Web
Inspector console variables][WebKit console variables], [other browsers’ console
variables][] and the Node.js console variables.

Several disadvantages to these prior approaches may increase the probability of
developer surprise, in which “surprise” refers to behavior difficult to predict
by the developer.

One disadvantage arises from their frequent dynamic binding rather than lexical
binding, as the former is not [statically analyzable][] and is more stateful
than the latter. It may also cause surprising results when coupled with
bare/tacit calls: it becomes more difficult to tell whether a bare identifier
`print` is meant to be a simple variable reference or a bare function call on
the topic value.

Another disadvantage arises from the ability to clobber or overwrite the value of the
topic variable, which may affect code in surprising ways.

However, JavaScript’s topic references are is different than this prior art. It
is lexically bound, with [simple scoping][], and it is [statically
analyzable][]. It also cannot be accidentally bound; [the developer must opt
into binding it][opt-in behavior] by using the pipeline operator `|>`. (This
includes [Additional Feature TS][] and [Additional Feature FS][], which both
require the use of `|>`.) The topic also cannot be accidentally used; it is an
[early error][] when `#` is used outside of a pipeline body (see [Core
Proposal][] and [static analyzability][]). The proposal is as a whole designed
to [prevent footguns][“don’t shoot me in the foot”].

The topic is [conceptually general][conceptual generality] and could be extended
to other forms. This proposal is [forward compatible][] with such extensions,
which would increase its [expressive versatility][], and potentially multiplying
its benefits toward [untangled flow][], [terse variables][], and [human
writability][], while still preserving [simple scoping][] and [static
analyzability][].

## `do` expressions
There is a TC39 proposal for [`do` expressions][] at Stage 1. Smart pipelines do
**not** require `do` expressions. However, if [`do` expressions][] also become
part of JavaScript, then, as with **any** other type of expression, a pipeline
in [topic style][] may use a `do` as its body, as long as the `do` expression
contains the topic reference `#`. The topic reference `#` is bound to the
pipeline head’s value, the `do` expression is evaluated, then the result of the
`do` block becomes the final result of that pipeline, and the lexical
environment is reset – all as usual.

In this manner, pipelines with `do` expressions act as a way to create a
“topic-context block”, similarly to [Perl 6’s given block][]. Within this block,
statements may use the topic reference may be used as an abbreviation for the
same value. This can be useful for embedding side effects, `if` `else`
statements, `try` statements, and `switch` statements within pipeline chains.
They may be made even pithier with [Additional Feature BP][], explained later.

However, smart pipelines do not depend on `do` expressions, with the exception
of [Additional Feature BP][].

## Function binding
An existing proposal for [ECMAScript function binding][] has three use cases:

1. Extracting a method from an object as a standalone function:
   `object.method.bind(object)` as `::object.method`.
2. Calling a function as if it were a method call on an object:
   `func.call(object, ...args)` as `object::func(...args)`
3. Creating a function by binding an object to it:
   `func.bind(object)` as `object::func`.

The smart-pipelines [Core Proposal][] + [Additional Feature PF][] **subsumes**
the [ECMAScript function binding][] proposal in the **first use case** (prefix
`::`). But the **other two** use cases (infix `::`) are **not addressed** by
smart pipelines. Smart pipelines and **infix** function binding `::` **can and
should coexist**. In fact, infix function binding could be made more ergonomic
in many cases by replacing prefix `::function` with a shortcut for the
expression `#::function`.

<table>
<thead>
<tr>
<th>With smart pipelines
<th>

With [existing proposal][ECMAScript function binding]

<tbody>
<tr>
<td>

**Some** forms of **method extraction** can be addressed by pipeline functions
alone, as a natural result of their pipeline-operator-like semantics.\
`+> console.log` is equivalent to `(...$) => console.log(...$)`.
```js
Promise.resolve(123).then(+> console.log);
```

<td>

```js
Promise.resolve(123).then((...$) => console.log(...$));
```

<tr>
<td>

```js
$('.some-link').on('click', +> view.reset);
```

<td>

```js
$('.some-link').on('click', ::view.reset);
```

<tr>
<td>

Note that this is *not* the same as `console.log.bind(console.log)`, which
creates an exotic function that always uses whatever value `console.log`
evaluates into – even if `console.log` is reassigned later.
```js
const consoleLog =
  console.log.bind(console.log);
const arrayFrom =
  Array.from.bind(Array.from);
const arrayMap =
  Function.bind.call(Function.call,
    Array.prototype.map);
…
input
|> process
|> consoleLog;
input
|> arrayFrom
|> arrayMap(#, $ => $ + 1)
|> consoleLog;
```
This [robust method extraction][] is a use case that this proposal leaves to
another operator, such as prefix `::` or prefix `&`.

<td>

```js
const consoleLog =
  console.log.bind(console.log);
const arrayFrom =
  Array.from.bind(Array.from);
const arrayMap =
  Function.bind.call(Function.call, Array.prototype.map);
…
consoleLog(
  process(input));
consoleLog(
  arrayMap(arrayFrom(input), $ => $ + 1));
```

<tr>
<td>

```js
…
input
|> process
|> &console.log;
input
|> &Array.from
|> #::&Array.prototype.map($ => $ + 1)
|> &console.log;
```
Pipeline functions would not preclude adding another operator that addresses
[robust method extraction][] with [inline caching][method-extraction inline
caching], such as the hypothetical prefix `&` operators (for cached method
extraction) and infix `::` operators (for `this` binding) shown here. Such
hypothetical notations could even be eventually accommodated by a new [bare
style][] notation, shown here with `… |> &console.log` and `… |> &Array.from`.

<td>

```js
…
consoleLog(
  process(input));
consoleLog(
  &Array.from(input)
  ::&Array.prototype.map($ => $ + 1));
```

<tr>
<td>

```js
const { hasOwnProperty } = Object.prototype;
const x = { key: 5 };
x::hasOwnProperty;
x::hasOwnProperty('key');
```
For terse **method calling/binding**, the infix `::` operator would also still
be required.

<td>

```js
const { hasOwnProperty } = Object.prototype;
const x = { key: 5 };
x::hasOwnProperty;
x::hasOwnProperty('key');
```

<tr>
<td>

The [function bind operator `::`][]’s proposal switches its prefix form
`::function` to mean `#::function` instead of `object.bind(object)`, then many
nested functions would become even terser.
```js
a(1, +>
  ::b(2, +> …)
);
```
See [block parameters][] for further examples.

<td>

```js
a(1, $ =>
  $::b(2, $ => …)
);
```

</table>

## Function composition
**[Terse composition][]** on unary functions is a goal of smart pipelines.
It is equivalent to piping a value through several function calls, within a
unary function, starting with the outer function’s tacit unary parameter.

There are several existing proposals for unary functional composition, which
[Additional Feature PF][] would all subsume. Additional Feature PF can compose
not only unary functions but [expressions of any type][expressive versatility],
including object methods, async functions, and `if` `else` statements. And with
[Additional Feature NP][], even functional composition into n-ary functions
would be supported, which no current proposal yet addresses.

<table>
<thead>
<tr>
<th>With smart pipelines
<th>With alternative proposals

<tbody>
<tr>
<td>

```js
array.map(+> f |> g |> h(2, #) |> # + 2);
```

<td>

```js
array.map($ => h(2, g(f($))) + 2);
```

<tr>
<td>

When compared to the proposal for [syntactic functional composition by
TheNavigateur][TheNavigateur functional composition], this syntax does not need
to give implicit special treatment to async functions. There is instead an async
version of the pipe-function operator, within which `await` may be used, simply
as usual.
```js
const doubleThenSquareThenHalfAsync =
  async +>
    |> double
    |> await squareAsync
    |> half;
```
This example uses [Additional Feature BA][] for `… |> await squareAsync`.

<td>

```js
const doubleThenSquareThenHalfAsync =
  async $ =>
    half(await squareAsync(double($)));
```
```js
const doubleThenSquareThenHalfAsync =
  double +> squareAsync +> half;
```
From the proposal for [syntactic functional composition by
TheNavigateur][TheNavigateur functional composition].

<tr>
<td>

```js
const toSlug =
  $ => $
  |> #.split(' ')
  |> #.map($ => $.toLowerCase())
  |> #.join('-')
  |> encodeURIComponent;
```
This example also uses [Additional Feature PP][] for its second line:
```js
const toSlug = +>
|> #.split(' ')
|> #.map(+> #.toLowerCase())
|> #.join('-')
|> encodeURIComponent;
```
When compared to the proposal for [syntactic functional composition by Isiah
Meadows][isiahmeadows functional composition], this syntax does not need to
surround each non-function expression with an arrow function. The [smart body
syntax][] has more powerful [expressive versatility][], improving the
readability of the code.

<td>

```js
const toSlug = $ =>
  encodeURIComponent(
    $.split(' ')
      .map(str =>
        str.toLowerCase())
      .join('-'));
```
```js
const toSlug =
  _ => _.split(" ")
  :> _ => _.map(str =>
    str.toLowerCase())
  :> _ => _.join("-")
  :> encodeURIComponent;
```
From the proposal for [syntactic functional composition by Isiah
Meadows][isiahmeadows functional composition].

<tr>
<td>

Lifting of non-sync-function expressions into function expressions is
unnecessary for composition with Additional Feature PF.
```js
const getTemperatureFromServerInLocalUnits =
  async +>
  |> await getTemperatureKelvinFromServerAsync
  |> convertTemperatureKelvinToLocalUnits;
```
This example uses [Additional Feature BA][] for
`… |> await getTemperatureKelvinFromServerAsync`.

<td>

```js
Promise.prototype[Symbol.lift] = f => x =>
  x.then(f);
const getTemperatureFromServerInLocalUnits =
  getTemperatureKelvinFromServerAsync
  :> convertTemperatureKelvinToLocalUnits;
```
From the proposal for [syntactic functional composition by Isiah
Meadows][isiahmeadows functional composition].

<tr>
<td>

```js
// Functional Building Blocks
const car = +>
|> startMotor
|> useFuel
|> turnKey;
const electricCar = +>
|> startMotor
|> usePower
|> turnKey;

// Control Flow Management
const getData = +>
|> truncate
|> sort
|> filter
|> request;

// Argument Assignment
const sortBy = 'date';
const getData = +>
|> truncate
|> sort
|> #::filter(sortBy)
|> request;
```
This example also uses [function binding][].

<td>

```js
// Functional Building Blocks
const car = startMotor.compose(
  useFuel, turnKey);
const electricCar = startMotor.compose(
  usePower, turnKey);

// Control Flow Management
const getData = truncate.compose(
  sort, filter, request);

// Argument Assignment
const sortBy = 'date';
const getData = truncate.compose(
  sort,
  $ => filter.bind($, sortBy),
  request
);
```
From the proposal for [syntactic functional composition by Simon
Staton][simonstaton functional composition].

<tr>
<td>

```js
const pluck = +> map |> prop;
```

<td>

```js
const pluck = compose(map)(prop);
```
From a [comment about syntactic functional composition by Tom Harding][i-am-tom
functional composition].

</table>

## Partial function application
[Terse partial application][] is a goal of smart pipelines. The current proposal
for syntactic [ECMAScript partial application][] by [Ron Buckton][] would be
subsumed by [Additional Feature PF][] and [Additional Feature NP][]. Terse
partial application into an N-ary function is equivalent to piping N tacit
parameters into an N-ary function-call expression, within which the parameters
are resolvable topic references. (Additional Feature PF alone would only address
partial application into unary functions.)

Pipeline functions look similar to the alternative proposal, except that
partial-application expressions are simply pipeline bodies that are prefixed by
the pipeline-function operator, and consecutive `?` placeholders are instead
consecutive topic references `#`, `##`, `###`.

The current proposal for [partial function application][] assumes that each use
of the same `?` placeholder token represents a different parameter. In contrast,
each use of `#` within the same scope always refers to the same value. This is
why additional topic parameters are required.

The resulting model is more flexible: with Additional Feature NP with
[Additional Feature PF][], `+> f(#, 4, ##)` is different from `+> f(#, 4, #)`.
The former refers to a **binary** function: a function with two parameters,
essentially `(x, y) => f(x, 4, y)`. The latter refers to a **unary** function
that passes the same one argument into both the first and third parameters of
the original function `f`: `x => f(x, 4, x)`. The same symbol refers to the same
value in the same lexical environment.

<table>
<thead>
<tr>
<th>With smart pipelines
<th>With alternative proposals

<tbody>
<tr>
<td>

```js
array.map($ => $ |> f(2, #));
array.map(+> f(2, #));
```

<td>

```js
array.map(f(2, ?));
array.map($ => f(2, $));
```

<tr>
<td>

```js
const addOne = +> add(1, #);
addOne(2); // 3
```

<td>

```js
const addOne = add(1, ?);
addOne(2); // 3
```

<tr>
<td>

```js
const addTen = +> add(#, 10);
addTen(2); // 12
```

<td>

```js
const addTen = add(?, 10);
addTen(2); // 12
```

<tr>
<td>

```js
let newScore = player.score
|> add(7, #)
|> clamp(0, 100, #);
```

<td>

```js
let newScore = player.score
|> add(7, ?)
|> clamp(0, 100, ?);
```

<tr>
<td>

```js
const toSlug = +>
|> encodeURIComponent
|> _.split(#, " ")
|> _.map(#, _.toLower)
|> _.join(#, "-");
```
Additional Feature PF simultaneously handles function composition and
partial application into unary functions.

<td>

```js
const toSlug =
encodeURIComponent
:> _.split(?, " ")
:> _.map(?, _.toLower)
:> _.join(?, "-");
```
From the proposal for [syntactic functional composition by Isiah
Meadows][isiahmeadows functional composition].

<tr>
<td>

```js
[ { x: 22 }, { x: 42 } ]
  .map(+> #.x)
  .reduce(+> # - ##, 0);
```

<td>

```js
[ { x: 22 }, { x: 42 } ]
  .map(el => el.x)
  .reduce((_0, _1) => _0 - _1, 0);
```

<tr>
<td>

```js
const f = (x, y, z) => [x, y, z];
const g = +> f(#, 4, ##);
g(1, 2) // [1, 4, 2];
```

<td>

```js
const f = (x, y, z) => [x, y, z];
const g = f(?, 4, ?);
g(1, 2) // [1, 4, 2];
```

<tr>
<td>

```js
const maxGreaterThanZero =
  +> Math.max(0, ...);
maxGreaterThanZero(1, 2); // 2
maxGreaterThanZero(-1, -2); // 0
```

Partial application into a variadic function is also naturally handled by
Additional Feature NP with [Additional Feature PF][].

<td>

```js
const maxGreaterThanZero =
  Math.max(0, ...);
maxGreaterThanZero(1, 2); // 2
maxGreaterThanZero(-1, -2); // 0
```
In this case, the topic function version looks once again nearly identical to
the other proposal’s code.

</table>

## Optional `catch` binding
In addition, a bare `catch` form, completely lacking a parenthesized antecedent,
has already been proposed as [ECMAScript optional `catch` binding][]. This bare
form is mutually compatible with this proposal, including with [Additional
Feature TS][]. The developer must **[opt into using Additional
Feature TS][opt-in behavior]** by using a pipeline token `|>`, followed by the
pipeline body. No existing code would be affected. Any, some, or none of the
three clauses in a `try` statement may be in a pipeline form versus the regular
block form or the bare block form.

<table>
<thead>
<tr>
<th>With smart pipelines too
<th>With optional `catch` binding only

<tbody>
<tr>
<td>

```js
value
|> f
|> {
  try {
    |> 1 / #;
  }
  catch {
    { type: error };
  }
}
|> g;
```
Even with [Additional Feature TS][], omitting `catch` binding still works,
although the topic cannot be used within the block without a `|>`.

<td>

```js
let _1;
try {
  _1 = 1 / f(value);
}
catch (error) {
  _1 = { message: error.message };
}
g (_1, 1);
```

<tr>
<td>

```js
value
|> f
|> {
  try { 1 / #; }
  catch {
    #.message |> console.error;
  }
}
|> g(#, 1);
// 🚫 Syntax Error:
// Lexical context `catch { … }`
// contains a topic reference
// but has no topic binding.
```
If the developer leaves out the `|>` after the `catch`, then no topic binding is
established. As per the [early error rules][] in [Core Proposal][], topic
references are not allowed in regular `catch` blocks. This sort of [opt-in
behavior][] is a goal of this proposal and helps ensure that the developer does
not [shoot themselves in the foot][“don’t shoot me in the foot”] by accidentally
using the topic value from an unexpected outer environment.

</table>

## Pattern matching
The smart pipelines and topic references of the [Core Proposal][] would be a
boon to the proposal for [ECMAScript pattern matching][].

<table>
<thead>
<tr>
<th>With smart pipelines
<th>With pattern matching only

<tbody>
<tr>
<td>

```js
…
|> f
|> match (#) {
  100:
    #,
  Array:
    #.length,
  /(\d)(\d)(\d)/:
    #.groups |> #[0] + #[1] + #[2],
}
```
The `match` expression would **bind the topic reference** within the scope of a
successfully matching clause’s scope. The topic value would be the **truthy
result** of the successful **`Symbol.matches`** call.

<td>

```js
match (f(…)) {
  100:
    x,
  Array -> a:
    x.length,
  /(\d)(\d)(\d)/ -> m:
    m.groups |> #[0] + #[1] + #[2],
}
```
With a topic binding, the `-> a` and `-> m` bindings would be unnecessary.

<tr>
<td>

```js
…
|> f
|> match {
  { x, y }:
    (x ** 2 + y ** 2)
    |> Math.sqrt,
  [...]:
    #.length,
  else:
    throw new Error(#),
}
```
[ECMAScript pattern matching][] could also have a **completely tacit version**,
in which the parenthesized antecedent (`(#)` in `match (#)`) is completely
omitted in favor of tacitly using the outer context’s topic.

<td>

```js
match (f(…)) {
  { x, y }:
    (x ** 2 + y ** 2)
    |> Math.sqrt,
  [...] -> a:
    a.length,
  else:
    throw new Error(vector),
}
```

<tr>
<td>

```js
try { … }
catch
|> match {
  SyntaxError:
  |> f,
  TypeError:
  |> g |> h(#, {strict: true}),
  Error:
  |> throw #,
};
```
With ECMAScript pattern matching + the [Core Proposal][] + [Additional
Feature PP][] + [Additional Feature TS][], handling caught errors (and promise
rejections) based on error type becomes more ergonomic.

<td>

```js
try { … }
catch (error) {
  match (error) {
    SyntaxError:
      f(error),
    TypeError:
      h(g(error), {strict: true}),
    Error:
      throw error,
  };
}
```
The version with pattern matching alone is more verbose with five more `error`
variables, distracting the reader from how the error is actually handled.

</table>

<tr>

<th>

## Block parameters
The proposed syntax of [ECMAScript block parameters][] may greatly benefit from
the pipeline and topic concepts, which would be able to explain much of their
desired behavior.

<table>
<thead>
<tr>
<th>With smart pipelines
<th>With block parameters only

<tbody>
<tr>
<td>

<tr>
<td>

```js
materials.map { |> f |> .length; }
```
This example uses [Additional Feature PP][]. The first parameter of the arrow
function that the block parameter implicitly creates is bound to the topic,
which in turn is fed into the pipeline `|> f |> .length`.

<td>

```js
materials.map do (m) { f(m).length; }
```

The block-parameter proposal itself has not yet settled on how to parameterize
its block parameters. The topic reference may be the key to solving this
problem, making other, special block parameters unnecessary.

<tr>
<td>

Note that this would be the same as:
```js
materials.map(+> f |> .length);
```

<td>

```js
materials.map (m) { f(m).length; };
```

<tr>
<td>

```js
a(1) {
  #::b(2) { … };
};
```
The block-parameter proposal in particular has not settled on how to nest block
parameters. But the [Core Proposal][]’s simple syntax rules already can handle
nested block parameters.

<td>

```js
a(1) {
  ::b(2) { … };
};
```
The block-parameter proposal’s authors have been exploring using a sygil –
perhaps related to the [function bind operator `::`][function binding] – in
order to refer to the parent block param, using some to-be-determined symbolic
magic that would be unnecessary with topic references.

<tr>
<td>

This would simply be equivalent to:
```js
a(1, +> {
  #::b(2, +> { … });
});
```

<td>

```js
a(1) {
  ::b(2) { … };
};
```

<tr>
<td>

…and if the [function bind operator `::`][function binding]’s proposal switches
its prefix form `::function` to mean `#::function`, then this can become even
terser, without additional magic:

```js
a(1) {
  ::b(2) { … };
};
```

<td>

```js
a(1) {
  ::b(2) { … };
};
```

<tr>
<td>

```js
server(app) {
  #::get('/') do (response) {
    request()
    |> .get('param1')
    |> `hello world ${#}`
    |> response.send;
  };

  #::listen(3000) {
    log('hello');
  };
}
```
And again, if the [function bind operator `::`][function binding]’s proposal
switches its prefix form `::function` to mean `#::function`, then `#::get` and
`#::listen` could become simply `::get` and `::listen`.

<td>

```js
server(app) do (_app) {
  _app::get('/') do (response) {
    request()
    |> #.get('param1')
    |> `hello world ${#}`
    |> response.send;
  };

  _app::listen(3000) {
    log('hello');
  };
};
```

</table>

### Topic metaprogramming references
In the event that TC39 seriously considers the topic function definitions
shown above, a **`function.topic`** metaprogramming operator, in the style of
the [`new.target`][] operator, could be useful in creating topic-aware functions.

This might be especially useful in creating APIs resembling [domain-specific
languages][DSLs] with [ECMAScript block parameters][]. This example creates
three functions that form an API resembling [Visual Basic’s `select`
statement][]. Two of these functions (`when` and `otherwise`) that are expected
to be called always within the third function (`select`)’s callback block.

Such a solution is not yet specified by the current proposal for [ECMAScript
block parameters][]. Lexical topics can fill in that gap. (The example below
also uses [Additional Feature BC][].)

```js
class CompletionRecord {
  type, value;

  constructor (testValue) {
    this.testValue = testValue;
  }
}

export function select (value, block) {
  const selectBlockTopic = new CompletionRecord();
  return block(topic)
  |> match {
    CompletionRecord:
      #.value,
    else:
      throw 'Invalid clause was used in select block'
      |> new Error,
  };
}

export function otherwise (block) {
  return function.topic
  |> match (selectBlockTopic) {
    CompletionRecord: {
      if (#.type === undefined) {
        #.type = 'normal';
        #.value = {};
      }
      #;
    },
    else:
      throw 'Invalid otherwise clause was used outside select block'
      |> new Error,
  };
}

export function when (caseValue, block) {
  return function.topic
  |> match (selectBlockTopic) {
    CompletionRecord:
    |> applyWhen(#, caseValue, block),
    else:
      throw 'Invalid when clause used was outside select block'
      |> new Error,
  };
}

function applyWhen (selectBlockTopic, caseValue, block) {
  return match (#.value) {
    [...]:
      (selectBlockTopic, caseValue, block)
      |> applyWhenArray,
    else:
      (selectBlockTopic, caseValue, block)
      |> applyWhenValue,
  }
}

function applyWhenArray (selectBlockTopic, testArray, block) {
  return testArray.some(arrayValue =>
    contextTopic |> {
      if (when(arrayValue, block))
        #;
    });
}

function applyWhenValue (contextTopic, testValue, block) {
  return contextTopic |> {
    let match
    if (#.type !== undefined
      && (match = #[Symbol.matches](testValue))
    ) {
      #.type = 'normal';
      #.value = match |> block;
    }
    #;
  };
}
```
***
```js
select ('world') {
  when ([Boolean, Number]) {
  |> log;
  };
  when (String) {
  |> `Hello ${#}`
  |> log;
  };
  otherwise {
    throw `Error: ${|> format}`
    |> new Error;
  };
};
```

## `do` expressions
Because pipeline [topic style][] supports [arbitrary expressions][expressive
versatility], when [`do` expressions][] are added to JavaScript they too will be
supported within pipeline bodies. When this occurs, topic references would be
allowed within inner `do` expressions, along with arrow functions and `if` and
`try` statements. [Additional Feature BP][] would extend this further, also
supporting pipeline-body blocks that act nearly identically to `do` expressions.

## Private class fields, class decorators, nullish coalescing, and optional chaining
This proposal’s compatibility with these four proposals depends on its choice of
tokens for its topic references, such as `#`/`##`/`###`/`...`, `@`/`@@`/`@@@`,
or `?`/`??`/`???`. This is being bikeshedded at [tc39/proposal-pipeline-operator
issue #91][topic-token bikeshedding].

Because topic proposals are nullary operators, these are unambiguous with all
four proposals, with one exception: `?`/`??`/`???` is not compatible with
[nullish coalescing and optional chaining’s current choice of `… ??: …`,
`…??.…`, `…??[…]`, and `…??(…)`][`??:`]. This is not a problem with
`#`/`##`/`###`/`...` and with `@`/`@@`/`@@@`.

In fact, if [Additional Feature PF][] and [Additional Feature NP][] subsume the
current [partial function application][] proposal, which uses nullary `?`, then
single `?` might be freed up for optional chaining.

## Alternative pipeline Babel plugin
[Gajus Kuizinas wrote a Babel plugin for syntactic functional composition][gajus
functional composition], which may be useful to compare with this proposal’s
smart pipelines. Kuizinas’s plugin makes the choice to insert its pipeline
heads’ values into its pipeline bodies’ last parameter, which is convenient for
Ramda-style functions but not for Underscore-style functions.

<table>
<thead>
<tr>
<th>With smart pipelines
<th>Kuizinas’ Babel plugin

<tbody>
<tr>
<td>

```js
apple
|> foo('foo parameter 0', 'foo parameter 1', #)
|> bar('bar parameter 0', #)
|> baz('baz parameter 0', #);
```

<td>

```js
apple
::foo('foo parameter 0', 'foo parameter 1')
::bar('bar parameter 0')
::baz('baz parameter 0');
```
From [gajus/babel-plugin-transform-function-composition][gajus functional
composition].

<tr>
<td>

```js
{x: 'x'}
|> assocPath(['y', 'a'], 'a', #)
|> assocPath(['y', 'b'], 'b', #);
```

<td>

```js
{x: 'x'}
::assocPath(['y', 'a'], 'a')
::assocPath(['y', 'b'], 'b');
```

</table>

## Alternative pipeline proposals
There are several other alternative pipeline-operator proposals competing with
the smart-pipeline Core Proposal. The Core Proposal is only one variant of the
[first pipeline-operator proposal][] also championed by Ehrenberg; this variant
is listed as [**Proposal 4: Smart Mix** in the pipe-proposal wiki][Pipeline
Proposal 4]. The variant resulted from [previous discussions in the previous
pipeline-operator proposal][previous pipeline-placeholder discussions],
discussions which culminated in an [invitation by Ehrenberg to try writing a
specification draft][littledan invitation].

All variants attempt to address the goals of [untangled flow][],
[distinguishable punctuators][], [terse function calls][], and [human
writability][]. But the authors of this proposal believe that the smart pipeline
operator may be the best choice among these competing proposals at fulfilling
all the [goals][] listed above.

Only the smart pipeline operator does not need to create unnecessary one-off
arrow functions for non-function-call expressions, which better fulfills the
goal of [zero runtime cost][]. Only the smart pipeline operator has the [forward
compatibility][] and [conceptual generality][] to support not only [terse unary
function application][terse function calls] but also [terse N-ary function
application][terse function calls], [terse expression application][terse
function calls], [terse function composition][terse composition], [terse
expression composition][terse composition], [terse partial application][], and
[terse method extraction][] – all with a single [simple][cyclomatic simplicity]
and unified [general concept][conceptual generality].

Indeed, the original pipeline proposal was blocked from Stage 2 by TC39 during
its [60th meeting, on September 2017][TC39 60th meeting, pipelines], for similar
reasons. At that time, several members expressed concern that it could be
coordinated more with the proposals for [function binding][] and [partial
function application][] in a more coherent approach. Smart pipelines open the
door to such an approach to all these use cases.

Smart pipelines and their [smart body syntax][] sacrifice a small amount of
[simplicity][cyclomatic simplicity] in return for a vast amount of [expressive
versatility][] and [conceptual generality][]. And because it makes many of the
other operator proposals above either unnecessary or possibly simpler, it may
result in less complexity on average anyway. And thanks to its [syntactic
locality][] and numerous [statically detectable early errors][early errors], the mental
burden on the developer in remembering [smart body syntax][] is light.

The benefits of smart pipelines on many real-world examples are well
demonstrated in the [Motivation][] section above, and many of the examples are
not possible with the other pipeline proposals. It is hoped that the Core
Proposal is strongly considered by TC39, keeping in mind that its simple but
versatile syntax would open the door to addressing the use cases of many other
proposals in a uniform manner.

# Appendices
## Smart body syntax
Most pipelines will use topic references in their bodies. This style of pipeline
is called **[topic style][]**.

For three simple cases – unary functions, unary async functions, and unary
constructors – you may omit the topic reference from the body. This is called
**[bare style][]**.

When a pipe is in bare style, we refer to the pipeline as a **bare function
call**. (If [Additional Feature BC][] or [Additional Feature BA][] are used,
then a bare-style pipeline body may instead be a **bare async function call**,
or a **bare constructor call**, depending on the rules of bare style.) The body
acts as just a simple reference to a function, such as with `… |> capitalize` or
with `… |> console.log`. The body’s value would then be called as a unary
function, without having to use the topic reference as an explicit argument.

The two bare-style productions require no parameters, because they can only be
**simple references**, made up of identifiers and dots `.`. (If [Additional
Feature BC][] is used, then the simple reference may optionally be preceded by
`new`: `… |> new o.C`. If [Additional Feature BA][] is used, then the simple
reference may optionally be preceded by `await`: `… |> new o.af`. Even with
Additional Features BC or BA, `new` and `await` may not be used on their own
without a simple reference: `… |> o.C |> new` 🚫 and `… |> o.af |> await` 🚫 are
invalid pipelines. Instead, use either the bare style `… |> new o.C` and
`… |> await o.af`, or use topic style: `… |> af |> await #`.

With the [Core Proposal][] only:

| Valid [topic style][]   | Valid [bare style][]                     | Invalid pipeline
| ----------------------- | ---------------------------------------- | --------------------
|`… \|> f(#)`             |`… \|> f`                                 |  `… \|> f()` 🚫
| ″″                      | ″″                                       | `… \|> (f)` 🚫
| ″″                      | ″″                                       | `… \|> (f())` 🚫
|`… \|> o.f(#)`           |`… \|> o.f`                               | `… \|> o.f()` 🚫
|`… \|> o.f(arg, #)`      |`const f = $ => o::f(arg, $); … \|> f`    | `… \|> o.f(arg)` 🚫
|`… \|> o.make()(#)`      |`const f = o.make(); … \|> f`             | `… \|> o.make()` 🚫
|`… \|> o[symbol](#)`     |`const f = o[symbol]; … \|> f`            | `… \|> o[symbol]` 🚫

With [Additional Feature BC][]:

| Valid [topic style][]   | Valid [bare style][]                     | Invalid pipeline
| ----------------------- | ---------------------------------------- | --------------------
|`… \|> new C(#)`         |`… \|> new C`                             | `… \|> new C()` 🚫
| ″″                      | ″″                                       | `… \|> (new C)` 🚫
| ″″                      | ″″                                       | `… \|> (new C())` 🚫
| ″″                      | ″″                                       | `… \|> new (C)` 🚫
| ″″                      | ″″                                       | `… \|> new (C())` 🚫
|`… \|> new o.C(#)`       |`… \|> new o.C`                           | `… \|> new o.f()` 🚫
|`… \|> new o.C(arg, #)`  |`const f = $ => new o::C(arg, $); … \|> f`| `… \|> new o.C(arg)` 🚫
|`… \|> new o.make()(#)`  |`const C = o.make(); … \|> new C`         | `… \|> new o.make()` 🚫
|`… \|> new o[symbol](#)` |`const f = new o[symbol]; … \|> f`        | `… \|> new o[symbol]` 🚫
|`… \|> await new o.make()(#)`|`const af = new o.make(); … \|> await af`| `… \|> new await o.make()` 🚫

With [Additional Feature BA][]:

| Valid [topic style][]   | Valid [bare style][]                     | Invalid pipeline
| ----------------------- | ---------------------------------------- | --------------------
|`… \|> await af(#)`      |`… \|> await af`                          | `… \|> await af()` 🚫
| ″″                      | ″″                                       | `… \|> (await f)` 🚫
| ″″                      | ″″                                       | `… \|> (await f())` 🚫
| ″″                      | ″″                                       | `… \|> await (f)` 🚫
| ″″                      | ″″                                       | `… \|> await (f())` 🚫
|`… \|> af \|> await #`   | ″″                                       |  `… \|> af \|> await` 🚫
|`… \|> await o.f(#)`     |`… \|> await o.f`                         | `… \|> await o.f()` 🚫
|`… \|> await o.make()(#)`|`const af = o.make(); … \|> await af`     | `… \|> await o.make()` 🚫
|`… \|> await new o.make()(#)`|`const af = new o.make(); … \|> await af`| `… \|> new await o.make()` 🚫

### Bare style
The **bare style** supports using simple identifiers, possibly with chains of
simple property identifiers. If there are any operators, parentheses (including
for method calls), brackets, or anything other than identifiers and dot
punctuators, then it is in [topic style][], not in bare style.

If the body is a merely a simple reference, then that identifier is interpreted
to be a **bare function call**. The pipeline’s value will be the result of
calling the body with the current topic as its argument.

That is: **if a pipeline** is of the form\
**_topic_ `|>` _identifier_**\
or **_topic_ `|>` _identifier0_`.`_identifier1_**\
or **_topic_ `|>` _identifier0_`.`_identifier1_`.`_identifier2_**\
or so forth,\
then the pipeline is a bare function call.

**With [Additional Feature BC][]:**\
If a pipeline body starts with `new`, followed by a mere identifier, optionally
with a chain of properties, and with no parentheses or brackets, then that
identifier is interpreted to be a **bare constructor**.

That is: **if a pipeline** is of the form\
**_topic_ `|>` `new` _identifier_**\
or **_topic_ `|>` `new` _identifier0_`.`_identifier1_**\
or **_topic_ `|>` `new` _identifier0_`.`_identifier1_`.`_identifier2_**\
or so forth,\
then the pipeline is a bare constructor call.

**With [Additional Feature BA][]:**\
If a pipeline body starts with `await`, followed by a mere identifier, optionally with
a chain of properties, and with no parentheses or brackets, then that identifier
is interpreted to be a **bare awaited function call**.

That is: **if a pipeline** is of the form\
**_topic_ `|>` `await` _identifier_**\
or **_topic_ `|>` `await` _identifier0_`.`_identifier1_**\
or **_topic_ `|>` `await` _identifier0_`.`_identifier1_`.`_identifier2_**\
or so forth,\
then the pipeline is a bare async function call.

### Topic style
The presence or absence of topic tokens (`#`, `##`, `###`) is *not* used in the
grammar to distinguish topic style from bare style to fulfill the goal of
[syntactic locality][]. Instead, **if a pipeline** of the form _topic_
|> _body_ does ***not* match the [bare style][]** (that is, it is *not* a bare
function call, bare async function call, or bare constructor call), then it
**must be in topic style**. Topic style **requires** that there be a topic
reference in the pipeline body; otherwise it is an [early error][].

A pipeline body that is not in bare style is usually a **topic expression**.
This is any expression at the [precedence level once tighter than pipeline-level
expressions][operator precedence] – that is, any conditional-level expression.

**With [Additional Feature BA][]:**\
A pipeline body that is a block `{` … `}` containing a list of statements is a
**topic block**. The last statement in the block is used as the result of the
whole pipeline, similarly to [`do` expressions][].

### Practical consequences
Therefore, a pipeline in **[bare style][] *never*** has **parentheses `(…)` or
brackets `[…]`** in its body. Neither `… |> object.method()` nor
`… |> object.method(arg)` nor `… |> object[symbol]` nor `… |> object.createFunction()`
are in bare style (in fact, they all are Syntax Errors, due to their being in
[topic style][] without any topic references).

**When a body needs parentheses or brackets**, then **don’t use bare style**,
and instead **use a topic reference** in the body ([topic style][])…or **assign
the body to a variable**, then **use that variable as a bare body**.

## Operator precedence and associativity
As a infix operation forming compound expressions, the [operator precedence and
associativity][MDN operator precedence] of pipelining must be determined, relative
to other operations.

Precedence is tighter than arrow functions (`=>`), assignment (`=`, `+=`, …),
generator `yield` and `yield *`, and sequence `,`; and it is looser than every
other type of expression. If the pipe operation were any tighter than this
level, its body would have to be parenthesized for many frequent types of
expressions. However, the result of a pipeline is also expected to often serve
as the body of an arrow function or a variable assignment, so it is tighter than
both types of expressions.

All operation-precedence levels in JavaScript are listed here, from **tightest
to loosest**. Each level may contain the parse types listed for that level –
**as well as** any expression types from any precedence level that is listed
**above** it.

| Level          | Type                    | Form           | Associativity / fixity   |
| -------------- | ----------------------- | -------------- | ------------------------ |
| Primary        | This                    |`this`          | Nullary                  |
| ″″             | **[Primary topic][]**   |**`#`**         | ″″                       |
| ″″             | **[Secondary topic][]** |**`##`**        | ″″                       |
| ″″             | **[Tertiary topic][]**  |**`###`**       | ″″                       |
| ″″             | **[Rest topic][]**      |**`...`**       | ″″                       |
| ″″             | Identifiers             |`a` …           | ″″                       |
| ″″             | Null                    |`null`          | ″″                       |
| ″″             | Booleans                |`true` `false`  | ″″                       |
| ″″             | Numerics                |`0` …           | ″″                       |
| ″″             | Arrays                  |`[…]`           | Circumfix                |
| ″″             | Object                  |`{…}`           | ″″                       |
| ″″             | Function                |`function (…) {…}`| ″″                     |
| ″″             | Classes                 |`class … {…}`   | ″″                       |
| ″″             | Generators              |`function * (…) {…}`| ″″                   |
| ″″             | Async functions         |`async function (…) {…}`| ″″               |
| ″″             | Regular expression      |`/…/…`          | ″″                       |
| ″″             | Templates               |`` …`…` ``      | ″″                       |
| ″″             | Parentheses             |`(…)`           | Circumfix                |
| LHS            | Static properties       |`….…`           | ″″                       |
| ″″             | Dynamic properties      |`…[…]`          | LTR infix with circumfix |
| ″″             | Tagged templates        |`` …`…` ``      | Unchainable infix with circumfix|
| ″″             | Super call op.s         |`super(…)`      | Unchainable prefix       |
| ″″             | Super properties        |`super.…`       | ″″                       |
| ″″             | Meta properties         |`meta.…`        | ″″                       |
| ″″             | Object construction     |`new …`         | Prefix                   |
| ″″             | Function call           |`…(…)`          | LTR infix with circumfix |
| Postfix unary  | Postfix incrementing    |`…++`           | Postfix                  |
| ″″             | Postfix decrementing    |`…--`           | ″″                       |
| Prefix unary   | Prefix incrementing     |`++…`           | RTL prefix               |
| Prefix unary   | Prefix decrementing     |`--…`           | ″″                       |
| ″″             | Deletes                 |`delete …`      | ″″                       |
| ″″             | Voids                   |`void …`        | ″″                       |
| ″″             | Unary `+`/`-`           |`+…`            | ″″                       |
| ″″             | Bitwise NOT `~…`        |`~…`            | ″″                       |
| ″″             | Logical NOT `!…`        |`!…`            | ″″                       |
| ″″             | Awaiting                |`await …`       | ″″                       |
| Exponentiation | Exponentiation          |`… ** …`        | RTL infix                |
| Multiplicative | Multiplication          |`… * …`         | LTR infix                |
| ″″             | Division                |`… / …`         | ″″                       |
| ″″             | Modulus                 |`… % …`         | ″″                       |
| Additive       | Addition                |`… + …`         | ″″                       |
| ″″             | Subtraction             |`… - …`         | ″″                       |
| Bitwise shift  | Left shift              |`… << …`        | ″″                       |
| ″″             | Right shift             |`… >> …`        | ″″                       |
| ″″             | Signed right shift      |`… >> …`        | ″″                       |
| Relational     | Greater than            |`… < …`         | ″″                       |
| ″″             | Less than               |`… > …`         | ″″                       |
| ″″             | Greater than / equal to |`… >= …`        | ″″                       |
| ″″             | Less than / equal to    |`… <= …`        | ″″                       |
| ″″             | Containment             |`… in …`        | ″″                       |
| ″″             | Instance-of             |`… instanceof …`| ″″                       |
| Equality       | Abstract equality       |`… == …`        | ″″                       |
| ″″             | Abstract inequality     |`… != …`        | ″″                       |
| ″″             | Strict equality         |`… === …`       | ″″                       |
| ″″             | Strict inequality       |`… !== …`       | ″″                       |
| Bitwise AND    |                         |`… & …`         | ″″                       |
| Bitwise XOR    |                         |`… ^ …`         | ″″                       |
| Bitwise OR     |                         |`… \| …`        | ″″                       |
| Logical AND    |                         |`… ^^ …`        | ″″                       |
| Logical OR     |                         |`… \|\| …`      | ″″                       |
| Conditional    |                         |`… ? … : …`     | RTL ternary infix        |
| Pipeline       | **[Pipelines][]**       |**`… \|> …`**   | LTR infix                |
| Assignment     | **[Pipeline functions][]**|**`+> …`**    | Prefix                   |
| ″″             | **[Async pipeline functions][]**|**`async +> …`**|Prefix            |
| ″″             | Arrow functions         |`… => …`        | RTL infix                |
| ″″             | Async arrow functions   |`async … => …`  | RTL infix                |
| ″″             | Assignment              |`… = …`         | ″″                       |
| ″″             |                         |`… += …`        | ″″                       |
| ″″             |                         |`… -= …`        | ″″                       |
| ″″             |                         |`… *= …`        | ″″                       |
| ″″             |                         |`… %= …`        | ″″                       |
| ″″             |                         |`… **= …`       | ″″                       |
| ″″             |                         |`… <<= …`       | ″″                       |
| ″″             |                         |`… >>= …`       | ″″                       |
| ″″             |                         |`… >>>= …`      | ″″                       |
| ″″             |                         |`… &= …`        | ″″                       |
| ″″             |                         |`… \|= …`       | ″″                       |
| Yield          | Yielding                |`yield …`       | Prefix                   |
| ″″             | Flat yielding           |`yield * …`     | ″″                       |
| ″″             | Spreading               |`...…`          | ″″                       |
| Comma level    | Comma                   |`…, …`          | LTR infix                |
| Base statements| Expression statements   |`…;`            | Postfix with [ASI][]     |
| ″″             | Empty statements        |`;`             | Nullary with [ASI][]     |
| ″″             | Debugger statements     |`debugger;`     | ″″                       |
| ″″             | Block statements        |`{…}`           | Circumfix                |
| ″″             | Labelled statements     |`…: …`          | Prefix                   |
| ″″             | Continue statements     |`continue …;`   | Circumfix with [ASI][]   |
| ″″             | Break statements        |`break …;`      | ″″                       |
| ″″             | Return statements       |`return …;`     | ″″                       |
| ″″             | Throw statements        |`throw …;`      | ″″                       |
| ″″             | Variable statements     |`var …;`        | ″″                       |
| ″″             | Lexical declarations    |`let …;`        | ″″                       |
| ″″             | ″″                      |`const …;`      | ″″                       |
| ″″             | Hoistable declarations  |`function … (…) {…}`| Circumfix with prefix|
| ″″             | ″″                      |`async function … (…) {…}`| ″″             |
| ″″             | ″″                      |`function * … (…) {…}`| ″″                 |
| ″″             | ″″                      |`async function * … (…) {…}`| ″″           |
| ″″             | Class declarations      |`class … {…}`   | ″″                       |
| Compound statements| If statements       |`if (…) … else …`| Circumfix with prefix   |
| ″″             | Switch statements       |`switch (…) …`  | ″″                       |
| ″″             | Iteration statements    |                | ″″                       |
| ″″             | With statements         |`with (…) {…}`  | ″″                       |
| ″″             | Try statements          |`try {…} catch (…) {…} finally {…}` | ″″   |
| Statement list | Case clause             |`case: …`       | Unchainable prefix       |
| Root           | Script                  |                | Root                     |
| ″″             | Module                  |                | ″″                       |

## Explanation of nomenclature
The term [“**topic**” comes from linguistics][topic and comment] and have
precedent in prior programming languages’ use of “topic variables”.

The term “**head**” is preferred to “**topic expression**” because, in the
future, the topic concept could be extended to other syntaxes, as with
[Additional Feature TS][] and [Additional Feature FS][], not just pipelines.

In addition, “head” is preferred to “**LHS**”, because “LHS” in the ECMAScript
specification usually refers to the [LHS of assignments][ECMAScript LHS expressions],
which may be confusing. However, “topic expression” and “LHS” are still fine and
acceptable, if not imprecise, names for a pipeline’s head.

The term “**topic reference**” is preferred to the phrase “**topic variable**”
because the latter is a misnomer. The topic reference is *not* a variable
identifier. Unlike variables, it cannot be manually declared (`const #` is a
syntax error), nor can it be assigned with a value (`# = 3` is a syntax error).

“Topic reference” is also preferred to “**topic placeholder**”, to avoid
confusion with the placeholders of another TC39 proposal – [partial function
application][]. These placeholders (currently denoted by nullary `?`) are of a
different nature than topic references. Instead of referring to a single value
bound earlier in the surrounding lexical context, these **parameter
placeholders** act as the parameter to a new function. When this new function is
called, those parameter placeholders will be bound to multiple argument values.

The term “**body**” is preferred instead of “**RHS**” because “topic” is
preferred to “LHS”. However, “RHS” is still a fine and acceptable name for the
body of the pipeline operator.

“**[Bare style][]**” can also be called “**tacit style**”, but the former is
preferred to the latter. Eventually, certain possible future extensions to the
topic concept, such as [Additional Feature TS][] and [Additional Feature FS][],
would enable [tacit programming][] even without using bare-style pipelines.

## Term rewriting
### Core Proposal
Pipeline chains can be rewritten into a nested [`do` expression][]. There are
many ways to illustrate this equivalency. (It can also be less simply rewritten
without `do` expressions.) The simplest way is to use a single `do` expression
containing a series of autogenerated variable declarations, in which the
variables are arbitrary except they are [lexically hygienic][]: that is, the
variables can be anything as long as they do not conflict with other
already-existing variables.

With this notation, each line in this example would be equivalent to all the
other lines.
```js
1 |> # + 2 |> # * 3;

// Static term rewriting
do { const _0 = 1, _1 = _0 + 2; _1 * 3; };

// Runtime evaluation
do { const _1 = 1 + 2; _1 * 3; };
do { 3 * 3; };
9;
```

Consider also the [motivating first example above][Core Proposal]:
```js
promise
|> await #
|> # || throw new TypeError(
  `Invalid value from ${promise}`)
|> doubleSay(#, ', ')
|> capitalize
|> # + '!'
|> new User.Message(#)
|> await stream.write(#)
|> console.log;
```

This would be statically equivalent to the following:
```js
do {
  const
    _0 = promise,
    _1 = await _0,
    _2 = _1 || throw new TypeError(
      `Invalid value from ${promise}`),
    _3 = doubleSay(_2),
    _4 = capitalize(_3);
    _5 = _4 + '!',
    _6 = new User.Message(_5),
    _7 = await stream.write(_6);
  console.log(_7);
}
```
Suppose that we can generate a series of new [lexically hygienic][] variables
(`_0`, `_1`, `_2`, `_3`, …). Each autogenerated variable will replace every
topic reference `#` in each consecutive pipeline body. These `_n` variables
need only be unique within their lexical scopes: they must not shadow any
outer lexical environment’s variable, and they must not be shadowed by any
deeply inner lexical environment’s variable.

With this notation, then in general, given a pipeline chain:\
𝐸₀ `|>` 𝐸₁ `|>` 𝐸₂ `|>` … `|>` 𝐸ᵤ₋₂ `|>` 𝐸ᵤ₋₁,\
…then that pipeline chain is equivalent to:\
`do` `{`\
  `const `\
    #₀ `=` 𝐸₀ `,`\
    #₁ `=` sub(𝐸₁, #₀) `,`\
    #₂ `=` sub(𝐸₂, #₁) `,`\
    … `,`\
    #ᵤ₋₂ `=` sub(𝐸ᵤ₋₂, #ᵤ₋₃) `;`\
  sub(𝐸ᵤ₋₁, #ᵤ₋₂) `;`\
`}`,\

* If 𝑃 is a [bare function call][] – then sub(𝑃, #) is 𝑃 `(` # `)`.
* If 𝑃 is a [bare awaited function call][] – then sub(𝑃, #) is
  `await` 𝑃 `(` # `)`.
* If 𝑃 is a [bare constructor call][] – then sub(𝑃, #) is `new` 𝑃 `(` # `)`.
* If 𝑃 is in [topic style][] – then sub(𝑃, #) is 𝑃 but in which all unshadowed
  instances of the topic reference `#` are replaced by #.

### Additional Feature BP
Using the same notation from the first subsection, then in general:

* If 𝑃 is a [bare function call][] – then sub(𝑃, #) is 𝑃 `(` # `)`.
* If 𝑃 is a [bare awaited function call][] – then sub(𝑃, #) is
  `await` 𝑃 `(` # `)`.
* If 𝑃 is a [bare constructor call][] – then sub(𝑃, #) is `new` 𝑃 `(` # `)`.
* If 𝑃 is in [topic style][] – then sub(𝑃, #) is 𝑃 but in which all unshadowed
  instances of the topic reference `#` are replaced by #.
* **If 𝑃 is in the form `{` 𝑆₀ `;` 𝑆₁ `;` … `;` 𝑆ᵥ₋₂ `;` 𝑆ᵥ₋₁ `;` `}` – then sub(𝑃, #) is
  `do {` sub(𝑆₀, #) `;` sub(𝑆₁, #) `;` … `;` sub(𝑆ᵥ₋₂, #) `;` sub(𝑆ᵥ₋₁, #) `;` `}`**.

### Additional Feature PP
Using the same notation from the first subsection, then in general, given a
pipeline chain:\
`|>` 𝐸₁ `|>` 𝐸₂ `|>` … `|>` 𝐸ᵤ₋₂ `|>` 𝐸ᵤ₋₁,\
…then that pipeline chain is equivalent to:\
`do` `{`\
  `const `\
    **#₀ `=` `#` `,`**\
    #₁ `=` sub(𝐸₁, #₀) `,`\
    #₂ `=` sub(𝐸₂, #₁) `,`\
    … `,`\
    #ᵤ₋₂ `=` sub(𝐸ᵤ₋₂, #ᵤ₋₃) `;`\
  sub(𝐸ᵤ₋₁, #ᵤ₋₂) `;`\
`}`.

### Additional Feature NP
Adapted from a [previous example][Additional Feature NP]:
```js
x = (a, b, ...c, d, e)
|> f(##, x, ...)
|> g;
```
This would be statically equivalent to the following:
```js
x = do {
  const
    [_0, __0, ...s_0] = [a, b, ...c, d, e]
    _1 = f(__0, x, ...s_0);
  g(_1);
};
```

Another one:
```js
x = (a, b)
|> (f, # ** c + ##)
|> # - ##
|> g;
```
This would be statically equivalent to the following:
```js
x = do {
  const
    [_0, __0] = [a, b]
    [_1, __1] = [f(_0), _0 ** c + __0];
    _2 = _1 - __1;
  g(_2);
};
```

From a [previous Lodash example][Lodash + CP + BP + PP + PF + NP]:
```js
x = number
|> `${#}e`
|> ...#.split('e')
|> `${#}e${+## + precision}`
|> func;
```
This would be statically equivalent to the following:
```js
x = do {
  const
    _0 = number,
    _1 = `${_0}e`,
    [_2, __2] = [..._1.split('e')];
  func(_2, __2);
};
```
…which of course may be simplified to:
```js
x = do {
  const
    _0 = number,
    _1 = `${_0}e`,
    [_2, __2] = [..._1.split('e')];
  func(_2, __2);
}
```

From a [previous WHATWG Streams example][WHATWG Streams + CP + BP + PF + NP]
with [Additional Feature BC][]:
```js
x = value
|> (#, offset, #.byteLength - offset)
|> new Uint8Array
|> await reader.read
|> (#.buffer, #.byteLength)
|> readInto(#, offset + ##);
```
This would be statically equivalent to the following:
```js
x = do {
  const
    [_0] = [value],
    [_1, __1, ___1] = [_0, offset(_0), __0.byteLength - offset],
    _2 = new Uint8Array(_1),
    _3 = await reader.read(_2),
    [_4, __4] = [_3.buffer, _3.byteLength];
  readInto(_4, offset + __4);
};
```
…which of course may be simplified to:
```js
x = do {
  const
    _0 = value,
    _1 = _0,
    __1 = offset,
    ___1 = _0.byteLength - offset,
    _2 = new Uint8Array(_1),
    _3 = await reader.read(_2),
    _4 = _3.buffer,
    __4 = _3.byteLength;
  readInto(_4, offset + __4);
}
```

Using the same notation from the first subsection, then consider any
pipeline chain:\
𝐸₀ `|>` 𝐸₁ `|>` 𝐸₂ `|>` … `|>` 𝐸ᵤ₋₂ `|>` 𝐸ᵤ₋₁\
…in which, for each i from 0 until n−1, 𝐸ᵢ is either:

* A single expression 𝐸ᵢ[0] (which may start with `...`).
* Or an argument list `(` 𝐸ᵢ[0] `,` 𝐸ᵢ[1] `,` … `,` 𝐸ᵢ[width(𝐸ᵢ)−2] `,`
  𝐸ᵢ[width(𝐸ᵢ)−1] `)`, where each element of the argument list may be an
  expression, an expression starting with `...`, or a blank elision.

The last pipeline body, 𝐸ᵤ₋₁, is an exception: it must be a **single**
expression that does **not** start with `...`, and it cannot be a parenthesized
argument list either.

The pipeline chain is therefore equivalent to:\
`do` `{`\
  `const `\
    `[` #₀[0] `,` … `,` #₀[max topic index(𝐸₀)] `,` `...` #₀[r] `]` `=`\
      `[`\
        𝐸₀[0] `,`\
        𝐸₀[1] `,`\
        … `,`\
        𝐸₀[width(𝐸₀)−2]\
        𝐸₀[width(𝐸₀)−1]\
    `]` `,`\
    `[` #₁[0] `,` … `,` #₁[max topic index(𝐸₁)] `,` `... ` #₁[r] `]` `=`\
      `[`\
          sub(𝐸₁[0], #₀[0], #₀[1], #₀[2], #₀[r]) `,`\
          sub(𝐸₁[1], #₀[0], #₀[1], #₀[2], #₀[r]) `,`\
          …\
          sub(𝐸₁[width(𝐸₁)−2], #₀[0], #₀[1], #₀[2], #₀[r]) `,`\
          sub(𝐸₁[width(𝐸₁)−1], #₀[0], #₀[1], #₀[2], #₀[r]) `,`\
      `]` `,`\
    `[` #₂[0] `,` … `,` #₂[max topic index(𝐸₂)] `,` `... ` #₂[r] `]` `=`\
      `[`\
          sub(𝐸₂[0], #₀[0], #₀[1], #₀[2], #₀[r]) `,`\
          sub(𝐸₂[1], #₀[0], #₀[1], #₀[2], #₀[r]) `,`\
          …\
          sub(𝐸₂[width(𝐸₂)−2], #₀[0], #₀[1], #₀[2], #₀[r]) `,`\
          sub(𝐸₂[width(𝐸₂)−1], #₀[0], #₀[1], #₀[2], #₀[r]) `,`\
      `]` `,`\
    … `,`\
    `[` #ᵤ₋₂[0] `,` #ᵤ₋₂[1] `,` … `,` `... ` (#ᵤ₋₂)ₛ `]` `=`\
      `[`\
          sub(𝐸ᵤ₋₂[0], #₀[0], #₀[1], #₀[2], #₀[r]) `,`\
          sub(𝐸ᵤ₋₂[1], #₀[0], #₀[1], #₀[2], #₀[r]) `,`\
          … `,`\
          sub(𝐸ᵤ₋₂[width(𝐸ᵤ₋₂)−2], #₀[0], #₀[1], #₀[2], #₀[r]) `,`\
          sub(𝐸ᵤ₋₂[width(𝐸ᵤ₋₂)−1], #₀[0], #₀[1], #₀[2], #₀[r]) `,`\
      `]` `;`\
  sub(𝐸ᵤ₋₁, #ᵤ₋₂[0], #ᵤ₋₂[1], …, #ᵤ₋₂[width(𝐸ᵤ₋₂)−1]) `;`\
`}`.

***

[TODO: Define width(𝐸) and max topic index(𝐸).]

***

* If 𝑃 is a [bare function call][] – then sub(𝑃, #[0], #[1], …, #[m]) is
  𝑃 `(` [TODO] `)`.
* If 𝑃 is a [bare awaited function call][] – then sub(𝑃, #[0], #[1], …, #[m])
  is `await` 𝑃 `(` [TODO] `)`.
* If 𝑃 is a [bare constructor call][] – then sub(𝑃, #[0], #[1], …, #[m]) is
  `new` 𝑃 `(` [TODO] `)`.
* [TODO] If 𝑃 is in [topic style][] – then sub(𝑃, #[0], #[1], #[2], #ₛ) is 𝑃 but in which
  all unshadowed instances of the primary topic reference `#` are replaced by
  #[0], unshadowed instances of the secondary topic reference `##` are replaced
  by #[1], unshadowed instances of the tertiary topic reference `###` are
  replaced by #[2], and unshadowed instances of the rest topic reference `...`
  are replaced by `...` #ₛ.

[“data-to-ink” visual ratio]: https://www.darkhorseanalytics.com/blog/data-looks-better-naked
[“don’t break my code”]: #dont-break-my-code
[“don’t make me overthink”]: #dont-make-me-overthink
[“don’t shoot me in the foot”]: #dont-shoot-me-in-the-foot
[“make my code easier to read”]: #make-my-code-easier-to-read
[`??:`]: https://github.com/tc39/proposal-nullish-coalescing/pull/23
[`do` expression]: #do-expressions
[`do` expressions]: #do-expressions
[`for` iteration statements]: https://tc39.github.io/ecma262/#sec-iteration-statements
[`in` relational operator]: https://tc39.github.io/ecma262/#sec-relational-operators
[`match` expressions]: #pattern-matching
[`new.target`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/new.target
[Abstract: Get Topic Environment]: #abstract-get-topic-environment
[Additional Feature BA]: #additional-feature-ba
[Additional Feature BC]: #additional-feature-bc
[Additional Feature BP]: #additional-feature-bp
[Additional Feature FS]: #additional-feature-fs
[Additional Feature NP]: #additional-feature-np
[Additional Feature PF]: #additional-feature-pf
[Additional Feature PP]: #additional-feature-pp
[Additional Feature TS]: #additional-feature-ts
[additional features]: #smart-pipelines
[annevk]: https://github.com/annevk
[antecedent]: https://en.wikipedia.org/wiki/Antecedent_(grammar)
[arbitrary associativity]: #arbitrary-associativity
[ASI]: https://tc39.github.io/ecma262/#sec-automatic-semicolon-insertion
[associative property]: https://en.wikipedia.org/wiki/Associative_property
[async pipeline functions]: #additional-feature-pf
[Babel plugin]: https://github.com/valtech-nyc/babel/
[Babel update summary]: https://github.com/babel/proposals/issues/29#issuecomment-372828328
[background]: #background
[backward compatibility]: #backward-compatibility
[bare awaited function call]: #bare-style
[bare constructor call]: #bare-style
[bare function call]: #bare-style
[bare style]: #bare-style
[binding]: https://en.wikipedia.org/wiki/Binding_(linguistics)
[block parameters]: #block-parameters
[Clojure compact function]: https://clojure.org/reference/reader#_dispatch
[Clojure pipe]: https://clojuredocs.org/clojure.core/as-%3E
[completion records]: https://timothygu.me/es-howto/#completion-records-and-shorthands
[concatenative programming]: https://en.wikipedia.org/wiki/Concatenative_programming_language
[conceptual generality]: #conceptual-generality
[Contains]: #static-contains
[Core Proposal]: #core-proposal
[currying]: https://en.wikipedia.org/wiki/Currying
[cyclomatic complexity]: https://en.wikipedia.org/wiki/Cyclomatic_complexity#Applications
[cyclomatic simplicity]: #cyclomatic-simplicity
[dataflow programming]: https://en.wikipedia.org/wiki/Dataflow_programming
[distinguishable punctuators]: #distinguishable-punctuators
[don’t break my code]: #dont-break-my-code
[DSLs]: https://en.wikipedia.org/wiki/Domain-specific_language
[early error rule]: #static-analyzability
[early error rules]: #static-analyzability
[early error]: #static-analyzability
[early errors]: #static-analyzability
[ECMAScript _Identifier Name_]: https://tc39.github.io/ecma262/#prod-IdentifierName
[ECMAScript _Identifier Reference_]: https://tc39.github.io/ecma262/#prod-IdentifierReference
[ECMAScript _Member Expression_]: https://tc39.github.io/ecma262/#prod-MemberExpression
[ECMAScript § The Syntactic Grammar]: https://tc39.github.io/ecma262/#sec-syntactic-grammar
[ECMAScript `new` operator, § RS: Evaluation]: https://tc39.github.io/ecma262/#sec-new-operator-runtime-semantics-evaluation
[ECMAScript arrow functions, § SS: Contains]: https://tc39.github.io/ecma262/#sec-arrow-function-definitions-static-semantics-contains
[ECMAScript Assignment-level Expressions]: https://tc39.github.io/ecma262/#sec-assignment-operators
[ECMAScript block parameters]: https://github.com/samuelgoto/proposal-block-params
[ECMAScript Blocks, § RS: Block Declaration Instantiation]: https://tc39.github.io/ecma262/#sec-blockdeclarationinstantiation
[ECMAScript Blocks, § RS: Evaluation]: https://tc39.github.io/ecma262/#sec-block-runtime-semantics-evaluation
[ECMAScript Declarative Environment Records]: https://tc39.github.io/ecma262/#sec-declarative-environment-records
[ECMAScript function binding]: https://github.com/zenparsing/es-function-bind
[ECMAScript Function Calls, § RS: Evaluation]: https://tc39.github.io/ecma262/#sec-function-calls-runtime-semantics-evaluation
[ECMAScript Function Environment Records]: https://tc39.github.io/ecma262/#sec-function-environment-records
[ECMAScript Functions and Classes § Generator Function Definitions]: https://tc39.github.io/ecma262/#sec-generator-function-definitions
[ECMAScript Get This Environment]: https://tc39.github.io/ecma262/#sec-getthisenvironment
[ECMAScript headless-arrow proposal]: https://bterlson.github.io/headless-arrows/
[ECMAScript Lexical Environments]: https://tc39.github.io/ecma262/#sec-lexical-environments
[ECMAScript Lexical Grammar]: https://tc39.github.io/ecma262/#sec-ecmascript-language-lexical-grammar
[ECMAScript LHS expressions]: https://tc39.github.io/ecma262/#sec-left-hand-side-expressions
[ECMAScript Lists and Records]: https://tc39.github.io/ecma262/#sec-list-and-record-specification-type
[ECMAScript Notational Conventions, § Algorithm Conventions]: https://tc39.github.io/ecma262/#sec-algorithm-conventions-abstract-operations
[ECMAScript Notational Conventions, § Grammars]: https://tc39.github.io/ecma262/#sec-syntactic-and-lexical-grammars
[ECMAScript Notational Conventions, § Lexical Grammar]: https://tc39.github.io/ecma262/#sec-lexical-and-regexp-grammars
[ECMAScript Notational Conventions, § Runtime Semantics]: https://tc39.github.io/ecma262/#sec-runtime-semantics
[ECMAScript optional `catch` binding]: https://github.com/tc39/proposal-optional-catch-binding
[ECMAScript partial application]: https://github.com/tc39/proposal-partial-application
[ECMAScript pattern matching]: https://github.com/tc39/proposal-pattern-matching
[ECMAScript Primary Expressions]: https://tc39.github.io/ecma262/#prod-PrimaryExpression
[ECMAScript Property Accessors, § RS: Evaluation]: https://tc39.github.io/ecma262/#sec-property-accessors-runtime-semantics-evaluation
[ECMAScript Punctuators]: https://tc39.github.io/ecma262/#sec-punctuators
[ECMAScript static semantic rules]: https://tc39.github.io/ecma262/#sec-static-semantic-rules
[Elixir pipe]: https://hexdocs.pm/elixir/Kernel.html#%7C%3E/2
[Elm pipe]: http://elm-lang.org/docs/syntax#infix-operators
[essential complexity]: https://en.wikipedia.org/wiki/Essential_complexity
[examples]: #examples
[expressions and operators (MDN)]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Expressions_and_Operators
[expressive versatility]: #expressive-versatility
[F# pipe]: https://docs.microsoft.com/en-us/dotnet/fsharp/language-reference/functions/index#function-composition-and-pipelining
[first pipeline-operator proposal]: https://github.com/tc39/proposal-pipeline-operator/blob/37119110d40226476f7af302a778bc981f606cee/README.md
[footguns]: https://en.wiktionary.org/wiki/footgun
[formal BA]: https://jschoi.org/18/es-smart-pipelines/spec#sec-additional-feature-ba
[formal BC]: https://jschoi.org/18/es-smart-pipelines/spec#sec-additional-feature-bc
[formal BP]: https://jschoi.org/18/es-smart-pipelines/spec#sec-additional-feature-bp
[formal CP]: https://jschoi.org/18/es-smart-pipelines/spec
[formal FS]: https://jschoi.org/18/es-smart-pipelines/spec#sec-additional-feature-fs
[formal grammar]: #grammar
[formal NP]: https://jschoi.org/18/es-smart-pipelines/spec#sec-additional-feature-np
[formal PF]: https://jschoi.org/18/es-smart-pipelines/spec#sec-additional-feature-pf
[formal pipeline specification]: https://jschoi.org/18/es-smart-pipelines/spec
[formal PP]: https://jschoi.org/18/es-smart-pipelines/spec#sec-additional-feature-pp
[formal TS]: https://jschoi.org/18/es-smart-pipelines/spec#sec-additional-feature-ts
[forward compatibility]: #forward-compatibility
[forward compatible]: #forward-compatibility
[function bind operator `::`]: #function-bind-operator
[function binding]: #function-binding
[function composition]: #function-composition
[functional programming]: https://en.wikipedia.org/wiki/Functional_programming
[gajus functional composition]: https://github.com/gajus/babel-plugin-transform-function-composition
[garden-path syntax]: https://en.wikipedia.org/wiki/Garden_path_sentence
[GitHub issue tracker]: https://github.com/tc39/proposal-pipeline-operator/issues
[goals]: #goals
[grammar parameters]: #grammar-parameters
[Hack pipe]: https://docs.hhvm.com/hack/operators/pipe-operator
[Huffman coding]: https://en.wikipedia.org/wiki/Huffman_coding
[human writability]: #human-writability
[i-am-tom functional composition]: https://github.com/fantasyland/ECMAScript-proposals/issues/1#issuecomment-306243513
[identity function]: https://en.wikipedia.org/wiki/Identity_function
[IIFEs]: https://en.wikipedia.org/wiki/Immediately-invoked_function_expression
[immutable objects]: https://en.wikipedia.org/wiki/Immutable_object
[incidental complexity]: https://en.wikipedia.org/wiki/Incidental_complexity
[intro]: #smart-pipelines
[isiahmeadows functional composition]: https://github.com/isiahmeadows/function-composition-proposal
[jashkenas]: https://github.com/jashkenas
[jQuery + CP]: #jquery-core-proposal-only
[jQuery]: https://jquery.com/
[jquery/src/core/access.js]: https://github.com/jquery/jquery/blob/2.2-stable/src/core/access.js
[jquery/src/core/init.js]: https://github.com/jquery/jquery/blob/2.2-stable/src/core/init.js
[jquery/src/core/parseHTML.js]: https://github.com/jquery/jquery/blob/2.2-stable/src/core/parseHTML.js
[JS Foundation]: https://js.foundation/
[Julia pipe]: https://docs.julialang.org/en/stable/stdlib/base/#Base.:|>
[lexical grammar]: #lexical-grammar
[lexical topic]: https://jschoi.org/18/es-smart-pipelines/spec#sec-lexical-topics
[lexically hygienic]: https://en.wikipedia.org/wiki/Hygienic_macro
[littledan invitation]: https://github.com/tc39/proposal-pipeline-operator/issues/89#issuecomment-363853394
[littledan]: https://github.com/littledan
[LiveScript pipe]: http://livescript.net/#operators-piping
[Lodash + CP + BP + PF]: #lodash-core-proposal--additional-feature-bppf
[Lodash + CP + BP + PP + PF + NP]: #lodash-core-proposal--additional-features-bppppfmt
[Lodash + CP]: #lodash-core-proposal-only
[Lodash]: https://lodash.com/
[mAAdhaTTah]: https://github.com/mAAdhaTTah/
[make my code easier to read]: #make-my-code-easier-to-read
[MDN operator precedence]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Operator_Precedence
[MDN operator precedence]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Operator_Precedence#Table
[method-extraction inline caching]: https://github.com/tc39/proposal-bind-operator/issues/46
[mindeavor]: https://github.com/gilbert
[mode errors]: https://en.wikipedia.org/wiki/Mode_(computer_interface)#Mode_errors
[motivation]: #motivation
[Node-stream piping]: https://nodejs.org/api/stream.html#stream_readable_pipe_destination_options
[Node.js `util.promisify`]: https://nodejs.org/api/util.html#util_util_promisify_original
[nomenclature]: #nomenclature
[novice learnability]: #novice-learnability
[nullish coalescing proposal]: https://github.com/tc39/proposal-nullish-coalescing/
[object initializers’ Computed Property Contains rule]: https://tc39.github.io/ecma262/#sec-object-initializer-static-semantics-computedpropertycontains
[OCaml pipe]: http://blog.shaynefletcher.org/2013/12/pipelining-with-operator-in-ocaml.html
[operator precedence]: #operator-precedence
[opt-in behavior]: #opt-in-behavior
[optional `catch` binding]: #optional-catch-binding
[optional-chaining syntax proposal]: https://github.com/tc39/proposal-optional-chaining
[other browsers’ console variables]: https://www.andismith.com/blogs/2011/11/25-dev-tool-secrets/
[other ECMAScript proposals]: #other-ecmascript-proposals
[other goals]: #other-goals
[partial function application]: #partial-function-application
[PEP 20]: https://www.python.org/dev/peps/pep-0020/
[Perl 6 pipe]: https://docs.perl6.org/language/operators#infix_==&gt;
[Perl 6 topicization]: https://www.perl.com/pub/2002/10/30/topic.html/
[Perl 6’s given block]: https://docs.perl6.org/language/control#given
[pipeline body]: https://jschoi.org/18/es-smart-pipelines/spec#prod-PipelineBody
[pipeline functions]: #additional-feature-pf
[pipeline head]: https://jschoi.org/18/es-smart-pipelines/spec#prod-PipelineHead
[Pipeline Proposal 1]: https://github.com/tc39/proposal-pipeline-operator/wiki#proposal-1-f-sharp-only
[Pipeline Proposal 4]: https://github.com/tc39/proposal-pipeline-operator/wiki#proposal-4-smart-mix
[pipeline syntax]: #pipeline-syntax
[pipelines]: #core-proposal
[previous pipeline-placeholder discussions]: https://github.com/tc39/proposal-pipeline-operator/issues?q=placeholder
[primary topic]: #core-proposal
[private class fields]: https://github.com/tc39/proposal-class-fields/
[pure functions]: https://en.wikipedia.org/wiki/Pure_function
[R pipe]: https://cran.r-project.org/web/packages/magrittr/vignettes/magrittr.html
[Ramda + CP + BP + PF + NP]: #ramda-core-proposal--additional-features-bppfmt
[Ramda + CP + BP + PF]: #ramda-core-proposal--additional-feature-bppf
[Ramda wiki cookbook]: https://github.com/ramda/ramda/wiki/Cookbook
[Ramda]: http://ramdajs.com/
[relations to other work]: #relations-to-other-work
[REPLs]: https://en.wikipedia.org/wiki/Read–eval–print_loop
[resolving topics]: #resolve-topic
[rest topic]: #additional-feature-np
[reverse Polish notation]: https://en.wikipedia.org/wiki/Reverse_Polish_notation
[robust method extraction]: https://github.com/tc39/proposal-pipeline-operator/issues/110#issuecomment-374367888
[Ron Buckton]: https://github.com/rbuckton
[runtime semantics]: #runtime-semantics
[secondary topic]: #additional-feature-np
[simonstaton functional composition]: https://github.com/simonstaton/Function.prototype.compose-TC39-Proposal
[simple scoping]: #simple-scoping
[sindresorhus]: https://github.com/sindresorhus
[smart body syntax]: #smart-body-syntax
[smart pipelines]: #smart-pipelines
[Standard Style]: https://standardjs.com/
[static analyzability]: #static-analyzability
[statically analyzable]: #static-analyzability
[statically detectable early errors]: #static-analyzability
[syntactic locality]: #syntactic-locality
[tacit programming]: https://en.wikipedia.org/wiki/Tacit_programming
[TC39 60th meeting, pipelines]: https://tc39.github.io/tc39-notes/2017-09_sep-26.html#11iia-pipeline-operator
[TC39 process]: https://tc39.github.io/process-document/
[tc39/proposal-decorators#30]: tc39/proposal-decorators#30
[tc39/proposal-decorators#42]: tc39/proposal-decorators#42
[tc39/proposal-decorators#60]: tc39/proposal-decorators#60
[Tennent correspondence principle]: http://gafter.blogspot.com/2006/08/tennents-correspondence-principle-and.html
[term rewriting with autogenerated variables]: #term-rewriting-with-autogenerated-variables
[term rewriting with single dummy variable]: #term-rewriting-with-single-dummy-variable
[term rewriting]: #term-rewriting
[terse composition]: #terse-composition
[terse function application]: #terse-function-application
[terse function calls]: #terse-function-calls
[terse method extraction]: #terse-method-extraction
[terse parentheses]: #terse-parentheses
[terse partial application]: #terse-partial-application
[terse variables]: #terse-variables
[tertiary topic]: #additional-feature-np
[TheNavigateur functional composition]: https://github.com/TheNavigateur/proposal-pipeline-operator-for-function-composition
[topic and comment]: https://en.wikipedia.org/wiki/Topic_and_comment
[topic references in other programming languages]: #topic-references-in-other-programming-languages
[topic style]: #topic-style
[topic variables in other languages]: https://rosettacode.org/wiki/Topic_variable
[topic-token bikeshedding]: https://github.com/tc39/proposal-pipeline-operator/issues/91
[Underscore.js + CP + BP + PP]: #underscorejs-core-proposal--additional-feature-bppp
[Underscore.js + CP]: #underscorejs-core-proposal-only
[Underscore.js]: http://underscorejs.org
[Unix pipe]: https://en.wikipedia.org/wiki/Pipeline_(Unix
[untangled flow]: #untangled-flow
[Visual Basic’s `select` statement]: https://docs.microsoft.com/en-us/dotnet/visual-basic/language-reference/statements/select-case-statement
[WebKit console variables]: https://webkit.org/blog/829/web-inspector-updates/
[WHATWG Fetch + CP]: #whatwg-fetch-standard-core-proposal-only
[WHATWG Fetch Standard]: https://fetch.spec.whatwg.org/
[WHATWG Streams + CP + BP + PF + NP]: #whatwg-streams-standard-core-proposal--additional-features-bppfmt
[WHATWG Streams + CP + BP + PF]: #whatwg-streams-standard-core-proposal--additional-feature-bppf
[WHATWG Streams Standard]: https://stream.spec.whatwg.org/
[WHATWG-stream piping]: https://streams.spec.whatwg.org/#pipe-chains
[Wikipedia: term rewriting]: https://en.wikipedia.org/wiki/Term_rewriting
[zero runtime cost]: #zero-runtime-cost
