KSQL-Developed by confluent
===========================
KSQL- Streaming SQL for Apache Kafka

Databases are used for doing on-demand lookups and modifications to stored data. KSQL doesn’t do lookups (yet), what it does is continuous transformations— that is, stream processing. For example, imagine that I have a stream of clicks from users and a table of account information about those users being continuously updated. KSQL allows me to model this stream of clicks, and table of users, and join the two together.

So what KSQL runs are continuous queries — transformations that run continuously as new data passes through them — on streams of data in Kafka topics

Core Abstractions in KSQL:
1.Stream--> A stream is an unbounded sequence of structured data 
Ex:
```
create stream userdata(userid VARCHAR,regionid VARCHAR,gender VARCHAR,registertime BIGINT) with (VALUE_FORMAT='AVRO',KAFKA_TOPIC='users');
```
2.table-->A table is a view of a STREAM or another TABLE and represents a collection of evolving facts. 
Ex:
```
create table countrytable(countrycode VARCHAR PRIMARY KEY,countryname VARCHAR) with (KAFKA_TOPIC='country_csv',VALUE_FORMAT='delimited');
```

----------------------------------------------------------------------------------------------------------------------
Start KSQL :
 start ksql shell using -- docker exec -it 3fffd9c2b0ad ksql http://ksqldb-server:8088

```
[user@node ~]$ docker exec -it 3fffd9c2b0ad ksql http://ksqldb-server:8088
OpenJDK 64-Bit Server VM warning: Option UseConcMarkSweepGC was deprecated in version 9.0 and will likely be removed in a future release.

                  ===========================================
                  =       _              _ ____  ____       =
                  =      | | _____  __ _| |  _ \| __ )      =
                  =      | |/ / __|/ _` | | | | |  _ \      =
                  =      |   <\__ \ (_| | | |_| | |_) |     =
                  =      |_|\_\___/\__, |_|____/|____/      =
                  =                   |_|                   =
                  =  Event Streaming Database purpose-built =
                  =        for stream processing apps       =
                  ===========================================

Copyright 2017-2020 Confluent Inc.

CLI v6.1.0, Server v6.1.0 located at http://ksqldb-server:8088
Server Status: RUNNING

Having trouble? Type 'help' (case-insensitive) for a rundown of how things work!

ksql>
```

Create a topic and start producing on the topics:

```kafka node
[appuser@broker ~]$ kafka-topics --zookeeper zookeeper:2181 --create --topic streaming-data --partitions 1 --replication-factor 1
Created topic streaming-data.
[appuser@broker ~]$ kafka-console-producer --broker-list localhost:9092 --topic streaming-data
>Anudeep
>Abcd
>Efgh
>klmn
>my name is anudeep
>i am working at Modak
>
```
```ksql node
ksql> list topics;

 Kafka Topic                 | Partitions | Partition Replicas
---------------------------------------------------------------
 default_ksql_processing_log | 1          | 1
 docker-connect-configs      | 1          | 1
 docker-connect-offsets      | 25         | 1
 docker-connect-status       | 5          | 1
 streaming-data              | 1          | 1
---------------------------------------------------------------

ksql> show topics;#same as list topics

  Kafka Topic                 | Partitions | Partition Replicas
---------------------------------------------------------------
 default_ksql_processing_log | 1          | 1
 docker-connect-configs      | 1          | 1
 docker-connect-offsets      | 25         | 1
 docker-connect-status       | 5          | 1
 streaming-data              | 1          | 1
---------------------------------------------------------------

ksql> print 'streaming-data';
Key format: ¯\_(ツ)_/¯ - no data processed
Value format: KAFKA_STRING
rowtime: 2021/03/06 08:08:28.737 Z, key: <null>, value: Anudeep
rowtime: 2021/03/06 08:08:39.984 Z, key: <null>, value: Abcd
rowtime: 2021/03/06 08:08:44.931 Z, key: <null>, value: Efgh
rowtime: 2021/03/06 08:08:46.985 Z, key: <null>, value: klmn
rowtime: 2021/03/06 08:09:09.619 Z, key: <null>, value: my name is anudeep
rowtime: 2021/03/06 08:09:15.572 Z, key: <null>, value: i am working at Modak
^CTopic printing ceased

ksql> print 'streaming-data' from beginning;
Key format: ¯\_(ツ)_/¯ - no data processed
Value format: KAFKA_STRING
rowtime: 2021/03/06 08:08:28.737 Z, key: <null>, value: Anudeep
rowtime: 2021/03/06 08:08:39.984 Z, key: <null>, value: Abcd
rowtime: 2021/03/06 08:08:44.931 Z, key: <null>, value: Efgh
rowtime: 2021/03/06 08:08:46.985 Z, key: <null>, value: klmn
rowtime: 2021/03/06 08:09:09.619 Z, key: <null>, value: my name is anudeep
rowtime: 2021/03/06 08:09:15.572 Z, key: <null>, value: i am working at Modak

ksql> print 'streaming-data' from beginning limit 2;
Key format: ¯\_(ツ)_/¯ - no data processed
Value format: KAFKA_STRING
rowtime: 2021/03/06 08:08:28.737 Z, key: <null>, value: Anudeep
rowtime: 2021/03/06 08:08:39.984 Z, key: <null>, value: Abcd
Topic printing ceased
ksql>
```


Streams:
========
A sequence of events from start of time.
Constanty arrived to a particular topic in a time orderd manner.
Can be processed independently.Ex:clickstream ,twitter etc..
There are two types of queries in streams:
->Push Queries--constantly output queries,EMIT CHANGES indicates that its a push query.The push query doesnt stop until the query is terminated or exceed limit contition
->pull queries --current state of the system.Returns results and terminates.Supports only on aggregate tables.

```
ksql> list streams;

 Stream Name         | Kafka Topic                 | Key Format | Value Format | Windowed
------------------------------------------------------------------------------------------
 KSQL_PROCESSING_LOG | default_ksql_processing_log | KAFKA      | JSON         | false
------------------------------------------------------------------------------------------

ksql> show streams;#same as list streams

 Stream Name         | Kafka Topic                 | Key Format | Value Format | Windowed
------------------------------------------------------------------------------------------
 KSQL_PROCESSING_LOG | default_ksql_processing_log | KAFKA      | JSON         | false
------------------------------------------------------------------------------------------

#create a ksql stream on topic streaming data
ksql> create stream random_streaming(random_id INT ,random_name VARCHAR ,random_salary DOUBLE) with (KAFKA_TOPIC='streaming-data',VALUE_FORMAT='DELIMITED');

 Message
----------------
 Stream created
----------------

ksql> show streams;

 Stream Name         | Kafka Topic                 | Key Format | Value Format | Windowed
------------------------------------------------------------------------------------------
 KSQL_PROCESSING_LOG | default_ksql_processing_log | KAFKA      | JSON         | false
 RANDOM_STREAMING    | streaming-data              | KAFKA      | DELIMITED    | false
------------------------------------------------------------------------------------------

ksql> describe extended random_streaming;

Name                 : RANDOM_STREAMING
Type                 : STREAM
Timestamp field      : Not set - using <ROWTIME>
Key format           : KAFKA
Value format         : DELIMITED
Kafka topic          : streaming-data (partitions: 1, replication: 1)
Statement            : CREATE STREAM RANDOM_STREAMING (RANDOM_ID INTEGER, RANDOM_NAME STRING, RANDOM_SALARY DOUBLE) WITH (KAFKA_TOPIC='streaming-data', KEY_FORMAT='KAFKA', VALUE_FORMAT='DELIMITED');

 Field         | Type
---------------------------------
 RANDOM_ID     | INTEGER
 RANDOM_NAME   | VARCHAR(STRING)
 RANDOM_SALARY | DOUBLE
---------------------------------

Local runtime statistics
------------------------


(Statistics of the local KSQL server interaction with the Kafka topic streaming-data)

We can also insert data into a stream.same as sql syntax
ksql> drop stream if exists RANDOM_STREAMING;

 Message
------------------------------------------------------
 Source `RANDOM_STREAMING` (topic: streaming-data) was dropped.
------------------------------------------------------


#Generating a stream (datagen):
[appuser@ksql-datagen ~]$ ksql-datagen schema=./userprofile.avro format=json topic=streaming-data key=userid msgRate=5000 iterations=10 bootstrap-server=broker:9092;

```

Send events to that topic:
CSV stream:
```kafka broker
[appuser@broker ~]$ kafka-console-producer --broker-list localhost:9092 --topic streaming-data
>1,anudeep,12000
>2,naveen,14000
>3,Siva,15000
>
```

```ksql 
ksql> select random_id,random_name,random_salary from RANDOM_STREAMING EMIT CHANGES;
+------------------------------------------------+----------------------------------------+-----------------------------------------------------------+
|RANDOM_ID                                       |RANDOM_NAME                             |RANDOM_SALARY                                              |
+-------------------------------- ---------------+----------------------------------------+-----------------------------------------------------------+
|1                                               |anudeep                                 |12000.0                                                    |
|2                                               |naveen                                  |14000.0                                                    |
|3                                               |Siva                                    |15000.0                                                    |
^CQuery terminated

ksql> SET 'auto.offset.reset'='earliest';
Successfully changed local property 'auto.offset.reset' to 'earliest'. Use the UNSET command to revert your change.
ksql> select random_id,random_name,random_salary from RANDOM_STREAMING EMIT CHANGES limit 2;
+------------------------------------------------+----------------------------------------+-----------------------------------------------------------+
|RANDOM_ID                                       |RANDOM_NAME                             |RANDOM_SALARY                                              |
+-------------------------------- ---------------+----------------------------------------+-----------------------------------------------------------+
|1                                               |anudeep                                 |12000.0                                                    |
|2                                               |naveen                                  |14000.0                                                    | 
Limit Reached
Query terminated

ksql> drop stream if exists USERDATA delete topic;

 Message
------------------------------------------------------
 Source `USERDATA` (topic: user_profile) was dropped.
------------------------------------------------------

```

JSON stream:
```kafka
[appuser@broker ~]$ kafka-console-producer --broker-list localhost:9092 --topic streaming-data
>{"random_id":1,"random_name":"anudeep","random_salary":12000}
>{"random_id":1,"random_name":"anudeep"}
>{"random_id":1,"random_name":"anudeep"}
>{"random_id":1}

```

```
ksql> create stream json_streaming(random_id INT ,random_name VARCHAR ,random_salary DOUBLE) with (KAFKA_TOPIC='streaming-data',VALUE_FORMAT='JSON');

 Message
----------------
 Stream created
----------------
ksql> select * from json_streaming EMIT CHANGES;
+-----------------------------------------------------------+------------------------ ------------------+------------------------------------------------+
|RANDOM_ID                                                  |RANDOM_NAME                                |RANDOM_SALARY                                    |
+-----------------------------------------------------------+----------------- -------------------------+-------------------------------------------------+
|1                                                          |anudeep                                    |12000.0                                          |
|1                                                          |anudeep                                    |null                                             |
|1                                                          |anudeep                                    |null                                             |
|1                                                          |null                                       |null                                             |
```

Avro stream:
Avro data is generated to a topic called users from datagen connector from confluent ui.
```
ksql> create stream avro_streaming_final(userid VARCHAR ,regionid VARCHAR ,gender VARCHAR, registertime BIGINT) with (KAFKA_TOPIC='users',VALUE_FORMAT='AVRO')
>;

 Message
----------------
 Stream created
----------------
ksql> select * from avro_streaming_final emit changes;
+--------------------------+-------------------------------------------+-------------------------------------------+-------------------------------------------+
|USERID                    |REGIONID                                   |GENDER                                     |REGISTERTIME                               |
+--------------------------+-------------------------------------------+-------------------------------------------+-------------------------------------------+
|User_5                    |Region_8                                   |FEMALE                                     |1506156834746                              |
|User_9                    |Region_8                                   |FEMALE                                     |1503638277267                              |
|User_6                    |Region_7                                   |OTHER                                      |1489472503412                              |
|User_6                    |Region_7                                   |MALE                                       |1491014414080                              |
|User_2                    |Region_6                                   |OTHER                                      |1504360471046                              |
|User_9                    |Region_2                                   |OTHER                                      |1508498903790                              |
|User_5                    |Region_8                                   |MALE                                       |1511034156237                              |
^CQuery terminated

ksql> SELECT userid,regionid,gender,TIMESTAMPTOSTRING(REGISTERTIME, 'dd/MMM HH:mm') as time from  avro_streaming_final emit changes;
+--------------------------+-------------------------------------------+-------------------------------------------+-------------------------------------------+
|USERID                    |REGIONID                                   |GENDER                                     |TIME                                       |
+--------------------------+-------------------------------------------+-------------------------------------------+-------------------------------------------+
|User_5                    |Region_8                                   |FEMALE                                     |23/Sep 08:53                               |
|User_9                    |Region_8                                   |FEMALE                                     |25/Aug 05:17                               |
|User_6                    |Region_7                                   |OTHER                                      |14/Mar 06:21                               |
|User_6                    |Region_7                                   |MALE                                       |01/Apr 02:40                               |
|User_2                    |Region_6                                   |OTHER                                      |02/Sep 13:54                               |
Query terminated
```

Create a stream from other streams:

```
ksql> create stream avro_stream2 as SELECT userid,regionid,gender,TIMESTAMPTOSTRING(REGISTERTIME, 'dd/MMM HH:mm') as time from  avro_streaming_final where gender='FEMALE';
>

 Message
--------------------------------------------
 Created query with ID CSAS_AVRO_STREAM2_39
--------------------------------------------
ksql> describe avro_stream2;

Name                 : AVRO_STREAM2
 Field    | Type
----------------------------
 USERID   | VARCHAR(STRING)
 REGIONID | VARCHAR(STRING)
 GENDER   | VARCHAR(STRING)
 TIME     | VARCHAR(STRING)
----------------------------
For runtime statistics and query details run: DESCRIBE EXTENDED <Stream,Table>;
ksql> describe extended avro_stream2;

Name                 : AVRO_STREAM2
Type                 : STREAM
Timestamp field      : Not set - using <ROWTIME>
Key format           : KAFKA
Value format         : AVRO
Kafka topic          : AVRO_STREAM2 (partitions: 1, replication: 1)
Statement            : CREATE STREAM AVRO_STREAM2 WITH (KAFKA_TOPIC='AVRO_STREAM2', PARTITIONS=1, REPLICAS=1) AS SELECT
  AVRO_STREAMING_FINAL.USERID USERID,
  AVRO_STREAMING_FINAL.REGIONID REGIONID,
  AVRO_STREAMING_FINAL.GENDER GENDER,
  TIMESTAMPTOSTRING(AVRO_STREAMING_FINAL.REGISTERTIME, 'dd/MMM HH:mm') TIME
FROM AVRO_STREAMING_FINAL AVRO_STREAMING_FINAL
WHERE (AVRO_STREAMING_FINAL.GENDER = 'FEMALE')
EMIT CHANGES;

 Field    | Type
----------------------------
 USERID   | VARCHAR(STRING)
 REGIONID | VARCHAR(STRING)
 GENDER   | VARCHAR(STRING)
 TIME     | VARCHAR(STRING)
----------------------------

Queries that write from this STREAM
-----------------------------------
CSAS_AVRO_STREAM2_39 (RUNNING) : CREATE STREAM AVRO_STREAM2 WITH (KAFKA_TOPIC='AVRO_STREAM2', PARTITIONS=1, REPLICAS=1) AS SELECT   AVRO_STREAMING_FINAL.USERID USERID,   AVRO_STREAMING_FINAL.REGIONID REGIONID,   AVRO_STREAMING_FINAL.GENDER GENDER,   TIMESTAMPTOSTRING(AVRO_STREAMING_FINAL.REGISTERTIME, 'dd/MMM HH:mm') TIME FROM AVRO_STREAMING_FINAL AVRO_STREAMING_FINAL WHERE (AVRO_STREAMING_FINAL.GENDER = 'FEMALE') EMIT CHANGES;

For query topology and execution plan please run: EXPLAIN <QueryId>

Local runtime statistics
------------------------
messages-per-sec:   4316.41   total-messages:    429651     last-message: 2021-03-06T11:35:47.774Z

(Statistics of the local KSQL server interaction with the Kafka topic AVRO_STREAM2)

Consumer Groups summary:

Consumer Group       : _confluent-ksql-default_query_CSAS_AVRO_STREAM2_39

Kafka topic          : users
Max lag              : 660774

 Partition | Start Offset | End Offset | Offset  | Lag
----------------------------------------------------------
 0         | 0            | 1919578    | 1258804 | 660774
----------------------------------------------------------

ksql> select * from avro_stream2 emit changes;
+-------------------------------------------+-------------------------------------------+-------------------------------------------+-------------------------------------------+
|USERID                                     |REGIONID                                   |GENDER                                     |TIME                                       |
+-------------------------------------------+-------------------------------------------+-------------------------------------------+-------------------------------------------+
|User_5                                     |Region_8                                   |FEMALE                                     |23/Sep 08:53                               |
|User_9                                     |Region_8                                   |FEMALE                                     |25/Aug 05:17                               |
|User_5                                     |Region_6                                   |FEMALE                                     |07/Dec 02:54                               |
|User_9                                     |Region_6                                   |FEMALE                                     |17/Jun 19:32                               |

^CQuery terminated

ksql> terminate CSAS_AVRO_STREAM2_39;

 Message
-------------------
 Query terminated.
-------------------
ksql> drop stream if exists avro_stream2;

 Message
----------------------------------------------------------
 Source `AVRO_STREAM2` (topic: AVRO_STREAM2) was dropped.
----------------------------------------------------------

```

Tables:
=======
A table in kafka is a state now.

updates a msg with a same key and adds if no key exists.

```
ksql> create table countrytablelatest(countrycode VARCHAR PRIMARY KEY ,countryname VARCHAR) with (KAFKA_TOPIC='streaming-csv',VALUE_FORMAT='delimited');

 Message
---------------
 Table created
---------------
ksql> select * from countrytablelatest;

```

JOINS:
=======
Same as join in sql
1.stream join stream---stream
2.table join table --table
3.stra, jpin table --stream


pull queries:
=============
```
ksql> create table pullqtest as select userid,count(*) as users_cnt from AVRO_STREAMING_FINAL group by userid;

 Message
-----------------------------------------
 Created query with ID CTAS_PULLQTEST_47
-----------------------------------------
ksql> select * from pullqtest;
Missing WHERE clause.  See https://cnfl.io/queries for more info.
Add EMIT CHANGES if you intended to issue a push query.
Pull queries require a WHERE clause that:
 - limits the query to a single key, e.g. `SELECT * FROM X WHERE <key-column>=Y;`.
ksql> select * from pullqtest where userid='User_1';
+--------------------------------------------------------------------+-----------------------------------------------------------------------------------------+
|USERID                                                              |USERS_CNT                                                                                |
+--------------------------------------------------------------------+-----------------------------------------------------------------------------------------+
|User_1                                                              |218936                                                                                   |
Query terminated
ksql>

#Automatically terminates query 
```

QueryExplain plans:
===================

Explain shows the query plan of a query

```

ksql> show queries;

 Query ID                | Query Type | Status    | Sink Name       | Sink Kafka Topic | Query String                                                                                  
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 CSAS_USER_DATA_NEW_1_13 | PERSISTENT | RUNNING:1 | USER_DATA_NEW_1 | USER_DATA_NEW_1  | CREATE STREAM USER_DATA_NEW_1 WITH (KAFKA_TOPIC='USER_DATA_NEW_1', PARTITIONS=1, REPLICAS=1) AS SELECT   TIMESTAMPTOSTRING(USER_DATA.REGISTERTIME, 'dd/MMM HH:mm') KSQL_COL_0,   * FROM USER_DATA USER_DATA EMIT CHANGES;
 CTAS_PULLQTEST_47       | PERSISTENT | RUNNING:1 | PULLQTEST       | PULLQTEST        | CREATE TABLE PULLQTEST WITH (KAFKA_TOPIC='PULLQTEST', PARTITIONS=1, REPLICAS=1) AS SELECT   AVRO_STREAMING_FINAL.USERID USERID,   COUNT(*) USERS_CNT FROM AVRO_STREAMING_FINAL AVRO_STREAMING_FINAL GROUP BY AVRO_STREAMING_FINAL.USERID EMIT CHANGES;
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
For detailed information on a Query run: EXPLAIN <Query ID>;
ksql> Explain CSAS_USER_DATA_NEW_1_13;

ID                   : CSAS_USER_DATA_NEW_1_13
Query Type           : PERSISTENT
SQL                  : CREATE STREAM USER_DATA_NEW_1 WITH (KAFKA_TOPIC='USER_DATA_NEW_1', PARTITIONS=1, REPLICAS=1) AS SELECT
  TIMESTAMPTOSTRING(USER_DATA.REGISTERTIME, 'dd/MMM HH:mm') KSQL_COL_0,
  *
FROM USER_DATA USER_DATA
EMIT CHANGES;
Host Query Status    : {ksqldb-server:8088=RUNNING}

 Field        | Type
--------------------------------
 KSQL_COL_0   | VARCHAR(STRING)
 USERID       | VARCHAR(STRING)
 REGIONID     | VARCHAR(STRING)
 GENDER       | VARCHAR(STRING)
 REGISTERTIME | BIGINT
--------------------------------

Sources that this query reads from:
-----------------------------------
USER_DATA

For source description please run: DESCRIBE [EXTENDED] <SourceId>

Sinks that this query writes to:
-----------------------------------
USER_DATA_NEW_1

For sink description please run: DESCRIBE [EXTENDED] <SinkId>

Execution plan
--------------
 > [ SINK ] | Schema: KSQL_COL_0 STRING, USERID STRING, REGIONID STRING, GENDER STRING, REGISTERTIME BIGINT | Logger: CSAS_USER_DATA_NEW_1_13.USER_DATA_NEW_1
                 > [ PROJECT ] | Schema: KSQL_COL_0 STRING, USERID STRING, REGIONID STRING, GENDER STRING, REGISTERTIME BIGINT | Logger: CSAS_USER_DATA_NEW_1_13.Project
                                 > [ SOURCE ] | Schema: USERID STRING, REGIONID STRING, GENDER STRING, REGISTERTIME BIGINT, ROWTIME BIGINT | Logger: CSAS_USER_DATA_NEW_1_13.KsqlTopic.Source


Processing topology
-------------------
Topologies:
   Sub-topology: 0
    Source: KSTREAM-SOURCE-0000000000 (topics: [users])
      --> KSTREAM-TRANSFORMVALUES-0000000001
    Processor: KSTREAM-TRANSFORMVALUES-0000000001 (stores: [])
      --> Project
      <-- KSTREAM-SOURCE-0000000000
    Processor: Project (stores: [])
      --> KSTREAM-SINK-0000000003
      <-- KSTREAM-TRANSFORMVALUES-0000000001
    Sink: KSTREAM-SINK-0000000003 (topic: USER_DATA_NEW_1)
      <-- Project



ksql> Explain CTAS_PULLQTEST_47;

ID                   : CTAS_PULLQTEST_47
Query Type           : PERSISTENT
SQL                  : CREATE TABLE PULLQTEST WITH (KAFKA_TOPIC='PULLQTEST', PARTITIONS=1, REPLICAS=1) AS SELECT
  AVRO_STREAMING_FINAL.USERID USERID,
  COUNT(*) USERS_CNT
FROM AVRO_STREAMING_FINAL AVRO_STREAMING_FINAL
GROUP BY AVRO_STREAMING_FINAL.USERID
EMIT CHANGES;
Host Query Status    : {ksqldb-server:8088=RUNNING}

 Field     | Type
------------------------------------
 USERID    | VARCHAR(STRING)  (key)
 USERS_CNT | BIGINT
------------------------------------

Sources that this query reads from:
-----------------------------------
AVRO_STREAMING_FINAL

For source description please run: DESCRIBE [EXTENDED] <SourceId>

Sinks that this query writes to:
-----------------------------------
PULLQTEST

For sink description please run: DESCRIBE [EXTENDED] <SinkId>

Execution plan
--------------
 > [ SINK ] | Schema: USERID STRING KEY, USERS_CNT BIGINT | Logger: CTAS_PULLQTEST_47.PULLQTEST
                 > [ PROJECT ] | Schema: USERID STRING KEY, USERS_CNT BIGINT | Logger: CTAS_PULLQTEST_47.Aggregate.Project
                                 > [ AGGREGATE ] | Schema: USERID STRING KEY, USERID STRING, ROWTIME BIGINT, KSQL_AGG_VARIABLE_0 BIGINT | Logger: CTAS_PULLQTEST_47.Aggregate.Aggregate
                                                 > [ GROUP_BY ] | Schema: USERID STRING KEY, USERID STRING, ROWTIME BIGINT | Logger: CTAS_PULLQTEST_47.Aggregate.GroupBy
                                                                 > [ PROJECT ] | Schema: USERID STRING, ROWTIME BIGINT | Logger: CTAS_PULLQTEST_47.Aggregate.Prepare
                                                                                 > [ SOURCE ] | Schema: USERID STRING, REGIONID STRING, GENDER STRING, REGISTERTIME BIGINT, ROWTIME BIGINT | Logger: CTAS_PULLQTEST_47.KsqlTopic.Source


Processing topology
-------------------
Topologies:
   Sub-topology: 0
    Source: KSTREAM-SOURCE-0000000000 (topics: [users])
      --> KSTREAM-TRANSFORMVALUES-0000000001
    Processor: KSTREAM-TRANSFORMVALUES-0000000001 (stores: [])
      --> Aggregate-Prepare
      <-- KSTREAM-SOURCE-0000000000
    Processor: Aggregate-Prepare (stores: [])
      --> KSTREAM-FILTER-0000000003
      <-- KSTREAM-TRANSFORMVALUES-0000000001
    Processor: KSTREAM-FILTER-0000000003 (stores: [])
      --> Aggregate-GroupBy
      <-- Aggregate-Prepare
    Processor: Aggregate-GroupBy (stores: [])
      --> Aggregate-GroupBy-repartition-filter
      <-- KSTREAM-FILTER-0000000003
    Processor: Aggregate-GroupBy-repartition-filter (stores: [])
      --> Aggregate-GroupBy-repartition-sink
      <-- Aggregate-GroupBy
    Sink: Aggregate-GroupBy-repartition-sink (topic: Aggregate-GroupBy-repartition)
      <-- Aggregate-GroupBy-repartition-filter

  Sub-topology: 1
    Source: Aggregate-GroupBy-repartition-source (topics: [Aggregate-GroupBy-repartition])
      --> KSTREAM-AGGREGATE-0000000005
    Processor: KSTREAM-AGGREGATE-0000000005 (stores: [Aggregate-Aggregate-Materialize])
      --> Aggregate-Aggregate-ToOutputSchema
      <-- Aggregate-GroupBy-repartition-source
    Processor: Aggregate-Aggregate-ToOutputSchema (stores: [])
      --> Aggregate-Project
      <-- KSTREAM-AGGREGATE-0000000005
    Processor: Aggregate-Project (stores: [])
      --> KTABLE-TOSTREAM-0000000011
      <-- Aggregate-Aggregate-ToOutputSchema
    Processor: KTABLE-TOSTREAM-0000000011 (stores: [])
      --> KSTREAM-SINK-0000000012
      <-- Aggregate-Project
    Sink: KSTREAM-SINK-0000000012 (topic: PULLQTEST)
      <-- KTABLE-TOSTREAM-0000000011



Overridden Properties
---------------------
 Property          | Value
------------------------------
 auto.offset.reset | earliest
------------------------------

ksql>
#CTAS_xxx=---create table as select 
#CSAS_xxx=---create stream as select
```




Kafka connect :
=============

Kafka connect  is a framework is used to connect to data sources and stream data into kafka and send data from kafka.

Kafka Connect includes two types of connectors:

Source connector – Ingests entire databases and streams table updates to Kafka topics. A source connector can also collect metrics from all your application servers and store these in Kafka topics, making the data available for stream processing with low latency.
Sink connector – Delivers data from Kafka topics into secondary indexes such as Elasticsearch, or batch systems such as Hadoop for offline analysis.

```
ksql> CREATE SOURCE CONNECTOR `jdbc-connector` WITH(
>    "connector.class"='io.confluent.connect.jdbc.JdbcSourceConnector',
>    "connection.url"='jdbc:postgresql://localhost:5432/my.db',
>    "mode"='incrementing',
>    "topic.prefix"='jdbc-',
>    "table.whitelist"='test',
>    "key"='id');
```

File Stream source connect:
===========================

Three params that are needed for any kafka connect:
1.name:used for offsets groupid etc.
2.connectorclass:class for the connector
3.tasks.max:1