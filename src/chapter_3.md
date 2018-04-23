# chapter 3: basic semantics

## the stack

[A stack](https://en.wikipedia.org/wiki/Stack_\(abstract_data_type\)) is more
or less an array of items which can be dynamically grown and shrunk, and which
has a "bottom" and a "top". The primary (and often only) way of manipulating
a stack is from the top, generally by **push**ing values onto the top, and
**pop**ping values off of the top (you wouldn't eat a stack of pancakes from
the bottom, would you?).

In ataraxia *there are no named variables*. There are only global declarations
of single, immutable expressions that are written out in the source. Instead of
manipulating variables to get things done, you manipulate the stack.

ataraxia uses
[postfix notation](https://en.wikipedia.org/wiki/Reverse_Polish_notation)
(often referred to as "reverse polish notation", or RPN for short). That means
that in order to apply a function to its argument(s), you must *first* place
the arguments on the top of the stack (in the correct order, if it matters),
and *then* invoke the function, which will consume its arguments from off of
the stack and replace them with zero or more resulting values.

For example, to place the sum \\( 13 + 29 \\) onto the stack, we might do this
(nevermind the fact that you can just use a single literal):

```rust
13 29 +
```

When we write values that aren't naked functions (`+` is a naked function, as
opposed to a quoted one like `{+}`), they are implicitly **push**ed onto the
top of the stack. So if we were to do this in a
[REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop)
style where we print out the entire contents of the stack every time an
expression is entered, it would look like this (the bottom of the stack is on
the left-hand side):

```rust
>>>
\ empty stack
>>> 13
13
>>> 29
13 29
>>> +
42
```

We can also **pop** values off of the stack by explicitly invoking the `pop`
function:

```rust
>>> 13
13
>>> 29
13 29
>>> 121
13 29 121
>>> pop
13 29
>>> +
42
>>> pop
\ empty stack
```

There are other ways to manipulate the stack, too. In ataraxia, the stack is so
central that there are tons of ways to manipulate the stack, many of which you
would never do with an "ordinary stack". Here are a few:

- `dup` : Duplicate the value on top (`0 13` \\( \rightarrow \\) `0 13 13`).
- `swap`/`rot`/`rotr` : Switch the order of the top and second-to-top values.
- `rot3`/`rot4`/`rot5`/etc. : Rotate the top \\( n \\) values of the stack
  towards the bottom (`0 1 2 rot3` \\( \rightarrow \\) `1 2 0`).
- `rotr3`/`rotr4`/`rotr5`/etc. : Rotate the top \\( n \\) values of the stack
  towards the top (`0 1 2 rot3` \\( \rightarrow \\) `2 0 1`).
- `over`/`over3`/`over4`/etc. : Make a copy of the \\( n \\)th value from the
  top and push it onto the top (`over` = `over2`, the top value is the "1th"
  from the top).
- `drop`/`drop3`/`drop4`/etc. : Delete the \\( n \\)th value from the top
  (`drop[n]` = `rot[n] pop`).
