# Smart pipelines
ECMAScript Stage-0 Proposal. Living Document. J. S. Choi, 2018-02.

<nav><details>
<summary><strong>Table of Contents</strong></summary>

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [Motivation](#motivation)
  - [Core Proposal](#core-proposal)
    - [WHATWG Fetch Standard (Core Proposal only)](#whatwg-fetch-standard-core-proposal-only)
    - [jQuery (Core Proposal only)](#jquery-core-proposal-only)
    - [Underscore.js (Core Proposal only)](#underscorejs-core-proposal-only)
  - [Additional Feature PP](#additional-feature-pp)
    - [WHATWG Fetch Standard (Core Proposal + Additional Feature PP)](#whatwg-fetch-standard-core-proposal--additional-feature-pp)
    - [jQuery (Core Proposal + Additional Feature PP)](#jquery-core-proposal--additional-feature-pp)
    - [Underscore.js (Core Proposal + Additional Feature PP)](#underscorejs-core-proposal--additional-feature-pp)
  - [Additional Feature CT](#additional-feature-ct)
  - [Additional Feature PF](#additional-feature-pf)
    - [Lodash](#lodash)
  - [Additional Feature MT](#additional-feature-mt)
    - [Ramda](#ramda)
    - [WHATWG Streams Standard](#whatwg-streams-standard)
- [Goals](#goals)
  - [“Don’t break my code.”](#dont-break-my-code)
    - [Backward compatibility](#backward-compatibility)
    - [Zero runtime cost](#zero-runtime-cost)
    - [Forward compatibility](#forward-compatibility)
  - [“Don’t make me overthink.”](#dont-make-me-overthink)
    - [Syntactic locality](#syntactic-locality)
    - [Cyclomatic simplicity](#cyclomatic-simplicity)
    - [Expressive versatility](#expressive-versatility)
  - [“Don’t shoot me in the foot.”](#dont-shoot-me-in-the-foot)
    - [Simple scoping](#simple-scoping)
    - [Static analyzability](#static-analyzability)
    - [ASI safety](#asi-safety)
  - [“Make my code easier to read.”](#make-my-code-easier-to-read)
    - [Untangled flow](#untangled-flow)
    - [Distinguishable punctuators](#distinguishable-punctuators)
    - [Terse parentheses](#terse-parentheses)
    - [Terse variables](#terse-variables)
    - [Terse function calls](#terse-function-calls)
  - [Other Goals](#other-goals)
    - [Conceptual generality](#conceptual-generality)
    - [Human writability](#human-writability)
    - [Novice learnability](#novice-learnability)
- [Smart body syntax](#smart-body-syntax)
- [Relations to other work](#relations-to-other-work)
  - [Other ECMAScript proposals](#other-ecmascript-proposals)
    - [`do` expressions](#do-expressions)
  - [Possible future extensions to the topic concept](#possible-future-extensions-to-the-topic-concept)
  - [Alternative solutions explored](#alternative-solutions-explored)
- [Appendix: Explanation of nomenclature](#appendix-explanation-of-nomenclature)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->
</details></nav>

***

This document is an **explainer for** the [**formal specification** of a proposed
**smart pipeline operator `|>`**][formal pipeline specification] in
**JavaScript**, along with several other additional features. It is currently
tentatively at **Stage 0** of the [TC39 process][TC39 process] but and is
planned to be presented, along with a [competing proposal][Pipeline Proposal 1],
to TC39 by [Daniel “**littledan**” Ehrenberg of Igalia][littledan]. The proposal
is divided into **one Stage-0 Core Proposal** plus **four compatible Additional
Features**:

|Name                     |Features                                      |Purpose                                                |
|-------------------------|----------------------------------------------|-------------------------------------------------------|
|[Core Proposal][]        |Infix pipe `\|>` and lexical topic `#`        |Unary application                                      |
|[Additional Feature PP][]|Prefix pipe `\|>`                             |Application in `do`/`if`/`try` blocks                  |
|[Additional Feature PF][]|Pipeline functions `=\|>`                     |Partial application<br>Composition<br>Method extraction|
|[Additional Feature MT][]|Multiple lexical topics `##`, `###`, and `...`|N-ary application                                      |
|[Additional Feature TC][]|Topical `catch` blocks                        |Application to errors                                  |

The **Core Proposal** is a **variant** of the [first pipe-operator proposal][]
also championed by Ehrenberg. This variant is listed as [**Proposal 4: Smart
Mix** on the pipe-proposal wiki][Pipeline Proposal 4]. The variant resulted from
[previous discussions in the previous pipe-operator proposal][previous
pipeline-placeholder discussions], discussions which culminated in an
[invitation by Ehrenberg to try writing a specification draft][littledan
invitation]. A **prototype Babel plugin** is also brewing.

The **Additional Features** are **not part of the Stage-0 Core Proposal**. They
are included to illustrate possible **separate follow-up proposals** for the case
in which the Core Proposal advances past Stage 1. Together, the Core Proposal
and the Additional Features demonstrate a **unified vision** of a future in
which composition, partial application, method extraction, and error handling
are all tersely expressible with the same simple pipeline/topic concept.

You can take part in the discussions on the **[GitHub issue tracker][]**. When you
file an issue, please note in it that you are talking **specifically** about
**[“Proposal 4: Smart Mix”][Pipeline Proposal 4]**.

**This specification uses `#`** as its [topic reference][nomenclature]. However,
this is **not set** in stone. In particular, **`@` or `?`** could also be used.
**Bikeshedding discussions** over what characters to use for the topic token has
been occurring on GitHub at [tc39/proposal-pipeline-operator
issue #91][topic-token bikeshedding].

# Motivation
This section gives a brief overview of the motivations behind the smart pipeline
operator’s Core Proposal, as well the Additional Features listed above.
**Examples from real-world libraries** are juxtaposed with their original
versions. The original versions have been lightly edited (e.g., breaking up
lines, removing semicolons), in order to fit their horizontal widths into this
table. **Examples that use Additional Features** are included **only to
illustrate** the power of the pipeline/topic concept and are always simply
**rewritable** into forms that use **only the Core Proposal**.

## Core Proposal

<table>
<thead>
<tr>
<th>With smart pipelines
<th>Status quo

<tbody>
<tr>
<td>

The binary “smart” pipeline operator `|>` proposed here would provide a
**[backwards- and forwards-compatible][don’t break my code]** style of
**chaining nested expressions** into a readable, **left-to-right** manner.

Using a **[zero-cost abstraction][zero runtime cost]**, **nested** data
transformations become [**untangled** into **short steps**][untangled flow].

<td>

```js
new User.Message(
  capitalize(
    doubledSay(
      await promise
        ??: throw new TypeError(
          `Invalid value from ${promise}`)
    ), ', '
  ) + '!'
)
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
  |> # ??: throw new TypeError(
    `Invalid value from ${promise}`)
  |> doubleSay(#, ', ')
  |> capitalize
  |> # + '!'
  |> new User.Message
```

With smart pipelines, the code above could be **terser** and, literally,
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
new User.Message(
  capitalize(
    doubledSay(
      await promise
        ??: throw new TypeError(
          `Invalid value from ${promise}`)
    ), ', '
  ) + '!'
)
```
Compared with the pipeline version, the original code requires **additional
indentation and grouping** on each step. This requires three more levels of
indentation and three more pairs of parentheses.

In addition, much related code is here separated by unrelated code. Rather than
a **uniform** postfix chain, operations appear **either before** the previous
step’s expression (`new User.Message(…)`, `capitalize(…)`, `doubledSay(…)`,
`await …`) but also **after** (`… ??: throw new TypeError()`, `… + '!'`).
An additional argument to function calls (such as `, ` in `doubledSay(…, ', ')`)
is also separated from its function calls, forming another easy-to-miss
“postfix” argument.


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

<td>

<tr>
<td>

For instance, the chained pipeline:
```js
5 |> # - 3
  |> -#
  |> # * 2
  |> Math.max(#, 0)
```

<td>

…is equivalent to the tangled nested expression:
```js
Math.max(
  -(5 - 3) * 2,
  0
)
```

<tr>
<td>

The syntax is [**statically term rewritable** into already valid code][term
rewriting] in this way, with [theoretically **zero runtime cost**][zero runtime
cost].

Similar use cases appear **numerous times** in JavaScript code, whenever any value
is transformed by **[expressions of any type][expressive versatility]**:
function calls, property calls, method calls, object constructions, arithmetic
operations, logical operations, bitwise operations, `typeof`, `instanceof`,
`await`, `yield` and `yield *`, and `throw` expressions.

<tr>
<td>

```js
promise
  |> await #
  |> # ??: throw new TypeError()
  |> doubleSay(#, ', ')
  |> capitalize
  |> # + '!'
  |> new User.Message
```
Note that, in the example above, it is **not necessary** to include
**parentheses** for `capitalize` or `new User.Message`; they were **tacitly
implied**, respectively forming a **tacit unary function call** and a **tacit
unary constructor call**. In other words, the example above is equivalent to
the version below.

<td>

```js
new User.Message(
  capitalize(
    doubledSay(
      await promise
        ??: throw new TypeError(
          `Invalid value from ${promise}`)
    ), ', '
  ) + '!'
)
```

<tr>
<td>

```js
promise
  |> await #
  |> # ??: throw new TypeError(
    `Invalid value from ${#}`)
  |> doubleSay(#, ', ')
  |> capitalize(#)
  |> # + '!'
  |> new User.Message(#)
```
This version is equivalent to the version above, except that the `capitalize`
and `new User.Message` pipeline bodies explicitly include optional topic
references `#`, making the expressions slightly wordier than necessary.

<td>

```js
new User.Message(
  capitalize(
    doubledSay(
      await promise
        ??: throw new TypeError(
          `Invalid value from ${promise}`)
    ), ', '
  ) + '!'
)
```

<tr>
<td>

Being able to automatically detect this **“bare style”** is the [**smart** part
of the “smart pipeline operator”][smart body syntax]. The styles of
[**functional** programming][functional programming], [**dataflow**
programming][dataflow programming], and [**tacit** programming][tacit
programming] may particularly benefit from bare pipelines and their [terse
function application][].

<td>

<tr>
<td>

```js
value
  |> f
  |> # + 2
  |> # * 3
  |> -#
  |> g(#, x)
```
This pipeline is a very flat expression, with only one level of indentation, and
with each transformation step on its own line.

Note that `|> f` is a bare unary function call. This is the same as `|> f(#)`,
but the topic reference `#` is unnecessary; it is invisibly, tacitly implied.

This is the [**smart** part of the smart pipeline operator][smart body syntax],
which can distinguish between two syntax styles (**bare style** vs. **topic
style**) by using a simple rule: **bare** style uses only **identifiers, dots,
and `new`**, and **never parentheses, brackets, braces**, or other
**operators**. And **topic** style **always** contains at least one **topic
reference**. For more information, see the reference below about the **[smart
body syntax][]**.

<td>

```js
g(
  -(f(value) + 2)
    * 3,
  x
)
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
value |> x + 50 |> f |> g(x, 2)
// 🚫 Syntax Error:
// Pipeline body `|> x + 50`
// binds topic but contains no topic reference.
// 🚫 Syntax Error:
// Pipeline body `|> f(x, 2)`
// binds topic but contains no topic reference.
```
In order to fulfill the [goal][goals] of [“don’t shoot me in the foot”][],
when a **pipeline is in topic style** but its **body has no topic reference**,
that is an **[early error][]**. Such a degenerate pipeline has a very good
chance of actually being an accidental bug. (Note that the bare-style pipeline
body `|> f` is *not* an error. The bare style is not supposed to contain any
topic references `#`.)

<td>

<tr>
<td>

```js
function doubleSay (str, separator) {
  return `${str}${separator}${string}`
}

function capitalize (str) {
  return str[0].toUpperCase()
    + str.substring(1)
}

promise
  |> await #
  |> # ??: throw new TypeError()
  |> doubleSay(#, ', ')
  |> capitalize |> # + '!'
  |> new User.Message
```
This pipeline is also relatively flat, with only one level of indentation, and
with each transformation step on its own line.

(`|> capitalize` is a bare unary function call equivalent to `|> capitalize(#)`.
Similarly, `|> new User.Message` is a bare unary constructor call, abbreviated
from `|> new User.Message(#)`.)

<td>

```js
function doubleSay (str, separator) {
  return `${str}${separator}${str}`
}

function capitalize (str) {
  return str[0].toUpperCase()
    + str.substring(1)
}

new User.Message(
  capitalizedString(
    doubledSay(
      await promise
        ??: throw new TypeError()
    ), ', '
  ) + '!'
)
```
This deeply nested expression has four levels of indentation instead of two.
Reading its data flow requires checking both the beginning of each expression
(`new User.Message`, `capitalizedString`, `doubledSay`, `await promise` and
end of each expression (`??: throw new TypeError()`, `, ', '`, ` + '!'`)).

<tr>
<td>

```js
… |> f(#, #)
… |> [#, # * 2, # * 3]
```
The topic reference may be used multiple times in a pipeline body. Each use
refers to the same value (wherever the topic reference is not overridden by
another, inner pipeline’s topic scope). Because it is bound to the result of the
topic, the topic is still only ever evaluated once.

<td>

```js
do {
  const $ = …;
  f($, $)
}
do {
  const $ = …;
  [$, $ * 2, $ * 3]
}
```
This is equivalent to storing the topic value in a unique variable, then using
that variable multiple times in an expression. `do` expressions are used here to
remain equivalent to the pipeline versions, which are themselves expressions that
are embeddable in other expressions.

<tr>
<td>

```js
promise
  |> await #
  |> # ??: throw new TypeError()
  |> `${#}, ${#}`
  |> #[0].toUpperCase() + #.substring(1)
  |> # + '!'
  |> new User.Message
```
When tiny functions are only used once, and when their bodies would be obvious and
self-documenting in meaning, then they might be ritual boilerplate that a developer
may prefer to inline: trading off self-documentation for localization of code.

<td>

```js
new User.Message(do {
  const value = do {
    const value = await promise
      ??: throw new TypeError()
    `${value}, ${value}`
  }
  value[0].toUpperCase()
    + value.substring(1)
} + '!')
```
Inlining these functions directly into nested expressions using `do` is
less successful than inlining the functions with the pipeline, both in
writability and in readability.

<tr>
<td>

```js
promise
  |> await #
  |> # ??: throw new TypeError()
  |> `${#}, ${#}`
  |> #[0].toUpperCase() + #.substring(1)
  |> # + '!'
  |> new User.Message
```

<td>

```js
do {
  const promiseValue = await promise
    ??: throw new TypeError()
  const doubledValue =
    `${promiseValue}, ${promiseValue}`
  const capitalizedValue
    = doubledValue[0].toUpperCase()
      + doubledValue.substring(1)
  const exclaimedValue
    = capitalizedValue + '!'
  new User.Message(exclaimedValue)
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
  |> # ??: throw new TypeError()
  |> normalize
  |> `${#}, ${#}`
  |> #[0].toUpperCase() + #.substring(1)
  |> # + '!'
  |> new User.Message
```
With a pipeline, there are no unnecessary variable identifiers. Inserting a new
step in between two steps (or deleting a step) only touches one new line. Here,
a call of a function `normalize` was inserted between the second and third steps.

<td>

```js
do {
  const promiseValue = await promise
    ??: throw new TypeError()
  const normalizedValue = normalize()
  const doubledValue =
    `${normalizedValue}, ${normalizedValue}`
  const capitalizedValue
    = doubledValue[0].toUpperCase()
      + doubledValue.substring(1)
  const exclaimedValue
    = capitalizedValue + '!'
  new User.Message(exclaimedValue)
}
```
This code underwent a similar insertion of `normalize`. With a series of
variables, inserting a new step in between two other steps (or deleting a step)
requires editing the variable names in the following step.

<tr>
<td>

```js
value
  |> x => # + x
```
The body of a pipeline may contain an inner arrow function but no other
type of block expression. Both versions of this code return an arrow function.

<td>

```js
do {
  const $ = value
  x => $ + x
}
```
The arrow function lexically closes over the topic value, takes one parameter,
and returns the sum of the topic value and the parameter.

<tr>
<td>

```js
value
  |> settimeout(() => # * 5)
```
This ability to create arrow functions, which do not lexically shadow the topic,
can be useful for using callbacks in a pipeline.

<td>

```js
do {
  const $ = value
  settimeout(() => $ * 5)
}
```
The topic value (here represented by a normal variable `$`) is still lexically
accessible within the arrow function’s body in both examples.

<tr>
<td>

```js
value
  |> (() => # * 5)
  |> settimeout
```
The arrow function can also be created on a separate pipeline step.

<td>

```js
do {
  const $ = value;
  settimeout(() => $ * 5)
}
```
The result here is the same.

<tr>
<td>

```js
value
  |> () => # * 5
  |> settimeout
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
(value |> ()) =>
  (# * 5 |> settimeout)
// 🚫 Syntax Error:
// Unexpected token `=>`.
// Cannot parse base expression.
```

<td>

<tr>
<td>

Both the head and the body of a pipeline may contain nested inner pipelines.
```js
value
  |> f(x =>
    # + x |> g |> # * 2)
  |> #.toString()
```

<td>

A nested pipeline works consistently. It merely shadows the outer context’s
topic with the topic within its own body’s inner context.
```js
do {
  const $ = value;
  f(x => g($ + x) * 2)
    .toString()
}
```

<tr>
<td>

```js
value
  |> # ** 2
  |> f(x => #
      |> g(#, x)
      |> [# * 3, # * 5])
```

<td>

```js
do {
  const $ = value ** 2;
  f(x => {
    const _$ = g($, x);
    return [_$ * 3, _$ * 5]
  })
}
```

<tr>
<td>

**Most statements cannot** use an **outside context’s topic** in their
expressions. This includes block statements; function, async-function,
generator, async-generator, and class definitions, `for` and `while` statements;
and `catch` clauses. (Exceptions include arrow functions and `do`, `if`, `try`,
`return`, `yield`, and `yield *` statements.)

This behavior is in order to fulfill the [goals][] of [simple scoping][] and of
[“don’t shoot me in the foot”][]: it prevents the origin of any topic from being
difficult to find.

```js
value |> function () { return # }
// 🚫 Syntax Error:
// Lexical context `function () { return # }`
// contains a topic reference
// but has no topic binding.
// 🚫 Syntax Error:
// Pipeline body `|> function () { … }`
// binds topic but contains no topic reference.
```
<td>

<tr>
<td>

```js
value |> class { m: () { return # }}
// 🚫 Syntax Error:
// Pipeline body `|> class { … }`
// binds topic but contains no topic reference.
```

<td>

<tr>
<td>

```js
value
  |> await f(#, 5)
  |> do {
    # + 30
  }
  |> g
```
The statements that may contain topic references from outer lexical environments
are **[`do` expressions][]**, **arrow functions, `if` statements**, `try`
statements (though not `catch` clauses), and `return`, `yield`, and `yield *`
statements.

This example demonstrates how a `do` expression can be a pipeline body if it
contains an outer topic reference `#`.

<td>

```js
g(await f(value, 5) + 30)
```

<tr>
<td>

```js
value
  |> await f(#, 5)
  |> do {
    if (# > 20)
      # + 30
    else
      # - 10
  }
  |> g
```
Using `do` expressions then allows the embedding of arbitrary statements such as
`if` statements inside pipeline bodies, greatly increasing their expressiveness.

<td>

```js
g(do {
  const value_ = await f(value, 5)
  if (value_ > 20)
    value_ + 30
  else
    value_ - 10
})
```

<tr>
<td>

```js
value
  |> await f(#, 5)
  |> do {
    if (x > 20)
      x + 30
    else
      x - 10
  }
  |> g
// 🚫 Syntax Error:
// Pipeline body `|> do { … }`
// binds topic but contains no topic reference.
```
But the same [early error rules][early errors] that apply to any topical
pipeline body apply also to topical bodies that are `do` expressions.
<td>


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
  |> playBlob
```

<td>

```js
fetch('/music/pk/altes-kamuffel')
  .then(res => res.blob())
  .then(playBlob)
```
```js
playBlob(
  await (
    await fetch('/music/pk/altes-kamuffel')
  ).blob()
)
```

<tr>
<td>

```js
'https://example.com/'
  |> await fetch(#, { method: 'HEAD' })
  |> #.headers.get('content-type')
  |> console.log
```

<td>

```js
fetch('https://example.com/',
  { method: 'HEAD' }
).then(response =>
  console.log(
    response.headers.get('content-type'))
)
```

<tr>
<td>

```js
'https://example.com/'
  |> await fetch(#, { method: 'HEAD' })
  |> #.headers.get('content-type')
  |> console.log
```

<td>

```js
console.log(
  (await
    fetch('https://example.com/',
      { method: 'HEAD' }
    )
  ).headers.get('content-type')
)
```

<tr>
<td>

```js
'https://example.com/'
  |> await fetch(#, { method: 'HEAD' })
  |> #.headers.get('content-type')
  |> console.log
```

<td>

```js
do {
  const url = 'https://example.com/'
  const response =
    await fetch(url, { method: 'HEAD' })
  const contentType =
    response.headers.get('content-type')
  console.log(contentType)
}
```

<tr>
<td>

```js
'https://pk.example/berlin-calling'
  |> await fetch(#, { mode: 'cors' })
  |> do {
    if (
      |> #.headers.get('content-type')
      |> #??.toLowerCase()
      |> #.indexOf('application/json')
      |> # >= 0
    )
      throw new new TypeError()
    else
      |> await #.json()
      |> processJSON
  }
}
```

<td>

```js
fetch('https://pk.example/berlin-calling',
  { mode: 'cors' }
).then(response => {
  if (response.headers.get('content-type')
    ??.toLowerCase()
    .indexOf('application/json') >= 0
  )
    return response.json()
  else
    throw new TypeError()
}).then(processJSON)
```

<tr>
<td>

```js
'https://pk.example/berlin-calling'
  |> await fetch(#, { mode: 'cors' })
  |> do {
    if (
      # |> #.headers.get('content-type')
        |> #??.toLowerCase()
        |> #.indexOf('application/json')
        |> # >= 0
    )
      throw new new TypeError()
    else
      # |> await #.json()
        |> processJSON
  }
}
```

<td>

```js
do {
  const response =
    await fetch('https://pk.example/berlin-calling',
      { mode: 'cors' }
    )
  const json = do {
    if (response.headers.get('content-type')
      ??.toLowerCase()
      .indexOf('application/json') >= 0
    ) {
      response.json()
    } else {
      throw new TypeError()
    }
  processJSON(json)
}
```

<tr>
<td>

```js
'https://pk.example/berlin-calling'
  |> await fetch(#, { mode: 'cors' })
  |> do {
    if (
      # |> #.headers.get('content-type')
        |> #??.toLowerCase()
        |> #.indexOf('application/json')
        |> # >= 0
    )
      throw new new TypeError()
    else
      # |> await #.json()
        |> processJSON
  }
}
```

<td>

```js
do {
  const response =
    await fetch('https://pk.example/berlin-calling',
      { mode: 'cors' }
    )
  const json = do {
    if (response.headers.get('content-type')
      ??.toLowerCase()
      .indexOf('application/json') >= 0
    ) {
      response.json()
    } else {
      throw new TypeError()
    }
  processJSON(json)
}
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
  |> jQuery.merge([], #)
```
The path that a reader’s eyes must trace while reading this pipeline moves
straight down, with some movement toward the right then back: from `data` to
`buildFragment` (and its arguments), then `.childNodes`, then `jQuery.merge`.
Here, no one-off-variable assignment is necessary.

<td>

```js
parsed = buildFragment(
  [ data ], context, scripts
)
return jQuery.merge(
  [], parsed.childNodes
)
```
From [jquery/src/core/parseHTML.js][]. In this code, the eyes first must look
for `data` – then upwards to `parsed = buildFragment` (and then back for
`buildFragment`’s other arguments) – then down, searching for the location of
the `parsed` variable in the next statement – then right when noticing its
`.childNodes` postfix – then back upward to `return jQuery.merge`.

<tr>
<td>

```js
(key |> toType) === 'object'
key |> toType |> # === 'object'
```
`|>` has a looser precedence than most operators, including `===`. (Only
assignment operators, arrow function `=>`, yield operators, and the comma
operator are any looser.)

<td>

```js
toType(key) === 'object'
```
From [jquery/src/core/access.js][].

<tr>
<td>

```js
context = context
  |> # instanceof jQuery
    ? #[0] : #
```

<td>

```js
context =
  context instanceof jQuery
    ? context[0] : context
```
From [jquery/src/core/access.js][].

<tr>
<td>

```js
context
  |> #??.nodeType
      ? #.ownerDocument || #
      : document
  |> jQuery.parseHTML(match[1], #, true)
  |> jQuery.merge
```

<td>

```js
jQuery.merge(
  this, jQuery.parseHTML(
    match[1],
    context??.nodeType
      ? context.ownerDocument
        || context
      : document,
    true
  )
)
```
From [jquery/src/core/init.js][]. Used `??.` in both versions for conciseness.

<tr>
<td>

```js
match |> do {
  if (this[match] |> isFunction)
    # |> context[#] |> this[match](#)
  else
    # |> context[#] |> this.attr(match, #)
}
```
Note how, in this version, the parallelism between the two clauses is very
clear: they both share the form `match |> context[#] |> something(match, #)`.

<td>

```js
if (isFunction(this[match])) {
  this[match](context[match])
} else
  this.attr(match, context[match])
}
```
From [jquery/src/core/init.js][]. Here, the parallelism between the clauses
is somewhat less clear: the common expression `context[match]` is at the end
of both clauses, at a different offset from the margin.

<tr>
<td>

```js
elem = match[2]
  |> document.getElementById
```

<td>

```js
elem = document.getElementById( match[ 2 ] )
```
From [jquery/src/core/init.js][].

<tr>
<td>

```js
return context |> do {
  // Handle HTML strings
  if (…)
    …
  // Handle $(expr, $(...))
  else if (!# || #.jquery)
    # |> # || root
      |> #.find(selector)
  // Handle $(expr, context)
  else
    # |> this.constructor
      |> #.find(selector)
}
```
The parallelism between the final two clauses becomes clearer here too.
They both are of the form `# |> something |> #.find(selector)`.

<td>

```js
// Handle HTML strings
if (…) {
  …
// Handle $(expr, $(...))
} else if (!context || context.jquery) {
  return (context || root).find(selector)
// Handle $(expr, context)
} else {
  return this.constructor(context)
    .find(selector);
}
```
From [jquery/src/core/init.js][]. The parallelism is much less clear here.

<tr>
<td>

```js
return selector |> do {
  if (typeof # === 'string')
    …
  else if (# |> isFunction) {
    if (root.ready !== undefined)
      root.ready(#)
    else
      #(jQuery)
  }
  else
    jQuery.makeArray(#, this)
}
```

<td>

```js
if (typeof selector === 'string') {
  …
} else if (isFunction(selector)) {
  return root.ready !== undefined
    ? root.ready(selector)
    : selector(jQuery)
}
return jQuery.makeArray(selector, this)
```
From [jquery/src/core/access.js][].

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
    |> _.filter(obj, #, context)
}
```

<td>

```js
function (obj, pred, context) {
  return _.filter(obj,
    _.negate(cb(pred)),
    context
  )
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
  return self
}
```

<tr>
<td>

```js
function (obj) {
  return obj |> do {
    if (# == null)
      0
    else if (|> isArrayLike)
      |> #.length
    else
      |> _.keys
      |> #.length
  }
}
```
Pipelines make parallelism between all three clauses becomes clearer: `0` for
the `if` clause, `# |> #.length` for the `else if`, and `# |> something |> #.length`
for the `else`. [This particular example becomes even clearer][Underscore.js +
CP + UP] when paired with [Additional Feature PP][].

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

## Additional Feature PP
The first Additional Feature adds a “headless” tacit prefix form of the pipeline
operator. The tacit, default head is the topic reference `#` itself, which
must be resolvable within the outer lexical environment. This may occur within
`if` statements, `try` statements, and eventually [`do` expressions][].

<table>
<thead>
<tr>
<th>With smart pipelines
<th>Status quo

<tbody>
<tr>
<td>

```js
x |> do {
  if (# |> predicate)
    # |> f |> # ** 2
  else
    # |> g |> # ** 3
}
```
In this version, which uses Core Proposal syntax only, several pipelines start
with the phrase `# |>`.

<td>

```js
do {
  if (predicate(x))
    f(x) ** 2
  else
    g(x) ** 3
}
```
Note that the topic reference in the repeated `# |>` here all refer to the same
topic from the same lexical environment – `x` – into `predicate`, `f`, and `g`.

<tr>
<td>

```js
x |> do {
  if (|> predicate)
    |> f |> # ** 2
  else
    |> g |> # ** 3
}
```
In this version, which also uses Additional Feature PP, those pipelines omit the
phrase `# |>`, using a tacit prefix pipeline `|>`, which is implied to use `#`
as the value of their topics.

<td>

```js
do {
  if (predicate(x))
    f(x) ** 2
  else
    g(x) ** 3
}
```
The unary pipeline `|>` still piped in the same tacit topic from the same
lexical environment – `x` – into `predicate`, `f`, and `g`. The result is still
the same as before.

<tr>
<td>

```js
function () {
  |> f |> g
}
// 🚫 Syntax Error:
// Lexical context `function () { |> f |> g }`
// contains a prefix pipeline `|> f`
// but has no topic binding.
```
If a prefix pipeline is used within a context in which the topic is not
resolvable, then this is an [early error][], just like how it is an error to use
explicit topic references within a context without a topic in general:

```js
function () {
  # |> f |> g
}
// 🚫 Syntax Error:
// Lexical context `function () { # |> f |> g }`
// contains a topic reference
// but has no topic binding.
```

<td>

</table>

### WHATWG Fetch Standard (Core Proposal + Additional Feature PP)
Revisiting the [examples above from the WHATWG Fetch Standard][WHATWG Fetch +
Core Proposal] with [Additional Feature PP][] shows how terseness could be
further improved within inner `do` expressions and inner `if` statements.

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
  |> do {
    if (
      # |> #.headers.get('content-type')
        |> #??.toLowerCase()
        |> #.indexOf('application/json')
        |> # >= 0
    )
      throw new new TypeError()
    else
      # |> await #.json()
        |> processJSON
  }
}
```
This pipeline version uses [Core Proposal][] syntax only. Note that several
expressions start with `# |>`.

<td>

```js
fetch('https://pk.example/berlin-calling',
  { mode: 'cors' }
).then(response => {
  if (response.headers.get('content-type')
    ??.toLowerCase()
    .indexOf('application/json') >= 0
  )
    return response.json()
  else
    throw new TypeError()
}).then(processJSON)
```

<tr>
<td>

```js
'https://pk.example/berlin-calling'
  |> await fetch(#, { mode: 'cors' })
  |> do {
    if (
      |> #.headers.get('content-type')
      |> #??.toLowerCase()
      |> #.indexOf('application/json')
      |> # >= 0
    )
      throw new new TypeError()
    else
      |> await #.json()
      |> processJSON
  }
}
```
This pipeline version also uses [Additional Feature PP][]. The repeated `# |>`
has been elided, but it is still tacitly there.

<td>

```js
fetch('https://pk.example/berlin-calling',
  { mode: 'cors' }
).then(response => {
  if (response.headers.get('content-type')
    ??.toLowerCase()
    .indexOf('application/json') >= 0
  )
    return response.json()
  else
    throw new TypeError()
}).then(processJSON)
```

</table>

### jQuery (Core Proposal + Additional Feature PP)
Similarly, revisiting the [examples above from jQuery][jQuery + Core Proposal]
with [Additional Feature PP][] shows how terseness could be further improved
within inner `do` expressions and inner `if` statements.

<table>
<thead>
<tr>
<th>With smart pipelines
<th>Status quo

<tbody>
<tr>
<td>

```js
match |> do {
  if (this[match] |> isFunction)
    # |> context[#] |> this[match](#)
  else
    # |> context[#] |> this.attr(match, #)
}
```
This pipeline version uses [Core Proposal][] syntax only. Note that several
expressions start with `# |>`.

<td>

```js
if (isFunction(this[match])) {
  this[match](context[match])
} else
  this.attr(match, context[match])
}
```
From [jquery/src/core/init.js][].

<tr>
<td>

```js
match |> do {
  if (this[match] |> isFunction)
    |> context[#] |> this[match](#)
  else
    |> context[#] |> this.attr(match, #)
}
```
This pipeline version also uses [Additional Feature PP][]. The repeated `# |>`
has been elided, but it is still tacitly there.

<td>

```js
if (isFunction(this[match])) {
  this[match](context[match])
} else
  this.attr(match, context[match])
}
```
From [jquery/src/core/init.js][].

<tr>
<td>

```js
return context |> do {
  // Handle HTML strings
  if (…)
    …
  // Handle $(expr, $(...))
  else if (!# || #.jquery)
    # |> # || root
      |> #.find(selector)
  // Handle $(expr, context)
  else
    # |> this.constructor
      |> #.find(selector)
}
```
This pipeline version uses [Core Proposal][] syntax only. Note that several
expressions start with `# |>`.

<td>

```js
// Handle HTML strings
if (…) {
  …
// Handle $(expr, $(...))
} else if (!context || context.jquery) {
  return (context || root).find(selector)
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
return context |> do {
  // Handle HTML strings
  if (…)
    …
  // Handle $(expr, $(...))
  else if (!# || #.jquery)
    |> # || root
    |> #.find(selector)
  // Handle $(expr, context)
  else
    |> this.constructor
    |> #.find(selector)
}
```
This pipeline version also uses [Additional Feature PP][]. The repeated `# |>`
has been elided, but it is still tacitly there.

<td>

```js
// Handle HTML strings
if (…) {
  …
// Handle $(expr, $(...))
} else if (!context || context.jquery) {
  return (context || root).find(selector)
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
return selector |> do {
  if (typeof # === 'string')
    …
  else if (# |> isFunction)
    root.ready !== undefined
      ? root.ready(#)
      : #(jQuery)
  else
    jQuery.makeArray(#, this)
}
```
This pipeline version uses [Core Proposal][] syntax only. Note that several
expressions start with `# |>`.

<td>

```js
if (typeof selector === 'string') {
  …
} else if (isFunction(selector)) {
  return root.ready !== undefined
    ? root.ready(selector)
    : selector(jQuery)
}
return jQuery.makeArray(selector, this)
```
From [jquery/src/core/access.js][].

<tr>
<td>

```js
return selector |> do {
  if (typeof # === 'string')
    …
  else if (# |> isFunction)
    root.ready !== undefined
      ? root.ready(#)
      : #(jQuery)
  else
    jQuery.makeArray(#, this)
}
```
This pipeline version also uses [Additional Feature PP][]. The repeated `# |>`
has been elided, but it is still tacitly there.

<td>

```js
if (typeof selector === 'string') {
  …
} else if (isFunction(selector)) {
  return root.ready !== undefined
    ? root.ready(selector)
    : selector(jQuery)
}
return jQuery.makeArray(selector, this)
```
From [jquery/src/core/access.js][].

</table>

### Underscore.js (Core Proposal + Additional Feature PP)
One of the [examples above from Underscore.js][Underscore.js + Core Proposal]
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
  return obj |> do {
    if (# == null)
      0
    else if (# |> isArrayLike)
      # |> #.length
    else
      # |> _.keys
        |> #.length
  }
}
```
Pipelines make parallelism between all three clauses becomes clearer.

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
  return obj |> do {
    if (# == null)
      0
    else if (|> isArrayLike)
      |> #.length
    else
      |> _.keys
      |> #.length
  }
}
```
By removing `# |>` noise, [Additional Feature PP][] makes this parallelism even
clearer: `0` for the `if` clause, `|> #.length` for the `else if` clause, and
`|> something |> #.length` for the `else` clause.

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

## Additional Feature CT
With the [Core Proposal][] only, all `try` statements’ `catch` clauses would
prohibit the use of the topic reference `#` within their bodies, except where
the topic reference `#` is inside an inner pipeline inside the `catch` clause:
this is one of the Core Proposal’s [early errors][] mentioned above.

The second Additional Feature makes all `catch` clauses implicitly bind their
caught errors to the topic reference `#`. This implicit binding would be in
addition to the explicit binding of a normal variable `error` declared within
any parenthesized antecedent `try { … } catch (error) { … }`.

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
  …
} catch {
  #.message |> console.log
} finally {
  …
}
```

The second Additional Feature makes all `catch` clauses implicitly bind their
caught errors to the topic reference `#`.

An additional bare `catch` form, completely lacking a parenthesized antecedent,
has already been proposed as [ECMAScript optional catch binding][]. This bare
form would also support the tacit topic binding.

<td>

```js
try {
  …
} catch (error) {
  log(error.message)
} finally {
  …
}
```

The implicit topic binding would be in addition to the explicit binding of a
normal variable `error` declared within any parenthesized antecedent.

</table>

## Additional Feature PF
The third Additional Feature introduces a **new prefix operator `=|> …`**, which
creates a new type of function, the **pipeline function**. `=|> …` interprets
its inner expression as a **pipeline body** but wraps it in a **unary arrow
function**, which plugs its single parameter into the pipeline body as if it
were a pipeline head. In other words, a pipeline function would act as if it
were `$ => $ |> …`, where `$` is a hygienically unique variable.

A pipe function takes **no** a parameter list; its unary parameter is implicitly
bound to the tacit pipeline head. And just like with regular pipelines, a
pipeline function may be in **bare style or topic style**.

**More than any other** possible extension in this table, pipeline functions would
dramatically increase the potential of tacit programming. Just this single
additional operator seems to solve:\
tacit unary-**functional composition**,\
tacit unary-functional **partial application**,\
and tacit **method extraction**,\
…all with a single additional concept.

The precise appearance of the pipeline-function operator does not have to be
`=|>`. It could also be `=|`, `->`, or something else to be decided after
much bikeshedding.

<table>
<thead>
<tr>
<th>With smart pipelines
<th>Status quo

<tbody>
<tr>
<td>

```js
array.map($ => $ |> #)
```
```js
array.map($ => $)
```
These functions are the same. They both pipe a unary parameter into a
topic-style pipeline whose bodies evaluate simply to the topic, unmodified.

<td>

```js
array.map($ => $)
```
In other words, they are both [identity function][]s.

<tr>
<td>

```js
array.map($ => $ |> # + 2)
array.map(=|> # + 2)
```
These functions are also the same with each other. They both pipe a unary
parameter into a topic-style pipeline whose bodies are the topic plus two.

<td>

```js
array.map($ => $ + 2)
```

<tr>
<td>

```js
array.map($ => $ |> f)
array.map(=|> f)
```
These functions are also the same as each other. However, their pipelines
are in **bare mode**, so no topic reference is needed in their bodies.

<td>

```js
array.map($ => f($))
```

<tr>
<td>

```js
array.map($ => $ |> f |> g |> h |> # * 2)
array.map(=|> f |> g |> h |> # * 2)
```
Pipelines may be chained within a pipeline function. The prefix
pipeline-function operator `=|>` would have looser precedence than the infix
pipeline operator `|>`.

<td>

```js
array.map($ => h(g(f($))) * 2)
```

<tr>
<td>

```js
array.map(=|> |> f |> g |> h |> # * 2)
array.map($ =|> # |> f |> g |> h |> # * 2)
array.map($ => $ |> # |> f |> g |> h |> # * 2)
array.map($ => $ |> f |> g |> h |> # * 2)
array.map($ =|> f |> g |> h |> # * 2)
```
When coupled with [Additional Feature PP][], the phrase `=|> |>` (that is,
prefix pipeline function `|=>` immediately followed by prefix pipeline `|>`)
cancels out into simply the prefix pipeline function `=|>`. All four of these
expressions here are equivalent.

<td>

```js
array.map($ => h(g(f($))) * 2)
```

<tr>
<td>

```js
=|> x + 2
// 🚫 Syntax Error:
// Pipeline body `=|> x + 2`
// binds topic but contains no topic reference.
```

This is an [early error][], as usual. The topic is not used anywhere
in the pipeline function’s body – just like with `… |> x + 2`.

<td>

<tr>
<td>

```js
=> # + 2
// 🚫 Syntax Error:
// Unexpected token `=>`.
```
If the pipeline-function operator `=|>` is typoed as an arrow function `=>`
instead, then this is another syntax error, because the arrow function `=>`
expects to always have a parameter antecedent as its head.

<td>

<tr>
<td>

```js
() => # + 2
// 🚫 Syntax Error:
// Lexical context `() => # + 2`
// contains a topic reference
// but has no topic binding.
```
But even if that typo also includes a parameter head for the arrow function
`=>`, this is would still be an [early error][]…unless the outer
lexical environment does have a topic binding.

<td>

<tr>
<td>

```js
array.map(=|> f |> g |> h(2, #) |> # + 2)
```
**Functional composition** on unary functions is equivalent to piping a value
through several function calls, within a unary function, starting with the outer
function’s single tacit parameter.

<td>

```js
array.map($ => h(2, g(f($))) + 2)
```

<tr>
<td>

```js
const doubleThenSquareThenHalfAsync =
  =|> double |> await squareAsync # |> half
```
Unlike the other version, this syntax does not need to give implicit special
treatment to async functions.

<td>

```js
const doubleThenSquareThenHalfAsync =
  double +> squareAsync +> half
```
From the proposal for [syntactic functional composition][]
by [Gilbert “mindeavor”][mindeavor].

<tr>
<td>

**Partial application into a unary function** is equivalent to piping a tacit
parameter into a function-call expression, within which the one parameter is
resolvable.
```js
array.map($ => $ |> f(2, #))
array.map(=|> f(2, #))
```

<td>

Pipeline functions look similar to the proposal for [syntactic partial
application][] by [Ron Buckton][], except that partial-application expressions
are simply pipeline bodies that are prefixed by a topic arrow.
```js
array.map(f(2, ?))
array.map($ => f(2, $))
```

<tr>
<td>

```js
const addOne = =|> add(1, #)
addOne(2) // 3
```

<td>

```js
const addOne = add(1, ?)
addOne(2) // 3
```

<tr>
<td>

```js
const addTen = =|> add(#, 10)
addTen(2) // 12
```

<td>

```js
const addTen = add(?, 10)
addTen(2) // 12
```

<tr>
<td>

```js
let newScore = player.score
  |> add(7, #)
  |> clamp(0, 100, #)
```

<td>

```js
let newScore = player.score
  |> add(7, ?)
  |> clamp(0, 100, ?)
```

<tr>
<td>

```js
Promise.resolve(123).then(=|> console.log)
```
**Method extraction** can be addressed by pipeline functions alone, as a natural
result of their pipe-operator-like semantics.\
`=|> console.log` is equivalent to `$ => $ |> console.log`, which is a pipeline in
bare style. This in turn is `$ => console.log($)`…

<td>

```js
Promise.resolve(123).then(console.log.bind(console))
Promise.resolve(123).then(::console.log)
```
And `$ => console.log($)` is equivalent to `console.log.bind(console)`.

<tr>
<td>

```js
$('.some-link').on('click', =|> view.reset)
```

<td>

```js
$('.some-link').on('click', ::view.reset)
```

<tr>
<td>

```js
const { hasOwnProperty } = Object.prototype
const x = { key: 5 }
x::hasOwnProperty
x::hasOwnProperty('key')
```
To do terse **method calling/binding**, the `::` operator would still be required.

<td>

```js
const { hasOwnProperty } = Object.prototype
const x = { key: 5 }
x::hasOwnProperty
x::hasOwnProperty('key')
```
But the `::` would only need to handle method calls. No operator overloading of
`::` for method extraction would be needed.

</table>

### Lodash
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
    |> do {
      if (nativeCreate)
        |> #[key]
        |> # === HASH_UNDEFINED
          ? undefined : #
      else
        |> hashOwnProperty.call(#, key)
          ? #[key] : undefined
    }
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
  (this, key)
    |> getMapData
    |> #['delete']
    |> #(key)
    |> do {
      this.size -= # ? 1 : 0
      return #
    }
}
```

<td>

```js
function mapCacheDelete (key) {
  var result =
    getMapData(this, key)['delete'](key)
  this.size -= result ? 1 : 0
  return result
}
```

<tr>
<td>

```js
function castPath (value, object) {
  return value |> do {
    if (|> isArray)
      #
    else if (|> isKey(#, object))
      [#]
    else
      |> toString |> stringToPath
  }
}
```

<td>

```js
function castPath (value, object) {
  if (isArray(value)) {
    return value
  }
  return isKey(value, object)
    ? [value]
    : stringToPath(toString(value))
}
```

<tr>
<td>

```js
function createRound (methodName) {
  var func = Math[methodName]
  return function (number, precision) {
    number = number |> toNumber
    precision = precision |> do {
      if (# == null)
        0
      else
        |> toInteger |> nativeMin(#, 292)
    } {
      if (precision) {
        // Shift with exponential notation
        // to avoid floating-point issues.
        // See https://mdn.io/round#Examples.
        number
          |> `${#}e`
          |> ...#.split('e')
          |> `${#}e${+## + precision}`
          |> func
          |> `${#}e`
          |> ...#.split('e')
          |> `${#}e${+## - precision}`
          |> +#
      } else
        number
          |> func
    }
  }
}
```

<td>

```js
function createRound (methodName) {
  var func = Math[methodName]
  return function (number, precision) {
    number = toNumber(number)
    precision = precision == null
      ? 0
      : nativeMin(toInteger(precision), 292)
    if (precision) {
      // Shift with exponential notation
      // to avoid floating-point issues.
      // See https://mdn.io/round#Examples.
      var pair = (toString(number) + 'e')
        .split('e'),
      value = func(
        pair[0] + 'e' + (
          +pair[1] + precision))

      pair = (toString(value) + 'e')
        .split('e')
      return +(
        pair[0] + 'e' + (
          +pair[1] - precision))
    }
    return func(number)
  }
}

```

</table>

## Additional Feature MT
Another Additional Feature introduces **multiple lexical topics** at once: not
only the **primary** topic reference `#`, but also the **secondary** `##`,
**tertiary** `###`, and **rest** `...` **topic references**. It also enables
both **n-ary application** and **n-ary partial application**.

This would be somewhat akin to [Clojure’s compact anonymous functions][Clojure
compact function], which use `%` aka `%1`, then `%2`, `%3`, … for its parameters
within the compact functions’ bodies.

This explainer limits this Additional Feature to three topic references plus a
rest topic reference. This limit could theoretically be lifted, but readability
would rapidly suffer with five, six, seven different topics at once. Arrow
functions could always be used instead for such many-parameter functions.\
The precise appearances of the secondary, tertiary, and rest topic references do
not have to be `##`, `###`, and `...`. For instance, they could instead be `#1`,
`#2`, and `#...`; this is yet to be bikeshedded.

<table>
<thead>
<tr>
<th>With smart pipelines
<th>Status quo

<tbody>
<tr>
<td>

```js
(a, b)
  |> f
```

<td>

```js
f(a, b)
```

<tr>
<td>

```js
(a, b, ...c, d)
  |> f
```

<td>

```js
f(a, b, ...c, d)
```

<tr>
<td>

```js
(a, b)
  |> f(#, x, ##)
```

<td>

```js
f(a, x, b)
```

<tr>
<td>

```js
(a, b, ...c, d)
  |> f(#, x, ...)
```

<td>

```js
f(a, ...[b, ...c, d])
```

<tr>
<td>

```js
(a, b, ...c, d)
  |> f(#, ###, x, ...)
```

<td>

```js
do {
  const [_2, ..._spread] = c
  f(a, _2, x, ...[...c, d])
}
```

<tr>
<td>

```js
(a, b)
  |> # - ##
```

<td>

```js
a - b
```

<tr>
<td>

```js
(a, b)
  |> (f, # ** c + ##)
  |> # - ##
```

<td>

```js
f(a) - (a ** c + b)
```

<tr>
<td>

```js
...a
  |> #.toString()
```

<td>

```js
[...a].toString()
```

<tr>
<td>

```js
array.sort(=|> # - ##)
```

<td>

```js
array.sort((_0, _1) => _0 - _1)
```

<tr>
<td>

```js
[ { x: 22 }, { x: 42 } ]
  .map(=|> #.x)
  .reduce(=|> # - ##, 0)
```

<td>

```js
[ { x: 22 }, { x: 42 } ]
  .map(el => el.x)
  .reduce((_0, _1) => _0 - _1, 0)
```

<tr>
<td>

```js
const f = (x, y, z) => [x, y, z]
const g = f(#, 4, ##)
g(1, 2) // [1, 4, 2]
```
**Partial application into an n-ary function** is solved by pipeline functions with
multiple topics.

<td>

```js
const f = (x, y, z) => [x, y, z]
const g = =|> f(?, 4, ?)
g(1, 2) // [1, 4, 2]
```
[R. Buckton’s current proposal][syntactic partial application] assumes that each
use of the same `?` placeholder token represents a different parameter. In contrast,
each use of `#` within the same scope always refers to the same value. This is
why additional topic parameters are required. The resulting model is more
flexible: `=|> f(#, 4, ##)` is different from `=|> f(#, 4, #)`. The latter sensibly
refers to a *unary* function that passes the same *one* argument into both the
first and third parameters of the original function `f`.

<tr>
<td>

```js
const maxGreaterThanZero =
  =|> Math.max(0, ...)
maxGreaterThanZero(1, 2) // 2
maxGreaterThanZero(-1, -2) // 0
```
Partial application into a variadic function requires a multi-topic environment
and a rest-topic reference `...`.

<td>

```js
const maxGreaterThanZero =
  Math.max(0, ...)
maxGreaterThanZero(1, 2) // 2
maxGreaterThanZero(-1, -2) // 0
```
In this case, the topic function version looks once again nearly identical to
the other proposal’s code.

</table>

### Ramda
[Ramda][] is a utility library focused on [functional programming][] with [pure
functions][] and [immutable objects][]. Its functions are automatically
[curried][currying]. Smart pipelines with [Additional Feature PF][]
would address many of Rambda’s use cases, particularly when also coupled with
[Additional Feature MT][]. The examples below were taken from the [Ramda wiki
cookbook][]. They use smart pipelines with vanilla JavaScript APIs when possible
(such as `Array.prototype.map` instead of `R.map`), but they also use Ramda
functions wherever no terse JavaScript equivalent yet exists (such as with
`R.zipWith` and `R.adjust`).

<table>
<thead>
<tr>
<th>With smart pipelines
<th>Status quo

<tbody>
<tr>
<td>

```js
const pickIndexes = =|> R.values |> R.pickAll
['a', 'b', 'c'] |> pickIndexes([0, 2], #)
// ['a', 'c']
```

<td>

```js
const pickIndexes = R.compose(
  R.values, R.pickAll)
pickIndexes([0, 2], ['a', 'b', 'c'])
// ['a', 'c']
```

<tr>
<td>

```js
const list = =|> [...]
list(1, 2, 3)
// [1, 2, 3]
```

<td>

```js
const list = R.unapply(R.identity)
list(1, 2, 3)
// [1, 2, 3]
```

<tr>
<td>

```js
const cssQuery = =|> ##.querySelectorAll(#)
const setStyle = =|> { ##.style = # }
document
  |> cssQuery('a, p', #)
  |> #.map(=|> setStyle({ color: 'red' }))
```

<td>

```js
const cssQuery = R.invoker(1,
  'querySelectorAll')
const setStyle = R.assoc('style')
R.pipe(
  cssQuery('a, p'),
  R.map(setStyle({ color: 'red' }))
)(document)
```

<tr>
<td>

```js
const disco = =|>
  |> R.zipWith(=|> #(##),
    [ red, green, blue ])
  |> #.join(' ')
[ 'foo', 'bar', 'xyz' ]
  |> disco
  |> console.log
```

<td>

```js
const disco = R.pipe(
  R.zipWith(
    R.call,
    [ red, green, blue ]),
  R.join(' ')
)
console.log(
  disco([ 'foo', 'bar', 'xyz' ]))
```

<tr>
<td>

```js
const dotPath = =|>
  |> (#.split('.'), ##)
  |> R.path(#, ##)
const propsDotPath = =|>
  |> (R.map(dotPath), [##])
  |> R.ap
const obj = {
  a: { b: { c: 1 } },
  x: 2
}
propsDotPath(['a.b.c', 'x'], obj)
// [ 1, 2 ]
```

<td>

```js
const dotPath = R.useWith(
  R.path,
  [R.split('.')])
const propsDotPath = R.useWith(
  R.ap,
  [R.map(dotPath), R.of])
const obj = {
  a: { b: { c: 1 } },
  x: 2
}
propsDotPath(['a.b.c', 'x'], obj)
// [ 1, 2 ]
```

<tr>
<td>

```js
const getNewTitles = async =|>
  |> await fetch
  |> parseJSON
  |> #.flatten()
  |> #.map(=|> #.items)
  |> #.map(=|> #.filter(=|> #))
  |> #.map(=|> #.title)

try {
  '/products.json'
    |> getNewTitles
    |> console.log
} catch {
  |> console.error
}

const fetchDependent = async =|>
  |> await fetch
  |> JSON.parse
  |> #.flatten()
  |> #.map(=|> #.url)
  |> #.map(fetch)
  |> #.flatten()

try {
  'urls.json'
    |> fetchDependent
    |> console.log
} catch {
  |> console.error
}
```

<td>

```js
const getNewTitles = R.compose(
  R.map(R.pluck('title')),
  R.map(R.filter(R.prop('new'))),
  R.pluck('items'),
  R.chain(JSON.parse),
  fetch
)

getNewTitles('/products.json')
  .fork(console.error, console.log);

const fetchDependent = R.compose(
  R.chain(fetch),
  R.pluck('url'),
  R.chain(parseJSON),
  fetch
)

fetchDependent('urls.json')
  .fork(console.error, console.log)
```

<tr>
<td>

```js
number
  |> R.repeat(Math.random, #)
  |> #.map(=|> #())
```

<td>

```js
R.map(R.call,
  R.repeat(Math.random, number))
```

<tr>
<td>

```js
const renameBy = (fn, obj) =>
  [...obj]
    |> #.map(R.adjust(fn, 0)),
    |> {...#}
{ A: 1, B: 2, C: 3 }
  |> renameBy(=|> `a${#}`))
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
)
renameBy(R.concat('a'), { A: 1, B: 2, C: 3 })
// { aA: 1, aB: 2, aC: 3 }
```

</table>

### WHATWG Streams Standard
The [WHATWG Streams Standard][] provides an efficient, standardized stream API,
inspired by Node.js’s Streams API, but also applicable to the DOM. The
specification contains numerous usage examples that would become more readable
with smart pipelines. The Core Proposal alone would untangle much of this code,
and the Additional Features would further improve its terseness.

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
    |> console.log
} catch {
  ("Error", #)
    |> console.error
}
```

<td>

```js
readableStream.pipeTo(writableStream)
  .then(() => console.log("Success"))
  .catch(e => console.error("Error", e))
```

<tr>
<td>

```js
const reader = readableStream
  .getReader({ mode: "byob" })

try {
  new ArrayBuffer(1024)
    |> await readInto
    |> ("The first 1024 bytes:", #)
    |> console.log
} catch {
  ("Something went wrong!", #)
    |> console.error
}

async function readInto(buffer, offset = 0) {
  return buffer |> do {
    if (#.byteLength === offset)
      #
    else
      |> (#, offset, #.byteLength - offset)
      |> new Uint8Array
      |> await reader.read
      |> (#.buffer, #.byteLength)
      |> readInto(#, offset + ##)
  }
}
```

<td>

```js
const reader = readableStream
  .getReader({ mode: "byob" })

let startingAB = new ArrayBuffer(1024)
readInto(startingAB)
  .then(buffer =>
    console.log("The first 1024 bytes:", buffer))
  .catch(e =>
    console.error("Something went wrong!", e))

function readInto(buffer, offset = 0) {
  if (offset === buffer.byteLength) {
    return Promise.resolve(buffer);
  }
  const view = new Uint8Array(
    buffer, offset, buffer.byteLength - offset)
  return reader.read(view).then(newView => {
    return readInto(newView.buffer,
      offset + newView.byteLength);
  })
}
```

<tr>
<td>

```js
class LipFuzzTransformer {
  constructor(substitutions) {
    this.substitutions = substitutions;
    this.partialChunk = "";
    this.lastIndex = undefined;
  }

  transform(chunk, controller) {
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
        =|> this.replaceTag)
      |> partialAtEndRegexp.exec
      |> do {
        if (#) {
          this.partialChunk =
            |> #.index
            |> chunk.substring
          # |> #.index
            |> chunk.substring(0, #)
        }
        else
          chunk
      }
      |> controller.enqueue
  }

  flush(controller) {
    this.partialChunk |> do {
      if (#.length > 0) {
        |> controller.enqueue
      }
    }
  }

  replaceTag(match, p1, offset) {
    return this.substitutions
      |> #[p1]
      |> # === undefined ? '' : #
      |> do {
        this.lastIndex =
          |> #.length
          |> offset + #
        #
      }
    }
  }
}
```

<td>

```js
class LipFuzzTransformer {
  constructor(substitutions) {
    this.substitutions = substitutions;
    this.partialChunk = "";
    this.lastIndex = undefined;
  }

  transform(chunk, controller) {
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

  flush(controller) {
    if (this.partialChunk.length > 0) {
      controller.enqueue(
        this.partialChunk);
    }
  }

  replaceTag(match, p1, offset) {
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

# Goals

There are seventeen ordered goals that the smart body syntax tries to fulfill,
which may be summarized,<br>
“Don’t break my code,”<br>
“Don’t make me overthink,”<br>
“Don’t shoot me in the foot,”<br>
“Make my code easier to read,”<br>
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
 5. [Cyclomatic simplicity](#cyclomatic-simplicity)
 6. [Expressive versatility](#expressive-versatility)

<td>

**“Don’t shoot me in the foot.”**

 7. [Simple scoping](#simple-scoping)
 8. [Static analyzability](#static-analyzability)
 9. [Arbitrary associativity](#arbitrary-associativity)

<tr>
<td>

**“Make my code easier to read.”**

10. [Untangled flow](#untangled-flow)
11. [Distinguishability](#distinguishability)
12. [Terse parentheses](#terse-parentheses)
13. [Terse variables](#terse-variables)
14. [Terse function calls](#terse-function-calls)

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
may also be difficult to debug (see [Goal 6][TODO]).

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
The syntax should not preclude other proposals: both already-proposed features,
such as [syntactic partial application][] and [private class fields][] – as well
as [possible future extensions to the topic concept][], such as topic-binding
versions of `function`, `for`, and `catch` blocks.

This proposal is forward compatible with all these proposals, in both its choice
of topic reference and in its prohibition of topic references within any block
(other than arrow functions).

Forward compatibility is elaborated in the section on [relations to other
work][]. See also [Goal 9][TODO] below. See also [inner blocks in pipelines][inner
blocks].

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

33. Functional partial application?
34. Unary-function composition?
35. Function binding?

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
behavior, may cause different performance behavior (see example in [Goal 2][TODO]), or
may require dramatic rearrangement of logic to conserve the old code’s behavior.

It would be possible to add ad-hoc handling, for selected other expression
types, to the operator’s grammar. This would expand its benefits to that type.
However, this conflicts with [Goal 5][TODO] (adding cyclomatic complexity to the parsing
process, proportional to the number of ad-hoc handled cases). It also does not
fulfill this goal well either: excluding, perhaps arbitrarily, whatever classes
its grammar’s branches do not handle.

Such new [incidental complexity][] makes code less readable and distracts the
developer from the program’s [essential logic][essential complexity]. A pipeline
operator that improves readability should be versatile (this goal) but
conceptually and cyclomatically simple ([Goal 5][TODO]). Such an operator should be able
to handle **all** expressions, in a **single** manner **uniformly**
**universally** applicable to **all** expressions. It is the hope of this
proposal’s authors that its [smart body syntax][] fulfills both criteria.

## “Don’t shoot me in the foot.”
The syntax should not be a footgun: it should not easy for a developer to
accidentally shoot themselves in the foot with it.

### Simple scoping
It should not be easy to accidentally shadow a reference from an outer lexical
scope. When the developer does so, any use of that reference could result in
subtle, pernicious bugs.

The rules for when the topic is bound should be simple and consistent. It should
be clear and obvious when a topic is bound and in what scope it exists. And
forgetting these rules should result in early, compile-time errors, not subtle
runtime bugs.

The rules of topic scoping is simple: **Topic references are bound in the bodies
of pipelines, and they cannot be used within any block other than arrow
functions.** See the section on [inner blocks][].

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

### ASI safety
[TODO]

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
new User.Message(
  capitalize(
    doubledSay(
      (await promise)
        ??: throw new TypeError(`Invalid value from ${promise}`)
    )
  ) + '!'
)
```
…the deep inner expression `await promise` is relatively short. In
contrast, the shallow outer expression
`` capitalize(doubledSay((await promise) ??: throw new TypeError(`Invalid value from ${promise}`))) + '!'`) ``
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
  |> # ??: throw new TypeError()
  |> doubleSay(#, ', ')
  |> capitalize
  |> # + '!'
  |> new User.Message
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
of expressions. If the topic reference hypothetically were `?`, and it were used
anywhere near the visually similar [optional-chaining syntax proposal][], then
the topic reference might be lost or unnoticed by the developer: for example,
`?.??m(?)`.

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
be balanced with [Goals 4 and 5][TODO]. Excessive implicitness compromises
comprehensibility, at least without low-level tracing of tacit arguments’
invisible paths, rather than the actual, high-level meaning of the code. Yet at
the same time, excessive explicitness generates ritual, verbose boilerplate that
also interferes with reading comprehension. Therefore, [Goal 10][TODO] must be balanced
with [Goals 1, 4, and 5][TODO].

[The Zen of Python][PEP 20] famously says, “Explicit is better than implicit,”
but it also says, “Flat is better than nested,” and, “Sparse is better than
dense.”

### Terse function calls
Unary function / constructor calls are a particularly frequent type of
expression and a good target for especial human optimization. However, such
extra shortening might dramatically reduce the verbosity of unary function
calls, but again this must be balanced with [Goals 1, 4, and 5][TODO].

It is the hope of this proposal’s authors that its [smart body syntax][] reaches
a good balance between this goal and [Goals 4 and 5][TODO], in the same manner that
[Huffman coding][] optimizes textual symbols’ length for their frequency of use:
more commonly used symbols are shorter.

## Other Goals
Although these have been prioritized last, they are still important.

### Conceptual generality
If a concept is uniformly generalizable to many other cases, then this
multiplies its usefulness. The more versatile its concepts, the more it may be
applied to other syntax, including existing syntax and future syntax (compare
with [Goal 3][TODO]).

This proposal’s concept of a **topic reference does not need to be coupled only
to pipelines**. The [topic concept is **generalizable to many syntactic
forms**][possible future extensions to the topic concept]. These generalizations
are **out of scope** of this proposal, which is only for the smart pipe
operator; they are **deferred** to [other, future proposals][possible future
extensions to the topic concept].

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

Achieving [Goal 8][TODO] therefore also improves the ease of composing and editing code.
By flattening deeply nested expression trees into single threads of postfix
steps, a step may be added oredited in isolation on a single line, it may be
rearranged up or down, it may be removed – all without affecting the pipeline’s
other steps in the lines above or below it.

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

# Smart body syntax
[TODO]

# Relations to other work

[TODO: https://github.com/gajus/babel-plugin-transform-function-composition]

[TODO: refer to #background list of programming languages]

The concept of a pipeline operator appears in numerous other languages, variously
called “pipeline”, “threading”, and “feed” operators. This is because developers
find the concept useful.

<table>
<tr>
<th>

`|`

<td>

[Unix shells, PowerShell][Unix pipe]

<tr>
<th>

`|>`

<td>

[Elixir and Erlang][Elixir pipe], [Elm][Elm pipe], [F# / F-sharp][F# pipe],
[Hack][Hack pipe], [Julia][Julia pipe], [LiveScript][LiveScript pipe], [OCaml][OCaml pipe],

<tr>
<th>

`|>` with `$$`

<td>

[Hack][Hack pipe]

<tr>
<th>

`%>%`

<td>

[R with magrittr][R pipe]

<tr>
<th>

`==>`

<td>

[Perl 6][Perl 6 pipe]

<tr>
<th>

`=|>` `=|>>`\
`as->` `as->>`\
`some->` `some->>`\
`cond->` `cond->>`

<td>

[Clojure][Clojure pipe]

<tr>
<th>

[Term concatenation][concatenative programming]

<td>

Factor, Forth, Joy, Onyx, PostScript, RPL

</table>

Pipeline operators are also conceptually similar to [WHATWG-stream piping][] and
[Node-stream piping][].


## Other ECMAScript proposals
[TODO: `do` expressions]

[TODO: Partial application: “topic reference” vs. “placeholder”.]

[TODO: Private class fields and `#`.]

[TODO: Class decorators and `@`.]

[TODO: Block params: https://github.com/samuelgoto/proposal-block-params]

[TODO: Function bind: https://github.com/zenparsing/es-function-bind]

[TODO: pattern matching https://github.com/tc39/proposal-pattern-matching]

### `do` expressions

## Possible future extensions to the topic concept
The [concept of the “topic variable” already exists in many other programming
languages][topic variables in other languages], commonly named with an
underscore `_` or `$_`. These languages often integrate their topic variables
into their function-call control-flow syntaxes, with [Perl 6 as perhaps the most
extensive, unified example][Perl 6 topicization]. Integration of topic with
syntax enables especially pithy, terse [tacit programming][].

In addition, many JavaScript console [REPLs][], such as those of the WebKit Web
Inspector and the Node.js interactive console… [TODO]

Several disadvantages to these prior approaches may increase the probability of
developer surprise, in which “surprise” refers to behavior difficult to predict
by the developer.

One disadvantage arises from their frequent dynamic binding rather than lexical
binding, as the former is not [statically analyzable][static analyzability] and is more stateful than
the latter. It may also cause surprising results when coupled with bare/tacit
calls: it becomes more difficult to tell whether a bare identifier `print` is
meant to be a simple variable reference or a bare function call on the topic
value.

Another disadvantage arises from the ability to clobber or overwrite the value of the
topic variable, which may affect code in surprising ways.

However, JavaScript’s topic reference `#` is different than this prior art. It
is lexically bound and statically analyzable. It is also cannot be accidentally
bound; the developer must opt into binding it by using the pipeline operator.

The topic also cannot be accidentally used; it is a syntax error when `#` is used
outside of a pipeline body. [TODO: Link to pertinent grammar sections.]

The topic is [conceptually general][conceptual generality] and could be extended
to other forms. This proposal is [forward compatible][forward compatibility]
with such extensions, which would increase its [expressive versatility][], and
potentially multiplying its benefits toward [untangled flow][], [terse
variables][], and [human writability][], while still preserving [simple
scoping][] and [static analyzability][].

<table>
<tr>

<th>

Topic `for` loops

<td>

With this smart-pipe proposal only, `for`–`of` statements would prohibit the use
of `#` within their bodies, except where `#` is inside an inner pipeline inside
the `for` loop.

With another, future proposal, all `for`–`of` loops would implicitly bind each
iterator value to `#`. This implicit binding would be in addition to the
explicit binding of a normal variable `i` declared within the parenthesized
antecedent `for (const i of … { … })`.

An additional tacit `for` loop form, completely lacking a parenthesized
antecedent, would also be added. This tacit form is what is used in this example.
[TODO: Link to section on deep nesting.] This example also uses the hypothetical
headless pipelining syntax from above.

<tr>
<td>

```js
for (range(0, 50)) {
  log(# ** 2);
  log(#|> Math.sqrt);
}
```

<td>

```js
for (const i of range(0, 50)) {
  log(i |> # ** 2);
  log(i |> Math.sqrt);
}
```

<tr>
<th>

Topic `for`–`await` loops

<td>

This is similar to the tacit topic synchronous `for` loop above. With this
proposal only, `for`–`await`–`of` statements would prohibit the use of `#`
within their bodies, except where `#` is inside an inner pipeline inside the
`for` loop.

With another, future proposal, all `for`–`await`–`of` loops would implicitly bind
each iterator value to `#`. This implicit binding would be in addition to the
explicit binding of a normal variable `i` declared within the parenthesized
antecedent `for await (const i of …) { … }`.

An additional tacit `for await` loop form, completely lacking a parenthesized
antecedent, would also be added. This tacit form is what is used in this
example. [TODO: Link to section on deep nesting.] This example also uses the
hypothetical headless pipelining syntax from above.

<tr>
<td>

```js
for await (stream) {
  yield #
    |> f
    |> # + 3
}
```

<td>

```js
for await (const c of stream) {
  yield c
    |> f
    |> # + 3
}
```

<tr>

<th>

Topic block parameters

<td>

The proposed syntax of [ECMAScript block parameters][] may greatly benefit from
using the topic concept. As with topic function definitions, making all block
parameters topic would enable the use of the topic reference as an implicit
first parameter.

<tr>
<td>

```js
materials.map { #|> f |> .length }
```
(Here, `#|> f` is just a stylistic variant of `# |> f`, which is already valid
in this proposal’s rules.)

Note that this would be the same as:
```js
materials.map(=|> f |> .length)
```

<td>

```js
materials.map { f(???).length }
```

The block-parameter proposal itself has not yet settled on how to parameterize
its block parameters. The topic reference may be the key to solving this
problem, making other, special block parameters unnecessary. This example also
uses the hypothetical headless property syntax and headless pipelining syntax
from above.

<tr>
<td>

```js
server(app) {
  #.get('/') do (response) {
    request()
      |> .get('param1')
      |> `hello world ${#}`
      |> response.send
  }

  #.listen(3000) {
    log('hello')
  }
}
```

<td>

```js
server(app) {
  ???.get('/') do (response) {
    request()
      |> #.get('param1')
      |> `hello world ${#}`
      |> response.send
  }

  ???.listen(3000) {
    log('hello')
  }
}
```

<tr>
<th>

Topic pattern matching

<td>

The proposed syntax of [ECMAScript pattern matching][] would bind the topic
reference within the scope of a successful match clause’s scope. The topic value
would be the truthy result of the successful `Symbol.matches` call. This example
also uses the hypothetical headless property syntax from above.

<tr>
<td>

```js
match (x) {
  100: #
  Array:
    .length
  /(\d)(\d)(\d)/:
    #.groups |> #[0] + #[1] + #[2]
}
```

<td>

```js
match (x) {
  100: x
  Array:
    x.length
  /(\d)(\d)(\d)/ =|> m:
    m.groups |> #[0] + #[1] + #[2]
}
```

<tr>
<th>

Tacit pattern matching

<td>

[ECMAScript pattern matching][] could also have a completely tacit version, in
which the parenthesized antecedent is completely omitted in favor of tacitly
using the outer context’s topic. (This would have to somehow be distinguishable
from a call to a function named `match` with a [bare block argument][ECMAScript
block parameters].) This example also uses the hypothetical headless pipelining
syntax from above.

<tr>
<td>

```js
… |> f
  |> match {
    { x, y }:
      (x ** 2 + y ** 2)
        |> Math.sqrt
    [...]:
      #.length
    else:
      throw new Error(#)
  }
}
```

<td>

```js
… |> f
  |> match (#) {
    { x, y }:
      (x ** 2 + y ** 2)
        |> Math.sqrt
    [...]:
      #.length
    else:
      throw new Error(vector)
  }
}
```

<tr>
<th>

Topic metaprogramming references

<td>

In the event that TC39 seriously considers the topic function definitions
shown above, a **`function.topic`** metaprogramming operator, in the style of
the [`new.target`][] operator, could be useful in creating topic-aware functions.

This might be especially useful in creating APIs resembling [domain-specific
languages][DSLs] with [ECMAScript block parameters][]. This example creates
three functions that form an API resembling [Visual Basic’s `select`
statement][]. Two of these functions (`when` and `otherwise`) that are expected
to be called always within the third function (`select`)’s callback block.

An alternate solution without metaprogramming topics is not yet specified by the
current proposal for [ECMAScript block parameters][].

<tr>
<td colspan=2>

```js
class CompletionRecord { [[TODO]] }

function select (value, callback) {
  const contextTopic =
    [[TODO: create completion record]]
  return callback(topic) // TO DO
}

function otherwise (callback) { [[TODO]] }

function when (testValue, callback) {
  const contextTopic = function.topic
  return match (contextTopic) {
    [TODO]:
      |> applyWhen(#, testValue, callback)
    else:
      throw new Error('when clause was used outside select block')
  }
}

function applyWhen (contextTopic, testValue) {
  match (.value) {
    [...]: |> applyWhenArray
    else: |> applyWhenValue
  }
}

function applyWhenArray (contextTopic, testArray) {
  .some(arrayValue =>
    contextTopic |> when(arrayValue, callback))
}

function applyWhenValue (contextTopic, testArray) {
  return #[Symbol.matches](contextValue)
    ? contextTopic |> callback |> [[TODO]]
    : [[TODO: Pass to next when]]
}
```
***
```js
select ('world') {
  when ([Boolean, Number]) {
    log(#)
  }
  when (String) {
    log(`Hello ${#}`)
  }
  otherwise {
    throw new Error(`Error: ${#|> format}`)
  }
}
```

</table>

## Alternative solutions explored
There are a number of other ways of potentially accomplishing the above use
cases. However, the authors of this proposal believe that the smart pipe
operator may be the best choice. [TODO]

# Appendix: Explanation of nomenclature
The term [“**topic**” comes from linguistics][topic and comment] and have
precedent in prior programming languages’ use of “topic variables”.

The term “**head**” is preferred to “**topic expression**” because, in the
future, the [topic concept could be extended to other syntaxes such as
`for`][possible future extensions to the topic concept], not just pipelines.

In addition, “head” is preferred to “**LHS**”, because “LHS” in the ECMAScript
specification usually refers to the [LHS of assignments][ECMAScript LHS expressions],
which may be confusing. However, “topic expression” and “LHS” are still fine and
acceptable, if not imprecise, names for a pipeline’s head.

The term “**topic reference**” is preferred to the phrase “**topic variable**”
because the latter is a misnomer. The topic reference is *not* a variable
identifier. Unlike variables, it cannot be manually declared (`const #` is a
syntax error), nor can it be assigned with a value (`# = 3` is a syntax error).

“Topic reference” is also preferred to “**topic placeholder**”, to avoid
confusion with the placeholders of another TC39 proposal – [syntactic partial
application][]. These placeholders (currently denoted by nullary `?`) are of a
different nature than topic references. Instead of referring to a single value
bound earlier in the surrounding lexical context, these **parameter
placeholders** act as the parameter to a new function. When this new function is
called, those parameter placeholders will be bound to multiple argument values.

The term “**body**” is preferred instead of “**RHS**” because “topic” is
preferred to “LHS”. However, “RHS” is still a fine and acceptable name for the
body of the pipeline operator.

“**Bare style**” can also be called “**tacit style**”, but the former is
preferred to the latter. Eventually, certain [possible future extensions to the
topic concept][] may enable [tacit programming][] even without using bare-style
pipelines.

<!--
# Appendix: Term rewriting
## Term rewriting topic style
Pipe bodies in topic style can be rewritten into a nested `do` expression.
There are two ways to illustrate this equivalency. The first way is to [replace
each pipe expression’s topic references with an autogenerated variable][term
rewriting with autogenerated variables], which must be guaranteed to be
[lexically hygienic][] and to not conflict with other variables. The alternative
way is to [use two variables – the topic reference `#` and a single dummy
variable][term rewriting with single dummy variable] – which also preserves
[lexical hygiene][lexically hygienic].

### Term rewriting with autogenerated variables
The first way to illustrate the operator’s semantics is to replace each pipe
expression’s topic references with an autogenerated variable, which must be
guaranteed to not conflict with other variables.

Let us pretend that each pipe expression autogenerates a new, [lexically
hygienic][] variable (`#₀`, `#₁`, `#₂`, `#₃`, …), which in turn replaces each
topic reference `#` in each pipeline body. (These `#ₙ` variables are not true
syntax; it is merely for illustrative purposes. You cannot actually assign or
use `#ₙ` variables.) Let us also group the expressions with left associativity
(although this is arbitrary, because [right associativity would also
work][arbitrary associativity]).

With this notation, each line in this example would be equivalent to the other lines.

```js
1 |> # + 2 |> # * 3

// Static term rewriting
(1 |> # + 2) |> # * 3
do { const #₀ = 1; # + 2 } |> # * 3
do { const #₁ = do { const #₀ = 1; # + 2 }; #₁ * 3 }

// Runtime evaluation
do { const #₀ = do { 1 + 2 }; #₀ * 3 }
do { const #₀ = 3; #₀ * 3 }
do { do { 3 * 3 } }
9
```

Consider also the motivating first example above:

```js
promise
  |> await #
  |> # ??: throw new TypeError()
  |> doubleSay // a bare unary function call
  |> capitalize // also a bare unary function call
  |> # + '!'
```

Under left associativity, this would be statically equivalent to the following:

```js
do {
  const #₃ = do {
    const #₂ = do {
      const #₁ = do {
        const #₀ = await promise;
        #₀ ??: throw new TypeError()
      };
      doubleSay(#₁)
    };
    capitalize(#₂)
  };
  #₃ + '!'
}
```

In general, for each pipe expression `topic |> body`, assuming that `body` is in
topic style, that is, assuming that `body` contains an unshadowed topic
reference:

* Let _#<sub>n</sub>_ be a [hygienically autogenerated][lexically hygienic] topic
  reference, _#<sub>n</sub>_, where <var>n</var> is a number that would not conflict with
  the name of any other autogenerated topic reference in the scope of the
  entire pipe expression.
* Also let _substituted Body_ be `body` but with all instances of `#` replaced
  with _#<sub>n</sub>_.
* Then the static term rewriting (left associative and inside to outside) would
  simply be: `do { const ` _#<sub>n</sub>_ `= topic; ` _substituted Body_ `}`.
  This `do` expression would act as at the topic scope.

### Term rewriting with single dummy variable
The other way to demonstrate topic style is to use two variables: the topic
reference `#` and single [lexically hygienic][] dummy variable `•`. It should be
noted that `const # = …` is not a valid statement under this proposal’s actual
syntax; likewise, `•` is not a part of the proposal’s syntax. Both forms are for
illustrative purposes here only.

With this notation, no variable autogeneration is required; instead, the nested
`do` expressions will redeclare the same variables `#` and `•`, shadowing the
external variables of the same name as needed. The number example above becomes
the following. Each line is still equivalent to the other lines.
```js
1 |> # + 2 |> # * 3

// Static term rewriting
(1 |> # + 2) |> # * 3
do { const • = 1; do { const # = •; # + 2 } } |> # * 3
do { const • = (do { const • = 1; do { const # = •; # + 2 } }); do { const # = •; # * 3 } }

// Runtime evaluation
do { const • = do { do { const # = 1; # + 2 } }; do { const # = •; # * 3 } }
do { const • = do { do { const 1 + 2 } }; do { const # = •; # * 3 } }
do { const • = 3; do { const # = •; # * 3 } }
do { do { const # = 3; # * 3 } }
do { do { 3 * 3 } }
9
```

Consider also the motivating first example above:
```js
promise
  |> await #
  |> # ??: throw new TypeError()
  |> doubleSay // a bare unary function call
  |> capitalize // also a bare unary function call
  |> # + '!'
```

Under left associativity, this would be statically equivalent to the following:
```js
do {
  const • = do {
    const • = do {
      const • = do {
        const • = await promise;
        do { const # = •; # ??: throw new TypeError() }
      };
      do { const # = •; doubleSay(#) }
    };
    do { const # = •; capitalize(#) }
  };
  do { const # = •; # + '!' }
}
```

For each pipe expression, evaluated left associatively and inside to outside,
the steps of the computation would be:

1. The head is first evaluated in the current lexical context.
2. The topic’s result is bound to a hidden special variable `•`.
3. In a new inner lexical context (the topic scope), the value of `•` is
  bound to the topic reference `#`.
4. The pipe’s body is evaluated within this inner lexical context.
5. The pipe’s result is the result of the body.

## Term rewriting • Arbitrary associativity
The pipeline operator is presented above as a left-associative operator. However, it
is theoretically [arbitrarily associative][associative property]: how a
pipeline’s expressions are particularly grouped is functionally arbitrary. One
could force right associativity by parenthesizing a pipeline, such that it
itself becomes the body of another, outer pipeline.

Consider the above example `1 |> # + 2 |> # * 3`, whose terms were statically
rewritten using left associativity and autogenerated, [lexically hygienic][]
variables.
```js
// With left associativity and autogenerated hygienic variables.
1 |> # + 2 |> # * 3

// Static term rewriting
(1 |> # + 2) |> # * 3
do { const #₀ = 1; # + 2 } |> # * 3
do { const #₁ = do { const #₀ = 1; # + 2 }; #₁ * 3 }

// Runtime evaluation
do { const #₀ = do { 1 + 2 }; #₀ * 3 }
do { const #₀ = 3; #₀ * 3 }
do { do { 3 * 3 } }
9
```

But if right associativity is forced with `1 |> (# + 2 |> # * 3)`, then the
result would be the same: `9`:
```js
// With right associativity and autogenerated hygienic variables.
1 |> # + 2 |> # * 3

// Static term rewriting
1 |> (# + 2 |> # * 3)
1 |> do { const #₀ = # + 2; #₀ * 3 }
do { const #₁ = 1; do { const #₀ = #₁ + 2; #₀ * 3 } }

// Runtime evaluation
do { do { const #₀ = 1 + 2; #₀ * 3 } }
do { do { const #₀ = 3; #₀ * 3 } }
do { do { 3 * 3 } }
9
```

Similarly, `1 |> # + 2 |> # * 3` was also statically term rewritten using a
different method: under left associativity and a single dummy variable.
```js
// With left associativity and single dummy variable.
1 |> # + 2 |> # * 3

// Static term rewriting
(1 |> # + 2) |> # * 3
do { const • = 1; do { const # = •; # + 2 } } |> # * 3
do { const • = (do { const • = 1; do { const # = •; # + 2 } }); do { const # = •; # * 3 } }

// Runtime evaluation
do { const • = do { do { const # = 1; # + 2 } }; do { const # = •; # * 3 } }
do { const • = do { do { const 1 + 2 } }; do { const # = •; # * 3 } }
do { const • = 3; do { const # = •; # * 3 } }
do { do { const # = 3; # * 3 } }
do { do { 3 * 3 } }
9
```

If right associativity is forced with `1 |> (# + 2 |> # * 3)`, then the result
would be the same: `9`:
```js
// With right associativity and single dummy variable.
1 |> # + 2 |> # * 3

// Static term rewriting
1 |> (# + 2 |> # * 3)
1 |> do { const • = # + 2; do { const # = •; # * 3 } }
do { • = 1; do { const # = •; do { const • = # + 2; do { const # = •; # * 3 } } } }

// Runtime evaluation
do { do { const # = 1; do { const • = # + 2; do { const # = •; # * 3 } } }
do { do { do { const • = 1 + 2; do { const # = •; # * 3 } } }
do { do { do { const • = 3; do { const # = •; # * 3 } } }
do { do { do { do { const # = 3; # * 3 } } }
do { do { do { do { 3 * 3 } } }
9
```
-->

[“data-to-ink” visual ratio]: https://www.darkhorseanalytics.com/blog/data-looks-better-naked
[“don’t break my code”]: #dont-break-my-code
[“don’t make me overthink”]: #dont-make-me-overthink
[“don’t shoot me in the foot”]: #dont-shoot-me-in-the-foot
[“make my code easier to read”]: #make-my-code-easier-to-read
[`for` iteration statements]: https://tc39.github.io/ecma262/#sec-iteration-statements
[`in` relational operator]: https://tc39.github.io/ecma262/#sec-relational-operators
[`new.target`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/new.target
[Abstract: Get Topic Environment]: #abstract-get-topic-environment
[annevk]: https://github.com/annevk
[antecedent]: https://en.wikipedia.org/wiki/Antecedent_(grammar)
[arbitrary associativity]: #arbitrary-associativity
[associative property]: https://en.wikipedia.org/wiki/Associative_property
[background]: #background
[backward compatibility]: #backward-compatibility
[bare style: Grammar]: #bare-style-syntactic-grammar
[binding]: https://en.wikipedia.org/wiki/Binding_(linguistics)
[Clojure pipe]: https://clojuredocs.org/clojure.core/as-%3E
[completion records]: https://timothygu.me/es-howto/#completion-records-and-shorthands
[concatenative programming]: https://en.wikipedia.org/wiki/Concatenative_programming_language
[conceptual generality]: #conceptual-generality
[Contains]: #static-contains
[cyclomatic complexity]: https://en.wikipedia.org/wiki/Cyclomatic_complexity#Applications
[cyclomatic simplicity]: #cyclomatic-simplicity
[littledan]: https://github.com/littledan
[dataflow programming]: https://en.wikipedia.org/wiki/Dataflow_programming
[distinguishable punctuators]: #distinguishable-punctuators
[DSLs]: https://en.wikipedia.org/wiki/Domain-specific_language
[early error]: #static-early-errors
[early errors]: #static-early-errors
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
[ECMAScript optional catch binding]: https://github.com/tc39/proposal-optional-catch-binding
[ECMAScript pattern matching]: https://github.com/tc39/proposal-pattern-matching
[ECMAScript Primary Expressions]: https://tc39.github.io/ecma262/#prod-PrimaryExpression
[ECMAScript Property Accessors, § RS: Evaluation]: https://tc39.github.io/ecma262/#sec-property-accessors-runtime-semantics-evaluation
[ECMAScript Punctuators]: https://tc39.github.io/ecma262/#sec-punctuators
[ECMAScript static semantic rules]: https://tc39.github.io/ecma262/#sec-static-semantic-rules
[Elixir pipe]: https://elixir-lang.org/getting-started/enumerables-and-streams.html
[Elm pipe]: http://elm-lang.org/docs/syntax#infix-operators
[essential complexity]: https://en.wikipedia.org/wiki/Essential_complexity
[examples]: #examples
[expressions and operators (MDN)]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Expressions_and_Operators
[expressive versatility]: #expressive-versatility
[F# pipe]: https://docs.microsoft.com/en-us/dotnet/fsharp/language-reference/functions/index#function-composition-and-pipelining
[WHATWG Fetch Standard]: https://fetch.spec.whatwg.org/
[first pipe-operator proposal]: https://github.com/tc39/proposal-pipeline-operator/blob/37119110d40226476f7af302a778bc981f606cee/README.md
[footguns]: https://en.wiktionary.org/wiki/footgun
[formal grammar]: #grammar
[forward compatibility]: #forward-compatibility
[functional programming]: https://en.wikipedia.org/wiki/Functional_programming
[garden-path syntax]: https://en.wikipedia.org/wiki/Garden_path_sentence
[GitHub issue tracker]: https://github.com/tc39/proposal-pipeline-operator/issues
[goals]: #goals
[grammar parameters]: #grammar-parameters
[Hack pipe]: https://docs.hhvm.com/hack/operators/pipe-operator
[Huffman coding]: https://en.wikipedia.org/wiki/Huffman_coding
[human writability]: #human-writability
[IIFEs]: https://en.wikipedia.org/wiki/Immediately-invoked_function_expression
[incidental complexity]: https://en.wikipedia.org/wiki/Incidental_complexity
[inner blocks]: #inner-blocks
[jashkenas]: https://github.com/jashkenas
[Julia pipe]: https://docs.julialang.org/en/stable/stdlib/base/#Base.:|>
[lexical grammar]: #lexical-grammar
[lexically hygienic]: https://en.wikipedia.org/wiki/Hygienic_macro
[littledan invitation]: https://github.com/tc39/proposal-pipeline-operator/issues/89#issuecomment-363853394
[LiveScript pipe]: http://livescript.net/#operators-piping
[MDN operator precedence]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Operator_Precedence
[mindeavor]: https://github.com/gilbert
[motivation]: #motivation
[Node-stream piping]: https://nodejs.org/api/stream.html#stream_readable_pipe_destination_options
[nomenclature]: #nomenclature
[novice learnability]: #novice-learnability
[object initializers’ Computed Property Contains rule]: https://tc39.github.io/ecma262/#sec-object-initializer-static-semantics-computedpropertycontains
[OCaml pipe]: http://blog.shaynefletcher.org/2013/12/pipelining-with-operator-in-ocaml.html
[operator precedence]: #operator-precedence
[optional-chaining syntax proposal]: https://github.com/tc39/proposal-optional-chaining
[other ECMAScript proposals]: #other-ecmascript-proposals
[other goals]: #other-goals
[PEP 20]: https://www.python.org/dev/peps/pep-0020/
[Perl 6 pipe]: https://docs.perl6.org/language/operators#infix_==&gt;
[Perl 6 topicization]: https://www.perl.com/pub/2002/10/30/topic.html/
[pipeline syntax]: #pipeline-syntax
[possible future extensions to the topic concept]: #possible-future-extensions-to-topic-concept
[previous pipeline-placeholder discussions]: https://github.com/tc39/proposal-pipeline-operator/issues?q=placeholder
[private class fields]: https://github.com/tc39/proposal-class-fields/
[Pipeline Proposal 4]: https://github.com/tc39/proposal-pipeline-operator/wiki#proposal-4-smart-mix
[R pipe]: https://cran.r-project.org/web/packages/magrittr/index.html
[relations to other work]: #relations-to-other-work
[REPLs]: https://en.wikipedia.org/wiki/Read–eval–print_loop
[resolving topics]: #resolve-topic
[reverse Polish notation]: https://en.wikipedia.org/wiki/Reverse_Polish_notation
[Ron Buckton]: https://github.com/rbuckton
[runtime semantics]: #runtime-semantics
[simple scoping]: #simple-scoping
[sindresorhus]: https://github.com/sindresorhus
[smart body syntax]: #smart-body-syntax
[smart pipelines]: #smart-pipelines
[static analyzability]: #static-analyzability
[syntactic functional composition]: https://github.com/TheNavigateur/proposal-pipeline-operator-for-function-composition
[syntactic locality]: #syntactic-locality
[syntactic partial application]: https://github.com/tc39/proposal-partial-application
[tacit programming]: https://en.wikipedia.org/wiki/Tacit_programming
[TC39 process]: https://tc39.github.io/process-document/
[Tennent correspondence principle]: http://gafter.blogspot.com/2006/08/tennents-correspondence-principle-and.html
[term rewriting: arbitrary associativity]: #term-rewriting-arbitrary-associativity
[term rewriting topic style]: #term-rewriting-topic-style
[term rewriting with autogenerated variables]: #term-rewriting-with-single-dummy-variable
[term rewriting with single dummy variable]: #term-rewriting-with-single-dummy-variable
[term rewriting]: https://en.wikipedia.org/wiki/Term_rewriting
[terse function calls]: #terse-function-calls
[terse parentheses]: #terse-parentheses
[terse variables]: #terse-variables
[topic and comment]: https://en.wikipedia.org/wiki/Topic_and_comment
[topic variables in other languages]: https://rosettacode.org/wiki/Topic_variable
[topic-token bikeshedding]: https://github.com/tc39/proposal-pipeline-operator/issues/91
[Underscore.js]: http://underscorejs.org
[Unix pipe]: https://en.wikipedia.org/wiki/Pipeline_(Unix
[untangled flow]: #untangled-flow
[WHATWG-stream piping]: https://streams.spec.whatwg.org/#pipe-chains
[zero runtime cost]: #zero-runtime-cost
[formal pipeline specification]: https://jschoi.org/18/es-smart-pipelines/spec
[Core Proposal]: #core-proposal
[Additional Feature PF]: #additional-feature-pf
[Additional Feature MT]: #additional-feature-mt
[Additional Feature TE]: #additional-feature-te
[Additional Feature PP]: #additional-feature-pp
[Additional Feature TC]: #additional-feature-tc
[Pipeline Proposal 1]: https://github.com/tc39/proposal-pipeline-operator/wiki#proposal-1-f-sharp-only
[Node.js `util.promisify`]: https://nodejs.org/api/util.html#util_util_promisify_original
[don’t break my code]: #dont-break-my-code
[make my code easier to read]: #make-my-code-easier-to-read
[lexical topic]: https://jschoi.org/18/es-smart-pipelines/spec#sec-lexical-topics
[pipeline head]: https://jschoi.org/18/es-smart-pipelines/spec#prod-PipelineHead
[pipeline body]: https://jschoi.org/18/es-smart-pipelines/spec#prod-PipelineBody
[terse function application]: #terse-function-application
[`do` expressions]: #do-expressions
[jQuery]: https://jquery.com/
[JS Foundation]: https://js.foundation/
[jquery/src/core/parseHTML.js]: https://github.com/jquery/jquery/blob/2.2-stable/src/core/parseHTML.js
[jquery/src/core/access.js]: https://github.com/jquery/jquery/blob/2.2-stable/src/core/access.js
[jquery/src/core/init.js]: https://github.com/jquery/jquery/blob/2.2-stable/src/core/init.js
[Lodash]: https://lodash.com/
[Ramda]: http://ramdajs.com/
[Ramda wiki cookbook]: https://github.com/ramda/ramda/wiki/Cookbook
[pure functions]: https://en.wikipedia.org/wiki/Pure_function
[immutable objects]: https://en.wikipedia.org/wiki/Immutable_object
[currying]: https://en.wikipedia.org/wiki/Currying
[WHATWG Streams Standard]: https://stream.spec.whatwg.org/
[MDN operator precedence]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Operator_Precedence#Table
[tc39/proposal-decorators#30]: tc39/proposal-decorators#30
[tc39/proposal-decorators#42]: tc39/proposal-decorators#42
[tc39/proposal-decorators#60]: tc39/proposal-decorators#60
[Visual Basic’s `select` statement]: https://docs.microsoft.com/en-us/dotnet/visual-basic/language-reference/statements/select-case-statement
[WHATWG Fetch + Core Proposal]: #whatwg-fetch-standard-core-proposal
[jQuery + Core Proposal]: #jquery-core-proposal
[Underscore.js + Core Proposal]: #underscorejs--proposal
[Underscore.js + CP + UP]: #underscorejs-core-proposal-additional-feature-up
[identity function]: https://en.wikipedia.org/wiki/Identity_function
[Clojure compact function]: https://clojure.org/reference/reader#_dispatch
