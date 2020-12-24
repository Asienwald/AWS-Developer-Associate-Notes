

09 Advanced S3 & Athena
===



## Table of Contents

- [09 Advanced S3 & Athena](#09-advanced-s3---athena)
  * [Table of Contents](#table-of-contents)
  * [More S3](#more-s3)
    + [S3 MFA-Delete](#s3-mfa-delete)
    + [S3 Default Encryption VS Bucket Policies](#s3-default-encryption-vs-bucket-policies)
    + [S3 Access Logs](#s3-access-logs)
      - [Access Logs: Warning](#access-logs--warning)
    + [S3 Replication (CRR & SRR)](#s3-replication--crr---srr-)
    + [Pre-Signed URLs](#pre-signed-urls)
      - [Examples](#examples)
    + [S3 Storage Classes](#s3-storage-classes)
      - [S3 Standard - General Purpose](#s3-standard---general-purpose)
      - [S3 Standard - Infrequent Access (IA)](#s3-standard---infrequent-access--ia-)
      - [S3 One Zone - Infrequent Access (IA)](#s3-one-zone---infrequent-access--ia-)
      - [S3 Intelligent Tiering](#s3-intelligent-tiering)
      - [Amazon Glacier](#amazon-glacier)
      - [Amazon Glacier & Glacier Deep Archive](#amazon-glacier---glacier-deep-archive)
      - [S3 Classes Comparison](#s3-classes-comparison)
    + [S3 Lifecycle Policies](#s3-lifecycle-policies)
      - [Moving Between Storage Classes](#moving-between-storage-classes)
      - [S3 Lifecycle Rules](#s3-lifecycle-rules)
      - [Scenario Questions](#scenario-questions)
    + [S3 Baseline Performance](#s3-baseline-performance)
      - [S3 KMS Limitation](#s3-kms-limitation)
    + [S3 Performance Optimisations](#s3-performance-optimisations)
      - [S3 Performance - S3 Byte-Range Fetches](#s3-performance---s3-byte-range-fetches)
    + [S3 Select and Glacier Select](#s3-select-and-glacier-select)
    + [S3 Event Notifications](#s3-event-notifications)
    + [S3 Object Lock & Glacier Vault Lock](#s3-object-lock---glacier-vault-lock)
    + [Console/CLI](#console-cli)
      - [Default Encryption](#default-encryption)
      - [S3 Access Logs](#s3-access-logs-1)
      - [Replication](#replication)
      - [Generate Pre-signed URL](#generate-pre-signed-url)
      - [Storage Classes](#storage-classes)
      - [Lifecycle Rules](#lifecycle-rules)
      - [Event Notifications](#event-notifications)
  * [AWS Athena](#aws-athena)
    + [Console](#console)
  * [Quiz](#quiz)



More S3
---
### S3 MFA-Delete
- mfa forces user to generate code on device before doing impt operations on s3
- to use mfa, enable versioning on s3 bucket
    - will need mfa to
        - permenantly delete obj version
        - suspend versioning on bucket
    - wont need mfa to
        - enable versioning
        - list deleted versions
- only bucket owner (root acc) can enable/disable mfa-delete
    - not even admin acc can enable/disable 
- mfa-delete currently can only be enabled using cli
    - very hard to setup ?
    - no console for this

### S3 Default Encryption VS Bucket Policies
- old way to enable default encryption is to use bucket policy & refuse any http command w/o proper headers

![](https://i.imgur.com/q487q5V.png)
- on left, deny any req with no header saying to use AES256
    - on right, deny anything that is unencrypted
- new way is to use default encryption option in s3
- NOTE
    - bucket policies evaluated before default encryption


### S3 Access Logs
- for audit purposes, want to log all access to s3 buckets
    - any req made to s3 from any acc, authorised or denied logged into another s3 bucket
    - data can be analysed with data analysis tools
        - OR amazon athena (see later)
- log format is at https://docs.aws.amazon.com/AmazonS3/latest/dev/LogFormat.html

![](https://i.imgur.com/Cn5Astq.png)


#### Access Logs: Warning
- do not set logging bucket to be the monitored bucket
    - it will create logging loop and bucket will grow in size exponentially

![](https://i.imgur.com/nNG8xQ9.png)


### S3 Replication (CRR & SRR)
- must enable versioning in src and dst
    - cross region replication (CRR)
        - 2 buckets in diff region
    - same region replication (SRR)
        - 2 buckets in same region
- buckets can be in diff accs
- copying is async
    - is very quick
- must give proper IAM perms to s3
    - need to create iam role for copying
- use cases
    - CRR
        - compliance
        - lower latency access
        - replication across accs
    - SRR
        - log aggregation
            - diff logging buckets and want to centralise them into one bucket
        - live replication between production and test accs
- NOTE
    - after activating, only new objs are replicated
        - not retroactive - wont copy existing states of s3 bucket
    - for DELETE operations
        - if delete w/o version id, it adds delete marker
            - not replicated
        - if delete with ver id, it deletes in the source
            - not replicated
        - basically any delete op is __not replicated__
    - there is no chaining of replication
        - if bucket 1 has replication into bucket 2 which has replication into bucket 3 then objs created in bucket 1 not replicated into bucket 3

![](https://i.imgur.com/R57r5Qx.png)

### Pre-Signed URLs
- can generate pre-signed urls using SDK or cli
    - for downloads
        - easy, can use cli
    - for uploads
        - harder, must use sdk
- valid for default of 3600 secs (1h)
    - can change timeout with `--expires-in <time by seconds>` arg
- users given pre-signed url inherit perms of person who generated url for GET/PUT

#### Examples
- only allow logged-in users to download premium vid on s3 bucket
- allow everchanging list of users to download files by generating urls dynamically
- allow temp a user to upload file to precise location on bucket


### S3 Storage Classes
- amazon s3 standard - general purpose
- amazon s3 standard-infrequent access (IA)
    - when files infrequently accessed
- amazon s3 one zone-infrequent access
    - can recreate data
- amazon s3 intelligent tiering
    - move data between storage classes intelligently
- amazon glacier
    - for archives
- amazon glacier deep archive
    - for archives u dont need right away
- amazon s3 reduced redundancy storage
    - deprecated - omitted

#### S3 Standard - General Purpose
- high durability
    - 99.999% (11 9s) of objs across multiple AZ
    - if store 10,000,000 objs in s3, can on average expect to incur loss of a single obj once every 10,000 years
- 99.99% availability over a given year
- sustain 2 concurrent facility failures
    - very resistant to facility failures
- use cases
    - big data analytics
    - mobile & gaming apps
    - content distributions


#### S3 Standard - Infrequent Access (IA)
- suitable for data that is less frequently accessed but requires rapid access when needed
- high durability
    - 99.99% of objs across multiple az
        - though 1 9 less durability than standard
- 99.99% availability
- low cost compared to amazon s3 standard
- sustain 2 concurrent facility failures
- use cases
    - data store for disaster recovery
    - backups
    - any files you expect to access less frequently


#### S3 One Zone - Infrequent Access (IA)
- same as IA but data stored in single AZ
    - other than that is same as IA above
- high durability (99.99%) of objs in single az
    - same durability as standard IA
    - data lost when az destroyed
- 99.5% availability
- low latency & high throughput perf
- supports SSL for data at transit and encryption at rest
- low cost compared to IA
    - by 20%
- use cases
    - storing secondary backup copies of on-premise data
    - storing data you can recreate
        - Eg. recreate thumbnails from an img
        - img on s3 general purpose, thumbnail on one zone IA

#### S3 Intelligent Tiering
- same low latency & high throughput perf of s3 standard
- small monthly monitoring & auto-tiering fee
    - automatically moves objs between 2 access tiers based on changing access patterns
- designed for durability of 99.999$ of objs across multiple az
    - 11 9s, same as standard
- resilient against events that impact an entire az
    - designed for 99.9% avail over given year


#### Amazon Glacier
- low cost obj storage meant for achiving/backup
- data retained for longer term
    - 10s of years
- alternative to on-premise magnetic tape storage
    - whr u would store data on magnetic tapes & put tapes away
- avrg annual durability is 99.9999%
    - 11 9s, same as standard
- cost per storage per month ($0.004/gb) + retrieval cost
    - very low
- ea item in glacier is called __archive__
    - archive is file up to 40gb
    - archives stored in __vaults__

#### Amazon Glacier & Glacier Deep Archive
- amazon glacier - 3 retrieval options
    - expedited
        - 1-5 mins
        - a lot more expensive than other options
    - standard
        - 3-5 hours
    - bulk - multiple files
        - 5-12 hours
    - min storage duration of 90 days
- amazon glacier deep archive
    - for super long term storage
    - even cheaper
    - retrieval options
        - standard
            - 12 hours
        - bulk
            - 48 hours
    - min storage duration of 180 days

#### S3 Classes Comparison
![](https://i.imgur.com/Ptp71x3.png)
- https://aws.amazon.com/s3/storage-classes/
- SLA - is what amazon will guarantee you to reimburse you
    - not sth to know abt

![](https://i.imgur.com/rV4hgcF.png)
- price comparison Eg. us-east-2

### S3 Lifecycle Policies
#### Moving Between Storage Classes
- can transition objs between storage classes
- for infrequently accessed obj, move them to `STANDARD_IA`
    - for archive objs u dont need in real-time, `GLACIER` or `DDEP_ARCHIVE`
- moving objs can be automated using __lifecycle configuration__
    - need to know how to config in exam

![](https://i.imgur.com/k8MudUg.png)

#### S3 Lifecycle Rules
- transition actions - defines when objs are transitioned to another storage class
    - move objs to standard IA class 60 days after creation
    - move to glacier for achiving after 6 months
- expiration actions - config objs to expire (delete) after some time
    - access log files can be set to delete after 365 days
    - can be used to delete old versions of files
        - if versioning is enabled
    - can be used to delete incomplete multi-part uploads
- rules can be created for certain prefix
    - Eg. `s://mybucket/mp3/*`
- rules can be created for certain obj tags
    - Eg. Department: Finance

#### Scenario Questions
![](https://i.imgur.com/5w07V3W.png)

![](https://i.imgur.com/LLWKZSZ.png)
- noncurrent versions unlikely to be accessed
- transition after 15days


### S3 Baseline Performance
- amazon s3 auto scales to high request rates
    - latency 100-200ms
        - very fast
    - app can achieve 3,500 PUT/COPY/POST/DELETE and 5,500 GET/HEAD requests per second per prefix in bucket
- no limits to num of prefixes in bucket
- Eg. object path => prefix
![](https://i.imgur.com/R5nk318.png)
- prefix is anything between bucket and file
    - if spread reads across all 4 prefixes evenly, can achieve 22,000 (5500 x 4) reqs per second for GET and HEAD

#### S3 KMS Limitation
- if use SSE-KMS, may be impacted by KMS limits
- when you upload, it calls the `GenerateDataKey` KMS api
    - when you download, it calls the `Decrypt` kms api
- the 2 reqs above will count towards the KMS quota per second
    - 5500, 10000, 30000 req/s based on region
    - as of today, cannot request a quota inc for kms

![](https://i.imgur.com/KexRoXD.png)


### S3 Performance Optimisations
- multi-part upload
    - recommended for files > 100mb
        - must use for files > 5gb
    - can help parallelise uploads
        - speed up transfers
    - big file divided into smaller parts and uploaded parallel-ly

![](https://i.imgur.com/7Xaqapj.png)

- s3 transfer acceleration
    - upload only
        - not for download
    - inc transfer speed by transferring file to aws edge location which will forward data to s3 bucket in target region
        - aws has more edge locations than regions (like 200 and growing)
    - compatible with multi-part upload
    - upload to edge location through pub internet
        - edge location transfer to bucket through vpc (priv aws)
        - priv aws network v fast hence inc transfer speed

![](https://i.imgur.com/Y3h8spe.png)

#### S3 Performance - S3 Byte-Range Fetches
- to read file in most efficient way
    - above is all for uploading
- parallise GETs by requesting specific byte ranges
- better resilience in case of failures
    - if fail to get specific byte range, can retry smaller byte range hence better resilience
- can be used to speed up downloads
    - cuz all diff ranges can be requested in parallel
- can be used to retrieve only partial data
    - Eg. head of file

![](https://i.imgur.com/eaxKP0W.png)

### S3 Select and Glacier Select
- retrieve less data using sql by performing __server side filtering__
    - can filter by rows & cols
        - very simple sql statements
        - cannot do complicated sql statements
            - use amazon athena
- less network transfer, less cpu cost client-side
    - s3 will perform filtering then return what you need
- 400% faster, 80% cheaper as filtering is on server side
- https://aws.amazon.com/blogs/aws/s3-glacier-select/

![](https://i.imgur.com/vmfkigz.png)


### S3 Event Notifications
- react to actions like `S3:ObjectCreated`, `S3:ObjectRemoved`, `S3:ObjectRestore`, `S3:Replication` etc.. through event notifs
    - obj name filtering possible
        - Eg. *.jpg
- use case
    - generate thumbnails of imgs uploaded to s3
- can create as many s3 events as desired
- s3 event notifs typically deliver events in seconds but sometimes take min or longer
    - if 2 writes made to single non-versioned obj at same time, possible that only single event notif sent
    - if want to ensure event notif is sent for every successful write, can enable versioning for bucket

![](https://i.imgur.com/uj2r6sg.png)
- possible targets for event notifs
    - sns (simple notif service)
        - send notif in emails
    - sqs (simple queue service)
        - add msgs into a queue
    - lambda functions
        - generate custom code

### S3 Object Lock & Glacier Vault Lock
- s3 obj lock
    - adopt WORM (write once read many) model
    - blk an obj ver deletion for specified amt of time
        - after writing once, nobody can touch/modify the file
- glacier vault lock
    - adopt WORM model
    - lock policy prevents future edits to file
        - can no longer be changed
        - the policy is set in stone - once u set it, nobody can delete the policy
    - helpful for compliance & data retention

![](https://i.imgur.com/pLAKgzN.png)



### Console/CLI
![](https://i.imgur.com/uzAu2Rg.png)
- config mfa device in root acc
    - mfa delete need use root access key
        - not recommended elsewhere (do not generate or keep)

![](https://i.imgur.com/FdQcQad.png)

- `aws s3api put-bucket-versioning --bucket <bucket name> --versioning-configuration Status=Enabled,MFADelete=Enabled --mfa <arn of mfa device> <mfa code> --profile <profile name>`

![](https://i.imgur.com/rdwOVgC.png)
- now u cant do some stuff with mfadelete on
- NOTE
    - if not using mfadelete pls delete root acc credentials so nobody can use
        - VERY IMPT

#### Default Encryption
![](https://i.imgur.com/QtRRCSk.png)
- config when creating bucket
    - or in bucket properties

![](https://i.imgur.com/lzulVsN.png)
- bucket policy evaluated first


#### S3 Access Logs
![](https://i.imgur.com/qEMKBgx.png)
- choose bucket to log to (target)
    - the bucket might take hours to log and update lol

![](https://i.imgur.com/Xgm3KAd.png)
- example log
    - can use athena to analyse these files

#### Replication
- need to enable versioning for replication

![](https://i.imgur.com/8HwsTY5.png)
- manage tab of bucket

![](https://i.imgur.com/cvP56IN.png)
![](https://i.imgur.com/1m3jR5p.png)

![](https://i.imgur.com/eddUhNz.png)
- CRR iam role auto created

#### Generate Pre-signed URL
- use CLI, if use console, is not presigned
    - will give access denied

![](https://i.imgur.com/ugfHyFh.png)
- before presigning, need to config aws cli to generate sig version called __s3v4__
    - `aws configure set default.signature_version s3v4`
    - allow generated url to be compatible with KMS encrypted obj

![](https://i.imgur.com/BpOcZkC.png)
- give url of obj
- set expires in
- set region


#### Storage Classes
![](https://i.imgur.com/8KOmhxA.png)
- chosen when uploading files

![](https://i.imgur.com/vszxMov.png)
- can change storage classes of obj from obj properties too


#### Lifecycle Rules
![](https://i.imgur.com/enXSWbL.png)
- can create more than 1 lifecycle rule per bucket based on prefixes/filters

![](https://i.imgur.com/9yppRfd.png)
![](https://i.imgur.com/wayB9qC.png)
- can transition current or noncurrent versions
    - can select both too

![](https://i.imgur.com/50ODJyb.png)
![](https://i.imgur.com/ue17JhX.png)


#### Event Notifications
![](https://i.imgur.com/UvzRtFN.png)
- in bucket properties

![](https://i.imgur.com/j646QjZ.png)
![](https://i.imgur.com/Nwed9Lt.png)
- need provide arn
    - sqs will be explained later

![](https://i.imgur.com/QFCbd4i.png)
- these error means that need to set correct permissions in your sqs queue

![](https://i.imgur.com/ffIzhCu.png)
- select sendmessage action

![](https://i.imgur.com/xiwRImq.png)
![](https://i.imgur.com/82TrM4O.png)
- msgs queued when obj created


AWS Athena
---
- serverless service to perform analytics directly against s3 files
    - uses sql lang to query files
    - usually have to load file in s3 to database which has redshift & do queries thr or sth
- has JDBC (java db connectivity)/ODBC (open db connectivity) driver
    - if want to connect your BI (business int) tools to it
    - JDBC - API for java which defines how client may access a db
    - ODBC - API for accessing db management systems (DBMS)
- charged per query & amt of data scanned
- supports CSV, JSON, ORC, Avro and Parquet (built on Presto)
    - backend runs presto (is query engine)
- use cases
    - business intelligence/analytics/reporting
    - analyse and query VPC flow logs
    - ELB logs
    - cloudtrial trials
    - s3 excess logs
    - cloudfront logs
    - etc.
- exam tip
    - analyse data directly on s3 = use athena
- NOTE
    - just google sample sql queries for athena for diff service logs on google to save braincells

### Console
![](https://i.imgur.com/V4Z66e3.png)
- set query on a bucket (prob the one with the access logs)
- need to setup query result location in amazon s3 before start 1st query

![](https://i.imgur.com/9OHz2yf.png)

- copy athena sql queries from this link to analyse s3 access logs
    - https://aws.amazon.com/premiumsupport/knowledge-center/analyze-logs-athena/

![](https://i.imgur.com/riynqBG.png)
- just need to change LOCATION to your bucket to run query on

![](https://i.imgur.com/69tSOPP.png)
- can preview table by right clicking it
    - or running the sql query

![](https://i.imgur.com/1rFzreb.png)
- example queries u can do to analyse the data



Quiz
---
![](https://i.imgur.com/phcOnI0.png)
![](https://i.imgur.com/4X82WWr.png)
![](https://i.imgur.com/o9E9I6B.png)




###### tags: `AWS Developer Associate` `Notes`