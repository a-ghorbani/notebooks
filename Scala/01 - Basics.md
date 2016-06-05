
# Intro

Scala is a general purpose programming language with the following properties (not limited to):
* Multi-paradigm (Object-Oriented, Functional, Imperative, Concurrent)
* Strong static type system
* Concise
* Source code is compiled to Java bytecode
* Executable code runs on a JVM

Scala was designed by Martin Odersky and first appeared on January 20, 2004.
The name Scala is a portmanteau of "scalable" and "language".

# Examples of functional features

## Except few, all other statements are expressions 

* `if-else`, `while` and `throw` are expressions.
  Types :
  * **if-else**: common supertype of all the branches.
  * **while**: `Unit`.
  * **throw**: `Nothing`.

example:
```Scala
val result: String = if(marks >= 50) "passed" else "failed"
```

## Type inference

In Scala the type of variables, function return values, and many other expressions can typically be omitted.

```Scala
object typeInference {
  val x = 1 + 2 * 3         // the type of x is Int
  val y = x.toString()      // the type of y is String
  def succ(x: Int) = x + 1  // method succ returns Int values
}
```
is equivalent to
```Scala
object typeInference {
  val x: Int = 1 + 2 * 3         
  val y: String = x.toString()    
  def succ(x: Int): Int = x + 1 
}
```

## Anonymous functions

Anonymous function is a function definition that is not bound to an identifier, which are often:

1. arguments being passed to higher-order functions, or
2. used for constructing the result of a higher-order function that needs to return a function.

example:
```Scala
x => x + 1
```
in
```Scala
myList.map(x => x + 1)
```

or 

```Scala
def myMapFunction(a: Int) = {x: Int => x + a}
myList.map(myMapFunction(2))
```

## Immutability

In Scala `var` is used to define mutable variable and `val` for immutable variables (indeed not variable ;) ).

Function arguments are `val` by default.

Scala collections systematically distinguish between mutable and immutable collections. 
A mutable collection can be updated or extended in place. 
Immutable collections, by contrast, never change.

By default in scala environment immutable collections are loaded.
For mutable collections you need to import them manually.

If you want to use both mutable and immutable versions of collections,
a useful conventionis to import just the package `collection.mutable`.

```Scala
import scala.collection.mutable

val myImmutableSet = Set(1,2,3)
val myMutableSet = mutable.Set(1,2,3)
```

> #### A balanced attitude for Scala programmers
>     Prefer vals, immutable objects, and methods without side effects. 
>     Reach for them first. Use vars, mutable objects, and methods with 
>     side effects when you have a specific need and justification for them.

> M Odersky, L Spoon, B Venners *"Programming in Scala"*, second edition, p. 98

## Lazy evaluation
## Delimited continuations (since 2.8)
## Higher-order functions
## Nested functions
## Currying
## Pattern matching
## Algebraic data types (through "case classes")
## Tuples
