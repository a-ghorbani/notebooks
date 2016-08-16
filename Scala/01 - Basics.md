
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

You can declare a variable lazy with the `lazy` keyword.
```Scala 
val x = {println ("x defined as 1 ") ; 1}
lazy val y = {println ("y defined as 2"); 2}

println("The variables x and y were defined.")
println(x)
println(y) 

// Output:
//
// x defined as 1 
// The variables x and y were defined.
// 1
// y defined as 2
// 2

```

get the point?

## Higher-order functions

A higher-order function is a function that does at least one its input (arguments) or output is a function.

## Currying
Scala also supports [Currying](https://en.wikipedia.org/wiki/Currying).

Example:
```Scala
def myParameterizedMapFunction(a: Int)(x: Int) = x + a
myList.map(myParameterizedMapFunction(2))
```
In the above example `myParameterizedMapFunction(2)` returns a function which is `x: Int => x + 2`.

## Pattern matching
Scala allows to match on any sort of data with a first-match policy.

Indeed, the Scala’s pattern matching in a very powerfull switch-case statement, that can cover different types of comparision, like numeric values, string values, types etc.

A nice discussion [here](https://kerflyn.wordpress.com/2011/02/14/playing-with-scalas-pattern-matching/)

Few examples:
```Scala
def parseArgument(arg: String) = arg match {
    case "-h" | "--help" => println("Heeeeeelp!")
    case "-v" | "--version" => println("supversion")
    case whatever => println("O_O You typed: " + whatever)
  }
  
def fact(n: Int): Int = n match {
    case 0 => 1
    case m => m * fact(m - 1)
  }
```

## Algebraic data types

Algebraic data types are modelling data in two pattern:

* *or* or *sum* type
* *and* or *product* type

Example:
```Scala
sealed trait SumADT
final case class type1(a: Int) extends SumADT
final case class type2(a: Int, b: String) extends SumADT

// example of AlgDT in pattern matching

val aSumADT: SumADT = aFunctionThatReturnsSumADT()

aSumADT match {
  case type1(a)   => doSomeThing(a)
  case type2(a,b) => doSomeThing(a,b)
}
```
In the above example `SumADT` is a **sum type** with possible types of `type1` and `type2`.
And `type2` is a **product type** of `Int` and `String`.

There is a good explanation of AlgDT [here](http://noelwelsh.com/programming/2015/06/02/everything-about-sealed/)
## Tuples
For examples see [here](http://www.tutorialspoint.com/scala/scala_tuples.htm)

Scala tuple combines a fixed number of items together so that they can be passed around as a whole.

```Scala
// define a tuple
val t = (1, "hello", Console)

// access tuple's item
println(t._1 + t._2 + t._3 )

// iterate over items
t.productIterator.foreach{ i =>println("Value = " + i )}
```
