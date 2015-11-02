Welcome to the SQL Masterclass

Lab 3:

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

Lab 2:
    
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