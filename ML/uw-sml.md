---
title: Standard ML
date: 2020-09-16
tags:
  - Polymorphic λ
---

### ML Variable Bindings and Expressions

```sml
val z = 3;
(* static environment: z : int *)
(* dynamic environment: z --> 3 *)

val w = z + 1;
(* static environment: z : int, w : int *)
(* dynamic environment: z --> 3, w --> 4 *)
```

1. The semantics
- Syntax is just how you write something
- Semantics is what something means
  - Type-checking (before program runs)
  - Evaluation (as program runs)
- For variable bindings:
  - Type-check expression and extend static environment 
  - Evaluate expression and extend dynamic environment

2. Rules for expressions:
- Every kind of expression has:
  - Syntax 
  - Type-checking rules. Produce a type or fails.
  - Evaluation rules (used only on things that type-check).  
    Produce a value (or exception or infinite-loop).

3. Reasons for shadowing:
```sml
val a = 1
val b = a (* b is bound to 1 *)
val a = 2
```
- Expressions in variable bindings are evaluated **eagerly**
  - Before the variable binding finishes
  - Afterwards, the expression producing the value is irrelevant
- There is no way to assign to a variable in ML
  - Can only shadow in a later environment  

### ML Functions

1. Functions informally
```sml
fun pow (x : int, y : int) = 
  if y = 0 
  then 1
  else x * pow (x, y - 1)
```
- Cannot refer to later function bindings
  - Simply ML`s rule
  - Helper functions must come before their uses
  - Need special construct for mutual recursion 

2. Function bindings formally
- Syntax: `fun x0 (x1 : t1, ..., xn : tn) = e`
- Type-checking: 
  - Add binding `x0 : (t1 * ... * tn) -> t` if:
  - Can type-check body `e` to have type `t` in the static environment containing:
    - Enclosing static environment (earlier bindings)
    - `x1 : t1, ..., xn : tn` (arguments with their types)
    - `x0 : (t1 * ... * tn) -> t` (for recursion) 
- Evaluation: 
  - A function is a value — we simply add x0 to the environment as a function that can be called later. 
  - As expected for recursion, x0 is in the dynamic environment in the function body and for subsequent bindings.

  
3. Function calls formally
- Syntax: `e0 (e1,...,en)` with the parentheses optional if there is exactly one argument.
- Type-checking: require that `e0` has a type that looks like `t1 * ... * tn -> t` and for 1 ≤ i ≤ n, `ei` has type `ti`. Then the whole call has type `t`.
- Evaluation:
  - Evaluate `e0` to `fun x0 (x1 : t1, ..., xn : tn) = e`. Since call type-checked, result will be a function. Evaluate arguments to values `v1, ..., vn` (Under current dynamic environment).
  - Result is evaluation of `e` in an environment extended to map `x1` to `v1, ..., xn` to `vn` (An environment is actually the environment where the function was defined, and includes `x0` for recursion).

### Pairs and Other Tuples

1. Build
- Syntax: `(e1, e2)`
- Type-checking: if `e1` has type `ta` and `e2` has type `tb`, then the pair expression has type `ta * tb` - a new kind of type.
- Evaluation: evaluate `e1` to `v1` and `e2` to `v2`, result is `(v1, v2)` - a pair of values is a value.

2. Access
- Syntax: `#1 e` and `#2 e`
- Type-checking: if `e` has type `ta * tb`, then `#1 e` has type `ta` and `#2 e` has type `tb`.
- Evaluation: evaluate `e` to a pair of values and return first or second piece.

```sml
fun swap (pr : int * bool) = 
  (#2 pr, #1 pr)

fun sum_two_pairs (pr1 : int * int, pr2 : int * int) =
  (#1 pr1) + (#2 pr1) + (#1 pr2) + (#2 pr2)

fun div_mod (x: int, y: int) =
  (x div y, x mod y)

fun sort_pair (pr : int * int) =
  if (#1 pr) < (#2 pr)
  then pr
  else (#2 pr, #1 pr)
```

3. Tuple is a generalization of pairs. 
- Pairs and tuples can be nested (implied by the syntax and semantics)

```sml
val x1 = (7, (true, 9))
val x2 = #1 (#2 x1)
```

### Lists

1. Build: `[v1, v2, ... ,vn]`, `::` pronounced **cons**
2. Access: `null`, `hd`, `tl` 
3. Functions over lists are usually recursive, only way to get to all the elements.

```sml
fun sum_list (xs : int list) =
  if null xs
  then 0
  else hd(xs) + sum_list(tl xs)
  
fun countdown (x : int) =
  if x = 0
  then []
  else x :: countdown(x - 1)

fun append (xs : int list, ys : int list) =
  if null xs
  then ys
  else (hd xs) :: append(tl xs, ys)
```

### Let Expressions

```sml
fun countup_from (x : int) =
  let 
    fun count (from : int) =
      if from = x
      then x :: []
      else from :: count (from + 1)
  in
    count (1)
  end

fun max (xs : int list) =
  if null xs
  then 0
  else if null (tl xs)
  then hd xs
  else 
    (* for style, could also use a let-binding for (hd xs) *)
    let val tl_ans = max (tl xs)
    in 
      if hd xs > tl_ans
      then hd xs
      else tl_ans
    end
```
1. Syntax: `let b1 b2 ... bn in e end`. Each `bi` is any binding and `e` is any expression.
2. Type-checking: Type-check each `bi` and `e` in a static environment that includes the previous bindings. Type of whole let-expression is the type of `e`.
3. Evaluation: Evaluate each `bi` and `e` in a dynamic environment that includes the previous bindings. Result of whole let-expression is result of evaluating `e`.
4. Scope: where a binding is in the environment. Only in later bindings and body of the let-expression.
5. The key to improve efficiency of recursion is not to do repeated work that might do repeated work. Saving recursive results in local bindings is essential.

### Options

```sml
fun max1 (xs : int list) =
  if null xs
  then NONE
  else 
    let val tl_ans = max1 (tl xs)
    in if isSome tl_ans andalso valOf tl_ans > hd xs
      then tl_ans
      else SOME (hd xs)
    end
```
1. Building: `NONE` has type `'a option`, `SOME e` has type `t option` if `e` has type `t`.
2. Accessing: `isSome` has type `'a option -> bool`, `valOf` has type `'a option -> 'a`  (exception if given `NONE`).

### Booleans and Comparison Operations

1. Syntax: `e1 andalso e2`, `e1 orelse e2`, `not e1`.
2. Short-circuiting evaluation means `andalso` and `orelse` are not functions, but `not` is just a pre-defined function.
3. `=` `<>` can be used with any equality type but not with `real`.

### No Mutation: ML vs. Imperative Languages

1. In ML, we create aliases all the time without thinking about it because it is impossible to tell where there is aliasing.
   - Example: `tl` is constant time; does not copy rest of the list.
   - So do not worry and focus on the algorithm.
2. In language with mutable data (e.g. Java), programmers are obsessed with aliasing and object identity.
   - They have to be (!) so that subsequent assignments affect the right parts of the program.
   - Often crucial to make copies in just the right places.
3. Reference (alias) vs. copy does not matter if code is immutable.

### Pieces of a Language

1. Syntax
2. Semantics (evaluation rules)
3. Idioms (typical patterns for using language features to express the computation)
4. Libraries
5. Tools

### Building Compound Types

Three most important type building-blocks in any language:
1. Each of: A `t` value contains values of each of `t1 t2 ... tn`. Such as tuples and records (idea of syntactic sugar and connection with tuples).
2. One of: A `t` value contains values of ont of `t1 t2 ... tn`. Such as option and a type that contains an int or string (will lead to pattern-matching).
3. Self reference: A `t` value can refer to other `t` values.

### Records

1. Record values have fields (any name) holding values `{f1 = v1, ... , fn = vn}`. Record types have fields holding types `{f1 : t1, ..., fn : tn}`. The order of fields in a record value of type never matters. REPL alphabetizes fields just for consistency.
2. Building records: `{f1 = e1, ..., fn = en}`
3. Accessing pieces: `#myfieldname e`
4. Evaluation rules and type-checking as expected.
5. A common decision for a construct's syntax is whether to refer to things by position (as in tuples) or by some (field) name (as with records).
   - A common hybrid is like with Java method arguments (and ML functions are used so far):
     - Caller uses position
     - Callee uses variables
     - Could do it differently
6. Tuples are just syntactic sugar for records with fields named 1, 2, ...
   - Syntactic: Can describe the semantics entirely by the corresponding record syntax.
   - They simplify understanding / implementing the langauge.

### Datatype Bindings

1. A way to make one-of types: 
   - A `datatype` binding
  ```sml
  datatype mytype = 
    TwoInts of int * int
    | Str of string
    | Pizza
  ```
  - Adds a new type `mytype` to the environment
  - Adds constructors to the environment: `TwoInts`, `Str` and `Pizza`
    - `TwoInts : int * int -> mytype`
    - `Str : string -> mytype`
    - `Pizza : mytype`
2. A constructor is (among other things), a **function** that makes values of the new type (or is a value of the new type):
  - Any value of type `mytype` is made from one of the constructors.
  - The value contains:
    - A tag for which constructor (e.g. TwoInts)
    - The corresponding data (e.g. (7, 9)) 
3. In many other contexts, these datatypes are called tagged unions.
4. There are two aspects to accessing a datatype value:
   - Check what variant it is (what constructor made it). `null` and `isSome` check variants.
   - Extract the data (if that variant has any). `hd` `tl` `valOf` extract data (raise exception on wrong variant).
5. Useful datatypes:
   - Enumerations, including carrying other data.
   - Alternate ways of identifying real-world things/people.
   - Expression trees. This is an example using self-reference:
    ```sml
    datatype exp = 
      Constant of int
      | Negate of exp
      | Add of exp * exp
      | Multiply of exp * exp

    (* functions over recursive datatypes are usually recursive *)

    fun eval e = 
      case e of 
        Constant i => i
        | Negate e2 => ~ (eval e2)
        | Add (e1, e2) => (eval e1) + (eval e2)
        | Multiply (e1, e2) => (eval e1) * (eval e2)

    fun number_of_adds e = 
      case e of
        Constant i => 0
        | Negate e2 => number_of_adds e2
        | Add (e1, e2) => 1 + number_of_adds e1 + number_of_adds e2
        | Multiply (e1, e2) => number_of_adds e1 + number_of_adds e2
    ```
    - Lists and Options are datatypes.
    ```sml
    datatype my_int_list = Empty | Cons of int * my_int_list

    fun inc_or_zero intoption = 
      case intoption of
        NONE => 0
        | SOME i => i + 1

    fun append (xs, ys) =
      case xs of
        [] => ys
        | x :: xs' => x :: append (xs', ys)
    ```
6. Polymorphic Datatypes:
   - Syntax: put one or more type variables before datatype name. Can use these type variables in constructor definitions. Binding then introduces a type constructor, not a type. (Must say `int mylist`, `'a mylist`, not `mylist`).
  ```sml
  datatype 'a mylist = Empty | Cons of 'a * 'a mylist

  datatype ('a, 'b) tree = 
    Node of 'a * ('a, 'b) tree * ('a, 'b) tree
    | Leaf of 'b
  ```
  - Use constructors and case expressions as usual. No change to evaluation rules. Type-checking will make sure types are used consistently. For example, cannot mix element types of list. Functions will be polymorphic or not based on how data is used.

### Case Expressions

1. ML combines the two aspects of accessing a one-of value with a case expression and pattern-matching.

```sml
fun f (x : mytype) = 
  case x of
    Pizza => 3
    | Str s => 8
    | TwoInts (i1, i2) => i1 + i2
```

- A multi-branch conditional to pick branch based on variant.
- Extracts data and binds to variables local to that branch.
- Type-checking: all branches must have same type.
- Evaluation: evaluate between case ... of and the right branch.
- Patterns are not expressions: We do not evaluate them. We see if the result of `e0` matches them.

### Type Synonyms

1. Creating new types:
- A datatype binding introduces a new type name.
  - Distinct from all existing types.
  - Only way to create values of the new type is the constructors.
- A type synonym is a new kind of binding `type aname = t`.
  - Just creates another name for a type.
  - The type and the name are interchangeable in every way.
  - Do not worry about that REPL prints: picks what it wants just like it picks the order of record field names.
2. A convenience for talking about types. Related to modularity.

### Each of Pattern Matching

1. Pattern matching also works for records and tuples.
2. Val-binding patterns: variables are just one kind of pattern. 
3. A function argument can also be a pattern. What we call multi-argument functions are just functions taking one tuple argument, implemented with a tuple pattern in the function binding. Zero arguments is the unit pattern () matching the unit value ().

```sml
fun sum_triple triple = 
  case triple of
    (x, y, z) => x + y + z

fun sum_triple1 triple = 
  let val (x, y, z) = triple
  in
    x + y + z
  end

fun sum_triple2 (x, y, z) = x + y + z
```

### Polymorphic and Equality types

1. The **more general** rule: A type t1 is more general than the type t2 if you can take t1, replace its type variables consistently, and get t2.
- Example: replace each `'a` with `int * int`
- Example: replace each `'b` with `'a` and each `'a` with `'a`
2. Can combine the **more general** rule with rules for equivalence:
- Use of type synonyms does not matter
- Order of field names does not matter
3. Type variables with a second quote such as `''a list * ''a -> bool` are **equality types** that arise from using the = operator. The = operator works on lots of types: int, string, tuples containing all equality types... But not all types: function types, real...
4. The rules for more general are exactly the same except you have to replace an equality-type variable with a type taht can be used with =.

```sml
(* ''a * ''a -> string *)
fun same_thing (x, y) =
  if x = y then "yes" else "no"
```

### Nested Patterns

```sml
fun zip list_triple =
  case list_triple of
    ([], [], []) => []
    | (hd1 :: tl1, hd2 :: tl2, hd3 :: tl3) => 
      (hd1, hd2, hd3) :: zip (tl1, tl2, tl3)
    | _ => raise ListLengthMismatch

fun unzip lst = 
  case lst of
    [] => ([], [], [])
    | (a, b, c) :: tl => 
      let val (l1, l2, l3) = unzip tl
      in (a :: l1, b :: l2, c :: l3) end

fun multsign (x1, x2) = (* int * int -> sgn *)
  let fun sign x = if x = 0 then Z else if x > 0 then P else N
  in
    case (sign x1, sign x2) of
      (Z, _) => Z
    | (_, Z) => Z
    | (P, P) => P
    | (N, N) => P
    | (N, P) => N
    | (P, N) => N
  end
```

1. We can nest patterns as deep as we want. The full meaning of pattern matching is to compare a pattern against a value for the same shape and bind variables to the right parts.
2. A common idiom is matching against a tuple of datatypes to compare them.
3. Wildcards are good style: use them instead of variables when you do not need the data.
4. The semantics for pattern-matching takes a pattern p and a value v and decides does it match and if so what variable bindings are introduced. Since patterns can nest, the definition is elegantly recursive, with a separate rule for each kind of pattern.
5. Function patterns: syntactic sugar of case expression.

```sml
fun f p1 = e1
  | f p2 = e2
  | f pn = en

fun append ([], ys) = ys
  | append (x :: xs', ys) = x :: append (xs', ys)
```

### Exceptions

```sml
fun maxlist (xs, ex) = (* int list * exn -> int *)
  case xs of
    [] => raise ex
  | x :: [] => x
  | x :: xs' => Int.max(x, maxlist (xs', ex))

exception MyException of int * int
exception MySimpleException

val x = maxlist ([3, 4, 5], MySimpleException)
        handle MySimpleException => 0

val w = maxlist ([], MyException (2, 3))
        handle MyException (x, y) => x + y
```

1. Exceptions are a lot like datatype constructors.
2. Declaring an exception makes adds a constructor for type `exn`.
3. Can pass values of `exn` anywhere (e.g. function arguments).
4. Handle can have multiple branches with patterns for type `exn`.

### Tail Recursion

```sml
fun fact n = 
  let fun aux (n, acc) =
    if n = 0
    then acc
    else aux (n - 1, acc * n)
  in aux (n, 1) end

fun rev' xs = 
  case xs of
    [] => []
  (* append must traverse the first list *)
  | x :: xs' => (rev' xs') @ [x]

fun rev xs = 
  let fun aux (xs, acc) =
    case xs of
      [] => acc
    | x :: xs' => aux (xs', x :: acc)
  in aux (xs, []) end
```

1. While a program runs, there is a call stack of function calls that have started but not yet returned.
- Calling a function f pushes an instance of f on the stack.
- When a call to f finished, it is popped from the stack.
- These stack-frames store information like the value of local variables and what is left to do in function. Due to recursion, multiple stack-frames may be calls to the same function.
2. An optimization: It is unnecessary to keep around a stack-frame just so it can get a callee's result and return it without any further evaluaion. ML recognizes these tail calls in the compiler and treats them differently:
- Pop the caller before the call, allowing callee to reuse the same stack space (tail-calls: nothing left for caller to do, so the pop caller).
- Along with other optimizations, as efficient as a loop.
![](../images/sml/tail.png)
3. Accumulators for tail recursion: 
- Tail-recursive: recursive calls are tail-calls.
- A methodology that can often guide this transformation: Create a helper function that takes an accumulator. Old base case becomes initial accumulator. New base case becomes final accumulator.
4. There are certainly cases where recursive functions cannot be evaluated in a constant amount of space. Most obvious examples are functions that process trees. In these cases, the natural recursive approach is the way to go.
5. A tail call is a function call in tail position. If an expression is not in tail position, then no subexpressions are. The nothing left for caller to do intuition usually suffices.

### First-Class Functions

```sml
fun n_times (f, n, x) = 
  if n = 0 
  then x
  else f (n_times (f, n - 1, x))

fun increment x = x + 1
fun double x = x + x

val x1 = n_times (double, 4, 7)
val x2 = n_times (increment, 4, 7)
val x3 = n_times (tl, 2, [4, 8, 12, 16])

fun triple_n_times (n, x) = 
  n_times ((fn x => 3 * x), n, x)
```

1. Functions are values too. Higher order functions are the most useful ones when you want to abstract over what to compute with.
2. Function closure: Functions can use bindings from outside the function definition (in scope where function is defined).
3. Anonymous functions: most commonly used as argument to a higher-order function. Do not need a name just to pass a function. Can not use an anonymous function for a recursive function. If not for recursion, `fun` bindings would be syntatic sugar for `val` bindings and anonymous functions.
```sml
fun triple x = 3 * x
val triple = fn y => 3 * y
```
4. Map and filter: predefined in `List`.
```sml
(* ('a -> 'b) * 'a list -> 'b list *)
fun map (f, xs) =
  case xs of
    [] => []
    | x :: xs' => (f x) :: map (f, xs')

val x1 = map ((fn x => x + 1), [4, 8, 12])
val x2 = map (hd, [[1, 2], [3, 4], [5, 6, 7]])

(* ('a -> bool) * 'a list -> 'a list *)
fun filter (f, xs) = 
  case xs of
    [] => []
    | x :: xs' => if f x
                  then x :: (filter (f, xs'))
                  else filter (f, xs')

fun is_even v = 
  (v mod 2 = 0)

fun all_even xs = filter (is_even, xs)

fun all_even_snd xs = filter ((fn (_, v) => is_even v), xs)
```
5. Fold: (synonyms / close relatives reduce, inject, etc.) is another very famous iterator over recursive structures. Accumulates an answer by repeatedly applying f to answer so far. These iterator-like functions are not built into SML. It is just a programming pattern. This pattern separates recursive traversal from data processing. Can reuse same traversal for different data processing. Can reuse same data processing for different data structures.
```sml
fun fold (f, acc, xs) = 
  case xs of 
    [] => acc
    | x :: xs => fold (f, f (acc, x), xs)
```

### Lexical Scope

1. Closures:
```sml
val x = 1 (* x maps to 1 *)
fun f y = x + y (* f maps to a function that adds 1 to its argument *)
val x = 2 (* x maps to 2 *)
val y = 3 (* y maps to 3 *)
val z = f (x + y) (* call the function defined on line 4 with 5 *) 
(* z maps to 6 *)
```
- How can functions be evaluated in old enviroments that are not around anymore? The language implementation keeps them around as necessary. Can define the semantics of functions as follows:
  - A function value has two parts: the code and the environment that was current when the function was defined.
  - This is a pair but unlike ML pairs. You cannot access the pieces. All you can do is call this pair. This pair is called a function closure.
  - A call evaluates the code part in the environment part (extended with the function argument).
- Line 2 creates a closure and binds f to it.
  - Code: take y and body x + y
  - Environment: x maps to 1. (Plus whatever else is in scope, including f for recursion)
- Line 5 calls the closure defined in line 2 with 5. So body evaluated in environment x maps to 1 extended with y maps to 5.
2. Lexical scope and higher-order functions: the rule stays the same. A function body is evaluated in the environment where the function was defined. (Extended with the function argument). Nothing changes to this rule when we take and return functions. (But the environment may involve nested let-expressions, not just the top-level sequence of bindings). Makes first-class functions much more powerful.
3. Why lexical scope:
- Lexical scope: use environment where function is defined.
- Dynamic scope: use environment where function is called.
- Function meaning does not depend on variable names used:
  - Can change body of f to use q everywhere instead of x.
  - Can remove unused variables.
- Functions can be type-checked and reasoned about where defined.
- Closures can easily store the data they need.
- Lexical scope for variables is definitely the right default (very common across languages). Dynamic scope is occassionally convenient in some situations. So some languages (e.g. Racket) have special ways to do it. Exception handling is more like dynamic scope. `raise e` transfers control to the current innermost handler. Does not have to be syntactically inside a handle expression (and usually is not).

### Closures and Recomputation

1. When things evaluate: a function body is not evaluated until the function is called. A function body is evaluated every time the function is called. A variable binding evaluates its expression when the bining is evaluated, not every time the variable is used. With closures, this means we can avoid repeating computations that do not depend on function arguments.
```sml
fun filter (f, xs) =
  case xs of
    [] => []
    | x :: xs' => if f x
                  then x :: filter (f, xs')
                  else filter (f, xs')

fun allShorterThan1 (xs, s) = 
  filter (fn x => String.size x < String.size s, xs)

fun allShorterThan2 (xs, s) =
  let 
    val i = String.size s
  in 
    filter (fn x => String.size x < i, xs) 
  end
```
2. Closure Idioms:
- Pass functions with private data to iterators
- Combining functions (e.g. composition)
```sml
fun compose (f, g) = fn x => f (g x)
fun sqrt_of_abs i = Math.sqrt (Real.fromInt (abs i))
fun sqrt_of_abs' i = (Math.sqrt o Real.fromInt o abs) i
val sqrt_of_abs'' = Math.sqrt o Real.fromInt o abs

infix !>
fun x !> f = f x
fun sqrt_of_abs''' i = i !> abs !> Real.fromInt !> Math.sqrt

fun backup1 (f, g) = fn x => case f x of 
                                NONE => g x
                              | SOME y => y

fun backup2 (f, g) = fn x => f x handle _ => g x
```
- Currying (multi-arg functions and partial application). Efficiency: SML/NJ compiles tuples more efficiently. But many other functional language implementations do better with currying (OCaml, F#, Haskell).
```sml
val sorted3 = fn x => fn y => fn z => z >= y andalso y >= x

val t1 = ((sorted3 7) 9) 11
val t2 = sorted3 7 9 11

fun sorted3_nicer x y z = z >= y andalso y >= x

fun fold f acc xs = 
  case xs of
    [] => acc
    | x :: xs' => fold f (f(acc, x)) xs'

fun sum xs = fold (fn (x, y) => x + y) 0 xs 

fun exists predicate xs =
  case xs of
    [] => false
  | x :: xs' => predicate x orelse exists predicate xs'

val hasZero = exists (fn x => x = 0)

fun curry f = fn x => fn y => f (x, y)
fun curry1 f x y = f (x, y)

fun uncurry f (x, y) = f x y
```
- Callbacks (e.g. in reactive programming): Library takes functions to apply later, when an event occurs. 
- Implementing an ADT with a record of functions.
3. Closure idioms without closures:
- Higher-order programming is great.
```sml
datatype 'a mylist = Cons of 'a * ('a mylist) | Empty

fun map f xs =
  case xs of
    Empty => Empty
  | Cons (x, xs) => Cons (f x, map f xs)

fun filter f xs = 
  case xs of
    Empty => Empty
  | Cons (x, xs) => if f x
                    then Cons (x, filter f xs)
                    else filter f xs

fun length xs = 
  case xs of
    Empty => 0
  | Cons (x, xs) => 1 + length xs

val doubleAll = map (fn x => x * 2)

fun countNs (xs, n : int) = length (filter (fn x => x = n) xs)
```
- In OOP with one-method interfaces. An interface is a named [polymorphic] type. An object with one method can serve as a closure.
```java
interface Func<B, A> {
  B m(A x);
}

interface Pred<A> {
  boolean m(A x);
}

class List<T> {
  T head;
  List<T> tail;

  List(T x, List<T> xs) {
    head = x;
    tail = xs;
  }

  static <A, B> List<B> map(Func<B, A> f, List<A> xs) {
    if (xs == null)
      return null;
    return new List<B>(f.m(xs.head), map(f, xs.tail));
  }

  static <A> List<A> filter(Pred<A> f, List<A> xs) {
    if (xs == null)
      return null;
    if (f.m(xs.head))
      return new List<A>(xs.head, filter(f, xs.tail));
    return filter(f, xs.tail);
  }

  static <A> int length(List<A> xs) {
    int ans = 0;
    while (xs != null) {
      ++ans;
      xs = xs.tail;
    }
    return ans;
  }
}

public class Clients {
  static List<Integer> doubleAll(List<Integer> xs) {
    return List.map((new Func<Integer, Integer>() {
      public Integer m(Integer x) {
        return x * 2;
      }
    }), xs);
  }

  static int countNs(List<Integer> xs, final int n) {
    return List.length(List.filter((new Pred<Integer>() {
      public boolean m(Integer x) {
        return x == n;
      }
    }), xs));
  }
}
```
- In procedural with explicit environment arguments. In C, a function pointer is only a code pointer. A common technique: always define function pointers and higher-order functions to take an extra explicit environment argument. But without generics, no good choice for type of list elements or the environment. Use `void*` and various type casts...
```c
typedef struct List list_t;

struct List
{
  void *head;
  list_t *tail;
};

list_t *makelist(void *x, list_t *xs)
{
  list_t *ans = (list_t *)malloc(sizeof(list_t));
  ans->head = x;
  ans->tail = xs;
  return ans;
}

list_t *map(void *(*f)(void *, void *), void *env, list_t *xs)
{
  if (xs == NULL)
    return NULL;
  return makelist(f(env, xs->head), map(f, env, xs->tail));
}

list_t *filter(bool (*f)(void *, void *), void *env, list_t *xs)
{
  if (xs == NULL)
    return NULL;
  if (f(env, xs->head))
    return makelist(xs->head, filter(f, env, xs->tail));
  return filter(f, env, xs->tail);
}

int length(list_t *xs)
{
  int ans = 0;
  while (xs != NULL)
  {
    ++ans;
    xs = xs->tail;
  }
  return ans;
}

void *doubleInt(void *ignore, void *i)
{
  return (void *)(((intptr_t)i) * 2);
}

// assumes list holds intptr_t fileds
list_t *doubleAll(list_t *xs)
{
  return map(doubleInt, NULL, xs);
}

// type casts to match what filter expects
bool isN(void *n, void *i)
{
  return ((intptr_t)n) == ((intptr_t)i);
}

int countNs(list_t *xs, intptr_t n)
{
  return length(filter(isN, (void *)n, xs));
}
```

### Mutable References

1. New types: `t ref` where `t` is a type
2. New expressions:
- `ref e` to create a reference with initil contents `e`
- `e1 := e2` to update contents
- `!e` to retrieve contents (not negation)
3. A variable bound to a reference is still immutable: it will always refer to the same reference. But the contents of the reference may change via :=. And there may be aliases to the reference, which matters a lot. Reference are first-class values. Like a one-field mutable object, := and ! do not specify the field.

### Type Inference

1. Type inference problem: give every binding/expression a type such that type-checking succeeds. Fail if and only if no solution exists. In principle, could be a pass before the type-checker. But often implemented together. 
2. Central feature of ML type inference: it can infer types with type variables - relation to polymorphism. ML is in a sweet pot because type inference is more difficult without polymorphism or with subtyping.
3. Key steps: 
- Determine types of bindings in order (except for mutual recursion). So you can not use later bindings: will not type-check.
- For each `val` or `fun` binding:
  - Analyze definition for all necessary facts (constraints)
  - Example: if see `x > 0` then x must be type int
  - Type error if no way for all facts to hold (over-constrained)
- Afterward, use type variables (e.g. 'a) for any unconstrained types. Example: an unused argument can have any type.
- Finally, enforce the value restriction. A variable-binding can have a polymorphic type only if the expression is a variable or value. Else get a warning and unconstrained types are filled in with dummy types (basically unusable). The value restriction can cause problems when it is neccessary because we are not using mutation (Usually wrap in a function binding to fix it).

### Mutual Recursion

1. Allow f to call g and g to call f:
```sml
fun f1 p1 = e1
and f2 p2 = e2
and f3 p3 = e3

datatype t1 = ...
and t2 = ...
and t3 = ...
```
2. State-machine: each state of the computation is a function. State transition is call another function with rest of input. Generalizes to any finite-state-machine example.
```sml
fun match xs =
  let 
    fun s_need_one xs =
      case xs of
        [] => true
      | 1 :: xs' => s_need_two xs'
      | _ => false
    and s_need_two xs = 
      case xs of
        [] => false
      | 2 :: xs' => s_need_one xs'
      | _ => false
  in s_need_one xs end
```

### Modules

1. ML has structures to define modules. `structure MyModule = struct bindings end`. This is namespace management. Can use `open ModuleName` to get direct access to a module bindings. But often better to create local val-bindings for just the bindings.
2. A signature is a type for a module (what bindings does it have and what are their types). Can define a signature and ascribe it to modules. Real value of signatures is to hide bindings and type definitions.
```sml
signature MATHLIB =
sig
  val fact : int -> int
  val half_pi : int
end

structure MyMathLib :> MATHLIB =
struct
  fun fact x =
    if x = 0
    then 1
    else x * fact (x - 1)
  
  val half_pi = Math.pi / 2.0

  fun doubler y = y + y

  val eight = doubler 4
end
```
3. Modules with the same signatures still define different types. Different modules have different internal invariants. In fact, they have different type definitions.

### Equivalence

1. It is much easier to be equivalent if there are fewer possible arguments, e.g. with a type system and abstraction or avoiding side-effects: mutation, I/O, exceptions...
2. Different definitions of equivalence:
- PL Eq: given same inputs, same outputs and effects.
- Asymptotic Eq: ignore constant factors.
- Systems Eq: Account for constant overheads, performance tune.

> Note: for relevant code and more, see https://github.com/raptazure/pluw
