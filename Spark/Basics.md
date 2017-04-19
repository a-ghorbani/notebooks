
# Intro

# A Scala Application

```Scala
import org.apache.spark.{SparkContext, SparkConf}
import org.apache.spark.SparkContext._ // for implicit conversion

object Main {
  val usage = """
    Usage: Main --input|-i <input_file_name> --output|-o <output_file_name>
  """

  def main(args: Array[String]) {

    // Parse arguments
    if (args.length != 4) println(usage)
    val arglist = args.toList
    type OptionMap = Map[Symbol, Any]
    def parseOption(map : OptionMap, list: List[String]) : OptionMap = {
      list match {
        case Nil => map
        case ("--input" | "-i")::value::tail =>
                               parseOption(map ++ Map('input -> value), tail)
        case ("--output" | "-o"):: value :: tail  =>
                               parseOption(map ++ Map('output -> value), tail)
        case option :: tail => println("Unknown option "+option)
                               exit(1)
      }
    }

    val options = parseOption(Map(),arglist)
    val inputFile = options('input).toString
    val outputFile = options('output).toString

    // Create a Scala Spark Context.
    val conf = new SparkConf().setAppName("wordCount")
    val sc = new SparkContext(conf)

    // Load our input data.
    sc.textFile(inputFile)                        // Load file
      .flatMap(line => line.split(" "))           // Split up into words
      .map(word => (word, 1))                     // Map each word to 1
      .reduceByKey{case (x, y) => x + y}          // Reduce by summing up all values
      .saveAsTextFile(outputFile)                 // Save the result back out to a file

  }
}
```

# Resilient Distributed Datasets (RDD)

can be created in three ways:

* from an in-memory collection: `val myRDD = sc.parallelize(myScalaCollection, 10)`
* creating a reference to an external data: `val myRDD = sc.textFile(myFilePath, 10)`
* transforming an existing RDD: `val myTransformedTDD = myRDD.map(myMapperFunction)`

# Lazy evaluation

No `transformation` is performed until an `action` operation.

* a transformation generates an RDD
* an action triggers computation on an RDD

# Aggregation

* `groupByKey()`: When called on a dataset of (K, V) pairs, returns a dataset of (K, Iterable<V>) pairs.  
   Avoid `groupByKey()` if you are grouping in order to perform an aggregation.  
   Example: list of salary in each department:  
   ```
   sc.parallelize(Seq(("dep1", 75000),("dep1", 90000),("dep2", 110000),("dep1", 85000)), 2)
     .groupByKey()
     .collect()

   // Array[(String, Iterable[Int])] = Array((dep1,CompactBuffer(75000, 90000, 85000)), (dep2,CompactBuffer(110000)))
   ```

* `reduceByKey()`: input function should be commutative and associative, i.e. grouping and order should not matter. The results of aggregation has the same type of each elements of rdd.  
   Example: max of salary in each department:  
   ```
   sc.parallelize(Seq(("dep1", 75000),("dep1", 90000),("dep2", 110000),("dep1", 85000)), 2)
     .reduceByKey(math.max)
     .collect()

   // Array[(String, Int)] = Array((dep1,90000), (dep2,110000))
   ```

* `foldByKey()`: same as `reduceByKey()` but accepts initial value (a natural zero) and in only makes sense if it is idempotent.  
   Example: `rdd.foldByKey(0)(_+_)`
   ```
   sc.parallelize(Seq(("dep1", 75000),("dep1", 90000),("dep2", 110000),("dep1", 85000)), 2)
     .foldByKey(0)(_+_)
     .collect()

   // Array[(String, Int)] = Array((dep1,250000), (dep2,110000))
   ```

* `aggregateByKey()`: aggregate values for each key, and potentially can return different value type.  
   Example 1: `rdd.aggregateByKey(new HashSet[Int])(_+=_, _++=_)`  
   `new HashSet[Int]`: creates a new mutalbe set  
   `_+=_`: adds a value to a `HashSet[Int]` for each partition   
   `_++=_`: adds all the elements of the second set to the first one in each combiner in the map task.  
   Example 2: average salary for each department
   ```
   sc.parallelize(Seq(("dep1", 75000),("dep1", 90000),("dep2", 110000),("dep1", 85000)), 2)
     .aggregateByKey((0,0))(
       (acc, value) => (acc._1 + value, acc._2 + 1),
       (acc1, acc2) => (acc1._1 + acc2._1, acc1._2 + acc2._2))
     .mapValues(x => x._1 / x._2)
     .collect()

   // Array[(String, Int)] = Array((dep1,83333), (dep2,110000))
   ```

* `combineByKey()`: is more general than `aggregateByKey`. It allows to define an initial lambda function to create the initial accumulator.  
   Example:  
   ```
   rdd.combineByKey(HashSet[Int](_),  
                    (aggr: HashSet[Int], value) => aggr+=value,  
                    (aggr1: HashSet[Int], aggr2: HashSet[Int]) => aggr1++=aggr2)
   ```
   Example 2:  
   ```
   sc.parallelize(Seq(("dep1", 75000),("dep1", 90000),("dep2", 110000),("dep1", 85000)), 2)
     .combineByKey(x => (x.toDouble,1),
       (acc: (Double, Int), value) => (acc._1 + value, acc._2 + 1),
       (acc1: (Double, Int), acc2: (Double, Int)) => (acc1._1 + acc2._1, acc1._2 + acc2._2))
     .mapValues(x => x._1 / x._2)
     .collect()

   // Array[(String, Double)] = Array((dep1,83333.33333333333), (dep2,110000.0))
   ```
# Cache

* `cache()` persists each partition of the RDD in the executor's memory.
* If there is not enough memory for an executor the partition will be recomputed.
* `persist()` allows different levels of persistence.
  * `MEMORY_ONLY`: like `cache()`, most CPU-efficient option
  * `MEMORY_ONLY_SER`: like `MEMORY_ONLY` + serialization
  * `MEMORY_AND_DISK`: spills to disk if the RDD doesn't fit in memory. Use it if reading from disk is faster than recomputing the RDD.
  * `MEMORY_AND_DISK_SER`: like `MEMORY_AND_DISK` + serialization.
  * `DISK_ONLY`	Store the RDD partitions only on disk.
  * `MEMORY_ONLY_2`, `MEMORY_AND_DISK_2`, etc.	Same as the levels above, but replicate each partition on two cluster nodes.
  * `OFF_HEAP` (experimental) : Store RDD in serialized format in [Alluxio](http://www.alluxio.org/) (formerly Tachyon).

* Spark automatically removes partitions in a least-recently-used (LRU) fashion.
* or use `RDD.unpersist()` to remove manually.

# Serialization

* Java serialization: By default, Spark serializes objects using Java’s ObjectOutputStream framework, and can work with any class you create that implements java.io.Serializable. You can also control the performance of your serialization more closely by extending java.io.Externalizable.
* Kryo serialization: Kryo is significantly faster and more compact than Java serialization (often as much as 10x), but does not support all Serializable types.

# Shared Variables

## Broadcast Variables

Broadcast variables allows a read-only variable be cached on each executer rather than shipping a copy of it with tasks.
Note that the broadcast variables cannot be updated.
```
val broadcastVar: Broadcast[Map[Int, String]] = sc.broadcast(Map(1 -> "a", 2-> "b", 3 -> "c"))

val myRdd = sc.parallelize(Array(1,1,1,3,3,2), 2)
val result = myRdd.map(broadcastVar.value(_))
result.collect()

// res91: Array[String] = Array(a, a, a, c, c, b)
```

## Accumulators

An accumulators are shared variables that only can be add to through an associative operation.

```
val accum = sc.accumulator(0, "My Accumulator")
sc.parallelize(1 to 100, 2).foreach(x => accum += x)

accum
// org.apache.spark.Accumulator[Int] = 5050
```

# Cluster

http://spark.apache.org/docs/latest/cluster-overview.html
