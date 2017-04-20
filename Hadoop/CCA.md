# Data Ingest
The skills to transfer data between external systems and your cluster. This includes the following:

* Import data from a MySQL database into HDFS using Sqoop
  ```
  > sqoop import --connect jdbc:mysql://host/db \
                 --table tablename \
                 --target-dir /path/to/db/ \
                 --username user --password pass \
                 [--fields-terminated-by ',' \ ]
                 [--split-by colname \]
                 [--as-avrodatafile | --as-parquetfile | --as-sequencefile | --as-textfile \]
                 [--direct -- -u **** -p"*********"]

  ```
  * Load data into Hive 
     * From Hadoop 
        * text format 
         ```
         > sqoop create-hive-table --connect jdbc:mysql://host/db \
                                   --table tablename \
                                   --username user --password pass \
                                   --fields-terminated-by ','

         > beeline
         beeline> !connect jdbc:hive2://host:10000 user pass
         0: jdbc:hive2://host:10000> LOAD DATA INPATH "/path/to/file" INTO TABLE tablename; 
         ```
         * avro format 
         ```
         > java -jar /opt/cloudera/parcels/CDH/jars/avro-tools-1.7.6-cdh5.8.0.jar \
                 getschema hdfs://nm-host:8020/path/to/avro/node/part-m-00000.avro > tablename_avro.avsc
         > hdfs dfs -put tablename_avro.avsc /path/to/schemas/

         > beeline
         beeline> !connect jdbc:hive2://host:10000 user pass
         0: jdbc:hive2://host:10000> CREATE EXTERNAL TABLE test_avro STORED AS AVRO LOCATION '/path/to/avro/node' 
         . . . . . . . . . . . . . > TBLPROPERTIES('avro.schema.url'='hdfs:///path/to/schemas/tablename_avro.avsc');
         ```
     * Directly from MySQL 
     ```
     > sqoop import --connect jdbc:mysql://host/db \
                    --hive-import \
                    --table tablename \
                    --username user --password pass [\]
                    [--split-by colname \]
                    [--as-parquetfile \]
                    [--hive-database databasename \]
                    [--hive-table tablename \]
     ```
* Export data to a MySQL database from HDFS using Sqoop
  * export from HDFS to MySQL
  ```
  > sqoop export --connect jdbc:mysql://host/db \ 
                 --table tablename \
                 --username user --password pass \
                 --export-dir /path/to/data [\]
                 [--fields-terminated-by ',']
  ```
  * Export from Hive (specificly parquet format should be done this way)
  ```
  > sqoop export --connect jdbc:mysql://host/db \ 
                 --table tablename
                 --username user --password pass
                 --hcatalog-database hive-database 
                 --hcatalog-table hive-table
  ```
* Change the delimiter and file format of data during import using Sqoop
  * see the above commands
* Ingest real-time and near-real-time streaming data into HDFS
  * Flume
    * Have a look [here](https://github.com/a-ghorbani/notebooks/blob/master/Hadoop/Flume.md)
* Process streaming data as it is loaded onto the cluster
* Load data into and out of HDFS using the Hadoop File System commands
  * hdfs dfs -get /user/hadoop/file localfile
  * hdfs dfs -getmerge [-nl] /user/hadoop/dir localfile
  * hdfs dfs -put localfile /user/hadoop/file

# Transform, Stage, and Store
Convert a set of data values in a given format stored in HDFS into new data values or a new data format and write them into HDFS.
* Load RDD data from HDFS for use in Spark applications
  ```
  sc.textFile("/user/path/to/file")
  ```
* Write the results from an RDD back into HDFS using Spark
  ```
  myRDD.saveAsTextFile("/user/path/to/file"
  ```
* Read and write files in a variety of file formats
  * squenceFile
  ```
  sc.sequenceFile("/user/path/to/file")
  myRDD.saveAsSequenceFile("/user/path/to/file")
  ```
  * ORC
  ```
  myRDD.toDF().write.orc("/user/path/to/ORCfile")
  sqlContext.read.orc("/user/path/to/ORCfile")
  ```
  * JSON
  ```
  myRDD.toDF().write.json("/user/path/to/JSONfile")
  sqlContext.read.json("/user/path/to/JSONfile")
  ```
  * parquet
  ```
  myRDD.toDF().write.parquet("/user/path/to/parquetfile")
  sqlContext.read.parquet("/user/path/to/parquetfile")
  ```
  * Avro
  ```
  import com.databricks.spark.avro._
  myRDD.toDF().write.avro("/user/path/to/Avrofile")
  sqlContext.read.avro("/user/path/to/Avrofile")
  ```
    > Limitations
    >  Because Spark is converting data types, keep the following in mind:
    >
    >  Enumerated types are erased - Avro enumerated types become strings when they are read into Spark because Spark does not support enumerated types.
    >  Unions on output - Spark writes everything as unions of the given type along with a null option.
    >  Avro schema changes - Spark reads everything into an internal representation. Even if you just read and then write the data, the schema for the output is different.
    >  Spark schema reordering - Spark reorders the elements in its schema when writing them to disk so that the elements being partitioned on are the last elements. For an example, see Writing Partitioned Data.
    
* Perform standard extract, transform, load (ETL) processes on data

# Data Analysis
Use Spark SQL to interact with the metastore programmatically in your applications. Generate reports by using queries against loaded data.
* Use metastore tables as an input source or an output sink for Spark applications
* Understand the fundamentals of querying datasets in Spark
* Filter data using Spark
* Write queries that calculate aggregate statistics
* Join disparate datasets using Spark
* Produce ranked or sorted data

# Configuration
This is a practical exam and the candidate should be familiar with all aspects of generating a result, not just writing code.

Supply command-line options to change your application configuration, such as increasing available memory
