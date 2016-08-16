# Hadoop basics

The main resources are:
* Internet
* *Hadoop: The Definitive Guide*, 4th edition, Tom White (March 2015)

## HDFS (Hadoop Distributed Filesystem)

is good for: **Very larg files, Streaming data access** (reading large part of data instead of seeking), **Commodity harware**

is not good where: **Low-latency data access** (alternative: HBase), **Lots of small files** and **Multiple writes  and file modifications** are required. 

The size of blocks for a disks are 512 byes, for FS usually around ~few KB, whereas for HDFS the default is 128 MB.

### HDFS daemons
* daemons running on Master nodes:
  * **NameNode**: maintains the filesystem tree and the metadata for all the files and directories in the tree. The namespace image and the edit log is stored on the local disk, but not block locatoins (which is reconstructed after system starts). Web UI: *http://localhost:50070/*
  * **Secondary NameNode**: periodically merges namesapce image with the edit log. Also keeps a copy of the merged namespace image, which can be used in case of NameNode failure. Note: the secondary NameNode lags the primary so in cas of total failure of the primary, the backup namenode's metadata files that are on NFS has to be used.
* daemons running on Slave nodes:
  * **DataNode**: store and retrieve blocks, and report back to the NameNode periodically with lists of blocks that thery are stroing.

TODO: add some notes on Hadoop HA (high availability), QJM (quorum journal manager) and failover and fencing.

#### Usefull commands
* hadoop fs -cat URI
* hadoop fs -checksum URI
* hadoop fs -copyFromLocal <localsrc> URI
* hadoop fs -copyToLocal [-ignorecrc] [-crc] URI <localdst>
* hadoop fs -count [-q] [-h] [-v] <paths>     (Count the number of directories, files and bytes under the paths that match the specified file pattern. )
* hadoop fs -df [-h] URI [URI ...]
* hadoop fs -du [-s] [-h] URI [URI ...]
* hadoop fs -find <path> ... <expression> ...
* hadoop fs -ls [-d] [-h] [-R] <args>
* hadoop fs -put <localsrc> ... <dst>
* hadoop fs -get [-ignorecrc] [-crc] <src> <localdst>
* hadoop fs -test -[defsz] URI
  * -d: f the path is a directory, return 0.
  * -e: if the path exists, return 0.
  * -f: if the path is a file, return 0.
  * -s: if the path is not empty, return 0.
  * -z: if the file is zero length, return 0.

For more commands and explanation look [here](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/FileSystemShell.html).

NOTE: **`hadoop fs` vs `hadoop dfs` vs `hdfs dfs`**: (from [here](http://stackoverflow.com/questions/18142960/whats-the-difference-between-hadoop-fs-shell-commands-and-hdfs-dfs-shell-co/24404136#24404136))
* **`hadoop fs <args>`** : `fs` relates to a generic file system (i.e. Local FS, HFTP FS, S3 FS). 
* **`hadoop dfs <args>`** :  `dfs` is very specific to HDFS (this command depricated, instead use `hdfs dfs`). 
* **`hdfs dfs <args>`** : specific to HDFS.

### MapReduce-1 daemons
* daemons running on Master nodes:
  * **JobTracker**: scheduling tasks to run on TaskTrackers. Task progress monitoring (e.g. handling failed tasks).
* daemons running on Slave nodes:
  * **TaskTracker**s: run tasks and send progress report to the jobtracker.

### YARN daemons
* **ResourceManager**: The ResourceManager is the ultimate authority that arbitrates resources among all the applications in the system. Web UI: *http://localhost:8088/*
* **ApplicationMaster (AM)**: per-application daemon, which is responsible for negotiating resources from the ResourceManager and execute and monitor the tasks.
* **NodeManager (NM)**: is per-machine slave, which is responsible for launching the applications’ containers, monitoring their resource usage.

Advantage of YARN over MapReduce-1:
* Scalability: 4,000 noeds - 40,000 tasks => 10,000 nodes - 100,000 tasks.
* Availability: Hadoop 2 supports HA for both ResourceManager and ApplicationMaster.
* Utilization: in contrast to fixed-size "slots" in MR-1, NodeManager manages a pool of resources.
* Multitenancy: opens up Hadoop to other types of distributed application than MR.

## YRAN

### Scheduler
In CDH the default scheduler is Fair Scheduler.
If not it can be changed in *yarn-site.xml*:
```xml
<property>
  <name>yarn.resourcemanager.scheduler.class</name>
  <value>org.apache.hadoop.yarn.server.resourcemanager.scheduler.fair.FairScheduler</value>
</property>
```
or `org.apache.hadoop.yarn.server.resourcemanager.scheduler.capacity.CapacityScheduler` for Capacity Scheduler.

* **FIFO Scheduler**: First in first out.
* **Capacity Scheduler**: Separate queues can be configured. Each queue will be a FIFO.
 * Each organization can set up dedicated queue. 
 * Organization's queue can be further split up between users.
 * *queue elasticity*: if there is more than one job in the queue and there are idle resources availible then scheduler may allocate more resources that queue's capacity. The level of elasticity is configurable.
 * Config file: *capacity-scheduler.xml*.
 * If maximum capacity is not specified for the queue, it may use all the capacity of the uder utulized queue.
 * To place a job in a queue in MapReduce the property `mapreduce.job.queuename` is used. The name of the queue is the last part of the hierarchical name.
* **Fair Scheduler**: When second job starts the resources will be shared between the two (with a lag for the second job as some of tasks of the first job should be finished to release the resources).
  * Config file: *fair-scheduler.xml*.
  * Queue placement:
  * Preemption:
  * Delay scheduling:
  * Dominant Resource Faiarness:

## Hadoop IO

### Integrity 

* **HDFS**: HDFS uses CRC-32C for checksum. DataNodes are responsible for checksum verification.
* **`LocalFileSystem`**: Performs client-side checksumming. To disable checksumming use instead `RawLocalFileSystem`.
* **`ChecksumFileSystem`**: creates a checksum file for each raw file. It is just a wrapper around `FileSystem`.
* Use command `hadoop fs -checksum URI` to see the checksum of the given file.

### Compression

Pros:
* reduces space
* speeds up data transfer
Cons:
* time needed for compression and decompression

| Compression format | Tool  | Algorithm | File extention | Splittable | Codec                                      |
| ------------------ | ----- | --------- | -------------- | ---------- | ------------------------------------------ |
| DEFLATE            | N/A   | DEFLATE   | .deflate       | No         | org.apache.hadoop.io.compress.DeflateCodec |
| gzip               | gzip  | DEFLATE   | .gz            | No         | org.apache.hadoop.io.compress.GzipCodec    |
| bzip2              | bizp2 | bzip2     | .bz2           | YES        | org.apache.hadoop.io.compress.BZip2Codec   |
| LZO                | lzop  | LZO       | .lzo           | if indexed | com.hadoop.compression.lzo.LzopCodec       |
| LZ4                | N/A   | LZ4       | .lz4           | NO         | org.apache.hadoop.io.compress.Lz4Codec     |
| Snappy             | N/A   | Snappy    | .snappy        | No         | org.apache.hadoop.io.compress.SnappyCodec  |

NOTE: If compression is not splittable then in a MR task the whole file should be processed with one MR-task.

Suggestions on compression format, in order of effectivness:
* Using container file fomrmats like: sequence files, Avro, ORCFiles, Parquet. Use conpressors like LZO, LZ4, or Snappy as splitability is not a factor here.
* Compressors that supports splitting, like bzip2 or LZO with indexing.
* Split the files in chunks, and copress the chunks separately using any compression, also if is not splittable.
* Store the file uncompressed.

### Serialization

Hadoop uses `Writable` for serialization.

One of the advantages of `Writable` over `java.io.Serializable` is that in `Writable` the expected class is known and therefore for some opperatin in MR like soriting it does not need to deserialize the object.

| Java primitive | Writable implementation	| Serialized size (bytes) |
| -------------- | ----------------------- | ----------------------- |
| boolean	       | BooleanWritable	        | 1                       |
| byte	          | ByteWritable	           | 1                       |
| short	         | ShortWritable	          | 2                       |
| int	           | IntWritable	            | 4                       |
| 	              | VIntWritable	           | 1–5                     |
| float	         | FloatWritable	          | 4                       |
| long	          | LongWritable	           | 8                       |
| 	              | VLongWritable	          | 1–9                     |
| double	        | DoubleWritable	         | 8                       |

### Persistent data structure
| Data structure | Type	                | Orientation  | Note |
| -------------- | -------------------- | -----------  | ----------------------- |
| SequenceFile	  | Binary key/value	    | row | Variants: *Uncompressed* format, *Record Compressed* format and *Block-Compressed* |
| MapFile	       | Sorted SequenceFile  | row | Variants: `SetFile`, `ArrayFile` and `BloomMapFile` |
| Avro	          | Binary data serialization | row    | portable across differnt programming languages |
| ORCFile        | | column | |
| Parquet        | | column | based on google's Dremel|


