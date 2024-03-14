# 2024-03-14

# Products and coproducts, recap

Recall the definition of products and coproducts: we're going to recast it in a way that looks more computational.

An object $P$ in a category $C$ is said to be a *product* of two objects *A* and *B* when the following things happen:
- *(projections)* There are two morphisms $\text{fst} \in C(P,A), \text{snd} \in C(P,B)$,
- *(universal morphism)* For any other object $X$ and morphisms $l \in C(X,A)$, $r \in C(X,B)$, there is a morphism into the product $\lang l,r \rang \in C(X,P)$. Moreover, this morphism is such that the following equations hold $$\lang l,r \rang\,;\text{fst} = l$$ $$\lang l,r \rang\,;\text{snd} = r.$$ We can call these the *product equalities*.
- *(uniqueness)* The universal morphism is *unique* among those which satisfy the *product equalities*, in the sense that **if** there is another morphism $p \in C(X,P)$ such that $$p\,;\text{fst} = l$$ $$p\,;\text{snd} = r.$$ **then** $$p = \lang l,r \rang.$$

An alternative way of phrasing this last condition is the following:

- *(eta-uniqueness)* for any object $X$ and $p \in C(X,P)$, the following equation holds:

$$p = \lang (p\,;\text{fst}),\ (p\,;\text{snd}) \rang.$$

*Proof. Uniqueness to eta-uniqueness.*
Assume uniqueness. Simply take the two equations and replace $l$ and $r$ in the last expression.

*Proof. Eta-uniqueness to uniqueness.*
Use the equations in the opposite directions to replace $p\,;\text{fst}$ and $p\,;\text{snd}$ into $l$ and $r$ in the eta-uniqueness condition.

> Take a category $C$. If a product $P$ for two objects $A$, $B$ exists in the category, we say that the *category has all products*.

> If a category has all products and has a terminal object, the category is said to be *cartesian*. (think of the *cartesian product of sets*)

# Back to Crust

Adding more structure to our categories increases the expressivity of our language.

- `Unit` type
    - Terminal object in $\text{Prog}$
    - *Functions into `Unit`*: functions which return no value
    - *Functions from `Unit`*: functions which take no argument
- `Empty` type
    - Initial object in $\text{Prog}$
    - *Functions into `Empty`*: functions which never return
    - *Functions from `Empty`*: functions which cannot be called
- `Pair<A,B>` type
    - Products in $\text{Prog}$ for any object `A`, `B`
    - *Functions into `Pair<A,B>`*: functions which return multiple values
    - *Functions from `Pair<A,B>`*: functions taking multiple arguments

Recall the definition of the `Pair<A,B>` type:

```rust
type Pair<A,B> = pair(A,B)
```

For this section we will consider the Crust language with the following types:
- `Int`, `String`, `Bool` types
- Functions only take one argument, as always
- A `Pair<A,B>` type for any two types `A`, `B`.
    - (In particular, we must also admit `Pair<Int,Pair<String,Int>>`, `Pair<Pair<String,String>,Bool>`, etc...!)

Let's show that $\text{Prog}$ has all products, where now `Prog` takes as objects all the types we just listed and all total programs between as morphisms.

*Proof.*

- For any two types `A`, `B`, we say that there is an object which satisfies the property of being a product of `A`, `B`. The object we select is `Pair<A,B>`.
- *(projections)* We need to provide two morphisms $\texttt{fst} \in \text{Prog}(\texttt{Pair<A,B>},\texttt A), \texttt{snd} \in \text{Prog}(\texttt{Pair<A,B>},\texttt B)$. We define them like this:

```rust
fn fst(p: Pair<A,B>): A =>
    match p with
    | pair(a,b) => a

fn snd(p: Pair<A,B>): B =>
    match p with
    | pair(a,b) => b
```

- *(universal morphism)* Take any type `X`, and assume we are given two programs

```rust
fn l(v: X): A => ...
fn r(v: X): B => ...
```

The universal morphism `pairing_l_r` for `l` and `r` is defined as follows:

```rust
fn pairing_l_r(v: X): Pair<A,B> =>
    pair(l(v), r(v))
```

Let's verify the equations $\lang l,r \rang\,;\text{fst} = l$, $\lang l,r \rang\,;\text{snd} = r$.
Remember that composition is "substitution" of one definition into the other, and that equality is program equivalence.

- $\lang l,r \rang\,;\text{fst} = l$

    The first morphism is this program:
    ```rust
        fn comp_(pairing_l_r)_fst(v: X): A =>
           v.pairing_l_r().fst()
    ```
    Let's reason by program equivalence, for some argument `v: X`:
    ```rust
           v.pairing_l_r().fst()             // (Function expansion.)
         = pair(l(v), r(v)).fst()         // (Function expansion.)
         = (match pair(l(v),r(v)) with
           | pair(a,b) => a)              // (Match evaluation.)
         = l(v)
    ```
    which is exactly the definition of ```fn l(v: X): A```.

- $\lang l,r \rang\,;\text{snd} = r$

    The first morphism is this program:
    ```rust
        fn comp_(pairing_l_r)_snd(v: X): A =>
           v.pairing_l_r().snd()
    ```
    Let's reason by program equivalence, for some argument `v: X`:
    ```rust
           v.pairing_l_r().snd()             // (Function expansion.)
         = pair(l(v), r(v)).snd()         // (Function expansion.)
         = (match pair(l(v),r(v)) with
           | pair(a,b) => b)              // (Match evaluation.)
         = r(v)
    ```
    which is exactly the definition of ```fn r(v: X): A```.

- *(Eta uniqueness)*

    Assume to have a morphism `fn p(v: X): Pair<A,B>` which also satisfies the *product equalities*. We show that is program equivalent to `pairing_(comp_p_fst)_(comp_p_snd)`.

    Assume to have a value `v: X`.
    and let's assume that when we call `p` on this value we get a result `pair(v1,v2)` for some values `v1: A` and `v2: B`.
    ```rust
           v.p()
         = pair(v1, v2)        // (General principle.)
                               // Assumpution that the result of this function is given
                               // by combining some values `v1`, `v2`.
    ```
    ```rust
           v.pairing_(comp_p_fst)_(comp_p_snd)           // (Function unfolding.)
         = pair(v.comp_p_fst(), v.comp_p_snd())       // (Function unfolding.)
         = pair(v.p().fst(), v.p().snd())             // (Assumption.)
         = pair(pair(v1,v2).fst(), pair(v1,v2).snd()) // (Function unfolding.)
         = pair(match pair(v1,v2)
                | pair(a,b) => a,
                match pair(v1,v2)
                | pair(a,b) => b)                     // (Match evaluation.)
         = pair(v1, v2)
    ```

# `match`ing and `pair`ing are equivalent to *having all categorical products*

It turns out that having the `pair` constructor and being able to use `match` in the language *really* is the same thing as simply stating that `Pair<A,B>` is a product in the category $\text{Prog}$, and using the properties that hold for products.

- > *Being able to construct elements of a pair is the same as the existence of the universal morphism.*

    Informally:

    ```rust
    pair(<expr1>[x], <expr2>[x])
    ```

    can be replaced by

    ```rust
      pairing_expr1_expr2[x]
    = pair(<expr1>[x], <expr2>[x]) // (Our definition of `pairing_l_r`)
    ```
    (we need to consider `expr1` and `expr2` as "functions" which take their free variables as arguments.)

- > *Being able to `match` is the same as having `fst` and `snd` as functions of our language.*

    ```rust
    match <expr> with
    | pair(a,b) => <body>[a,b]
    ```
    becomes
    ```rust
    <body>[<expr>.fst(),<expr>.snd()]
    ```

    For example:

    ```rust
    fn isAdultWithNameLength(p: Pair<Int, String>): Pair<Bool, Int> =>
        match p with
        | pair(age,name) => pair(age >= 18, name.length())
    ```

    is program equivalent to

    ```rust
    fn isAdultWithNameLength(p: Pair<Int, String>): Pair<Bool, Int> =>
        pair(p.fst() >= 18, p.snd().length())
    ```
- > The product equalities of the universal morphism say how the `fst` and `snd` functions behave, and allows us to use the *(Match evaluation)* rule when running a program and in our program equalities. (Remember that `fst` and `snd` were defined by match evaluation.)
- > The eta-uniqueness of the universal morphism says that pairs are determined by their `fst` and `snd`, and nothing else.

    ```rust
    let (a,b) = p
    return pair(a,b)
    ```

    really is the same as just avoiding decomposition and recomposition

    ```rust
    p
    ```

# Uniqueness of programs up to (unique) isomorphism

Let's prove that any two products are isomorphic in Crust.

**In other words, we assume to have two different ways of constructing "pairs", and we show that they really are the same.**

Assume that
- - `P1` is a product type of `A` and `B`,
  - `fst1` and `snd1` are its two projections/product morphisms,
  - `pairing1_l_r` is its family of pairingersal morphisms.
  - The product equalities and eta-uniqueness for `pairing1_l_r` holds.
- - `P2` is a product type of `A` and `B`,
  - `fst2` and `snd2` are its two projections/product morphisms,
  - `pairing2_l_r` is its family of pairingersal morphisms.
  - The product equalities and eta-uniqueness for `pairing2_l_r` holds.

Let's construct two isomorphisms:


```rust
    fn iso12(x: P1): P2 => pair2(x.fst1(), x.snd1())
    fn iso21(x: P2): P1 => pair1(x.fst2(), x.snd2())
```

> *Note:*
> - `pair2(x.f(), x.g())` is a shorthand for `pairing2_f_g(x)`.
> - `pair1(x.f(), x.g())` is a shorthand for `pairing1_f_g(x)`.
>
> Purely using morphisms, the previous definition really should be written as:
> ```rust
>     fn iso12(x: P1): P2 => pairing2_fst1_snd1(x)
>     fn iso21(x: P2): P1 => pairing1_fst2_snd2(x)
> ```

We show that the two maps really are isomorphisms by checking that they compose to the identity in both directions.

- Assume to have a value `x: P1`:

    ```rust
          x.comp_iso12_iso21()
        = x.iso12().iso21()                       // Function unfolding
        = (pair2(x.fst1(), x.snd1())).iso21()     // Function unfolding
        = pair1(pair2(x.fst1(), x.snd1()).fst2()  // First product equality for pairing2_fst1_snd2
               ,pair2(x.fst1(), x.snd1()).snd2()) // Second product equality for pairing2_fst1_snd2
        = pair1(x.fst1()
               ,x.snd1())
        = x
    ```

    So we have shown that $\texttt{iso12}\,;\texttt{iso21} = \text{id}_\texttt{P1}$.

- Assume to have a value `x: P2`:

    ```rust
          x.comp_iso21_iso12()
        = x.iso21().iso12()                       // Function unfolding
        = (pair1(x.fst2(), x.snd2())).iso12()     // Function unfolding
        = pair2(pair1(x.fst2(), x.snd2()).fst1()  // First product equality for pairing2_fst2_snd1
               ,pair1(x.fst2(), x.snd2()).snd1()) // Second product equality for pairing2_fst2_snd1
        = pair2(x.fst2()
               ,x.snd2())
        = x
    ```

    So we have shown that $\texttt{iso21}\,;\texttt{iso12} = \text{id}_\texttt{P2}$.

# Products are symmetric

For any two types `A`, `B`, we show a type isomorphism between `Pair<A,B>` and `Pair<B,A>`.

*Proof.*

```rust
<A,B> fn ABtoBA(x: Pair<A,B>): Pair<B,A> => pair(x.snd(), x.fst())
<A,B> fn BAtoAB(x: Pair<B,A>): Pair<A,B> => pair(x.snd(), x.fst())
```
Note: these two are actually the same function! It's just this by `X=A,Y=B` and `X=B,Y=A`, respectively.
```rust
<X,Y> fn swap(x: Pair<X,Y>): Pair<Y,X> => pair(x.snd(), x.fst())
```

Let's prove that $\texttt{swap}\,;\texttt{swap}=\text{id}_{\texttt{Pair<A,B>}}$.

Assume `x: Pair<X,Y>`.

```rust
    x.swap().swap()
  = pair(x.snd(), x.fst()).swap()
  = pair(pair(x.snd(), x.fst()).snd(),
         pair(x.snd(), x.fst()).fst()) // (Function unfolding + match evaluation, twice)
  = pair(x.fst(), x.snd())             // (Eta uniqueness for products.)
  = x
```

The other direction is $\texttt{swap}\,;\texttt{swap}=\text{id}_{\texttt{Pair<B,A>}}$... which we have already proven!

> Note: we have not relied on any property of the two types `A`,`B`. This isomorphism is *parametric* in the sense that we have seen during the lecture.

# Associativity of products *(Exercise!)*

Show that, for any two types `A`,`B`, the type `Pair<A,Pair<B,C>>` is isomorphic with the type `Pair<Pair<A,B>,C>`.

# Products and terminal objects

> We now assume to also have the terminal object `Unit` in Crust and in our category.

Let's show that, for any two types `A`, the type `Pair<Unit,A>` is isomorphic to the type `A`.

```rust
type Unit = unit()

<A> fn construct(x: A): Pair<Unit,A> => pair(unit(), x)
<A> fn destruct(x: Pair<Unit,A>): A => x.snd()
```

Let's prove these two maps are isomorphisms:

- Assume `x: A`.
  ```rust
    x.construct().destruct()   // (Function evaluation.)
  = pair(unit(), x).destruct() // (Function unfolding.)
  = pair(unit(), x).snd()      // (Function evaluation.)
  = x
  ```

- Assume `x: Pair<Unit,A>`.
  ```rust
    x.destruct().construct()
  = x.snd().construct()
  = pair(unit(), x.snd())      // (Uniqueness of every expression of type Unit.)
  = pair(x.fst(), x.snd())     // (Eta-uniqueness of pairs.)
  ```

# Coproducts

An object $S$ in a category $C$ is said to be a *coproduct* of two objects *A* and *B* when the following things happen:
- *(injections)* There are two morphisms $\text{left} \in C(A,S), \text{right} \in C(B,S)$,
- *(universal morphism)* For any other object $X$ and morphisms $l \in C(A,X)$, $r \in C(B,X)$, there is a morphism from the coproduct $[l,r] \in C(S,X)$. Moreover, this morphism is such that the following equations hold $$\text{left}\,;[l,r] = l$$ $$\text{right}\,;[l,r] = r.$$ We can call these the *coproduct equalities*.
- *(uniqueness)* The universal morphism is *unique* among those which satisfy the *coproduct equalities*, in the sense that **if** there is another morphism $s \in C(S,X)$ such that $$\text{left}\,;s = l$$ $$\text{right}\,;s = r.$$ **then** $$s = [l,r].$$

An alternative way of phrasing this last condition is the following:

- *(eta-uniqueness)* for any object $X$ and $s \in C(S,X)$, the following equation holds:

$$s = [ (\text{left}\,;s),\ (\text{right}\,;s) ].$$

*Proof. Uniqueness to eta-uniqueness.*
Assume uniqueness. Simply take the two equations and replace $l$ and $r$ in the last expression.

*Proof. Eta-uniqueness to uniqueness.*
Use the equations in the opposite directions to replace $\text{left}\,;s$ and $\text{right}\,;s$ into $l$ and $r$ in the eta-uniqueness condition.

> Take a category $C$. If a coproduct $S$ for two objects $A$, $B$ exists in the category, we say that the *category has all coproducts*.

> If a category has all coproducts and has an initial object, the category is said to be *cocartesian*.

# Back to Crust

The definition for coproducts is something we have not seen yet, but that should be familiar: it corresponds to the concept of *tagged unions* in C, TypeScript, and algebraic data types with multiple cases in Haskell, Rust. In Java this mechanism can be emulated by creating an interface and implementing it with exactly *two* classes.

```rust
type Either<A,B> = left(A)
                 | right(B)

// Multiple constructors
type Maybe<A> = some(A)
              | none()
```

Let's make some examples:

```rust
// Either celsius or fahrenheit
fn areYouSick(temperature: Either<Float,Int>): Bool =>
    match temperature with
    | left(celsius)     => celsius > 37.8
    | right(fahrenheit) => fahrenheit > 99

// Give the numerical value of a constant, fail otherwise
fn getConstantName(name: String): Either<String,Float> =>
    if name == "pi" then
        right(3.14)
    else if name == "e" then
        right(6.28)
    else if name == "tau" then
        left("This constant will be loaded soon!")
    else
        left("Sorry! I don't know the constant ${name}.")

// Compare this last function with the following one, using `Maybe`, which we have seen already.

fn getConstantName(name: String): Maybe<Float> =>
    if name == "pi" then
        some(3.14)
    else if name == "e" then
        some(6.28)
    else if name == "tau" then
        none()
    else
        none()

// No information about the error can be given!
```

## Back to expressiveness

- `Unit` type
    - Terminal object in $\text{Prog}$
    - *Functions into `Unit`*: functions which return no value
    - *Functions from `Unit`*: functions which take no argument
- `Empty` type
    - Initial object in $\text{Prog}$
    - *Functions into `Empty`*: functions which never return
    - *Functions from `Empty`*: functions which cannot be called
- `Pair<A,B>` type
    - Products in $\text{Prog}$ for any object `A`, `B`
    - *Functions into `Pair<A,B>`*: functions which return multiple values
    - *Functions from `Pair<A,B>`*: functions taking multiple arguments
- `Either<A,B>` type
    - Coproducts in $\text{Prog}$ for any object `A`, `B`
    - *Functions into `Either<A,B>`*: functions which can return values of different types (e.g., failure)
    - *Functions from `Either<A,B>`*: functions taking in input values of different types

For this section we will consider the Crust language with the following types:
- `Int`, `String`, `Bool` types
- Functions only take one argument, as always
- A `Either<A,B>` type for any two types `A`, `B`.
    - (In particular, we must also admit `Either<Int,Either<String,Int>>`, `Either<Either<String,String>,Bool>`, etc...!)

Let's show that $\text{Prog}$ has all coproducts, where now `Prog` takes as objects all the types we just listed and all total programs between as morphisms.

*Proof.*

- For any two types `A`, `B`, we say that there is an object which satisfies the property of being a coproduct of `A`, `B`. The object we select is `Either<A,B>`.
- *(injections)* We need to provide two morphisms $\texttt{left} \in \text{Prog}(\texttt A,\texttt{Either<A,B>}), \texttt{right} \in \text{Prog}(\texttt B,\texttt{Pair<A,B>})$. We define them like this:

```rust
fn left(p: B): Either<A,B> => left(p)

fn right(p: A): Either<A,B> => right(p)
```

- *(universal morphism)* Take any type `X`, and assume we are given two programs

```rust
fn l(v: A): X => ...
fn r(v: B): X => ...
```

The universal morphism `cases_l_r` for `l` and `r` is defined as follows:

```rust
fn cases_l_r(v: Either<A,B>): X =>
    match v with
    | left(a) => l(a)
    | right(b) => r(b)
```

Let's verify the equations $\text{left}\,;[l,r] = l$, $\text{right}\,;[l,r] = r$.

- $\text{left}\,;[l,r] = l$

    The first morphism is this program:
    ```rust
        fn comp_left_(pairing_l_r)(v: X): A =>
           v.left().pairing_l_r()
    ```
    Assume `v: X`:
    ```rust
           v.left().pairing_l_r()           // (Function expansion.)
         = match left(v) with
           | left(a) => l(a)
           | right(b) => r(b)               // (Match evaluation.)
         = l(v)
    ```
    which is exactly the definition of ```fn l(v: A): X```.

- $\text{right}\,;[l,r] = r$

    The first morphism is this program:
    ```rust
        fn comp_right_(pairing_l_r)(v: X): A =>
           v.right().pairing_l_r()
    ```
    Assume `v: X`:
    ```rust
           v.right().pairing_l_r()           // (Function expansion.)
         = match right(v) with
           | left(a) => l(a)
           | right(b) => r(b)               // (Match evaluation.)
         = r(v)
    ```
    which is exactly the definition of ```fn r(v: B): X```.

- *(Eta uniqueness)*

    Assume to have a morphism `fn s(v: Either<A,B>): X`. We show that is program equivalent to `matching_(comp_left_s)_(comp_right_s)`.

    Assume to have a value `v: Either<A,B>`.
    There are two cases to check:
    - **either** `v = left(a)` for some value `a`,
    - or `v = right(b)` for some value `b`.

    In both cases, we verify that the conclusion holds, and these are only two cases we need to check.

    - case `v = left(a)` for some value `a: A`:
    ```rust
        v.matching_(comp_left_s)_(comp_right_s)       // (Assumption.)
      = left(a).matching_(comp_left_s)_(comp_right_s) // (Function unfolding.)
      = match left(a) with
        | left(a) => comp_left_s(a)
        | right(b) => comp_right_s(b)                 // (Match evaluation.)
      = comp_left_s(a)                                // (Function unfolding.)
      = left(a).s()                                   // (Assumption.)
      = v.s()
    ```
    - case `v = right(b)` for some value `b: B`:
    ```rust
        v.matching_(comp_left_s)_(comp_right_s)        // (Assumption.)
      = right(b).matching_(comp_left_s)_(comp_right_s) // (Function unfolding.)
      = match right(b) with
        | left(a) => comp_left_s(a)
        | right(b) => comp_right_s(b)                 // (Match evaluation.)
      = comp_right_s(b)                               // (Function unfolding.)
      = right(b).s()                                  // (Assumption.)
      = v.s()
    ```

# `match`ing and `left`/`right`ing are equivalent to *having all categorical coproducts*

Similarly as with the case of products, having the `left` and `right` constructors and being able to use `match` in the language *really* is the same thing as simply stating that `Either<A,B>` is a coproduct in the category $\text{Prog}$, and using the properties that hold for coproducts.

- > *Being able to construct elements of an either is the same as the existence of the injections.*
- > *Being able to `match` is the same as having `matching_l_r` for any two functions of our language.*

    ```rust
    match <expr> with
    | left(a) => <body1>[a]
    | right(b) => <body2>[b]
    ```
    becomes
    ```rust
    matching_<body1>_<body2>(<expr>)
    ```

    (And indeed the two correspond in our definition.)
- > The coproduct equalities of the universal morphism allows us to use the *(Match evaluation)* rule when running a program and in our program equalities. They simply say that

    ```rust
    match left(v) with
    | left(a) => <body1>[a]
    | right(b) => <body2>[b]
    ```

    is equivalent to

    ```rust
    <body1>[v]
    ```

    and similar with the `right` case.
- > The eta-uniqueness of the universal morphism says that the following expression:

    ```rust
    match v with
    | left(a) => left(a)
    | right(b) => right(b)
    ```

    is simply equal to `v`: unpacking and then packing again with the same constructor is equal to doing nothing.

# Uniqueness of coproducts up to (unique) isomorphism *(Exercise!)*

# Symmetry of coproducts *(Exercise!)*

Show that, for any two types `A`,`B`, the type `Either<A,B>` is isomorphic with the type `Either<B,A>`.

# Associativity of coproducts *(Exercise!)*

Show that, for any two types `A`,`B`, the type `Either<A,Either<B,C>>` is isomorphic with the type `Either<Either<A,B>,C>`.

# Initial objects and coproducts

We will show that `Either<Empty,A>` is isomorphic to `A`.

> We now assume to also have the initial object `Empty` in Crust and in our category.

```rust
type Empty

<A> fn construct(x: A): Either<Empty,A> => right(x)
<A> fn destruct(x: Either<Empty,A>): A =>
    match x with
    | left(e) =>
       (match e with
       )
    | right(a) => a
```

Let's prove these two maps are isomorphisms:

- Assume `x: A`.
  ```rust
    x.construct().destruct() // (Function evaluation.)
  = right(x).destruct()      // (Function unfolding.)
  = match right(x) with
    | left(e) =>
       (match e with
       )
    | right(a) => a // Match evaluation
  = x
  ```

- Assume `x: Either<Empty,A>`.
  There are two possible cases to check: either `x = left(e)` for some value `e: Empty`, or `x = right(a)` for some value `a: A`.
    - Case `x = left(e)` for some value `e: Empty`:
        - We need to check that two programs are equal for any possible value of type `e: Empty`. Since there is *no* value of type `Empty`, we are done! We have checked it for all cases.
    - Case `x = right(a)` for some value `a: A`:
        ```rust
          x.destruct().construct()        // (Assumption.)
        = right(a).destruct().construct() // (Function unfolding.)
        = (match right(a) with
          | left(e) => (match e with)
          | right(a) => a).construct() // (Match evaluation.)
        = a.construct()                // (Function unfolding.)
        = right(a)                     // (Assumption.)
        = x
        ```

# Special exercise: distributivity

Show that ```Pair<A,Either<B,C>> ≃ Either<Pair<A,B>,Pair<A,C>>```.
