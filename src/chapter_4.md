# chapter 4: typing

ataraxia's type system can be thought of as a variant of the type system found
in the [Rust](https://www.rust-lang.org) language (or the
[Haskell](https://www.haskell.org/) language, if you happen to be familiar, but
Haskell's type system is much more expressive), but without any of the
substructural ownership/lifetime stuff. Just plain ol' values (Rust has the
notion of "pass by value", which is supposed to translate to "moving" values
except in the case of types that implement `Copy`).

There are only two sorts of types that really exist in ataraxia: base types,
and function types (this is vaguely reminiscient of a
[typed lambda calculus](https://en.wikipedia.org/wiki/Typed_lambda_calculus)
such as \\( \lambda^\rightarrow \\) or System \\( \text{F} \\)).

## type syntax

Let's take a close look at a simplified version of the (abstract!) type syntax
(simplified because we don't have quantification or type constraints yet; we'll
get there):

\\[
\begin{align}
    \tau   &::= \tau_B \mid \tau_f                           \\\\
    \tau_B &::= \text{T}                                     \\\\
    \tau_f &::= s_0 \rightarrow s_1                          \\\\
    s      &::= \tau_0 \\; \tau_1 \\; ... \\; \tau_n \mid () \\\\
\end{align}
\\]

Here, I use \\( \tau \\) to denote *any types* in general, \\( \tau_B \\) to
denote *base types*, \\( \tau_f \\) to denote *function types*, and \\( s \\)
to denote what I'm going to just call "stack types". \\( \text{T} \\) stands
for just a simple identifier, like `Bool` or `Nat`. Finally, \\( () \\) is the
*empty stack*, the "type" of... an empty stack.

So we see now that functions are from one "stack type" to another stack type.
This may seem a little odd at first since we only have functions from one type
to another when we're dealing with typed lambda calculi; here, a stack type is
not &mdash; properly speaking &mdash; a type at all, as evidenced by the above
syntax. You simply cannot give a declaration a stack type. Instead, so-called
stack types are used solely to describe *stack transformations*, which is what
a "function" and its type **actually** represent in ataraxia.

Stack types can be thought of as a strange hybrid between two things: a
crippled kind of \\( \Pi \\) type with special-sauce semantics, and a
"\\( \text{State} \\)" type (or "\\( \text{World} \\)" type). They "are"
\\( \Pi \\) types since a type signature that looks something like

\\[
    \tau_0 \\; \tau_1 \rightarrow \tau_2 \\; \tau_3
\\]

represents a function that must take both a \\( \tau_0 \\) *and* a
\\( \tau_1 \\), in that order, and somehow yield a \\( \tau_2 \\) *and* a
\\( \tau_3 \\), in that order. So it's vaguely like

\\[
    \tau_0 \times \tau_1 \rightarrow \tau_2 \times \tau_3,
\\]

except that really this function is a *state transition* from a state where the
stack contains \\( \tau_0 \\; \tau_1 \\) to a state where there is
\\( \tau_2 \\; \tau_3 \\) in its place. That's really what's going on here. Of
course, a stack "type" can have a single \\( \tau \\), in which case a function
type from one such stack type to another such stack type coincides with the
usual notion of \\( \tau_0 \rightarrow \tau_1 \\) in a typed lambda calculus,
just lacking actual parameters.

## the scope of stack "types"

The other major thing to know about stack "types" is that they determine how
much of the entire stack a function "sees". When you call a function from
inside of some block, you might have e.g. 3 values currently on the stack. If
the function that you call has type \\( A \rightarrow B \\) for some concrete
types \\( A \\) and \\( B \\), then the function that you call can only "see"
the top value on your stack. As far as it is concerned, the total height of the
stack is one, not the three that you can see. As a trivial example, this is
guaranteed to be a type error for any two concrete types `A` and `B`:

```rust
type_error : A -> B := {
    pop pop
}
```

That's because there's only one value on the stack at the beginning of this
function, so there's no way to immediately pop two items from the stack without
first pushing at least one item onto it. As mentioned before, you can in fact
represent stack states of size/height zero: just use `()`.

```rust
also_a_type_error : () -> () := {
    pop
}

well_typed : () -> () := {
    137 pop
}
```

## quantifying over types (a.k.a. System \\( \text{F} \\): the nasty version)

Ok, so the abstract type syntax introduced not to far above was a slight lie.
Not really a lie, but it's only a subset of the actual types we can express.
One reason for that is that we can express
"[universal quantification](https://en.wikipedia.org/wiki/Universal_quantification)"
over types.

"Universal quantification" is just a fancy mumbo-jumbo way of saying that
something applies to a variable "for all" possible instances of that variable.
Simple example: if \\( P(x) \\) is a predicate that tests if \\( x \\) has a
biological heart, then it would be true to say "\\( P(m) \\) for all mammals
\\( m \\)", since all mammals have hearts. For short, we just write
\\( \forall m. P(m) \\), since usually the domain that we're talking about
(mammals in this case) is implied. Otherwise we might say that
\\( \mathcal{M} \\) is the set of all mammals and so
\\( \forall m \\! \in \\! \mathcal{M}. \\, P(m) \\).

Well, we can do this universal quantification for *types*, too. Let's first add
this to our abstract syntax:

\\[
\begin{align}
    \tau             &::= \forall v_0, v_1, ..., v_n. \\, \tau_\Lambda \mid \tau_\kappa                    \\\\
    \tau_\kappa      &::= \tau_B \mid \tau_f                                                               \\\\
    \tau_\Lambda     &::= \tau_B \mid \tau^\Lambda_f                                                       \\\\
    \tau_{\Lambda v} &::= \tau_\Lambda \mid v                                                              \\\\
    \tau_B           &::= \text{T} \mid \\, \perp                                                          \\\\
    \tau_f           &::= s_0 \rightarrow s_1                                                              \\\\
    \tau^\Lambda_f   &::= s^\Lambda_0 \rightarrow s^\Lambda_1                                              \\\\
    s                &::= \tau_{\kappa_0} \\; \tau_{\kappa_1} \\; ... \\; \tau_{\kappa_n} \mid ()          \\\\
    s^\Lambda        &::= \tau_{\Lambda v_0} \\; \tau_{\Lambda v_1} \\; ... \\; \tau_{\Lambda v_n} \mid () \\\\
    v                &::= \text{t}                                                                         \\\\
\end{align}
\\]

Well, ok, that complicates things. No worries, it's basically the same thing
except that instead of always having to name actual concrete types in stack
"types" (i.e. in function types), we can put quantified type *variables*
anywhere we would normally put those concrete types. Here a type **v**ariable
is denoted \\( v \\), and the \\( \text{t} \\) is just a lowercase identifier,
like `t` or `a` or `beans`. Concrete base types have uppercase identifiers (at
least, they *start* with an uppercase letter), and type variables have
lowercase identifiers. Also note that here, \\( \kappa \\) indicates that the
type is not quantified over (it is \\( \kappa \\)oncrete), and \\( \Lambda \\)
indicates the opposite (since quantifying over types is like lambda
abstraction, but on the level of types).

Notice that there's actually one other change besides that. A base type
\\( \tau_B \\) can now be whatever \\( \perp \\) is. "\\( \perp \\)" is
pronounced "[bottom](https://en.wikipedia.org/wiki/Bottom_type)", and it is a
special-sauce type that **has no values**. Seems useless, right? Obviously
nothing can be of that type since that type has no values, so why even include
it in the syntax? Well, first off, the reason why we only include it now is
because we are using it as a substitute for universally quantifying over base
types. You'll notice that while we do quantify over the types that make up a
\\( \tau_f \\), we do no such thing with \\( \tau_B \\)'s. That's because, if
we *did* do likewise with the syntax for \\( \tau_B \\)'s, then we would get

\\[
    \forall v. v
\\]

as a possible production of \\( \tau_B \\). But this type doesn't make any
sense. This would correspond to a function that, no matter what type you
"asked for" (i.e. no matter how you specialized the function), it would be able
to magically return a value of that type, even if that type had... no values.
In any case, it would be impossible for such a function to "know" what type was
being "asked for" without the use of some kind of type introspection, so even
if you called it with the concrete type `Nat` (which certainly has values), it
would have no way to distinguish between that and another specialization of
type `Str`.

Such a function is actually isomorphic to proving all possible statements (and
their negations) according to the
[Curry-Howard correspondence](https://en.wikipedia.org/wiki/Curry%E2%80%93Howard_correspondence),
which is clearly bogus. So instead of writing \\( \forall v. v \\) we simply
write \\( \perp \\) (note that these are **not** actually the same thing, but
rather, the \\( \forall v. v \\) is just a motivation for adding \\( \perp \\)
into the type system). This type can be used for functions which do not yield
anything at all, i.e. once you call such a function, you never get back control
of the program. This could be an infinite loop, or maybe a `panic` which aborts
all execution with an error. Also note that a value of this type can be used
anywhere and it will always typecheck, since it doesn't yield a value anyways.

## constrained quantification over types (a.k.a. System \\( \text{F} \\): the extra nasty version)

Ok, unfortunately, before we get on to actually typing some expressions, we're
going to have to reveal yet another part of how types actually work in
ataraxia. That's right: even the last abstract syntax figure was not complete.

We want to *constrain* the type variables that we quantify over, so that they
can't just be any type at all, but only certain types that satisfy a certain
property/implementation. For example, we might want to make the type for our
function be "generic" (*for all...*), but only over
[integral](https://en.wikipedia.org/wiki/Integer) types like 32-bit integers,
64-bit integers, `Nat`s, and so on. This enables us to do what is usually known
as "[ad-hoc polymorphism](https://en.wikipedia.org/wiki/Ad_hoc_polymorphism)".
Let's see what it looks like:

\\[
\begin{align}
    \tau                           &::= \forall v^{\vartriangleleft}\_0, v^{\vartriangleleft}\_1, ..., v^{\vartriangleleft}\_n. \\, \tau_{\Lambda\vartriangleleft}
                                   \mid \forall v^{\vartriangleleft?}\_0, v^{\vartriangleleft?}\_1, ..., v^{\vartriangleleft?}\_n. \\, \tau_\Lambda \mid \tau_\kappa        \\\\
    \tau_\kappa                    &::= \tau_B \mid \tau_f                                                                                                                  \\\\
    \tau_{\Lambda\vartriangleleft} &::= v \mid \tau_\Lambda                                                                                                                 \\\\
    \tau_\Lambda                   &::= \tau_B \mid \tau^\Lambda_f                                                                                                          \\\\
    \tau_B                         &::= \text{T} \mid \\, \perp                                                                                                             \\\\
    \tau_f                         &::= s_0 \rightarrow s_1                                                                                                                 \\\\
    \tau^\Lambda_f                 &::= s^\Lambda_0 \rightarrow s^\Lambda_1                                                                                                 \\\\
    s                              &::= \tau_{\kappa_0} \\; \tau_{\kappa_1} \\; ... \\; \tau_{\kappa_n} \mid ()                                                             \\\\
    s^\Lambda                      &::= \tau_{{\Lambda\vartriangleleft}\_0} \\; \tau_{{\Lambda\vartriangleleft}\_1} \\; ... \\; \tau_{{\Lambda\vartriangleleft}\_n} \mid () \\\\
    v                              &::= \text{t}                                                                                                                            \\\\
    v^{\vartriangleleft?}          &::= v \mid v^\vartriangleleft                                                                                                           \\\\
    v^\vartriangleleft             &::= v \vartriangleleft c                                                                                                                \\\\
    c                              &::= \text{C}_0 \\; \\& \\; \text{C}_1 \\; \\& \\; ... \\, \\& \\; \text{C}_n                                                            \\\\
\end{align}
\\]

Ouch. Well alright, let's break down the new stuff. We use
\\( v^\vartriangleleft \\) to mean "constrained type variable" and
\\( v^{\vartriangleleft?} \\) to mean "maybe constrained type variable" (the
point of the distinction is to disallow quantifying over a single lone type
variable without constraint(s)). \\( c \\) is a "constraint", and
\\( v \vartriangleleft c \\) means "\\( v \\) is *constrained by* \\( c \\)".
We use \\( \text{C} \\) to mean the literal name of a constraint (like
`Integral`), and \\( \text{C}_0 \\; \\& \\; \text{C}_1 \\) is just the
conjunction of two constraints, so that you have to satisfy *both*.

So what is the point of adding these "constraints"? What do they do for us?
The point is that, if you know a bit of extra info about a type even though
it's quantified over (like that it is an integral type), then you can do
special things that you wouldn't be able to do with just *any* type. A good
example for integral types is that you can add them together (some types can't
be added with `+`), or another is that they can be used as indices (like into a
list, perhaps?).

ataraxia already comes with types that are made to satisfy certain constraints
(e.g. \\( Nat \vartriangleleft Integral \\)), but you can make types satisfy
constraints yourself if you want to. You just have to implement zero or more
functions that that constraint is associated with (like if you wanted your
type to be constrained by `Integral`, you would have to implement `+` for your
type, among other things).

## sprinkling in type constructors (a.k.a. System \\( \text{F}_\omega \\): please stop doing this, this is an abuse of type theory)

I apologize, but there is just one more thing. It will be a quick one. We need
type constructors (type operators with
[kinds](https://en.wikipedia.org/wiki/Kind_\(type_theory\))
\\( \* \rightarrow \* \rightarrow \\; ... \rightarrow \* \rightarrow \* \\)).
Think `Vec<T>` in Rust, or `Seq a` in Haskell; these both are type constructors
that take a single concrete type and produce from it another concrete type
(kind \\( \* \rightarrow \* \\)). Now we can apply both of our genericising
tools (universal quantification, and constraints) to produce types. Luckily the
changes to the abstract syntax aren't so big:

\\[
\begin{align}
    \tau                           &::= \forall v^{\vartriangleleft}\_0, v^{\vartriangleleft}\_1, ..., v^{\vartriangleleft}\_n. \\, \tau_{\Lambda\vartriangleleft}
                                   \mid \forall v^{\vartriangleleft?}\_0, v^{\vartriangleleft?}\_1, ..., v^{\vartriangleleft?}\_n. \\, \tau_\Lambda \mid \tau_\kappa                         \\\\
    \tau_\kappa                    &::= \tau_B \mid \tau_f                                                                                                                                   \\\\
    \tau_{\Lambda\vartriangleleft} &::= v \mid \tau_\Lambda                                                                                                                                  \\\\
    \tau_\Lambda                   &::= \tau^\Lambda_B \mid \tau^\Lambda_f                                                                                                                   \\\\
    \tau_B                         &::= \text{T} \mid \omega_\kappa \mid \\, \perp                                                                                                           \\\\
    \tau^\Lambda_B                 &::= \tau_B \mid \omega\_\Lambda                                                                                                                          \\\\
    \tau_f                         &::= s_0 \rightarrow s_1                                                                                                                                  \\\\
    \tau^\Lambda_f                 &::= s^\Lambda_0 \rightarrow s^\Lambda_1                                                                                                                  \\\\
    s                              &::= \tau_{\kappa_0} \\; \tau_{\kappa_1} \\; ... \\; \tau_{\kappa_n} \mid ()                                                                              \\\\
    s^\Lambda                      &::= \tau_{{\Lambda\vartriangleleft}\_0} \\; \tau_{{\Lambda\vartriangleleft}\_1} \\; ... \\; \tau_{{\Lambda\vartriangleleft}\_n} \mid ()                  \\\\
    v                              &::= \text{t}                                                                                                                                             \\\\
    v^{\vartriangleleft?}          &::= v \mid v^\vartriangleleft                                                                                                                            \\\\
    v^\vartriangleleft             &::= v \vartriangleleft c                                                                                                                                 \\\\
    c                              &::= \text{C}_0 \\; \\& \\; \text{C}_1 \\; \\& \\; ... \\, \\& \\; \text{C}_n                                                                             \\\\
    \omega\_\kappa                 &::= \text{T}\langle\tau\_{B\_0} \\; \tau\_{B\_1} \\; ... \\; \tau\_{B\_n}\rangle                                                                         \\\\
    \omega\_\Lambda                &::= \text{T}\langle\tau\_{{\Lambda\vartriangleleft}\_0} \\; \tau\_{{\Lambda\vartriangleleft}\_1} \\; ... \\; \tau\_{{\Lambda\vartriangleleft}\_n}\rangle \\\\
\end{align}
\\]

We introduce here the notion of \\( \omega \\), which is a type-constructed
type which may (\\( \omega\_\Lambda \\)) or may not (\\( \omega\_\kappa \\))
have its type arguments be quantified over. \\( \text{T} \\) still stands for
an identifier that starts with an uppercase letter, and the items in angle
brackets (\\( \langle \\; \rangle \\)) are type arguments to that constructor.

## typing expressions

Since there are three kinds of expressions, let's look at how those three kinds
of expressions each get their types.

### typing literals

Typing literals is generally pretty straightforward, but it has its subtleties.
Some literals are unambiguous:

- Boolean literals (`true`, `false`) always have type `Bool`.
- Character literals always have type `Char`.
- String literals always have type `Str`.

Some literals are more ambiguous, like floating point literals. ataraxia
supports both 32-bit and 64-bit floating point numbers, so there's no way to
know from a float literal alone which one it is.

There are type constraints built into the standard library that allow
genericising over floating point numbers and integers: `Floating` and
`Integral`, respectively. If the particular situation allows, these constraints
can just be directly applied and give the respective float and int literals the
most generic types possible:

\\[
\begin{align}
    floatLiteral &: \forall f \vartriangleleft Floating. \\; f \\\\
    intLiteral   &: \forall i \vartriangleleft Integral. \\; i \\\\
\end{align}
\\]

But if the literals in question are not found in genericised code that allows
this use of generic typing, they will automatically take on default types of
`F32` and `I32`, respectively (these correspond to 32-bit floating point
numbers and 32-bit signed integers, respectively).

List literals are even a bit trickier. Lists are of type `List<?>` where `?`
could be a concrete type, or a type variable in some cases. If the values in
your list literal don't all have the same type, then it won't typecheck at all,
of course. Otherwise the rules are the same for whatever type its elements
have, and you then substitute that type in for the `?`. Lists, though, don't
have a "default type", so if the type cannot be found from the elements then
you may still get a type error.

### typing identifiers

Giving identifiers types is really easy, since all identifiers are top-level
declarations that are required to have literal type signatures anyways.

### typing blocks

Blocks that aren't top-level declarations generally have their types inferred.
Blocks **always have a function type**, never a plain value type. Keep in mind
[the rules about stack scoping](#the-scope-of-stack-types) when considering the
left and right stack "types" that would be part of an anonymous block's type
signature.

The block `{}` has a type signature of `() -> ()`. The block `{?}` has the same
type signature as the inner `?` if `?` is a function (i.e. the function is
quoted), and a type signature of `() -> t` if `?` is not a function and has
type `t` (since `?` is implicitly pushed to the stack).
