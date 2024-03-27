# Natural transformations

## The `List<_>` functor

Arrays and lists form the basis of most examples of programming in the wild. Even though we do not yet know how to say that we have a notion of "`List<_>`" functor categorically, many familiar examples of familiar functions which are natural transformations will use this functor.

```rust
type List<A> = nil()
             | cons(A, List<A>)
```

The type `List<_>` is a functor in the sense that we've seen during the previous lecture.
The functorial action is defined as follows, given a function `fn f(a: A): B`:

```
fn<A> List_map_f(l: List<A>): List<B> =>
  match l with
  | nil() => nil()
  | cons(a, as) => cons(f(a), List_map_f(as))
```

**Exercise**: check that identities and compositions are preserved.

## Parametric polymorphism

We've seen during many lectures that functions can be expressed "polymorphically"/"parametrically" on any given type `A`. We use this syntax:

```rust
// From Lecture 1
fn<A> identity(a: A): A => a

// From Lecture 1
fn<A> createCourse(data: A): Pair<String,A> => pair("category theory", data)

// From Lecture 2
fn<A> fromEmpty(a: Empty): A => match a with

// From Lecture 3
fn<A,B> fst(x: Pair<A,B>): A =>
  match x with
  | pair(a, b) => a

// From Lecture 3
fn<A> construct(x: A): Pair<Unit,A> => pair(unit(), x)

// Multiple parameters
fn<A,B> ignoreSecond(x: A, y: B): B => x
```

This ability to define *families of functions*, one for each type in a way that is uniform in the type `A`, is called *parametric polymorphism*. Let's see some more examples.

```rust
// Return a pair with two copies of `a`
fn<A> duplicate(a: A): Pair<A,A> => pair(a, a)

// Extract the right element from an Either, if possible
fn<A,B> getRight(e: Either<A,B>): Maybe<B> =>
  match e with
  | left(a) => none()
  | right(b) => some(b)

// Convert a Maybe<_> into Either<Unit,_>
fn<A,B> maybeToEither(e: Maybe<B>): Either<Unit,B> =>
/*
  match e with
  | none() => left(unit())
  | right(b) => right(b)
*/
```

Some examples using the `List<_>` functor:

```rust
// Extract the first element of a list, return `none()` if the list is empty
fn<A> head(l: List<A>): Maybe<A> =>
  match l with
  | nil() => none()
  | cons(a, as) => some(a)

// Extract the tail of a list, return `nil()` if the list is empty
fn<A> tail(l: List<A>): List<A> =>
  match l with
  | nil() => nil()
  | cons(a, as) => as

// Return the number of elements of a list
fn<A> length(l: List<A>): Int =>
  match l with
  | nil() => 0
  | cons(a, as) => 1 + length(as)

// Reverse a list
fn<A> reverse(l: List<A>): List<A> =>
  match l with
  | nil() => nil()
  | cons(a, as) => reverse(as).appendAtEnd(a)

// Add an element `e` at the end of the list
fn<A> appendAtEnd(l: List<A>, e: A): List<A> =>
  match l with
  | nil() => cons(e, nil())
  | cons(a, as) => cons(a, appendAtEnd(e, as))

// Remember that, categorically, functions with multiple arguments
// correspond to functions with a single argument but which take a product.
fn<A> appendAtEndProd(le: Pair<List<A>, A>): List<A> =>
  match le with
  | pair(l, e) =>
    match l with
    | nil() => cons(e, nil())
    | cons(a, as) => cons(a, appendAtEnd(pair(e, as)))

// Append one list into the other
fn<A> append(l1: List<A>, l2: List<A>): List<A> =>
/*
  match l1 with
  | nil() => l2
  | cons(a, l1_) => cons(a, append(l1_, l2))
*/

// Remember that, categorically, functions with multiple arguments
// correspond to functions with a single argument but which take a product.
fn<A> append(l1l2: Pair<List<A>, List<A>>): List<A> =>
/*
  match l1l2 with
  | pair(l1, l2) =>
    match
    | nil() => l2
    | cons(a, l1_) => cons(a, append(pair(l1_, l2)))
*/

// Similar definition of tail, but return `none()` when the list is empty
fn<A> maybeTail(l: List<A>): Maybe<List<A>> =>
/*
  match l with
  | nil() => none()
  | cons(a, as) => some(as)
*/
```

Categorically, these families of functions remind us of natural transformations.
Let's recall the definition.

## Natural transformations

> Given two categories $C$, $D$, and two functors $F, G : C \to D$, a natural transformation $\alpha$ is a family of morphisms in $D$ for each $X \in C_0$: $$\alpha_X : F_0(X) \to G_0(X)$$ Moreover, for any morphism $f : X \to Y$ of $C$ the following equation in $D$ holds, which we call "naturality": $$F(f) \,; \alpha_Y = \alpha_X \,; G(f)$$

> ***Intuition***: the intuition for naturality is that each $\alpha_X$ is "defined uniformly" with respect to the parameter $X$, and so each morphism behaves "independently" of variations (morphisms) in the parameter $X$.

We have seen that using parametric polymorphism in Rust defines a (uniform) family of morphisms in Crust.
Indeed, we'll spend the rest of the lectures showing how the naturality condition is indeed satisfied by the programs we defined in Crust.

*Note*: the functors we've seen in Crust are all functors from $\textrm{Prog}$ into $\textrm{Prog}$, so we're only defining natural transformations between endofunctors.

---

## Naturality of `head`

> `head<A>` is a natural transformation between the `List<_>` and `Maybe<_>`.

> *Intuition*: mapping a function `f` over a list, and then taking the first element, is the same as taking the first element and then mapping the function `f`.

*Equationally in Crust* the following two programs are equivalent:

```
comp_(List_map_f)_head = comp_head_(Maybe_map_f)
```

By program equivalence, this means that for every type `X` and object `xs: List<X>`:

```
xs.List_map_f().head() = xs.head().Maybe_map_f()
```

*Proof.*

- Case `xs = nil()`:

  ```
    nil().List_map_f().head()
  = nil()
  = none()
  ```
  ```
    nil().head().Maybe_map_f()
  = none()
  = none()
  ```
- Case `xs = cons(a, as)`:

  ```
    cons(a, as).List_map_f().head()
  = cons(f(a), List_map_f(as)).head()
  = some(f(a))
  ```
  ```
    cons(a, as).head().Maybe_map_f()
  = some(a).Maybe_map_f()
  = some(f(a))
  ```

## Naturality of `tail`

*Exercise!*

## Naturality of `length`

*What are the two functors at play here?*
The type `A` does not appear in the expression of the return type.

*Recall* that a constant functors on a type $K \in D$ is given by mapping objects in $C$ into $K \in D$, and by mapping every morphism into the identity on $K$.

> `length` is a natural transformation between the `List` functor and the constant functor at `Int`.

*Equationally in Crust*: by program equivalence, the naturality condition this means that for every type `X` and object `xs: List<X>`:

```
xs.List_map_f().length() = xs.length().ConstInt_map_f()
                         = xs.length().id()
                         = xs.length()
```

> *Intuition*: for any list, mapping a function over a list does not change its length.

*Proof.*

- Case `xs = nil()`:

  ```
    nil().List_map_f().length()
  = nil().length()
  = 0
  ```
  ```
    nil().length().ConstInt_map_f()
  = nil().length()
  = 0
  ```
- Case `xs = cons(a, as)`:

  ```
    cons(a, as).List_map_f().length()
  = cons(f(a), as.List_map_f()).length()
  = 1 + as.List_map_f()
  ```
  ```
    cons(a, as).length().ConstInt_map_f()
  = 1 + length(as)
  ```

  By inductive hypothesis, `as.List_map_f() = length(as)`.
  Therefore, the two expressions are equal.

## Naturality of `appendAtEnd`

*Exercise!*

> ***Note***: categorically, `appendAtEnd` can be seen as just a shorthand for `appendAtEndProd`.

> `appendAtEndProd` is a natural transformation from the functor (`A` $\mapsto$ `Pair<List<A>, A>`) to the `List` functor.

You can check that (`A` $\mapsto$ `Pair<List<A>, A>`) and doing the intuitive thing on morphisms is indeed a functor.

Naturality says that for any pair `p: Pair<List<A>, A> = pair(xs, a): `,
```
pair(xs, a).appendAtEndProd().List_map_f() = pair(xs, a).PairListA_map_f.appendAtEndProd()
```
Or, in other words,
```
pair(xs, a).appendAtEndProd().List_map_f() = pair(xs.List_map_f(), f(a)).appendAtEndProd()
```

## Naturality of `reverse`

> `reverse` is a natural transformation between the `List` functor and the `List` functor.

```
xs.List_map_f().reverse() = xs.reverse().List_map_f()
```

> *Intuition*: mapping a function and then reversing is the same as reversing and then mapping.
> ```
>     [   a,    b,    c,    d,    e] <mapping>
>     [f(a), f(b), f(c), f(d), f(e)] <reverse>
>     [f(e), f(d), f(c), f(b), f(a)]
> ```
> ```
>     [   a,    b,    c,    d,    e] <reverse>
>     [   e,    d,    c,    b,    a] <mapping>
>     [f(e), f(d), f(c), f(b), f(a)]
> ```

*Proof.*

- Case `xs = nil()`:

  ```rust
    nil().List_map_f().reverse()
  = nil().reverse()
  = nil()
  ```
  ```rust
    nil().reverse().List_map_f()
  = nil().List_map_f()
  = nil()
  ```
- Case `xs = cons(a, as)`:

  ```rust
    cons(a, as).List_map_f().reverse()
  = cons(f(a), as.List_map_f()).reverse()
  = (as.List_map_f().reverse()).appendAtEnd(f(a))
  = as.List_map_f().reverse().appendAtEnd(f(a))
  ```

  ```rust
    cons(a, as).reverse().List_map_f()
  = as.reverse().appendAtEnd(a).List_map_f() // Naturality of appendAtEnd<A>
  = as.reverse().List_map_f().appendAtEnd(f(a))
  ```

  By inductive hypothesis,

  ```rust
    as.List_map_f().reverse() = as.reverse().List_map_f()
  ```

  which makes the two expressions equal.

---

## Naturality of `getRight`

> For any `K`, `getRight<K,A>` is a natural transformation between `Either<K,_>` and `Maybe<_>`.

For any type `A`, and value `e: Either<K,A>`:

```
e.EitherK_map_f().getRight() = e.getRight().Maybe_map_f()
```

*Proof.*

- Case `e = left(k)`:

  ```
    left(k).EitherK_map_f().getRight()
  = left(k).getRight()
  = none()
  ```
  ```
    left(k).getRight().Maybe_map_f()
  = none().Maybe_map_f()
  = none()
  ```
- Case `e = right(a)`:

  ```
    right(a).EitherK_map_f().getRight()
  = right(f(a)).getRight()
  = some(f(a))
  ```
  ```
    right(a).getRight().Maybe_map_f()
  = some(a).Maybe_map_f()
  = some(f(a))
  ```

## Naturality of `duplicate`

*What are the two functors at play here?*

```rust
fn<A> duplicate(a: A): Pair<A,A> => pair(a, a)
```

The second functor sends each type `A` to the type `Pair<A,A>`, we checked last time that this is indeed a functor.

*Recall* the identity functor $id_C : C \to C$: on objects, it assigns to each type $K \in D$ the same type $K$. On morphisms it acts as the identity function, sending `f` to `f`.

> `duplicate` is a natural transformation between the identity functor `_` and `Pair<_,_>`.

(We refer explicitly to `Identity` and `DoublePair` as the two functors.)

*Equationally in Crust*:

For any type `A`, and value `a: A`:

```
a.Identity_map_f().duplicate() = a.duplicate().DoublePair_map_f()
```

By unfolding,

```
a.f().duplicate() = a.duplicate().DoublePair_map_f()
```

*Proof.*

```
  a.f().duplicate()
= pair(f(a), f(a))
```
```
  a.duplicate().DoublePair_map_f()
= pair(a, a).DoublePair_map_f()
= pair(f(a), f(a))
```
