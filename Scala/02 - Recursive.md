
# Scala recursion

In Scala there are two kinds of recursion, namely *head* and *tail* recursion. 

[Here](https://oldfashionedsoftware.com/2008/09/27/tail-recursion-basics-in-scala/) you may find a good discussion on this topic.

The important point in using recursion is to taking care of that *head* recursion, which might fail due to `StackOverflowError`.

Therefore, you need to make sure either you use *tail* recursion or if you use *head* recursion 
it should not be applied to a long list or something like that.

Example:

```Scala
def listSumHeadRec(list: List[Int]): Int = {
  if (list == Nil) 0
  else list.head + listSumHeadRec(list.tail)
}
```
The above recursive code is head recursion and will fail if the list is too big.

```Scala
def listSumTailRec(list: List[Int], init: Int = 0): Int = {
  if (list == Nil) init
  else listSumTailRec(list.tail, list.head + init)
}
```

```Scala
var shorList = 1 to 10 tolist
var longList = 1 to 100000 toList

listSumHeadRec(shorList) // works fine
listSumHeadRec(longList) // results in java.lang.StackOverflowError

listSumTailRec(shorList) // works fine
listSumTailRec(longList) // works fine
```

If you are writting a complex recursive code and you want make sure that it is a tail recursive use `@tailrec` annotation.

For example the following code will lead to compile time error:
```Scala
import scala.annotation.tailrec


@tailrec
def listSumHeadRec(list: List[Int]): Int = {
  if (list == Nil) 0
  else list.head + listSumHeadRec(list.tail)
}

// error: could not optimize @tailrec annotated method listSumHeadRec: 
// it contains a recursive call not in tail position
// else list.head + listSumHeadRec(list.tail)
//                ^
```
