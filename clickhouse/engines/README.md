# TABLE ENGINE  
## MergeTree Family  
#### MergeTree  
The MergeTree engine and other engines of this family(*MergeTree) are the most robust ClickHouse table engines.  
Engines in the MergeTree family are designed for inserting a very large amount of data into a table. The data is quickly written to the table part by part, then rules are applied for merging the parts in the background. This method is much more efficient than continually rewriting the data in storage during insert.  
Main features:  
- Stores data sorted by primary key.  
- This allows you to create a small sparse index that helps find data faster.  
- Partitions can be used if the partitioning key is specified.  
- ClickHouse supports certain operations with partitions that are more effective than general operations on the same data with the same result. ClickHouse also automatically cuts off the partition data where the partitioning key is specified in the query. This also improves query performance.  
- Data replication support.  
- The family of ReplicatedMergeTree tables provides data replication. For more information, see Data replication.  
- Data sampling support.  
- If necessary, you can set the data sampling method in the table.  
#### ReplacingMergeTree
The engine differs from MergeTree in that it removes duplicate entries with the same primary key value (or more accurately, with the same sorting key value).  
Data deduplication occurs only during a merge. Merging occurs in the background at an unknown time, so you can’t plan for it. Some of the data may remain unprocessed. Although you can run an unscheduled merge using the OPTIMIZE query, don’t count on using it, because the OPTIMIZE query will read and write a large amount of data.  
Thus, ReplacingMergeTree is suitable for clearing out duplicate data in the background in order to save space, but it doesn’t guarantee the absence of duplicates.  
#### SummingMergeTree
The engine inherits from MergeTree. The difference is that when merging data parts for SummingMergeTree tables ClickHouse replaces all the rows with the same primary key (or more accurately, with the same sorting key) with one row which contains summarized values for the columns with the numeric data type. If the sorting key is composed in a way that a single key value corresponds to large number of rows, this significantly reduces storage volume and speeds up data selection.  
We recommend to use the engine together with MergeTree. Store complete data in MergeTree table, and use SummingMergeTree for aggregated data storing, for example, when preparing reports. Such an approach will prevent you from losing valuable data due to an incorrectly composed primary key.  
#### AggregatingMergeTree
The engine inherits from MergeTree, altering the logic for data parts merging. ClickHouse replaces all rows with the same primary key (or more accurately, with the same sorting key) with a single row (within a one data part) that stores a combination of states of aggregate functions.  
You can use AggregatingMergeTree tables for incremental data aggregation, including for aggregated materialized views.  
#### CollapsingMergeTree
The engine inherits from MergeTree and adds the logic of rows collapsing to data parts merge algorithm.  
CollapsingMergeTree asynchronously deletes (collapses) pairs of rows if all of the fields in a sorting key (ORDER BY) are equivalent excepting the particular field Sign which can have 1 and -1 values. Rows without a pair are kept. For more details see the Collapsing section of the document.  
The engine may significantly reduce the volume of storage and increase the efficiency of SELECT query as a consequence.  
#### VersionedCollapsingMergeTree
This engine allows quick writing of object states that are continually changing and deletes old object states in the background.  This significantly reduces the volume of storage.  
The engine inherits from MergeTree and adds the logic for collapsing rows to the algorithm for merging data parts.  VersionedCollapsingMergeTree serves the same purpose as CollapsingMergeTree but uses a different collapsing algorithm that allows inserting the data in any order with multiple threads. In particular, the Version column helps to collapse the rows properly even if they are inserted in the wrong order. In contrast, CollapsingMergeTree allows only strictly consecutive insertion.  
#### GraphiteMergeTree  
This engine is designed for thinning and aggregating/averaging (rollup) Graphite data. It may be helpful to developers who want to use ClickHouse as a data store for Graphite.  
#### Data Replication  
Replication is only supported for tables in the MergeTree family:  
- ReplicatedMergeTree  
- ReplicatedSummingMergeTree  
- ReplicatedReplacingMergeTree  
- ReplicatedAggregatingMergeTree  
- ReplicatedCollapsingMergeTree  
- ReplicatedVersionedCollapsingMergeTree  
- ReplicatedGraphiteMergeTree  

Replication works at the level of an individual table, not the entire server. A server can store both replicated and non-replicated tables at the same time.  
Replication does not depend on sharding. Each shard has its own independent replication.  
ClickHouse uses Apache ZooKeeper for storing replicas meta information. Use ZooKeeper version 3.4.5 or newer.  
## Log  
Engines:  
- Store data on a disk.  
- Append data to the end of file when writing.  
- Support locks for concurrent data access.  
- During INSERT queries, the table is locked, and other queries for reading and writing data both wait for the table to unlock. If there are no data writing queries, any number of data reading queries can be performed concurrently.  
- Do not support mutation operations.  
- Do not support indexes.  
- This means that SELECT queries for ranges of data are not efficient.  
- Do not write data atomically.  
- You can get a table with corrupted data if something breaks the write operation, for example, abnormal server shutdown.  
#### TinyLog  
This table engine is typically used with the write-once method: write data one time, then read it as many times as necessary. For example, you can use TinyLog-type tables for intermediary data that is processed in small batches. Note that storing data in a large number of small tables is inefficient.  
Queries are executed in a single stream. In other words, this engine is intended for relatively small tables (up to about 1,000,000 rows). It makes sense to use this table engine if you have many small tables, since it’s simpler than the Log engine (fewer files need to be opened).  
#### StripeLog  
Use this engine in scenarios when you need to write many tables with a small amount of data (less than 1 million rows).  
#### Log  
Log differs from TinyLog in that a small file of “marks” resides with the column files. These marks are written on every data block and contain offsets that indicate where to start reading the file in order to skip the specified number of rows. This makes it possible to read table data in multiple threads.  
For concurrent data access, the read operations can be performed simultaneously, while write operations block reads and each other.  
The Log engine does not support indexes. Similarly, if writing to a table failed, the table is broken, and reading from it returns an error. The Log engine is appropriate for temporary data, write-once tables, and for testing or demonstration purposes.  
## Integration Engines  
#### Kafka  
This engine works with Apache Kafka.  
Kafka lets you:  
- Publish or subscribe to data flows.  
- Organize fault-tolerant storage.  
- Process streams as they become available.  
#### MySQL  
The MySQL engine allows you to perform SELECT queries on data that is stored on a remote MySQL server.  
#### ODBC  
Allows ClickHouse to connect to external databases via ODBC.  
#### JDBC  
Allows ClickHouse to connect to external databases via JDBC.  
#### HDFS  
This engine provides integration with Apache Hadoop ecosystem by allowing to manage data on HDFSvia ClickHouse. This engine is similar to the File and URL engines, but provides Hadoop-specific features.  
## Special Engines  
#### Distributed  
Tables with Distributed engine do not store any data by themself, but allow distributed query processing on multiple servers.  
Reading is automatically parallelized. During a read, the table indexes on remote servers are used, if there are any.  
#### MaterializedView  
Used for implementing materialized views (for more information, see CREATE TABLE). For storing data, it uses a different engine that was specified when creating the view. When reading from a table, it just uses this engine.  
#### Dictionary  
The Dictionary engine displays the dictionary data as a ClickHouse table.  
#### Merge  
The Merge engine (not to be confused with MergeTree) does not store data itself, but allows reading from any number of other tables simultaneously.  
Reading is automatically parallelized. Writing to a table is not supported. When reading, the indexes of tables that are actually being read are used, if they exist.  
The Merge engine accepts parameters: the database name and a regular expression for tables.  
#### File  
The File table engine keeps the data in a file in one of the supported file  
#### Null  
When writing to a Null table, data is ignored. When reading from a Null table, the response is empty.  
#### Set  
A data set that is always in RAM. It is intended for use on the right side of the IN operator.  
#### Join  
Prepared data structure for using in JOIN operations.  
#### URL  
Manages data on a remote HTTP/HTTPS server. This engine is similar to the File engine.  
#### View  
Used for implementing views (for more information, see the CREATE VIEW query). It does not store data, but only stores the specified SELECT query. When reading from a table, it runs this query (and deletes all unnecessary columns from the query).  
#### Memory  
The Memory engine stores data in RAM, in uncompressed form. Data is stored in exactly the same form as it is received when read. In other words, reading from this table is completely free.  
#### Buffer  
Buffers the data to write in RAM, periodically flushing it to another table. During the read operation, data is read from the buffer and the other table simultaneously.  
# DATABASE ENGINE  
Database engines allow you to work with tables.  
By default, ClickHouse uses its native database engine, which provides configurable table engines and an SQL dialect.  
You can also use the following database engines:  
#### Lazy  
Keeps tables in RAM only expiration_time_in_seconds seconds after last access. Can be used only with *Log tables.  
It’s optimized for storing many small *Log tables, for which there is a long time interval between accesses.  
#### MySQL  
Allows to connect to databases on a remote MySQL server and perform INSERT and SELECT queries to exchange data between ClickHouse and MySQL.  
The MySQL database engine translate queries to the MySQL server so you can perform operations such as SHOW TABLES or SHOW CREATE TABLE.  
You cannot perform the following queries:  
- RENAME
- CREATE TABLE
- ALTER

