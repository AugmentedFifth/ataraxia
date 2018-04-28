# chapter 5: built-in entities

ataraxia comes with a handful of things that are so core to the language that
they are built in and do not even require the standard library. There are also
a number of things that are implemented using intrinsics/special-sauce
functions built in to the reference interpreter for ataraxia bytecode (`*.abc`,
as emitted by [ataraxiac](https://gitlab.com/AugmentedFourth/ataraxiac)).

## core entities

### literal entities

All types that are ordinarily inhabited by [literals](chapter_2.html#literals)
are core types. This includes the following types:

- `I8` (signed 8-bit integers)
- `I32` (signed 32-bit integers)
- `I64` (signed 64-bit integers)
- `F32` (32-bit floating point numbers)
- `F64` (64-bit floating point numbers)
- `Char` (Unicode scalar values, UTF-8 encoded)
- `Str` (UTF-8 encoded string)
- `List<a>` (dynamic array of elements of type `a`)

#### `I8`, `I32`, `I64`, `F32`, and `F64`: the numerics

All of these types have built-in core arithmetic functions for addition (`+`),
subtraction (`-`), multiplication (`*`), division (`/`, ordinary truncating
integer division for integer types), remainder (`%`), and negation (`neg`). You
generally don't want to use these core functions, since they have ugly
overly-specific names like `#+32` for `I32` addition and `#/f64` for `F64`
division. Instead, you will want to use the `+` and `/` functions from the
standard library, respectively, which are polymorphic and work for any of these
numeric types. Similar remarks apply to the core increment/decrement functions,
which look more like `#inc64` for incrementing an `I64`, but which have nicer
analogues in the standard library (`inc`, `dec`). Note that floating point
numbers do not have core increment and decrement functions and so do not
implement `inc` and `dec`, which are part of the `Integral` standard library
typeclass.

| **name**  | **description**                            | **type**         |
| ---------:|:------------------------------------------ |:---------------- |
| `#+8`     | Adds together two `I8`s.                   | `I8 I8 -> I8`    |
| `#-8`     | Subtracts two `I8`s.                       | `I8 I8 -> I8`    |
| `#*8`     | Takes the product of two `I8`s.            | `I8 I8 -> I8`    |
| `#/8`     | Takes the integral quotient of two `I8`s.  | `I8 I8 -> I8`    |
| `#%8`     | Takes the remainder of two `I8`s.          | `I8 I8 -> I8`    |
| `#neg8`   | Takes the negation of an `I8`.             | `I8 -> I8`       |
| `#inc8`   | Increments an `I8`.                        | `I8 -> I8`       |
| `#dec8`   | Decrements an `I8`.                        | `I8 -> I8`       |
| `#+32`    | Adds together two `I32`s.                  | `I32 I32 -> I32` |
| `#-32`    | Subtracts two `I32`s.                      | `I32 I32 -> I32` |
| `#*32`    | Takes the product of two `I32`s.           | `I32 I32 -> I32` |
| `#/32`    | Takes the integral quotient of two `I32`s. | `I32 I32 -> I32` |
| `#%32`    | Takes the remainder of two `I32`s.         | `I32 I32 -> I32` |
| `#neg32`  | Takes the negation of an `I32`.            | `I32 -> I32`     |
| `#inc32`  | Increments an `I32`.                       | `I32 -> I32`     |
| `#dec32`  | Decrements an `I32`.                       | `I32 -> I32`     |
| `#+64`    | Adds together two `I64`s.                  | `I64 I64 -> I64` |
| `#-64`    | Subtracts two `I64`s.                      | `I64 I64 -> I64` |
| `#*64`    | Takes the product of two `I64`s.           | `I64 I64 -> I64` |
| `#/64`    | Takes the integral quotient of two `I64`s. | `I64 I64 -> I64` |
| `#%64`    | Takes the remainder of two `I64`s.         | `I64 I64 -> I64` |
| `#neg64`  | Takes the negation of an `I64`.            | `I64 -> I64`     |
| `#inc64`  | Increments an `I64`.                       | `I64 -> I64`     |
| `#dec64`  | Decrements an `I64`.                       | `I64 -> I64`     |
| `#+f32`   | Adds together two `F32`s.                  | `F32 F32 -> F32` |
| `#-f32`   | Subtracts two `F32`s.                      | `F32 F32 -> F32` |
| `#*f32`   | Takes the product of two `F32`s.           | `F32 F32 -> F32` |
| `#/f32`   | Takes the quotient of two `F32`s.          | `F32 F32 -> F32` |
| `#%f32`   | Takes the remainder of two `F32`s.         | `F32 F32 -> F32` |
| `#negf32` | Takes the negation of an `F32`.            | `F32 -> F32`     |
| `#+f64`   | Adds together two `F64`s.                  | `F64 F64 -> F64` |
| `#-f64`   | Subtracts two `F64`s.                      | `F64 F64 -> F64` |
| `#*f64`   | Takes the product of two `F64`s.           | `F64 F64 -> F64` |
| `#/f64`   | Takes the quotient of two `F64`s.          | `F64 F64 -> F64` |
| `#%f64`   | Takes the remainder of two `F64`s.         | `F64 F64 -> F64` |
| `#negf64` | Takes the negation of an `F64`.            | `F64 -> F64`     |

These types also all have core coercion functions that allow you to cast a
value of one type to another, like to get an `F64` from an `F32`, or an `F32`
from an `I32`. Again, the actual core functions (`#f32f64` and `#i32f32`,
respectively) are pretty ugly, but the standard library provides nicer
functions like `i2i` for integer to integer casting, `i2f` for integer to float
casting, etc.

| **name**  | **description**                | **type**     |
| ---------:|:------------------------------ |:------------ |
| `#i8i32`  | Converts an `I8` to an `I32`.  | `I8 -> I32`  |
| `#i8i64`  | Converts an `I8` to an `I64`.  | `I8 -> I64`  |
| `#i32i8`  | Converts an `I32` to an `I8`.  | `I32 -> I8`  |
| `#i32i64` | Converts an `I32` to an `I64`. | `I32 -> I64` |
| `#i64i8`  | Converts an `I64` to an `I8`.  | `I64 -> I8`  |
| `#i64i32` | Converts an `I64` to an `I32`. | `I64 -> I32` |
| `#i32f32` | Converts an `I32` to an `F32`. | `I32 -> F32` |
| `#i64f32` | Converts an `I64` to an `F32`. | `I64 -> F32` |
| `#i32f64` | Converts an `I32` to an `F64`. | `I32 -> F64` |
| `#i64f64` | Converts an `I64` to an `F64`. | `I64 -> F64` |
| `#f32i32` | Converts an `F32` to an `I32`. | `F32 -> I32` |
| `#f32i64` | Converts an `F32` to an `I64`. | `F32 -> I64` |
| `#f64i32` | Converts an `F64` to an `I32`. | `F64 -> I32` |
| `#f64i64` | Converts an `F64` to an `I64`. | `F64 -> I64` |

These types **also** all have core comparison functions that allow comparing
for equality (`=`), inequality (`!=`), less than (`<`), greater than (`>`),
less than or equal (`<=`), and greater than or equal (`>=`). There are only two
types of core functions for this: equality tests (`#eqtest64` compares 64-bit
integers for equality), and comparison tests (`#cmpf32` compares two `F32`s).
Equality tests are of course used to implement `=` and `!=` in the standard
library, while comparison tests are used to implement the other four.

| **name**    | **description**                                                                                                   | **type**                |
| -----------:|:----------------------------------------------------------------------------------------------------------------- |:----------------------- |
| `#eqtest8`  | Compares two `I8`s, generating `1` if they are equal, and `0` otherwise.                                          | `I8 I8 -> I8 I8 I8`     |
| `#eqtest32` | Compares two `I32`s, generating `1` if they are equal, and `0` otherwise.                                         | `I32 I32 -> I32 I32 I8` |
| `#eqtest64` | Compares two `I64`s, generating `1` if they are equal, and `0` otherwise.                                         | `I64 I64 -> I64 I64 I8` |
| `#cmp8`     | Compares two `I8`s, generating `-1` if the top is greater than the bottom, `1` if it is less, and `0` otherwise.  | `I8 I8 -> I8 I8 I8`     |
| `#cmp32`    | Compares two `I32`s, generating `-1` if the top is greater than the bottom, `1` if it is less, and `0` otherwise. | `I32 I32 -> I32 I32 I8` |
| `#cmp64`    | Compares two `I64`s, generating `-1` if the top is greater than the bottom, `1` if it is less, and `0` otherwise. | `I64 I64 -> I64 I64 I8` |
| `#isnan32`  | Checks if an `F32` value is `NaN`, generating `1` if the answer is "yes", and `0` otherwise.                      | `F32 -> F32 I8`         |
| `#isnan64`  | Checks if an `F64` value is `NaN`, generating `1` if the answer is "yes", and `0` otherwise.                      | `F64 -> F64 I8`         |

#### `Char`

`Char` comes equipped with the following core functions:

| **name**    | **description**                                 | **type**      |
| -----------:|:----------------------------------------------- |:------------- |
| `#pushchar` | Pushes a character to the stack.                | `() -> Char`  |
| `#i32char`  | Converts an `I32` to the bit-equivalent `Char`. | `I32 -> Char` |
| `#chari32`  | Converts an `Char` to the bit-equivalent `I32`. | `Char -> I32` |

#### `Str`

`Str` comes equipped with the following core functions:

| **name**            | **description**                                                                                                                          | **type**                 |
| -------------------:|:---------------------------------------------------------------------------------------------------------------------------------------- |:------------------------ |
| `#newstr`           | Creates a new empty `Str` with a specified capacity (in bytes) and pushes a pointer to it onto the stack.                                | `I64 -> Str`             |
| `#strpush`          | Pushes a `Char` from the stack onto the `Str` pointed to by the pointer below it.                                                        | `Str Char -> Str`        |
| `#strpop`           | Pops a `Char` off of the `Str` pointed to by the stack and pushes it onto the stack.                                                     | `Str -> Str Char`        |
| `#lenstr`           | Gets the length of a `Str` in bytes, as an `I64`.                                                                                        | `Str -> Str I64`         |
| `#getbytestr`       | Gets the byte in a `Str` indexed by an `I64` on the stack.                                                                               | `Str I64 -> Str I8`      |
| `#rmcharstr`        | Removes a `Char` from a `Str` and places it on the stack. The `Char` starts at a byte index indicated by an `I64`.                       | `Str I64 -> Str Char`    |
| `#inscharstr`       | Similar to `#rmcharstr`, but inserts a `Char` from the stack.                                                                            | `Str Char I64 -> Str`    |
| `#rmrangestr`       | Removes the bytes from indices specified by two `I64`s from the stack, representing a range (c.f. Python's slice syntax, e.g. `s[3:7]`). | `Str I64 I64 -> Str`     |
| `#catstr`           | Inserts the string data from one `Str` into another.                                                                                     | `Str Str -> Str`         |
| `#catstrix`         | Copies the string data from one `Str` into another, starting at a given byte index into the one being inserted into.                     | `Str Str I64 -> Str Str` |
| `#writecharstr`     | Directly writes a `Char` from the stack into a byte index of a `Str`.                                                                    | `Str Char I64 -> Str`    |
| `#clearstr`         | Clears the contents of a `Str`.                                                                                                          | `Str -> Str`             |
| `#byteixstr`        | Gets the byte index corresponding to a given `Char` index into a `Str`.                                                                  | `Str I64 -> Str I64`     |
| `#nextcharboundstr` | Gets the byte index of the next `Char` in a `Str`, starting at a given byte index.                                                       | `Str I64 -> Str I64`     |

#### `List<a>`

`List<a>` comes equipped with the following core functions:

| **name**       | **description**                                                                                                                         | **type**                     |
| --------------:|:--------------------------------------------------------------------------------------------------------------------------------------- |:---------------------------- |
| `#newlist`     | Creates a new empty `List<a>` with a specified capacity and pushes a pointer to it onto the stack.                                      | `I64 -> List<a>`             |
| `#listpush`    | Pushes an `a` from the stack onto the `List<a>` pointed to by the pointer below it.                                                     | `List<a> a -> List<a>`       |
| `#listpop`     | Pops an `a` off of the `List<a>` pointed to by the stack and pushes it onto the stack.                                                  | `Llist<a> -> List<a> a`      |
| `#lenlist`     | Gets the length of a `List<a>`, as an `I64`.                                                                                            | `List<a> -> List<a> I64`     |
| `#getlist`     | Gets the `a` in a `List<a>` indexed by an `I64` on the stack.                                                                           | `List<a> I64 -> List<a> a`   |
| `#rmlist`      | Removes an `a` from a `List<a>` and places it on the stack. The element is indexed by an `I64`.                                         | `List<a> I64 -> List<a> a`   |
| `#inslist`     | Similar to `#rmlist`, but inserts an `a` from the stack.                                                                                | `List<a> a I64 -> List<a>`   |
| `#rmrangelist` | Removes the `a`s from indices specified by two `I64`s from the stack, representing a range (c.f. Python's slice syntax, e.g. `l[3:7]`). | `List<a> I64 I64 -> List<a>` |
| `#catlist`     | Inserts the list data from one `List<a>` into another.                                                                                  | `List List -> List`          |
| `#catlistix`   | Copies the list data from one `List<a>` into another, starting at a given index into the one being inserted into.                       | `List List I64 -> List List` |
| `#writelist`   | Directly writes an `a` from the stack into an index of a `List<a>`.                                                                     | `List<a> a I64 -> List<a>`   |
| `#clearlist`   | Clears the contents of a `List<a>`.                                                                                                     | `List<a> -> List<a>`         |

### pointers and the `Stack` and `Heap` type constraints

Pointers are not values that you interact with directly in ataraxia. Instead,
ataraxia essentially decides for you if something goes directly on the stack
or if it is *represented* as being on the stack by what is actually a pointer
value that points to the "real thing" on the heap.

However, if you wish to have extra control over the inner workings of how
values of *your* type are handled in memory, you can gain some by explicitly
implementing `Heap` for your type (forcing your type to be represented by a
pointer to where it is allocated on the heap) or implementing `Stack` for it
(forcing your type to be actually stored on the stack). These type constraints
have no associated functions, and the compiler will automatically implement one
or the other for your type if you do not.

The representation of pointers, we leave unspecified; pointers must thus be
treated as opaque values.

However, there are a few core functions associated with "pointers" (really, on
the ataraxia-semantics-level, heap types; so these functions work on any types
that implement `Heap`, like `List<a>` for example):

| **name**                           | **description**                                                                                                                                                                                                                              | **type**                  |
| ----------------------------------:|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |:------------------------- |
| `#freeptr`                         | Frees the memory associated with a pointer. To be used with incredible wisdom, although it is not particularly unsafe unless combined with other evil operators.                                                                             | `Heap t => t -> ()`       |
| `#aliasptr`                        | Aliases a pointer; essentially the `dup` function except that it *only copies the pointer*. Again, not necessarily unsafe *per se*, but it is rumored that if you call this function directly, you can feel the devil's breath on your neck. | `Heap t => t -> t t`      |
| `#unsafeincrementpointerinplace`   | Increments a pointer by a specified number of bytes. **EXTREMELY UNSAFE USE WITH INCREDIBLE CAUTION**.                                                                                                                                       | `Heap t => t -> t`        |
| `#unsafedecrementpointerinplace`   | See above, but decrement instead. **EXTREMELY UNSAFE USE WITH INCREDIBLE CAUTION**.                                                                                                                                                          | `Heap t => t -> t`        |
| `#unsafewritedirectlytopointer`    | Writes the top value on the stack to the memory pointed to by the second-to-top value of the stack. **EXTREMELY UNSAFE USE WITH INCREDIBLE CAUTION**.                                                                                        | `Heap t => t u -> t u`    |
| `#unsafedonotusethistocoercetypes` | Changes the type of one `Heap` value to another `Heap` type. **EXTREMELY UNSAFE USE WITH INCREDIBLE CAUTION**.                                                                                                                               | `Heap t Heap u => t -> u` |

### panic

Another core function is called `#panic`, and it causes the program to panic
and immediately exit with an error. This may **or may not** involve performing
any "cleanup".

```haskell
#panic : _|_
```

The standard library exposes a function called `panic` which takes a `Str` as
an argument and prints the `Str` to `stderr` before calling `#panic`.

```haskell
panic : Str -> _|_
```

### bit twiddling and the `Repr` constraints

Of course, we need bit twiddling functions like "bitwise exclusive or" and
"bitwise and", right? Well there are some unsafe core functions that do just
that, and they are used to implement the bit twiddling operations in the
standard library (`^`, `&`, etc.) as well as the boolean operations (`xor`,
`and`).

These core functions operate over types that implement `Repr8`, `Repr32`, and
`Repr64`, which are type constraints that **cannot be implemented by the**
**user**. Instead, the compiler automatically implements them appropriately
when it knows that the type has an 8-bit, 32-bit, or 64-bit representation in
memory, respectively.

| **name** | **description**                                | **type**                 |
| --------:|:---------------------------------------------- |:------------------------ |
| `#<<8`   | Shift an 8-bit value leftward.                 | `Repr8 t => t I8 -> t`   |
| `#>>8`   | Shift an 8-bit value rightward.                | `Repr8 t => t I8 -> t`   |
| `#&8`    | Perform a bitwise "and" of two 8-bit values.   | `Repr8 t => t t -> t`    |
| `#|8`    | Perform a bitwise "or" of two 8-bit values.    | `Repr8 t => t t -> t`    |
| `#^8`    | Perform a bitwise "xor" of two 8-bit values.   | `Repr8 t => t t -> t`    |
| `#~8`    | Perform a bitwise negation of an 8-bit value.  | `Repr8 t => t -> t`      |
| `#<<32`  | Shift an 32-bit value leftward.                | `Repr32 t => t I32 -> t` |
| `#>>32`  | Shift an 32-bit value rightward.               | `Repr32 t => t I32 -> t` |
| `#&32`   | Perform a bitwise "and" of two 32-bit values.  | `Repr32 t => t t -> t`   |
| `#|32`   | Perform a bitwise "or" of two 32-bit values.   | `Repr32 t => t t -> t`   |
| `#^32`   | Perform a bitwise "xor" of two 32-bit values.  | `Repr32 t => t t -> t`   |
| `#~32`   | Perform a bitwise negation of an 32-bit value. | `Repr32 t => t -> t`     |
| `#<<64`  | Shift an 64-bit value leftward.                | `Repr64 t => t I64 -> t` |
| `#>>64`  | Shift an 64-bit value rightward.               | `Repr64 t => t I64 -> t` |
| `#&64`   | Perform a bitwise "and" of two 64-bit values.  | `Repr64 t => t t -> t`   |
| `#|64`   | Perform a bitwise "or" of two 64-bit values.   | `Repr64 t => t t -> t`   |
| `#^64`   | Perform a bitwise "xor" of two 64-bit values.  | `Repr64 t => t t -> t`   |
| `#~64`   | Perform a bitwise negation of an 64-bit value. | `Repr64 t => t -> t`     |

### invocation
