<!--
    Licensed to the Apache Software Foundation (ASF) under one or more 
    contributor license agreements.  See the NOTICE file distributed with
    this work for additional information regarding copyright ownership. 
    The ASF licenses this file to you under the Apache License, Version 2.0
    (the "License"); you may not use this file except in compliance with 
    the License.  You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0
    
    Unless required by applicable law or agreed to in writing, software 
    distributed under the License is distributed on an "AS IS" BASIS, 
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and 
    limitations under the License.
-->

# Carbon as Spark's datasource guide

Carbon fileformat can be integrated to Spark using datasource to read and write data without using CarbonSession.


# Create Table with DDL

Carbon table can be created with spark's datasource DDL syntax as follows.

```
 CREATE [TEMPORARY] TABLE [IF NOT EXISTS] [db_name.]table_name
     [(col_name1 col_type1 [COMMENT col_comment1], ...)]
     USING carbon
     [OPTIONS (key1=val1, key2=val2, ...)]
     [PARTITIONED BY (col_name1, col_name2, ...)]
     [CLUSTERED BY (col_name3, col_name4, ...) INTO num_buckets BUCKETS]
     [LOCATION path]
     [COMMENT table_comment]
     [TBLPROPERTIES (key1=val1, key2=val2, ...)]
     [AS select_statement]
``` 

## Supported OPTIONS

| Property | Default Value | Description |
|-----------|--------------|------------|
| table_blocksize | 1024 | Size of blocks to write onto hdfs |
| table_blocklet_size | 64 | Size of blocklet to write |
| local_dictionary_threshold | 10000 | Cardinality upto which the local dictionary can be generated  |
| local_dictionary_enable | false | Enable local dictionary generation  |
| sort_columns | all dimensions are sorted | comma separated string columns which to include in sort and its order of sort |
| sort_scope | local_sort | Sort scope of the load.Options include no sort, local sort ,batch sort and global sort |
| long_string_columns | null | comma separated string columns which are more than 32k length |

## Example 

```
 CREATE TABLE CARBON_TABLE (NAME  STRING) USING CARBON OPTIONS(‘table_block_size’=’256’)
```

Note: User can only apply the features of what spark datasource like parquet supports. It cannot support the features of carbon session like IUD, compaction etc. 

# Using DataFrame

Carbon format can be used in dataframe also using the following way.

Write carbon using dataframe 
```
df.write.format("carbon").save(path)
```

Read carbon using dataframe
```
val df = spark.read.format("carbon").load(path)
```

## Example

```
import org.apache.spark.sql.SparkSession

val spark = SparkSession
  .builder()
  .appName("Spark SQL basic example")
  .config("spark.some.config.option", "some-value")
  .getOrCreate()

// For implicit conversions like converting RDDs to DataFrames
import spark.implicits._
val df = spark.sparkContext.parallelize(1 to 10 * 10 * 1000)
     .map(x => (r.nextInt(100000), "name" + x % 8, "city" + x % 50, BigDecimal.apply(x % 60)))
      .toDF("ID", "name", "city", "age")
      
// Write to carbon format      
df.write.format("carbon").save("/user/person_table")

// Read carbon using dataframe
val dfread = spark.read.format("carbon").load("/user/person_table")
dfread.show()
```

Reference : [list of carbon properties](./configuration-parameters.md)

