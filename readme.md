<details open>
<summary>Table of Contents</summary>

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Smart pipelines](#smart-pipelines)
  - [Background](#background)
  - [Motivation](#motivation)
    - [Goals](#goals)
      - [“Don’t break my code.”](#dont-break-my-code)
        - [Backward compatibility](#backward-compatibility)
        - [Zero runtime cost](#zero-runtime-cost)
        - [Forward compatibility](#forward-compatibility)
      - [“Don’t make me overthink.”](#dont-make-me-overthink)
        - [Syntactic locality](#syntactic-locality)
        - [Cyclomatic simplicity](#cyclomatic-simplicity)
      - [“Don’t shoot me in the foot.”](#dont-shoot-me-in-the-foot)
        - [Simple scoping](#simple-scoping)
        - [Static analyzability](#static-analyzability)
      - [“Make my code easier to read.”](#make-my-code-easier-to-read)
        - [Untangled flow](#untangled-flow)
      - [Expressive versatility](#expressive-versatility)
        - [Distinguishable punctuators](#distinguishable-punctuators)
          - [Terse parentheses](#terse-parentheses)
          - [Terse variables](#terse-variables)
          - [Terse function calls](#terse-function-calls)
      - [Other Goals](#other-goals)
        - [Conceptual generality](#conceptual-generality)
        - [Human writability](#human-writability)
        - [Novice learnability](#novice-learnability)
    - [Real-world examples](#real-world-examples)
      - [Prior pipeline proposal](#prior-pipeline-proposal)
      - [Underscore.js](#underscorejs)
      - [Pify](#pify)
      - [Fetch Web Standard](#fetch-web-standard)
  - [Nomenclature](#nomenclature)
    - [Pipe operator, pipeline, pipeline-level expression](#pipe-operator-pipeline-pipeline-level-expression)
    - [Head, head value, body, pipeline value, topical style, bare style](#head-head-value-body-pipeline-value-topical-style-bare-style)
    - [Topic, topic reference](#topic-topic-reference)
    - [Explanation of conventions](#explanation-of-conventions)
  - [Grammar](#grammar)
    - [Lexical grammar](#lexical-grammar)
    - [Grammar parameters](#grammar-parameters)
    - [Syntax and static semantics](#syntax-and-static-semantics)
      - [Static “Contains?”](#static-contains)
      - [Static “Is Function Definition?”](#static-is-function-definition)
      - [Static “Is Identifier Reference?”](#static-is-identifier-reference)
      - [Static “Is Valid Simple Assignment Target?”](#static-is-valid-simple-assignment-target)
      - [Static “Uses Outer Topic?”](#static-uses-outer-topic)
      - [Static Early Errors](#static-early-errors)
    - [Operator precedence](#operator-precedence)
    - [Topic reference • Syntax grammar](#topic-reference-%E2%80%A2-syntax-grammar)
    - [Topic reference • Static semantics](#topic-reference-%E2%80%A2-static-semantics)
    - [Topic reference • Runtime semantics](#topic-reference-%E2%80%A2-runtime-semantics)
    - [Pipeline-level expressions • Syntax grammar](#pipeline-level-expressions-%E2%80%A2-syntax-grammar)
    - [Pipeline-level expressions • Static semantics](#pipeline-level-expressions-%E2%80%A2-static-semantics)
    - [Pipeline-level expressions • Runtime semantics](#pipeline-level-expressions-%E2%80%A2-runtime-semantics)
    - [Smart body syntax](#smart-body-syntax)
      - [Bare style](#bare-style)
        - [Simple reference](#simple-reference)
        - [Bare function call](#bare-function-call)
        - [Bare constructor call](#bare-constructor-call)
      - [Topical style](#topical-style)
    - [Topic resolution](#topic-resolution)
      - [Lexical Environments](#lexical-environments)
      - [Abstract operations](#abstract-operations)
    - [Multiple topic references and inner functions](#multiple-topic-references-and-inner-functions)
    - [Inner blocks](#inner-blocks)
    - [Nested pipelines](#nested-pipelines)
  - [Relations to other work](#relations-to-other-work)
    - [Other ECMAScript proposals](#other-ecmascript-proposals)
    - [Possible future extensions to the topic concept](#possible-future-extensions-to-the-topic-concept)
      - [Headless property access](#headless-property-access)
      - [Headless pipelining](#headless-pipelining)
      - [Topical `for` loop](#topical-for-loop)
      - [Topical `for`–`await` loop](#topical-forawait-loop)
      - [Topical function / method definition](#topical-function--method-definition)
      - [Topical block parameter](#topical-block-parameter)
      - [Topical thin-arrow function](#topical-thin-arrow-function)
      - [Topical pattern matching](#topical-pattern-matching)
      - [Tacit pattern matching](#tacit-pattern-matching)
      - [Tacit error capture](#tacit-error-capture)
      - [Topical metaprogramming reference](#topical-metaprogramming-reference)
    - [Alternative solutions explored](#alternative-solutions-explored)
  - [Term rewriting](#term-rewriting)
    - [Term rewriting topical style](#term-rewriting-topical-style)
      - [Term rewriting with autogenerated variables](#term-rewriting-with-autogenerated-variables)
      - [Term rewriting with single dummy variable](#term-rewriting-with-single-dummy-variable)
    - [Bidirectional associativity](#bidirectional-associativity)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->
</details>

# Smart pipelines
ECMAScript Stage-(−1) Proposal by J. S. Choi, 2018-02.

This repository contains the formal specification for a proposed “smart pipe
operator” `|>` in JavaScript. It is currently not even in Stage 0 of the [TC39
process][TC39 process] but it may eventually be presented to TC39.

## Background
<details open>
<summary>The concept of a pipe operator appears in numerous other languages,
variously called “pipeline”, “threading”, and “feed” operators. This is because
developers find the concept useful.</summary>

* [Clojure’s `->` and `as->`][Clojure pipe]
* [Forth’s and Joy’s, then Factor’s, Onyx’s, PostScript’s, and RPL’s term
  concatenation][concatenative programming]
* [Elixir / Erlang’s `|>`][Elixir pipe]
* [Elm’s `|>`][Elm pipe]
* [F#’s `|>`][F# pipe]
* [Hack’s `|>` and `$$`][Hack pipe]
* [Julia’s `|>`][Julia pipe]
* [LiveScript’s `|>`][LiveScript pipe]
* [OCaml’s `|>`][OCaml pipe]
* [Perl 6’s `==>`][Perl 6 pipe]
* [R / magrittr’s `%>%`][R pipe]
* [Unix shells’ and PowerShell’s `|` ][Unix pipe]

Pipe operators are also conceptually similar to [WHATWG-stream piping][] and
[Node-stream piping][].

</details>

****

The binary smart pipe operator proposed here would provide a backwards- and
forwards-compatible style of chaining nested expressions into a readable,
left-to-right manner. Nested transformations become untangled into short steps
in a zero-cost abstraction.

The proposal is a variant of the [first pipe-operator proposal][] championed by
[Daniel “littledan” Ehrenberg of Igalia][]. This variant is listed as
[Proposal 4: Smart Mix on the pipe-proposal wiki][]. The variant resulted from
[previous discussions about pipeline placeholders in the previous pipe-operator
proposal][previous pipeline-placeholder discussions], which culminated in an
[invitation by Ehrenberg to try writing a specification draft][littledan
invitation]. A prototype Babel plugin is also privately brewing.

You can take part in the discussions on the [GitHub issue tracker][]. When you
file an issue, please note in it that you are talking specifically about
“Proposal 4: Smart Mix”.

**This specification uses `#`** as its [“topic token”][nomenclature]. However,
this is not set in stone. In particular, **`?` and `@` could also be used**.
Either would be similarly terse and typeable.
Bikeshedding over what characters to use for the topic token is occurring on
GitHub at [tc39/proposal-pipeline-operator, issue #91][topic-token bikeshedding].

## Motivation
Nested, deeply composed expressions occur often in JavaScript. They occur
whenever any single value must be processed by a series of transformations,
whether they be operations, functions, or constructors. Unfortunately, these
deeply nested expressions often result in messy spaghetti code, due to their
mixing of prefix, infix, and postfix expressions together. Writing such code
requires many nested levels of indentation. Reading the such requires checking
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
to right through a **single flat thread of postfix expressions**, essentially
forming a [Reverse Polish notation][].
```js
stringPromise
  |> await #
  |> # ?? throw new TypeError()
  |> doubleSay(#, ', ')
  |> capitalize // a bare unary function call
  |> # + '!'
  |> new User.Message // a bare unary constructor call
```

Each such postfix expression (called a **pipeline body**) is in its own inner
lexical scope, within which a special token `#` is defined. This `#` is a
reference to the **topic** of the pipeline (`#` itself is called the **topic
reference**). When the pipeline’s head (the expression at its left-hand side) is
evaluated, it then becomes the pipeline’s topic. A new lexical environment it
created, within which `#` is bound to the topic, and within which the pipeline’s
body (the expression at its righthand side) is evaluated using that topic
binding.

whatever the pipeline head evaluated into. For instance, `5 |> # - 3 |> # * 2`
is precisely the same as `((5 - 3)) * 2`. The syntax statically is [term
rewritable into already valid code][term rewriting] with theoretically zero
runtime cost.

The resulting code’s terseness and flatness may be both easier for the
JavaScript developer to read and to edit. The reader may follow the flow of data
more easily through this single flattened thread of postfix operations. And the
developer may more easily add or remove operations at the beginning, end, or
middle of the thread, without changing the indentation of many nearby, unrelated
lines.

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
  |> new User.Message
```

Being able to automatically detect this [bare style][] is the [**smart** part
of this “smart pipe operator”][smart body syntax].

<details open>

### Goals

<summary>
There are fourteen ordered Goals that the smart body syntax tries to fulfill,
which may be summarized,
“Don’t break my code,”<br>
“Don’t make me overthink,”<br>
“Don’t shoot me in the foot,”<br>
“Make my code easier to read,”<br>
and other.
</summary>

Listed by priority, from most to least important:

- “Don’t break my code.”
   1. [Backward compatibility](#backward-compatibility)
   2. [Zero runtime cost](#zero-runtime-cost)
   3. [Forward compatibility](#forward-compatibility)

- “Don’t make me overthink.”
   4. [Syntactic locality](#syntactic-locality)
   5. [Cyclomatic simplicity](#cyclomatic-simplicity)

- “Don’t shoot me in the foot.”
   6. [Simple scoping](#simple-scoping)
   7. [Static analyzability](#static-analyzability)

- “Make my code easier to read.”
   8. [Untangled flow](#untangled-flow)
   9. [Expressive versatility](#expressive-versatility)
  10. [Distinguishability](#distinguishability)
  11. [Terse parentheses](#terse-parentheses)
  12. [Terse variables](#terse-variables)
  13. [Terse function calls](#terse-function-calls)

- Other
  14. [Conceptual generality](#conceptual-generality)
  15. [Human writability](#human-writability)
  16. [Novice learnability](#novice-learnability)

#### “Don’t break my code.”

##### Backward compatibility
The syntax must avoid stepping on the toes of existing code, including but not
limited to JavaScript libraries such as jQuery and Underscore.js. In particular,
the topic reference should not be an existing identifier such as `$` or `_`,
which both may cause surprising results to a developer who adopts pipelines
while also using a globally bound convenience variable. It is a common pattern
to do this even without a library: `var $ = document.querySelectorAll`”. The
silent shadowing of such outer-context variables may silently cause bugs, which
may also be difficult to debug (see Goal 6).

Nor can it cause previously valid code to become invalid. This includes, to a
lesser extent, common nonstandard extensions to JavaScript: for instance, using
`<>` for the topic reference would retroactively invalidate existing E4X and JSX
code.

This proposal uses `#` for its topic reference. This is compatible with all
known previous JavaScript code. `?` and `@` could be chosen instead, which are
each also backwards compatible.

##### Zero runtime cost
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

##### Forward compatibility
The syntax should not preclude other proposals: both already-proposed features,
such as [syntactic partial application][] and [private class fields][] – as well
as [possible future extensions to the topic concept][], such as topic-binding
versions of `function`, `for`, and `catch` blocks.

This proposal is forward compatible with all these proposals, in both its choice
of topic reference and in its prohibition of topic references within any block
(other than arrow functions).

Forward compatibility is elaborated in the section on [relations to other
work][]. See also Goal 9 below. See also [inner blocks in pipelines][inner
blocks].

#### “Don’t make me overthink.”
##### Syntactic locality
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
possible—such as `… |> compose(f, g, h, i, j, k, #)`. Syntax becomes more
locally readable. It becomes easier to reason about code without thinking about
code elsewhere.

##### Cyclomatic simplicity
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

#### “Don’t shoot me in the foot.”
##### Simple scoping
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

##### Static analyzability
[Early errors][] help the editing JavaScript developer avoid common [footguns][]
at compile time, such as preventing them from accidentally omitting a topic
reference where they meant to put one. For instance, if `x |> 3` were not an
error, then it would be a useless operation and almost certainly not what the
developer intended. Situations like these should be statically detectable and
cause compile-time [early errors][].

#### “Make my code easier to read.”
The new syntax should increase the human readability and writability of much
common code. It should be simpler to read and comprehend. And it should be
easier to compose and update. Otherwise, the new syntax would be useless.

Making JavaScript expressions more ergonomic for humans is the prime, original
purpose of this proposal. To a computer, the form of complex expressions –
whether as deeply nested groups or as flat threads of postfix steps – should not
matter. But to a human, it can make a significant difference.

##### Untangled flow
When a human reads deeply nested groups of expressions – which are very common
in JavaScript code – their attention must switch between the start and end of
each nested expression. And these expressions will dramatically differ in
length, depending on their level in the syntactic tree. To use the example above:
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
…the deep inner expression `await stringPromise` is relatively short. In
contrast, the shallow outer expression `` capitalize(doubledSay((await
stringPromise) ?? throw new TypeError(`Expected string from
${stringPromise}`))) + '!'`) `` is very long. Yet both are
quite similar: they are transformations of a string into another. This
insight is lost in the deeply nested noise.

With pipelines, the code forms a flat thread of postfix steps. It is much
easier for a human to read and comprehend. Each of its steps are roughly the
same length. In order to understand what occurs before a given step, one
only need to scan left, rather than in both directions as the deeply nested
tree would require. To read the whole thing, a reader may simply follow
along left to right, not back and forth.
```js
stringPromise
  |> await #
  |> # ?? throw new TypeError()
  |> doubleSay(#, ', ')
  |> capitalize
  |> # + '!'
  |> new User.Message
```

The introduction to this [motivation][] section already explained much of
the readability rationale, but it may also be useful to study the
[examples][] below.

##### Expressive versatility
JavaScript is a language rich with [expressions of numerous kinds][MDN
expressions and operators], each of which may usefully transform data from one
form to another. There is **no single type** of expression that forms a
**majority of used expressions**.

* Arithmetic operations.
* Array literals.
* Arrow functions.
* Assignment operations.
* `await` expressions.
* Class definitions.
* Conditional operations.
* Constructor calls one argument.
* Constructor calls with many n-ary arguments.
* Equality operations.
* Function calls with one unary argument.
* Function calls with many n-ary arguments.
* Function and async-function definitions.
* [Functional partial application (eventually)][Syntactic partial application].
* Generator and async-generator definitions.
* `instanceof` and `in` operations.
* Object literals.
* Property accessors and method calls.
* References to variables, `this`, and `new.target`.
* Regular-expression literals.
* `super` calls.
* Template literals.
* `typeof` operations.
* Unary function composition (eventually?).
* `yield` expressions.

The goal of the pipe operator is to untangle deeply nested expressions into flat
threads of postfix expressions. To limit it to only one type of expression, even
a common type, truncates its benefits to that one type only and compromises its
expressivity and versatility.

In particular, relying on immediately invoked function expressions ([IIFEs][])
to accomodate non-unary function is insufficient for idiomatic JavaScript code.
JavaScript functions have never fulfilled the [Tennent correspondence
principle][]. Several common types of expressions cannot be equivalently used
within inner functions, particularly `await` and `yield`. In these frequent
cases, attempting to replacing code with “equivalent” IIFEs may cause different
behavior, may cause different performance behavior (see example in Goal 2), or
may require dramatic rearrangement of logic to conserve the old code’s behavior.

It would be possible to add ad-hoc handling, for selected other expression
types, to the operator’s grammar. This would expand its benefits to that type.
However, this conflicts with Goal 5 (adding cyclomatic complexity to the parsing
process, proportional to the number of ad-hoc handled cases). It also does not
fulfill this Goal well either: excluding, perhaps arbitrarily, whatever classes
its grammar’s branches do not handle.

Such new [incidental complexity][] makes code less readable and distracts the
developer from the program’s [essential logic][essential complexity]. A pipeline
operator that improves readability should be versatile (this Goal) but
conceptually and cyclomatically simple (Goal 5). Such an operator should be able
to handle **all** expressions, in a **single** manner **uniformly**
**universally** applicable to **all** expressions. It is the hope of this
proposal’s authors that its **[topical style][]** fulfills both criteria.

##### Distinguishable punctuators
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

##### Terse parentheses
Terseness also aids distinguishability by obviating the need for boilerplate
syntactic noise. Parentheses are a prominent example: as long as operator
precedence is clear, then reducing parentheses always would JavaScript code more
visually terse and less cluttered.

The example above demonstrates how numerous verbose parentheses could become
unnecessary with pipelines. In these cases the [“data-to-ink” visual ratio][]
would significantly increase, emphasizing the program’s essential information.
The developer’s cognitive burden – of ignoring unimportant incidental symbols as
they read – has hopefully lightened.

##### Terse variables
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
>
This sort of terseness, in which the explicit is made tacit and implicit, must
be balanced with Goals 4 and 5. Excessive implicitness compromises
comprehensibility, at least without low-level tracing of tacit arguments’
invisible paths, rather than the actual, high-level meaning of the code. Yet at
the same time, excessive explicitness generates ritual, verbose boilerplate that
also interferes with reading comprehension. Therefore, Goal 10 must be balanced
with Goals 1, 4, and 5.

[The Zen of Python][PEP 20] famously says, “Explicit is better than implicit,”
but it also says, “Flat is better than nested,” and, “Sparse is better than
dense.”

##### Terse function calls
Unary function / constructor calls are a particularly frequent type of
expression and a good target for especial human optimization. However, such
extra shortening might dramatically reduce the verbosity of unary function
calls, but again this must be balanced with Goals 1, 4, and 5.

It is the hope of this proposal’s authors that its [smart body syntax][] reaches
a good balance between this Goal and Goals 4 and 5, in the same manner that
[Huffman coding][] optimizes textual symbols’ length for their frequency of use:
more commonly used symbols are shorter.

#### Other Goals
##### Conceptual generality
If a concept is uniformly generalizable to many other cases, then this
multiplies its usefulness. The more versatile its concepts, the more it may be
applied to other syntax, including existing syntax and future syntax (compare
with Goal 3).

This proposal’s concept of a **topic reference does not need to be coupled only
to pipelines**. The [topic concept is **generalizable to many syntactic
forms**][possible future extensions to the topic concept]. These generalizations
are **out of scope** of this proposal, which is only for the smart pipe
operator; they are **deferred** to [other, future proposals][possible future
extensions to the topic concept].

##### Human writability
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

Achieving Goal 8 therefore also improves the ease of composing and editing code.
By flattening deeply nested expression trees into single threads of postfix
steps, a step may be added oredited in isolation on a single line, it may be
rearranged up or down, it may be removed – all without affecting the pipeline’s
other steps in the lines above or below it.

##### Novice learnability
Learnability of the syntax is a desirable Goal: the more intuitive the syntax
is, the more rapidly it might be adopted by developers. However, learnability in
of itself is not more desirable than the other Goals above. Most JavaScript
developers would be novices to this syntax at most once, during which the
intuitiveness of the syntax will dominate their experience. But after that
honeymoon period, the syntax’s usability in workaday programming will instead
affect their reading and writing most.

So instead, readability, comprehensibility, locality, simplicity,
expressiveness, and terseness are prioritized first, where they would conflict
with learnability itself. However, a syntax that is simple but expressive – and,
most of all, readable – could well be easier to learn. Its up-front cost in
learning could be small, particularly in comparison to the large gains in
readability and comprehensibility that it might bring to code in general.

</details>

### Examples

#### Prior proposals’ examples

##### First pipe-operator proposal
[tc39/pipeline-operator-proposal][first pipe-operator proposal]. [Gilbert
“mindeavor”][mindeavor] &c. ECMA International. 2017–2018. BSD License.

<table>
<thead>
<tr>
<th>With smart pipes
<th>Status quo

<tbody>
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

stringPromise
  |> await #
  |> # ?? throw new TypeError()
  |> doubleSay(#, ', ')
  |> capitalize |> # + '!'
  |> new User.Message
```
Note that `|> capitalize` is a bare unary function call. The `#` is tacitly,
invisibly implied. `|> capitalize(#)` would work but the `#` is unnecessary.

Ditto for `|> new User.Message`, which is a bare unary constructor call,
abbreviated from `|> new User.Message(#)`.

This is the [smart part of the pipe operator][smart body syntax], which can
distinguish between a [bare style][] and the usual `#`-using [topical style][]
by using a simple rule: only identifiers, dots, and `new`; no parentheses,
brackets, braces, or other operators.

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
      (await stringPromise)
        ?? throw new TypeError()
    )
  ) + '!'
)
```
In contrast to the version with pipes, this code is deeply nested, not flat. The
expression has four levels of indentation instead of two. Reading its data flow
requires checking both the beginning and end of each expression, and each step
expression gradually increases in size. Inserting or removing any step of the
data flow also requires changes to the indentation of any previous steps’ lines.

<tr>
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
When tiny functions are only used once, and their bodies would be obvious and
self-documenting in meaning, they might be ritual boilerplate that a developer
may prefer to inline.

<td>″″

<tr>
<td>

```js
// 🚫 Syntax Error:
// Ambiguous await at start of pipeline.
await stringPromise
  |> # ?? throw new TypeError()
  |> `${#}, ${#}`
  |> #[0].toUpperCase() + #.substring(1)
  |> # + '!'
  |> new User.Message
```
This is a static [early error][], designed to [avoid a footgun at compile
time][static analyzability]. If this were a statement, then does the developer
want to apply the pipeline’s steps to `stringPromise`, *then* await the
pipeline’s result: `await (stringPromise |> …)`? Or does the developer want to
first await `stringPromise` and then apply the rest of the pipeline:
`(await stringPromise) |> …`? To avoid this footgun, `await` (and `yield`) are
prohibited from the start of pipeline heads. Just wrap the head in parentheses
`(await stringPromise) |> …` or move the `await` to another line
`stringPromise |> await # |> …`. [TO DO: Link to section on await / yield in
head expressions.]

<td>″″

</table>

#### Real-world examples
<details open>
<summary>It is also useful to look at code from real-world libraries or
standards and compare their readability with smart-pipelined versions. Numerous
examples of code that may benefit from smart pipelines abound.</summary>

#### Underscore.js
[Underscore.js][]. [Jeremy Ashkenas][jashkenas] &c. 2009–2018. MIT License.

<table>
<thead>
<tr>
<th>With smart pipes
<th>Status quo

<tbody>
<tr>
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
  if (obj == null) return 0;
  return obj |> isArrayLike
    ? obj.length
    : obj |> _.keys |> #.length;
}
```

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

#### Pify
[Pify][]. [Sindre Sorhus][sindresorhus] &c. 2015–2018. MIT License.

<table>
<thead>
<tr>
<th>With smart pipes
<th>Status quo

<tbody>
<tr>

<td>

```js
'package.json'
  |> await pify(fs.readFile)(#, 'utf8')
  |> JSON.parse |> #.name
  |> console.log
```

<td>

```js
pify(fs.readFile)('package.json', 'utf8')
  .then(data => {
    console.log(JSON.parse(data).name)
  })
```

</table>

#### Fetch Web Standard
[Fetch Standard][]. [Anne van Kesteren][annevk] &c. 2011–2018. WHATWG. Creative
Commons BY.

<table>
<thead>
<tr>
<th>With smart pipes
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

<tr>
<td>

```js
'https://example.com/'
  |> await fetch(#, { method: 'HEAD' })
  |> #.headers.get('content-type')
  |> log
```

<td>

```js
fetch('https://example.com/',
  { method: 'HEAD' }
).then(res =>
  log(res.headers.get('content-type'))
)
```

<tr>
<td>

```js
'https://pk.example/berlin-calling'
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

</table>

</details>

## Nomenclature
<details open>
<summary>Because this proposal introduces several new concepts, it is important
to use a consistent set of terminology.</summary>

### Pipe operator, pipeline, pipeline-level expression
The binary operator itself `|>` may be referred to as a **pipe**, a **pipe
operator**, or a **pipeline operator**; all these names are equivalent. This
specification will prefer the term “pipe operator”.

A pipe operator between two expressions forms a **pipe expression**. One or more
pipe expressions in a chain form a **pipeline**, and each pipe expression is
a **step** of the pipeline.

A **pipeline-level expression** is an expression at the same [precedence level
of the pipe operator][operator precedence]. Although all pipelines are
pipeline-level expressions, most pipeline-level expressions are not actually
pipelines. Conditional operations, logical-or operations, or any other
expressions that have tighter [operator precedence][] than the pipe operation –
those are also pipeline-level expressions.

### Head, head value, body, pipeline value, topical style, bare style
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

A pipeline’s body may be in one of two **styles**:\
topical style and\
bare style.

**[Topical style][]** is the default style. A pipeline body in topical style forms
an inner lexical scope – called the pipeline’s **topical scope** – within which
a special token is bound to the value of the head; the section below explains.

Alternatively, you may omit the topic references entirely, if the body is just a
**simple reference** to a function or constructor, such as with `… |>
capitalize` and `… |> new User.Message`. Such a pipeline body is in **[bare
style][]**; bare style is described in more detail below.

### Topic, topic reference
The **topic** (or **topic value**) of a lexical context is a value that the
lexical context is “about”. Not all lexical contexts has a topic. But in each
lexical context that does, its topic is bound to `#`, a special token called the
**topic reference**, aka the **topic bindee**, **topic placeholder** or **topic
anaphor**. `#` is a nullary operator that acts like a special variable:
implicitly bound to the topic value, but still lexically scoped.

The topic reference could also be called a “**topic variable**” or “**topic
identifier**”, [as they are called in other programming languages][topic
variables in other languages]. But in JavaScript, these phrases would be
misnomers. The topic reference is *not* actually a variable identifier and
cannot be manually declared (`const #` is a syntax error), nor can it be
assigned with a value (`# = 3` is a syntax error). Instead, the topic reference
is implicitly, lexically bound only within pipeline bodies.

### Explanation of conventions
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

</details>

## Grammar
This grammar of the pipeline operator juxtaposes brief rules written for the
JavaScript developer with formally written changes to the ECMAScript standard.
The grammar itself is a [context-free grammar supplemented with static
semantics][syntax and static semantics].

<details open>
<summary>The ECMAScript specification explains further.</summary>

> A context-free grammar consists of a number of **productions**. Each
> production has an abstract symbol called a **nonterminal** as its left-hand
> side, and a sequence of zero or more nonterminal and **terminal** symbols as
> its right-hand side. For each grammar, the terminal symbols are drawn from a
> specified alphabet.

</details>

***

This proposal uses the [same grammatical notation as that from the ECMAScript
standard][ECMAScript Notational Conventions, § Grammars] to denote its lexical
and syntactic grammars.

### Lexical grammar
The smart pipe operator adds two new tokens to JavaScript: `|>` the binary pipe,
and `#` the topic reference.

<details open>
<summary>The lexical rule for punctuator tokens would be modified so that
these two tokens are added.</summary>

The specification explains in [ECMAScript Notational Conventions, § Lexical
Grammar][]:

> A lexical grammar for ECMAScript is given in [ECMAScript Lexical Grammar][].
> This grammar has as its terminal symbols Unicode code points…It defines a set
> of productions…that describe how sequences of such code points are translated
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

The _Punctuators_ production is defined in [ECMAScript Punctuators][]. This
production would be changed from this:

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
<details open>
<summary>In the ECMAScript standard, the rules that produce expressions are
often parameterized with three flags, which are then recursively passed into
their constituent rules. These parameters thus must also be used by the new
rules in this proposal.</summary>

* **_In_**: Whether the current context allows the [`in` relational operator][],
  which is false only in the headers of [`for` iteration statements][].
* **_Yield_**: Whether the current context allows a `yield`
  expression/statement (that is, is the current function context a generator?).
* **_Await_**: Whether the current context allows an `await`
  expression/statement (that is, is the current function context an async
  function/generator?).

</details>

### Syntax and static semantics
The syntactic grammar of JavaScript can transform token sequences (defined by
the [lexical grammar][]) into **parse trees**: rooted tree data structures made
of **Parse Nodes**. This is described further in [ECMAScript Lexical Grammar][].

<details open>

> When a stream of code points is to be parsed as an ECMAScript Script or
> Module, it is first converted to a stream of input elements by repeated
> application of the lexical grammar; this stream of input elements is then
> parsed by a single application of the syntactic grammar. The input stream is
> syntactically in error if the tokens in the stream of input elements cannot be
> parsed as a single instance of the goal nonterminal (Script or Module), with
> no tokens left over.
>
> When a parse is successful, it constructs a parse tree, a rooted tree
> structure in which each node is a **Parse Node**. Each Parse Node is an
> instance of a symbol in the grammar; it represents a span of the source text
> that can be derived from that symbol. The root node of the parse tree,
> representing the whole of the source text, is an instance of the parse's goal
> symbol. When a Parse Node is an instance of a nonterminal, it is also an
> instance of some production that has that nonterminal as its left-hand side.
> Moreover, it has zero or more children, one for each symbol on the
> production's right-hand side: each child is a Parse Node that is an instance
> of the corresponding symbol.

</details>

***

The syntactic grammar of JavaScript further relies upon several functions that
analyze its syntactic structures. These functions are polymorphic on the types
of their input syntactic structures, and their definitions are often also
recursive. The ECMAScript specification goes into more detail on these **static
semantic rules** in [ECMAScript Static Semantic Rules][].

<details open>

> Context-free grammars are not sufficiently powerful to express all the rules
> that define whether a stream of input elements form a valid ECMAScript Script
> or Module that may be evaluated. In some situations additional rules are
> needed that may be expressed using either ECMAScript algorithm conventions or
> prose requirements. Such rules are always associated with a production of a
> grammar and are called the static semantics of the production.
>
> Static semantic rules have names and typically are defined using an algorithm.
> Named static semantic rules are associated with grammar productions and a
> production that has multiple alternative definitions will typically have for
> each alternative a distinct algorithm for each applicable named static
> semantic rule.

</details>

***

This specification defines additions for the following static semantic rules:

| Form                              | Notes                                             |
| --------------------------------- | ------------------------------------------------- |
| [Contains][]                      | Already defined in ES for nearly all nodes        |
| Is Function Definition            | Already defined in ES: all expressions            |
| Is Identifier Reference           | Already defined: primary- & LHS-level expressions |
| Is Valid Simple Assignment Target | Already defined: primary- & LHS-level expressions |
| [Early Errors][]                  | Already defined in ES for nearly all nodes        |

It should be noted that, in the ECMAScript standard, the Contains rule is
currently written as an infix operator: “… Contains …” for historical reasons.
This is unlike any other static semantic rule, which would be written as prefix
operators “_RuleName_ of … with arguments …”. There are plans to change all
static semantic rules to instead have a consistent infix syntax resembling
method calls: “…._ruleName_(…)”. For self-consistency, this proposal will use
that planned method-like syntax.

#### Static “Contains?”
<details open>
<summary>The ECMAScript spec implicitly defines the Contains rule for nearly all
nodes. Conceptually, a node Contains another node if the latter is somewhere in
the former.</summary>

[This quotation from the ECMAScript spec][ECMAScript static semantic rules] is
modified to use the new method-like syntax.

>    b. If _child_ is an instance of a nonterminal, then
>
>       i.  Let contained be the result of _child_.contains(_symbol_).
>       ii. If contained is true, return true.
>
> 2. Return false.
>
> The above definition is explicitly over-ridden for specific productions.

</details>

***

In the ECMAScript standard, the Contains rule is used by many other Static
Semantic Rules, such as [object initializers’ Computed Property Contains
rule][]. The rule is also generally overridden by methods definitions and other
function definitions, such that they hide their substructure from the rule.

(Uniquely among the static semantic rules, Contains is written as an infix
operator in the ECMAScript specification – “… Contains …” – for historical
reasons. This proposal will instead use the planned future new syntax
“….contains(…)”.)

**This proposal will use Contains to determine whether a pipeline’s body uses
its `#` topic reference.** This is so that many [footguns may be statically
detected as an early error][static analyzability] – for instance, using a
pipeline in topical style without ever using its topic in its body [TO DO: Link].

Contains does not penetrate into the bodies of function and method definitions,
hiding them from the rules in outside contexts. All definitions for functions,
generators, methods, and so forth override Contains to always return false, with
this note:

> Static semantic rules that depend upon substructure generally do not look into
> function definitions.

There is one exception: arrow functions expose the use of `new.target`, `this`,
and `super`, because, unlike other functions, they do no rebind those three
forms; they use the outer context to evaluate them. (See [ECMAScript arrow
functions, § SS: Contains][]).

This proposal further extends that exception so that arrow functions also reveal
any use of `#` within their bodies. This is because arrow functions, alone among
functions, also do not rebind or shadow the outer context’s topic. (`#` cannot
be used within arrow-function parameter lists or any function’s parameter list.)
See [TO DO: Topics and inner functions].

<details>

[ECMAScript arrow functions, § SS: Contains][] is amended.

With parameter _symbol_.

* _ArrowFunction_ : _ArrowParameters_ `=>` _ConciseBody_

  1. If _symbol_ is not one of _NewTarget_, _SuperProperty_, _SuperCall_,
    `super`, `this` or `#`, return false.
  2. If _ArrowParameters_.contains(_symbol_) is true, return true.
  3. Return _ConciseBody_.contains(_symbol_).

* _ArrowParameters_ : _CoverParenthesizedExpressionAndArrowParameterList_

  [Unchanged.]

</details>

#### Static Early Errors
Certain syntax errors cannot be detected by the context-free grammar alone yet
must still be detected at compile time. Early Error Rules are Static Semantic
Rules that define when such extra syntax errors occur.

<details open>

> A special kind of static semantic rule is an Early Error Rule. Early error
> rules define early error conditions (see clause 16) that are associated with
> specific grammar productions. Evaluation of most early error rules are not
> explicitly invoked within the algorithms of this specification. A conforming
> implementation must, prior to the first evaluation of a Script or Module,
> validate all of the early error rules of the productions used to parse that
> Script or Module. If any of the early error rules are violated the Script or
> Module is invalid and cannot be evaluated.

</details>

***

One such static early error is mentioned in both the section on the Goal [static
analyzability][] and the section on the [Contains][] rule. There are also several
others, each designed to prevent a footgun of ambiguity, by forcing the developer
to clarify their intent.

* Pipeline heads:
  * Pipeline heads must not directly start with `await` or `yield`. This is an early
    error. If `await promise |> …` were a statement, then did the author want to apply
    the pipeline’s steps to `promise`, *then* await the pipeline’s result:
    `await (promise |> …)`? Or does the developer want to first await `promise`
    and then apply the rest of the pipeline: `(await promise) |> …`? To avoid
    this footgun, `await` (and `yield`) are prohibited from the start of
    pipeline heads. Just wrap the head in parentheses `(await promise) |> …` or
    move the `await` to another line `promise |> await # |> …`. [TO DO: Link to
    section on await / yield in head expressions.]

* Pipeline bodies:
  * Pipelines that are in topical style but that do not ever use their topics
    anywhere in their bodies, such as `x |> 3`, are an early error. Such expressions
    would be always useless and almost certainly not what the author had intended.
    [TO DO: Link.]

  * Pipeline bodies that start with `yield` must be parenthesized. Otherwise
    they are an early error. This is because the `yield` operator has such a
    loose precedence that `x |> yield # |> f` is an ambiguous footgun. It is very
    likely that the developer meant `(x |> (yield #)) |> f`, but because `yield`
    has such loose precedence, without parentheses, the pipeline will be parsed
    instead as `x |> (yield (# |> f))`, which has a very different meaning. With
    this early error, the developer is forced to clarify their `yield`: either
    `x |> (yield #) |> f` or `x |> (yield # |> f)`. [TO DO: Link.]

[TO DO: Add bidirectional associativity to Goals]

### Operator precedence
As a binary operation forming compound expressions, the [operator precedence and
associativity][MDN operator precedence] of pipelining must be determined,
relative to other operations.

Precedence is tighter than assignment (`=`, `+=`, …), generator `yield` and
`yield *`, and sequence `,`; and it is looser than every other type of
expression. If the pipe operation were any tighter than this level, its body
would have to be parenthesized for many frequent types of expressions. However,
the result of a pipeline is also expected to often serve as the body of a
variable assignment `=`, so it is tighter than assignment operators.

The pipe operator actually has [bidirectional associativity][]. However, for the
purposes of this grammar, it will have left associativity.

<details open>
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
| ″″             | Regular expression      |`/…/…`          | ″″                       |
| ″″             | Templates               |```…```         | ″″                       |
| ″″             | Parentheses             |`(…)`           | ″″                       |
| LHS            | Dynamic properties      |`…[…]`          | LTR infix with circumfix |
| ″″             | Static properties       |`….…`           | ″″                       |
| ″″             | Tagged templates        |`` …`…` ``      | ″″                       |
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
| ″″             | Unary `+`/`-`           |`+…`            | ″″                       |
| ″″             | Bitwise NOT `~…`        |`~…`            | ″″                       |
| ″″             | Logical NOT `!…`        |`!…`            | ″″                       |
| ″″             | Await                   |`await …`       | ″″                       |
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
| Bitwise OR     |                         |`… \| …`        | ″″                       |
| Logical AND    |                         |`… ^^ …`        | ″″                       |
| Logical OR     |                         |`… \|\| …`      | ″″                       |
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
| ″″             |                         |`… \|= …`       | ″″                       |
| Yield level    |                         |`yield …`       | RTL prefix               |
| ″″             |                         |`yield * …`     | ″″                       |
| Ultimate level | Comma                   |`…, …`          | LTR infix                |

</details>

### Topic reference • Syntax grammar
The topic reference integrates into the ECMAScript syntax as one of the
[ECMAScript Primary Expressions][], just like `this`. Their production rule
needs to be modified so that the `#` appears as one of the types of primary
expressions.

<details open>
<summary>An assignment-level expression currently may be a this reference,
identifier reference, null / undefined / true / false literal, array / object /
regular-expression / template literal, function / async-function / generator /
class expression. These possibilities are given the same parameters that the
assignment-level expression happens to have also gotten, except where they
would be unnecessary, such as for the this token.</summary>

The old version:
```
PrimaryExpression[Yield, Await]:
  `this`
  IdentifierReference[?Yield, ?Await]
  Literal
  ArrayLiteral[?Yield, ?Await]
  ObjectLiteral[?Yield, ?Await]
  FunctionExpression
  ClassExpression[?Yield, ?Await]
  GeneratorExpression
  AsyncFunctionExpression
  RegularExpressionLiteral
  TemplateLiteral[?Yield, ?Await, ~Tagged]
  CoverParenthesizedExpressionAndArrowParameterList[?Yield, ?Await]
```

</details>

***

<details open>
<summary>Added to this list would be the topic token.</summary>

The new version:
```
PrimaryExpression[Yield, Await]:
  `this`
  `#`
  IdentifierReference[?Yield, ?Await]
  Literal
  …
```

</details>

### Topic reference • Static semantics

<details open>
<summary>The topic reference is neither a function definition nor an identifier
reference. This is the same as almost every other primary expression, except
for identifiers, parenthesized expressions, and arrow parameter lists.</summary>

* IsIdentifierRef
  * `PrimaryExpression : IdentifierReference`

    Return true.

  * ``PrimaryExpression : `this` | `#` | Literal | …``

    Return false.

* IsValidSimpleAssignmentTarget
  * ``PrimaryExpression : `this` | `#` | Literal | …``

    Return false.

  * `PrimaryExpression : CoverParenthesizedExpressionAndArrowParameterList`

    [Unchanged from original specification.]

</details>

### Topic reference • Runtime semantics

During runtime, the topic reference uses the [ResolveTopic abstract
operation][resolving topics] on the running execution context’s lexical
environment.

<details open>

* Evaluation
  * PrimaryExpression : `#`
    * Return ? ResolveTopic()

</details>

### Pipeline-level expressions • Syntax grammar
The production rule for [ECMAScript Assignment-level Expressions][] needs to be
modified so that pipe expressions slip in between it and conditional-level
expressions in the hierarchy. Then the conditional-expression rule would be used
in the production for pipeline-level expressions (also defined soon), preserving
the unbroken recursive hierarchy of expression types.

<details open>
<summary>An assignment-level expression currently may be a conditional
expression, yield expression, arrow function, async arrow function, or
assignment. These possibilities are given the same parameters that the
assignment-level expression happens to have also gotten.</summary>

The old version:

```
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

<details open>
<summary>In this proposal, the conditional-expression production rule would be
replaced with one for pipeline-level expressions, which will be defined
next.</summary>

The new version:

```
AssignmentExpression[In, Yield, Await] :
  PipelineExpression[?In, ?Yield, ?Await]
  [+Yield] YieldExpression[?In, ?Await]
  ArrowFunction[?In, ?Yield, ?Await]
  …
```

</details>

***

An expression is a pipeline-level expression (given the usual three [grammar
parameters][]) only if:

* It is either also a conditional-level expression, with the same parameters
  used by the pipeline-level expression;
* Or it is another pipeline-level expression, followed by a `|>` token, then a
  pipeline body (defined next), with the same parameters as above.

<details open>
<summary>This would be defined in a new production rule.</summary>

```
// New rule
PipelineExpression[In, Yield, Await] :
  ConditionalExpression[?In, ?Yield, ?Await]
  PipelineExpression[?In, ?Yield, ?Await] `|>`
    PipelineBody[?In, ?Yield, ?Await]
```

</details>

### Pipeline-level expressions • Static semantics

<details open>
<summary>The pipeline-level expression is neither a function definition nor an
identifier reference.</summary>

* IsIdentifierRef
  * PipelineExpression : PipelineExpression `|>` PipelineBody

    Return false.

* IsValidSimpleAssignmentTarget
  * PipelineExpression : PipelineExpression `|>` PipelineBody

    Return false.

</details>

<details open>A pipeline expression uses its outer lexical context’s topic only if
the pipeline’s head uses the outer context’s topic. The pipeline’s body cannot
use the outer context’s topic, because the body is evaluated within a second,
inner lexical context, within which the topic reference is rebound to another
value. If there is a topic reference defined in the outer context, then it is
shadowed within the body.</summary>

* UsesOuterTopic

  [TO DO]

</details>

### Pipeline-level expressions • Runtime semantics
During runtime, [TO DO]

<details open>

* Evaluation
  * PipelineExpression : PipelineExpression `|>` PipelineBody
    1. Let _headRef_ be the result of evaluating ? _PipelineExpression_.
    2. Let _headValue_ be the result of ? GetValue(_headRef_).
    3. Let _bodyRef_ be PipelineBodyEvaluation of _PipelineBody_ with argument
       _headValue_.
    4. Return ? GetValue(_bodyRef_).

</details>

### Smart body syntax
Most pipelines will use the topic reference `#` in their bodies. As already
explained above in [nomenclature][], this style of pipeline is called **topical
style**.

But for two certain simple cases – unary functions and constructors – you may
omit the topic reference from the body. This is called **bare style**.

When a pipe is in bare style, we refer to the body as a **bare function** or a
**bare constructor**, depending on the rules in [bare style][]. The body acts
as just a simple reference to a function or constructor, such as with `… |>
capitalize` and `… |> new User.Message`. The body’s value would then be called
as a unary function or constructor, without having to use the topic reference as
an explicit argument.

<details open>

<summary>Syntactic grammar</summary>

[TO DO: Note no parameters in bare style.]

```
// New rule
PipelineBody[In, Yield, Await] :
  PipelineBareFunctionCall
  PipelineBareConstructorCall
  PipelineTopicalBody[?In, ?Yield, ?Await]
```

</details>

***

<details open>

<summary>The rules of the two respective styles will be explained in more
detail, but an overview is given in a table.</summary>

| Valid topical style     | Valid bare style                  | Invalid bare style
| ----------------------- | --------------------------------- | --------------------
|`… \|> o(#)`             |`… \|> o`                          |  `… \|> o()` 🚫
| ″″                      | ″″                                | `… \|> (o)` 🚫
| ″″                      | ″″                                | `… \|> (o())` 🚫
|`… \|> new o(#)`         |`… \|> new o`                      | `… \|> new o()` 🚫
| ″″                      | ″″                                | `… \|> (new o)` 🚫
| ″″                      | ″″                                | `… \|> (new o())` 🚫
| ″″                      | ″″                                | `… \|> new (o)` 🚫
| ″″                      | ″″                                | `… \|> new (o())` 🚫
|`… \|> o.m(#)`           |`… \|> o.m`                        | `… \|> o.m()` 🚫
| ″″                      |`const m = o::m; … \|> m`          | `… \|> o.m()` 🚫
|`… \|> new o.m(#)`       |`… \|> new o.m`                    | `… \|> o.m()` 🚫
| ″″                      |`const m = o::m; … \|> m`          | `… \|> o.m()` 🚫
|`… \|> o.m(arg, #)`      |`const m = o::m(arg); … \|> m`     | `… \|> o.m(arg)` 🚫
|`… \|> new o.m(arg, #)`  |`const m = new o::m(arg); … \|> m` | `… \|> new o.m(arg)` 🚫
|`… \|> o[symbol](#)`     |`const m = o[symbol]; … \|> m`     | `… \|> o[symbol]` 🚫
|`… \|> new o[symbol](#)` |`const m = new o[symbol]; … \|> m` | `… \|> new o[symbol]` 🚫
|`… \|> o.makeFn()(#)`    |`const m = o.makeFn(); … \|> m`    | `… \|> o.makeFn()` 🚫
|`… \|> new o.makeFn()(#)`|`const m = new o.makeFn(); … \|> m`| `… \|> new o.makeFn()` 🚫

</details>

#### Bare style
The **bare style** supports using simple identifiers, possibly with chains of
simple property identifiers. If there are any operators, parentheses (including
for method calls), brackets, or anything other than identifiers and dot
punctuators, then it is in topical style, not in bare style.

##### Simple reference
First, let’s call a mere identifier – optionally with a chain of properties, and
with no parentheses, brackets, or operators – a **simple reference**.

**If an expression** is of the form **_identifier_**\
or _topic_ `|>` _identifier0_`.`_identifier1_\
or _topic_ `|>` _identifier0_`.`_identifier1_._identifier2_\
or …), then the pipeline is a **simple reference**.

<details open>

This is achieved by defining the _SimpleReference_ production using [ECMAScript
_IdentifierReference_][], [ECMAScript _IdentifierName_][], and left recursion,
in imitation of how [ECMAScript _MemberExpression_][] handles method chains.

```
SimpleReference :
  IdentifierReference
  SimpleReference `.` IdentifierName
```

</details>

***

<details open>
<summary>Simple references’ runtime semantics are exactly the same as the
member expressions they resemble.</summary>

This section is adapted from the [ECMAScript Property Accessors, § RS:
Evaluation][].

* Evaluation
  * SimpleReference : SimpleReference `.` IdentifierName
    * Is evaluated in exactly the same manner as [MemberExpression `:`
      MemberExpression `.` IdentifierName][ECMAScript Property Accessors,
      § RS: Evaluation] except that the contained `SimpleReference` is evaluated
      in step 1.

</details>

##### Bare function call
If the body is a merely a simple reference, then that identifier is interpreted
to be a **bare function call**. The pipeline’s value will be the result of
calling the body with the current topic as its argument.

That is: **if a pipeline** is of the form **_topic_ `|>` _identifier_**\
or _topic_ |> _identifier0_._identifier1_\
or _topic_ |> _identifier0_._identifier1_._identifier2_\
or …,\
then the pipeline is a **bare function call**. The **pipeline’s value** is
**_body_`(`_topic_`)`**.

<details open>
<summary>Syntactic grammar</summary>

```
PipelineBareFunctionCall :
  SimpleReference
```

</details>

[TO DO: Make sure that `eval` and other special “functions” are not allowed.]

<details open>
<summary>Runtime semantics</summary>

This algorithm was adapted from [ECMAScript Function Calls, § RS:
Evaluation][].

* PipelineBodyEvaluation
  * With parameter _headValue_.
  * Note that this PipelineBodyEvaluation rule is used in the evaluation of
    PipelineExpression, defined previously.
  * PipelineBareFunctionCall : SimpleReference
    1. Let _ref_ be the result of evaluating _SimpleReference_.
    2. Let _func_ be ? GetValue(_ref_).
    3. Let _thisCall_ be this _PipelineBareFunctionCall_.
    4. Let _tailCall_ be IsInTailPosition(thisCall).
    5. Let _Arguments_ be a [List][ECMAScript Lists and Records] containing
       the one element which is _headValue_.
    6. Return ? EvaluateCall(_func_, _ref_, Arguments, tailCall).

</details>

##### Bare constructor call
If the body starts with `new`, followed by mere identifier, optionally with a
chain of properties, and with no parentheses or brackets, then that identifier
is interpreted to be a **bare constructor**.

That is: **if a pipeline** is of the form **_topic_ |> _identifier_**\
or _topic_ |> _identifier0_._identifier1_\
or _topic_ |> _identifier0_._identifier1_._identifier2_\
or …,\
then the pipeline is a **bare function call**. The **pipeline’s value** is
**_body_`(`_topic_`)`**.

<details open>
<summary>Syntax grammar</summary>

```
PipelineBareConstructorCall :
  `new` SimpleReference
```

</details>

<details open>
<summary>Runtime semantics</summary>

This algorithm was adapted from [ECMAScript `new` operator, § RS: Evaluation][].

* PipelineBodyEvaluation
  * With parameter _headValue_.
  * Note that this PipelineBodyEvaluation rule is used in the evaluation of
    PipelineExpression, defined previously.
  * PipelineBareConstructorCall : `new` SimpleReference
    * [TO DO: Can we use EvaluateNew if _SimpleReference_ is technically not the
      same as MemberExpression? Should we just use MemberExpression with some
      limitations?]

</details>

***

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
**If a pipeline** of the form _topic_ |> _body_ is ***not* in bare
style** (that is, it is *not* a bare function call or bare constructor call),
then it **must be in topical style**.

<details open>
<summary>The pipeline’s value is whatever the body expression evaluates into,
assuming that the topic value is first bound to the topic reference within the
body scope.</summary>

But more precisely, it binds the topic to the pipeline’s head value then
evaluates the RHS [TO DO]

* Evaluation
  * PipelineExpression : PipelineExpression `|>` PipelineBody
    1. Let _headValue_ be the result of evaluating _PipelineExpression_.
    2. [TO DO: Create topic environment]
    3. [TO DO: Evaluate body in new environment]

Topical style behaves like **`do { const ` _topicIdentifier_ `=` _topic_`;
`_substitutedBody_` }`**, where:

* _topicVariable_ is any [identifier that is *not* already used by any
  variable][lexically hygienic], in the outer lexical context or the body’s
  inner topical context,
* And _substitutedBody_ is _body_ but with every instance of outside of
  the topic reference replaced by _topicVariable_.

[TO DO: Add link to term-rewriting appendix.]

</details>

### Topic resolution
Resolving the topic reference is a [TO DO]

#### Lexical Environments

<details open>
<summary>The ECMAScript spec associates Identifiers with variables or functions
using an abstract data structure called a Lexical Environment, which is
essentially a linked list of Lexical Environments. A single piece of the chain
of Lexical Environments is called an Environment Record. Syntactic structures
such as functions and blocks each have their own Lexical Environments, created
whenever such code is evaluated at runtime. </summary>

> A Lexical Environment is a specification type used to define the association
> of Identifiers to specific variables and functions based upon the lexical
> nesting structure of ECMAScript code. A Lexical Environment consists of an
> Environment Record and a possibly null reference to an outer Lexical
> Environment. Usually a Lexical Environment is associated with some specific
> syntactic structure of ECMAScript code such as a FunctionDeclaration, a
> BlockStatement, or a Catch clause of a TryStatement and a new Lexical
> Environment is created each time such code is evaluated.
>
> An Environment Record records the identifier bindings that are created within
> the scope of its associated Lexical Environment. It is referred to as the
> Lexical Environment's EnvironmentRecord
>
> The outer environment reference is used to model the logical nesting of
> Lexical Environment values. The outer reference of a (inner) Lexical
> Environment is a reference to the Lexical Environment that logically surrounds
> the inner Lexical Environment. An outer Lexical Environment may, of course,
> have its own outer Lexical Environment. A Lexical Environment may serve as the
> outer environment for multiple inner Lexical Environments. For example, if a
> FunctionDeclaration contains two nested FunctionDeclarations then the Lexical
> Environments of each of the nested functions will have as their outer Lexical
> Environment the Lexical Environment of the current evaluation of the
> surrounding function.
>
> A global environment is a Lexical Environment which does not have an outer
> environment. The global environment's outer environment reference is null. A
> global environment's EnvironmentRecord may be prepopulated with identifier
> bindings and includes an associated global object whose properties provide
> some of the global environment's identifier bindings. As ECMAScript code is
> executed, additional properties may be added to the global object and the
> initial properties may be modified.
>
> A module environment is a Lexical Environment that contains the bindings for
> the top level declarations of a Module. It also contains the bindings that are
> explicitly imported by the Module. The outer environment of a module
> environment is a global environment.
>
> A function environment is a Lexical Environment that corresponds to the
> invocation of an ECMAScript function object. A function environment may
> establish a new this binding. A function environment also captures the state
> necessary to support super method invocations.
>
> Lexical Environments and Environment Record values are purely specification
> mechanisms and need not correspond to any specific artefact of an ECMAScript
> implementation. It is impossible for an ECMAScript program to directly access
> or manipulate such values.

</details>

***

Any topic-binding syntactic [TO DO]

A topic environment is a Lexical Environment that corresponds

[TO DO: Change “topical style” to “topic style”, to be consistent with “topic
environment”. After all, this is a style of topics, not a style that itself
is “topical” in the usual adjectival sense.]

[TO DO]

#### Abstract operations
ResolveTopic is a new abstract operation that acts upon a Lexical Environment.

[TO DO]

### Multiple topic references and inner functions
<details open>
<summary>The topic reference may be used multiple times in a pipeline body. Each
use refers to the same value (wherever the topic reference is not overridden by
another, inner pipeline’s topical scope). Because it is bound to the result of
the topic, the topic is still only ever evaluated once.</summary>

The lines in each of the following rows are equivalent.

| Pipeline                         | Block                                             |
| -------------------------------- | ------------------------------------------------- |
|`… \|> f(#, #)`                   |`const $ = …; f($, $)`                             |
|`… \|> [#, # * 2, # * 3]`         |`const $ = …; [$, $ * 2, $ * 3]`                   |

[TO DO]

</details>

### Inner blocks
<details open>
<summary>The body of a pipeline may contain an inner arrow function but no other
type of block expression.</summary>

The lines in each of the following rows are equivalent.

| Pipeline                         | Block                                              |
| -------------------------------- | -------------------------------------------------- |
|`… \|> x => # + x`                |`const $ = …; x => # + x`                           |
|`… \|> settimeout(() => # * 5)`   |`const $ = …; settimeout(() => $ * 5)`              |

However, you cannot use use topic references inside of other types of blocks:
function, async function, generator, async generator, or class.

More precisely, all block expressions (other than arrow functions) shadow any
outer lexical context’s topic with its own *absence* of a topic. This behavior
is in order to fulfill both [Goals 3 and 6][goals].

| Pipeline                         |                                                    |
| -------------------------------- |--------------------------------------------------- |
|`… \|> function () { # }`         | Syntax Error: Topic never used by pipeline’s body. |

[TO DO]

</details>

### Nested pipelines
<details open>
<summary>Both the head and the body of a pipeline may contain nested inner
pipelines. Nested pipelines in the body is not encouraged, but it is still
permitted.</summary>

The lines in each of the following rows are equivalent.

| Pipeline                         | Block                                              |
| -------------------------------- | -------------------------------------------------- |
|`… \|> f(() => f(#) * 5)`         |`const $ = …; f(x => f($) * 5)`                     |
|`… \|> f(() => f(#) \|> # * 5)`   |`const $ = …; f(x => f($) \|> # * 5)`               |
|`… \|> f(() => # \|> f \|> # * 5)`|`const $ = …; f(x => $ \|> f \|> # * 5)`            |

[TO DO]

</details>

## Relations to other work
[TO DO: https://github.com/gajus/babel-plugin-transform-function-composition]

[TO DO: refer to #background list of programming languages]

### Other ECMAScript proposals
[TO DO: `do` expressions]

[TO DO: Partial application: “topic reference” vs. “placeholder”.]

[TO DO: Private class fields and `#`.]

[TO DO: Class decorators and `@`.]

[TO DO: Block params: https://github.com/samuelgoto/proposal-block-params]

[TO DO: Function bind: https://github.com/zenparsing/es-function-bind]

[TO DO: pattern matching https://github.com/tc39/proposal-pattern-matching]

### Possible future extensions to the topic concept
<details open>

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
reading and writing, while perhaps preserving [static analyzability][] and… [TO DO]

[TO DO: Note on forward compatibility with these possibilities.]

[TO DO: Add, to above, a version of second example with `do` blocks showcasing
the `#|>` idiom.]

[TO DO: Can partial application be integrated with topics?]

#### Headless property access
This example demonstrates a possible future “headless property” syntax in which
the callee of a property-access expression may be omitted, assuming that no
possible expression immediately precedes it. The omitted, invisible, tacit
callee value is the lexical context’s topic. This would greatly increase the
potential of tacit programming, especially when combined with the hypothetical
syntaxes below.

(It should be noted that headless properties would introduce a mild ASI hazard:
if a possible callee precedes the headless property, even on another line of
code, then the headless property would instead chain onto that preceding callee.
A semicolon would required to separate the possible callee and the headless
property. More exploration would be needed to assess how severe this hazard
would be compared to the benefits it would bring.)

<table>
<thead>
<tr>
<th>With hypothetical proposal
<th>With only this proposal

<tbody>
<tr>
<td>

```js
x |> f |> .property |> g
```

<td>

```js
x |> f |> #.property |> g
```

</table>

#### Headless pipelining
This example demonstrates a possible future “headless
pipeline” syntax in which the head of a pipeline operation may be omitted,
assuming that no possible expression immediately precedes it. The omitted,
invisible, tacit pipeline head value is the outer lexical context’s topic.

This also would greatly increase the potential of tacit programming when
combined with the hypothetical syntaxes below, which define additional contexts
in which the topic is lexically bound.

(It should be noted that headless properties would also introduce a mild ASI
hazard: if a possible callee precedes the headless pipeline, even on another
line of code, then the headless pipeline would instead chain onto that preceding
pipeline. A semicolon would required to separate the possible callee and the
headless property. More exploration would be needed to assess how severe this
hazard would be compared to the benefits it would bring.)

[TO DO: Add handling of `yield`/`await` statements versus expressions.
Example: Is
`function * () => yield |> 3` grouped as
`function * () => (yield) |> 3` or is it
`function * () => yield (|> 3)`?
Answer: bare `yield` and `await` are forbidden from the heads of pipelines.
This should be done in this pipeline proposal, throwing a syntax error,
because it’d be unclear anyway even without headless pipelining.]

<table>
<thead>
<tr>
<th>With hypothetical proposal
<th>With only this proposal

<tbody>
<tr>
<td>

```js
f |> .property |> g
```

<td>

```js
# |> f |> #.property |> g
```

</table>

#### Topical `for` loop
With this smart-pipe proposal only, `for`–`of` statements would prohibit the use
of `#` within their bodies, except where `#` is inside an inner pipeline inside
the `for` loop.

With another, future proposal, all `for`–`of` loops would implicitly bind each
iterator value to `#`. This implicit binding would be in addition to the
explicit binding of a normal variable `i` declared within the parenthesized
antecedent `for (const i of … { … })`.

An additional tacit `for` loop form, completely lacking a parenthesized
antecedent, would also be added. This tacit form is what is used in this example.
[TO DO: Link to section on deep nesting.] This example also uses the hypothetical
headless pipelining syntax from above.

<table>
<thead>
<tr>
<th>With hypothetical proposal
<th>With only this proposal

<tbody>
<tr>
<td>

```js
for (range(0, 50)) {
  log(# ** 2);
  log(|> Math.sqrt);
}
```

<td>

```js
for (const i of range(0, 50)) {
  log(i |> # ** 2);
  log(i |> Math.sqrt);
}
```

</table>

#### Topical `for`–`await` loop
This is similar to the tacit topical synchronous `for` loop above. With this
proposal only, `for`–`await`–`of` statements would prohibit the use of `#`
within their bodies, except where `#` is inside an inner pipeline inside the
`for` loop.

With another, future proposal, all `for`–`await`–`of` loops would implicitly bind
each iterator value to `#`. This implicit binding would be in addition to the
explicit binding of a normal variable `i` declared within the parenthesized
antecedent `for await (const i of …) { … }`.

An additional tacit `for await` loop form, completely lacking a parenthesized
antecedent, would also be added. This tacit form is what is used in this
example. [TO DO: Link to section on deep nesting.] This example also uses the
hypothetical headless pipelining syntax from above.

<table>
<thead>
<tr>
<th>With hypothetical proposal
<th>With only this proposal

<tbody>
<tr>
<td>

```js
for await (stream) {
  yield
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

</table>

#### Topical function / method definition
With this smart-pipe proposal only, all function / method definitions would
prohibit the use of `#` within their bodies, except where `#` is inside an inner
pipeline inside the function / method. (However, arrow functions do not have
this restriction; they may use `#`, which refers to their outer scope’s topic.)

With another, future proposal, all function / method definitions would
implicitly bind their first arguments to `#`. This implicit binding would be in
addition to the explicit binding of a normal parameter variable `x` declared
within the parenthesized antecedent of `function (x, …) { … }` or a `method { m
(x, …) { … } }`. `#` here would be practically similar to `arguments`. But
unlike `arguments`, `#` in function bodies would obey the same lexical static
rules imposed upon `#` in pipeline bodies and elsewhere.

As is already possible, a tacit `function` definition, completely lacking a
parenthesized antecedent, could also be used. This tacit form is what is used in
this example. [TO DO: Link to section on deep nesting.] This example also uses
the hypothetical headless property syntax from above.

<table>
<thead>
<tr>
<th>With hypothetical proposal
<th>With only this proposal

<tbody>
<tr>
<td>

```js
function capitalize {
  return
    #[0].toUpperCase()
    + .substring(1)
}
```

<td>

```js
function capitalize (str) {
  return str[0].toUpperCase()
    + str.substring(1)
}
```

</table>

#### Topical block parameter
The proposed syntax of [ECMAScript block parameters][] may greatly benefit from
using the topic concept. As with topical function definitions, making all block
parameters topical would enable the use of the topic reference as an implicit
first parameter. The block-parameter proposal itself has not yet settled on how
to parameterize its block parameters. The topic reference may be the key to
solving this problem, making other, special block parameters unnecessary. This
example also uses the hypothetical headless property syntax and headless
pipelining syntax from above.

<table>
<thead>
<tr>
<th>With hypothetical proposal
<th>With only this proposal

<tbody>
<tr>
<td>

```js
materials.map { |> f |> .length }
```
```js
server(app) {
  .get('/') do (response) {
    request()
      |> .get('param1')
      |> `hello world ${#}`
      |> response.send
  }

  .listen(3000) {
    log('hello')
  }
}
```

<td>

```js
materials.map { f(???).length }
```
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

</table>

#### Topical thin-arrow function
This example uses a new token, the thin arrow, which is similar to the `=>` fat
arrow in that it creates arrow functions. The only difference is that it also
lexically binds `#` to its first argument, unlike the fat arrow. This example
also uses the hypothetical headless pipelining syntax from above.

<table>
<thead>
<tr>
<th>With hypothetical proposal
<th>With only this proposal

<tbody>
<tr>
<td>

```js
materials.map(-> |> f |> .length)
```

<td>

```js
materials.map(m => m |> f |> #.length)
```

</table>

#### Topical pattern matching
The proposed syntax of [ECMAScript pattern matching][] would bind the topic
reference within the scope of a successful match clause’s scope. The topic value
would be the truthy result of the successful `Symbol.matches` call. This example
also uses the hypothetical headless property syntax from above.

<table>
<thead>
<tr>
<th>With hypothetical proposal
<th>With only this proposal

<tbody>
<tr>
<td>

```js
match (x) {
  100: #
  Array:
    .length
  /(\d)(\d)(\d)/:
    #.groups |>
      #[0] + #[1] + #[2]
}
```

<td>

```js
match (x) {
  100: x
  Array:
    x.length
  /(\d)(\d)(\d)/ -> m:
    m.groups |>
      #[0] + #[1] + #[2]
}
```

</table>

#### Tacit pattern matching
[ECMAScript pattern matching] could also have a completely tacit version, in
which the parenthesized antecedent is completely omitted in favor of tacitly
using the outer context’s topic. (This would have to somehow be distinguishable
from a call to a function named `match` with a [bare block argument][ECMAScript
block parameters].) This example also uses the hypothetical headless pipelining
syntax from above.

<table>
<thead>
<tr>
<th>With hypothetical proposal
<th>With only this proposal

<tbody>
<tr>
<td>

```js
function getLength {
  match {
    { x, y }:
      (x ** 2 + y ** 2)
        |> Math.sqrt
    [...]:
      .length
    else:
      throw new Error(#)
  }
}
```

<td>

```js
function getLength (vector) {
  match (vector) {
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

</table>

#### Tacit error capture
With this smart-pipe proposal only, all `try` statements’ `catch` clauses would
prohibit the use of `#` within their bodies, except where `#` is inside an inner
pipeline inside the `catch` clause. [TO DO: Link to sections explaining these
inner block rules.]

With another, future proposal, all `catch` causes would implicitly bind
their caught errors to `#`. This implicit binding would be in addition to the
explicit binding of a normal variable `error` declared within the parenthesized
antecedent `try { … } catch (error) { … }`.

An additional bare `catch` form, completely lacking a parenthesized antecedent,
has already been proposed as [ECMAScript optional catch binding][]. This bare
form would also support implicit `#` binding, serving as the fully tacit form
used in this example. [TO DO: Link to section on deep nesting.] The bare form,
along with the hypothetical headless property syntax from above, are
demonstrated here.

<table>
<thead>
<tr>
<th>With hypothetical proposal
<th>With only this proposal

<tbody>
<tr>
<td>

```js
try {
  …
} catch {
  log(.message)
} finally {
  …
}
```

<td>

```js
try {
  …
} catch (error) {
  log(#.message)
} finally {
  …
}
```

<tr>
<td>

```js
try {
  …
} catch {
  match {
    MyError:
      #|> f
    TypeError:
      #|> g
    SyntaxError:
      #|> f |> g
    Error:
      `Error: ${#.message}`
  }
}
```

<td>

```js
try {
  …
} catch (error) {
  match (error) {
    MyError:
      error |> f
    TypeError:
      error |> g
    SyntaxError:
      error |> f |> g
    Error:
      `Error: ${error.message}`
  }
}
```

</table>

#### Topical metaprogramming reference
In the event that TC39 seriously considers the topical function definitions
shown above, a **`function.topic`** metaprogramming operator, in the style of
the [`new.target`][] operator, could be useful in creating topic-aware functions.

This might be especially useful in creating APIs resembling [domain-specific
languages][DSLs] with [ECMAScript block parameters][]. This example creates
three functions that form an API resembling [Visual Basic’s `select`
statement][]. Two of these functions (`when` and `otherwise`) that are expected
to be called always within the third function (`select`)’s callback block.

An alternate solution without metaprogramming topics is not yet specified by the
current proposal for [ECMAScript block parameters][].

```js
class CompletionRecord { [[TO DO]] }

function select (value, callback) {
  const contextTopic =
    [[TO DO: create completion record]]
  return callback(topic) // TO DO
}

function otherwise (callback) { [[TO DO]] }

function when (testValue, callback) {
  const contextTopic = function.topic
  return match (contextTopic) {
    [TO DO]:
      |> applyWhen(#, testValue, callback)
    else:
      throw new Error('when used outside select block')
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
    ? contextTopic |> callback |> [[TO DO]]
    : [[TO DO: Pass to next when]]
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
    throw new Error(`Error: ${# |> format}`)
  }
}
```

</details>

### Alternative solutions explored
There are a number of other ways of potentially accomplishing the above use
cases. However, the authors of this proposal believe that the smart pipe
operator may be the best choice. [TO DO]

## Term rewriting

<details open>

### Term rewriting topical style
Pipe bodies in topical style can be rewritten into a nested `do` expression.
There are two ways to illustrate this equivalency. The first way is to [replace
each pipe expression’s topic references with an autogenerated variable][term
rewriting with autogenerated variables], which must be guaranteed to be
[lexically hygienic][] and to not conflict with other variables. The alternative
way is to [use two variables – the topic reference `#` and a single dummy
variable][term rewriting with single dummy variable] – which also preserves
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

1. The head is first evaluated in the current lexical context.
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

***

[`for` iteration statements]: https://tc39.github.io/ecma262/#sec-iteration-statements
[`in` relational operator]: https://tc39.github.io/ecma262/#sec-relational-operators
[annevk]: https://github.com/annevk
[antecedent]: https://en.wikipedia.org/wiki/Antecedent_(grammar)
[associative property]: https://en.wikipedia.org/wiki/Associative_property
[bare style]: #bare-style
[bidirectional associativity]: #bidirectional-associativity
[binding]: https://en.wikipedia.org/wiki/Binding_(linguistics)
[Clojure pipe]: https://clojuredocs.org/clojure.core/as-%3E
[concatenative programming]: https://en.wikipedia.org/wiki/Concatenative_programming_language
[cyclomatic complexity]: https://en.wikipedia.org/wiki/Cyclomatic_complexity#Applications
[Daniel “littledan” Ehrenberg of Igalia]: https://github.com/littledan
[dataflow programming]: https://en.wikipedia.org/wiki/Dataflow_programming
[ECMAScript _IdentifierName_]: https://tc39.github.io/ecma262/#prod-IdentifierName
[ECMAScript _IdentifierReference_]: https://tc39.github.io/ecma262/#prod-IdentifierReference
[ECMAScript _MemberExpression_]: https://tc39.github.io/ecma262/#prod-MemberExpression
[ECMAScript Assignment-level Expressions]: https://tc39.github.io/ecma262/#sec-assignment-operators
[ECMAScript Function Calls, § RS: Evaluation]: https://tc39.github.io/ecma262/#sec-function-calls-runtime-semantics-evaluation
[ECMAScript Lexical Grammar]: https://tc39.github.io/ecma262/#sec-ecmascript-language-lexical-grammar
[ECMAScript LHS expressions]: https://tc39.github.io/ecma262/#sec-left-hand-side-expressions
[ECMAScript Lists and Records]: https://tc39.github.io/ecma262/#sec-list-and-record-specification-type
[ECMAScript Notational Conventions, § Grammars]: https://tc39.github.io/ecma262/#sec-syntactic-and-lexical-grammars
[ECMAScript Notational Conventions, § Lexical Grammar]: https://tc39.github.io/ecma262/#sec-lexical-and-regexp-grammars
[ECMAScript Primary Expressions]: https://tc39.github.io/ecma262/#prod-PrimaryExpression
[ECMAScript Property Accessors, § RS: Evaluation]: https://tc39.github.io/ecma262/#sec-property-accessors-runtime-semantics-evaluation
[ECMAScript Punctuators]: https://tc39.github.io/ecma262/#sec-punctuators
[ECMAScript static semantic rules]: https://tc39.github.io/ecma262/#sec-static-semantic-rules
[Elixir pipe]: https://elixir-lang.org/getting-started/enumerables-and-streams.html
[Elm pipe]: http://elm-lang.org/docs/syntax#infix-operators
[expressions and operators (MDN)]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Expressions_and_Operators
[F# pipe]: https://docs.microsoft.com/en-us/dotnet/fsharp/language-reference/functions/index#function-composition-and-pipelining
[Fetch Standard]: https://fetch.spec.whatwg.org
[formal grammar]: #grammar
[functional programming]: https://en.wikipedia.org/wiki/Functional_programming
[garden-path syntax]: https://en.wikipedia.org/wiki/Garden_path_sentence
[GitHub issue tracker]: https://github.com/tc39/proposal-pipeline-operator/issues
[goals]: #goals
[grammar parameters]: #grammar-parameters
[Hack pipe]: https://docs.hhvm.com/hack/operators/pipe-operator
[inner blocks]: #inner-blocks
[jashkenas]: https://github.com/jashkenas
[Julia pipe]: https://docs.julialang.org/en/stable/stdlib/base/#Base.:|>
[lexical grammar]: #lexical-grammar
[lexically hygienic]: https://en.wikipedia.org/wiki/Hygienic_macro
[littledan invitation]: https://github.com/tc39/proposal-pipeline-operator/issues/89#issuecomment-363853394
[LiveScript pipe]: http://livescript.net/#operators-piping
[MDN operator precedence]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Expressions_and_Operators
[mindeavor]: https://github.com/gilbert
[Node-stream piping]: https://nodejs.org/api/stream.html#stream_readable_pipe_destination_options
[nomenclature]: #nomenclature
[object initializers’ Computed Property Contains rule]: https://tc39.github.io/ecma262/#sec-object-initializer-static-semantics-computedpropertycontains
[OCaml pipe]: http://blog.shaynefletcher.org/2013/12/pipelining-with-operator-in-ocaml.html
[operator precedence]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Operator_Precedence
[Perl 6 pipe]: https://docs.perl6.org/language/operators#infix_==&gt;
[Perl 6 topicalization]: https://www.perl.com/pub/2002/10/30/topic.html/
[Pify]: https://github.com/sindresorhus/pify
[pipeline syntax]: #pipeline-syntax
[possible future extensions to the topic concept]: #possible-future-extensions-to-topic-concept
[previous pipeline-placeholder discussions]: https://github.com/tc39/proposal-pipeline-operator/issues?q=placeholder
[first pipe-operator proposal]: https://github.com/tc39/proposal-pipeline-operator/blob/37119110d40226476f7af302a778bc981f606cee/README.md
[private class fields]: https://github.com/tc39/proposal-class-fields/
[Proposal 4: Smart Mix on the pipe-proposal wiki]: https://github.com/tc39/proposal-pipeline-operator/wiki#proposal-4-smart-mix
[R pipe]: https://cran.r-project.org/web/packages/magrittr/index.html
[relations to other work]: #relations-to-other-work
[REPLs]: https://en.wikipedia.org/wiki/Read–eval–print_loop
[resolving topics]: #resolve-topic
[reverse Polish notation]: https://en.wikipedia.org/wiki/Reverse_Polish_notation
[runtime semantics]: #runtime-semantics
[sindresorhus]: https://github.com/sindresorhus
[smart body syntax]: #smart-body-syntax
[syntactic partial application]: https://github.com/tc39/proposal-partial-application
[tacit programming]: https://en.wikipedia.org/wiki/Tacit_programming
[TC39 process]: https://tc39.github.io/process-document/
[term rewriting topical style]: #term-rewriting-topical-style
[term rewriting with autogenerated variables]: #term-rewriting-with-single-dummy-variable
[term rewriting with autogenerated variables]: #term-rewriting-with-single-dummy-variable
[term rewriting with single dummy variable]: #term-rewriting-with-single-dummy-variable
[term rewriting]: https://en.wikipedia.org/wiki/Term_rewriting
[topic and comment]: https://en.wikipedia.org/wiki/Topic_and_comment
[topic variables in other languages]: https://rosettacode.org/wiki/Topic_variable
[topic-token bikeshedding]: https://github.com/tc39/proposal-pipeline-operator/issues/91
[Underscore.js]: http://underscorejs.org
[Unix pipe]: https://en.wikipedia.org/wiki/Pipeline_(Unix
[WHATWG-stream piping]: https://streams.spec.whatwg.org/#pipe-chains

[optional-chaining syntax proposal]: https://github.com/tc39/proposal-optional-chaining
[“data-to-ink” visual ratio]: https://www.darkhorseanalytics.com/blog/data-looks-better-naked
[topical style]: #topical-style
[motivation]: #motivation
[Huffman coding]: https://en.wikipedia.org/wiki/Huffman_coding
[ECMAScript `new` operator, § RS: Evaluation]: https://tc39.github.io/ecma262/#sec-new-operator-runtime-semantics-evaluation
[ECMAScript pattern matching]: https://github.com/tc39/proposal-pattern-matching
[ECMAScript block parameters]: https://github.com/samuelgoto/proposal-block-params
[ECMAScript optional catch binding]: https://github.com/tc39/proposal-optional-catch-binding
[`new.target`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/new.target
[DSLs]: https://en.wikipedia.org/wiki/Domain-specific_language
[Tennent correspondence principle]: http://gafter.blogspot.com/2006/08/tennents-correspondence-principle-and.html
[IIFEs]: https://en.wikipedia.org/wiki/Immediately-invoked_function_expression
[PEP 20]: https://www.python.org/dev/peps/pep-0020/
[incidental complexity]: https://en.wikipedia.org/wiki/Incidental_complexity
[essential complexity]: https://en.wikipedia.org/wiki/Essential_complexity
[examples]: #examples
[ECMAScript arrow functions, § SS: Contains]: https://tc39.github.io/ecma262/#sec-arrow-function-definitions-static-semantics-contains
[footguns]: https://en.wiktionary.org/wiki/footgun
[early errors]: #static-early-errors
[Contains]: #static-contains
