

04 AWS Fundamentals P3: RDS/Aurora/ElastiCache
===




## Table of Contents

- [04 AWS Fundamentals P3: RDS/Aurora/ElastiCache](#04-aws-fundamentals-p3--rds-aurora-elasticache)
  * [Table of Contents](#table-of-contents)
  * [Regional Database Service (RDS)](#regional-database-service--rds-)
    + [Pros of RDS vs DB on EC2](#pros-of-rds-vs-db-on-ec2)
    + [RDS Backups](#rds-backups)
    + [RDS Read Replicas for Read Scalability](#rds-read-replicas-for-read-scalability)
      - [Use Cases](#use-cases)
      - [Network Costs](#network-costs)
    + [RDS Multi-AZ (Disaster Recovery)](#rds-multi-az--disaster-recovery-)
    + [RDS Security - Encryption](#rds-security---encryption)
      - [Encryption Operations](#encryption-operations)
    + [RDS Security - Network & IAM](#rds-security---network---iam)
      - [IAM Authentication](#iam-authentication)
      - [RDS Security Summary](#rds-security-summary)
    + [Console](#console)
      - [Security](#security)
      - [SQLectron](#sqlectron)
  * [Amazon Aurora](#amazon-aurora)
    + [Aurora High Availability and Read Scaling](#aurora-high-availability-and-read-scaling)
    + [Aurora DB Cluster](#aurora-db-cluster)
    + [Features of Aurora](#features-of-aurora)
    + [Aurora Security](#aurora-security)
    + [Aurora Serverless](#aurora-serverless)
    + [Global Aurora](#global-aurora)
    + [Console](#console-1)
      - [Auto Scaling](#auto-scaling)
  * [AWS ElastiCache](#aws-elasticache)
    + [ElastiCache Solution Architecture - DB Cache](#elasticache-solution-architecture---db-cache)
    + [ElastiCache Solution Architecture - User Session Store](#elasticache-solution-architecture---user-session-store)
    + [Redis VS Memcached](#redis-vs-memcached)
    + [Caching Implementation Considerations](#caching-implementation-considerations)
      - [Lazy Loading/Cache-Aside/Lazy Population](#lazy-loading-cache-aside-lazy-population)
      - [WriteThrough](#writethrough)
      - [Cache Evictions & Time-to-Live (TTL)](#cache-evictions---time-to-live--ttl-)
      - [Extra Considerations](#extra-considerations)
    + [Console](#console-2)


Regional Database Service (RDS)
---
- rds - managed db service for db use sql as query lang
- allows you to create db in cloud managed by aws
    - postgres
    - mysql
    - mariadb
    - oracle
    - microsoft sql server
    - aurora (aws proprietary db)

### Pros of RDS vs DB on EC2
- rds is managed service
    - automated provisioning, OS patching
    - continuous backups & restore to specific timestamp
        - AKA point in time restore
    - monitoring dashboards
    - read replicas for improved read perf
    - multi az setup for disaster recovery (DR)
    - maintenance windows for upgrades
    - scaling capacity
        - vertical/hori
    - storage backed by ebs
        - gp2 or io1
- BUT cant ssh into rds instances
    - cuz managed service so dont have access to underlying ec2 instance 
- NOTE
    - multi az uses same conn string regardless which db
        - read replicas need to reference indiv as ea has its own dns name

### RDS Backups
- backups are automatically enabled in rds
- automated backups
    - daily full backup of db
        - during maintenance window that u define
    - daily transaction logs backed-up by rds every 5mins
    - daily full backup + transaction logs give ability to restore to any pt in time
        - Eg. from oldest backup to 5 mins ago
    - 7 days retention
        - can be inc to 35 days
- db snapshots
    - backups manually triggered by user
    - retention of backup for as long as you want

### RDS Read Replicas for Read Scalability
- read replicas
    - help to scale your reads
        - main instance receives too many reqs
- up to 5 read replicas
- within az, cross az or cross region
- replication is __async__ so read eventually consistent
- replicas can be promoted to their own db
    - out of replication process after that
- apps must update conn string to leverage read replicas

![](https://i.imgur.com/JzVt9tq.png)

#### Use Cases
- have production db that's taking normal load
- want to run reporting app to run analytics
    - will overload/slow down main rds instance if you do analytics thr
- create read replica to run new workload thr
    - async replication between main and replica
- production app is unaffected
- read replicas used for SELECT only kind of statements
    - read only
    - cannot INSERT, UPDATE or DELETE
        - cannot alter db

![](https://i.imgur.com/nqNagc2.png)


#### Network Costs
- in aws there's network cost when data goes between az
- to reduce cost, have read replicas in same az

![](https://i.imgur.com/VH8H72o.png)


### RDS Multi-AZ (Disaster Recovery)
- SYNC replication
    - not async
- 1 dns name - automatic app failover to standby
    - inc availability
    - auto failover to standby db in case of loss of az/loss of network/instance or storage failure
        - standby db become new master
- no manual intervention in apps
- not used for scaling
    - is just for standby/failover
    - nobody can read/write to it
- NOTE
    - read replicas can be setup as multi az for disaster recovery (DR)
        - common exam qns

![](https://i.imgur.com/0Kx0YoO.png)

### RDS Security - Encryption
- at rest encryption
    - possibility to encrypt master & read replicas with AWS KMS - AES-256 encryption
    - encryption has to be defined at launch time
    - if master not encyrpted, read replicas cannot be encrypted
        - common exam qns
    - __transparent data encryption (TDE)__ available for oracle & sql server
- in-flight encryption
    - ssl certs to encrypt data to rds in flight
        - during traffic
    - provide ssl options with trust certificate when connecting to db
    - to enforce ssl (make sure all clients must use ssl)
        - postgresql
            - `rds.force_ssl=1` in aws rds console (params grps)
            - this is parameter grp
        - mysql
            - `GRANT USAGE ON *.* TO '.ysqluser'@'%' REQUIRE SSL` in db itself

#### Encryption Operations
- encrypting rds backups
    - snapshots of unencrypted rds db are unencrypted
        - for encrypted is encrypted
    - can copy unencrypted snapshot into encrypted one
        - when u copy it, can create encrypted ver of that snapshot
- to encrypt un-encrypted rds db
    - create snapshot of unencrypted db
    - copy snapshot & enable encryption for snapshot
        - now have copied encrypted snapshot
    - restore db from encrypted snapshot
    - migrate apps to new db & delete old db

### RDS Security - Network & IAM
- network security
    - rds db usually deployed within priv subnet, not in public one
        - dont expose db to internet
    - rds security works by leveraging security grps
        - same concept as ec2 instances
        - it controls which ip/sec grp can comm with rds
- access/user management
    - IAM policies help control who can manage aws rds 
        - through rds api
    - traditional username & password can be used to login into db
    - IAM-based auth can be used to login into rds mysql & postgresql
        - just these 2 only
- NOTE
    - db security is usually from within the db

#### IAM Authentication
- IAM db auth works with mysql and postgresql
- dont need password, just auth token from IAM & RDS API calls
    - auth token has lifetime of 15mins
- benefits
    - network in/out must be encrypted using ssl
    - IAM centrally manage users instead of db
    - can leverage IAM roles & ec2 instance profiles for easy integration

![](https://i.imgur.com/9oSWRR2.png)

#### RDS Security Summary
![](https://i.imgur.com/QRRKSdF.png)



### Console
![](https://i.imgur.com/Es6ME35.png)
![](https://i.imgur.com/GZyaKoH.png)
- aurora is new aws db
    - dont work with amazon free tier

![](https://i.imgur.com/5IiRSbd.png)

- also choose instance config for rds instance
    - like ec2

![](https://i.imgur.com/iuLgUi3.png)

![](https://i.imgur.com/5FTJ9OB.png)
- can create read replica here

#### Security
![](https://i.imgur.com/zmxbTrb.png)
- dont make db publicly accessible
- also have to choose vpc security grp
    - can also login through iam roles

![](https://i.imgur.com/htk3LVE.png)
- deletion protection
    - cannot delete db without first removing this protection

#### SQLectron
- GUI for connecting to db
    - have many db types

![](https://i.imgur.com/pxo0wmF.png)
- details to connect remotely to rds db

![](https://i.imgur.com/P3ZdboY.png)
- enable ssl to have secure conn


Amazon Aurora
---
- is proprietary tech from aws
    - not open sourced
- postgres and mysql both supported as aurora db
    - means drivers work as if aurora is posthres/mysql db
- is aws cloud optimised
    - claims 5x perf improvement over mysql on rds
    - over 3x perf of postgres on rds
    - have more perf improvements in other ways
- aurora storage auto grows in increments of 10gb
    - up to 64gb
    - dont need to worry about monitoring your disks
- can have 15 replicas while mysql has 5
    - replication process is faster
        - sub 10ms replica lag
- failover in aurora is instantaneous
    - is high availability (HA) native
- costs more than rds 
    - 20% more
    - but more efficient

### Aurora High Availability and Read Scaling
- 6 copies of data across 3 az
    - only need 4/6 copies needed for writes
        - if 1 AZ down you're fine
    - 3/6 copies need for reads
        - highly avail for reads
    - self healing with peer-to-peer replication
        - if some data corrupted/bad can self healing
    - storage stripped across 100s of vols
- 1 aurora instance takes writes (master)
    - automated failover for master in less than 30 secs if master doesnt work
- up to 15 aurora read replicas serve reads
    - any of these read replicas can be master if master fails
- support for cross region replication
    - for read replicas

![](https://i.imgur.com/xeciqii.png)

### Aurora DB Cluster
![](https://i.imgur.com/0DRC14x.png)
- shared storage vol
    - only master writes to this
    - since master can change and failover, aurora provides a __writer endpoint__
        - dns name pointing to master
        - redirects to failover if any
- auto scaling of read replicas 
    - so always have right num of read replicas
- __reader endpoint__
    - helps with conn load balancing
        - lb happens at conn lvl not statement lvl
    - connects automatically to all read replicas
- NOTE
    - impt to understand this diagram to understand how aurora works

### Features of Aurora
- automatic failover
- backup and recovery
- isolation and security
- industry compliance
- push-button scaling
- automated patching with 0 downtime
- advanced monitoring
- routine maintenance
- backtrack
    - restore data at any pt of time w/o backups
    - actually doesnt rely on backups but sth else

### Aurora Security
- similar to rds as use same engine
- encryption at rest using KMS
- automated backups, snapshots and replicas also encrypted
- encryption in flight using ssl
    - same process as mysql or postgres if want to enforce it
- possibility to auth using IAM token
    - same method as rds
- you're responsible for protecting instance with security grps
- cannot ssh
- NOTE
    - pretty similar to rds security
    - postgres does not support transparent data encryption (TDE) on top of KMS

### Aurora Serverless
- automated db instantiation and auto-scaling based on actual usage
    - good for infrequent/intermittent/unpredictable workloads
- no capacity planning needed
- pay per sec
    - can be more cost effective

![](https://i.imgur.com/G5eygoF.png)
- proxy fleet client will connect to 
- if more load, more aws aurora db will be created automatically
    - can auto scale down too
    - 0 db if no usage
    - no server as no scaling to do 
        - no capacity planning
        - will scale based on demand



### Global Aurora
- aurora cross region read replicas
    - useful for disaster recovery
    - simple to put in place
        - just create read replica in another region
- aurora global db (recommended)
    - 1 pri region
        - read/write
    - up to 5 secondary regions
        - read only
        - replication lag less than 1sec
    - up to 16 read replicas per secondary region
    - helps for decreasing latency
    - promoting another region (for disaster recovery) has recovery time objective (RTO) of <1 min
        - rto - max time after outage that company is willing to wait for recovery process to fin
        - in <1 min, sec db in another region will become pri and ready to take on writes if pri fails

![](https://i.imgur.com/QiLhmB8.png)


### Console
![](https://i.imgur.com/7g0xJvl.png)
- aurora not free tier
- select aurora when creating new rds db
    - choose with postgres/mysql compatibility

![](https://i.imgur.com/sTZjch5.png)
- choose db features
    - 1 writer multiple reader
        - for general purpose
    - parallel query to analyse/improve perf of analytics queries
    - multiple writers
        - huge amt of writing going on
    - serverless
        - unpredictable workload
        - needs to be more scalable
- NOTE
    - exam need know general and serverless one

![](https://i.imgur.com/YqYjyby.png)
- choose db instance size
    - whr u choose perf of db
    - 2 choices
        - memory optimised classes 
            - r and x classes
            - better for production type workload
        - burstable classes 
            - t classes
            - cheaper
            - better for dev/test

![](https://i.imgur.com/5bmn4WQ.png)

![](https://i.imgur.com/dJ07PdM.png)

![](https://i.imgur.com/nshl42x.png)

- NOTE
    - exam impt stuff
        - multi az
        - the 2 main db features

![](https://i.imgur.com/QOnXqWr.png)
- writer and reader endpt
    - reader endpt has `-ro` which means read only 
    - recommended use this endpt but u can still get specific db endpt

![](https://i.imgur.com/iHpjYTY.png)

#### Auto Scaling
![](https://i.imgur.com/StT6NwQ.png)
- similar to ASGs


AWS ElastiCache
---
- same way rds is to get managed regional db, elasticache is to get managed redis or memcached
- caches are in-memory db with really high perf, low latency
    - helps reduce load off db for read intensive workloads
        - read for cache instead of db
    - helps make app stateless
        - by storing states in common cache
- write scaling using sharding
    - read scaling using read replicas
- multi az with failover capability
    - like rds
- aws takes care of
    - OS maintenance/patching
    - optimisations
    - setup
    - configuration
    - monitoring
    - failure recovery
    - backups
- using elasticache involves heavy app code changes
- NOTE
    - basically rds for caches

### ElastiCache Solution Architecture - DB Cache
- apps first queries elasticache
    - if not avail, get from rds and store in elasticache
- helps relieve load in rds
- cache must have invalidation strategy to make sure only most current data is used in thr

![](https://i.imgur.com/tyx5nzx.png)
- cache hit - get into elasticache and it works
    - retrieval super quick and rds dont see a thing
- cache miss - request data and doesnt exist
    - need to query rds db and get ans
    - app shld be programmed to write back to cache/results in elasticache

### ElastiCache Solution Architecture - User Session Store
- user logs into any of the app
- app writes session data into elasticache
- user hits another instance of our app
- instance retrieves data and user alr logged in
    - all instances can retrieve this data so user dont have to reauth everytime

![](https://i.imgur.com/p95oIo8.png)
- all the apps need to know that user is logged in
    - use elasticache to share states like user session store so all apps is stateless
        - retrieve and write these sessions in realtime

### Redis VS Memcached
- 2 types of elasticache clusters
- redis
    - multi az with auto-failover
    - read replicas to scale reads and high availability
    - data durability using append only files (AOF) persistence
        - even if cache stopped and restarted, can still have data that was in cache before stopping
    - backup and restore features
    - NOTE
        - think of 2 instances, 1 pri and 1 sec 
            - persistence
            - backup
            - restore
        - similar to rds
- memcached
    - multi-node for partitioning of data
        - AKA sharding
    - non persistent cache
        - if memcached node goes down, data is lost
    - no backup and restore
    - multi-threaded architecture
    - NOTE
        - part of cache in 1st shard, other part in 2nd shard
            - ea shard is memcached node
        - is pure cache living in memory
            - very diff from rds

![](https://i.imgur.com/mCVYTd4.png)


### Caching Implementation Considerations
- read more
    - https://aws.amazon.com/caching/implementation-considerations/
- isit safe to cache data?
    - data may be outdated, eventually consistent
    - is not for every type of dataset
- is caching effective for data?
    - pattern (great for caching)
        - data changing slowly
        - few keys frequently needed
    - anti patterns
        - data changing rapidly
        - all large key space freq needed
- is data structured for caching?
    - Eg. key value caching or caching of aggregations results (great for caching)
    - make sure data structured so save time
- which caching design pattern most appropriate??
    - see below for patterns

#### Lazy Loading/Cache-Aside/Lazy Population
- pros
    - only requested data cached
        - cache isnt filled with unused data
    - node failures not fatal
        - just inc latency to warm cache
            - warm - all reads has to go to rds then cache
- cons
    - cache miss penalty that results in 3 round trips
        - 3 network calls done
            - from app to elasticache
            - app to rds to read from db
            - write to cache
        - noticeable delay for that request
            - may be bad user experience
    - stale data
        - data can be updated in db & outdated in cache
            - is my data ok to be outdated and eventually consistent?

![](https://i.imgur.com/P0ak5bA.png)
- cache miss

__Python Pseudocode__
![](https://i.imgur.com/Zop1CP7.png)


#### WriteThrough
- add/update cache when db updated
- pros
    - data in cache nvr stale
        - when change in rds, change in cache (data always updated)
    - reads quick
    - write penalty VS read penalty
        - ea write requires 2 calls
        - longer writes but read quicker
- cons
    - missing data until its added/updated in db
        - mitigation - implement lazy loading strategy
            - if cache miss, also do lazy loading into rds
    - cache churn
        - a lot of data will nvr be read

![](https://i.imgur.com/x9yFqnq.png)


__Python Pseudocode__
![](https://i.imgur.com/qe9Jqmw.png)
- used tgt with `get_user` func from lazy loading


#### Cache Evictions & Time-to-Live (TTL)
- 3 ways for cache eviction
    - delete item explicitly in cache
    - item evicted as memory full and not recently used (LRU - least recently used)
    - set item TTL
        - after time limit in ttl, data is evicted
- ttl helpful for any kind of data
    - leaderboards
    - comments
    - activity streams
- ttl can range from few secs to hours to days
- if too many evictions happen due to memory, shld scale up/out
    - cache always full mem


#### Extra Considerations
- lazy loading/cache aside easy to implement & work for many situations as a foundation
    - especially on read side
- write-through usually combined with lazy loading as targeted for queries/workloads that benefit from this optimisation
    - more of optimisation on top of lazy loading
        - not usually a strat on its own
- setting ttl is usually not bad idea except when using write-through
    - set to sensible value for app
- only cache data that makes sense
    - Eg. user profiles, blogs etc.


### Console
![](https://i.imgur.com/xhcwHO9.png)

![](https://i.imgur.com/QMa9uqB.png)
- choose encryption
- Redis Auth
    - set token to wtv u want
    - token needed for apps to connect to redis
    - only used for encryption in transit

![](https://i.imgur.com/AFgfg7z.png)
- backups is redis only feature
    - memcached dh


###### tags: `AWS Developer Associate` `Notes`