
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
    if (args.length != 2) println(usage)
    val arglist = args.toList
    type OptionMap = Map[Symbol, Any]
    def parseOption(map : OptionMap, list: List[String]) : OptionMap = {
      def isSwitch(s : String) = (s(0) == '-')
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
