# 2024-04-25

In last lectures, we saw how different types in our programming language can be modeled by appropriate structure in our category.

A very common feature that you can find in modern programming languages is the ability to represent functions/programs as first-class objects: functions themselves can be stored within variables, passed to other programs, which in turn can call them (even multiple times, with different arguments). This capability sometimes takes the name of "functional programming", and we say that the language supports the functional programming paradigm when there is enough support for this to happen.

It turns out that asking that our language has function types is equivalent to asking that a class of functors definable in the category have a left adjoint.

## Function types

The idea is to introduce a new type `Func<A,B>`, which represents the _"type of functions that take an argument of type `A` and return something of type `B`."_

> Note: the function type does not arise as the initial algebra for some functor.

Some examples on how to use something of type `Func<Int,Bool>` and how to create something of that type

```rust
fn isEven(a: Int): Bool => a % 2 == 0
fn greaterThan100(a: Int): Bool => a > 100

fn check(p: Func<Int,Bool>): String =>
  if p(42) then
    "the predicate is true on 42"
  else
    "not true on 42"

> check(isEven)
"the predicate is true on 42"
> check(greaterThan100)
"not true on 42"
// Lambda expressions/anonymous functions
> check(fn(x: Int) => x % 2 == 0)
"the predicate is true on 42"
> check(fn(y: Int) => y % 2 == 0)
"the predicate is true on 42"
```

```rust
// Anonymous functions
fn coursesPassed(courses: List<Pair<String,Int>>): Int =>
    courses.filter(fn(course: Pair<String,Int>) => course.snd() > 60)
           .length()
```

Functions can also be returned as values.

```rust
fn createAdder(base: Int): Func<Int,Int> =>
  fn(x: Int) => base + x

> let f = createAdder(100)
> f(3)
103
> let g = createAdder(40)
> g(f(10))
150

fn createAccount(balance: Int): Func<Int,String> =>
  fn(request: Int) =>
    if request > balance then
      "You do not have enough money."
    else if request == 0 then
      "Warning: you selected zero!"
    else
      "Valid transaction."
> let myAccount = createAccount(1000)
> myAccount(3400)
"You do not have enough money."
```

And can be called more than once, as well as doing all previously mentioned things.

```rust
fn double(a: Int): Int => 2 * a

fn<A> applyTwice(f: Func<A,A>): Func<A,A> =>
  fn(x: A) => f(f(x))

> let f = applyTwice(double)
> f(10)
40
```

## Currying

A program which returns a function type is essentially the same as a program which takes two arguments.

```rust
fn isExamPassed(increaseDifficulty: Bool): Func<Int,Bool> =>
  if increaseDifficulty then
    fn(grade: Int) => grade > 75
  else
    fn(grade: Int) => grade > 60

> let f = isExamPassed(true)
> f(65)
false
> f(81)
true
> isExamPassed(false)(62)
true
```

The process of converting a function with a similar type into a usual function with *two* arguments is called **uncurrying**.

```rust
fn isExamPassedTwoArgs(increaseDifficulty: Bool, grade: Int): Bool =>
  if increaseDifficulty then
    grade > 75
  else
    grade > 60

> isExamPassedTwoArgs(false,62)
true
```

This process can be inverted, and takes the name of `curry`ing from the name of the logician Haskell Curry: a function taking two arguments is the same as a function that takes one and then "delays" executing the code by returning a function that takes the second argument.

```rust
fn enclose(tag: String, s: String): String =>
  "<" + tag + ">" + s + "</" + tag + ">"

fn encloseCurried(tag: String): Func<String,String> =>
  fn(s: String) =>
    enclose(tag,s)

> let makeTitle = encloseCurried("h1")
> let makeLink = encloseCurried("a")
> makeTitle("this is the content.")
"<h1>this is the content.</h1>"
> makeLink("https://github.com/iwilare/programming-in-ct")
"<a>https://github.com/iwilare/programming-in-ct</a>"
```

## Recall: the adjunction between products and function types

> **Adjunctions.** Given two categories $C,D$ and two functors $F : C \to D$ and $G : D \to C$ we say that
> - $F$ is a left adjoint of $G$
> - $G$ is a right adjoint of $F$
> - $F \dashv G$
>
> whenever there is a parametric isomorphism in $\textbf{Set}$
> $$\frac{D(F(X), Y)}{C(X, G(Y))}$$

We now describe a specific adjunction which, in Crust, allows us to talk and model about function types.

> We say that a cartesian category $C$ (the category has all products) has **exponentials** (or "function types") if, for any fixed object $A \in C$ the functor $A \times -$ (defined by $X \mapsto A \times X$) has a right adjoint $R_A$ which we denote as $A \Rightarrow -$. This means that there must be a isomorphism between the following hom-sets of $C$, parametric in $X,Y,A$:
> $$\frac{C(A \times X, Y)}{C(X, A \Rightarrow Y)}$$
> *Note*: this isomorphism is always an isomorphism between sets; there are two morphisms in $\textbf{Set}$ such that they compose to the identity in both directions.

We will now show that adding function types to Crust precisely means that $\texttt{Prog}$ is cartesian closed, and that the currying and uncurrying operations are the two conversions that implement this isomorphism.

In the case of Crust, for some fixed type `A`, the right adjoint is the functor whichs ends the type `X` into the type `Func<A,X>`.

*Proof*.

For any object `A,X,Y` of Crust, and any morphism `fn f(p: Pair<A,X>): Y` we define a morphism

```rust
fn f_curry(x: X): Func<A,Y> =>
  fn(a: A) => f(pair(x, a))
```

Similarly, for any morphism `fn f(p: X): Func<A,Y>` we define a morphism

```rust
fn f_uncurry(p: Pair<A,X>): Y =>
  f(p.snd())(p.fst())

fn f_uncurry'(a: A, x: X): Y =>
  f(x)(a)
```

These correspondences are isomorphisms, in the sense that first doing one and then the other gives you back the function you started with (in the sense that they are program equivalent):

- Assume a generic function `f: Func<Pair<A,B>,C>>`, and first apply `curry` and then `uncurry`. We show that the result we get is a function which is program equivalent to `f`.

  ```rust
    (f_curry)_uncurry
  = (fn(a: A) =>
      fn(b: B) =>
        f(pair(a,b)))_uncurry    // Definition unfolding.
  = fn(p: Pair<A,B>) =>
      match p with
      | pair(a, b) =>
        (fn(a: A) =>
          fn(b: B) =>
            f(pair(a,b)))(a)(b)      // Function application.
  = fn(p: Pair<A,B>) =>
    match p with
    | pair(a, b) => f(pair(a, b))  // Eta equivalence of products.
  = fn(p: Pair<A,B>) => f(p)       // Eta equivalence of functions.
  = f
  ```
- Assume a generic function `f: Func<A,Func<B,C>>`, and first apply `uncurry` and then `curry`. We show that the result we get is a function which is program equivalent to `f`.
  ```rust
    (f_uncurry)_curry
  = (fn(p: Pair<A,B>) =>
      match p with
      | pair(a, b) => f(a)(b))_curry         // Definition unfolding.
  = fn(a: A) =>
      fn(b: B) =>
        (fn(p: Pair<A,B>) =>
         match p with
         | pair(a, b) => f(a)(b))(pair(a,b)) // Function application.
  = fn(a: A) =>
      fn(b: B) =>
        match pair(a,b) with
        | pair(a, b) => f(a)(b)              // Eta equivalence of products.
  = fn(a: A) => fn(b: B) => f(a)(b)          // Eta equivalence of functions.
  = fn(b: B) => f(a)                         // Eta equivalence of functions.
  = f
  ```

## Unit and counit of the adjunction

What happens when we apply the hom-set isomorphisms to the identity functions? The morphisms we obtain are called the `unit` and `counit` morphisms, and they are in particular natural transformations (thanks to the parametricity of the hom-set isomorphism).

```rust
fn eval(a: A, f: Func<A,B>): B => f(a)

fn eval(p: Pair<A,Func<A,B>>): B => f(a)

fn unit(b: B): Func<A,Pair<A,B>> =>
  fn(a: A) =>
    pair(a,b)
```

*Theorem.* `eval = (id_Pair<A,B>)_curry`, `unit = (id_Func<A,B>)_curry`.

```rust
fn<A,B> const(a: A): Func<B,A> =>
  fn(b: B) => a
```

*Theorem.* `const = snd_curry`.

```rust
fn<X,A> identity(u: X): Func<A,A> =>
  fn(x: A) => x
```

*Theorem.* `identity = fst_curry`.

## More examples

```rust
// Action on morphisms (as `Func`s) of the List<_> functor
fn<A,B> map(a: List<A>, f: Func<A,B>): List<B> =>
  match a with
  | nil() => nil()
  | cons(a, as) => cons(f(a), vs.map(f))

// Given a list, return a new list with only the elements which satisfy the predicate
fn<A,B> filter(a: List<A>, p: Func<A,Bool>): List<B> =>
  match a with
  | nil() => nil()
  | cons(v, vs) =>
    if p(a) then
      cons(v, vs.map(f))
    else
      vs.map(f)

fn isAdult(age: Int): Bool => age >= 18

// Using the names of the functions
fn averageAge(people: List<Pair<String,Int>>): Float =>
  people.filter(isAdult)
        .map(snd)
        .average()

// Using anonymous functions
fn averageAge(people: List<Pair<String,Int>>): Float =>
  people.filter(fn(p: Pair<String,Int>) => p.age() >= 18)
        .map(fn(p: Pair<String,Int>) => match p with pair(n,a) => a)
        .average()

// Using anonymous function, but simply calling the original function
fn averageAge(people: List<Pair<String,Int>>): Float =>
  people.filter(fn(p: Pair<String,Int>) => p.isAdult())
        .map(fn(p: Pair<String,Int>) => p.fst())
        .average()
```

## Internalization

Many of the constructions we have seen during the course are parameterized with respect to other morphisms:

- (currying and uncurrying)
The `uncurry`ing function can be defined internally in Crust:

  ```rust
  fn<A,B,C> uncurry(f: Func<A,Func<B,C>>): Func<Pair<A,B>,C> =>
    fn(p: Pair<A,B>) =>
      match p with
      | pair(a, b) => f(a)(b)

  > let f = uncurry(isExamPassed)
  > f(false, 62)
  > true
  ```

  ```rust
  fn<A,B,C> curry(f: Func<Pair<A,B>,C>): Func<A,Func<B,C>> =>
    fn(a: A) =>
      fn(b: B) =>
        f(pair(a,b))
  ```

  > *Theorem.* `curry` and `uncurry` are type isomorphisms. In particular,
  > $$\texttt{uncurry};\texttt{curry} = \text{id}_{\texttt{Func<A,Func<B,C>}}$$
  > $$\texttt{curry};\texttt{uncurry} = \text{id}_{\texttt{Func<Pair<A,B>,C>}}$$

- (composition in a category, say, $\texttt{Par}$) For any two morphisms
  ```rust
  fn f(x: A): Maybe<B>
  ```
  and
  ```rust
  fn g(x: B): Maybe<C>
  ```
  we define their composition in the category of partial programs to be the morphism
  ```rust
  fn comp_f_g(x: A): Maybe<C> =>
      match x.f() with
      | none() => none()
      | some(v) => v.g()
  ```

  ---
  **Internalized version:**

  ```rust
  fn comp(f: Func<A,B>, g: Func<B,C>): Func<A,C> =>
    fn(x: A) => x.f().g()
  ```

  ```rust
  fn compPar(f: Func<A,Maybe<B>>, g: Func<B,Maybe<C>>): Func<A,Maybe<C>> =>
    fn(x: A) =>
      match x.f() with
      | none() => none()
      | some(b) => g(b)
  ```

- (pairing functions for products) For any two morphisms

  ```rust
  fn l(v: X): A => ...
  fn r(v: X): B => ...
  ```

  The universal morphism `pairing_l_r` for `l` and `r` is defined as follows:

  ```rust
  fn pairing_l_r(v: X): Pair<A,B> =>
      pair(l(v), r(v))
  ```

  ---
  **Internalized version:**

  ```rust
  fn<A,B,C> pairing(f: Func<A,B>, g: Func<A,C>): Func<A,Pair<B,C>> =>
    fn(x: A) => pair(f(x), g(x))
  ```
- (functors)

  ```rust
  fn<A,B> List_map(a: List<A>, f: Func<A,B>): List<B> =>
    match a with
    | nil() => nil()
    | cons(a, as) => cons(f(a), as.List_map(f))
  ```

  ```rust
  fn<A,B> Maybe_map(m: Maybe<A>, f: Func<A,B>): Maybe<B> =>
    match m with
    | none() => none()
    | some(a) => some(f(a))
  ```

- (initial algebras and unique morphisms)

  Fix a type `A`. We know that `List<A>` with the program
  ```rust
  fn<A> structure(a: Either<Unit,Pair<A,List<A>>): List<A> =>
    match a with
    | left(unit()) => nil()
    | right(pair(a, as)) => cons(a, as)
  ```
  is the initial algebra `(List<A>,structure)` in the category of algebras for the functor $F = X \mapsto 1 + A \times X$ (in Crust, `A ↦ Either<Unit,Pair<A,List<A>>`).

  Initiality means that, for any other algebra
  $α = (E, \texttt{h} : 1 + A \times E \to E)$, there is a unique morphism `fn rec_α(a: List<A>): E` defined as follows:

  ```rust
  fn rec_α(a: List<A>): E =>
    match a with
    | nil() => h(left(unit()))
    | cons(a, as) => h(right(pair(a, rec_α(as))))
  ```

  which is the only one satisfying the property that it is an algebra morphism, i.e., the equation
  $$
  \texttt{structure}\,;\texttt{rec}_\alpha=F(\texttt{rec}_\alpha)\,;\texttt{h}_\alpha
  $$
  holds (i.e., a certain square commutes.)

  ---
  **Internalized version:**

  ```rust
  fn<A,E> rec(a: List<A>, h: Func<Either<Unit,Pair<A,E>>,E>): E =>
    match a with
    | nil() => h(left(unit()))
    | cons(a, as) => h(right(pair(a, rec_α(as))))
  ```
  or, using the fact that functions from coproducts `Either<_,_>` are essentially pairs of functions from each component of the `Either`,

  ```rust
  fn<A,E> rec(a: List<A>, z: Func<Unit,E>, f: Func<Pair<A,E>,E>): E =>
    match a with
    | nil() => z(unit())
    | cons(a, as) => f(a, rec(as,z,f))
  ```

  and then, using the fact that functions from `Unit` into `E` basically correspond to an object of type `E`,

  ```rust
  fn<A,E> rec(a: List<A>, z: E, f: Func<Pair<A,E>,E>): E =>
    match a with
    | nil() => z
    | cons(a, as) => f(a, rec(as,z,f))
  ```
  You might have seen this function as `foldr` in Haskell!
