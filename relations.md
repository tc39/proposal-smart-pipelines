# Relations to other work
Living Document. J. S. Choi, 2018-12.

<details>
<summary>Table of contents</summary>

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


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

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

</details>

## Pipelines in other programming languages
The concept of a pipe operator appears in numerous other languages, variously
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

Pipe operators are also conceptually similar to [WHATWG-stream piping][] and
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
into binding it][opt-in behavior] by using the pipe operator `|>`. (This
includes [Additional Feature TS][], which requires the use of `|>`.) The topic
also cannot be accidentally used; it is an [early error][] when `#` is used
outside of a pipeline step (see [Core Proposal][] and [static analyzability][]).
The proposal is as a whole designed to [prevent footguns][“don’t shoot me in the
foot”].

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
step in [topic style][] may use a `do` expression, as long as the `do` expression
contains the topic reference `#`. The topic reference `#` is bound to its
input’s value, the `do` expression is evaluated, then the result of the
`do` block becomes the final result of that pipeline, and the lexical
environment is reset – all as usual.

In this manner, pipelines with `do` expressions act as a way to create a
“topic-context block”, similarly to [Perl 6’s given block][]. Within this block,
statements may use the topic reference may be used as an abbreviation for the
same value. This can be useful for embedding side effects, `if` `else`
statements, `try` statements, and `switch` statements within pipelines.
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
alone, as a natural result of their pipe-operator-like semantics.\
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
```js
const toSlug =
+> #.split(' ')
|> #.map(+> #.toLowerCase())
|> #.join('-')
|> encodeURIComponent;
```
When compared to the proposal for [syntactic functional composition by Isiah
Meadows][isiahmeadows functional composition], this syntax does not need to
surround each non-function expression with an arrow function. The [smart step
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
partial-application expressions are simply pipeline steps that are prefixed by
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
pipeline step. No existing code would be affected. Any, some, or none of the
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
    #|> f,
  TypeError:
    #|> g |> h(#, {strict: true}),
  Error:
    throw #,
};
```
With ECMAScript pattern matching + the smart-pipelines [Core Proposal][] +
[Additional Feature TS][], handling caught errors (and promise rejections) based
on error type becomes more ergonomic.

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
materials.map { #|> f |> .length; }
```
The first parameter of the arrow function that the block parameter implicitly
creates is bound to the primary topic, which in turn is fed into the pipeline:
`#|> f |> .length`.

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
supported within pipeline steps. When this occurs, topic references would be
allowed within inner `do` expressions, along with arrow functions and `if` and
`try` statements. [Additional Feature BP][] would extend this further, also
supporting pipeline-step blocks that act nearly identically to `do` expressions.

## Private class fields, class decorators, nullish coalescing, and optional chaining
This proposal’s compatibility with these four proposals depends on its choice of
tokens for its topic references, such as `#`/`##`/`###`/`...`, `@`/`@@`/`@@@`,
or `?`/`??`/`???`. This is being bikeshedded at [tc39/proposal-pipe-operator
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
smart pipelines. Kuizinas’s plugin makes the choice to insert input values
into its pipeline steps’ last parameters, which is convenient for Ramda-style
functions but not for (equally reasonable) Underscore-style functions.

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
There are several other alternative pipe-operator proposals competing with
the smart-pipeline Core Proposal. The Core Proposal is only one variant of the
[first pipe-operator proposal][] also championed by Ehrenberg; this variant
is listed as [**Proposal 4: Smart Mix** in the pipe-proposal wiki][Pipeline
Proposal 4]. The variant resulted from [previous discussions in the previous
pipe-operator proposal][previous pipeline-placeholder discussions],
discussions which culminated in an [invitation by Ehrenberg to try writing a
specification draft][littledan invitation].

All variants attempt to address the goals of [untangled flow][],
[distinguishable punctuators][], [terse function calls][], and [human
writability][]. But the authors of this proposal believe that the smart pipe
operator may be the best choice among these competing proposals at fulfilling
all the [goals][] listed above.

Only the smart pipe operator does not need to create unnecessary one-off
arrow functions for non-function-call expressions, which better fulfills the
goal of [zero runtime cost][]. Only the smart pipe operator has the [forward
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

Smart pipelines and their [smart step syntax][] sacrifice a small amount of
[simplicity][cyclomatic simplicity] in return for a vast amount of [expressive
versatility][] and [conceptual generality][]. And because it makes many of the
other operator proposals above either unnecessary or possibly simpler, it may
result in less complexity on average anyway. And thanks to its [syntactic
locality][] and numerous [statically detectable early errors][early errors], the mental
burden on the developer in remembering [smart step syntax][] is light.

The benefits of smart pipelines on many real-world examples are well
demonstrated in the [Motivation][] section above, and many of the examples are
not possible with the other pipeline proposals. It is hoped that the Core
Proposal is strongly considered by TC39, keeping in mind that its simple but
versatile syntax would open the door to addressing the use cases of many other
proposals in a uniform manner.

["data-to-ink" visual ratio]: https://www.darkhorseanalytics.com/blog/data-looks-better-naked
["don’t break my code"]: ./goals.md#dont-break-my-code
["don’t make me overthink"]: ./goals.md#dont-make-me-overthink
["don’t shoot me in the foot"]: ./goals.md#dont-shoot-me-in-the-foot
["make my code easier to read"]: ./goals.md#make-my-code-easier-to-read
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
[additional features]: ./readme.md#additional-features
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
[bare awaited function call]: ./core-syntax.md#bare-style
[bare constructor call]: ./core-syntax.md#bare-style
[bare function call]: ./core-syntax.md#bare-style
[bare style]: ./core-syntax.md#bare-style
[binding]: https://en.wikipedia.org/wiki/Binding_(linguistics)
[block parameters]: ./relations.md#block-parameters
[Clojure compact function]: https://clojure.org/reference/reader#_dispatch
[Clojure pipe]: https://clojuredocs.org/clojure.core/as-%3E
[completion records]: https://timothygu.me/es-howto/#completion-records-and-shorthands
[concatenative programming]: https://en.wikipedia.org/wiki/Concatenative_programming_language
[conceptual generality]: ./goals.md#conceptual-generality
[Core Proposal]: ./readme.md
[core-real-examples.md]: ./core-real-examples.md
[currying]: https://en.wikipedia.org/wiki/Currying
[cyclomatic complexity]: https://en.wikipedia.org/wiki/Cyclomatic_complexity#Applications
[cyclomatic simplicity]: ./goals.md#cyclomatic-simplicity
[dataflow programming]: https://en.wikipedia.org/wiki/Dataflow_programming
[distinguishable punctuators]: ./goals.md#distinguishable-punctuators
[don’t break my code]: ./goals.md#dont-break-my-code
[DSLs]: https://en.wikipedia.org/wiki/Domain-specific_language
[early error rule]: ./goals.md#static-analyzability
[early error rules]: ./goals.md#static-analyzability
[early error]: ./goals.md#static-analyzability
[early errors]: ./goals.md#static-analyzability
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
[expressive versatility]: ./goals.md#expressive-versatility
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
[forward compatibility]: ./goals.md#forward-compatibility
[forward compatible]: ./goals.md#forward-compatibility
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
[jQuery + CP]: ./core-real-examples.md#jquery
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
[Lodash + CP]: ./core-real-examples.md#lodash
[Lodash]: https://lodash.com/
[mAAdhaTTah]: https://github.com/mAAdhaTTah/
[make my code easier to read]: ./goals.md#make-my-code-easier-to-read
[MDN operator precedence]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Operator_Precedence
[MDN operator precedence]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Operator_Precedence#Table
[method-extraction inline caching]: https://github.com/tc39/proposal-bind-operator/issues/46
[mindeavor]: https://github.com/gilbert
[mode errors]: https://en.wikipedia.org/wiki/Mode_(computer_interface)#Mode_errors
[motivation]: ./readme.md
[Node-stream piping]: https://nodejs.org/api/stream.html#stream_readable_pipe_destination_options
[Node.js `util.promisify`]: https://nodejs.org/api/util.html#util_util_promisify_original
[nomenclature]: ./nomenclature.md
[novice learnability]: ./goals.md#novice-learnability
[nullish coalescing proposal]: https://github.com/tc39/proposal-nullish-coalescing/
[object initializers’ Computed Property Contains rule]: https://tc39.github.io/ecma262/#sec-object-initializer-static-semantics-computedpropertycontains
[OCaml pipe]: http://blog.shaynefletcher.org/2013/12/pipelining-with-operator-in-ocaml.html
[operator precedence]: ./core-syntax.md#operator-precedence-and-associativity
[opt-in behavior]: ./goals.md#opt-in-behavior
[optional `catch` binding]: ./relations.md#optional-catch-binding
[optional-chaining syntax proposal]: https://github.com/tc39/proposal-optional-chaining
[other browsers’ console variables]: https://www.andismith.com/blogs/2011/11/25-dev-tool-secrets/
[other ECMAScript proposals]: ./relations.md#other-ecmascript-proposals
[other goals]: ./goals.md#other-goals
[pattern matching]: ./relations.md#pattern-matching
[partial function application]: ./goals.md#partial-function-application
[PEP 20]: https://www.python.org/dev/peps/pep-0020/
[Perl 6 pipe]: https://docs.perl6.org/language/operators#infix_==&gt;
[Perl 6 topicization]: https://www.perl.com/pub/2002/10/30/topic.html/
[Perl 6’s given block]: https://docs.perl6.org/language/control#given
[pipeline functions]: ./additional-feature-pf.md
[Pipeline Proposal 1]: https://github.com/tc39/proposal-pipeline-operator/wiki#proposal-1-f-sharp-only
[Pipeline Proposal 4]: https://github.com/tc39/proposal-pipeline-operator/wiki#proposal-4-smart-mix
[pipeline syntax]: ./core-syntax.md
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
[semantic clarity]: ./goals.md#semantic-clarity
[simonstaton functional composition]: https://github.com/simonstaton/Function.prototype.compose-TC39-Proposal
[simple scoping]: ./goals.md#simple-scoping
[sindresorhus]: https://github.com/sindresorhus
[smart step syntax]: ./core-syntax.md
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
[topic style]: ./core-syntax.md#topic-style
[topic variables in other languages]: https://rosettacode.org/wiki/Topic_variable
[topic-token bikeshedding]: https://github.com/tc39/proposal-pipeline-operator/issues/91
[Underscore.js + CP + BP + PP]: ./additional-feature-pp.md#underscorejs-core-proposal--additional-feature-bppp
[Underscore.js + CP]: ./core-real-examples.md#underscorejs
[Underscore.js]: http://underscorejs.org
[Unix pipe]: https://en.wikipedia.org/wiki/Pipeline_(Unix
[untangled flow]: ./goals.md#untangled-flow
[Visual Basic’s `select` statement]: https://docs.microsoft.com/en-us/dotnet/visual-basic/language-reference/statements/select-case-statement
[WebKit console variables]: https://webkit.org/blog/829/web-inspector-updates/
[WHATWG Fetch + CP]: ./core-real-examples.md#whatwg-fetch-standard
[WHATWG Fetch Standard]: https://fetch.spec.whatwg.org/
[WHATWG Streams + CP + BP + PF + NP]: ./additional-feature-np.md#whatwg-streams-standard-core-proposal--additional-features-bppfmt
[WHATWG Streams + CP + BP + PF]: ./additional-feature-pf.md#whatwg-streams-standard-core-proposal--additional-feature-bppf
[WHATWG Streams Standard]: https://stream.spec.whatwg.org/
[WHATWG-stream piping]: https://streams.spec.whatwg.org/#pipe-chains
[Wikipedia: term rewriting]: https://en.wikipedia.org/wiki/Term_rewriting
[zero runtime cost]: ./goals.md#zero-runtime-cost
        