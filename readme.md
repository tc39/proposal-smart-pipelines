# Smart pipelines
ECMAScript Stage-−1 Proposal by J. S. Choi, 2018-02.

This repository contains the formal specification for a proposed “smart pipe operator” `|>` in JavaScript. It is currently not even in Stage 0 of the [TC39 process](https://tc39.github.io/process-document/) but it may eventually be presented to TC39.

## Background
The binary operator proposed here would provide a backwards- and forwards-compatible style of chaining nested expressions into a readable, left-to-right manner. Nested transformations become untangled into short steps in a zero-cost abstraction. The operator is similar to [Hack’s `|>` and `$$`](https://docs.hhvm.com/hack/operators/pipe-operator), [F#’s `|>`](https://docs.microsoft.com/en-us/dotnet/fsharp/language-reference/functions/index#function-composition-and-pipelining), [OCaml’s `|>`](http://blog.shaynefletcher.org/2013/12/pipelining-with-operator-in-ocaml.html), [Elixir/Erlang’s `|>`](https://elixir-lang.org/getting-started/enumerables-and-streams.html), [Elm’s `|>`](http://elm-lang.org/docs/syntax#infix-operators), [Julia’s `|>`](https://docs.julialang.org/en/stable/stdlib/base/#Base.:|>), [LiveScript’s `|>`](http://livescript.net/#operators-piping), [R / magrittr’s `%>%`](https://cran.r-project.org/web/packages/magrittr/index.html), [Perl 6’s `==>`](https://docs.perl6.org/language/operators#infix_==&gt;) and [Unix shells’ and PowerShell’s `|`](https://en.wikipedia.org/wiki/Pipeline_(Unix)). It is conceptually similar but should not be confused with [WHATWG-stream piping](https://streams.spec.whatwg.org/#pipe-chains) and [Node-stream piping](https://nodejs.org/api/stream.html#stream_readable_pipe_destination_options).

The proposal is a variant of a [previous pipe-operator proposal](https://github.com/tc39/proposal-pipeline-operator) championed by [Daniel Ehrenberg of Igalia](https://github.com/littledan). This variant is listed as [Proposal 4: Smart Mix on the pipe-proposal wiki](https://github.com/tc39/proposal-pipeline-operator/wiki#proposal-4-smart-mix). The variant resulted from [previous discussions about pipeline placeholders in the previous pipe-operator proposal](https://github.com/tc39/proposal-pipeline-operator/issues?q=placeholder), which culminated in an [invitation by Daniel Ehrenberg, champion on the current pipe proposal, to try writing a specification draft](https://github.com/tc39/proposal-pipeline-operator/issues/89#issuecomment-363853394). A prototype Babel plugin is also pending.

You can take part in the discussions on the [GitHub issue tracker](https://github.com/tc39/proposal-dynamic-import/issues). When you file an issue, please note in it that you are talking specifically about “Proposal 4: Smart Mix”.

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

Being able to automatically detect this tacit style is the “smart” part of these “smart pipeline operators”; for more information, see the [“smart RHS syntax” section](#smart-rhs-syntax).

Now following are various examples adapted from useful, real-world code.

### Underscore.js
Adapted from [Underscore.js 1.8.3](http://underscorejs.org) by Jeremy Ashkenas et al. under MIT License.
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

_.size = function(obj) {
  if (obj == null) return 0;
  return obj |> isArrayLike
    ? obj.length
    : obj |> _.keys |> #.length;
};
```

### Pify
Adapted from [Pify 3.0.0](https://github.com/sindresorhus/pify) by Sindre Sorhus under MIT License.
```js
pify(fs.readFile)('package.json', 'utf8').then(data => {
  console.log(JSON.parse(data).name)
})

console.log(JSON.parse((await pify(fs.readFile)('package.json', 'utf8')).name));

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

return opts.include
  ? opts.include.some(match)
  : opts.exclude.some(match) |> !#
```

### Fetch Standard
Adapted from [Fetch, the living web standard](https://fetch.spec.whatwg.org) by WHATWG under Creative Commons BY.
```js
fetch("/music/pk/altes-kamuffel.flac")
  .then(res => res.blob())
  .then(playBlob)

playBlob(await (await fetch("/music/pk/altes-kamuffel.flac")).blob())

"/music/pk/altes-kamuffel.flac"
  |> await fetch(#)
  |> await #.blob()
  |> playBlob
```
***
```js
fetch("/", { method: "HEAD" })
  .then(res => log(res.headers.get("strict-transport-security")))

(await fetch("/", { method: "HEAD" }))
  .headers.get("strict-transport-security")

"/"
  |> await fetch(#, { method: "HEAD" })
  |> #.headers.get("strict-transport-security")
```
***
```js
fetch("https://pk.example/berlin-calling.json", { mode: "cors" })
  .then(res => {
    if (res.headers.get("content-type")??.toLowerCase()
      .indexOf("application/json") >= 0
    ) {
      return res.json()
    } else {
      throw new TypeError()
    }
  }).then(processJSON)

do {
  const res = await fetch("https://pk.example/berlin-calling.json", {
    mode: "cors"
  });
  processJSON(await do {
    if (res.headers.get("content-type")??.toLowerCase()
      .indexOf("application/json") >= 0
    ) {
      res.json()
    } else {
      throw new TypeError()
    }
  })
}

"https://pk.example/berlin-calling.json"
  |> await fetch(#, { mode: "cors" })
  |>
    #.headers.get("content-type")
      ??.toLowerCase().indexOf("application/json")
      >= 0
    ? #.json()
    : throw new TypeError()
  |> processJSON
```

## Nomenclature
The binary operator itself may be referred to as a <dfn>pipe</dfn>, a <dfn>pipe operator</dfn>, or a <dfn>pipeline operator</dfn>; all these names are equivalent. This specification will prefer the term “pipe operator”.

A pipe operator between two expressions forms a <dfn>pipe expression</dfn>. One or more pipe expressions in a chain form a <dfn>pipeline</dfn>. For each pipe expression, the expression before the pipe is the pipeline’s <dfn>left-hand side (LHS)</dfn>; the expression after the pipe is its <dfn>right-hand side (RHS)</dfn>. The pipe operator is said <dfn>to pipeline</dfn> the LHS’s value <dfn>through</dfn> the RHS expression, where “pipeline” here is used as a verb.

The special token `#` is a <dfn>topic variable</dfn>: it is a nullary operator that acts as a special variable. The topic variable is *not* an identifier and cannot be manually declared (`const #` is a syntax error), nor can it be assigned with a value (`# = 3` is a syntax error). Instead, the topic variable may be implicitly, lexically bound using a pipeline. When a pipeline’s RHS is in <dnf>topic-variable style</dfn>, the RHS forms an inner lexical scope—called the pipeline’s <dfn>RHS scope</dfn>—within which the topic variable is implicitly bound to the value of the LHS, acting as a **placeholder** for the LHS’s value.

Alternatively, you may omit the RHS’s topic variables if the RHS is just a simple reference to a function or constructor, such as with `… |> capitalize` and `… |> new User.Message` from the preceding example. The RHS would then be called as a unary function or constructor, without having to use the topic variable as an explicit argument. This is called [<dfn>tacit style</dfn> or <dfn>point-free style</dfn>](https://en.wikipedia.org/wiki/Tacit_programming). When a pipe is in tacit style, the RHS is called a <dfn>tacit function</dfn> or a <dfn>tacit constructor</dfn>, depending on if it starts with `new`.

## Syntax

### Tokens
The rule for <var>Punctuator</var> tokens would be modified: two new tokens, `|>` and `#`, would be added to the Punctuators:
```
Punctuator :: one of
  `{` `(` `)` `[` `]` `.` `...` `;` `,` `<` `>` `<=` `>=` `==` `!=` `===` `!==`
  `+` `-` `*` `%` `**` `++` `--` `<<` `>>` `>>>` `&` `|` `^` `!` `~` `&&` `||`
  `?` `:` `|>` `#`
  `=` `+=` `-=` `*=` `%=` `**=` `<<=` `>>=` `>>>=` `&=` `|=` `^=` `=>`
```

### Pipe expressions
```
***
```

### Loose precedence
The pipe operator’s precedence is quite loose. It is tighter than assignment (`=`, `+=`, …), generator `yield` and `yield *`, and sequence `,`; and it is looser than logical ternary conditional (`… ? … : …`), logical and/or `&&`/`||`, bitwise and/or/xor, `&`/`|`/`^`, equality/inequality `==`/`===`/`!=`/`!==`, and every other type of expression.

Being any tighter than this level would require its RHS to be parenthesized for many frequent types of expressions. However, the result of a pipeline is also expected to often serve as the RHS of a variable assignment `=`.

Thus, [TO DO: Add to expression rule]

### Smart RHS syntax
When the parser checks the RHS, it must determine what style it is in. There are four outcomes: tacit constructor, tacit function, parameterized expression, or invalid RHS.

The goal here is to minimize the parsing lookahead that the compiler must check before it can distinguish between tacit style and topic-token style. By restricting the space of valid tacit RHS expressions without topic variables, the rule prevents [garden-path syntax](https://en.wikipedia.org/wiki/Garden_path_sentence) that would otherwise be possible: such as `… |> compose(f, g, h, i, j, k, #)`.

Another goal is to statically prevents a writing JavaScript programmer from accidentally omitting a topic variable where they meant to put one. For instance, if `x |> 3` were not a syntax error, then it would be a useless operation and almost certainly not what the writer intended. The JavaScript programmer is encouraged to use topic variables and avoid tacit style, where tacit style may be visually confusing to the reader.

The parsing rules are such that:

1. If the RHS is a mere identifier, optionally with a chain of properties, and with no parentheses or brackets, then that identifier is assumed to be a **tacit function**.

2. If the RHS starts with `new`, followed by mere identifier, optionally with a chain of properties, and with no parentheses or brackets, then that identifier is assumed to be a **tacit constructor**.

3. Otherwise, the RHS is now an arbitrary expression. Because the expression is not in tacit style, it must use that pipe’s topic variable. ([TO DO: Formalize this static semantic as `usesTopic()`.] Topic variables from the RHS scopes of other, inner pipelines do not count.) If there is no such topic variable in the expression, then a syntax error is thrown.

If a pipe RHS *never* uses topic variable, then it must be a permitted tacit unary function (single identifier or simple property chain). Otherwise, it is a syntax error. In particular, tacit style *never* uses parentheses. If they need to have parentheses, then they need to have use the topic variable. The following table summarizes these rules.

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

### Multiple topic variables in a pipeline’s RHS
The topic variable may be used multiple times in a pipeline’s RHS. Each use refers to the same value (wherever the topic variable is not overridden by another, inner pipeline’s RHS scope). Because it is bound to the result of the LHS, the LHS is still only ever evaluated once. The lines in each of the following are equivalent:
```js
… |> f(#, #)
do { const $ = …; f($, $) }
```
***
```js
… |> [#, # * 2, # * 3]
do { const $ = …; [$, $ * 2, $ * 3] }
```

### Inner functions
Both the LHS and the RHS of a pipeline may contain nested inner functions. This also works as expected. The lines in the following are equivalent:
```js
… |> settimeout(() => # * 500)
do { const $ = …; settimeout(() => # * 500) }
```

### Deeply nested pipelines
[TO DO]

### General static semantics
A pipe expression’s semantics are:

1. The LHS is evaluated into the LHS’s value; call this `lhsValue`.

2. [The RHS is tested for its type](#smart-rhs-syntax): Is it in tacit style (as a tacit function or a tacit constructor), is it in topic-variable style, or is it an invalid RHS?

  * If the RHS is a tacit function (such as `f` and `M.f`):
    1. The RHS is evaluated (in the current lexical context); call this value the `rhsFunction`.
    2. The entire pipeline’s value is `rhsFunction(lhsValue)`.

  * If the RHS is a tacit constructor (such as `new C` and `new M.C`):
    1. The portion of the RHS after `new` is evaluated (in the current lexical context); call this value the `RHSConstructor`.
    2. The entire pipeline’s value is `new RHSConstructor(lhsValue)`.

  * If the RHS is in topic-variable style (such as `f(#, n)`, `o[s](#)`, `# + n`, and `#.p`):
    1. An inner lexical context is entered.
    2. Within this inner context, `lhsValue` is bound to a variable.
    3. Within this inner context, with the variable binding to `lhsValue`, the RHS is evaluated; call the resulting value `rhsValue`.
    4. The entire pipeline’s value is `rhsValue`.

  * Otherwise, if the RHS is an invalid RHS (such as `f()`, `f(n)`, `o[s]`, `+ n`, and `n`), then throw a syntax error explaining that the pipeline’s topic variable is missing from its RHS.

The topic-variable expression case can be further explained with a nested `do` expression. There are two ways to illustrate this equivalency. The first way is to [replace each pipe expression’s topic variables with an autogenerated variable](#topic-variable-semantics-by-replacing-them-with-autogenerated-variables), which must be guaranteed to be [lexically hygienic](https://en.wikipedia.org/wiki/Hygienic_macro) and to not conflict with other variables. The alternative way is to [use two variables—the topic variable `#` and a single dummy variable](#topic-variable-semantics-by-alternating-lexical-shadowing-with-dummy-variable)—which also preserves [lexical hygiene](https://en.wikipedia.org/wiki/Hygienic_macro).

#### Topic-variable semantics by replacing them with autogenerated variables
The first way to illustrate the operator’s semantics is to replace each pipe expression’s topic variables with an autogenerated variable, which must be guaranteed to not conflict with other variables.

Let us say that each pipe expression autogenerates a new, [lexically hygienic](https://en.wikipedia.org/wiki/Hygienic_macro) variable (`#₀`, `#₁`, `#₂`, `#₃`, …), which in turn replaces each use of the topic variable `#` in each pipe’s RHS. (These `#ₙ` variables are not true syntax; it is merely for illustrative purposes. You cannot actually assign or use `#ₙ` variables.) Let us also group the expressions with left associativity (although this is arbitrary, because [right associativity would also work](#bidirectional-associativity)).

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

In general, for each pipe expression `lhs |> rhs`, assuming that `rhs` is in topic-variable style, that is, assuming that `rhs` contains an unshadowed topic variable:

* Let `#ₙ` be a [hygienically autogenerated](https://en.wikipedia.org/wiki/Hygienic_macro) topic variable, `#ₙ`, where <var>n</var> is a number that would not conflict with the name of any other autogenerated topic variable in the scope of the entire pipe expression.
* Also let `rhsSubstituted` be `rhs` but with all instances of `#` replaced with `#ₙ`.
* Then the static syntax rewriting (left associative and inside to outside) would simply be: `do { const #ₙ = lhs; rhsSubstituted }`. This `do` expression would act as at the RHS scope.

#### Topic-variable semantics by alternating lexical shadowing with dummy variable
The other way to demonstrate topic-variable style is to use two variables: the topic variable `#` and single [lexically hygienic](https://en.wikipedia.org/wiki/Hygienic_macro) dummy variable `•`. It should be noted that `const # = …` is not a valid statement under this proposal’s actual syntax; likewise, `•` is not a part of the proposal’s syntax. Both forms are for illustrative purposes here only.

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

1. The LHS expression is first evaluated in the current lexical context.
2. The LHS’s result is bound to a hidden special variable `•`.
3. In a new inner lexical context (the RHS scope), the value of `•` is bound to the topic variable `#`.
4. The pipe’s RHS expression is evaluated within this inner lexical context.
5. The pipe’s result is the result of the RHS.

### Bidirectional associativity
The pipe operator is presented above as a left-associative operator. However, it is theoretically [bidirectionally associative](https://en.wikipedia.org/wiki/Associative_property): how a pipeline’s expressions are particularly grouped is functionally arbitrary. One could force right associativity by parenthesizing a pipeline, such that it itself becomes the RHS of another, outer pipeline.

Consider the above example `1 |> # + 2 |> # * 3`, whose syntax was statically rewritten using left associativity and autogenerated [hygienic variables](https://en.wikipedia.org/wiki/Hygienic_macro).
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

## Runtime semantics

[TO DO]

## Alternative solutions explored
There are a number of other ways of potentially accomplishing the above use cases. However, the authors of this proposal believe that the smart pipe operator may be the best choice. [TO DO]

## Relation to existing work
[TO DO]

[TO DO: Include commentary on why “topic variable” instead of “placeholder”—because verbally confusing with partial-application placeholders—and because forward compatibility with extending the topic concept to other syntax, such as `for { … }`, `matches { … }`, and `given (…) { … }`.]

