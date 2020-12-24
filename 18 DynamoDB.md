

18 DynamoDB
===




## Table of Contents

- [18 DynamoDB](#18-dynamodb)
  * [Table of Contents](#table-of-contents)
  * [Introduction](#introduction)
  * [DynamoDB](#dynamodb)
    + [Basics](#basics)
    + [Primary Keys](#primary-keys)
      - [Example](#example)
    + [Provisioned Throughput](#provisioned-throughput)
      - [Write Capacity Units](#write-capacity-units)
      - [Strongly Consistent Read vs Eventually Consistent Read](#strongly-consistent-read-vs-eventually-consistent-read)
      - [Read Capacity Units](#read-capacity-units)
    + [Partitions Internal](#partitions-internal)
    + [Throttling](#throttling)
    + [Basic APIs](#basic-apis)
      - [Writing Data](#writing-data)
      - [Deleting Data](#deleting-data)
      - [Batching Writes](#batching-writes)
      - [Reading Data](#reading-data)
      - [Queries](#queries)
      - [Scan](#scan)
    + [Indexes](#indexes)
      - [Local Secondary Index](#local-secondary-index)
      - [Global Secondary Index](#global-secondary-index)
      - [Indexes and Throttling](#indexes-and-throttling)
    + [Concurrency](#concurrency)
    + [DynamoDB Accelerator (DAX)](#dynamodb-accelerator--dax-)
      - [DAX vs ElastiCache](#dax-vs-elasticache)
    + [DynamoDB Streams](#dynamodb-streams)
      - [Streams and Lambda](#streams-and-lambda)
    + [Time to Live (TTL)](#time-to-live--ttl-)
    + [CLI - Good to Know](#cli---good-to-know)
    + [Transactions](#transactions)
    + [Session State Cache](#session-state-cache)
    + [Write Sharding/Reverting Pattern](#write-sharding-reverting-pattern)
    + [Write Types](#write-types)
    + [DynamoDB with S3](#dynamodb-with-s3)
      - [Large Objects Pattern](#large-objects-pattern)
      - [Indexing S3 Objects Metadata](#indexing-s3-objects-metadata)
    + [Operations](#operations)
    + [Security and Other Features](#security-and-other-features)
    + [Console](#console)
      - [WCU and RCU](#wcu-and-rcu)
      - [Basic APIs](#basic-apis-1)
      - [LSI and GSI](#lsi-and-gsi)
      - [DAX](#dax)
      - [Streams](#streams)
      - [TTL](#ttl)
      - [CLI](#cli)
  * [Quiz](#quiz)




Introduction
---
- traditional architecture
    - trad apps leverage rdbms dbs
    - these dbs have sql query lang
    - strong requirements abt how data shld be modeled
    - ability to join, aggregations, computations
        - can become very compute heavy/costly
    - vertical scaling
        - means getting more powerful cpu/ram/io

![](https://i.imgur.com/7ZyiDG9.png)

- nosql dbs
    - are non-relational dbs and are distributed
    - include mongodb, dynamodb etc.
    - do not support join
        - all data that is needed for a query is present in 1 row
    - dont perform aggregations like sum
    - scale horizontally
- NOTE
    - no right or wrong for nosql vs sql
        - just abt modelling data differently and thinking abt user queries differently

DynamoDB
---
- fully managed, highly available with replication across 3 az
- nosql db - not relational db
- scales to massive workloads
    - distributed db
- millions of requests per sec
    - trillions of rows
    - 100s of tb of storage
- fast and consistent in perf
    - is because its distributed
    - low latency on retrieval
- integrated with iam for security, authorisation and administration
- enables event driven programming with dynamodb streams
- low cost and auto scaling capabilities

### Basics
- made of tables
    - ea table has __primary key__
        - decided at creation time
    - ea table can have infinite num of items (rows)
        - ea item has __attributes__
            - attrs can be added over time
            - can also be null
        - max size of item is 400kb
- data types supported
    - scalar types
        - string, number, binary, boolean, null
    - document types
        - list, map
    - set types
        - string set, number set, binary set
        - allows you to do nested stuff

### Primary Keys
- 2 main options (below)
- partition key only (HASH)
    - must be unique for ea item
    - must be diverse so data is distributed
    - eg. user_id for users table
- partition key + sort key
    - combination must be unique
    - data is grped by partition key
    - sort key == range key
        - used for efficient queries
    - eg. users-games table
        - user_id for partition key
        - game_id for sort key
        - can be same user but maybe they play a diff game

![](https://i.imgur.com/EuIom4A.png)
![](https://i.imgur.com/xsB8yRp.png)

#### Example
- building movie db
    - what's best partition key to maximise data distribution?
        - movie id
        - producer name
        - lead actor name
        - movie language
- answer
    - movie id has highest cardinality so it's good candidate
    - movie lang dont take many values and may be skewed towards english so not great partition key

### Provisioned Throughput
- table must have provisioned read and write capacity unis
- __read capacity units (RCU)__ - throughput for reads
- __write capacity units (WCU__ - throughput for writes
- options to setup autoscaling of throughput to meet demand
- throughput can be exceeded temporarily using __burst credit__
    - if burst credit empty, you'll get `ProvisionedThroughputException`
    - advised to do exponential backoff retry
- NOTE
    - exam might ask u to calculate some throughputs so have to rmb some formulas

#### Write Capacity Units
- 1 write capacity unit represents 1 write per second for an item up to 1kb in size
    - if items larger than 1kb, more wcu consumed
- examples
    - write 10 objs per seconds of 2kb each
        - need 2 * 10 = 20 wcu
    - write 6 objs per second of 4.5kb ea
        - 6 * 5 = 30wcu
        - 4.5 gets rounded to upper kb
    - writw 120 objs per min of 2kb ea
        - need 120/60 * 2 = 4 wcu

#### Strongly Consistent Read vs Eventually Consistent Read
- eventually consistent read
    - if read just after a write, its possible we'll get unexpected resp because of replication
- strongly consistent read
    - if read just after write, will get correct data
- by default
    - dynamodb uses eventually consistent reads but `GetItem`, `Query` and `Scan` provides a `ConsistentRead` param u can set to true
- why dw to use strongly consistent read all the time?
    - will have impact to your rcu

![](https://i.imgur.com/DoV4qSj.png)
- 3 apps, dynamodb will replicate it to 3 az
- app write to 1st server
    - data replicated to server 2 and 3
    - if read before replication has happened or fully complete, you are in eventual consistent read pattern
    - if ask for strongly consistent read, read will be a bit longer

#### Read Capacity Units
- 1 rcu represents 1 strongly consistent read per second or 2 eventually consistent reads per sec for an item up to 4kb in size
    - if items >4kb, more rcu consumed
- example
    - 10 strongly consistent reads per sec of 4kb ea
        - 10 * 4kb/4kb = 10rcu
    - 16 eventually consistent reads per seconds of 12kb ea
        - (16/2) * (12/4) = 24 rcu
    - 10 strongly consistent reads per sec of 6kb ea
        - 10 * 8kb/4 = 20 rcu
        - need to round up 6kb to 8kb
            - doesnt do intermediate in size, goes by increments by 4kb
- NOTE
    - need to rmb all the formulas, WILL ASK in exam!!

### Partitions Internal
- data divided in partitions
    - partition keys go through hashing algo to know which partition they go to
- to compute num of partitions (is advanced, not asked in exam)
    - by capacity
        - (total rcu / 3000) + (total wcu / 1000)
    - by size
        - total size / 10gb
    - total partitions
        - ceiling(max(capacity, size))
- wcu and rcu are spread evenly between partitions
    - if 10 partitions and 100 rcu and 100 wcu, ea partition gets 10

### Throttling
- if exceed our rcu or wcu, we get `ProvisionedThroughputExceededExceptions`
- reasons
    - hot keys
        - 1 partition key read too many times (popular item for ex)
    - hot partitions
    - very large items
        - rmb rcu and wcu depends on size of items
- solutions
    - exponential backoff when exception encountered
        - alr in sdk
    - distribute partition keys as much as possible
    - if rcu issue, can use dynamodb accelerator (DAX)
        - is basically a cache

### Basic APIs
#### Writing Data
- `PutItem`
    - write data to dynamodb
    - create data or full replace
    - consumes wcu
- `UpdateItem`
    - update data in dynamodb
    - partial update of attributes
    - possibility to use atomic counters and increase them
- `ConditionalWrites`
    - accept a write/update only if conditions respected else reject
        - eg. only update this row only if this attribute still has this value
    - helps with concurrent access to items
    - no performance impact

#### Deleting Data
- `DeleteItem`
    - delete an indiv row
    - abi to perform conditional delete
- `DeleteTable`
    - delete whole table and all its items
    - quicker deletion than calling `DeleteItem` on all items

#### Batching Writes
- `BatchWriteItem`
    - up to 25 `PutItem` and/or `DeleteItem` in 1 call
    - up to 16mb of data written
    - up to 400kb of data per item
- batching allows u to save in latency by reducing the num of api calls done against dynamodb
    - operations done in parallel for better efficiency
        - done by dynamodb for u
    - possible for part of batch to fail, in which case we'll try the failed items
        - using exponential backoff algo

#### Reading Data
- `GetItem`
    - read based on primary key
        - pri key = hash or hash-range
    - eventually consistent read by default
    - option to use strongly consistent reads
        - more rcu, might take longer
    - `ProjectExpression` can be specified to include only certain attributes
        - save network bandwidth
- `BatchGetItem`
    - up to 100 items
    - up to 16mb of data
    - items retrieved in parallel to minimise latency

#### Queries
- `Query` returns items based on
    - partition key val
        - must be `=` operator
    - sortkey value
        - `=, <, <=, >, >=, Between, Begin`
            - optional
        - `FilterExpression` to further filtering
            - filtering is done client side
    - eg. we know partition key in advance and use sort key to get tailored results
- returns
    - up to 1mb of data
    - or num of items specified in `Limit`
- able to do pagination on results
- can query table, local secondary index or global secondary index
- NOTE
    - this is the efficient way to query dynamodb

#### Scan
- `Scan` entire table then filter out data
    - very inefficient
- returns up to 1mb of data
    - use pagination to keep on reading
- consumes a lot of rcu
    - limit impact using `Limit` or reduce size of result and pause
- for faster performance, use __parallel scans__
    - multiple instances scan multiple partitions at same time
    - increase throughput and rcu consumed
    - limit impact of parallel scans just like you would for scans
- can use `ProjectExpression` + `FilterExpression`
    - no change to rcu

### Indexes
#### Local Secondary Index
- alternate range key for your table, local to the hash key
    - up to 5 local secondary indexes per table
- sort key consists of exactly 1 scalar attribute
    - the attr that you choose must be scalar string, number or binary
- __LSI must be defined at table creation time__

![](https://i.imgur.com/w5Tpdsq.png)
- in a table u can use partition or sort key to query rows
    - but u cant do that for timestamp..have to retrieve during a scan and then filter out
    - hence can have LSI defined on timestamp col to have efficient queries by timestamp
        - basically data will also be ordered in an index by user_id and game timestamp
        - can now sort based on user_id or timestamp directly

#### Global Secondary Index
- to speed up queries on non-key attributes, use global secondary index
    - GSI = partition key + optional sort key
    - basically allow u to define a whole new index
- index is new table and can project attributes to it
    - parition key and sort key of orig table are always projected
        - `KEYS_ONLY`
    - can specify extra attributes to project
        - `INCLUDE`
    - can use all attributes from main table
        - `ALL`
- must define rcu/wcu for index
- possibility to add/modify GSI as u go but not LSI

![](https://i.imgur.com/PjGtH0l.png)
- can query by user_id and game_id easily
    - say want to query by game_id, then we'll need to have game_id be the partition key
    - hence we define an INDEX (GSI) which have the queries by game_id
        - game_id is our partition key

![](https://i.imgur.com/EuORyOw.png)


#### Indexes and Throttling
- GSI
    - if writes are throttled on GSI, main table will be throttled
        - even if wcu on main tables are fine
        - __GSIs can throttle your main table__
    - hence, choose GSI partition key carefully
    - assign wcu capacity carefully
- LSI
    - use wcu and rcu on main table
    - no special throttling considerations
        - if LSI is throttled, means main table is throttled as well

### Concurrency
- dynamodb has feature called conditional update/delete
    - means u can ensure an item hasn't changed before altering it
    - eg. ensure we delete sth only if certain condition is validated
- makes dynamodb an __optimistic locking/concurrency db__

![](https://i.imgur.com/zB3DIxp.png)
- eg. client1 say update name to john only if item version is 1
    - maybe client2 will update it first hence client1's request will be denied

### DynamoDB Accelerator (DAX)
- seamless cache for dynamodb, no app re-write
    - writes go through dax to dynamodb
    - micro second latency for cached reads and queries
        - solves hot key problem (too many reads)
- 5 mins ttl for cache by default
- up to 10 nodes in cluster
    - can scale but have to provision in advance
- multi az
    - 3 nodes minimum recommended for production
    - resilient
- secure
    - encryption at rest with kms
    - vpc
    - iam
    - cloudtrial

![](https://i.imgur.com/1pMfYsm.png)

#### DAX vs ElastiCache
![](https://i.imgur.com/oUw5khB.png)
- for indiv obj cache, query or scan caches, dax is perfect
    - can just retrieve these items out of dax very quickly
    - dax is prob also built on elasticache
- can still use elasticache for more advanced stuff
    - eg. client gets a lot of data from dynamodb table and performs computations to aggregate the data
        - can store aggregation result in elasticache so we can just retrieve it there anytime

### DynamoDB Streams
- changes in dynamodb can end up in dynamodb stream
- this stream can be read by aws lambda and ec2 instances
    - hence we can
        - react to changes in realtime
            - eg. welcome email to new users
        - analytics
        - create derivative tables/views
            - so can read from table in realtime, process it and update another table based on that
        - insert into elasticache
            - to provide a search capability
- can implement cross region replication using streams
- stream has 24hours of data retention
    - u cannot change this

![](https://i.imgur.com/47vaBvz.png)

- we have to choose info that will be written to the stream whenever the data in the table is modified
    - `KEYS_ONLY` - only key attributes of modified item
    - `NEW_IMAGE` - entire item as it appears after its modified
    - `OLD_IMAGE` - entire item, as it appeared before it was modified
    - `NEW_AND_OLD_IMAGES` - both new and old imgs of the item
- dynamodb streams are made of shards just like kinesis data streams
    - u dont provision shards
        - automated by aws
- records are not retroactively populated in a stream after enabling it
    - once u enable the stream, only the future changes will end up in thr

#### Streams and Lambda
- need to define an event src mapping to read from dynamodb streams
- need to ensure lambda func has appropriate permissions
- lambda func is invoked synchronously

![](https://i.imgur.com/kRyAYyn.png)

### Time to Live (TTL)
- ttl - automatically delete an item after expiry date/time
- provided at no extra cost
    - deletions dont use wcu/rcu
- is background task operated by dynamodb service itself
    - helps reduce storage and manage table size over time
    - helps adhere to regulatory norms
- enabled per row
    - u define a ttl col and add a date thr
- dynamodb typically deletes expired items within 48 hours of expiration
    - deleted items due to ttl also deleted in GSI/LSI
- streams can help recover expired items
    - will have an event in streams after deletion due to ttl

### CLI - Good to Know
- `--project-expression`
    - attributes to retrieve
- `--filter-expression`
    - filter results
- general cli pagination options including dynamodb/s3
    - optimisation
        - `--page-size`
            - full dataset still received but ea api call will request less data
            - helps avoid timeouts
    - pagination
        - `--max-items`
            - max number of results returned by cli
            - returns `NextToken`
        - `--starting-token`
            - specify last received `NextToken` to keep on reading
- NOTE
    - alot of services have similar cli pattern as optimisation and pagination but project expr and filter expr is limited to dynamodb

### Transactions
- new feature from nov 2018
- transaction = abi to create/update/delete multiple rows in diff tables at same time
    - all or nothing type of operation
        - either everything happens or nothing happens
- write modes
    - standard
    - transactional
- read modes
    - eventual consistency
    - strong consistency
    - transactional
- consume 2x of wcu/rcu

### Session State Cache
- common to use dynamodb to store session state
- vs elasticache
    - elasticache is in-memory, but dynamodb is serverless
    - both are key/val stores
- vs efs
    - efs must be attached to ec2 instances as network drive
- vs ebs and instance store
    - ebs and instance store can only be used for local caching not shared caching
        - as must be physically attached
- vs s3
    - s3 is higher latency
        - not meant for small objs

### Write Sharding/Reverting Pattern
- eg. voting app with 2 candidates, A and B
    - if use part key of candidate_id, will run into partition issues as have 2 partitions
- solution
    - add suffix into part key
        - usually random suffix, sometimes calcuclated suffix
        - hence can scale writes across many shards
- NOTE
    - i still dont get why this is necessary

### Write Types
- concurrent writes
    - 2 people write at same time
        - 2nd write overwrites the 1st
    - may be a bad thing depending on how app behaves

![](https://i.imgur.com/xE9sne8.png)

- atomic writes
    - both writes succeed and add up tgt
        - what happens if it's string??? append?
    - eg. counting how many users visit website
    - not recommended if over/undercounting not tolerated

![](https://i.imgur.com/42CJInm.png)

- conditional writes
    - reject 2nd write since condition will fail

![](https://i.imgur.com/wjt4DpB.png)

- batch writes

![](https://i.imgur.com/jZhtLxD.png)


### DynamoDB with S3
#### Large Objects Pattern
![](https://i.imgur.com/bAzaz6b.png)
- large obj go into s3
    - metadata of bucket into dynamodb
        - obj key, id and whr its located in s3

#### Indexing S3 Objects Metadata
- s3 trigger event notif which triggers lambda func
    - func write metadata of s3 into dynamodb
        - dynamodb is much better way of storing data for indexing, searching and querying

### Operations
- table cleanup
    - option 1 - scan + delete
        - very slow, expensive
        - scan consumes a lot of rcu and wcu
    - option 2 - drop table + recreate table
        - fast, cheap, efficient
- copying a dynamodb table
    - option 1
        - use aws datapipeline
            - uses EMR (big data tool)
            - take dynamodb, put into s3 then take it out of s3 and into a new dynamodb table
            - though quite outdated
    - option 2
        - create backup and restore backup into new table name
            - can take some time
    - option 3
        - scan + write
            - write own code

![](https://i.imgur.com/Tg6lBA1.png)


### Security and Other Features
- security
    - vpc edpts
        - avail to dynamodb w/o internet
    - access fully controlled by iam
    - encryption at rest using kms
    - encryption in transit using ssl/tls
- backup and restore feature available
    - point in time restore like rds
    - no perf impact
        - based on strings
- global tables
    - multi region
    - fully replicated
    - high performance
- amazon dms can be used to migrate to dynamodb
    - from mongle, oracle, mysql, s3 etc.
- can launch dynamodb on your comp for developmental purposes









### Console
![](https://i.imgur.com/Khz3nSs.png)
- create table not database
- partition key can be in string, binary or number

![](https://i.imgur.com/KhD9s6P.png)
- settings
    - unlocked if u uncheck use default settings
- can set read/write capacity with estimated cost shown

![](https://i.imgur.com/namsl7l.png)
- a lot of tabs each containing alot of info

![](https://i.imgur.com/uR5ME8Z.png)

- arn can be found here too

![](https://i.imgur.com/sZG2thM.png)
- can add data here

![](https://i.imgur.com/a1nYc50.png)
- can append more fields when adding item

![](https://i.imgur.com/E5VzO7i.png)
- in dynamodb, fields in table can be null
    - u cannot enforce any field to not be null except for the partition key and sort key
    - the fields are also indepedent of one another

![](https://i.imgur.com/d2BjRwh.png)
- to access 3rd party api and give access to your table


#### WCU and RCU
![](https://i.imgur.com/S1BlAqh.png)
- capacity tab

![](https://i.imgur.com/g6R7D1c.png)
- included capacity calculator


- can also enable auto scaling
    - eg. 70% of wcu or rcu to be used all the time
    - also ninclude min and max units

#### Basic APIs
![](https://i.imgur.com/81wO3vg.png)
- actions u can apply on an item

![](https://i.imgur.com/AmUowJr.png)
- a scan is actually done automatically under the hood every time we open the items tab

![](https://i.imgur.com/g8iYqsd.png)
- switch to query tab from scan

#### LSI and GSI
![](https://i.imgur.com/1UUqLKH.png)
- can add secondary indexes when creating your table

![](https://i.imgur.com/2QLanU3.png)
- lets u choose a pri key
    - if the name of your partition key is same as pri key when creating your index, it allows you to create it as an LSI
        - else its greyed out
- LSI must be created at table creation time

![](https://i.imgur.com/phPjCVx.png)
![](https://i.imgur.com/1BkYe2Y.png)
- can switch indexes to query on

![](https://i.imgur.com/Ha5Tu7F.png)
- now can create GSI from table console

#### DAX
![](https://i.imgur.com/hBS6v85.png)
- just go to dax from the sidebar menu

![](https://i.imgur.com/MLaRUEz.png)
- create dax cluster
    - node type
    - cluster size
    - etc.

![](https://i.imgur.com/Fbnulzm.png)
- cluster settings can be left at default

#### Streams
![](https://i.imgur.com/BCANBzw.png)
- click manage stream in table console

![](https://i.imgur.com/ZwGAXfz.png)
- can choose to create new func or use existing

![](https://i.imgur.com/ZALtznD.png)
- edit dynamodb trigger 

![](https://i.imgur.com/OGUEPlp.png)

![](https://i.imgur.com/RwuomUg.png)

- after func has correct iam role to poll from dynamodb table, shld see func in the trigger tab now

![](https://i.imgur.com/uZUume7.png)
- example of a dynamodb record in cloudwatch log
    - new image (this is for insert)

![](https://i.imgur.com/hvy7f83.png)
- this is for modify
    - new and old image

![](https://i.imgur.com/Vic8PBC.png)
- this is for remove

#### TTL
![](https://i.imgur.com/EMwxkvL.png)
- create `expire_on` field in table and add time (in epoch)

![](https://i.imgur.com/G3mWRIy.png)
- click manage ttl in table overview tab

![](https://i.imgur.com/XvcbOse.png)
- run preview to see what items will expire by xxx date


#### CLI
![](https://i.imgur.com/q90pRRr.png)

![](https://i.imgur.com/93f4LuZ.png)
- filter expr
    - filter user_id = u
    - u is in expr attribute values obj
        - the S in u equates to string

![](https://i.imgur.com/yzsjLlr.png)
- if no more next token given when using max-items, means you're done reading the table

Quiz
---
![](https://i.imgur.com/WNYWrua.png)
![](https://i.imgur.com/wCgoD1H.png)





###### tags: `AWS Developer Associate` `Notes`