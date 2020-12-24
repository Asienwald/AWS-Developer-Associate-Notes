

17 AWS Lambda
===




## Table of Contents

- [17 AWS Lambda](#17-aws-lambda)
  * [Table of Contents](#table-of-contents)
  * [Serverless in AWS](#serverless-in-aws)
  * [AWS Lambda](#aws-lambda)
    + [AWS EC2 vs Lambda](#aws-ec2-vs-lambda)
    + [Benefits](#benefits)
    + [Language Support](#language-support)
    + [Main Service Integrations](#main-service-integrations)
      - [Example - Serverless Thumbnail Creation](#example---serverless-thumbnail-creation)
      - [Example - Serverless CRON Job](#example---serverless-cron-job)
      - [Example - Lambda Pricing](#example---lambda-pricing)
    + [Synchronous Invocations](#synchronous-invocations)
      - [Sync Invocations - Services](#sync-invocations---services)
    + [Lambda Integration with ALB](#lambda-integration-with-alb)
      - [Lambda Request: HTTP to JSON](#lambda-request--http-to-json)
      - [Lambda Response: JSON to HTTP](#lambda-response--json-to-http)
      - [ALB Multi-Header Values](#alb-multi-header-values)
    + [Lambda@Edge](#lambda-edge)
      - [Example - Global Application](#example---global-application)
      - [Use Cases](#use-cases)
    + [Asynchronous Invocations](#asynchronous-invocations)
      - [Services](#services)
    + [Integrating with Lambda](#integrating-with-lambda)
      - [Cloudwatch Events/EventBridge](#cloudwatch-events-eventbridge)
      - [S3 Event Notifications](#s3-event-notifications)
    + [Lambda - Event Source Mapping](#lambda---event-source-mapping)
      - [Streams - Kinesis and DynamoDB](#streams---kinesis-and-dynamodb)
      - [Queues - SQS & SQS FIFO](#queues---sqs---sqs-fifo)
      - [Event Mapper Scaling](#event-mapper-scaling)
    + [Destinations](#destinations)
    + [Lambda Permissions](#lambda-permissions)
      - [Lambda Execution Role (IAM)](#lambda-execution-role--iam-)
      - [Lambda Resource Based Policies](#lambda-resource-based-policies)
    + [Lambda Environment Variables](#lambda-environment-variables)
    + [Lambda Logging and Monitoring](#lambda-logging-and-monitoring)
      - [Tracing with X-Ray](#tracing-with-x-ray)
    + [Lambda in VPC](#lambda-in-vpc)
      - [VPC - Internet Access](#vpc---internet-access)
    + [Function Configuration](#function-configuration)
    + [Execution Context](#execution-context)
      - [Example - Initialising outside Handler](#example---initialising-outside-handler)
      - [/tmp space](#-tmp-space)
    + [Concurrency and Throttling](#concurrency-and-throttling)
      - [Concurrency Issue](#concurrency-issue)
      - [Concurrency and Asynchronous Invocations](#concurrency-and-asynchronous-invocations)
      - [Cold Starts and Provisioned Concurrency](#cold-starts-and-provisioned-concurrency)
      - [Reserved and Provisioned Concurrency](#reserved-and-provisioned-concurrency)
    + [Function Dependencies](#function-dependencies)
    + [Lambda and Cloudformation](#lambda-and-cloudformation)
      - [Inline](#inline)
      - [Through S3](#through-s3)
    + [Layers](#layers)
    + [Lambda Versions](#lambda-versions)
      - [Lambda Aliases](#lambda-aliases)
    + [Lambda and CodeDeploy](#lambda-and-codedeploy)
    + [Lambda Limits per Region](#lambda-limits-per-region)
    + [Best Practices](#best-practices)
    + [Console](#console)
      - [Sync Invocations](#sync-invocations)
      - [Multi-Value Headers](#multi-value-headers)
      - [Async Invocations](#async-invocations)
      - [Integrating with CloudWatch](#integrating-with-cloudwatch)
      - [S3 Event Notifs](#s3-event-notifs)
      - [Event Mappers](#event-mappers)
      - [Destinations](#destinations-1)
      - [Permissions](#permissions)
      - [Env Vars](#env-vars)
      - [Monitoring](#monitoring)
      - [VPC](#vpc)
      - [Execution Context](#execution-context-1)
      - [Concurrency](#concurrency)
      - [Function Dependencies](#function-dependencies-1)
      - [With Cloudformation](#with-cloudformation)
      - [Layers](#layers-1)
      - [Alias](#alias)
  * [Quiz](#quiz)




Serverless in AWS
---
- serverless is new paradigm in which devs dont manage servers anymore
    - they just deploy code and functions
    - initially serverless == faas (func as a service) but now its much more
- serverless was pioneered by aws lambda but now includes anything that's remotely managed
    - eg. DBs, msging, storage etc.
- serverless dont mean there's no servers
    - means u dont manage/provision/see them
- serverless in aws
    - aws lambda
    - dynamodb
    - cognito
    - api gateway
    - s3
    - sns and sqs
    - kinesis data firehose
    - aurora serverless
    - step funcs
    - fargate

![](https://i.imgur.com/Tvi6C3i.png)


AWS Lambda
---
### AWS EC2 vs Lambda
- aws ec2
    - virtual servers in cloud
    - limited by ram and cpu
    - continuously running
    - scaling means intervention to add/remove servers
- aws lambda
    - virtual funcs - no servers to manage
        - just provision code and funcs to run
    - limited by time
        - short executions up to 15mins
    - run on-demand
        - only billed when your func is running
    - scaling automated

### Benefits
- easy pricing
    - pay per request and compute time
    - free tier of 1,000,000 aws lambda requests and 400,000 gigabyte seconds of compute time
- integrated with whole aws suite of services
- integrated with many programming languages
- easy monitoring through aws cloudwatch
- easy to get more res per funcs
    - up to 3gb of ram per func
    - increasing ram of func will also inc cpu and network

### Language Support
- nodejs
- python
- java (java 8 compatible)
- c# (.NET core)
- golang
- c#/powershell
- ruby
- custom runtime api
    - community supported eg. rust
- NOTE
    - docker is not for aws lambda, it's for ecs/fargate

### Main Service Integrations
![](https://i.imgur.com/afHvHFh.png)
- api gateway to create rest api and invoke lambda funcs
- kinesis for data transformations on the fly
- dynamodb to create triggers 
    - when sth happens in db, lambda func triggered
- s3
    - lambda func trigger and file created in s3
- cloudfront will be lambda at edge
- eventbridge react to events in aws with lambda funcs
- cloudwatch logs to stream logs anywhr
- sns to react to notifs and sns topics
- sqs to process msgs from sqs queues
- cognito to react to whenever user logs into db

#### Example - Serverless Thumbnail Creation
![](https://i.imgur.com/NYdjgyc.png)
- img uploaded in s3
    - triggers s3 event notif which is a lambda func
    - lambda func code generates thumbnail
    - thumbnail either pushed to s3 or metadata inserted into dynamodb

#### Example - Serverless CRON Job
![](https://i.imgur.com/mpFwqit.png)
- cron - generate jobs every x mins on ec2 instances
- cloudwatch/eventbridge rule to trigger every 1 hour
    - integrated with lambda func to perform your task

#### Example - Lambda Pricing
- [Overall pricing info](https://aws.amazon.com/lambda/pricing/)
- pay per calls
    - 1st 1,000,000 requests free
    - $0.20 per 1mil requests thereafter
        - $0.0000002 per req
- pay per duration in increment of 100ms
    - 400,000 gb-seconds of compute time per month free
        - equals to 400,000 seconds if func is 1gb ram
        - equals 3,200,000 seconds if func is 128mb ram
    - after that $1 for 600,000 gb-seconds
- usually very cheap to run aws lambda so it's very popular

### Synchronous Invocations
- sync - cli, sdk, api gateway, app lb
    - result returned right away
    - err handling must happen in client side
        - eg. retries, exponential backoff etc.

![](https://i.imgur.com/m253oaY.png)
- client invoke api gateway which then proxies to lambda func which gives resp to gateway then client

#### Sync Invocations - Services
- user invoked
    - elastic lb (alb)
    - amazon api gateway
    - amazon cloudfront
        - lambda@edge
    - amazon s3 batch
- service invoked
    - amazon cognito
    - aws step funcs
- other services
    - amazon lex
    - amazon alexa
    - amazon kinesis data firehose
- NOTE
    - for dev associate, only need know alb, api gateway, cloudfront, cognito and step funcs

### Lambda Integration with ALB
- to expose lambda func as http(s) endpt can use alb or api gateway
    - funcs can also be either invoked through cli or sdk
- lambda func must be registered in target grp

![](https://i.imgur.com/9ffy6i2.png)

#### Lambda Request: HTTP to JSON
![](https://i.imgur.com/yq8Wwnw.png)
- elb info
    - which elb invokes and what's target grp
- http method and path
- query string params as key val pairs
- headers
- body (base64 encoded)

#### Lambda Response: JSON to HTTP
![](https://i.imgur.com/6HN1qHX.png)

#### ALB Multi-Header Values
- alb can support multi header vals
    - alb setting
- when you enable multi-val headers, http headers and query string params that are sent with multiple values are shown as arrays within lambda event and resp objs


![](https://i.imgur.com/1B1rTXO.png)


### Lambda@Edge
- have deployed a cdn using cloudfront
    - what if want to run global aws lambda alongside ea edge location?
    - or how to implement request filtering before rching your app?
- use Lambda@edge to deploy lambda funcs alongside cloudfront cdn
    - deployed not in specific region but alongside ea region arnd world with cloudfront cdn
    - build more responsive apps
    - dont manage servers
        - lambda is deployed globally
    - customise cdn content
    - pay only for what you use
- can use lambda to change cloudfront requests and responses
    - after cloudfront receives request from viewer
        - viewer request
    - before cloudfront forwards the request to origin
        - origin request
    - after cloudfront receives the response from origin
        - origin response
    - before cloudfront forwards the response to viewer
        - viewer response
- can also generate resps to viewers w/o sending req to origin

![](https://i.imgur.com/ci0Lk0Q.png)

#### Example - Global Application
![](https://i.imgur.com/GayrNw4.png)

#### Use Cases
- website security and privacy
- dynamic web app at edge
- search engine optimisation (SEO)
- intelligently route across origins and data centers
- bot mitigation at edge
- realtime image transformation
- a/b testing
- user auth and authorisation
- user prioritisation
- user tracking and analytics


### Asynchronous Invocations
- for s3, sns, cloudwatch events etc, events are placed in an __event queue__
- lambda attempts to retry on errors
    - 3 tries total
    - 1 min wait for 1st then 2 mins wait
- make sure processing is idempotent in case of retries
    - if func is retried, will see duplicate log entries in cloudwatch logs
- hence can define dlq (dead letter queue)
    - sns or sqs
    - for failed processing
        - need correct iam perms
- async invocations allow you to speed up processing if dont need to wait for result
    - Eg. need 1000 files processed

![](https://i.imgur.com/841rSpT.png)
- s3 create file event into event queue
    - lambda func read off queue
        - tries to process these events
        - if fails, will retry (3 times)
- __idempotency__ - in case of retries, results shld always be the same

#### Services
- simple storage service (s3)
    - event notifs
- simple notif service (sns)
- cloudwatch events/eventbridge
- codecommit, codepipeline etc.
- cloudwatch logs
- simple email service
- cloudformation
- aws config
- aws iot and iot events
- many more
- NOTE
    - for dev associate, only need know s3, sns and eventbridge


### Integrating with Lambda
#### Cloudwatch Events/EventBridge
![](https://i.imgur.com/OFjTy5H.png)
- 2 ways
    - serverless cron OR rate
        - eventbridge rule that triggers func every 1h
    - codepipeline eventbridge rule
        - eg. to detect everytime codepipeline state changes
        - invoke lambda on state change

#### S3 Event Notifications
- `S3:ObjectCreated`, `S3:ObjectRemoved`, `S3:ObjectRestore`, `S3:Replication` etc.
    - obj name filtering possible
        - eg. *.jpg
- use case
    - generate thumbnails of imgs uploaded to s3
- s3 event notifs typically deliver events in seconds but can sometimes take min or longer
- if 2 writes made to single non-versioned obj at same time, possible that only single event notif will be sent
    - if want to ensure that event notif is sent for every successful write, can enable versioning on bucket

![](https://i.imgur.com/Kr8D6Fv.png)
- s3 send to sns with fanout pattern with sqs
    - or send to sqs queue with lambda func
    - OR s3 event notif async invoke lambda func
        - dlq for any errors

__Event Pattern - Metadata Sync__
![](https://i.imgur.com/e9ukTDJ.png)
- metadata inserted into dyanmodb table or rds db


### Lambda - Event Source Mapping
- applies to
    - kinesis data streams
    - sqs and sqs fifo queue
    - dynamodb streams
- common denominator of all these services is that records needs to be polled from src
    - lambda needs to ask the service to get the records and then its returned
    - your lambda func is invoked synchronously
- 2 categories
    - streams
    - queues

![](https://i.imgur.com/jRkYHX1.png)
- lambda internally creates event src mapping that will poll from kinesis and return results
    - when event src mapping has data for lambda to process, will invoke lambda func synchronously with event batch

#### Streams - Kinesis and DynamoDB
- event src mapping creates iterator for ea shard and processes items in order
    - can read shard from start with new items from beginning or from timestamp
- processed items arent removed from stream
    - consumers can read them
- use case
    - low traffic - use batch window to accumulate records before processing
    - if high traffic
        - can process multiple batches in parallel at shard lvl
            - up to 10 batches per shard
            - in-order processing for ea batch still guaranteed for ea partition key
- https://aws.amazon.com/blogs/compute/new-aws-lambda-scaling-controls-for-kinesis-and-dynamodb-event-sources/

![](https://i.imgur.com/P2j1BYL.png)

- error handling
    - by default entire batch is reprocessed until func succeeds or items in batch expires
- to ensure in-order processing, processing for affected shard is paused until error resolved
- can config event src mapping to
    - discard old events
    - restrict num of retries
    - split batch on error to work arnd lambda timeout issues
- discarded event go to a __destination__

#### Queues - SQS & SQS FIFO
- event src mapping will poll sqs (long polling)
    - specify batch size
        - 1-10 msgs
- recommended - set queue visibility timeout to 6x the timeout of your lambda func
- to use dlq,
    - setup on the sqs queue not lambda
        - dlq for lambda is only for async invocations
    - or use lambda destination for failures

![](https://i.imgur.com/YZrArRn.png)
![](https://i.imgur.com/CtqZvJ1.png)
- sqs queue polled by lamda event src mapping
    - whenever batch returned, lambda func invoked synchronously with event batch

__More Information__
- lambda also supports in-order processing for fifo queues
    - scales up to num of active msg grps (grp id)
- for standard queues, items arent necessarily processed in order
    - lambda scales up to process a standard queue asap
- when error occurs, batches returned to queue as individual items
    - might not be processed in diff grping than orig batch
- occasionally event src mapping might receive same time from queue twice even if no func err occured
    - ensure idempotent processing for lambda func
- lambda deletes items from queue after processed successfully
    - can config src queue to send items to dlq if cant be processed

#### Event Mapper Scaling
- kinesis data streams and dynamodb streams
    - 1 lambda invocation per stream shard
    - if use parallelisation, up to 10 batches processed per shard simultaneously
- sqs standard
    - lambda adds 60 more instances per min to scale up
    - up to 1000 batches of msgs processed simultaneously
- sqs fifo
    - msgs with same grp id will be processed in order
    - lambda func scales to num of active msg grps

### Destinations
- nov 2019 - can now config to send result to a destination
    - hard to see result of async funcs hence can send result to a destination
- async invocations - can define dest for successful and failed events
    - destination types
        - sqs
        - sns
        - lambda
        - eventbridge bus
    - NOTE
        - aws recommends use of dest instead of dlq now
            - though can use both at the same time
            - why dest? cuz its newer and allows for more targets
                - dlq just allow u to send failure to sqs/sns but dest send both success and failure into sqs/sns/lambda/eventbridge
        - https://docs.aws.amazon.com/lambda/latest/dg/invocation-async.html

![](https://i.imgur.com/ZIQr48T.png)
- __event source mapping__ - for discarded event batches
    - sqs
    - sns
    - NOTE
        - can send events to dlq directly from sqs
        - https://docs.aws.amazon.com/lambda/latest/dg/invocation-eventsourcemapping.html
![](https://i.imgur.com/TfNPkhJ.png)

### Lambda Permissions
#### Lambda Execution Role (IAM)
- grants lambda func perms to aws services/res
- sample managed policies for lambda
    - `AwsLambdaBasicExecutionRole`
        - upload logs to cloudwatch
    - `AwsLambdaKinesisExecutionRole`
        - read from kinesis
    - `AwsLambdaDynamoDBExecutionRole`
        - read from dynamodb streams
    - `AwsLambdaSQSQueueExecutionRole`
        - read from sqs
    - `AwsLambdaVPCAccessExecutionRole`
        - deploy lambda func in vpc
    - `AwsXRayDaemonWriteAccess`
        - upload trace data to xray
- when you use event src mapping to invoke func, lambda uses execution role to read event data
- best practice
    - one lambda execution role per func

#### Lambda Resource Based Policies
- use res-based policies to give other accs and aws services perms to use lambda res
- similar to s3 bucket policies for s3 bucket
- iam principal can access lambda
    - if iam policy attached to principal authorises it
        - eg. user access
    - or if res-based policy authorises
        - eg. service access
- when aws services like s3 calls lambda func, res-based policy gives it access

### Lambda Environment Variables
- env var = key/val pair in string form
    - adjust func behaviour w/o updating code
- env vars are avail to code
- lambda service adds own system env vars too
- helps to store secrets
    - encrypted by kms
    - secrets can be encrypted by lambda service key or your own cmk

### Lambda Logging and Monitoring
- cloudwatch logs
    - aws lambda execution logs stored in aws cloudwatch logs
    - ensure func has execution role with iam policy that authorises writes to cloudwatch logs
        - included in lambda basic execution role
- cloudwatch metrics
    - lambda metrics are displayed in aws cloudwatch metrics
    - will display
        - invocations, durations, concurrent executions
        - err count, success rates, throttles
        - async delivery failures
        - iterator age
            - kinesis and dynamodb streams

#### Tracing with X-Ray
- enable in lambda config  - active tracing
    - runs xray daemon for you
    - uses aws xray sdk in code
- ensure func has correct iam execution role
    - managed policy is called `AWSXRayDaemonWriteAccess`
- env vars to communicate with xray
    - `_X_AMZN_TRACE_ID`
        - contains missing tracing header
    - `AWS_XRAY_CONTEXT_MISSING`
        - by default, `LOG_ERROR`
    - `AWS_XRAY_DAEMON_ADDRESS`
        - xray daemon `IP_ADDRESS:PORT`

### Lambda in VPC
- lambda by default
    - by default lambda func is launched outside your own vpc
        - in an aws-owned vpc
    - hence cannot access res in your vpc
        - eg. rds, elasticache, internal elb etc.
![](https://i.imgur.com/wQOsIQz.png)

- lambda in vpc
    - must define vpc id, subnets and security grps
    - lambda will create ENI (elastic network interface) in your subnets
        - `AWSLambdaVPCAccessExecutionRole` neede to create ths ENI

![](https://i.imgur.com/zLRDOMC.png)
- rds sec grp must allow access from lambda sec grp

#### VPC - Internet Access
- lambda func in vpc dont have internet access
    - deploying lambda func in public subnet dont give it internet access or public ip
        - only have internet if u have a __NAT gateway/instance__
    - can use vpc endpts to privately access as services w/o a NAT
- NOTE
    - cloudwatch logs works even w/o endpt or nat gateway

![](https://i.imgur.com/I585P75.png)
- func will go through public subnet with nat device
    - nat gateway talks to internet gateway of the vpc
    - internet gateway gives access to external api
- dynamodb can be accessed either through internet gateway or use vpc endpts

### Function Configuration
- ram
    - from 128mb to 3008mb in 64mb increments
        - the more ram you add, more vpu credits u get
    - at 1792mb, a func has equivalent of 1 full vcpu
        - after 1792mb, you get more than 1 cpu and need to use multi-threading in code to benefit from it
- if app is cpu-bound (computation heavy), increase ram if want to improve perf
- timeout
    - default 3 seconds, max 900 seconds (15mins)
    - anything above 15mins is not a good case of lambda
        - btr for fargate or ecs or ec2

### Execution Context
- execution context is temp runtime env that initialises any external dependencies of your labda code
    - great for db conns, http clients, sdk clients etc.
- execution context is maintained for some time in anticipation of another lambda func invocation
    - next func invocation can reuse the context to execution time and save time in initialising conn objs
- execution context includes `/tmp` dir
    - is a space to write files and is avail across executions

#### Example - Initialising outside Handler
- bad
    - db conn established at every func invocation

![](https://i.imgur.com/YWRYbZH.png)

- good
    - db conn established once and reused across invocations
    - improve perf

![](https://i.imgur.com/FalKC7Q.png)

#### /tmp space
- use /tmp when
    - lambda func needs to download big file to work
    - or if func need disk space to perform operations
- max size 512mb
- dir content remains when execution context frozen, providing transient cache that can be used for multiple invocations
    - helpful to checkpoint your work
- for permanent persistence of obj (non-temp), use s3
    - cannot use /tmp

### Concurrency and Throttling
- concurrency limit - up to 1000 concurrent executions
- can set __reserved concurrency__ at func lvl
    - limits num of concurrent executions
    - ea invocation over the concurrency limit will trigger a throttle
- throttle behaviour
    - if sync invocation
        - return ThrottleError - 429
    - if async invocation
        - retry automatically and then go to dlq
- if need higher limit (>1000), open a support ticket

#### Concurrency Issue
- if dont reserve concurrency (limit) all func concurrency might be used for 1 app
    - other apps get throttled
    - concurrency limit applies to all funcs in your acc
        - if 1 func goes over limit, all others might be throttled

![](https://i.imgur.com/nIncRDI.png)

#### Concurrency and Asynchronous Invocations
- if func dont have enough concurrency avail to process all events, extra requests are throttled
    - for throttling errors (429) and system errors (500 series), lambda return the event to queue and attempts to run the func again for up to 6 hours
        - lots of retries happen due to throttling
    - retry interval increases exponentially from 1 second after 1st attempt to max of 5 mins

![](https://i.imgur.com/OW7tNsu.png)

#### Cold Starts and Provisioned Concurrency
- cold start
    - new instance - code loaded and code outside the handler run (init)
    - if init is large (code, dependencies etc.), process can take a lot of time
    - 1st req served by new instances has higher latency than rest
        - users might be unhapppy
- hence use provisioned concurrency
    - concurrency allocated before func invoked (in advance)
        - so cold start nvr happens and all invocations have low latency
    - app auto scaling can manage concurrency
        - eg. for schedule or target utilisation
        - ensure enough lambda funcs to be ready and minimise cold start prob
- NOTE
    - cold starts in vpc are dramatically reduced in oct and nov 2019
    - https://aws.amazon.com/blogs/compute/announcing-improved-vpc-networking-for-aws-lambda-functions/

#### Reserved and Provisioned Concurrency
![](https://i.imgur.com/caNtfXe.png)
![](https://i.imgur.com/ZEQVwdX.png)
- https://docs.aws.amazon.com/lambda/latest/dg/configuration-concurrency.html
    - look at this in own time as its pretty complicated to describe

### Function Dependencies
- if func depends on external libs eg. aws xray sdk, db clients etc., need to install packages alongside your code and zip tgt
    - for nodejs, use `npm` and `node_modules` dir
    - for python, use `pip --target` options
    - for java, include relevant `.jar` files
- upload zip straight to lambda if less than 50mb else to s3 first
- native libs work
    - need to be compiled on amazon linux
- aws sdk comes by default with every lambda func

### Lambda and Cloudformation
- 2 ways to do it

#### Inline
- inline funcs very simple
    - use code.zipfile property
- cannot include func dependencies with inline funcs
    - so just for simple use cases

![](https://i.imgur.com/wfjxHYC.png)

#### Through S3
- must store lambda zip in s3
- must refer s3 zip location in cloudformation code
    - s3 bucket
    - s3 key - full path to zip
    - s3objectversion if versioned bucket
- if update code in s3 but dont update s3 bucket, s3 key or s3objectversion, cloudformation wont update your func

![](https://i.imgur.com/iflZWbV.png)


### Layers
- custom runtimes
    - eg. [c++](https://github.com/awslabs/aws-lambda-cpp)
    - eg. [rust](https://github.com/awslabs/aws-lambda-rust-runtime)
- other more common use case is to externalise dependencies to reuse them

![](https://i.imgur.com/LYa4Fvg.png)
- if want to change dependencies will have to keep repackaging app and reupload code
- goal is to externalise your app package 
    - create layer for your libraries that can be referenced from your code so dont need to repackage everytime for dependencies
    - other funcs can also reference your layer

### Lambda Versions
- when work on lambda func, we work on $LATEST
    - when we're ready to publish a lambda func, we create a ver
- versions are immutable
    - have increasing ver numbers
    - get their own ARN (amazon resource name)
- version = code + config
    - nothing can be changed - immutable
        - immutable = cannot change code or env vars or anything afterwards (fixed)
- ea ver of lambda func can be accessed

![](https://i.imgur.com/bdhiQfM.png)

#### Lambda Aliases
- __aliases__ are pointers to lambda func versions
    - can define a dev, test, prod alias and have them point at diff lambda vers
- aliases are mutable
- enable blue/green deployment by assigning weights to lambda funcs
    - enable stable config of event triggers/destinations
- aliases have their own ARNs
- aliases cannot reference aliases

![](https://i.imgur.com/TsCnh1r.png)
- eg. dev alias always pointing to the latest version
    - test alias to test v2 of our app
    - prod alias to point to v1 ver that we know is stable

### Lambda and CodeDeploy
- codedeploy can help u automate traffic shift for lambda aliases
    - feature is integrated within the SAM framework
        - hence this will be elaborated more in the SAM framework
- linear - grow traffic every N minutes until 100%
    - Linear10PercentEvery3Minutes
    - Linear10PercentEvery10Minutes
- canary - try x percent then 100%
    - Canary10Percent5Minutes
    - Canary10Percent30Minutes
- allatonce - immediate
- can create pre and post traffic hooks to check health of lambda func
    - if anything goes wrong traffic hooks can be failing

![](https://i.imgur.com/xJNYRVo.png)
- want to shift prod alias from v1 to v2
    - slowly from 100% to x%

### Lambda Limits per Region
- execution
    - memory allocation
        - 128mb to 3008mb
        - 64mb increments
    - max execution time
        - 90 seconds (15mins)
    - env vars
        - 4kb
    - disk capacity in the function container
        - in /tmp
            - for bigger files compared to env vars
        - 512mb
    - concurrency executions
        - 1000
        - can be increased
- deployment
    - lambda func deployment size
        - compress .zip
        - 50mb
    - size of uncompressed deployment
        - code + dependencies
        - 250mb
    - can use /tmp dir to load other files at startup
    - size of env vars
        - 4kb

### Best Practices
- perform heavy-duty work outside of func handler
    - connect to db outside of func handler
    - initialise aws sdk outside of func handler
    - pull in dependencies or datasets outside of func handler
- use env vars for
    - db conn strings, s3, etc.
        - dont put these values in your code
    - passwords, sensitive values etc.
        - can be encrypted using KMS
- minimise deployment package size to its runtime necessities
    - break down func if func is too big
    - rmb aws lambda limits
    - use layers when need to reuse libraries
- avoid using recursive code, never have a lambda func call itself








### Console
![](https://i.imgur.com/GqRB6f5.png)
![](https://i.imgur.com/k9NRVcb.png)
![](https://i.imgur.com/9dmmPcN.png)
- use blueprint so dont have to setup from scratch

![](https://i.imgur.com/ZdFq9LR.png)
- choose role that lambda func will use to execute its actions

![](https://i.imgur.com/xpQL2SW.png)
- prints  all 3 keys and return

![](https://i.imgur.com/rD1vuMj.png)
- can change func code in console

![](https://i.imgur.com/5BTJIF2.png)
- click on test btn to test the func

![](https://i.imgur.com/r5cXC4k.png)

![](https://i.imgur.com/iQgJMeR.png)
- execution role for our func created
    - has a res summary indicating access to cloudwatch logs


#### Sync Invocations
![](https://i.imgur.com/avDr7uF.png)
![](https://i.imgur.com/0l6HEMG.png)
- pass func called `hello-world`
    - in cli binary format to pass in adj val as text but have it converted in vase64 by cli
    - resp written in `response.json` by the func

#### Multi-Value Headers
![](https://i.imgur.com/qWLyevz.png)
- this time create func from scratch

![](https://i.imgur.com/sz4bq7f.png)
- config new alb in ec2 console

![](https://i.imgur.com/mqPmw5w.png)
- routing point to lambda func

![](https://i.imgur.com/3iKtMHE.png)
![](https://i.imgur.com/MEq7F7R.png)
- example of an event obj from lambda func

- if visit dns of alb, will download response
    - this is because orig no response with content type pointing to text/html

![](https://i.imgur.com/sboqDMV.png)
- change up the response obj so hello world is shown as html

![](https://i.imgur.com/Z1xyqrQ.png)

![](https://i.imgur.com/97HdbEO.png)
- to enable multi val, go to lb target grp and under attributes enable multi value headers

![](https://i.imgur.com/L0YpO8b.png)
- alb has a res based policy
    - is what allows the alb to invoke lambda func

#### Async Invocations
- to invoke async, cannot do through console have to do through cli

![](https://i.imgur.com/FKgU5Xc.png)
- returns status 202 which is for async invocations
    - know func is invoked but we're not waiting for results

![](https://i.imgur.com/WBc7NsC.png)
- func invoked as seen in logs but we did not wait for results to come back
    - hence if func has an error, we also wont know abt it
    - hence setup a dlq

![](https://i.imgur.com/MvUTwzo.png)
- edit your async settings

![](https://i.imgur.com/MqRmKgH.png)
- can specify num of retries and your dlq

![](https://i.imgur.com/BOf6ROO.png)
- have to go to iam and give your lambda func role write to sqs

![](https://i.imgur.com/XSrEq9W.png)
- there's a role specially for this

![](https://i.imgur.com/H9I9Gl7.png)
- cloudwatch has several logs with same request ids which reflects the lambda retries


#### Integrating with CloudWatch
- create a new lambda func

![](https://i.imgur.com/G7LTOj5.png)
- go to eventbridge and create new rule
    - choose schedule

![](https://i.imgur.com/Xqi9yLT.png)

![](https://i.imgur.com/Wjm3LzL.png)
- set target to lambda func

![](https://i.imgur.com/2jf8pQH.png)
- func invoked every min

#### S3 Event Notifs
- create new bucket with default settings

- add new event in bucket settings

![](https://i.imgur.com/aYJZ3hA.png)
- send to lambda func

![](https://i.imgur.com/mjNtpYj.png)
- now you'll see s3 as a source of lambda func in your func console

![](https://i.imgur.com/SH7WUDZ.png)
- res based policy allows invocation from amazon s3 service into our func

#### Event Mappers
![](https://i.imgur.com/QYCKB8g.png)
- add trigger sqs

![](https://i.imgur.com/qpY01Xd.png)

![](https://i.imgur.com/K7gVugW.png)
- event mapper dont have role with enough perms to read from lambda

![](https://i.imgur.com/tMRA2OZ.png)
![](https://i.imgur.com/bOKlvs9.png)
- now your func have enough perms

![](https://i.imgur.com/uIbbAho.png)
- can go to queue actions > add message to send demo msg

![](https://i.imgur.com/hdwmu2T.png)
- lambda func will have triggered from the event mapper
    - lambda will show 0 msgs avail since msgs are deleted after processed

![Uploading file..._e9cjikmyh]()
- disable event mapper if not in use so wont incur costs

![](https://i.imgur.com/JNi5Tt1.png)
- now a demo for kinesis
    - batch size
    - batch window - gather records tgt?

![](https://i.imgur.com/N0SXoHm.png)

- starting pos
    - whr u want to read from the latest data


- on failure dest
    - discard data to sqs queue/topic on failure dest?
- how many retries in case of errors?
- max age of record to process?
- split batch on error? in case batch is too big and lambda func timesout

![](https://i.imgur.com/2w4KEX5.png)

- concurrent batches per shard
    - how many processes u want on ea shard
    - 1 means all records in shard processed in order
    - if set 10 max, have 10 processors that will process data in shard
        - also get ordering by partition key id
- NOTE
    - dynamodb will have exact same configs


#### Destinations
![](https://i.imgur.com/dc2v13f.png)
- set src, condition and dest type

![](https://i.imgur.com/BEatmcx.png)
- sqs role also created to allow sendning msgs into sqs queue

![](https://i.imgur.com/GacsS2N.png)
- can see dest in console ui

![](https://i.imgur.com/ObQZfzm.png)
- 2 dest added
    - success and failure

![](https://i.imgur.com/07c85no.png)
- can view success/failure msgs in sqs queue

![](https://i.imgur.com/zm3PYOq.png)
- failure msg example
    - 2 trigger failure fail

#### Permissions
![](https://i.imgur.com/7aDCEH2.png)
- role created for lambda funcs

![](https://i.imgur.com/0UtxmKH.png)
- view res based policy from func console > permissions > scroll to res based policy

![](https://i.imgur.com/cBhz7c0.png)
- res based policy will be empty for lambda funcs with an event mapper
    - as sqs is not polling lambda, its lambda polling sqs

#### Env Vars
![](https://i.imgur.com/vjICjiW.png)
- have to first import os package in func to use env vars

- scroll down from func code to see env var window

![](https://i.imgur.com/6jnzTBB.png)
![](https://i.imgur.com/eK3Mc20.png)
- can set env var security configs too

- go to same window to edit env var

#### Monitoring
![](https://i.imgur.com/bp45rbq.png)
- view cloudwatch metrics in monitoring tab of lambda func

![](https://i.imgur.com/lM9qBFP.png)
- cloudwatch logs grped by log grps for ea func

![](https://i.imgur.com/KDLVgUo.png)
- can also enable xray in func console
    - enable active tracing

#### VPC
![](https://i.imgur.com/xlMcDgw.png)
- scroll down func console to view vpc settings

![](https://i.imgur.com/KUCstXV.png)
![](https://i.imgur.com/4Xqqpte.png)
- select subnets and sec grp

![](https://i.imgur.com/BeaUgbh.png)
- need role to create eli to deploy to vpc

#### Execution Context
![](https://i.imgur.com/LFhGS7W.png)
- edit basic settings in func console to edit ram configs
    - and timeout
- set timeout based on what func is supposed to do
    - if u know processing might take awhile, inc timeout
- db conn code shld also be outside of your func call instead of within the func itself

#### Concurrency
![](https://i.imgur.com/YkHG1dJ.png)
- scroll down func console to find concurrency window
- can choose to reserv concurrency
    - if set to 0, func is always throttled (used for testing)

![](https://i.imgur.com/xLbvsrc.png)
- can also set provisioned concurrency

#### Function Dependencies
![](https://i.imgur.com/X8yH9Qp.png)
- nodejs example
    - require awsxray sdk
    - then create s3 client
        - dont need to bundle since its alr bundled with lambda func deployments

![](https://i.imgur.com/K1WdxRw.png)
- install required sdks
    - creates node_modules dir and package-lock.json

![](https://i.imgur.com/m3jnisD.png)
- choose zip for code entry type in func console

![](https://i.imgur.com/ZF8l0bQ.png)
- cannow navigate node_modules in console too


- enable xray active tracing since code is using xray
- also give required execution roles

![](https://i.imgur.com/8ePwgKr.png)
 - service graph created in xray due to usage in lambda func

#### With Cloudformation
![](https://i.imgur.com/mjdCW0h.png)
- 3 bucket params impt for lambda func

![](https://i.imgur.com/KFTOeJm.png)

![](https://i.imgur.com/MyXc0V5.png)

- res
    - iam role
    - policies


![](https://i.imgur.com/5wIp0di.png)
- this is our lambda func
    - specify handler
    - specify role too (ref to role defined above)
    - for code, specify s3 params (also defined above and referenced)
    - runtime and timeout
    - enable xray through `TracingConfig`

![](https://i.imgur.com/tSy91UA.png)
- enable versioning for s3 bucket with your code

![](https://i.imgur.com/hZOVjvJ.png)
- when creating your cloudformation stack, enter details of your s3 bucket with your code

![](https://i.imgur.com/heRS0Ko.png)

#### Layers
![](https://i.imgur.com/yCEdYfs.png)
- click on layers in func console and click add layer

![](https://i.imgur.com/w9ObYHd.png)
![](https://i.imgur.com/pg2ofeB.png)
- can get python code from aws docs to copy into the func layer
    - code has import numpt and scipy.spatial on top

#### Alias
![](https://i.imgur.com/0VDVLEx.png)
- qualifiers tab has version and alias

![](https://i.imgur.com/ifnTbmZ.png)
- action > publish new version

- can use version tab to change diff func versions in lambda code

![](https://i.imgur.com/meqtgso.png)
- create alias to point to a specific version
    - can use alias to change version instead of using version number

![](https://i.imgur.com/dVIlHza.png)
- can give weight for each alias
    - eg. 10% of users on that alias will be on a specific version


Quiz
---
![](https://i.imgur.com/LUdnODS.png)
![](https://i.imgur.com/U0qpjsE.png)





###### tags: `AWS Developer Associate` `Notes`