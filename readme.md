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
  - [Additional Feature TS](#additional-feature-ts)
  - [Additional Feature PF](#additional-feature-pf)
    - [Ramda (Core Proposal + Additional Feature BP+PF)](#ramda-core-proposal--additional-feature-bppf)
    - [WHATWG Streams Standard (Core Proposal + Additional Features BP+PP+PF)](#whatwg-streams-standard-core-proposal--additional-features-bppppf)
  - [Additional Feature NP](#additional-feature-np)
    - [Lodash (Core Proposal + Additional Features BP+PP+PF+NP)](#lodash-core-proposal--additional-features-bppppfnp)
    - [Ramda (Core Proposal + Additional Features BP+PF+NP)](#ramda-core-proposal--additional-features-bppfnp)
    - [WHATWG Streams Standard (Core Proposal + Additional Features BP+PP+PF+NP)](#whatwg-streams-standard-core-proposal--additional-features-bppppfnp)
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
  - [Smart step syntax](#smart-step-syntax)
    - [Bare style](#bare-style)
    - [Topic style](#topic-style)
    - [Practical consequences](#practical-consequences)
  - [Operator precedence and associativity](#operator-precedence-and-associativity)
  - [Explanation of nomenclature](#explanation-of-nomenclature)
  - [Term rewriting](#term-rewriting)
    - [Core Proposal](#core-proposal-1)
    - [Additional Feature BP](#additional-feature-bp-1)
    - [Additional Feature NP](#additional-feature-np-1)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->
</details></nav>

***

This document is an **explainer for** the [**formal specification** of a proposed
**smart pipe operator `|>`**][formal pipeline specification] in
**JavaScript**, along with several other additional features. The specification
is divided into **one Stage-0 Core Proposal** plus **six** mutually
independent-but-compatible **Additional Features**:

|Name                     | Status  | Features                                                               | Purpose                                                                                                         |
| ----------------------- | ------- | ---------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
|[Core Proposal][]        | Stage 0 | Infix pipelines `… \|> …`<br>Lexical topic `#`                         | **Unary** function/expression **application**                                                                   |
|[Additional Feature BC][]| None    | Bare constructor calls `… \|> new …`                                   | Tacit application of **constructors**                                                                           |
|[Additional Feature BA][]| None    | Bare awaited calls `… \|> await …`                                     | Tacit application of **async functions**                                                                        |
|[Additional Feature BP][]| None    | Block pipeline steps `… \|> {…}`                                       | Application of **statement blocks**                                                                             |
|[Additional Feature PF][]| None    | Pipeline functions `+>  `                                              | **Partial** function/expression **application**<br>Function/expression **composition**<br>**Method extraction** |
|[Additional Feature TS][]| None    | Pipeline `try` statements                                              | Tacit application to **caught errors**                                                                          |
|[Additional Feature NP][]| None    | N-ary pipelines `(…, …) \|> …`<br>Lexical topics `##`, `###`, and `...`| **N-ary** function/expression **application**                                                                   |

The **Core Proposal** is currently at **Stage 0** of the [TC39 process][TC39
process] and is planned to be presented, along with a [competing
proposal][Pipeline Proposal 1], to TC39 by [Daniel “**littledan**” Ehrenberg of
Igalia][littledan]. The Core Proposal is a **variant** of the [first
pipe-operator proposal][] also championed by Ehrenberg; this variant is
listed as [**Proposal 4: Smart Mix** in the pipe-proposal wiki][Pipeline
Proposal 4]. The variant resulted from [previous discussions in the previous
pipe-operator proposal][previous pipeline-placeholder discussions],
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
This section gives a brief overview of the motivations behind the smart pipe
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

The infix “smart” pipe operator `|>` proposed here would provide a
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

A pipeline is made of a **head** expression, followed by a chain of postfix
expressions called **pipeline stages**. Each stage has its own **inner lexical
scope**, within which a special topic reference `#` is defined. This `#` is a
reference to the **[lexical topic][]** of the pipeline (`#` itself is called a
**topic reference**).
```js
input
|> # + 1 // step 1
|> f(x, #, y) // step 2
|> await g(#, z) // step 3
|> console.log(`${#}!`); // step 4
```
0. The **head** expression to the left of the pipeline steps is **first evaluated**.
1. It then is inputted into **pipeline step 1**, becoming that step’s **lexical
   topic**. A **new lexical environment** is created, scoped only to pipeline
   step 1, and within which `#` is immutably **bound to the topic**. Using that
   topic binding, the first pipeline step is then **evaluated**; the current
   lexical environment is then reset back to before.
2. The result of the first pipeline step becomes the input to step 1.
   A new lexical environment is created, scoped only to pipeline step 2, and
   whose topic binding is the result of evaluating step 1.
3. And so forth, until step 4 is evaluated, with the result of step 3 as its input.
   The result of step 4 is the result of the entire pipeline.

<td>

```js
console.log(`${ // step 4
  await g( // step 3
    f(x, // step 2
      input + 1, // step 1
      y), // step 2
    z) // step 3
}!`); // step 4
```

<tr>
<td>

```js
input |> (# = 50);
// 🚫 Reference Error:
// Cannot assign to topic reference.
```
The topic binding is immutable, established only once per lexical environment.
It is an error to attempt to assign a value to it using `=`, whether inside or
outside a pipeline step.

<td>

<tr>
<td>

This chained pipeline:
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
This version is equivalent to the version above, except that the
`|> capitalize(#)` and `|> console.log(#)` pipeline steps explicitly include
optional topic references `#`, making the expressions slightly wordier than
necessary.

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
part of the “smart pipe operator”][smart step syntax]. The styles of
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

This is the [**smart** part of the smart pipe operator][smart step syntax],
which can distinguish between two syntax styles (**[bare style][]** vs. **[topic
style][]**) by using a simple rule: **bare** style uses only **identifiers and
dots** – and **never parentheses, brackets, braces**, or **other operators**. And
**topic** style **always** contains at least one **topic reference**. For more
information, see the reference below about the **[smart step syntax][]**.

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
// Topic-style pipeline step
// `|> x + 50`
// binds topic but contains
// no topic reference.
// 🚫 Syntax Error:
// Topic-style pipeline step
// `|> g(x, 2)`
// binds topic but contains
// no topic reference.
```
In order to fulfill the [goal][goals] of [“don’t shoot me in the foot”][],
when a **pipeline step is in [topic style][]** but it **contains no topic
reference**, that is an **[early error][]**. Such a degenerate pipeline step has
a very good chance of actually being an accidental bug. (Note that the
bare-style pipeline step `|> f` is *not* an error. The [bare style][] is not
supposed to contain any topic references `#`.)

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
// Topic-style pipeline step
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
// Topic-style pipeline step
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
languages’ pipe operators variously do either][topic references in other
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
// Topic-style pipeline step
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
The topic reference may be used multiple times in a pipeline step. Each use
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
A topic-style pipeline step may contain array literals. These may be flattened,
just like any other sort of expression.

<td>

```js
g([0, 1, 2, ...f(input)]);
```

<tr>
<td>

A topic-style pipeline step may also contain object literals. However, pipeline
steps that are entirely object literals must be parenthesized. It is similar to
how arrow functions distinguish between object literals and blocks.
```js
input |> f |> ({ x: #, y: # }) |> g;
```
This fulfills the goal of [forward compatibility][] with [Additional
Feature BP][], which introduces block pipeline steps. (It is expected that
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
A topic-style pipeline step may contain an inner arrow function. Both versions
of this example result in an arrow function in a closure on the previous
pipeline’s result `input |> f`.

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
The topic value of the second pipeline step (here represented by a normal
variable `$`) is still lexically accessible within its body, an arrow function,
in both examples.

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
of its steps, then the arrow-function expression must be parenthesized.
(The same applies to assignment and yield operators, which are also looser than
the pipe operator.) The example above is being parsed as if it were:
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

Both the head and the steps of a pipeline may contain nested inner pipelines.
```js
x = input
|> f(x =>
  # + x |> g |> # * 2)
|> #.toString();
```

<td>

A nested pipeline works consistently. It merely shadows the outer context’s
topic with the topic within its own steps’ inner contexts.
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
* `for` and `while` statements,
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
// Pipeline step `|> function () { … }`
// binds topic but contains
// no topic reference.
```
<td>

<tr>
<td>

```js
x = input |> class { m: () { return #; } };
// 🚫 Syntax Error:
// Pipeline step `|> class { … }`
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

A function definition that is a pipeline step may contain topic references in
its default parameters’ expressions, because their scoping is similar to that of
the outside context’s: similar enough such that also allowing topic references
in them would fulfill the goal of [simple scoping][]. However, as stated above,
the function body itself still may not contain topic references.

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
    : #|> isArrayLike
    ? #|> #.length
    : #|> _.keys |> #.length;
  };
}
```
Smart pipelines make parallelism between all three clauses becomes clearer:\
`0` if it is nullish,\
`#|> #.length` if it is array-like, and\
`#|> something |> #.length` otherwise.\
(Technically, `#|> #.length` could simply be `#.length`, but it is written in
this redundant form in order to emphasis its parallelism with the other branch.)

[This particular example becomes even clearer][Underscore.js + CP + BP + PP]
when paired with [Additional Feature BP][].

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
    #|> isArray
    ? #
    : (#|> isKey(#, object))
    ? [#]
    : #|> toString |> stringToPath;
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
terser. It adds a mode to bare style: if a bare-style pipeline step is preceded
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
terser. It adds another mode to bare style: if a bare-style pipeline step is
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
step in [topic style][] may be `do` expression, as long as the `do` expression
contains the topic reference `#`. The topic reference `#` is bound to the input
value, the `do` expression is evaluated, then the result of the `do` block
becomes the result of that pipeline step, and the lexical environment is reset –
all as usual.

In this manner, pipelines with `do` expressions act as a way to create a
“topic-context block”, similarly to [Perl 6’s given block][]. Within this block,
statements may use the topic reference may be used as an abbreviation for the
same value. This can be useful for embedding side effects, `if` `else`
statements, `try` statements, and `switch` statements within pipelines.
They may be made even pithier with [Additional Feature BP][], explained later.

[`do` expressions][] as [topic-style][topic style] pipeline steps might be so
useful, in fact, that it might be worth building them into the pipe operator
`|>` itself as an add-on feature. This additional feature – **block pipelines**
– adds an additional [topic-style pipeline step syntax][smart step syntax],
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
Side effects may easily be embedded within block pipeline steps.

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
`if` `else` statements may also be used within block pipeline steps, as an
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
    #|> JSON.parse;
    catch (error) {
      { message: error.message };
    }
  }
}
|> g;
```
`try` statements would also be useful to embed in pipelines with block steps.
This example becomes even pithier with [Additional Feature TS][].

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
// Pipeline step `|> { if (…) … else … }`
// binds topic but contains
// no topic reference.
```
The same [early error rules][] that apply to any topic-style pipeline step apply
also to topic-style steps that are `do` expressions.

<tr>
<td>

As with all other [additional features][], Additional Feature BP is [forward
compatible][] with the [Core Proposal][]. This compatibility includes pipeline
steps that are object literals, which must be parenthesized.
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
Feature BP][] shows how terseness could be further improved.

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
    #|> this.constructor;
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
  else if (#|> isFunction)
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
    #|> isArray
    ? #
    : (#|> isKey(#, object))
    ? [#]
    : #|> toString |> stringToPath;
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
    if (#|> isArray)
      #;
    else if (#|> isKey(#, object))
      [#];
    else #
    |> toString |> stringToPath;
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

# Appendices
## Smart step syntax
Most pipeline steps will use topic references in their steps. This style of
pipeline step is called **[topic style][]**.

For three simple cases – unary functions, unary async functions, and unary
constructors – you may omit the topic reference from the pipeline step. This is
called **[bare style][]**.

When a pipe is in bare style, we refer to the pipeline as a **bare function
call**. (If [Additional Feature BC][] or [Additional Feature BA][] are used,
then a bare-style pipeline step may instead be a **bare awaited function call**,
or a **bare constructor call**, depending on the rules of bare style.) The step
acts as just a simple reference to a function, such as with `… |> capitalize` or
with `… |> console.log`. The step’s value would then be called as a unary
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

If the pipeline step is a merely a simple reference, then that identifier is
interpreted to be a **bare function call**. The pipeline’s value will be the
result of evaluating the step as an identifier or member expression, then
calling the result as a function, with the current topics as its arguments.

That is: **if a pipeline** is of the form\
**_topic_ `|>` _identifier_**\
or **_topic_ `|>` _identifier0_`.`_identifier1_**\
or **_topic_ `|>` _identifier0_`.`_identifier1_`.`_identifier2_**\
or so forth,\
then the pipeline is a bare function call.

**With [Additional Feature BC][]:**\
If a pipeline step starts with `new`, followed by a mere identifier, optionally
with a chain of properties, and with no parentheses or brackets, then that
identifier is interpreted to be a **bare constructor**.

That is: **if a pipeline** is of the form\
**_topic_ `|>` `new` _identifier_**\
or **_topic_ `|>` `new` _identifier0_`.`_identifier1_**\
or **_topic_ `|>` `new` _identifier0_`.`_identifier1_`.`_identifier2_**\
or so forth,\
then the pipeline is a bare constructor call.

**With [Additional Feature BA][]:**\
If a pipeline step starts with `await`, followed by a mere identifier, optionally with
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
[syntactic locality][]. Instead, **if a pipeline step** of the form
|> _step_ does ***not* match the [bare style][]** (that is, it is *not* a bare
function call, bare async function call, or bare constructor call), then it
**must be in topic style**. Topic style **requires** that there be a topic
reference in the pipeline step; otherwise it is an [early error][].

A pipeline step that is not in bare style is usually a **topic expression**.
This is any expression at the [precedence level once tighter than pipeline-level
expressions][operator precedence] – that is, any conditional-level expression.

**With [Additional Feature BA][]:**\
A pipeline step that is a block `{` … `}` containing a list of statements is a
**topic block**. The last statement in the block is used as the result of the
whole pipeline, similarly to [`do` expressions][].

### Practical consequences
Therefore, a pipeline step in **[bare style][] *never*** contains **parentheses `(…)` or
brackets `[…]`**. Neither `… |> object.method()` nor
`… |> object.method(arg)` nor `… |> object[symbol]` nor `… |> object.createFunction()`
are in bare style (in fact, they all are Syntax Errors, due to their being in
[topic style][] without any topic references).

**When a pipeline step needs parentheses or brackets**, then **don’t use bare style**,
and instead **use a topic reference** in the step ([topic style][])…or **assign
the step to a variable**, then **use that variable as a bare-style step**.

## Operator precedence and associativity
As a infix operation forming compound expressions, the [operator precedence and
associativity][MDN operator precedence] of pipelining must be determined, relative
to other operations.

Precedence is tighter than arrow functions (`=>`), assignment (`=`, `+=`, …),
generator `yield` and `yield *`, and sequence `,`; and it is looser than every
other type of expression. If the pipe operation were any tighter than this
level, its steps would have to be parenthesized for many frequent types of
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

The term “**topic-style pipeline**” is preferred to “**topic expression**”
because, in the future, the topic concept could be extended to other syntaxes,
as with [Additional Feature TS][], not just pipelines.

In addition, “**input** values of a pipeline step” is preferred to “pipeline
**LHS**”, because “LHS” in the ECMAScript specification usually refers to the
[LHS of assignments][ECMAScript LHS expressions], which may be confusing.
However, “LHS” is still a fine and acceptable, if not nonspecific, name for a
pipeline step’s input. The **head** of a pipeline – the input of a pipeline’s
first step – is distinguished from the pipeline itself. A pipeline head cannot
contain any topic references, and it is completely omitted in [pipeline
functions][].

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

The terms “**pipeline step**” and “**output** values of a pipeline step” is
preferred instead of “**RHS** of a pipeline”, just as how “input values” is
preferred to “LHS”. However, “RHS” is still a fine and acceptable name for the
right-hand side of a pipe operator. The **step** is the expression itself; it
evaluates into one or output values. The output values in turn either are fed
into the following pipeline step as its inputs or become the value of the entire
pipeline. (All pipelines are allowed to result in at most one output; it is an
[early error][] if it could ever return zero or more than one outputs.)

“**[Bare style][]**” could also be called “**tacit style**”, but the former is
preferred to the latter. Eventually, certain possible future extensions to the
topic concept, such as [Additional Feature TS][], would enable [tacit
programming][] even without using bare-style pipelines. **Bare style** could
also have been called **plain style**.

[“data-to-ink” visual ratio]: https://www.darkhorseanalytics.com/blog/data-looks-better-naked
[“don’t break my code”]: ./goals.md#dont-break-my-code
[“don’t make me overthink”]: ./goals.md#dont-make-me-overthink
[“don’t shoot me in the foot”]: ./goals.md#dont-shoot-me-in-the-foot
[“make my code easier to read”]: ./goals.md#make-my-code-easier-to-read
[`??:`]: https://github.com/tc39/proposal-nullish-coalescing/pull/23
[`do` expression]: ./relations.md#do-expressions
[`do` expressions]: ./relations.md#do-expressions
[`for` iteration statements]: https://tc39.github.io/ecma262/#sec-iteration-statements
[`in` relational operator]: https://tc39.github.io/ecma262/#sec-relational-operators
[`match` expressions]: ./relations.md#pattern-matching
[`new.target`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/new.target
[Additional Feature BA]: ./additional-feature-ba.md
[Additional Feature BC]: ./additional-feature-bc.md
[Additional Feature BP]: ./additional-feature-bp.md
[Additional Feature NP]: ./additional-feature-np.md
[Additional Feature PF]: ./additional-feature-pf.md
[Additional Feature TS]: ./additional-feature-ts.md
[additional features]: ./readme.md
[annevk]: https://github.com/annevk
[antecedent]: https://en.wikipedia.org/wiki/Antecedent_(grammar)
[arbitrary associativity]: ./goals.md#arbitrary-associativity
[ASI]: https://tc39.github.io/ecma262/#sec-automatic-semicolon-insertion
[associative property]: https://en.wikipedia.org/wiki/Associative_property
[async pipeline functions]: ./additional-feature-pf.md
[Babel plugin]: https://github.com/valtech-nyc/babel/
[Babel update summary]: https://github.com/babel/proposals/issues/29#issuecomment-372828328
[background]: ./readme.md
[backward compatibility]: ./goals.md#backward-compatibility
[bare awaited function call]: ./readme.md#bare-style
[bare constructor call]: ./readme.md#bare-style
[bare function call]: ./readme.md#bare-style
[bare style]: ./readme.md#bare-style
[binding]: https://en.wikipedia.org/wiki/Binding_(linguistics)
[block parameters]: ./relations.md#block-parameters
[Clojure compact function]: https://clojure.org/reference/reader#_dispatch
[Clojure pipe]: https://clojuredocs.org/clojure.core/as-%3E
[completion records]: https://timothygu.me/es-howto/#completion-records-and-shorthands
[concatenative programming]: https://en.wikipedia.org/wiki/Concatenative_programming_language
[conceptual generality]: ./readme.md#conceptual-generality
[Core Proposal]: ./readme.md
[currying]: https://en.wikipedia.org/wiki/Currying
[cyclomatic complexity]: https://en.wikipedia.org/wiki/Cyclomatic_complexity#Applications
[cyclomatic simplicity]: ./readme.md#cyclomatic-simplicity
[dataflow programming]: https://en.wikipedia.org/wiki/Dataflow_programming
[distinguishable punctuators]: ./readme.md#distinguishable-punctuators
[don’t break my code]: ./readme.md#dont-break-my-code
[DSLs]: https://en.wikipedia.org/wiki/Domain-specific_language
[early error rule]: ./readme.md#static-analyzability
[early error rules]: ./readme.md#static-analyzability
[early error]: ./readme.md#static-analyzability
[early errors]: ./readme.md#static-analyzability
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
[expressions and operators (MDN)]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Expressions_and_Operators
[expressive versatility]: ./readme.md#expressive-versatility
[F# pipe]: https://docs.microsoft.com/en-us/dotnet/fsharp/language-reference/functions/index#function-composition-and-pipelining
[first pipe-operator proposal]: https://github.com/tc39/proposal-pipeline-operator/blob/37119110d40226476f7af302a778bc981f606cee/README.md
[footguns]: https://en.wiktionary.org/wiki/footgun
[formal BA]: https://jschoi.org/18/es-smart-pipelines/spec#sec-additional-feature-ba
[formal BC]: https://jschoi.org/18/es-smart-pipelines/spec#sec-additional-feature-bc
[formal BP]: https://jschoi.org/18/es-smart-pipelines/spec#sec-additional-feature-bp
[formal CP]: https://jschoi.org/18/es-smart-pipelines/spec#introduction
[formal NP]: https://jschoi.org/18/es-smart-pipelines/spec#sec-additional-feature-np
[formal PF]: https://jschoi.org/18/es-smart-pipelines/spec#sec-additional-feature-pf
[formal pipeline specification]: https://jschoi.org/18/es-smart-pipelines/spec
[formal TS]: https://jschoi.org/18/es-smart-pipelines/spec#sec-additional-feature-ts
[forward compatibility]: ./readme.md#forward-compatibility
[forward compatible]: ./readme.md#forward-compatibility
[function bind operator `::`]: ./relations.md#function-bind-operator
[function binding]: ./relations.md#function-binding
[function composition]: ./relations.md#function-composition
[functional programming]: https://en.wikipedia.org/wiki/Functional_programming
[gajus functional composition]: https://github.com/gajus/babel-plugin-transform-function-composition
[garden-path syntax]: https://en.wikipedia.org/wiki/Garden_path_sentence
[GitHub issue tracker]: https://github.com/tc39/proposal-pipeline-operator/issues
[goals]: ./goals.md
[Hack pipe]: https://docs.hhvm.com/hack/operators/pipe-operator
[Huffman coding]: https://en.wikipedia.org/wiki/Huffman_coding
[human writability]: ./goals.md#human-writability
[i-am-tom functional composition]: https://github.com/fantasyland/ECMAScript-proposals/issues/1#issuecomment-306243513
[identity function]: https://en.wikipedia.org/wiki/Identity_function
[IIFEs]: https://en.wikipedia.org/wiki/Immediately-invoked_function_expression
[immutable objects]: https://en.wikipedia.org/wiki/Immutable_object
[incidental complexity]: https://en.wikipedia.org/wiki/Incidental_complexity
[intro]: readme.md
[isiahmeadows functional composition]: https://github.com/isiahmeadows/function-composition-proposal
[jashkenas]: https://github.com/jashkenas
[jQuery + CP]: ./readme.md#jquery-core-proposal-only
[jQuery]: https://jquery.com/
[jquery/src/core/access.js]: https://github.com/jquery/jquery/blob/2.2-stable/src/core/access.js
[jquery/src/core/init.js]: https://github.com/jquery/jquery/blob/2.2-stable/src/core/init.js
[jquery/src/core/parseHTML.js]: https://github.com/jquery/jquery/blob/2.2-stable/src/core/parseHTML.js
[JS Foundation]: https://js.foundation/
[Julia pipe]: https://docs.julialang.org/en/stable/stdlib/base/#Base.:|>
[lexical topic]: https://jschoi.org/18/es-smart-pipelines/spec#sec-lexical-topics
[lexically hygienic]: https://en.wikipedia.org/wiki/Hygienic_macro
[littledan invitation]: https://github.com/tc39/proposal-pipeline-operator/issues/89#issuecomment-363853394
[littledan]: https://github.com/littledan
[LiveScript pipe]: http://livescript.net/#operators-piping
[Lodash + CP + BP + PF]: ./additional-feature-pf.md#lodash-core-proposal--additional-feature-bppf
[Lodash + CP + BP + PP + PF + NP]: ./additional-feature-np.md#lodash-core-proposal--additional-features-bppppfmt
[Lodash + CP]: ./readme.md#lodash-core-proposal-only
[Lodash]: https://lodash.com/
[mAAdhaTTah]: https://github.com/mAAdhaTTah/
[make my code easier to read]: ./goals.md#make-my-code-easier-to-read
[MDN operator precedence]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Operator_Precedence
[MDN operator precedence]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Operator_Precedence#Table
[method-extraction inline caching]: https://github.com/tc39/proposal-bind-operator/issues/46
[mindeavor]: https://github.com/gilbert
[mode errors]: https://en.wikipedia.org/wiki/Mode_(computer_interface)#Mode_errors
[motivation]: ./readme.md#motivation
[Node-stream piping]: https://nodejs.org/api/stream.html#stream_readable_pipe_destination_options
[Node.js `util.promisify`]: https://nodejs.org/api/util.html#util_util_promisify_original
[nomenclature]: ./nomenclature.md
[novice learnability]: ./goals.md#novice-learnability
[nullish coalescing proposal]: https://github.com/tc39/proposal-nullish-coalescing/
[object initializers’ Computed Property Contains rule]: https://tc39.github.io/ecma262/#sec-object-initializer-static-semantics-computedpropertycontains
[OCaml pipe]: http://blog.shaynefletcher.org/2013/12/pipelining-with-operator-in-ocaml.html
[operator precedence]: ./readme.md#operator-precedence
[opt-in behavior]: ./goals.md#opt-in-behavior
[optional `catch` binding]: ./relations.md#optional-catch-binding
[optional-chaining syntax proposal]: https://github.com/tc39/proposal-optional-chaining
[other browsers’ console variables]: https://www.andismith.com/blogs/2011/11/25-dev-tool-secrets/
[other ECMAScript proposals]: ./relations.md#other-ecmascript-proposals
[other goals]: ./readme.md#other-goals
[partial function application]: ./goals.md#partial-function-application
[PEP 20]: https://www.python.org/dev/peps/pep-0020/
[Perl 6 pipe]: https://docs.perl6.org/language/operators#infix_==&gt;
[Perl 6 topicization]: https://www.perl.com/pub/2002/10/30/topic.html/
[Perl 6’s given block]: https://docs.perl6.org/language/control#given
[pipeline functions]: ./additional-feature-pf.md
[Pipeline Proposal 1]: https://github.com/tc39/proposal-pipeline-operator/wiki#proposal-1-f-sharp-only
[Pipeline Proposal 4]: https://github.com/tc39/proposal-pipeline-operator/wiki#proposal-4-smart-mix
[pipeline syntax]: ./readme.md#pipeline-syntax
[pipelines]: ./readme.md
[previous pipeline-placeholder discussions]: https://github.com/tc39/proposal-pipeline-operator/issues?q=placeholder
[primary topic]: ./readme.md
[private class fields]: https://github.com/tc39/proposal-class-fields/
[pure functions]: https://en.wikipedia.org/wiki/Pure_function
[R pipe]: https://cran.r-project.org/web/packages/magrittr/vignettes/magrittr.html
[Ramda + CP + BP + PF + NP]: ./additional-feature-np.md#ramda-core-proposal--additional-features-bppfmt
[Ramda + CP + BP + PF]: ./additional-feature-pf.md#ramda-core-proposal--additional-feature-bppf
[Ramda wiki cookbook]: https://github.com/ramda/ramda/wiki/Cookbook
[Ramda]: http://ramdajs.com/
[relations to other work]: ./relations.md
[REPLs]: https://en.wikipedia.org/wiki/Read–eval–print_loop
[rest topic]: ./additional-feature-np.md
[reverse Polish notation]: https://en.wikipedia.org/wiki/Reverse_Polish_notation
[robust method extraction]: https://github.com/tc39/proposal-pipeline-operator/issues/110#issuecomment-374367888
[Ron Buckton]: https://github.com/rbuckton
[secondary topic]: ./additional-feature-np.md
[simonstaton functional composition]: https://github.com/simonstaton/Function.prototype.compose-TC39-Proposal
[simple scoping]: ./goals.md#simple-scoping
[sindresorhus]: https://github.com/sindresorhus
[smart step syntax]: ./readme.md#smart-step-syntax
[smart pipelines]: ./readme.md#smart-pipelines
[Standard Style]: https://standardjs.com/
[static analyzability]: ./goals.md#static-analyzability
[statically analyzable]: ./goals.md#static-analyzability
[statically detectable early errors]: ./goals.md#static-analyzability
[syntactic locality]: ./goals.md#syntactic-locality
[tacit programming]: https://en.wikipedia.org/wiki/Tacit_programming
[TC39 60th meeting, pipelines]: https://tc39.github.io/tc39-notes/2017-09_sep-26.html#11iia-pipeline-operator
[TC39 process]: https://tc39.github.io/process-document/
[tc39/proposal-decorators#30]: tc39/proposal-decorators#30
[tc39/proposal-decorators#42]: tc39/proposal-decorators#42
[tc39/proposal-decorators#60]: tc39/proposal-decorators#60
[Tennent correspondence principle]: http://gafter.blogspot.com/2006/08/tennents-correspondence-principle-and.html
[term rewriting]: ./term-rewriting.md
[terse composition]: ./goals.md#terse-composition
[terse function application]: ./goals.md#terse-function-application
[terse function calls]: ./goals.md#terse-function-calls
[terse method extraction]: ./goals.md#terse-method-extraction
[terse parentheses]: ./goals.md#terse-parentheses
[terse partial application]: ./goals.md#terse-partial-application
[terse variables]: ./goals.md#terse-variables
[tertiary topic]: ./additional-feature-np.md
[TheNavigateur functional composition]: https://github.com/TheNavigateur/proposal-pipeline-operator-for-function-composition
[topic and comment]: https://en.wikipedia.org/wiki/Topic_and_comment
[topic references in other programming languages]: ./relations.md#topic-references-in-other-programming-languages
[topic style]: ./goals.md#topic-style
[topic variables in other languages]: https://rosettacode.org/wiki/Topic_variable
[topic-token bikeshedding]: https://github.com/tc39/proposal-pipeline-operator/issues/91
[Underscore.js + CP + BP + PP]: ./additional-feature-pp.md#underscorejs-core-proposal--additional-feature-bppp
[Underscore.js + CP]: ./readme.md#underscorejs-core-proposal-only
[Underscore.js]: http://underscorejs.org
[Unix pipe]: https://en.wikipedia.org/wiki/Pipeline_(Unix
[untangled flow]: ./goals.md#untangled-flow
[Visual Basic’s `select` statement]: https://docs.microsoft.com/en-us/dotnet/visual-basic/language-reference/statements/select-case-statement
[WebKit console variables]: https://webkit.org/blog/829/web-inspector-updates/
[WHATWG Fetch + CP]: ./readme.md#whatwg-fetch-standard-core-proposal-only
[WHATWG Fetch Standard]: https://fetch.spec.whatwg.org/
[WHATWG Streams + CP + BP + PF + NP]: ./additional-feature-np.md#whatwg-streams-standard-core-proposal--additional-features-bppfmt
[WHATWG Streams + CP + BP + PF]: ./additional-feature-pf.md#whatwg-streams-standard-core-proposal--additional-feature-bppf
[WHATWG Streams Standard]: https://stream.spec.whatwg.org/
[WHATWG-stream piping]: https://streams.spec.whatwg.org/#pipe-chains
[Wikipedia: term rewriting]: https://en.wikipedia.org/wiki/Term_rewriting
[zero runtime cost]: ./goals.md#zero-runtime-cost
