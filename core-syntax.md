# Core-proposal syntax
Most pipeline steps will use topic references in their steps. This style of
pipeline step is called **[topic style][]**.

For three simple cases – unary functions, unary async functions, and unary
constructors – you may omit the topic reference from the pipeline step. This is
called **[bare style][]**.

When a pipe is in bare style, we refer to the pipeline as a **bare function
call**. (If [Additional Feature BC][] or [Additional Feature BA][] are used,
then a bare-style pipeline step may instead be a **bare awaited function call**,
or a **bare constructor call**, depending on the rules of bare style.) The step
acts as just a simple reference to a function, such as with `… |> capitalize` or
with `… |> console.log`. The step’s value would then be called as a unary
function, without having to use the topic reference as an explicit argument.

The two bare-style productions require no parameters, because they can only be
**simple references**, made up of identifiers and dots `.`. (If [Additional
Feature BC][] is used, then the simple reference may optionally be preceded by
`new`: `… |> new o.C`. If [Additional Feature BA][] is used, then the simple
reference may optionally be preceded by `await`: `… |> new o.af`. Even with
Additional Features BC or BA, `new` and `await` may not be used on their own
without a simple reference: `… |> o.C |> new` 🚫 and `… |> o.af |> await` 🚫 are
invalid pipelines. Instead, use either the bare style `… |> new o.C` and
`… |> await o.af`, or use topic style: `… |> af |> await #`.

With the [Core Proposal][] only:

| Valid [topic style][]   | Valid [bare style][]                     | Invalid pipeline
| ----------------------- | ---------------------------------------- | --------------------
|`… \|> f(#)`             |`… \|> f`                                 |  `… \|> f()` 🚫
| ″″                      | ″″                                       | `… \|> (f)` 🚫
| ″″                      | ″″                                       | `… \|> (f())` 🚫
|`… \|> o.f(#)`           |`… \|> o.f`                               | `… \|> o.f()` 🚫
|`… \|> o.f(arg, #)`      |`const f = $ => o::f(arg, $); … \|> f`    | `… \|> o.f(arg)` 🚫
|`… \|> o.make()(#)`      |`const f = o.make(); … \|> f`             | `… \|> o.make()` 🚫
|`… \|> o[symbol](#)`     |`const f = o[symbol]; … \|> f`            | `… \|> o[symbol]` 🚫

With [Additional Feature BC][]:

| Valid [topic style][]   | Valid [bare style][]                     | Invalid pipeline
| ----------------------- | ---------------------------------------- | --------------------
|`… \|> new C(#)`         |`… \|> new C`                             | `… \|> new C()` 🚫
| ″″                      | ″″                                       | `… \|> (new C)` 🚫
| ″″                      | ″″                                       | `… \|> (new C())` 🚫
| ″″                      | ″″                                       | `… \|> new (C)` 🚫
| ″″                      | ″″                                       | `… \|> new (C())` 🚫
|`… \|> new o.C(#)`       |`… \|> new o.C`                           | `… \|> new o.f()` 🚫
|`… \|> new o.C(arg, #)`  |`const f = $ => new o::C(arg, $); … \|> f`| `… \|> new o.C(arg)` 🚫
|`… \|> new o.make()(#)`  |`const C = o.make(); … \|> new C`         | `… \|> new o.make()` 🚫
|`… \|> new o[symbol](#)` |`const f = new o[symbol]; … \|> f`        | `… \|> new o[symbol]` 🚫
|`… \|> await new o.make()(#)`|`const af = new o.make(); … \|> await af`| `… \|> new await o.make()` 🚫

With [Additional Feature BA][]:

| Valid [topic style][]   | Valid [bare style][]                     | Invalid pipeline
| ----------------------- | ---------------------------------------- | --------------------
|`… \|> await af(#)`      |`… \|> await af`                          | `… \|> await af()` 🚫
| ″″                      | ″″                                       | `… \|> (await f)` 🚫
| ″″                      | ″″                                       | `… \|> (await f())` 🚫
| ″″                      | ″″                                       | `… \|> await (f)` 🚫
| ″″                      | ″″                                       | `… \|> await (f())` 🚫
|`… \|> af \|> await #`   | ″″                                       |  `… \|> af \|> await` 🚫
|`… \|> await o.f(#)`     |`… \|> await o.f`                         | `… \|> await o.f()` 🚫
|`… \|> await o.make()(#)`|`const af = o.make(); … \|> await af`     | `… \|> await o.make()` 🚫
|`… \|> await new o.make()(#)`|`const af = new o.make(); … \|> await af`| `… \|> new await o.make()` 🚫

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

**With [Additional Feature BC][]:**\
If a pipeline step starts with `new`, followed by a mere identifier, optionally
with a chain of properties, and with no parentheses or brackets, then that
identifier is interpreted to be a **bare constructor**.

That is: **if a pipeline** is of the form\
**_topic_ `|>` `new` _identifier_**\
or **_topic_ `|>` `new` _identifier0_`.`_identifier1_**\
or **_topic_ `|>` `new` _identifier0_`.`_identifier1_`.`_identifier2_**\
or so forth,\
then the pipeline is a bare constructor call.

**With [Additional Feature BA][]:**\
If a pipeline step starts with `await`, followed by a mere identifier, optionally with
a chain of properties, and with no parentheses or brackets, then that identifier
is interpreted to be a **bare awaited function call**.

That is: **if a pipeline** is of the form\
**_topic_ `|>` `await` _identifier_**\
or **_topic_ `|>` `await` _identifier0_`.`_identifier1_**\
or **_topic_ `|>` `await` _identifier0_`.`_identifier1_`.`_identifier2_**\
or so forth,\
then the pipeline is a bare async function call.

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

**With [Additional Feature BA][]:**\
A pipeline step that is a block `{` … `}` containing a list of statements is a
**topic block**. The last statement in the block is used as the result of the
whole pipeline, similarly to [`do` expressions][].

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
