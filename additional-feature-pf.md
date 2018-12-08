|Name                     | Status  | Features                                                               | Purpose                                                                                                         |
| ----------------------- | ------- | ---------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
|[Core Proposal][]        | Stage 0 | Infix pipelines `… \|> …`<br>Lexical topic `#`                         | **Unary** function/expression **application**                                                                   |
|[Additional Feature BC][]| None    | Bare constructor calls `… \|> new …`                                   | Tacit application of **constructors**                                                                           |
|[Additional Feature BA][]| None    | Bare awaited calls `… \|> await …`                                     | Tacit application of **async functions**                                                                        |
|[Additional Feature BP][]| None    | Block pipeline steps `… \|> {…}`                                       | Application of **statement blocks**                                                                             |
|[Additional Feature PF][]| None    | Pipeline functions `+>  `                                              | **Partial** function/expression **application**<br>Function/expression **composition**<br>**Method extraction** |
|[Additional Feature TS][]| None    | Pipeline `try` statements                                              | Tacit application to **caught errors**                                                                          |
|[Additional Feature NP][]| None    | N-ary pipelines `(…, …) \|> …`<br>Lexical topics `##`, `###`, and `...`| **N-ary** function/expression **application**                                                                   |

# Additional Feature PF
ECMAScript No-Stage Proposal. Living Document. J. S. Choi, 2018-12.

This document is not yet intended to be officially proposed to TC39 yet; it merely shows a possible
extension of the [Core Proposal][] in the event that the Core Proposal is accepted.

This additional feature – **Pipeline Functions** – would dramatically increase the usefulness of
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
expression as a **pipeline** but wraps it in an arrow function that applies its
pipeline steps to its arguments.

A pipe function takes **no** a parameter list; no such list is needed.
Just like with regular pipelines, a pipeline function may be in **[bare style][]
or [topic style][]**.

If the pipeline function starts with [bare style][] (like `+> f |> # + 1`), then
the function is **variadic** and applies **all** its arguments to the function
reference to which the bare-style pipeline step evaluates (that is,
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

In general, Additional Feature NP would explain “`+>` _Pipeline_” as equivalent
to “`(...$rest) => ...$rest |>` _Pipeline_”.

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
topic-style pipeline whose only step evaluates simply to the topic, unmodified.

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
parameter into a topic-style pipeline whose only step is the topic plus two.

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
pipe operator `|>`.

<td>

```js
array.map((...$) => h(g(f(...$))) * 2);
```

<tr>
<td>

```js
+> x + 2;
// 🚫 Syntax Error:
// Pipeline step `+> x + 2`
// binds topic but contains
// no topic reference.
```

This is an [early error][], as usual. The topic is not used anywhere
in the pipeline function’s only step – just like with `… |> x + 2`.

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
are simply pipeline steps that are prefixed by the pipeline-function operator.
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
alone, as a natural result of their pipe-operator-like semantics.\
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

## Ramda (Core Proposal + Additional Feature BP+PF)
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

## WHATWG Streams Standard (Core Proposal + Additional Features BP+PP+PF)
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
        