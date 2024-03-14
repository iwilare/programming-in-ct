# 2024-02-29

# The `type` keyword

The `type` keyword allows to construct new types, possibly involving existing types. The constructs we will see will correspond to concepts that were presented in the theory lectures.

This is an informal schema for definitions using `type`:

```rust
type TypeName<P1, P2, ...> =
      cons1(Type11, ..., Type1N1)
    | cons2(Type21, ..., Type2N2)
    | ...
    | consM(TypeM1, ..., TypeMNm)
```

`cons1`, ... `consM` are called *constructors*. Instances of `TypeName` can ultimately only ever arise from the use of a constructor. Values constructed by constructor `consX` are composed of member values of the corresponding types `consX1`, ..., `consXN`. This is similar to `enum` definitions in Rust, or `data` definitions in Haskell.
A `type` definition can also have *no constructor*, in which case the equal sign is also omitted.


# The `Unit` type

```rust
type Unit = unit()

fn toUnit_Int(a: Int): Unit => unit()
fn toUnit_Str(a: Str): Unit => unit()
fn toUnit_Bool(a: Bool): Unit => unit()

// Alternatively, we could have defined it uniformly with a parametric schema:

fn<A> toUnit(a: A): Unit => unit()

fn justAnswer(a: Unit): Int => 
    match a with
    | unit() => 42
```

Notice the following:

> Given any type `A`, there is a morphism `fn toUnit(x: A): Unit`. Moreover, any morphism `fn f(a: A): Unit` is equivalent to `toUnit`.

*Proof.*

The morphism is `toUnit_A`. Let's assume to have another morphism `f` and let's see that it's equal to `toUnit_A`.

```
  f(x)       (General rule: there is no other option for `Unit`.)
= unit()         

  toUnit(x)  (Function evaluation.)
= unit()
```

# The `Empty` type

```rust
// No constructor!
type Empty 

fn fromEmpty_Int(a: Empty): Int => match a with 
fn fromEmpty_Str(a: Empty): Str => match a with 
fn fromEmpty_Bool(a: Empty): Bool => match a with 

// Alternatively, we could have defined it with a parametric schema:

fn<A> fromEmpty(a: Empty): A => 
  match a with 

/*
This is a valid Crust program!
The `match` construct needs one case for each possible way of constructing 
the type that it's matching on. In this case, there are no possible ways 
of constructing  something of type Empty, so it is okay to simply put no cases 
in our match expression.
Note that the return type is allowed to be any type A because no concrete value
needs to be provided, so it works with any type.
*/
```

Let's prove the following:

> Given any type `A`, there is a morphism `fn fromEmpty_A(x: Empty): A`. Moreover, any morphism `fn f(a: Empty): A` is equivalent to `fromEmpty_A`.

The morphism is `fromEmpty_A`. Let's assume to have another morphism `f` and let's see that it's equal to `fromEmpty_A`.

*Proof.*
```
Recall the definition of program equivalence.

> **Program equivalence:** given two types `A` and `B` two programs `fn f(x: A): B` and `fn g(x: A): B`, we say that they are *equal* (written `f = g`) when: *for every value `x` of type `A`, the two calls `f(x)` and `g(x)` return the same value*.

Therefore, we need to check that for every value `x` of type `Empty` the values `fromEmpty_A(x)` and `f(x)` are the same.

There is no value of type `Empty`, therefore we have already checked that in every possible case the values of the two functions is the same, so we are done.
```

# `Unit` is not a terminal object in `ParProg`.

> **Unit is NOT the terminal object of `Prog`.** Given any type `A`, there is a morphism `fn toUnit(x: A): Maybe<Unit>`. Moreover, *NOT EVERY* morphism `fn f(a: A): Maybe<Unit>` is equivalent to `toUnit`; that is, there are at least two morphisms which are not equivalent.

*Proof.*

The two morphisms are the following:

```
fn toUnit1(x: A): Maybe<Unit> => some(unit())
fn toUnit2(x: A): Maybe<Unit> => none()
```

They are clearly not equivalent.

# `Empty` is the initial object in `ParProg`.

> **`Empty` is the initial object in `ParProg`.** Given any type `A`, there is precisely one
morphism `fn fromEmpty_A(x: Empty): Maybe<A>`.

*Proof.*

We can define the morphism as follows:

```rust
fn fromEmpty_A(x: Empty): Maybe<A> => match x with
```

Moreover, consider any other morphism with signature `fn f(x: Empty): Maybe<A>`.
By a similar reasoning to what you saw in the last lab lecture, `fromEmpty_A` and `f` are equal 
if they return the same value for any input. But there are no inputs of type `Empty`, so this
requirement is trivially satisfied.

# `Empty` is also the terminal object in `ParProg`!


> **`Empty` is also the terminal object in `ParProg`.** Given any type `A`, there is precisely one
morphism `fn toEmpty_A(x: A): Maybe<Empty>`.

*Proof.*

We can define the morphism as follows:

```rust
fn toEmpty_A(x: A): Maybe<Empty> => none()
```

Moreover, consider any other morphism with signature `fn f(x: A): Maybe<Empty>`. We want 
to prove that for any `x: A`, `f(x) = toEmpty_A(x)`. So let's reason by case analysis on `f(x): Maybe<Empty>`:

 - if `f(x) = none()`, then it is also equal to `toEmpty_A(x) = none()`;
 - otherwise we should consider `f(x) = some(e)`, where `e: Empty`, but this case cannot occur, since
 there are no values of type `Empty`.
