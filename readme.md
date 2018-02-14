# Smart pipelines
ECMAScript Stage-(−1) Proposal by J. S. Choi, 2018-02.

This repository contains the formal specification for a proposed “smart pipe
operator” `|>` in JavaScript. It is currently not even in Stage 0 of the [TC39
process][TC39 process] but it may eventually be presented to TC39.

## Background
<details>
<summary>The concept of a pipe operator appears in numerous other languages,
variously called “pipeline”, “threading”, and “feed” operators.</summary>

* [Clojure’s `->` and `as->`][Clojure pipe]
* [Forth’s, Joy’s, Factor’s, Onyx’s, PostScript’s, and RPL’s term
  concatenation][concatenative programming]
* [Elixir/Erlang’s `|>`][Elixir pipe]
* [Elm’s `|>`][Elm pipe]
* [F#’s `|>`][F# pipe]
* [Hack’s `|>` and `$$`][Hack pipe]
* [Julia’s `|>`][Julia pipe]
* [LiveScript’s `|>`][LiveScript pipe]
* [OCaml’s `|>`][OCaml pipe]
* [Perl 6’s `==>`][Perl 6 pipe]
* [R / magrittr’s `%>%`][R pipe]
* [Unix shells’ and PowerShell’s `|` ][Unix pipe]

It is also conceptually similar to [WHATWG-stream piping][] and [Node-stream piping][].

</details>

The binary operator proposed here would provide a backwards- and
forwards-compatible style of chaining nested expressions into a readable,
left-to-right manner. Nested transformations become untangled into short steps
in a zero-cost abstraction.

The proposal is a variant of a [previous pipe-operator proposal][] championed by
[Daniel “littledan” Ehrenberg of Igalia][]. This variant is listed as
[Proposal 4: Smart Mix on the pipe-proposal wiki][]. The variant resulted from
[previous discussions about pipeline placeholders in the previous pipe-operator
proposal][previous pipeline-placeholder discussions], which culminated in an
[invitation by Ehrenberg to try writing a specification draft][littledan
invitation]. A prototype Babel plugin is also brewing.

You can take part in the discussions on the [GitHub issue tracker][]. When you
file an issue, please note in it that you are talking specifically about
“Proposal 4: Smart Mix”.

This specification uses `#` as its topic token, but note that this is not set in
stone. In particular, `@` could also be used. would be similarly terse, visually
distinguishable, and easily typeable. [Bikeshedding over what characters to use
for the topic token is occurring in a GitHub
issue](https://github.com/tc39/proposal-pipeline-operator/issues/91).

## Motivation

Nested, deeply composed expressions occur often in JavaScript. They occur
whenever any single value must be processed by a series of transformations,
whether they be operations, functions, or constructors. Unfortunately, this
nested expression – and many like it – can be quite messy spaghetti code, due to its
mixing of prefix, infix, and postfix expressions together. Writing the code
requires many nested levels of indentation. Reading the code requires checking
both the left and right of each subexpression to understand its data flow.

```js
new User.Message(
  capitalize(
    doubledSay(
      (await stringPromise)
        ?? throw new TypeError(`Expected string from ${stringPromise}`)
    )
  ) + '!'
)
```

With the smart pipe operator, the code above could be terser and, literally,
straightforward. Prefix, infix, and postfix expressions would be less tangled
together in threads of spaghetti. Instead, data values would be piped from left
to right through a single flat thread of postfix expressions, essentially
forming a [Reverse Polish notation][]. The syntax statically is [term rewritable
into already valid code][term rewriting] with theoretically zero runtime cost.

```js
stringPromise
  |> await #
  |> # ?? throw new TypeError()
  |> doubleSay(#, ', ')
  |> capitalize // a bare unary function call
  |> # + '!'
  |> new User.Message // a bare unary constructor call
```

This code’s terseness and flatness may be both easier for the JavaScript
developer to read and to edit. The reader may follow the flow of data more
easily through this single flattened thread of postfix operations. And the
editor may more easily add or remove operations at the beginning, end, or middle
of the thread, without changing the indentation of many nearby, unrelated lines.

Similar use cases appear numerous times in JavaScript code, whenever any value
is transformed by expressions of any type: function calls, property calls,
method calls, object constructions, arithmetic operations, logical operations,
bitwise operations, `typeof`, `instanceof`, `await`, `yield` and `yield *`, and
`throw` expressions. In particular, the styles of [functional programming][],
[dataflow programming][], and [tacit programming][] may benefit from pipelining.
The smart pipe operator can simply handle them all.

Note also that it was not necessary to include parentheses for `capitalize` or
`new User.Message`; they were implicitly included as a unary function call and a
unary constructor call, respectively. That is, the preceding example is
equivalent to:
```js
'hello'
  |> await #
  |> # ?? throw new TypeError(`Expected string from ${#}`)
  |> doubleSay(#, ', ')
  |> capitalize(#)
  |> # + '!'
```

Being able to automatically detect this [bare style][] is the [**smart** part
of this “smart pipe operator”][smart body syntax].

<details>
<summary>

### Real-world examples

</summary>

It is useful to look at code several real-world libraries or standards, and
compare their readability with pipelined versions.

<table>
<thead>
<tr>
<th>Source
<th>Status quo
<th>With smart pipes
<th>Notes

<tbody>
<tr>
<td>

**[Prior pipeline proposal][]**.\
[Gilbert “mindeavor”][mindeavor] &c.\
ECMA International.\
2017–2018.\
BSD License.

<td>

```js
function doubleSay (str, separatorStr) {
  return `${str}${separatorStr}${str}`
}

function capitalize (str) {
  return str[0].toUpperCase()
    + str.substring(1)
}

new User.Message(
  capitalizedString(
    doubledSay(
      (await stringPromise)
        ?? throw new TypeError()
    )
  ) + '!'
)
```

<td>

```js
function doubleSay (str, separatorStr) {
  return `${str}${separatorStr}${string}`
}

function capitalize (str) {
  return str[0].toUpperCase()
    + str.substring(1)
}

stringPromise
  |> await #
  |> # ?? throw new TypeError()
  |> doubleSay(#, ', ')
  |> capitalize |> # + '!'
  |> new User.Message
```

<td>

Note that `|> capitalize` is a bare unary function call. The `#` is tacitly,
invisibly implied. `|> capitalize(#)` would work but the `#` is unnecessary.\
Ditto for `|> new User.Message`, which is a bare unary constructor call,
abbreviated from `|> new User.Message(#)`.

<tr>
<td>″″
<td>″″
<td>

```js
stringPromise
  |> await #
  |> # ?? throw new TypeError()
  |> `${#}, ${#}`
  |> #[0].toUpperCase() + #.substring(1)
  |> # + '!'
  |> new User.Message
```

<td>When tiny functions are only used once, and their bodies would be
self-documenting, they are ritual boilerplate that a developer’s style may
prefer to inline.

<tr>
<td>

**[Underscore.js][]** 1.8.3.\
[Jeremy Ashkenas][jashkenas] &c.\
2009–2018.\
MIT License.

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

<td>

```js
function (obj, pred, context) {
  return obj
    |> isArrayLike(#) ? _.findIndex : _.findKey
    |> #(obj, pred, context)
    |> (# !== void 0 && # !== -1)
      ? obj[#] : undefined;
}
```

<td>

<tr>
<td>″″
<td>

```js
function (obj, pred, context) {
  return _.filter(obj,
    _.negate(cb(pred)),
    context
  )
}
```

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

<tr>
<td>″″
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

<td>

```js
function (
  srcFn, boundFn, ctxt, callingCtxt, args
) {
  if (callingCtxt |> # instanceof boundFn |> !#)
    return srcFn.apply(ctxt, args);
  var self = srcFn
    |> #.prototype |> baseCreate;
  return self
    |> srcFn.apply(#, args)
    |> _.isObject(#) ? # : self;
}
```

<td>

<tr>
<td>″″
<td>

```js
function (obj) {
  if (obj == null) return 0;
  return isArrayLike(obj)
    ? obj.length
    : _.keys(obj).length;
}
```

<td>

```js
function (obj) {
  if (obj == null) return 0;
  return obj |> isArrayLike
    ? obj.length
    : obj |> _.keys |> #.length;
}
```

<td>

<tr>
<td>

**[Pify][]** 3.0.0.\
[Sindre Sorhus][sindresorhus] &c.\
2015–2018.\
MIT License.

<td>

```js
pify(fs.readFile)('package.json', 'utf8')
  .then(data => {
    console.log(JSON.parse(data).name)
  })
```

<td>

```js
'package.json'
  |> await pify(fs.readFile)(#, 'utf8')
  |> JSON.parse |> #.name
  |> console.log
```

<td>

<tr>
<td>

**[Fetch Standard][]**.\
[Anne van Kesteren][annevk] &c.\
2011–2018.\
WHATWG.\
Creative Commons BY.

<td>

```js
fetch('/music/pk/altes-kamuffel')
  .then(res => res.blob())
  .then(playBlob)
```

<td>

```js
'/music/pk/altes-kamuffel'
  |> await fetch(#)
  |> await #.blob()
  |> playBlob
```

<td>

<tr>
<td>″″
<td>

```js
fetch('https://example.com/',
  { method: 'HEAD' }
).then(res =>
  log(res.headers.get('content-type'))
)
```

<td>

```js
'https://example.com/'
  |> await fetch(#, { method: 'HEAD' })
  |> #.headers.get('content-type')
  |> log
```

<td>

<tr>
<td>″″
<td>

```js
fetch('https://pk.example/berlin-calling',
  { mode: 'cors' }
).then(response => {
  if (response.headers.get('content-type')
    ??.toLowerCase()
    .indexOf('application/json') >= 0
  ) {
    return response.json()
  } else {
    throw new TypeError()
  }
}).then(processJSON)
```

<td>

```js
const response = 'https://pk.example/berlin-calling'
  |> await fetch(#, { mode: 'cors' });
response
  |> #.headers.get('content-type')
  |> #??.toLowerCase()
  |> #.indexOf('application/json')
  |> # >= 0
  |> # ? response : throw new TypeError()
  |> await #.json()
  |> processJSON
```

<td>

</table>

</details>

## Nomenclature
<details>
<summary>Because this proposal introduces several new concepts, it is important
to use a consistent set of terminology.</summary>

The binary operator itself `|>` may be referred to as a **pipe**, a **pipe
operator**, or a **pipeline operator**; all these names are equivalent. This
specification will prefer the term “pipe operator”.

A pipe operator between two expressions forms a **pipe expression**. One or more
pipe expressions in a chain form a **pipeline**.

For each pipe expression, the expression before the pipe is the pipeline’s
**head**. A pipeline’s head may also be called its **left-hand side (LHS)**,
because it’s left to the pipe. (The head could also be referred to as the pipe’s
**[antecedent][]** , its **topic expression**, or its **[binder][binding]**.)
The *value* to which the head evaluates may be referred to as the **topic
value**, **head value**, or **LHS value**.

The expression after a pipe is the pipeline’s **body**. A pipeline’s body may
also be called its **right-hand side (RHS)**, because it’s to the right of the
pipe. When the body is evaluated according to its [runtime semantics][], that
value may be referred to the **pipeline’s value**.

Where “pipeline” is used as a verb, the pipe operator is said **to pipeline its
topic through its body**, resulting in the pipeline’s value.

A **pipeline-level expression** is an expression at the same [ level level of
the pipe operator][operator precedence]. Although all pipelines are
pipeline-level expressions, most pipeline-level expressions are not actually
pipelines. Conditional operations, logical-or operations, or any other
expressions that have tighter [operator precedence][] than the pipe
operation – those are also pipeline-level expressions.

The special token `#` is a **topic reference**, a **topic bindee**, **topic
placeholder**, or **topic anaphor**. `#` is a nullary operator that acts like a
special variable: implicitly bound to its value, but still lexically scoped.

The topic reference could also be called a “**topic variable**” or “**topic
identifier**”, [as they are called in other programming languages][topic
variables in other languages]. But in JavaScript, these phrases would be
misnomers. The topic reference is *not* actually a variable identifier and
cannot be manually declared (`const #` is a syntax error), nor can it be
assigned with a value (`# = 3` is a syntax error). Instead, the topic reference
is implicitly, lexically bound only within pipeline bodies.

When a pipeline’s body is in this **topical style**. A pipeline body in topical
style forms an inner lexical scope – called the pipeline’s **topical
scope** – within which the topic reference is implicitly bound to the value of the
topic, acting as a **placeholder** for the topic’s value.

Alternatively, you may omit the topic references entirely, if the body is just a
**simple reference** to a function or constructor, such as with `… |>
capitalize` and `… |> new User.Message`. Such a pipeline body is in **bare
style**. [Bare style][] is described in more detail later.

The term [“**topic**” comes from linguistics][topic and comment] and have
precedent in prior programming languages’ use of “topic variables”.

The terms “**antecedent**” and “**topic expression**” are preferred to “**LHS**”
because, in the future, the [topic concept could be extended to other syntaxes
such as `for`][possible future extensions to the topic concept], not just
pipelines. In addition, “LHS” in the ES specification usually refers to the [LHS
of assignments][ES LHS expressions], which may be confusing. However, “LHS” is
still a fine and acceptable name for a pipeline’s antecedent.

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
preferred to the latter. Eventually, certain [future possible extensions to the
topic concept][] may enable [tacit programming][] even without using bare-style
pipelines.

</details>

## Grammar
This grammar of the pipeline operator juxtaposes brief rules written for the
JavaScript developer with formally written changes to the ES standard.

<details>
<summary>The ES specification explains further…</summary>

> A context-free grammar consists of a number of **productions**. Each
> production has an abstract symbol called a **nonterminal** as its left-hand
> side, and a sequence of zero or more nonterminal and **terminal** symbols as
> its right-hand side. For each grammar, the terminal symbols are drawn from a
> specified alphabet.

</details>

This proposal uses the [same notation as that from the ES standard][ES grammar
notation] to denote context-free grammars; see [ES grammar notation][] for more
information.

### Lexical grammar
The smart pipe operator adds two new tokens to JavaSCript: `|>` the binary pipe,
and `#` the topic reference.

<details>
<summary>The lexical rule for punctuator tokens would be modified so that
these two tokens are added.</summary>

The specification explains in [ES clause 5.1.2][]:

> A lexical grammar for ECMAScript is given in [ES clause 11][]. This grammar
> has as its terminal symbols Unicode code points…It defines a set of
> productions…that describe how sequences of such code points are translated
> into a sequence of input elements.
>
> Input elements other than white space and comments form the terminal symbols
> for the syntactic grammar for ECMAScript and are called ECMAScript tokens.
> These tokens are the reserved words, identifiers, literals, and
> **punctuators** of the ECMAScript language. Moreover, line terminators,
> although not considered to be tokens, also become part of the stream of input
> elements and guide the process of automatic semicolon insertion…Simple white
> space and…comments are discarded and do not appear in the stream of input
> elements for the syntactic grammar.

The _Punctuators_ production is defined in [ES punctuators][]. This production
would be changed from this:

```
Punctuator :: one of
  `{` `(` `)` `[` `]` `.` `...` `;` `,`
  `<` `>` `<=` `>=` `==` `!=` `===` `!==`
  `+` `-` `*` `%` `**` `++` `--`
  `<<` `>>` `>>>` `&` `|` `^` `!` `~` `&&` `||` `?` `:`
  `=` `+=` `-=` `*=` `%=` `**=`
  `<<=` `>>=` `>>>=` `&=` `|=` `^=` `=>`
```

…to this:

```
Punctuator :: one of
  `{` `(` `)` `[` `]` `.` `...` `;` `,` `#`
  `<` `>` `<=` `>=` `==` `!=` `===` `!==`
  `+` `-` `*` `%` `**` `++` `--`
  `<<` `>>` `>>>` `&` `|` `^` `!` `~` `&&` `||` `?` `:` `|>`
  `=` `+=` `-=` `*=` `%=` `**=`
  `<<=` `>>=` `>>>=` `&=` `|=` `^=` `=>`
```

</details>

### Grammar parameters
<details>
<summary>In the ES standard, the rules that produce expressions are often parameterized
with three flags, which are then recursively passed into their constituent
rules. These parameters thus must also be used by the new rules in this proposal.</summary>

* **`In`**: Whether the current context allows the [`in` relational operator][],
  which is false only in the headers of [`for` iteration statements][].
* **`Yield`**: Whether the current context allows a `yield`
  expression/declaration (that is, is the current function context a generator?).
* **`Await`**: Whether the current context allows an `await`
  expression/declaration (that is, is the current function context an async
  function/generator?).

</details>

### Syntax and static semantics
The syntactic grammar of JavaScript can transform token sequences (defined by the [lexical grammar][])
into **parse trees**: rooted tree data structures made of **Parse Nodes**.

<details>

This is described further in [ES Clause 5.1.4][].

When a stream of code points is to be parsed as an ECMAScript Script or Module, it is first converted to a stream of input elements by repeated application of the lexical grammar; this stream of input elements is then parsed by a single application of the syntactic grammar. The input stream is syntactically in error if the tokens in the stream of input elements cannot be parsed as a single instance of the goal nonterminal (Script or Module), with no tokens left over.

When a parse is successful, it constructs a parse tree, a rooted tree structure in which each node is a Parse Node. Each Parse Node is an instance of a symbol in the grammar; it represents a span of the source text that can be derived from that symbol. The root node of the parse tree, representing the whole of the source text, is an instance of the parse's goal symbol. When a Parse Node is an instance of a nonterminal, it is also an instance of some production that has that nonterminal as its left-hand side. Moreover, it has zero or more children, one for each symbol on the production's right-hand side: each child is a Parse Node that is an instance of the corresponding symbol.

</details>

The syntactic grammar of JavaScript further relies upon several functions that analyze its
syntactic structures. These functions are polymorphic on the types of their
input syntactic structures, and their definitions are often also recursive. The
ES specification goes into more detail on these **Static Semantic Rules** in [ES
Clause 5.2.4][].

<details>

> Context-free grammars are not sufficiently powerful to express all the rules
> that define whether a stream of input elements form a valid ECMAScript Script
> or Module that may be evaluated. In some situations additional rules are
> needed that may be expressed using either ECMAScript algorithm conventions or
> prose requirements. Such rules are always associated with a production of a
> grammar and are called the static semantics of the production.
>
> Static Semantic Rules have names and typically are defined using an algorithm.
> Named Static Semantic Rules are associated with grammar productions and a
> production that has multiple alternative definitions will typically have for
> each alternative a distinct algorithm for each applicable named static
> semantic rule.

</details>

This specification defines additions for the following Static Semantic Rules:

| Form                              | Notes                                       |
| --------------------------------- | ------------------------------------------- |
| Contains                          | Already defined in ES for nearly all nodes. |
| Is Function Definition            | Already defined in ES for many nodes.       |
| Is Valid Simple Assignment Target | Already defined in ES for many nodes.       |
| Binds Topic                       | New rule defined in this proposal.          |
| Early errors                      | Already defined in ES for many nodes.       |

It should be noted that, in the ES standard, the Contains rule is currently
written as an infix operator: “… Contains …” for historical reasons. This is
unlike any other Static Semantic Rule, which would be written as prefix
operators “{{Rule Name}} of … with arguments …”. There are plans to change all
Static Semantic Rules to instead have a consistent infix syntax resembling
method calls: “….{{rule}}(…)”. For self-consistency, this proposal will use that
planned method-like syntax.

#### Static Containment
<details>
<summary>The ES spec imply defines the Contains rule for nearly all nodes.
Conceptually, a node contains another node if the latter is somewhere in the
former.</summary>

>    b. If _child_ is an instance of a nonterminal, then
>
>       i.  Let contained be the result of _child_ Contains _symbol_.
>       ii. If contained is true, return true.
>
> 2. Return false.
>
> The above definition is explicitly over-ridden for specific productions.

</details>

In the ES standard, the Contains rule is used by many other Static Semantic
Rules, such as [object initializers’ Computed Property Contains rule][]. The
rule is also generally overridden by methods definitions and other function
definitions, such that they hide their substructure from the rule.

It should also be noted that, uniquely among the Static Semantic Rules, Contains
is written as an infix operator: “… Contains …” for historical reasons. This
proposal will instead use the planned future new syntax “….contains(…)”.

### Static Function Definition

#### Static Early Errors
<details>

> A special kind of static semantic rule is an Early Error Rule. Early error
> rules define early error conditions (see clause 16) that are associated with
> specific grammar productions. Evaluation of most early error rules are not
> explicitly invoked within the algorithms of this specification. A conforming
> implementation must, prior to the first evaluation of a Script or Module,
> validate all of the early error rules of the productions used to parse that
> Script or Module. If any of the early error rules are violated the Script or
> Module is invalid and cannot be evaluated.

</details>

### Operator precedence
As a binary operation forming compound expressions, the [operator precedence and
associativity][MDN’s guide on expressions and operators] of pipelining must be
determined, relative to other operations.

Precedence is tighter than assignment (`=`, `+=`, …), generator `yield` and
`yield *`, and sequence `,`; and it is looser than every other type of
expression. See also [MDN’s guide on expressions and operators][]. If the pipe
operation were any tighter than this level, its body would have to be
parenthesized for many frequent types of expressions. However, the result of a
pipeline is also expected to often serve as the body of a variable assignment
`=`, so it is tighter than assignment operators.

The pipe operator actually has [bidirectional associativity][]. However, for the
purposes of this grammar, it will have left associativity.

<details
<summary>A table shows how the topic reference and the pipe operator are
integrated into the hierarchy of operators.</summary>

All expression levels in JavaScript are listed here, from **tightest to
loosest**. Each level includes all the expression types listed for that
level – **as well as** any expression types from any precedence level that is
listed **above** it.

| Level          | Type                    | Form           | Associativity / fixity   |
| -------------- | ----------------------- | -------------- | ------------------------ |
| Primary        | This                    |`this`          | Nullary                  |
| ″″             | **Topic**               |**`#`**         | ″″                       |
| ″″             | Identifiers             |`a` …           | ″″                       |
| ″″             | Null                    |`null`          | ″″                       |
| ″″             | Booleans                |`true` `false`  | ″″                       |
| ″″             | Numerics                |`0` …           | ″″                       |
| ″″             | Arrays                  |`[…]`           | Circumfix                |
| ″″             | Object                  |`{…}`           | ″″                       |
| ″″             | Function                |                | ″″                       |
| ″″             | Classes                 |`class … {…}`   | ″″                       |
| ″″             | Generators              |                | ″″                       |
| ″″             | Async functions         |                | ″″                       |
| ″″             | Regular expression      |`/…/`…          | ″″                       |
| ″″             | Templates               |```…```         | ″″                       |
| ″″             | Parentheses             |`(…)`           | ″″                       |
| LHS            | Dynamic properties      |`…[…]`          | LTR infix with circumfix |
| ″″             | Static properties       |`….…`           | ″″                       |
| ″″             | Tagged templates        |``…`…```        | ″″                       |
| ″″             | Super properties        |`super.…`       | ″″                       |
| ″″             | Meta properties         |`meta.…`        | Unchainable prefix       |
| ″″             | Super call op.s         |`super(…)`      | ″″                       |
| ″″             | New                     |`new …`         | RTL prefix               |
| ″″             | Normal calls            |`…(…)`          | LTR infix with circumfix |
| Postfix unary  | Postfix increments      |`…++`           | Postfix                  |
| ″″             | Postfix decrements      |`…--`           | ″″                       |
| Prefix unary   | Prefix increments       |`++…`           | RTL prefix               |
| Prefix unary   | Prefix decrements       |`--…`           | ″″                       |
| ″″             | Deletes                 |`delete …`      | ″″                       |
| ″″             | Voids                   |`void …`        | ″″                       |
| ″″             | Unary `+`/`-`           |″″              | ″″                       |
| ″″             | Bitwise NOT `~…`        |″″              | ″″                       |
| ″″             | Logical NOT `!…`        |″″              | ″″                       |
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
| ″″             | Instance of             |`… instanceof …`| ″″                       |
| Equality       | Abstract equality       |`… == …`        | ″″                       |
| ″″             | Abstract inequality     |`… != …`        | ″″                       |
| ″″             | Strict equality         |`… === …`       | ″″                       |
| ″″             | Strict inequality       |`… !== …`       | ″″                       |
| Bitwise AND    |                         |`… & …`         | ″″                       |
| Bitwise XOR    |                         |`… ^ …`         | ″″                       |
| Bitwise OR     |                         |`… | …`         | ″″                       |
| Logical AND    |                         |`… ^^ …`        | ″″                       |
| Logical OR     |                         |`… || …`        | ″″                       |
| Conditional    |                         |`… ? … : …`     | RTL ternary infix        |
| **Pipeline**   |                         |**`… \|> …`**   | LTR infix                |
| Assignment     | Arrow functions         |`… => …`        | RTL infix                |
| ″″             | Async arrow functions   |`async … => …`  | RTL infix                |
| ″″             | Reference assignments   |`… = …`         | ″″                       |
| ″″             |                         |`… += …`        | ″″                       |
| ″″             |                         |`… -= …`        | ″″                       |
| ″″             |                         |`… *= …`        | ″″                       |
| ″″             |                         |`… %= …`        | ″″                       |
| ″″             |                         |`… **= …`       | ″″                       |
| ″″             |                         |`… <<= …`       | ″″                       |
| ″″             |                         |`… >>= …`       | ″″                       |
| ″″             |                         |`… >>>= …`      | ″″                       |
| ″″             |                         |`… &= …`        | ″″                       |
| ″″             |                         |`… |= …`        | ″″                       |
| Yield level    |                         |`yield …`       | RTL prefix               |
| ″″             |                         |`yield * …`     | ″″                       |
| Ultimate level | Comma                   |`…, …`          | LTR infix                |

</details>

The production rule for [ES assignment-level expressions][] needs to be modified
so that pipe expressions slip in between it and conditional-level expressions in
the hierarchy. Then the conditional-expression rule would be used in the
production for pipeline-level expressions (also defined soon), preserving the
unbroken recursive hierarchy of expression types.

<details>
<summary>An assignment-level expression currently may be a conditional
expression, yield expression, arrow function, async arrow function, or
assignment. These possibilities are given the same parameters that the
assignment-level expression happens to have also gotten.</summary>

```
// Old version
AssignmentExpression[In, Yield, Await] :
  ConditionalExpression[?In, ?Yield, ?Await]
  [+Yield] YieldExpression[?In, ?Await]
  ArrowFunction[?In, ?Yield, ?Await]
  AsyncArrowFunction[?In, ?Yield, ?Await]
  LeftHandSideExpression[?Yield, ?Await]
    `=` AssignmentExpression[?In, ?Yield, ?Await]
  LeftHandSideExpression[?Yield, ?Await]
    AssignmentOperator AssignmentExpression[?In, ?Yield, ?Await]
```


</details>

<details>
<summary>This conditional-expression production rule would be replaced with one for
pipeline expressions, which will be defined in the next section.</summary>

```
// New version
AssignmentExpression[In, Yield, Await] :
  PipelineExpression[?In, ?Yield, ?Await]
  [+Yield] YieldExpression[?In, ?Await]
  ArrowFunction[?In, ?Yield, ?Await]
  AsyncArrowFunction[?In, ?Yield, ?Await]
  LeftHandSideExpression[?Yield, ?Await]
    `=` AssignmentExpression[?In, ?Yield, ?Await]
  LeftHandSideExpression[?Yield, ?Await]
    AssignmentOperator AssignmentExpression[?In, ?Yield, ?Await]
```

</details>

### Pipeline expression

An expression is a pipeline-level expression (given the usual three [grammar
parameters][]) only if:

* It is either also a conditional-level expression, with the same parameters
  used by the pipeline-level expression;
* Or it is another pipeline-level expression, followed by a `|>` token, then a
  pipeline body (defined next), with the same parameters as above.

<details>
<summary>This would be defined in a new production rule.</summary>

```
// New rule
PipelineExpression[In, Yield, Await] :
  ConditionalExpression[?In, ?Yield, ?Await]
  PipelineExpression[?In, ?Yield, ?Await] `|>`
    PipelineBody[?In, ?Yield, ?Await]
```

</details>

#### Static Semantics

Several…

### Smart body syntax
Most pipelines will use the topic reference `#` in their bodies. As already
explained above in [nomenclature][], this style of pipeline is called **topical
style**.

But for two certain simple cases – unary functions and constructors – you may omit
the topic reference from the body. This is called **bare style**.

When a pipe is in bare style, we refer to the body as a **bare function** or a
**bare constructor**, depending on the rules in [bare style][]. The body acts
as just a simple reference to a function or constructor, such as with `… |>
capitalize` and `… |> new User.Message`. The body’s value would then be called
as a unary function or constructor, without having to use the topic reference as
an explicit argument.

<details>

<summary>The rules of the two respective styles will be explained in more
detail, but an overview is given in a table.</summary>

| Valid topical style     | Valid bare style                | Invalid bare style
| ----------------------- | -------------------------------- | --------------------
|`… \|> o(#)`             |`… \|> o`                         |  `… \|> o()` 🚫
| ″″                      | ″″                               | `… \|> (o)` 🚫
| ″″                      | ″″                               | `… \|> (o())` 🚫
|`… \|> new o(#)`         |`… \|> new o`                     | `… \|> new o()` 🚫
| ″″                      | ″″                               | `… \|> (new o)` 🚫
| ″″                      | ″″                               | `… \|> (new o())` 🚫
| ″″                      | ″″                               | `… \|> new (o)` 🚫
| ″″                      | ″″                               | `… \|> new (o())` 🚫
|`… \|> o.m(#)`           |`… |> o.m`                        | `… \|> o.m()` 🚫
| ″″                      |`const m = o::m; … |> m`          | `… \|> o.m()` 🚫
|`… \|> new o.m(#)`       |`… |> new o.m`                    | `… \|> o.m()` 🚫
| ″″                      |`const m = o::m; … |> m`          | `… \|> o.m()` 🚫
|`… \|> o.m(arg, #)`      |`const m = o::m(arg); … |> m`     | `… \|> o.m(arg)` 🚫
|`… \|> new o.m(arg, #)`  |`const m = new o::m(arg); … |> m` | `… \|> new o.m(arg)` 🚫
|`… \|> o[symbol](#)`     |`const m = o[symbol]; … |> m`     | `… \|> o[symbol]` 🚫
|`… \|> new o[symbol](#)` |`const m = new o[symbol]; … |> m` | `… \|> new o[symbol]` 🚫
|`… \|> o.makeFn()(#)`    |`const m = o.makeFn(); … |> m`    | `… \|> o.makeFn()` 🚫
|`… \|> new o.makeFn()(#)`|`const m = new o.makeFn(); … |> m`| `… \|> new o.makeFn()` 🚫

</details>

<details>

<summary>There are five ordered goals that the smart body syntax tries to
fulfill.</summary>

The first three goals can be summed up by saying, “Keep the parsing rules
simple!” and “Don’t make me think hard!” The last two rules address the new
functionality that this proposal would bring. From most to least important:

1. Short parser lookahead: Minimize the parsing lookahead that the compiler must
   check before it can distinguish between bare style and topical style. By
   restricting the space of valid bare-style pipeline bodies (that is, without
   topic references), the rule prevents [garden-path syntax][] that would
   otherwise be possible: such as `… |> compose(f, g, h, i, j, k, #)`.

2. Minimal parser branching: [TO DO]

3. Static analysis: Help the editing JavaScript developer to avoid common
   footguns at compile time. Preventing them from accidentally omitting a topic
   reference where they meant to put one. For instance, if `x |> 3` were not a
   syntax error, then it would be a useless operation and almost certainly not
   what the editor intended.

4. Versatile expressivity: [TO DO]

5. Tacit terseness: Make especially ergonomic the style of [**tacit
   programming**, aka **point-free programming**][tacit programming], for
   frequent cases in which topic references would be unnecessarily verbose:
   unary functions and unary constructors.

</details>

#### Bare style
The **bare style** supports using simple identifiers, possibly with chains of
simple property identifiers. If there are any operators, parentheses (including
for method calls), brackets, or anything other than identifiers and dot
punctuators, then it is in topical style, not in bare style.

If the body is a mere identifier, optionally with a chain of properties, and
   with no parentheses or brackets, then that identifier is interpreted to be a
   **bare function call**.

<details>

**If a pipeline** is of the form **`{{topic}} |> {{identifier}}`** (or
`{{topic}} |> {{identifier0}}.{{identifier1}}` or `{{topic}} |>
{{identifier0}}.{{identifier1}}.{{identifier2}}` or …), then the pipeline is a
**bare function call**. The **pipeline’s value** is **`{{body}}({{topic}})`**.

This is achieved by defining the _PipelineTacitFunction_ production using
[_IdentifierReference_][], [_IdentifierName_][], and left recursion, in
imitation of how [_MemberExpression_][] handles method chains.

```
PipelineTacitFunction :
  IdentifierReference
  PipelineTacitFunction `.` IdentifierName
```

</details>

If the body starts with `new`, followed by mere identifier, optionally with a
   chain of properties, and with no parentheses or brackets, then that identifier
   is interpreted to be a **bare constructor**.

<details>

**If a pipeline’s `body`** is of the form **`{{topic}} |> new {{identifier}}`**
(or `{{topic}} |> new {{identifier0}}.{{identifier1}}` or `{{topic}} |> new
{{identifier0}}.{{identifier1}}.{{identifier2}}` or …), then the pipeline is a
**bare constructor call**. The **pipeline’s value** is **`new
{{bodyConstructor}}({{topic}})`**, where {{bodyConstructor}} is {{body}} with
the `new` removed from its start.

```
PipelineTacitFunction :
  new IdentifierReference
  PipelineTacitFunction `.` IdentifierName
```

</details>

Therefore, a pipeline in **bare style *never*** has **parentheses `(…)` or
brackets `[…]`** in its body. Neither `… |> object.method()` nor `… |>
object.method(arg)` nor `… |> object[symbol]` nor `… |> object.createFunction()`
are in bare style (in fact, they all have invalid syntax, due to their being in
topical style without any topic references).

**When a body needs parentheses or brackets**, then **don’t use bare style**,
and instead **use a topic reference** in the body……or **assign the body to a
variable**, then **use that variable as a bare body**.

The JavaScript developer is encouraged to use topic references and avoid bare
style, where bare style may be visually confusing to the reader.

#### Topical style
**If a pipeline** of the form `{{topic}} |> {{body}}` is ***not* in bare
style** (that is, it is *not* a bare function call or bare constructor call),
then it **must be in topical style**.

<details>
<summary>The pipeline’s value is whatever the body expression evaluates into,
assuming that the topic value is first bound to the topic reference within the
body scope.</summary>

The **pipeline’s value** is **`do { const {{topicIdentifier}} = {{topic}};
{{substitutedBody}} }`**, where:

* `{{topicVariable}}` is any [identifier that is *not* already used by any
  variable in the outer lexical context or the body’s inner topical
  context][lexically hygienic],
* And `{{substitutedBody}}` is `{{body}}` but with every instance of outside of
  the topic reference replaced by `{{topicVariable}}`.

</details>

### Multiple topic references and inner functions
<details>
<summary>The topic reference may be used multiple times in a pipeline body. Each
use refers to the same value (wherever the topic reference is not overridden by
another, inner pipeline’s topical scope). Because it is bound to the result of
the topic, the topic is still only ever evaluated once.</summary>

The lines in each of the following rows are equivalent.

| Block                                          | Pipeline
| ---------------------------------------------- | ---------------------------------------
|`const $ = …; f($, $)`                          |`… |> f(#, #)`
|`const $ = …; [$, $ * 2, $ * 3]`                |`… |> [#, # * 2, # * 3]`

</details>

### Inner functions
<details>
<summary>Both the antecedent and the body of a pipeline may contain nested inner
functions.</summary>

The lines in each of the following rows are equivalent.

| Block                                          | Pipeline
| ---------------------------------------------- | ---------------------------------------
|`const $ = …; settimeout(() => $ * 5)`          |`… |> settimeout(() => # * 5)`|

</details>

### Deeply nested pipelines
Both the antecedent and the body of a pipeline may contain nested inner
functions. This is not encouraged, but it is still permitted.

<details>
<summary>Both the antecedent and the body of a pipeline may contain nested inner
functions.</summary>

The lines in each of the following rows are equivalent.

| Block                                          | Pipeline
| ---------------------------------------------- | ---------------------------------------
|`const $ = …; settimeout(() => f($) * 5)`       |`… |> settimeout(() => f(#) * 5)`
|`const $ = …; settimeout(() => f($) |> # * 5)`  |`… |> settimeout(() => f(#) |> # * 5)`
|`const $ = …; settimeout(() => $ |> f |> # * 5)`|`… |> settimeout(() => # |> f |> # * 5)`

</details>

## Possible future extensions to the topic concept
The [concept of the “topic variable” already exists in many other programming
languages][topic variables in other languages], commonly named with an
underscore `_` or `$_`. These languages often integrate their topic variables
into their function-call control-flow syntaxes, with [Perl 6 as perhaps the most
extensive, unified example][Perl 6 topicalization]. Integration of topic with
syntax enables especially pithy, terse [tacit programming][].

In addition, many JavaScript console [REPLs][], such as those of the WebKit Web
Inspector and the Node.js interactive console… [TO DO]

Several disadvantages to these prior approaches may increase the probability of
developer surprise, in which “surprise” refers to behavior difficult to predict
by the developer.

One disadvantage arises from their frequent dynamic binding rather than lexical
binding, as the former is not statically analyzable and is more stateful than
the latter. It may also cause surprising results when coupled with bare/tacit
calls: it becomes more difficult to tell whether a bare identifier `print` is
meant to be a simple variable reference or a bare function call on the topic
value.

Another disadvantage arises from the ability to clobber or overwrite the value of the
topic variable, which may affect code in surprising ways.

However, JavaScript’s topic reference `#` is different than this prior art. It
is lexically bound and statically analyzable. It is also cannot be accidentally
bound; the developer must opt into binding it by using the pipe operator. It
also cannot be accidentally used; it is a syntax error when `#` is used outside
of a pipeline body. [TO DO: Link to pertinent grammar sections.]

Should this proposal be accepted, the door becomes opened to extending the topic
concept to other syntax forms, potentially multiplying its benefits toward
reading and writing, while perhaps preserving static analyzability and… [TO DO]

[TO DO: Note on forward compatibility with these possibilities.]

[TO DO: Add, to above, a version of second example with `do` blocks showcasing
the `#|>` idiom.]

<table>
<thead>
<tr>
<th>
<th>With future proposal
<th>With only this proposal
<th>Notes

<tbody>
<tr>
<th>Topical for loop
<td>

```js
for (range(0, 50)) {
  console.log(# ** 2);
  console.log(#|> Math.sqrt);
}
```

<td>

```js
for (const i of range(0, 50)) {
  console.log(i |> # ** 2);
  console.log(i |> Math.sqrt);
}
```

<td>

A `for ` statement would bind the topic reference only when statement’s
parentheses is not of the form `(… of …)` or `(…; …; …)`. When this is so, then
it will act as if it were a `for (const # of …) { … }` loop: pulling each of the
given iterator’s items, then tacitly binding the item to the topic reference,
then running the block with that topic reference in its scope. This maintains
forward compatibility with pipelines whose bodies contain `for` statements that
in turn use the topic reference. [TO DO: Link to section on deep nesting.]

<tr>
<th>Topical for–await loop
<td>

```js
for await (lineStream) {
  yield #|>
    …
}
```

<td>

```js
for await (const line of lineStream) {
  yield line |>
    …
}
```

<td>Similar.

<tr>
<th>Topical arrow function
<td>

```js
materials.map(=> #.length)
```

<td>

```js
materials.map(m => m.length)
```

<tr>
<th>Topical match
<td>

```js
function getLength (#) {
  return match {
    …
  }
}
```

<td>

```js
function getLength (vector) {
  return match (vector) {
    …
  }
}
```

<tr>
<th>Topical function with topical match
<td>

```js
function getLength (#) {
  return match {
    …
  }
}
```

<td>

```js
function getLength (vector) {
  return match (vector) {
    …
  }
}
```

<tr>
<th>Topical arrow function with topical match
<td>

```js
const getLength = => match {
  …
}
```

<td>

```js
const getLength = vector => match (vector) {
  …
}
```

<tr>
<th>Topical error capture
<td>

```js
try {
  console.log('Outer try running...')

  try {
    console.log('Nested try running...')
    throw new Error(301, 'an error')
  } catch {
    console.log(`Nested catch caught: ${#.message}`)
    throw #
  } finally {
    console.log('Nested finally is running...')
  }

} catch {
  console.log(`Outer catch caught: ${#.message}`)

} finally {
  console.log('Outer finally running')
}
```

<td>

```js
try {
  console.log('Outer try running...')

  try {
    console.log('Nested try running...')
    throw new Error(301, 'an error')
  } catch (error) {
    console.log(`Nested catch caught: ${error.message}`)
    throw error
  } finally {
    console.log('Nested finally is running...')
  }

} catch (error) {
  console.log(`Outer catch caught: ${error.message}`)

} finally {
  console.log('Outer finally running')
}
```

<tr>
<th>Topical error capture with topical match
<td>

```js
try {
  …
} catch {
  match {
    MyError: …
    TypeError: …
  }
}
```

<td>

```js
try {
  …
} catch (error) {
  match (error) {
    MyError: …
    TypeError: …
  }
}
```

</table>

## Alternative solutions explored
There are a number of other ways of potentially accomplishing the above use
cases. However, the authors of this proposal believe that the smart pipe
operator may be the best choice. [TO DO]

## Relation to existing work
[TO DO]

[TO DO: Include commentary on why “topic reference” instead of “placeholder” –
because verbally confusing with partial-application placeholders – and because
forward compatibility with [possible future extensions to the topic concept].]

<details>

<summary>

## Previous draft appendices

</summary>

[TO DO: Rewrite everything in this section]

### Smart body syntax
When the parser checks the body/RHS, it must determine what style it is in.
There are four outcomes for each pipeline body: bare constructor, bare
function, topical body, or invalid syntax.

```
// New rule
PipelineBody[In, Yield, Await] :
  PipelineBareFunction
  PipelineBareConstructor
  PipelineTopicalBody[In, Yield, Await]
```

1. If the body is a mere identifier, optionally with a chain of properties, and
   with no parentheses or brackets, then that identifier is interpreted to be a
   **bare function**.
   ```
   PipelineBareFunction :
     SimpleMemberExpression
   ```

2. If the body starts with `new`, followed by mere identifier, optionally with a
   chain of properties, and with no parentheses or brackets, then that identifier
   is interpreted to be a **bare constructor**.
   ```
   PipelineBareConstructor :
     `new` SimpleMemberExpression
   ```

3. Otherwise, the body is now an arbitrary expression. Because the expression is
   not in bare style, it must use that pipe’s topic reference. ([TO DO: Formalize
   this static semantic as `usesTopic()`.] topic references from the topical scopes
   of other, inner pipelines do not count.) If there is no such topic reference in
   the expression, then a syntax error is thrown.

   `PipelineTopicalBody` will be defined in the next section.

If a pipeline body *never* uses a topic reference, then it must be a permitted
bare unary function (single identifier or simple property chain). Otherwise, it
is a syntax error. In particular, bare style *never* uses parentheses. If they
need to have parentheses, then they need to have use the topic reference. See
also [property-chained bare style][].

### General semantics
[TO DO: Rewrite this section to the conventions of the ES specification.]

A pipe expression’s semantics are:
1. The antecedent is evaluated into the topic’s value; call this `topicValue`.
2. [The body is tested for its type][smart body syntax]: Is it in bare style
   (as a bare function or a bare constructor), is it in topical style, or is it
   an invalid body?

  * If the body is a bare function (such as `f` and `M.f`):
    1. The body is evaluated (in the current lexical context); call this value
       the `bodyFunction`.
    2. The entire pipeline’s value is `bodyFunction(topicValue)`.

  * If the body is a bare constructor (such as `new C` and `new M.C`):
    1. The portion of the body after `new` is evaluated (in the current lexical
       context); call this value the `BodyConstructor`.
    2. The entire pipeline’s value is `new BodyConstructor(topicValue)`.

  * If the body is in topical style (such as `f(#, n)`, `o[s][#]`, `# + n`, and `#.p`):
    1. An inner lexical context is entered.
    2. Within this inner context, `topicValue` is bound to a variable.
    3. Within this inner context, with the variable binding to `topicValue`,
       the body is evaluated; call the resulting value `bodyValue`.
    4. The entire pipeline’s value is `bodyValue`.

  * Otherwise, if the body is an invalid body (such as `f()`, `f(n)`, `o[s]`,
    `+ n`, and `n`), then throw a syntax error explaining that the pipeline’s
    topic reference is missing from its body.

### Term rewriting topical style
Pipe bodies in topical style can be further explained with a nested `do`
expression. There are two ways to illustrate this equivalency. The first way is
to [replace each pipe expression’s topic references with an autogenerated
variable][term rewriting with autogenerated variables], which must be guaranteed
to be [lexically hygienic][] and to not conflict with other variables. The
alternative way is to [use two variables – the topic reference `#` and a single
dummy variable][term rewriting with single dummy variable] – which also preserves
[lexical hygiene][lexically hygienic].

#### Term rewriting with autogenerated variables
The first way to illustrate the operator’s semantics is to replace each pipe
expression’s topic references with an autogenerated variable, which must be
guaranteed to not conflict with other variables.

Let us pretend that each pipe expression autogenerates a new, [lexically
hygienic][] variable (`#₀`, `#₁`, `#₂`, `#₃`, …), which in turn replaces each
topic reference `#` in each pipeline body. (These `#ₙ` variables are not true
syntax; it is merely for illustrative purposes. You cannot actually assign or
use `#ₙ` variables.) Let us also group the expressions with left associativity
(although this is arbitrary, because [right associativity would also
work][bidirectional associativity]).

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
stringPromise
  |> await #
  |> # ?? throw new TypeError()
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
        const #₀ = await stringPromise;
        #₀ ?? throw new TypeError()
      };
      doubleSay(#₁)
    };
    capitalize(#₂)
  };
  #₃ + '!'
}
```

In general, for each pipe expression `topic |> body`, assuming that `body` is in
topical style, that is, assuming that `body` contains an unshadowed topic
reference:

* Let `#ₙ` be a [hygienically autogenerated][lexically hygienic] topic
  reference, `#ₙ`, where <var>n</var> is a number that would not conflict with
  the name of any other autogenerated topic reference in the scope of the
  entire pipe expression.
* Also let `substitutedBody` be `body` but with all instances of `#` replaced
  with `#ₙ`.
* Then the static term rewriting (left associative and inside to outside) would
  simply be: `do { const #ₙ = topic; substitutedBody }`. This `do` expression
  would act as at the topical scope.

#### Term rewriting with single dummy variable
The other way to demonstrate topical style is to use two variables: the topic
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
stringPromise
  |> await #
  |> # ?? throw new TypeError()
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
        const • = await stringPromise;
        do { const # = •; # ?? throw new TypeError() }
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

1. The antecedent is first evaluated in the current lexical context.
2. The topic’s result is bound to a hidden special variable `•`.
3. In a new inner lexical context (the topical scope), the value of `•` is
  bound to the topic reference `#`.
4. The pipe’s body is evaluated within this inner lexical context.
5. The pipe’s result is the result of the body.

### Bidirectional associativity
The pipe operator is presented above as a left-associative operator. However, it
is theoretically [bidirectionally associative][associative property]: how a
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

</details>

[`for` iteration statements]: https://tc39.github.io/ecma262/#sec-iteration-statements

[`in` relational operator]: https://tc39.github.io/ecma262/#sec-relational-operators

[annevk]: https://github.com/annevk

[antecedent]: https://en.wikipedia.org/wiki/Antecedent_(grammar)

[ES assignment-level expressions]: https://tc39.github.io/ecma262/#sec-assignment-operators

[associative property]: https://en.wikipedia.org/wiki/Associative_property

[bidirectional associativity]: #bidirectional-associativity

[binding]: https://en.wikipedia.org/wiki/Binding_(linguistics)

[Clojure pipe]: https://clojuredocs.org/clojure.core/as-%3E

[concatenative programming]: https://en.wikipedia.org/wiki/Concatenative_programming_language

[Daniel “littledan” Ehrenberg of Igalia]: https://github.com/littledan

[dataflow programming]: https://en.wikipedia.org/wiki/Dataflow_programming

[Elixir pipe]: https://elixir-lang.org/getting-started/enumerables-and-streams.html

[Elm pipe]: http://elm-lang.org/docs/syntax#infix-operators

[ES grammar notation]: https://tc39.github.io/ecma262/#sec-syntactic-and-lexical-grammars

[ES LHS expressions]: https://tc39.github.io/ecma262/#sec-left-hand-side-expressions

[F# pipe]: https://docs.microsoft.com/en-us/dotnet/fsharp/language-reference/functions/index#function-composition-and-pipelining

[formal grammar]: #grammar

[functional programming]: https://en.wikipedia.org/wiki/Functional_programming

[garden-path syntax]: https://en.wikipedia.org/wiki/Garden_path_sentence

[GitHub issue tracker]: https://github.com/tc39/proposal-pipeline-operator/issues

[Hack pipe]: https://docs.hhvm.com/hack/operators/pipe-operator

[jashkenas]: https://github.com/jashkenas

[Julia pipe]: https://docs.julialang.org/en/stable/stdlib/base/#Base.:|>

[lexically hygienic]: https://en.wikipedia.org/wiki/Hygienic_macro

[littledan invitation]: https://github.com/tc39/proposal-pipeline-operator/issues/89#issuecomment-363853394

[LiveScript pipe]: http://livescript.net/#operators-piping

[MDN’s guide on expressions and operators]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Expressions_and_Operators

[mindeavor]: https://github.com/gilbert

[Node-stream piping]: https://nodejs.org/api/stream.html#stream_readable_pipe_destination_options

[nomenclature]: #nomenclature

[OCaml pipe]: http://blog.shaynefletcher.org/2013/12/pipelining-with-operator-in-ocaml.html

[Perl 6 pipe]: https://docs.perl6.org/language/operators#infix_==&gt;

[Pify]: https://github.com/sindresorhus/pify

[pipeline syntax]: #pipeline-syntax

[possible future extensions to the topic concept]: #possible-future-extensions-to-topic-concept

[previous pipe-operator proposal]: https://github.com/tc39/proposal-pipeline-operator

[previous pipeline-placeholder discussions]: https://github.com/tc39/proposal-pipeline-operator/issues?q=placeholder

[prior pipeline proposal]: https://github.com/tc39/proposal-pipeline-operator/blob/37119110d40226476f7af302a778bc981f606cee/README.md

[Proposal 4: Smart Mix on the pipe-proposal wiki]: https://github.com/tc39/proposal-pipeline-operator/wiki#proposal-4-smart-mix

[ES punctuators]: https://tc39.github.io/ecma262/#sec-punctuators

[reverse Polish notation]: https://en.wikipedia.org/wiki/Reverse_Polish_notation

[R pipe]: https://cran.r-project.org/web/packages/magrittr/index.html

[runtime semantics]: #runtime-semantics

[sindresorhus]: https://github.com/sindresorhus

[smart body syntax]: #smart-body-syntax

[syntactic partial application]: https://github.com/tc39/proposal-partial-application

[tacit programming]: https://en.wikipedia.org/wiki/Tacit_programming

[bare style]: #bare-style

[TC39 process]: https://tc39.github.io/process-document/

[term rewriting]: https://en.wikipedia.org/wiki/Term_rewriting

[term rewriting topical style]: #term-rewriting-topical-style

[term rewriting with autogenerated variables]: #term-rewriting-with-single-dummy-variable

[term rewriting with autogenerated variables]: #term-rewriting-with-single-dummy-variable

[term rewriting with single dummy variable]: #term-rewriting-with-single-dummy-variable

[topic and comment]: https://en.wikipedia.org/wiki/Topic_and_comment

[topic variables in other languages]: https://rosettacode.org/wiki/Topic_variable

[Underscore.js]: http://underscorejs.org

[Unix pipe]: https://en.wikipedia.org/wiki/Pipeline_(Unix

[Fetch Standard]: https://fetch.spec.whatwg.org

[WHATWG-stream piping]: https://streams.spec.whatwg.org/#pipe-chains

[operator precedence]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Operator_Precedence

[operator precedence (MDN)]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Operator_Precedence

[ES clause 11]: https://tc39.github.io/ecma262/#sec-ecmascript-language-lexical-grammar

[ES clause 5.1.2]: https://tc39.github.io/ecma262/#sec-lexical-and-regexp-grammars

[grammar parameters]: #grammar-parameters

[_IdentifierReference_]: https://tc39.github.io/ecma262/#prod-IdentifierReference

[_MemberExpression_]: https://tc39.github.io/ecma262/#prod-MemberExpression

[_IdentifierName_]: https://tc39.github.io/ecma262/#prod-IdentifierName

[ES Clause 5.2.4]: https://tc39.github.io/ecma262/#sec-static-semantic-rules

[lexical grammar]: #lexical-grammar

[object initializers’ Computed Property Contains rule]: https://tc39.github.io/ecma262/#sec-object-initializer-static-semantics-computedpropertycontains

[REPLs]: https://en.wikipedia.org/wiki/Read–eval–print_loop

[Perl 6 topicalization]: https://www.perl.com/pub/2002/10/30/topic.html/
