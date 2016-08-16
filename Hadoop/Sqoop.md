
# Intro

Apache Sqoop is a tool for transferring bulk data between Hadoop and structured datastores.

Sqoop can also transfer data from database into HBase.

# Connectors

Sqoop supports JDBC for any DB that speaks JDBC.

Sqoop has native connectors for MySql, PostgreSQL, Oracle and Netezza for bulk transfers.

# MySql Example 

```bash
> sqoop import --connect jdbc:mysql://host/db --table tablename -m 1 
```
Sqoop runs a MR job and reads the table. `-m` specifies the number of map tasks.

Sqoop generates csv files by default.

A few points regarding mysql driver:
* Download the driver: `wget http://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.39.tar.gz`
* Extract: `tar -zxvf mysql-connector-java-5.1.39.tar.gz`
* Copy the jar file: `sudo cp mysql-connector-java-5.1.39/mysql-connector-java-5.1.39-bin.jar /var/lib/sqoop/`

```bash
sudo -u username sqoop import -m 1 --table tablename --connect \
"jdbc:mysql://mydbinstance.host.dns:3306/dbname?user=****&password=*********"
```

If the parameter of `-m` is bigger than 1, Sqoop uses primary key for split.
If there is no pk then `--split-by` should be used to specify a column for split.

Note: the version I tested (i.e. Sqoop 1.4.6-cdh5.7.0) if the column specified by `--split-by` contains `null` values those rows will be ignored. This issue has been reported [here](https://issues.apache.org/jira/browse/SQOOP-2676).

**import as Parquet**:

```bash
sudo -u username sqoop import -m 1 --as-parquetfile --target-dir dirname --table tablename --connect \
"jdbc:mysql://mydbinstance.host.dns:3306/dbname?user=****&password=*********"
```

## Direct mode 

```bash
sudo -u username sqoop import -m 1 --target-dir dirname --table tablename --connect \
"jdbc:mysql://mydbinstance.host.dns:3306/dbname?user=****&password=*********" --direct -- -u **** -p"*********"
```
Use `--direct` for using specific tools that databases provide to extract data, like `mysqldump` in case of MySql.

If the argument `--` is given on the command-line, then subsequent arguments are sent directly to the underlying tool.

Direct modes are usuaslly much for faster than JDBC.

## Import to Hive

In a few ways it can be done.

One way:

1. Import to hdfs the usual way:

   ```bash 
   sqoop import -m 1 --table tablename --connect "jdbc:mysql://host:3306/dbname?user=****&password=*********"
   ```
2. Create table in Hive:

   ```bash
   sqoop create-hive-table --table tablename \
   --connect "jdbc:mysql://host:3306/dbname?user=****&password=*********" --fields-terminated-by ','
   ```
3. Load data into Hive:

   ```bash
   %> hive
   hive> LOAD DATA INPATH "tablename" INTO TABLE tablename
   ```

Another way, using sqoop for data loading:

```bash
sqoop import -m 1 --hive-import --table tablename \
--connect "jdbc:mysql://host:3306/dbname?user=****&password=*********"
```

## Export

```bash
%> mysql
mysql> CREATE TABLE tablename_in_db (col1 INTEGER, col2 varchar(200));

%> sqoop export -m 1 --table tablename_in_db \
--export-dir /user/hive/warehouse/data_in_hive \
--input-fields-terminated-by '\0001' \
--connect "jdbc:mysql://host:3306/dbname?user=****&password=*********"
```

Direct strategy still can be used.
For example in case of MySql, Sqoop will use `mysqlimport`.


