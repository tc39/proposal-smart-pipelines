# Smart pipelines
ECMAScript Stage-(−1) Proposal by J. S. Choi, 2018-02.

This repository contains the formal specification for a proposed “smart pipe operator” `|>` in JavaScript. It is currently not even in Stage 0 of the [TC39 process][TC39 process] but it may eventually be presented to TC39.

## Background
The binary operator proposed here would provide a backwards- and forwards-compatible style of chaining nested expressions into a readable, left-to-right manner. Nested transformations become untangled into short steps in a zero-cost abstraction. The operator is similar to [Hack’s `|>` and `$$`][Hack pipe], [F#’s `|>`][F# pipe], [OCaml’s `|>`][OCaml pipe], [Elixir/Erlang’s `|>`][Elixir pipe], [Elm’s `|>`][6], [Julia’s `|>`][7], [LiveScript’s `|>`][8], [R / magrittr’s `%>%`][9], [Perl 6’s `==>`][10], [Clojure’s `->` and `as->`][Clojure pipe], and [Unix shells’ and PowerShell’s `|`][11]). It is conceptually similar but should not be confused with [WHATWG-stream piping][12] and [Node-stream piping][13].

The proposal is a variant of a [previous pipe-operator proposal][14] championed by [Daniel Ehrenberg of Igalia][15]. This variant is listed as [Proposal 4: Smart Mix on the pipe-proposal wiki][16]. The variant resulted from [previous discussions about pipeline placeholders in the previous pipe-operator proposal][17], which culminated in an [invitation by Ehrenberg to try writing a specification draft][18]. A prototype Babel plugin is also pending.

You can take part in the discussions on the [GitHub issue tracker][19]. When you file an issue, please note in it that you are talking specifically about “Proposal 4: Smart Mix”.

Note that the use of `#` as the topic variable is up for debate. `@` would be similarly terse, visually distinguishable, and easily typeable. [Bikeshedding over what characters to use for the topic token is occurring in a GitHub issue](https://github.com/tc39/proposal-pipeline-operator/issues/91).

## Motivation
Let these two functions be defined:
```js
function doubleSay (string, separatorString) {
  return `${string}${separatorString}${string}`
}

function capitalize (string) {
  return string[0].toUpperCase() + string.substring(1)
}
```

Now look at the nested expression below. Expressions like this happen often in JavaScript: whenever any value must be passed through a series of transformations, whether they be operations, functions, or constructors. Unfortunately, this nested expression—and many like it—are quite messy spaghetti. Writing it requires many levels of indentation, and reading it requires checking both the left and right of each subexpression to understand its data flow.
```js
new User.Message(
  capitalizedString(
    doubledString(
      (await stringPromise)
        ?? throw new TypeError(`Expected string from ${stringPromise}`)
    )
  ) + '!'
)
```

The code above could be much terser and (literally) straightforward. This proposal’s smart pipe operator would allow the piping of data through expressions. This terseness would make both reading and writing easier for the JavaScript programmer. It is statically rewritable into already valid code, it has zero runtime cost.
```js
stringPromise
  |> await #
  |> # ?? throw new TypeError()
  |> doubleSay(#, ', ')
  |> capitalize // a tacit unary function call
  |> # + '!'
  |> new User.Message // a tacit unary constructor call
```

Similar use cases appear numerous times in JavaScript code, whenever any value is transformed by expressions of any type: function calls, property calls, method calls, object constructions, arithmetic operations, logical operations, bitwise operations, `typeof`, `instanceof`, `await`, `yield` and `yield *`, and `throw` expressions. The smart pipe operator can handle them all.

Note also that it was not necessary to include parentheses for `capitalize` or `new User.Message`; they were implicitly included as a unary function call and a unary constructor call, respectively. That is, the preceding example is equivalent to:
```js
'hello'
  |> await #
  |> # ?? throw new TypeError(`Expected string from ${#}`)
  |> doubleSay(#, ', ')
  |> capitalize(#)
  |> # + '!'
```

Being able to automatically detect this tacit style is the “smart” part of this “smart pipe operator”; for more information, see the explanation of “[tacit style][]” and of “[smart body syntax][]”.

Now following are various examples adapted from useful, real-world code.

### Underscore.js
Adapted from [Underscore.js 1.8.3][21] by Jeremy Ashkenas et al. under MIT License.
```js
_.find = _.detect = function(obj, predicate, context) {
  var key;
  if (isArrayLike(obj)) {
    key = _.findIndex(obj, predicate, context);
  } else {
    key = _.findKey(obj, predicate, context);
  }
  if (key !== void 0 && key !== -1) return obj[key];
};
```
```js
_.find = _.detect = function(obj, predicate, context) {
  return obj
    |> isArrayLike(#) ? _.findIndex : _.findKey
    |> #(obj, predicate, context)
    |> (# !== void 0 && # !== -1) ? obj[#] : undefined;
};
```
***
```js
_.reject = function(obj, predicate, context) {
  return _.filter(obj,
    _.negate(cb(predicate)),
    context
  )
};
```
```js
_.reject = function(obj, predicate, context) {
  return predicate
    |> cb
    |> _.negate
    |> _.filter(obj, #, context)
};
```
***
```js
var executeBound = function(sourceFunc, boundFunc, context, callingContext, args) {
  if (!(callingContext instanceof boundFunc))
    return sourceFunc.apply(context, args);
  var self = baseCreate(sourceFunc.prototype);
  var result = sourceFunc.apply(self, args);
  if (_.isObject(result)) return result;
  return self;
};
```
```js
var executeBound = function(sourceFunc, boundFunc, context, callingContext, args) {
  if (callingContext |> # instanceof boundFunc |> !#)
    return sourceFunc.apply(context, args);
  var self = sourceFunc
    |> #.prototype
    |> baseCreate;
  return self
    |> sourceFunc.apply(#, args)
    |> _.isObject(#) ? # : self;
};
```
***
```js
_.size = function(obj) {
  if (obj == null) return 0;
  return isArrayLike(obj)
    ? obj.length
    : _.keys(obj).length;
};
```
```js
_.size = function(obj) {
  if (obj == null) return 0;
  return obj |> isArrayLike
    ? obj.length
    : obj |> _.keys |> #.length;
};
```

### Pify
Adapted from [Pify 3.0.0][22] by Sindre Sorhus under MIT License.
```js
pify(fs.readFile)('package.json', 'utf8').then(data => {
  console.log(JSON.parse(data).name)
})
```
```js
console.log(JSON.parse((await pify(fs.readFile)('package.json', 'utf8')).name));
```
```js
'package.json'
  |> await pify(fs.readFile)(#, 'utf8')
  |> JSON.parse
  |> #.name
  |> console.log
```
***
```js
return opts.include
  ? opts.include.some(match)
  : !opts.exclude.some(match)
```
```js
return opts.include
  ? opts.include.some(match)
  : opts.exclude.some(match) |> !#
```

### Fetch Standard
Adapted from [Fetch, the living web standard][23] by WHATWG under Creative Commons BY.
```js
fetch("/music/pk/altes-kamuffel.flac")
  .then(res => res.blob())
  .then(playBlob)
```
```js
playBlob(await (await fetch("/music/pk/altes-kamuffel.flac")).blob())
```
```js
"/music/pk/altes-kamuffel.flac"
  |> await fetch(#)
  |> await #.blob()
  |> playBlob
```
***
```js
fetch("/", { method: "HEAD" })
  .then(res => log(res.headers.get("strict-transport-security")))
```
```js
(await fetch("/", { method: "HEAD" }))
  .headers.get("strict-transport-security")
```
```js
"/"
  |> await fetch(#, { method: "HEAD" })
  |> #.headers.get("strict-transport-security")
```
***
```js
fetch("https://pk.example/berlin-calling.json", { mode: "cors" })
  .then(response => {
    if (response.headers.get("content-type")??.toLowerCase()
      .indexOf("application/json") >= 0
    ) {
      return response.json()
    } else {
      throw new TypeError()
    }
  }).then(processJSON)
```
```js
const res = await fetch("https://pk.example/berlin-calling.json", {
  mode: "cors"
});
processJSON(do {
  if (res.headers.get("content-type")??.toLowerCase()
    .indexOf("application/json") >= 0
  ) {
    await res.json()
  } else {
    throw new TypeError()
  }
})
```
```js
const response = "https://pk.example/berlin-calling.json"
  |> await fetch(#, { mode: "cors" });
response
  |> #.headers.get("content-type")
  |> #??.toLowerCase()
  |> #.indexOf("application/json")
  |> # >= 0
  |> # ? response : throw new TypeError()
  |> await #.json()
  |> processJSON
```

## Nomenclature
The binary operator itself may be referred to as a <dfn>pipe</dfn>, a <dfn>pipe operator</dfn>, or a <dfn>pipeline operator</dfn>; all these names are equivalent. This specification will prefer the term “pipe operator”.

A pipe operator between two expressions forms a <dfn>pipe expression</dfn>. One or more pipe expressions in a chain form a <dfn>pipeline</dfn>. For each pipe expression, the expression before the pipe is the pipeline’s <dfn>left-hand side (LHS)</dfn>, also known as its <dfn>topic expression</dfn> or its <dfn>topic</dfn>; the expression after the pipe is its <dfn>right-hand side (RHS)</dfn>, also known as its <dfn>body expression</dfn> or its <dfn>body</dfn>. The pipe operator is said <dfn>to pipeline</dfn> the LHS/topic’s value <dfn>through</dfn> the RHS/body expression, where “pipeline” here is used as a verb.

A <dfn>`PipelineExpression`</dfn> or <dfn>pipeline-level expression</dfn> is an expression at the same [precedence level of the pipe operator][pipeline syntax]. Although all pipelines are `PipelineExpression`s, most `PipelineExpression`s are not actually pipelines; conditional operations, logical-or operations, or any other expressions that have tighter syntactic precedence than the pipe operation—those are also `PipelineExpression`s.

### Topical style
The special token `#` is a <dfn>topic variable</dfn>: it is a nullary operator that acts as a special variable.

The topic variable is *not* an identifier and cannot be manually declared (`const #` is a syntax error), nor can it be assigned with a value (`# = 3` is a syntax error). Instead, the topic variable may be implicitly, lexically bound using a pipeline.

When a pipeline’s body expression is in <dnf>topical style</dfn>, the body expression forms an inner lexical scope—called the pipeline’s <dfn>body scope</dfn>—within which the topic variable is implicitly bound to the value of the topic, acting as a **placeholder** for the topic’s value.

### Tacit style
Alternatively, you may omit the body expression’s topic variables if the body expression is just a <dfn>simple reference</dfn> to a function or constructor, such as with `… |> capitalize` and `… |> new User.Message`.

The body expression would then be called as a unary function or constructor, without having to use the topic variable as an explicit argument.

This is called [<dfn>tacit style</dfn> or <dfn>point-free style</dfn>][24]. When a pipe is in tacit style, the body expression is called a <dfn>tacit function</dfn> or a <dfn>tacit constructor</dfn>, depending on if it starts with `new`. Either way, the body must otherwise consist of only a <dfn>simple property chain</dfn>, which is simply an identifier (like `abc`), optionally chained with member properties (like `abc.property`). They are more formally defined in [smart body syntax][].

Tacit style never has parentheses in the pipeline body (that is, do this `… |> fetch` and not this `… |> fetch()`). If you need parentheses, you must use a topic variable (like this `… |> fetch(#, { method: 'options' })`).

## Informative edge cases
These edge cases are natural results of the rules in the [formal grammar][]

### Multiple topic variables in a pipeline body
The topic variable may be used multiple times in a pipeline body. Each use refers to the same value (wherever the topic variable is not overridden by another, inner pipeline’s body scope). Because it is bound to the result of the topic, the topic is still only ever evaluated once. The lines in each of the following are equivalent:
```js
do { const $ = …; f($, $) }
… |> f(#, #)
```
***
```js
do { const $ = …; [$, $ * 2, $ * 3] }
… |> [#, # * 2, # * 3]
```

### Inner functions
Both the topic expression and the body expression of a pipeline may contain nested inner functions. The lines in the following are equivalent:
```js
do { const $ = …; settimeout(() => $ * 500) }
… |> settimeout(() => # * 500)
```

### Deeply nested pipelines
Both the topic expression and the body expression of a pipeline may contain nested inner functions. This is not encouraged, but it is still permitted. All lines in the following are equivalent:
```js
do { const $ = …; settimeout(() => f($) * 500) }
do { const $ = …; settimeout(() => f($) |> # * 500) }
do { const $ = …; settimeout(() => $ |> f |> # * 500) }
… |> settimeout(() => f(#) * 500)
… |> settimeout(() => f(#) |> # * 500)
… |> settimeout(() => # |> f |> # * 500)
```

### Property-chained tacit style
[TO DO]

| Expression                            | Result                          |
| ------------------------------------- | ------------------------------- |
| **`'2018' \|> Date(#)`**              | **`Date('2018')`**              |
| **`'2018' \|> Date`**                 | **`Date('2018')`**              |
|   `'2018' \|> Date()`                 |   syntax error: missing `#`     |
|   `'2018' \|> (Date)`                 |   syntax error: missing `#`     |
| **`'2018' \|> Date.parse(#)`**        | **`Date.parse('2018')`**        |
| **`'2018' \|> Date.parse`**           | **`Date.parse('2018')`**        |
|   `'2018' \|> Date.parse()`           |   syntax error: missing `#`     |
|   `'2018' \|> (Date.parse)`           |   syntax error: missing `#`     |
| **`'2018' \|> global.Date.parse(#)`** | **`global.Date.parse('2018')`** |
| **`'2018' \|> global.Date.parse`**    | **`global.Date.parse('2018')`** |
|   `'2018' \|> global.Date.parse()`    |   syntax error: missing `#`     |
|   `'2018' \|> (global.Date.parse)`    |   syntax error: missing `#`     |
| **`'2018' \|> new global.Date(#)`**   | **`new Date('2018')`**          |
| **`'2018' \|> new global.Date`**      | **`new Date('2018')`**          |
|   `'2018' \|> new global.Date()`      |   syntax error: missing `#`     |
|   `'2018' \|> (new global.Date)`      |   syntax error: missing `#`     |

## Grammar
This proposal uses the [same grammar notation as that from the ES standard][es-grammar].

### Lexing
The lexical rule for [`Punctuator` tokens][es-punctuators] would be modified. Two new tokens, `|>` for the binary pipe and `#` for the topic variable, would be added to the Punctuators:
```
Punctuator :: one of
  `{` `(` `)` `[` `]` `.` `...` `;` `,` `<` `>` `<=` `>=` `==` `!=` `===` `!==`
  `+` `-` `*` `%` `**` `++` `--`
  `<<` `>>` `>>>` `&` `|` `^` `!` `~` `&&` `||` `?` `:` `|>` `#`
  `=` `+=` `-=` `*=` `%=` `**=` `<<=` `>>=` `>>>=` `&=` `|=` `^=` `=>`
```

### Syntax parameters
In the ES standard, expressions in general are parameterized with three flags:

* <dfn>`In`</dfn>: Whether the current context allows the [`in` relational operator][], which is false only in the headers of [`for` iteration statements][].
* <dfn>`Yield`</dfn>: Whether the current context is within a `yield` expression/declaration.
* <dfn>`Await`</dfn>: Whether the current context is within an `await` expression/declaration.

### Pipeline syntax
A pipeline `topic |> body` is a left-associative (though see [bidirectional associativity][]) binary operation with a loose precedence.

Precedence is tighter than assignment (`=`, `+=`, …), generator `yield` and `yield *`, and sequence `,`; and it is looser than logical ternary conditional (`… ? … : …`), logical and/or `&&`/`||`, bitwise and/or/xor, `&`/`|`/`^`, equality/inequality `==`/`===`/`!=`/`!==`, and every other type of expression. See also [MDN’s guide on expressions and operators][]. If the pipe operation were any tighter than this level, its body would have to be parenthesized for many frequent types of expressions. However, the result of a pipeline is also expected to often serve as the body of a variable assignment `=`, so it is tighter than assignment operators.

The [assignment operator][] is already parameterized on `In`, `Yield`, and `Await`; it may be a conditional expression, yield expression, arrow function, async arrow function, or assignment:
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

Here, the conditional-expression rule (`AssignmentExpression : ConditionalExpression`) would be replaced with one for pipeline expressions (`AssignmentExpression : PipelineExpression`):
```
// New version
AssignmentExpression[In, Yield, Await] :
  PipelineExpression[?In, ?Yield, ?Await]
  [+Yield] YieldExpression[?In, ?Await]
  …
```

An expression is a pipeline-level expression (a `PipelineExpression`) only if:

* It is either also a conditional-level expression `ConditionalExpression`, with the same parameters above;
* Or it is another pipeline-level expression, followed by a `|>` token, then a pipeline body (`PipelineBody`, defined next), with the same parameters as above.

```
// New rule
PipelineExpression[In, Yield, Await] :
  ConditionalExpression[?In, ?Yield, ?Await]
  PipelineExpression[?In, ?Yield, ?Await] `|>`
    PipelineBody[?In, ?Yield, ?Await]
```

### Smart body syntax
When the parser checks the RHS/body expression, it must determine what style it is in. There are four outcomes for each pipeline body: tacit constructor, tacit function, topical-style expression, or syntax error.

```
// New rule
PipelineBody[In, Yield, Await] :
  PipelineTacitFunction
  PipelineTacitConstructor
  PipelineTopicalBody[In, Yield, Await]
```

The goal here is to minimize the parsing lookahead that the compiler must check before it can distinguish between tacit style and topic-token style. By restricting the space of valid tacit-style pipeline bodies (that is, without topic variables), the rule prevents [garden-path syntax][25] that would otherwise be possible: such as `… |> compose(f, g, h, i, j, k, #)`.

Another goal is to statically prevents a writing JavaScript programmer from accidentally omitting a topic variable where they meant to put one. For instance, if `x |> 3` were not a syntax error, then it would be a useless operation and almost certainly not what the writer intended. The JavaScript programmer is encouraged to use topic variables and avoid tacit style, where tacit style may be visually confusing to the reader.

1. If the body is a mere identifier, optionally with a chain of properties, and with no parentheses or brackets, then that identifier is interpreted to be a **tacit function**.
  ```
  PipelineTacitFunction :
    SimpleMemberExpression
  ```

2. If the body starts with `new`, followed by mere identifier, optionally with a chain of properties, and with no parentheses or brackets, then that identifier is interpreted to be a **tacit constructor**.
  ```
  PipelineTacitConstructor :
    `new` SimpleMemberExpression
  ```

3. Otherwise, the body is now an arbitrary expression. Because the expression is not in tacit style, it must use that pipe’s topic variable. ([TO DO: Formalize this static semantic as `usesTopic()`.] Topic variables from the body scopes of other, inner pipelines do not count.) If there is no such topic variable in the expression, then a syntax error is thrown.

  `PipelineTopicalBody` will be defined in the next section.

If a pipeline body *never* uses a topic variable, then it must be a permitted tacit unary function (single identifier or simple property chain). Otherwise, it is a syntax error. In particular, tacit style *never* uses parentheses. If they need to have parentheses, then they need to have use the topic variable. See also [property-chained tacit style][].

### General semantics
[TO DO: Rewrite this section to the conventions of the ES specification.]

A pipe expression’s semantics are:
1. The topic expression is evaluated into the topic’s value; call this `topicValue`.
2. [The body expression is tested for its type][26]: Is it in tacit style (as a tacit function or a tacit constructor), is it in topical style, or is it an invalid body?

  * If the body is a tacit function (such as `f` and `M.f`):
    1. The body is evaluated (in the current lexical context); call this value the `bodyFunction`.
    2. The entire pipeline’s value is `bodyFunction(topicValue)`.

  * If the body is a tacit constructor (such as `new C` and `new M.C`):
    1. The portion of the body after `new` is evaluated (in the current lexical context); call this value the `BodyConstructor`.
    2. The entire pipeline’s value is `new BodyConstructor(topicValue)`.

  * If the body is in topical style (such as `f(#, n)`, `o[s][#]`, `# + n`, and `#.p`):
    1. An inner lexical context is entered.
    2. Within this inner context, `topicValue` is bound to a variable.
    3. Within this inner context, with the variable binding to `topicValue`, the body is evaluated; call the resulting value `bodyValue`.
    4. The entire pipeline’s value is `bodyValue`.

  * Otherwise, if the body is an invalid body (such as `f()`, `f(n)`, `o[s]`, `+ n`, and `n`), then throw a syntax error explaining that the pipeline’s topic variable is missing from its body.

Body expressions in topical style can be further explained with a nested `do` expression. There are two ways to illustrate this equivalency. The first way is to [replace each pipe expression’s topic variables with an autogenerated variable][28], which must be guaranteed to be [lexically hygienic][29] and to not conflict with other variables. The alternative way is to [use two variables—the topic variable `#` and a single dummy variable][30]—which also preserves [lexical hygiene][31].

#### Topic-variable semantics by replacing them with autogenerated variables
The first way to illustrate the operator’s semantics is to replace each pipe expression’s topic variables with an autogenerated variable, which must be guaranteed to not conflict with other variables.

Let us say that each pipe expression autogenerates a new, [lexically hygienic][32] variable (`#₀`, `#₁`, `#₂`, `#₃`, …), which in turn replaces each use of the topic variable `#` in each pipe’s body. (These `#ₙ` variables are not true syntax; it is merely for illustrative purposes. You cannot actually assign or use `#ₙ` variables.) Let us also group the expressions with left associativity (although this is arbitrary, because [right associativity would also work][bidirectional associativity]).

With this notation, each line in this example would be equivalent to the other lines.
```js
1 |> # + 2 |> # * 3

// Static rewriting
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
  |> doubleSay // a tacit unary function call
  |> capitalize // also a tacit unary function call
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

In general, for each pipe expression `topic |> body`, assuming that `body` is in topical style, that is, assuming that `body` contains an unshadowed topic variable:

* Let `#ₙ` be a [hygienically autogenerated][34] topic variable, `#ₙ`, where <var>n</var> is a number that would not conflict with the name of any other autogenerated topic variable in the scope of the entire pipe expression.
* Also let `substitutedBody` be `body` but with all instances of `#` replaced with `#ₙ`.
* Then the static syntax rewriting (left associative and inside to outside) would simply be: `do { const #ₙ = topic; substitutedBody }`. This `do` expression would act as at the body scope.

#### Topic-variable semantics by alternating lexical shadowing with dummy variable
The other way to demonstrate topical style is to use two variables: the topic variable `#` and single [lexically hygienic][35] dummy variable `•`. It should be noted that `const # = …` is not a valid statement under this proposal’s actual syntax; likewise, `•` is not a part of the proposal’s syntax. Both forms are for illustrative purposes here only.

With this notation, no variable autogeneration is required; instead, the nested `do` expressions will redeclare the same variables `#` and `•`, shadowing the external variables of the same name as needed. The number example above becomes the following. Each line is still equivalent to the other lines.
```js
1 |> # + 2 |> # * 3

// Static rewriting
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
  |> doubleSay // a tacit unary function call
  |> capitalize // also a tacit unary function call
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

For each pipe expression, evaluated left associatively and inside to outside, the steps of the computation would be:

1. The topic expression is first evaluated in the current lexical context.
2. The topic’s result is bound to a hidden special variable `•`.
3. In a new inner lexical context (the body scope), the value of `•` is bound to the topic variable `#`.
4. The pipe’s body expression is evaluated within this inner lexical context.
5. The pipe’s result is the result of the body.

### Bidirectional associativity
The pipe operator is presented above as a left-associative operator. However, it is theoretically [bidirectionally associative][36]: how a pipeline’s expressions are particularly grouped is functionally arbitrary. One could force right associativity by parenthesizing a pipeline, such that it itself becomes the body of another, outer pipeline.

Consider the above example `1 |> # + 2 |> # * 3`, whose syntax was statically rewritten using left associativity and autogenerated [hygienic variables][37].
```js
// With left associativity and autogenerated hygienic variables.
1 |> # + 2 |> # * 3

// Static rewriting
(1 |> # + 2) |> # * 3
do { const #₀ = 1; # + 2 } |> # * 3
do { const #₁ = do { const #₀ = 1; # + 2 }; #₁ * 3 }

// Runtime evaluation
do { const #₀ = do { 1 + 2 }; #₀ * 3 }
do { const #₀ = 3; #₀ * 3 }
do { do { 3 * 3 } }
9
```

But if right associativity is forced with `1 |> (# + 2 |> # * 3)`, then the result would be the same: `9`:
```js
// With right associativity and autogenerated hygienic variables.
1 |> # + 2 |> # * 3

// Static rewriting
1 |> (# + 2 |> # * 3)
1 |> do { const #₀ = # + 2; #₀ * 3 }
do { const #₁ = 1; do { const #₀ = #₁ + 2; #₀ * 3 } }

// Runtime evaluation
do { do { const #₀ = 1 + 2; #₀ * 3 } }
do { do { const #₀ = 3; #₀ * 3 } }
do { do { 3 * 3 } }
9
```

Similarly, `1 |> # + 2 |> # * 3` was also statically rewritten using a different method: under left associativity and a single dummy variable.
```js
// With left associativity and single dummy variable.
1 |> # + 2 |> # * 3

// Static rewriting
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

If right associativity is forced with `1 |> (# + 2 |> # * 3)`, then the result would be the same: `9`:
```js
// With right associativity and single dummy variable.
1 |> # + 2 |> # * 3

// Static rewriting
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

## Static semantics
[TO DO: Brief explanation of static semantics]

### Early errors

### Contains?
[TO DO]

### Is function definition?
[TO DO]

### Is valid simple assignment target?
[TO DO]

### Is

## Runtime semantics
[TO DO]

## Alternative solutions explored
There are a number of other ways of potentially accomplishing the above use cases. However, the authors of this proposal believe that the smart pipe operator may be the best choice. [TO DO]

## Relation to existing work
[TO DO]

[TO DO: Include commentary on why “topic variable” instead of “placeholder”—because verbally confusing with partial-application placeholders—and because forward compatibility with extending the topic concept to other syntax, such as `for { … }`, `matches { … }`, and `given (…) { … }`.]

[TC39 process]: https://tc39.github.io/process-document/
[Hack pipe]: https://docs.hhvm.com/hack/operators/pipe-operator
[F# pipe]: https://docs.microsoft.com/en-us/dotnet/fsharp/language-reference/functions/index#function-composition-and-pipelining
[OCaml pipe]: http://blog.shaynefletcher.org/2013/12/pipelining-with-operator-in-ocaml.html
[Elixir pipe]: https://elixir-lang.org/getting-started/enumerables-and-streams.html
[6]: http://elm-lang.org/docs/syntax#infix-operators
[7]: https://docs.julialang.org/en/stable/stdlib/base/#Base.:|>
[8]: http://livescript.net/#operators-piping
[9]: https://cran.r-project.org/web/packages/magrittr/index.html
[10]: https://docs.perl6.org/language/operators#infix_==&gt;
[11]: https://en.wikipedia.org/wiki/Pipeline_(Unix
[12]: https://streams.spec.whatwg.org/#pipe-chains
[13]: https://nodejs.org/api/stream.html#stream_readable_pipe_destination_options
[14]: https://github.com/tc39/proposal-pipeline-operator
[15]: https://github.com/littledan
[16]: https://github.com/tc39/proposal-pipeline-operator/wiki#proposal-4-smart-mix
[17]: https://github.com/tc39/proposal-pipeline-operator/issues?q=placeholder
[18]: https://github.com/tc39/proposal-pipeline-operator/issues/89#issuecomment-363853394
[19]: https://github.com/tc39/proposal-dynamic-import/issues
[21]: http://underscorejs.org
[22]: https://github.com/sindresorhus/pify
[23]: https://fetch.spec.whatwg.org
[24]: https://en.wikipedia.org/wiki/Tacit_programming
[nomenclature]: #nomenclature
[pipeline syntax]: #pipeline-syntax
[es-grammar]: https://tc39.github.io/ecma262/#sec-syntactic-and-lexical-grammars
[es-lexical-grammar]: https://tc39.github.io/ecma262/#sec-ecmascript-language-lexical-grammar
[es-punctuators]: https://tc39.github.io/ecma262/#sec-punctuators
[in relational operator]: https://tc39.github.io/ecma262/#sec-relational-operators
[for iteration statements]: https://tc39.github.io/ecma262/#sec-iteration-statements
[assignment operator]: https://tc39.github.io/ecma262/#sec-assignment-operators
[25]: https://en.wikipedia.org/wiki/Garden_path_sentence
[26]: #smart-body-syntax
[28]: #topic-variable-semantics-by-replacing-them-with-autogenerated-variables
[29]: https://en.wikipedia.org/wiki/Hygienic_macro
[30]: #topic-variable-semantics-by-alternating-lexical-shadowing-with-dummy-variable
[31]: https://en.wikipedia.org/wiki/Hygienic_macro
[32]: https://en.wikipedia.org/wiki/Hygienic_macro
[bidirectional associativity]: #bidirectional-associativity
[34]: https://en.wikipedia.org/wiki/Hygienic_macro
[35]: https://en.wikipedia.org/wiki/Hygienic_macro
[36]: https://en.wikipedia.org/wiki/Associative_property
[37]: https://en.wikipedia.org/wiki/Hygienic_macro
[MDN’s guide on expressions and operators]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Expressions_and_Operators
[formal grammar]: #grammar
[Clojure pipe]: https://clojuredocs.org/clojure.core/as-%3E
[property-chained tacit style]: #property-chained-tacit-style
[smart body syntax]: #smart-body-syntax
