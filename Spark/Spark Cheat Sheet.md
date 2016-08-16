
# Resilient Distributed Dataset (RDD)

RDD is an immutable distributed collection of data, which is partitioned across machines in a cluster.

## transformation 

* `filter()` -- Return a new RDD containing only the elements that satisfy a predicate.
  * `var newRdd = rdd.filter(line => line.contains("Spark"))`
* `map()` -- Return a new RDD by applying a function to all elements of this RDD.
  * `var newRdd = rdd.map(x => (x,x+1)) `
* `mapPartitions` -- Return a new RDD by applying a function to each partition of this RDD. (Better than map)
  * `var newRdd = rdd.mapPartitions(iter => iter.map(x => (x,x+1)))` same as above!
  * `var newRdd = rdd.mapPartitions(iter => for { (i) <- iter if(i<5) }yield (i,i+1) )`
  
* intersection(otherDataset)	Return a new RDD that contains the intersection of elements in the source dataset and the argument.
* distinct([numTasks]))	Return a new dataset that contains the distinct elements of the source dataset.
* groupByKey([numTasks])	When called on a dataset of (K, V) pairs, returns a dataset of (K, Iterable<V>) pairs. 
* reduceByKey(func, [numTasks])	When called on a dataset of (K, V) pairs, returns a dataset of (K, V) pairs where the values for each key are aggregated using the given reduce function func, which must be of type (V,V) => V. Like in groupByKey, the number of reduce tasks is configurable through an optional second argument.
* aggregateByKey(zeroValue)(seqOp, combOp, [numTasks])	When called on a dataset of (K, V) pairs, returns a dataset of (K, U) pairs where the values for each key are aggregated using the given combine functions and a neutral "zero" value. Allows an aggregated value type that is different than the input value type, while avoiding unnecessary allocations. Like in groupByKey, the number of reduce tasks is configurable through an optional second argument.
* sortByKey([ascending], [numTasks])	When called on a dataset of (K, V) pairs where K implements Ordered, returns a dataset of (K, V) pairs sorted by keys in ascending or descending order, as specified in the boolean ascending argument.
* join(otherDataset, [numTasks])	When called on datasets of type (K, V) and (K, W), returns a dataset of (K, (V, W)) pairs with all pairs of elements for each key. Outer joins are supported through leftOuterJoin, rightOuterJoin, and fullOuterJoin.
* cogroup(otherDataset, [numTasks])	When called on datasets of type (K, V) and (K, W), returns a dataset of (K, (Iterable<V>, Iterable<W>)) tuples. This operation is also called groupWith.
* coalesce(numPartitions)	Decrease the number of partitions in the RDD to numPartitions. Useful for running operations more efficiently after filtering down a large dataset.
* repartition(numPartitions)	Reshuffle the data in the RDD randomly to create either more or fewer partitions and balance it across them. This always shuffles all data over the network.
* repartitionAndSortWithinPartitions(partitioner)	Repartition the RDD according to the given partitioner and, within each resulting partition, sort records by their keys. This is more efficient than calling repartition and then sorting within each partition because it can push the sorting down into the shuffle machinery.

## action

* reduce(func)	Aggregate the elements of the dataset using a function func (which takes two arguments and returns one). The function should be commutative and associative so that it can be computed correctly in parallel.
* collect()	Return all the elements of the dataset as an array at the driver program. This is usually useful after a filter or other operation that returns a sufficiently small subset of the data.
* count()	Return the number of elements in the dataset.
* first()	Return the first element of the dataset (similar to take(1)).
* take(n)	Return an array with the first n elements of the dataset.
* takeSample(withReplacement, num, [seed])	Return an array with a random sample of num elements of the dataset, with or without replacement, optionally pre-specifying a random number generator seed.
* takeOrdered(n, [ordering])	Return the first n elements of the RDD using either their natural order or a custom comparator.
* saveAsTextFile(path)	Write the elements of the dataset as a text file (or set of text files) in a given directory in the local filesystem, HDFS or any other Hadoop-supported file system. Spark will call toString on each element to convert it to a line of text in the file.
* saveAsSequenceFile(path) 
(Java and Scala)	Write the elements of the dataset as a Hadoop SequenceFile in a given path in the local filesystem, HDFS or any other Hadoop-supported file system. This is available on RDDs of key-value pairs that implement Hadoop's Writable interface. In Scala, it is also available on types that are implicitly convertible to Writable (Spark includes conversions for basic types like Int, Double, String, etc).
* saveAsObjectFile(path) 
(Java and Scala)	Write the elements of the dataset in a simple format using Java serialization, which can then be loaded using SparkContext.objectFile().
* countByKey()	Only available on RDDs of type (K, V). Returns a hashmap of (K, Int) pairs with the count of each key.
* foreach(func)	Run a function func on each element of the dataset. This is usually done for side effects such as updating an Accumulator or interacting with external storage systems. 
Note: modifying variables other than Accumulators outside of the foreach() may result in undefined behavior. See Understanding closures for more details.
