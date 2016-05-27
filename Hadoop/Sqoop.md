
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

A few points regarding mysql driver:
* Download the driver: `wget http://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.39.tar.gz`
* Extract: `tar -zxvf mysql-connector-java-5.1.39.tar.gz`
* Copy the jar file: `sudo cp mysql-connector-java-5.1.39/mysql-connector-java-5.1.39-bin.jar /var/lib/sqoop/`

```bash
sudo -H -u auser bash -c 'sqoop import -m 1 --table tablename --connect \
"jdbc:mysql://mydbinstance.host.dns:3306/dbname?user=****&password=*********"'
```

