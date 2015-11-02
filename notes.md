www.community.hortonworks.com

Phoenix: High Concurrency. You can run large number of user submitting queries with low latency. Easy to use compared to Hbase UI.

Hive drawbacks: Cannot have many concurrent users. It may be okay for tens of user but not hundreds of concurrent users.
SparkSQL also has high concurrency (many users) issues.
Phoenix has limitations on deep sql analytics.

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

Interopretability and split-ability

All these file formats can be used by Pig and Spark.

If there is ORC format table in Hive, you cannot load data from Sqoop.

Gotchas with Sqoop can be found below.

http://getindata.com/blog/post/surprising-sqoop-to-hive-gotchas/

Sqoop will NOT be used Knox. Knox is made for user interface. But data load is not goes via Knox. Hue doesn't use Knox.

Add your SerDe jars to HDFS and attach to Hive

UDFs are per Database

hive.server2.enable.doAs = false

Stinger initiate - Vector Query engine

We can have multiple Hive Server 2 instances - one for ETL, one for Business users, etc...

Calcite (Open-sourced framework) - for Cost-Based Optimization (CBO)

If you check "Explain" of Hive query and you see "Plan not optimized by CBO due to missing statistics. Please check log for more details.", you would consider running "COMPUTE STATISTICS for COLUMNS" and "COMPUTE STATISTICS" for those tables.

Example:
Hive SQL:
```SQL
SELECT e.first_name, e.last_name, e.hire_date, d.dept_name, x.from_date, x.to_date
FROM departments d, employees e, dept_emp x
WHERE d.dept_no = x.dept_no and e.emp_no = x.emp_no
ORDER BY d.dept_name, e.last_name, e.first_name, x.from_date;
```

### Calculate Statistics
```SQL
ANALYZE TABLE departments COMPUTE STATISTICS for COLUMNS;
ANALYZE TABLE dept_emp COMPUTE STATISTICS for COLUMNS;
ANALYZE TABLE employees COMPUTE STATISTICS for COLUMNS;

ANALYZE TABLE departments COMPUTE STATISTICS;
ANALYZE TABLE dept_emp COMPUTE STATISTICS;
ANALYZE TABLE employees COMPUTE STATISTICS;
```




