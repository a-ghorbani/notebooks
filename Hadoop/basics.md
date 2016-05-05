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
  * NameNode 
  * Secondary NameNode
  * JobTracker
* daemons running on Slave nodes:
  * DataNode 
  * TaskTracker
