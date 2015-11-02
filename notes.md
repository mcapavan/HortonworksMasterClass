www.community.hortonworks.com

Pheonix: High Concurrency. You can run large number of user submitting queries with low latency. Easy to use compared to Hbase UI.

Hive drawbacks: Cannot have many concurrent users. It may be okay for tens of user but not hundreds of concurrent users.
SparkSQL also has high concurrency (many users) issues.
Pheonix has limitations on deep sql analytics.

Hive is tested with 100 concurrent users and not able to go beyond 100 users. SparkSQL assign memory for each user request so it cannot go beyond the in-memory capacity.

Hive on MR will be an option only if you have custom code and the optimization is not 100% accurate on Tez. Otherwise Tez is highly recommended to use on Hive

Hive and HiveServer 2

Hive: has limitations on security and concurrency.

HiveServer2: good for concurrent users

Use Beeline - Don’t use Hive CLI

Stinger.Next: aim to meet SQL Server 2011 analytics

Hue Challenges: Not open project. HDP couldn't expand.  It is Cloudera product. Hive View and Hue are pretty much same. 

https://hortonworks-gallery.github.io/index.html?sort=asc&filter=featured

HDP is in process to remove hive metadata server (mySQL) and keep metadata in Hbase

Hive File formats: Row Oriented: Sequence Files and Avro
Column Oriented file formats: Parquet and ORC Files

Interpretability and split-ability

All these file formats can be used by Pig and Spark.

If there is ORC format table in Hive, you cannot load data from Sqoop.

Gotchas with Sqoop can be found below.

http://getindata.com/blog/post/surprising-sqoop-to-hive-gotchas/

Sqoop will NOT be used Knox. Knox is made for user interface. But data load is not goes via Knox. Hue doesn't use Knox.

Add your SerDe jars to HDFS and attach to Hive

UDFs are per Database


