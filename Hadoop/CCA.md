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
* Ingest real-time and near-real-time streaming data into HDFS
* Process streaming data as it is loaded onto the cluster
* Load data into and out of HDFS using the Hadoop File System commands

# Transform, Stage, and Store
Convert a set of data values in a given format stored in HDFS into new data values or a new data format and write them into HDFS.
* Load RDD data from HDFS for use in Spark applications
* Write the results from an RDD back into HDFS using Spark
* Read and write files in a variety of file formats
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
