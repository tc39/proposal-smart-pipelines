# Core-proposal syntax
ECMAScript Stage-0 Proposal. Living Document. J. S. Choi, 2018-12.

Most pipeline steps will use topic references in their steps. This style of
pipeline step is called **[topic style][]**.

For three simple cases – unary functions, unary async functions, and unary
constructors – you may omit the topic reference from the pipeline step. This is
called **[bare style][]**.

When a pipe is in bare style, we refer to the pipeline as a **bare function
call**.  The step acts as just a simple reference to a function, such as with `… |> capitalize` or
with `… |> console.log`. The step’s value would then be called as a unary
function, without having to use the topic reference as an explicit argument.

The two bare-style productions require no parameters, because they can only be
**simple references**, made up of identifiers and dots `.`. (There are optional
[Additional Features](./readme.md#additional-features) that extend these style in
limited ways for further convenience. These Additional Features are not part of this
Core Proposal but rather show possible future extensions.)

| Valid [topic style][]   | Valid [bare style][]                     | Invalid pipeline
| ----------------------- | ---------------------------------------- | --------------------
|`… \|> f(#)`             |`… \|> f`                                 |  `… \|> f()` 🚫
| ″″                      | ″″                                       | `… \|> (f)` 🚫
| ″″                      | ″″                                       | `… \|> (f())` 🚫
|`… \|> o.f(#)`           |`… \|> o.f`                               | `… \|> o.f()` 🚫
|`… \|> o.f(arg, #)`      |`const f = $ => o::f(arg, $); … \|> f`    | `… \|> o.f(arg)` 🚫
|`… \|> o.make()(#)`      |`const f = o.make(); … \|> f`             | `… \|> o.make()` 🚫
|`… \|> o[symbol](#)`     |`const f = o[symbol]; … \|> f`            | `… \|> o[symbol]` 🚫

## Bare style
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

## Topic style
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

## Practical consequences
Therefore, a pipeline step in **[bare style][] *never*** contains **parentheses `(…)` or
brackets `[…]`**. Neither `… |> object.method()` nor
`… |> object.method(arg)` nor `… |> object[symbol]` nor `… |> object.createFunction()`
are in bare style (in fact, they all are Syntax Errors, due to their being in
[topic style][] without any topic references).

**When a pipeline step needs parentheses or brackets**, then **don’t use bare style**,
and instead **use a topic reference** in the step ([topic style][])…or **assign
the step to a variable**, then **use that variable as a bare-style step**.

# Operator precedence and associativity
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
        