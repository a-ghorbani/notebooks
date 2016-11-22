
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

* `reduceByKey()`: input function should be commutative and associative, i.e. grouping and order should not matter. The results of aggregation has the same type of each elements of rdd.  
   Example: `rdd.reduceByKey(_+_)`  

* `foldByKey()`: same as `reduceByKey()` but accepts initial value (a natural zero).  
   Example: `rdd.foldByKey(0)(_+_)`

* `aggregateByKey()`: aggregate values for each key, and potentially can return different value type.  
   Example: `rdd.aggregateByKey(new HashSet[Int])(_+=_, _++=_)`  
   `new HashSet[Int]`: creates a new mutalbe set  
   `_+=_`: adds a value to a `HashSet[Int]` for each partition   
   `_++=_`: adds all the elements of the second set to the first one in each combiner in the map task.
   
* `combineByKey()`: is more general than `aggregateByKey`. It allows to define an initial lambda function to create the initial accumulator.  
   Example:  
   ```
   rdd.combineByKey(HashSet[Int](_),  
                     (aggr: HashSet[Int], value) => aggr+=value,  
                     (aggr1: HashSet[Int], aggr2: HashSet[Int]) => aggr1++=aggr2)
   ```
   
  
