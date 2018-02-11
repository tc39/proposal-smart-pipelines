# Smart pipelines
ECMAScript Stage-−1 Proposal by J. S. Choi, 2018-02.

This repository contains the formal specification for a proposed “smart pipeline operator” `|>` in JavaScript. It is currently not even in Stage 0 of the [TC39 process](https://tc39.github.io/process-document/) but it may eventually be presented to TC39.

## Background
The operator is a backwards-compatible way of chaining nested expressions in a readable, left-to-right manner. Nested transformations become untangled into short steps. It is similar to [Hack’s `|>` and `$$`](https://docs.hhvm.com/hack/operators/pipe-operator), [F#’s `|>`](https://docs.microsoft.com/en-us/dotnet/fsharp/language-reference/functions/index#function-composition-and-pipelining), [OCaml’s `|>`](http://blog.shaynefletcher.org/2013/12/pipelining-with-operator-in-ocaml.html), [Elixir/Erlang’s `|>`](https://elixir-lang.org/getting-started/enumerables-and-streams.html), [Elm’s `|>`](http://elm-lang.org/docs/syntax#infix-operators), [Julia’s `|>`](https://docs.julialang.org/en/stable/stdlib/base/#Base.:|>), [LiveScript’s `|>`](http://livescript.net/#operators-piping), and [Unix’s `|`](https://en.wikipedia.org/wiki/Pipeline_(Unix)).

The proposal is a variant of a [previous pipeline-operator proposal](https://github.com/tc39/proposal-pipeline-operator) championed by [Daniel Ehrenberg of Igalia](https://github.com/littledan). This variant is listed as [Proposal 4: Smart Mix on the pipe-proposal wiki](https://github.com/tc39/proposal-pipeline-operator/wiki#proposal-4-smart-mix). The variant resulted from [previous discussions about pipeline placeholders in the previous pipe-operator proposal](https://github.com/tc39/proposal-pipeline-operator/issues?q=placeholder), which culminated in an [invitation by Daniel Ehrenberg, champion on the current pipe proposal, to try writing a specification draft](https://github.com/tc39/proposal-pipeline-operator/issues/89#issuecomment-363853394). A prototype Babel plugin is also pending.

You can take part in the discussions on the [GitHub issue tracker](https://github.com/tc39/proposal-dynamic-import/issues). When you file an issue, please note in it that you are talking specifically about “Proposal 4: Smart Mix”.

## Motivation
Let these two functions be defined:

```js
function doubleSay (string) {
  return `${string}, ${string}`
}

function capitalize (string) {
  return string[0].toUpperCase() + string.substring(1)
}
```

This nested expression is quite messy. It is spaghetti that requires many levels of indentation. Reading this code requires checking both the left and right of each subexpression to understand the flow of data.

```js
capitalizedString(
  doubledString(
    (await stringPromise)
      ?? throw new TypeError(`Expected string from ${stringPromise}`)
  )
) + '!'
```

## Proposed solution
The code above would become much terser with pipes, making both reading and writing considerably easier.

```js
stringPromise
  |> await #
  |> # ?? throw new TypeError()
  |> doubleSay // a tacit unary function call
  |> capitalize // also a tacit unary function call
  |> # + '!'
```

This same use case appears numerous times in JavaScript code, whenever any value is transformed by expressions of any type: function calls, property calls, method calls, object constructions, arithmetic operations, logical operations, bitwise operations, `typeof`, `instanceof`, `await`, `yield` and `yield *`, and `throw` expressions.

## Syntactic sugar for nested `do`s
The example above is roughly equivalent to nested `do` expressions, each with a variable binding and a transformation of that variable’s value.

```js
do {
  const # = do {
    const # = do {
      const # = do {
        const # = await stringPromise;
        # ?? throw new TypeError()
      };
      doubleSay(#)
    };
    capitalize(#)
  };
  # + '!'
}
```

1. For each pipe operation, the left-hand side (LHS) is evaluated.
2. Then it is bound to a placeholder `#`: a nullary variable that acts as a variable. This variable is only defined within the pipe’s right-hand side (RHS), which acts as an inner lexical scope.
3. With this binding, the RHS is then evaluated too.

## Tacit unary function calls
As a further abbreviation, you may omit the placeholder if the operation is just a call of a named unary function. This is called [“tacit” or “point-free style”](https://en.wikipedia.org/wiki/Tacit_programming). This is the “smart” part of these “smart pipeline operators”.

If the RHS expression is merely an identifier (as with `… |> doubleSay` or `… |> capitalize`), possibly in a property chain (<i lang=lt>e.g.</i>, `… |> Symbol.for`), and possibly with a `new` operator (<i lang=lt>e.g.</i>, `… |> new global.Date`), then that identifier is assumed to be a unary function or unary constructor, which is then called with `#` (<i lang=lt>i.e.</i>, `… |> doubleSay(#)` or `… |> capitalize(#)`). For example:

```js
'hello'
  |> await #
  |> # ?? throw new TypeError(`Expected string from ${#}`)
  |> doubleSay
  |> capitalize
  |> # + '!'
```

…is equivalent to:
```js
'hello'
  |> await #
  |> # ?? throw new TypeError(`Expected string from ${#}`)
  |> doubleSay(#)
  |> capitalize(#)
  |> # + '!'
```

Tacit style is not permitted with expressions more complex than single identifiers or simple property chain with no method calls.

If a pipe RHS has *no* placeholder, then it must be a permitted tacit unary function (single identifier or simple property chain). Otherwise, it is a syntax error. In particular, tacit function calls *never* have parentheses. If they need to have parentheses, then they need to have a placeholder.

| Expression | Result |
| --- | --- |
| **`'2018' \|> Date(#)`** | **`Date('2018')`** |
| **`'2018' \|> Date`** | **`Date('2018')`** |
| `'2018' \|> Date()` | syntax error; missing `#` |
| `'2018' \|> (Date)` | syntax error; missing `#` |
| **`'2018' \|> Date.parse(#)`** | **`Date.parse('2018')`** |
| **`'2018' \|> Date.parse`** | **`Date.parse('2018')`** |
| `'2018' \|> Date.parse()` | syntax error; missing `#` |
| `'2018' \|> (Date.parse)` | syntax error; missing `#` |
| **`'2018' \|> global.Date.parse(#)`** | **`global.Date.parse('2018')`** |
| **`'2018' \|> global.Date.parse`** | **`global.Date.parse('2018')`** |
| `'2018' \|> global.Date.parse()` | syntax error; missing `#` |
| `'2018' \|> (global.Date.parse)` | syntax error; missing `#` |
| **`'2018' \|> new global.Date(#)`** | **`new Date('2018')`** |
| **`'2018' \|> new global.Date`** | **`new Date('2018')`** |
| `'2018' \|> new global.Date()` | syntax error; missing `#` |
| `'2018' \|> (new global.Date)` | syntax error; missing `#` |

This rule minimizes the parsing lookahead that the compiler must check before it can distinguish between tacit style and placeholder style. By restricting the space of valid tacit RHS expressions without placeholders, the rule prevents [garden-path syntax](https://en.wikipedia.org/wiki/Garden_path_sentence) that would otherwise be possible: <i lang=lt>e.g.</i>, `… |> compose(f, g, h, i, j, k, #)`.

The rule also statically prevents a writing JavaScript programmer from accidentally omitting a placeholder where they meant to put one. For instance, if `x |> 3` were not a syntax error, then it would be a useless operation and almost certainly not what the writer intended. The JavaScript programmer is encouraged to avoid using tacit style where its meaning may be visually confusing to the reader.

## Multiple placeholders in RHS
The placeholder may be used multiple times in the RHS, but each use refers to the same value. Because it is bound to the result of the LHS, the LHS is still only ever evaluated once.

```js
… |> f(#, #)
// equivalent to:
// do { const # = …; f(#, #) }
```

```js
… |> [#, # * 2, # * 3]
// equivalent to:
// do { const # = …; [#, # * 2, # * 3] }
```

## Nested inner lexical contexts
Inner lexical contexts in general are allowed in both sides of the pipe operator. [To do]

## RHS-nested pipe placeholders
The pipe operator is left associative. However, it is still possible to force right associativity by putting a pipeline in another… [TO DO]

## Associativity and precedence
The pipe operation is left associative, as its LHS-then-RHS evaluation order requires.

Its precedence is quite loose. It is tighter than assignment (`=`, `+=`, …), generator `yield` and `yield *`, and sequence `,`; and it is looser than logical ternary conditional (`… ? … : …`), logical and/or `&&`/`||`, bitwise and/or/xor, `&`/`|`/`^`, equality/inequality `==`/`===`/`!=`/`!==`, and every other type of expression.

Being any tighter than this level would require its RHS to be parenthesized for many frequent types of expressions. However, the result of a pipeline is also expected to often serve as the RHS of a variable assignment `=`.

## Alternative solutions explored
There are a number of other ways of potentially accomplishing the above use cases. However, the authors of this proposal believe that the smart pipe operator may be the best choice. [TO DO]

## Relation to existing work
[TO DO]


