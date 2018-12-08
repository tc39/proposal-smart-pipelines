|Name                     | Status  | Features                                                               | Purpose                                                                                                         |
| ----------------------- | ------- | ---------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
|[Core Proposal][]        | Stage 0 | Infix pipelines `… \|> …`<br>Lexical topic `#`                         | **Unary** function/expression **application**                                                                   |
|[Additional Feature BC][]| None    | Bare constructor calls `… \|> new …`                                   | Tacit application of **constructors**                                                                           |
|[Additional Feature BA][]| None    | Bare awaited calls `… \|> await …`                                     | Tacit application of **async functions**                                                                        |
|[Additional Feature BP][]| None    | Block pipeline steps `… \|> {…}`                                       | Application of **statement blocks**                                                                             |
|[Additional Feature PF][]| None    | Pipeline functions `+>  `                                              | **Partial** function/expression **application**<br>Function/expression **composition**<br>**Method extraction** |
|[Additional Feature TS][]| None    | Pipeline `try` statements                                              | Tacit application to **caught errors**                                                                          |
|[Additional Feature NP][]| None    | N-ary pipelines `(…, …) \|> …`<br>Lexical topics `##`, `###`, and `...`| **N-ary** function/expression **application**                                                                   |

# Additional Feature NP
ECMAScript No-Stage Proposal. Living Document. J. S. Choi, 2018-12.

This document is not yet intended to be officially proposed to TC39 yet; it merely shows a possible
extension of the [Core Proposal][] in the event that the Core Proposal is accepted.

An Additional Feature – **n-ary pipelines** – would enable the passing of
multiple arguments into a pipeline’s steps. `(a, b) |> f` is equivalent to
`f(a, b)`, and `a |> (f(#), g(#)) |> h` is equivalent to `h(f(a), g(a))`.

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
A pipeline step using commas would be interpreted as an argument list. The
arguments would then be applied to the next pipeline step as its inputs.

<td>

```js
f(a, b);
```

<tr>
<td>

```js
(a, b, ...c, d) |> f |> g;
```
Spread elements are permitted within pipeline steps, with the same meaning as in
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
When a pipeline step only consists of one item, its parentheses may be omitted,
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
When a pipeline step is in [topic style][], the first element in the argument
list is bound to the primary topic reference `#`, the second element is bound to
the secondary topic reference `##`, and the third element is bound to the
tertiary topic reference `###`. These are resolvable as usual within the
pipeline step.

<td>

```js
g(f(a, x, b));
```

<tr>
<td>

```js
...a |> f(x, ...) |> g;
```
A pipeline step also may bind a list of values to a rest topic reference `...`
within the next pipeline step. The list contains the arguments of the pipeline
step that were not bound to any other topic reference. `...` automatically
flattens, acting as a spread operator, and it is valid only where spread
operators are already valid (such as argument lists, array literals, and object
literals).

<td>

```js
g(f(x, ...a));
```

<tr>
<td>

```js
a |> f(#, x, ...) |> g;
```

<td>

```js
{
  const [$, ...$r] = a;
  f($, x, ...$r);
}
```

<tr>
<td>

```js
(a, b, ...c, d) |> f(#, x, ...) |> g;
```

<td>

```js
g(f(a, x, ...[b, ...c, d]));
```

<tr>
<td>

```js
...a |> f;
...a |> f(...); // Equivalent
```
Bare-style pipeline heads are now equivalent to spreading the rest topic
reference into the function’s arguments.

<td>

```js
f(...a);
```

<tr>
<td>

```js
x + 1 |> (f(#), g(#)) |> h;
x + 1 |> (f(#), g(#)) |> h(...); // Equivalent
```
A topic-style pipeline step can be n-ary itself. By taking the form of an
argument list, the step `(f(#), g(#))` passes multiple values into the following
step `h`. Each element of the argument list is itself an expression that may
use any of the topic references `#`, `##`, `###`, or `...` that were bound by
its own input (in the case of `x + 1`, only `#` and `...` are bound).

<td>

```js
{
  const $0 = x + 1;
  const $1 = f($0);
  const $$1 = g($0);
  h($1, $$1);
}
```

<tr>
<td>

```js
(x + 1, y + 1) |> (f(#), g(#, ##)) |> h;
```

<td>

```js
{
  const [$0, $$0] = [x + 1, y + 1];
  const [$1, $$1] = [f($0), g($0, $$0)];
  h($1, $$1);
}
```

<tr>
<td>

```js
...array |> (f(#), g(#, ##)) |> h;
array |> ... |> (f(#), g(#, ##)) |> h; // Equivalent
```
In the second (equivalent) line, the `...` (topic-style) step spreads the values
of `array` into the step’s topic values. Those topic values are in turn inputted
into the next step `(f(#), g(#, ##))`. Before that next step is evaluated, the
first topic value (which is the first value of `array`) is bound to `#`, and the
second topic value (which is the second value of `array`) is bound to `##`. The
result of `(f(#), g(#, ##))` in turn is inputted into `h`.

<td>

```js
{
  const [$, $$] = [...array];
  h(f($) + g($, $$));
}
```

<tr>
<td>

```js
a |> f
```

<td>

```js
f(a)
```

<tr>
<td>

```js
(a) |> f
```

<td>

```js
f(a)
```

<tr>
<td>

```js
(a, b) |> f
```

<td>

```js
f(a, b)
```

<tr>
<td>

```js
(...a) |> f
```

<td>

```js
f(...a)
```

<tr>
<td>

```js
...a |> f
```

<td>

```js
f(...a)
```

<tr>
<td>

```js
(a, b) |> # + ##
```

<td>

```js
a + b
```

<tr>
<td>

```js
() |> # + ##
// 🚫 Syntax Error: Pipeline
// head inputs 0 topic values
// `()` into following step that
// expects 1 topic value.
```

<td>

<tr>
<td>

```js
(a, b) |> f(#, 0, ##)
```

<td>

```js
f(a, 0, b)
```

<tr>
<td>

```js
(a, b) |> f(0, ##)
```

<td>

```js
f(0, b)
```

<tr>
<td>

```js
(a, b, c, d) |> f(#, 0, ...)
```

<td>

```js
f(a, 0, b, c, d)
```

<tr>
<td>

```js
(a, b, c, d) |> f(##, 0, ...)
```

<td>

```js
f(b, 0, c, d)
```

<tr>
<td>

```js
(a, b, c, d) |> f(##, 0, [...])
```

<td>

```js
f(b, 0, [c, d])
```

<tr>
<td>

```js
(a, ...[b, c, d]) |> f(##, 0, [...])
```

<td>

```js
f(b, 0, [c, d])
```

<tr>
<td>

```js
(a, b) |> (# * b, ##) |> f
```

<td>

```js
g(a * b, f(b))
```

<tr>
<td>

```js
(a, b) |> (## * b, ##) |> f
```

<td>

```js
g(b * b, f(b))
```

<tr>
<td>

```js
(a, b) |> (# * b, #) |> f
// 🚫 Syntax Error: Pipeline
// head inputs 2 topic values
// `(a, b)` into following step that
// expects 1 topic value.
```

<td>

<tr>
<td>

```js
(a, b) |> (# * b, f) |> f
// 🚫 Syntax Error:
// Topic-style pipeline step
// `f` in `(# * b, f)` binds
// topic but contains no topic
// reference.
```

<td>

<tr>
<td>

```js
(a, b) |> #
// 🚫 Syntax Error: Pipeline
// head inputs 2 topic values
// `(a, b)` into following step that
// expects 1 topic value.
```

<td>

<tr>
<td>

```js
a |> # + ##
// 🚫 Syntax Error: Pipeline
// head inputs 1 topic value
// `a` into following step that
// expects 2 topic values.
```

<td>

<tr>
<td>

```js
() |> # + 1
// 🚫 Syntax Error: Pipeline
// head inputs 0 topic values
// `()` into following step that
// expects 1 topic value.
```

<td>

<tr>
<td>

```js
(a, b) |> f(#, 0)
// 🚫 Syntax Error: Pipeline
// head inputs 2 topic values
// `(a, b)` into following step that
// expects 1 topic value.
```

<td>

<tr>
<td>

```js
(a, b) |> (#, ##)
// 🚫 Syntax Error: Pipeline
// terminates with a 2-ary
// pipeline step but pipelines
// must terminate with a unary
// pipeline step.
```

<td>

<tr>
<td>

```js
(a, b, c, d, e) |> f(##, x, ...) |> g;
```
The rest topic reference `...` starts from beyond the furthest topic reference
that is used within the pipeline step. Here, the furthest topic reference is the
secondary topic reference `##`: the second argument item. So `[c, d, e]` is
bound to the rest topic reference. The rest topic reference `...` may only be
used where the spread operator `...expression` would also be valid (that is,
argument lists, array literals, and object literals), and it automatically
spreads its elements into whatever expression surrounds it.

<td>

```js
{
  const [$, $$, $$$, ...$r]
    = [a, b, c, d, e];
  g(f(a, $$, x, ...$r));
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
because `##` is not used at all in the pipeline step.

<td>

```js
{
  const $r = [...d, e];
  g(f(a, $$$, x, ...$r));
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
  const [$$, $$$, ...$r] =
    [...b, c, ...d, e];
  g(f(a, $$, $$$, x, ...$r));
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
  const [$$, ...$r] =
    [...b, c, ...d, e];
  g(f(a, $$, x, ...$r));
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

N-ary pipeline steps may be chained by using comma expressions, forming a
**list-style pipeline step**.
```js
(a, b) |> (f, g) |> h;
```
The results of the list will be applied to the following pipeline step as its inputs.

<td>

```js
h(f(a), g(b));
```

<tr>
<td>

The elements in an N-ary pipeline step must be in topic style (like the `# ** c +
#` here).
```js
(a, b)
|> (f(#), # ** c + ##)
|> # - ##;
```
It would be the usual early Syntax Error if `f(#)` was instead just `f`, because
it would be a topic-style pipeline step without a topic reference. (`f(##)` and
`f(...)` would not be a Syntax Error, of course. But `f(###)` would also be a
Syntax Error, because it has only two inputs from the step before it.)

<td>

```js
f(a) - (a ** c + b);
```

<tr>
<td>

```js
(a, b)
|> (f(#), g(##))
|> h
|> (i(#), # + 1, k(###))
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
|> (f(#), g(##))
|> (h, i);
// 🚫 Syntax Error:
// Pipeline terminates with a
// 2-ary pipeline step but
// pipelines must terminate
// with a unary pipeline step.
```
It is an [early error][] for a pipeline to end with an n-ary pipeline step,
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
Because arrow functions have looser precedence than the pipe operator `|>`,
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
  const [$, , $$$, ...$r] =
    createRange(number);
  [$, $$$, $r];
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
  const [...$r] = f(input);
  g([0, 1, 2, ...$r]);
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
As a result of the rules, `… |> [...]` collects its input’s n-ary arguments into
a single flattened list, to which the rest topic reference `...` is then bound,
then spread into an array literal.

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

<td>

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
“`+>` _Pipeline_” as equivalent to a function with a variadic pipeline
“`(...$rest) => ...$rest |>` _Pipeline_”.
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

</table>

## Lodash (Core Proposal + Additional Features BP+PP+PF+NP)

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
      else #
      |> toInteger
      |> nativeMin(#, 292);
    };
    return number |> {
      if (precision) #
      // Shift with
      // exponential notation
      // to avoid
      // floating-point
      // issues. See
      // https://mdn.io/round.
      |> `${#}e`
      |> ...#.split('e')
      |> `${#}e${+## + precision}`
      |> func
      |> `${#}e`
      |> ...#.split('e')
      |> `${#}e${+## - precision}`
      |> +#;
      else #
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
      : nativeMin(
        toInteger(precision), 292)
    if (precision) {
      // Shift with
      // exponential notation
      // to avoid
      // floating-point
      // issues. See
      // https://mdn.io/round.
      var pair =
          (toString(number) + 'e')
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

## Ramda (Core Proposal + Additional Features BP+PF+NP)
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

## WHATWG Streams Standard (Core Proposal + Additional Features BP+PP+PF+NP)
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
    else #
    |> (#, offset, #.byteLength - offset)
    |> new Uint8Array
    |> await reader.read
    |> (#.buffer, #.byteLength + offset)
    |> readInto;
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
        