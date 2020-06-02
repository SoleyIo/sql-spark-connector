<p align="center">
  <img src="sql-spark-connector-icon.svg" alt="Apache Spark Connector for SQL Server and Azure SQL" width="150"/>
</p>

# Apache Spark Connector for SQL Server and Azure SQL
This library contains the source code for the Apache Spark Connector for SQL Server and Azure SQL.

[Apache Spark](https://spark.apache.org/) is a unified analytics engine for large-scale data processing.

## About This Release

This is a V1 release of the Apache Spark Connector for SQL Server and Azure SQL. It is a high-performance connector that enables you transfer data from Spark to SQLServer.

For main changes from previous releases and known issues please refer to [CHANGELIST](docs/CHANGELIST.md) 

## Supported Features
* Support for all Spark bindings (Scala, Python, R) 
* Basic authentication and Active Directory (AD) Key Tab support 
* Reordered DataFrame write support
* Support for write to SQLServer Single instance and Data Pool in SS19 Big Data Cluster
* Reliable connector support for Sql Server Single Instance
  

| Component | Versions Supported |
| --------- | ------------------ |
| Apache Spark | 2.4.5 or later |
| Scala | 2.11 or later (Java 8+) |
| Microsoft JDBC Driver for SQL Server | 6.2 or later |
| Microsoft SQL Server | SQL Server 2008 or later |
| Azure SQL Databases | Supported |

### Supported Options
sql spark connector supports options as defined [SQL DataSource JDBC](https://spark.apache.org/docs/latest/sql-data-sources-jdbc.html)

In addition following options are supported
| Option | Default | Description |
| --------- | ------------------ | ------------------------------------------ |
| reliabilityLevel | "BEST_EFFORT" | "BEST_EFFORT" or "NO_DUPLICATES". "NO_DUPLICATES" implements an reliable insert in executor restart scenarios |
| dataPoolDataSource | none | none implies value is not set and connector should write to Sql Server Single Instance. Set this value to data source name to write a Data Pool Table in Big Data Cluster|
| isolationLevel | "READ_COMMITTED" | Specify the isolation level |
| tableLock | "false" | Implements an insert with TABLOCK option to improve write performance |

Other [Bulk api options](https://docs.microsoft.com/en-us/sql/connect/jdbc/using-bulk-copy-with-the-jdbc-driver?view=sql-server-2017#sqlserverbulkcopyoptions) can be set as options on the dataframe and will be passed to bulkcopy apis on write

## Performance comparison
Apache Spark Connector for SQL Server and Azure SQL is up to 15x faster than generic JDBC connector for writing to SQL Server. Note performance characteristics vary on type, volume of data,  options used and may show run to run variations. The following performance results are the time taken to overwrite a sql table with 143.9M rows in a spark dataframe. The spark dataframe is constructed by reading store_sales HDFS table generated using [spark TPCDS Benchmark](https://github.com/databricks/spark-sql-perf). Time to read store_sales to dataframe is excluded. The results are averaged over 3 runs.

| Connector Type | Options | Description |  Time to write |
| --------- | ------------------ | -------------------------------------| ---------- |
| JDBCConnector | Default | Generic JDBC connector with default options |  1385s |
| sql-spark-connector | BEST_EFFORT | Best effort sql-spark-connector  with default options |580s |
| sql-spark-connector | NO_DUPLICATES | Reliable sql-spark-connector | 709s |
| sql-spark-connector | BEST_EFFORT + tabLock=true | Best effort sql-spark-connector with table lock enabled | 72s |
| sql-spark-connector | NO_DUPLICATES + tabLock=true| Reliable sql-spark-connector with table lock enabled| 198s |

Config
- Spark config : num_executors = 20, executor_memory = '1664m', executor_cores = 2
- Data Gen config : scale_factor=50, partitioned_tables=true
- Data file Store_sales with nr of rows 143,997,590

Environment
- [SQL Server Big Data Cluster](https://docs.microsoft.com/en-us/sql/big-data-cluster/release-notes-big-data-cluster?view=sql-server-ver15) CU5
- Master + 6 nodes
- Each node gen 5 server, 512GB Ram, 4TB NVM per node, NIC 10GB

## Get Started
The Apache Spark Connector for SQL Server and Azure SQL is based on the Spark DataSourceV1 API and SQL Server Bulk API and uses the same interface as the built-in JDBC Spark-SQL connector. This allows you to easily integrate the connector and migrate your existing Spark jobs by simply updating the format parameter with `com.microsoft.sqlserver.jdbc.spark`.

### Write to SQL Table
```python
server_name = "jdbc:sqlserver://{SERVER_ADDR}"
database_name = "database_name"
url = servername + ";" + "database_name=" + dbname + ";"

table_name = "table_name"
username = "username"
password = "password123!#" # Please specify password here

try:
  df.write \
    .format("com.microsoft.sqlserver.jdbc.spark") \
    .mode("overwrite") \
    .option("url", url) \
    .option("dbtable", table_name) \
    .option("user", username) \
    .option("password", password) \
    .save()
except ValueError as error :
    print("Connector write failed", error)
```

### Append to SQL Table
```python
try:
  df.write \
    .format("com.microsoft.sqlserver.jdbc.spark") \
    .mode("append") \
    .option("url", url) \
    .option("dbtable", table_name) \
    .option("user", username) \
    .option("password", password) \
    .save()
except ValueError as error :
    print("Connector write failed", error)
```

### Read from SQL Table
```python
jdbcDF = spark.read \
        .format("com.microsoft.sqlserver.jdbc.spark") \
        .option("url", url) \
        .option("dbtable", table_name) \
        .option("user", username) \
        .option("password", password).load()
```

# Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.opensource.microsoft.com.

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

## Roadmap
Visit the Connector project in the **Projects** tab to see needed / planned items. Feel free to make an issue and start contributing!
