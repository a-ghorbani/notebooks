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
  * **NameNode**: maintains the filesystem tree and the metadata for all the files and directories in the tree. The namespace image and the edit log is stored on the local disk, but not block locatoins (which is reconstructed after system starts).
  * **Secondary NameNode**: periodically merges namesapce image with the edit log. Also keeps a copy of the merged namespace image, which can be used in case of NameNode failure. Note: the secondary NameNode lags the primary so in cas of total failure of the primary, the backup namenode's metadata files that are on NFS has to be used.
* daemons running on Slave nodes:
  * **DataNode**: store and retrieve blocks, and report back to the NameNode periodically with lists of blocks that thery are stroing.

TODO: add some notes on Hadoop HA (high availability), QJM (quorum journal manager) and failover and fencing.

### MapReduce-1 daemons
* daemons running on Master nodes:
  * **JobTracker**: scheduling tasks to run on TaskTrackers. Task progress monitoring (e.g. handling failed tasks).
* daemons running on Slave nodes:
  * **TaskTracker**s: run tasks and send progress report to the jobtracker.

### YARN daemons
* **ResourceManager**: The ResourceManager is the ultimate authority that arbitrates resources among all the applications in the system. 
* **ApplicationMaster (AM)**: per-application daemon, which is responsible for negotiating resources from the ResourceManager and execute and monitor the tasks.
* **NodeManager (NM)**: is per-machine slave, which is responsible for launching the applications’ containers, monitoring their resource usage.

Advantage of YARN over MapReduce-1:
* Scalability: 4,000 noeds - 40,000 tasks => 10,000 nodes - 100,000 tasks.
* Availability: Hadoop 2 supports HA for both ResourceManager and ApplicationMaster.
* Utilization: in contrast to fixed-size "slots" in MR-1, NodeManager manages a pool of resources.
* Multitenancy: opens up Hadoop to other types of distributed application than MR.
