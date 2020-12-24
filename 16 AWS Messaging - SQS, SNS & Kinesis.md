

16 AWS Messaging - SQS, SNS & Kinesis
===



## Table of Contents

- [16 AWS Messaging - SQS, SNS & Kinesis](#16-aws-messaging---sqs--sns---kinesis)
  * [Table of Contents](#table-of-contents)
  * [Messaging in AWS](#messaging-in-aws)
  * [AWS SQS](#aws-sqs)
    + [Queues](#queues)
      - [Standard Queues](#standard-queues)
    + [Messages](#messages)
      - [Producing Messages](#producing-messages)
      - [Consuming Messages](#consuming-messages)
    + [Use Cases](#use-cases)
      - [SQS - Multiple EC2 Consumers](#sqs---multiple-ec2-consumers)
      - [SQS with ASG](#sqs-with-asg)
      - [Decouple between App Tiers](#decouple-between-app-tiers)
    + [SQS Security](#sqs-security)
    + [Message Visibility Timeout](#message-visibility-timeout)
    + [Dead Letter Queue](#dead-letter-queue)
    + [Delay Queue](#delay-queue)
    + [Long Polling](#long-polling)
    + [SQS Extended Client](#sqs-extended-client)
    + [Must Know API](#must-know-api)
    + [FIFO Queue](#fifo-queue)
      - [FIFO - Deduplication](#fifo---deduplication)
      - [FIFO - Message Grouping](#fifo---message-grouping)
    + [Console](#console)
      - [Message Visibility Timeout](#message-visibility-timeout-1)
      - [DLQ](#dlq)
      - [Delay Queue](#delay-queue-1)
      - [Long Polling](#long-polling-1)
      - [FIFO Queue](#fifo-queue-1)
  * [AWS SNS](#aws-sns)
    + [Publishing](#publishing)
    + [SNS Security](#sns-security)
    + [SNS + SQS: Fan Out](#sns---sqs--fan-out)
    + [S3 Events to Multiple Queues](#s3-events-to-multiple-queues)
    + [Console](#console-1)
      - [Fan Out](#fan-out)
  * [AWS Kinesis](#aws-kinesis)
    + [Streams Overview](#streams-overview)
      - [Streams Shards](#streams-shards)
    + [Kinesis API](#kinesis-api)
      - [Put Records](#put-records)
      - [Exceptions](#exceptions)
      - [Consumers](#consumers)
    + [Kinesis KCL in Depth](#kinesis-kcl-in-depth)
      - [Example](#example)
    + [Kinesis Security](#kinesis-security)
    + [Kinesis Data Analytics](#kinesis-data-analytics)
    + [Kinesis Firehose](#kinesis-firehose)
    + [Console](#console-2)
  * [SQS vs SNS vs Kinesis](#sqs-vs-sns-vs-kinesis)
    + [Data Ordering in Kinesis](#data-ordering-in-kinesis)
    + [Data Ordering in SQS](#data-ordering-in-sqs)
    + [Kinesis vs SQS Ordering](#kinesis-vs-sqs-ordering)
  * [Quiz](#quiz)



Messaging in AWS
---
- apps will have to comm with one another
    - 2 patterns of app comm
        - sync comms - app to app
        - async/event based - app to queue to app
- limitations
    - sync between apps can be problematic if thrs sudden spikes of traffic
- hence, better to debouple your apps
    - using SQS - queue model
    - using SNS - pub/sub model
    - using kinesis - realtime streaming model
    - these services can scale independently of app


AWS SQS
---
### Queues
![](https://i.imgur.com/F4C7a6V.png)
- simple queuing service (sqs)
    - __producer__ - sth that sends msgs into our sqs queue
        - can have multiple producers
    - __consumers__ - consume the msgs from the queue
        - poll msgs from queue and get info, process it and delete it from the queue
        - can also have multiple consumers

#### Standard Queues
- oldest offering - >10yo
- fully managed service
    - used to decouple apps
- attributes
    - unlimited throughput (msg sent per sec)
    - unlimited num of msgs in queue
    - default retention of msgs 4 days
        - max 14 days
    - low latency
        - < 10ms on publish and receive
    - limitation of 256kb per msg sent
- can have duplicate msgs
    - hence called at least once delivery, occasionally
- can have out of order msgs
    - best effort ordering

### Messages
#### Producing Messages
- produced to sqs using sdk
    - `SendMessage` API
- msg is persisted in sqs until consumer deletes it
- msg retention default 4 days
    - up to 14 days
- Eg. send an order to be process, msg will have
    - order id
    - customer id
    - any other attrs
- sqs standard - unlimited throughput

![](https://i.imgur.com/TQiEV4U.png)

#### Consuming Messages
- consumers - apps on ec2 instances, servers, aws lambda etc.
    - poll sqs for msgs
        - can receive up to 10 msgs at once
    - process msgs
        - Eg. insert msg into rds db
    - delete msgs with `DeleteMessage` API
        - guarantee that no other consumer can see these msgs
    - msg processing completes

![](https://i.imgur.com/VvPsJhd.png)

### Use Cases
#### SQS - Multiple EC2 Consumers
- consumers receive and process msgs __in parallel__
    - consumers delete msg after processing
- at least once delivery
- best-effort msg ordering
- can scale consumers horizontally to improve throughput for processing

![](https://i.imgur.com/KLvubZ7.png)

#### SQS with ASG
![](https://i.imgur.com/wMKHbmX.png)
- asg with ec2 polling for msgs
    - asg has to scale on a metric
        - can use queue length cloudwatch metric of sqs - `ApprroximateNumberOfMessages`
        - setup alarm whenever queue length goes over x
    - the more msgs in sqs, more ec2 instances provisioned to process said msgs at higher throughputs

#### Decouple between App Tiers
![](https://i.imgur.com/DuzPu2e.png)
- eg. vid processing needed
    - cant do it frontend, slows down website
    - hence decouple app - req and actual processing can happen in diff apps
        - send msg into sqs to process file
        - 2nd backend processing tier with own asg to receive msgs and process vids
        - finally inserts into s3 bucket
    - ec2 instances in backend can also be specially setup for vid processing
        - powerful GPUs

### SQS Security
- encryption
    - inflight encryption using https API
    - at-rest encryption using KMS keys
    - client-side encryption if client wants to perform encryption/decryption himself
- access controls
    - iam policies to regulate access to sqs api
- sqs access policies
    - similar to s3 bucket policies
    - useful for cross-acc access to sqs queues
    - useful for allowing other services (Eg. sns, s3 etc.) to write to sqs queue

### Message Visibility Timeout
- after msg polled by consumer, it becomes invisible to other consumers
    - by default, msg visibility timeout is 30 seconds
    - means msg has 30secs to be processed
- after msg visibility timeout over, the msg is visible in sqs

![](https://i.imgur.com/R0H17T1.png)
- if msg not processed within visibility timeout, it will be processed twice
    - since its received by 2 diff consumers or twice by same consumer
    - consumer can call `ChangeMessageVisbility` api to get more time
- if visi timeout is high (hours) and consumer crashes, reprocessing will take time
    - if visi timeout is low (seconds), may get duplicates
- consumers shld be programmed that if they know they need a bit more time, shld call changevisi api

### Dead Letter Queue
- if consumer fails to process msg within visi timeout, msg goes back to queue
    - can set threshold of how many times a msg can go back to queue
    - after `MaximumReceives` threshold exceeded, msg goes into __dead letter queue (DLQ)__
- very useful for debugging
    - msgs that cannot be processed sent to dlq to be analysed by another app
- make sure to process msg in dlq before they expire
    - good to set retention of 14 days in dlq for enough time for processing

![](https://i.imgur.com/7aksB4N.png)

### Delay Queue
- delay a msg up to 15mins
    - consumers dont see it immediately
    - default is 0 seconds (avail instantly)
        - can set default at queue level
        - can override default on send using `DelaySeconds` param

![](https://i.imgur.com/EuPIOkU.png)

### Long Polling
- when consumer reqs msgs from queue, can optionally wait for msgs to arrive if thr's none in queue
    - AKA long polling
    - long polling decreases the num of api calls made to sqs while increasing the efficiency and latency of your app
        - as when msg received by queue, instantly received by consumer too
    - wait time can be from 1sec to 20sec (preferable)
- long polling is preferable to short polling
- long polling can be enabled at the queue lvl or at API lvl using `WaitTimeSeconds`

### SQS Extended Client
- msg size limit is 256kb
    - how to send large msgs?
    - use sqs extended client (java library)
- use s3 bucket as repo for large data
    - small metadata with pointer to s3 sent in msg to sqs
    - consumer reads from sqs queue using the extended client lib and consume metadata and read/retrieve from the s3 bucket

![](https://i.imgur.com/vVnLCTj.png)

### Must Know API
- `CreateQueue`
    - can set `MessageRetentionPeriod`
- `DeleteQueue`
- `PurgeQueue` - delete all msgs in queue
- `SendMessage`
    - can set `DelaySeconds`
- `ReceiveMessage`
- `DeleteMessage`
- `ReceiveMessageWaitTimeSeconds` - long polling
- `ChangeMessageVisbility` - change msg timeout
- batch APIs for sendmsg, deletemsg and changemsgvisi helps decrease costs
    - by minimising num of api calls done to sqs

### FIFO Queue
- fifo (first in first out, denotes ordering of msgs in queue)
    - 1st msg to arrive is 1st to leave
- limited throughput
    - 300 msgs per sec w/o batching
    - 3000msgs per sec with batching
- exactly once send capability by removing duplicates
- msgs processed in order by consumer

![](https://i.imgur.com/jnwsmHh.png)

#### FIFO - Deduplication
- deduplication interval is 5mins
- 2 deduplication methods
    - content based deduplication
        - will do sha256 hash of msg body
        - hash then compared
    - explicitly provide msg deduplication id

![](https://i.imgur.com/AEaQRw8.png)

#### FIFO - Message Grouping
- if specify same val of `MessageGroupID` in sqs fifo queue, can only have 1 consumer
    - all msgs also in order
    - msg grp id is mandatory param
- to get ordering at lvl of subset of msgs, specify diff vals for msggrpid
    - msgs that share common msg grp id will be in order within the grp
    - ea grp id can have diff consumer
        - parallel processing
    - ordering across grps not guaranteed

![](https://i.imgur.com/tzFXhkj.png)



### Console
![](https://i.imgur.com/ND05RM3.png)
- 2 types of sqs queue
    - standard
    - FIFO

![](https://i.imgur.com/BMsCR4a.png)

![](https://i.imgur.com/sjL1TXl.png)
- access policy - who can access queue
    - by default, only queue owner can send/receive msg
- advanced option allows us to directly edit the json policy obj
    - or use policy generator

![](https://i.imgur.com/mJNmPjN.png)
- can set other accs, roles or iam users to send msgs into the queue

![](https://i.imgur.com/QR5CHer.png)
![](https://i.imgur.com/ULjA0zG.png)
- choose CMK (customer master key)
    - can enter default cmk
    - or choose alias if create own key
- set data key reuse - how long data key shld be used to encrypt data

![](https://i.imgur.com/ieSvvYM.png)
- top right side of sqs console to send/receive messages

![](https://i.imgur.com/XmEgMKo.png)
- can see sent msg if click on poll for msgs button at bottom

![](https://i.imgur.com/kDbgpD5.png)
- view msgs details when clicked
    - also view msg body and attributes
        - key vals can be placed in msg attrs
- the msg might also be recieved twice
    - as we didnt process it in enough time
        - after 30 seconds, the msg went back into queue and we received it again
    - if u poll again, msg received counter will go up again
    - to process a msg, you have to delete it

![](https://i.imgur.com/WiR0cAM.png)
- purging a queue deletes all msgs within

![](https://i.imgur.com/0X2lQGn.png)
- monitoring of queue

![](https://i.imgur.com/HrHQ3mV.png)

#### Message Visibility Timeout
![](https://i.imgur.com/Foq8jft.png)
- settings is in queue settings

#### DLQ
![](https://i.imgur.com/LC8TkoL.png)
- create a new standard queue
    - set retention to max (14 days)

![](https://i.imgur.com/22G8vkt.png)
- edit your __main queue__ to use the new queue as the dlq
    - also set max receives

#### Delay Queue
![](https://i.imgur.com/5yV2RYZ.png)
- create new standard queue and set delivery delay param

#### Long Polling
![](https://i.imgur.com/IjchqWj.png)
- edit receieve msg wait time in queue settings

#### FIFO Queue
![](https://i.imgur.com/xQ7jYYD.png)

- choose fifo when creating queue
    - you have to end your queue name with `.fifo` else it wont work

![](https://i.imgur.com/fgmMGRP.png)
- extra config content-based duplication
    - to deduplicate msgs if the same is sent twice in small 5min window

![](https://i.imgur.com/tITUKpJ.png)
- when sending msg, more configs present
    - msg grp id
    - msg deduplication id

![](https://i.imgur.com/IHpjEie.png)
- order of first received is read from bottom to top in the console
    - the bottom most msg is the 1st msg received


AWS SNS
---
- what if want to send 1 msg to many receivers?

![](https://i.imgur.com/9GHDDAi.png)
- create publish/subscribe system
    - client send msg to sns topic hence publishing msg into topic
    - topic will have many subscribers
- in sns (simple notif service), event producer only sends msg to 1 sns topic
    - can have many event listeners/subscribers to listen on sns topic notifs
    - ea subscriber will get all msgs in topic
        - thrs new feature to filter msgs
- up to 10,000,000 subscriptions per topic
    - 100,000 topics limits
- subscribers can be
    - sqs
    - http/https with delivery retries
    - lambda
    - emails
    - sms msgs
    - mobile notifs
- integrates with many aws services
    - cloudwatch for alarms
    - asg notifs
    - s3 bucket events
    - cloudformation when state changes (Eg. failed to build etc.)
    - more

### Publishing
- topic publish using sdk
    - create topic
    - create subscription
    - publish to topic
- direct publish for mobile apps sdk
    - create platform app
    - create platform endpt
    - publish to platform endpt
    - works with google gcm, apple apns, amazon adm etc.

### SNS Security
- encryption
    - inflight encryption using https api
    - at-rest encryption using kms keys
    - client side encryption if client wants to perform encryption/decryption itself
- access controls
    - iam policies to regulate access to sns api
- sns access policies
    - similar to s3 bucket policies
    - useful for cross acc access to sns topics
    - useful for allowing other services (Eg. s3) to write to sns topic

### SNS + SQS: Fan Out
- push once in sns, receive in all sqs queues that are subscribers
    - fully decoupled
    - no data loss
- sqs allows for 
    - data persistence
    - delayed processing
    - retries of work
- abillity to add more sqs subs over time
- make sure sqs queue access policy allows for sns to write
- sns cannot send msgs to sqs fifo queues
    - this is an aws limitation

![](https://i.imgur.com/rDbKZvS.png)


### S3 Events to Multiple Queues
- for same combination of event type (Eg. obj create) and prefix (Eg. images/), you can only have 1 s3 event rule
- if want to send same s3 event to many sqs queues, use fan-out
    - sns topic will send msg to multiple sqs queues or event lambda funcs
    - very helpful to circumvent the limitations of s3 bucket event rules

![](https://i.imgur.com/seaJKyN.png)




### Console
![](https://i.imgur.com/4cggmON.png)
![](https://i.imgur.com/RtIrYTk.png)
![](https://i.imgur.com/kAP90x6.png)
![](https://i.imgur.com/Ixg65NT.png)
![](https://i.imgur.com/yLjdLFC.png)
- many protocol to choose from
    - if choose email have to verify it

![](https://i.imgur.com/dYeVtME.png)

#### Fan Out
![](https://i.imgur.com/7mvuf1D.png)
- sqs can sub to sns topic

![](https://i.imgur.com/UCxelEU.png)
- for fifo its greyed out


AWS Kinesis
---
- kinesis is managed alternative to apacha kafka
- is basically big data streaming tool
    - great for app logs, metrics, iot, clickstreams
    - great for realtime big data
    - great for streaming processing frameworks
        - Eg. spark, nifi etc.
        - compatible with framework allowing you to perform computations in realtime on data that arrives through a stream
- data automatically replicated to 3 az
- 3 sub-kinesis products
    - kinesis streams
        - low latency streaming ingest at scale
    - kinesis analytics
        - perform realtime analytics on streams using sql
        - perform filters in realtime
    - kinesis firehose
        - load streams into s3, redshift, elastisearch etc.

![](https://i.imgur.com/ZlNy3Et.png)
- devices, streams, logs will be producing data directly into our kinesis streams
- process data in kinesis analytics
- store the analysis through firehose into s3 or redshift

### Streams Overview
- streams divided in ordered __shards/partitions__
    - if want to scale up our stream, just add shards
    - more shards = higher throughput
- data retention is 1 day by default
    - can go up to 7 days
- ability to reprocess/replay data
    - diff in sqs, once data consumed in sqs, its gone
- multiple apps can consume same stream
    - kinda like sns
- realtime processing with scale of throughput
- once data inserted into kinesis, cant be deleted
    - immutability

![](https://i.imgur.com/J2kjO5s.png)

#### Streams Shards
- 1 stream made of many diff shards
    - 1mb/s or 1000 msg per sec at write __per shard__
        - this is receiving
    - 2mb/s at read __per shard__
        - this is sending out msg
- billing per shard provisioned
    - can have as many shards as you want
- batching avail or per msg calls
    - allows to efficiently push msgs into kinesis
- num of shards can evolve over time
    - reshard - add shard
    - merging - reduce shard
    - can add some kind of auto scaling for your kinesis stream
- records are ordered per shard
    - in sqs no order
        - in sqs fifo only have 1 queue and all records going in that queue
    - kinesis is kind of in between
        - can have many shards and records ordered per shard

### Kinesis API
#### Put Records
- `PutRecord` api + partition key that gets hashed
    - same key goes to same partition
        - helps with ordering for specific key
        - just provide the key to a data pt and they'll order for you
- msgs sent gets a sequence number
    - seq num is always increasing
- choose partition key that is highly distributed
    - helps prevent hot partition
        - if key is not distributed, all data will go to 1 shard and it'll be overwhelmed (AKA hot partition)
    - eg. use user_id key if many users
        - since very distributed
    - Eg. not country_id if 90% of users are in 1 country
- use batching and putrecords to reduce costs and increase throughput
- `ProvisionedThroughputExceeded` if we go over the limits
    - for this just use retries and exponential backoff
- can use cli, aws sdk or producer libraries from various frameworks

![](https://i.imgur.com/R7UD2HO.png)


#### Exceptions
- `ProvisionedThroughputExceeded` exception
    - when sending more data exceeding mb/s or tps for any shard
    - ensure you dont have a hot shard
        - eg. when partition key is bad and too much data goes to partition
- solution
    - retries with backoff
    - inc shards (scaling)
    - ensure partition key is good one
        - well distributed

#### Consumers
- can use normal consumer
    - eg. cli, sdk etc.
- can use kinesis client library
    - eg. in java, node, python, ruby, .net
    - KCl uses dynamodb to checkpt offsets
    - kcl uses dynamodb to track other workers and share work amongst shards
    - kcl basically enables to consume from kinesis efficiently

![](https://i.imgur.com/3crutR2.png)

### Kinesis KCL in Depth
- kinesis client lib (KCL) is java lib that helps record from a kinesis stream with distributed apps sharing the read workload
    - rule - ea shard is read by only 1 kcl instance
    - eg.
        - 4 shards = max 4 kcl instances
            - 6 shards = max 6 kcl instances
- progress checkpointed into dynamodb
    - need iam access
- kcl can run on ec2, elastic beanstalk, on-premise app
- records are read in order at shard lvl

#### Example
![](https://i.imgur.com/it2pi59.png)
- ea shard consumed by 1 kcl app and ea kcl app consumes 2 shards

![](https://i.imgur.com/6MIC8xi.png)
- can scale to max (1 kcl for ea shard)

![](https://i.imgur.com/50GAmUI.png)
- if want to scale more, can do shard splitting
    - means add more shards
- can afterwards scale kcl to 6 too


### Kinesis Security
- control access/authorisation using iam policies
- encryption inflight using https endpts
    - encryption at rest using kms
- possibility to encrypt/decrypt data client side
    - though harder
- vpc endpts avail for kinesis to access within vpc

### Kinesis Data Analytics
- perform realtime analytics on kinesis streams using sql
- data anlytics
    - auto scaling
    - managed - no servers to provision
    - continuous (realtime)
        - no delay to consuming 
- pay for actual consumption rate
- can create streams out of realtime queries

### Kinesis Firehose
- fully managed service, no administration
- near realtime
    - 60 seconds latency
- load data in redshift/s3/elastisearch/splunk
- automatic scaling
- support many data formats
    - pay for conversion
- pay for amt of data going through firehose




### Console
![](https://i.imgur.com/RhOSzuk.png)
![](https://i.imgur.com/cA3hJxO.png)
![](https://i.imgur.com/ogkxObB.png)
- can go up to 200 shards before hitting acc limit

![](https://i.imgur.com/APZxXLG.png)
![](https://i.imgur.com/NqikvmV.png)
![](https://i.imgur.com/D7bwky8.png)
![](https://i.imgur.com/B00wkas.png)
- capture shard metrics

![](https://i.imgur.com/GOkcvaI.png)

![](https://i.imgur.com/gKSw30O.png)
- type `aws kinesis help` to see list of funcs
    - do `aws kinesis <func> help` for even more details to specific func
![](https://i.imgur.com/d79ChPB.png)

![](https://i.imgur.com/POHr69h.png)
- return shard id and sequence number

![](https://i.imgur.com/Xfh5RcG.png)
- use `get-shard-iterator` to get a shard iterator to place into `get-records` to get the records for a specific shard

![](https://i.imgur.com/DpIQ5rK.png)
- also shows arrival timestamp
    - data is base64 encoded


SQS vs SNS vs Kinesis
---
![](https://i.imgur.com/oSV5weM.png)

### Data Ordering in Kinesis
![](https://i.imgur.com/UdF43eL.png)
- as long as always specify same key, data will continue to flow into the specified shard the key is allocated to


### Data Ordering in SQS
- for sqs standard, no ordering
- for sqs fifo, u dont use grp id but msgs are consumed in order they are sent with only 1 consumer
- if want to scale num of consumers but want msgs to be grped when they are related to ea other, use grp id
    - similar to partition key in kinesis

![](https://i.imgur.com/GPWc7Ci.png)
![](https://i.imgur.com/i5fqjLs.png)

### Kinesis vs SQS Ordering
- assume 100 trucks, 5 kinesis shards and 1 sqs fifo
- kinesis data streams
    - on avg you'll have 20 trucks per shard
    - trucks will have their data stored within ea shard
    - max amt of consumers in parallel can have is 5
    - can receive up to 5mb/s of data
- sqs fifo
    - can only have 1 sqs fifo queue
    - will have 100 grp id
    - can have up to 100 consumers due to 100 grp id
    - can have up to 300 msgs per second
        - or 3000 is using batching
- NOTE
    - sometimes better to use fifo if have dynamic num of consumers
    - sometimes btr to use kinesis data streams if have a lot of data and also want ordering per shard


Quiz
---
![](https://i.imgur.com/DVEbY4f.png)
![](https://i.imgur.com/bxmGUqN.png)




###### tags: `AWS Developer Associate` `Notes`