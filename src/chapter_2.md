# chapter 2: basic syntax

A valid ataraxia program source is [UTF-8](https://en.wikipedia.org/wiki/UTF-8)
encoded text consisting of a series of one or more **declarations**.

## declarations

A declaration looks like this:

```rust
foo : Type := \{ ...expression goes here... \}
```

Breaking this down by parts from left to right:

`foo` is an identifier that names our declaration. You can then reference `foo`
elsewhere in the program by that same name.

The `:` is a symbol from type theory (and many programming languages like Rust
and Idris) that means "has type". All top-level declarations like the one we
have here *must* specify a type like this.

The `Type` is just a dummy name for a possible type signature.

The `:=` is a symbol originating with the programming languages ALGOL and
Pascal (sometimes also used in mathematics) which indicates **assignment**.
Many common languages just use a single equals sign, like `=` (think
JavaScript, Java, Python, C/C++/C#, Haskell, etc.), but in ataraxia we use `=`
for *mathematical equality*.

Finally, the `\{ ...expression goes here... \}` is a (possibly multi-line)
comment that I've used as a placeholder for what actually *should* be there: an
**expression**. Single-line comments are done using a single backslash (`\`).

## expressions

An expression can be one of three things: a **literal**, a single bare
**identifier**, or a **block**.

### literals

#### booleans

ataraxia supports C++/Rust-style boolean literals: either `true` or `false`.

#### integers

ataraxia supports three formats for integer literals.

The most common one is just decimal, like `121` which stands for
\\( 121_{10} \equiv 11^2 \\). Leading `0`s are allowed and do not change
anything, so `0000121` is the same thing as `121`. You can also put underscores
inside of these literals, like `1_867_098`.

The second one is hexadecimal, which is the same as the decimal ones described
above, except that they have an extra `0x` (or `0X`) at the beginning and
support hexadecimal digits (`[0-9a-fA-F]`), like so: `0xDEADBEEF`.

The third one is binary, which is like hexadecimal except instead of an `x`
(or `X`), you use a `b` (or `B`), and there are only `0`s and `1`s allowed,
like so: `0b1001110110101`.

Any integer literal can be made negative by simply prepending a minus sign, but
note that the minus sign must be directly adjacent to the beginning of the rest
of the literal. `-121` works as a literal, but `- 121` is two separate things
(an identifier `-`, and then a positive integer literal `121`).

#### floats

ataraxia supports syntax for float literals essentially identical to that found
in other languages, but allows underscores inside just like with integer
literals.

This includes an optional exponent at the end, like `-5.364327e-5` which stands
for \\( -5.364327 \cdot 10^{-5} \\).

For special floating point values, there are special literals. `NaN` is
[not a number](https://en.wikipedia.org/wiki/NaN), `inf` is positive infinity,
and `-inf` is negative infinity.

#### characters

ataraxia allows single-quotes (`'`) in identifiers, so for character literals a
`#` prefix is required, like so: `#'🎺'`. The content between the single-quotes
can be any single valid UTF-8 encoded scalar value, or a backslash followed by
an escape character. Currently supported escape characters:

- `\` \\( \rightarrow \\) backslash
- `'` \\( \rightarrow \\) single-quote
- `"` \\( \rightarrow \\) double-quote
- `n` \\( \rightarrow \\) newline/linefeed (LF)
- `r` \\( \rightarrow \\) carriage return (CR)
- `t` \\( \rightarrow \\) horizontal tab
- `0` \\( \rightarrow \\) null character

#### strings

ataraxia supports double-quoted strings, most similar to string literals in
[C](https://en.wikipedia.org/wiki/C_(programming_language)). The content
between the double-quoted strings can be zero or more of the possible contents
of character literals (UTF-8 encoded scalar value/backslash followed by an
escape character), e.g. `""`, `"the band: 🎹 🎺 🎷 🥁 🎸\n"`.

#### lists

ataraxia has a built-in list type (most similar to "vectors" in
[C++](http://en.cppreference.com/w/cpp/container/vector) and
[Rust](https://doc.rust-lang.org/std/vec/struct.Vec.html)), which allows for
literals to be constructed in-place.

A list literal is delimited by square brackets (`[` ... `]`), and contains zero
or more expressions (of any single type, including other lists!) separated by
whitespace. Note that lists have an element type, so for example `[12 "12"]`
is not valid (integers are not strings), but `["" "12"]` is, and so are
`[12 0]` and `[]`.

### identifiers

Identifiers in ataraxia are allowed to contain any ASCII characters except for
whitespace (SPACE, `\t`, `\n`, `\r`), parentheses/brackets/braces (`(`, `)`,
`[`, `]`, `{`, `}`), backslashes (`\`), and non-printable characters.

Identifiers also cannot *begin* with a dash (`-`), although they may otherwise
contain dashes.

### blocks

Blocks are delimited by curly brackets (`{` ... `}`) and contain zero or more
expressions, separated by whitespace.

The semantic significance of blocks is that they represent a sequence of
*things to do*, essentially a
[routine/function](https://en.wikipedia.org/wiki/Subroutine) body. Since blocks
are expressions, this basically means that functions are first-class values.
The expressions in a block are evaluated in order (left to right, top to
bottom).

A block with no expressions is essentially a
[no-op](https://en.wikipedia.org/wiki/NOP), and a block with a single
expression is essentially a way of "quoting" that expression (`+` adds the top
two elements of the stack, `{+}` pushes the addition function itself as its own
value onto the stack).

## type signatures

Every top-level declaration must have a type signature. Type signatures contain
three different kinds of tokens: type identifiers, arrows, and parentheses.

Type identifiers (type ids) are a more restrictive kind of identifier than
usual ones that name expressions. Type ids *must* begin with an uppercase ASCII
letter (`A-Z`), followed by zero or more ASCII letters (of any case), digits
(`0-9`), underscores (`_`), single-quotes (`'`), and dashes (`-`). A single,
bare type id is a valid type signature.

Arrows are denoted by `->`, and denote functions. All function type signatures
are of the form `In1 In2 ... -> Out1 Out2 ...`, where `In1 In2 ...` is a
whitespace-delimited list of valid type signatures for the input values, and
`Out1 Out2 ...` is a whitespace-delimited list of valid type signatures for the
output values.

Parentheses are there so that function type signatures can be nested (e.g.
`A (B -> C) -> D`), and also so that the unit type can be expressed (`()`). The
unit type represents an empty list of type signatures and thus has no
meaningful values (i.e. it has zero size in memory).
