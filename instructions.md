Welcome to the SQL Masterclass



###Lab 2:
    
- Use File view to navigate to e.g. `/masterclass/lab2/customers`
- Use Hive view to query a table
```SQL
  SELECT * 
  FROM customers 
  WHERE length(last_name) > 7
  ORDER BY last_name;
```
- Save the query

-----
Question: How do we create the customers table from that file? (Thanks)
Answer:
```SQL
CREATE EXTERNAL TABLE customers(
  first_name STRING,
  last_name STRING,
  address ARRAY<STRING>,
  email STRING,
  phone STRING
)
LOCATION "/masterclass/lab2/customers";
```
----

Q: is this the 'default' delimiter configuration (SOH,STX)? I guess there are commands to change this setup for text delimiters that are not acsii 1 & 2?


- Use Hive view to create a table and fill it
  
```SQL
  CREATE EXTERNAL TABLE IF NOT EXISTS cust2 (
    last_name STRING,
    first_name STRING,
    house STRING,
    street STRING,
    post_code STRING
  )
  LOCATION "/tmp/cust2";



  INSERT into cust2
  SELECT last_name, first_name, address[0], address[1], address[3] 
  FROM customers 
  ORDER BY last_name;
```

- Refresh the Database Explorer and examine `cust2`.

- Look at the visual explanation of the last SQL statement

- Again, use File View to navigate to `/tmp/cust2`


###Lab 3:

- Use File View to navigate to `/masterclass/lab3/tweets`
- Use Hive View to create Schema (with data)

```SQL
  CREATE EXTERNAL TABLE IF NOT EXISTS tweets_text_partition(
    tweet_id bigint,
    created_unixtime bigint,
    created_time string,
    displayname string,
    msg string,
    fulltext string
  )
  ROW FORMAT DELIMITED FIELDS TERMINATED BY "|"
  LOCATION "/masterclass/lab3/tweets";
```

  Use Database Explorer to look at schema and data 

- Prepare the data as ORC

```SQL
  CREATE EXTERNAL TABLE IF NOT EXISTS tweets_orc_msg_hashtags(
      tweet_id bigint,
      created_unixtime string,
      displayname string,
      msg string,
      hashtag string
  )
  STORED AS orc;
```

  __Note__: This is implemented via a SerDe. You could add: ROW FORMAT SERDE "org.apache.hadoop.hive.ql.io.orc.OrcSerde"

```SQL
  INSERT OVERWRITE TABLE tweets_orc_msg_hashtags
  SELECT
    tweet_id,
    from_unixtime(floor(created_unixtime/1000)),
    displayname,
    msg,
    get_json_object(fulltext,'$.entities.hashtags[0].text')
    FROM  tweets_text_partition;
```

Question: Is this complete? Thanks - that is ok now.

- Create a simple query

```SQL
  SELECT hashtag, count(*) as cnt
  FROM tweets_orc_msg_hashtags
  where hashtag is not null 
  group by hashtag
  having cnt>10
  LIMIT 10;
```



###Lab 4: Building a table on top of json

- Download the file `sample_twitter_data.txt` (from HDFS via File View) and examine the structure
```SQL
  CREATE TABLE IF NOT EXISTS twitter_json (
    geolocation STRUCT<lat: DOUBLE, long: DOUBLE>,
    tweetmessage STRING,
    createddate STRING,
    `user` STRUCT<screenname: STRING,
                  name: STRING,
                  id: INT,
                  geoenabled: BOOLEAN,
                  userlocation: STRING>
  ) 
  ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe';
  
  
  LOAD DATA INPATH '/masterclass/lab4/twitter/sample_twitter_data.txt' 
  INTO TABLE twitter_json;
```
- Query the file
```SQL
  SELECT * 
  FROM twitter_json;
  
  SELECT createddate, `user`.screenname 
  FROM twitter_json WHERE `user`.name LIKE 'Sarah%';


ADD JAR /usr/lib/hive/json-serde-1.1.9.9-Hive13-jar-with-dependencies.jar;
```

###Lab 5: Use HBase table from Hive

__Goal__: Understand external data sources using HBase as an example


##### Query HBase using Phoenix and Hive

- Use the terminal (ssh) access to query from Phoenix `phoenix-sqlline localhost:2181:/hbase-unsecure`:
```SQL
  CREATE VIEW "employees" ( "pk" VARCHAR PRIMARY KEY, "f"."birth_date" VARCHAR, "f"."first_name" VARCHAR, "f"."last_name" VARCHAR, "f"."gender" VARCHAR, "f"."hire_date" VARCHAR );

  SELECT * from "employees" limit 10;
```
- Link with Hive
```SQL
  CREATE EXTERNAL TABLE employees_hbase(
    key BIGINT, 
    birth_date STRING, 
    first_name STRING, 
    last_name STRING, 
    gender STRING, 
    hire_date STRING
  )
  STORED BY "org.apache.hadoop.hive.hbase.HBaseStorageHandler"
  WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,f:birth_date,f:first_name,f:last_name,f:gender,f:hire_date")
  TBLPROPERTIES("hbase.table.name" = "employees");
```
- Query new external table in Hive

`  SELECT * from employees_hbase limit 10;`



##### Create HBase table in Hive

- Use Hive view to create a table in HBase
```SQL
  CREATE TABLE tweets_hbase(
    tweet_id BIGINT, 
    created_unixtime BIGINT, 
    created_time STRING, 
    displayname STRING, 
    msg STRING, 
    fulltext STRING  
  )
  STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
  WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,f:c1,f:c2,f:c3,f:c4,f:c5")
  TBLPROPERTIES ("hbase.table.name" = "tweets");
```
- Goto Ambari Dashboard - HBase - Quick Links - HBase Master UI to check whether tweets table exists
- Insert data into HBase table
```SQL
  INSERT INTO tweets_hbase
    SELECT * from tweets_text_partition;
```
- Query from Hive

  `SELECT * from tweets_hbase;`
  
  
  Lab 6: Read a query plan (EXPLAIN)

- Sample query (select "employees" database in Hive View):

SELECT e.first_name, e.last_name, e.hire_date, d.dept_name, x.from_date, x.to_date
FROM departments d, employees e, dept_emp x
WHERE d.dept_no = x.dept_no and e.emp_no = x.emp_no
ORDER BY d.dept_name, e.last_name, e.first_name, x.from_date;


- Calculate Statistics

ANALYZE TABLE departments COMPUTE STATISTICS for COLUMNS;
ANALYZE TABLE dept_emp COMPUTE STATISTICS for COLUMNS;
ANALYZE TABLE employees COMPUTE STATISTICS for COLUMNS;

ANALYZE TABLE departments COMPUTE STATISTICS;
ANALYZE TABLE dept_emp COMPUTE STATISTICS;
ANALYZE TABLE employees COMPUTE STATISTICS;