

15 AWS Monitoring - Cloudwatch, X-Ray and Cloudtrial
===




## Table of Contents

- [15 AWS Monitoring - Cloudwatch, X-Ray and Cloudtrial](#15-aws-monitoring---cloudwatch--x-ray-and-cloudtrial)
  * [Table of Contents](#table-of-contents)
  * [Monitoring in AWS](#monitoring-in-aws)
    + [AWS Monitoring](#aws-monitoring)
  * [AWS CloudWatch](#aws-cloudwatch)
    + [Metrics](#metrics)
      - [CloudWatch EC2 Detailed Monitoring](#cloudwatch-ec2-detailed-monitoring)
      - [Custom Metrics](#custom-metrics)
    + [Alarms](#alarms)
    + [Logs](#logs)
      - [Logs for EC2](#logs-for-ec2)
    + [Log Agents and Unified Agent](#log-agents-and-unified-agent)
      - [Unified Agent - Metrics](#unified-agent---metrics)
      - [Logs Metric Filter](#logs-metric-filter)
    + [Cloudwatch Events](#cloudwatch-events)
    + [Console](#console)
      - [Metrics](#metrics-1)
      - [Alarms](#alarms-1)
      - [Logs](#logs-1)
        * [Configuring Beanstalk to Stream Logs to Cloudwatch](#configuring-beanstalk-to-stream-logs-to-cloudwatch)
      - [Metric Filter](#metric-filter)
      - [Events](#events)
  * [Amazon EventBridge](#amazon-eventbridge)
    + [Schema Registry](#schema-registry)
    + [EventBridge VS CloudWatch Events](#eventbridge-vs-cloudwatch-events)
    + [Console](#console-1)
  * [AWS X-Ray](#aws-x-ray)
      - [Limitations of Debugging](#limitations-of-debugging)
    + [Intro to X-Ray](#intro-to-x-ray)
    + [Advantages](#advantages)
    + [Compatibility](#compatibility)
    + [Leverages Tracing](#leverages-tracing)
    + [Enabling X-Ray](#enabling-x-ray)
    + [Troubleshooting](#troubleshooting)
    + [X-Ray Instrumentation](#x-ray-instrumentation)
    + [X-Ray Concepts](#x-ray-concepts)
    + [Sampling Rules](#sampling-rules)
      - [Custom Sampling Rules](#custom-sampling-rules)
    + [X-Ray APIs](#x-ray-apis)
      - [Write APIs](#write-apis)
      - [Read APIs](#read-apis)
    + [X-Ray with Elastic Beanstalk](#x-ray-with-elastic-beanstalk)
    + [ECS + X-Ray](#ecs---x-ray)
      - [Integration Options](#integration-options)
      - [Example Task Definition](#example-task-definition)
    + [Console](#console-2)
      - [Sampling Rules](#sampling-rules-1)
      - [Xray with Beanstalk](#xray-with-beanstalk)
  * [AWS CloudTrial](#aws-cloudtrial)
    + [Console](#console-3)
  * [CloudTrial VS CloudWatch VS X-Ray](#cloudtrial-vs-cloudwatch-vs-x-ray)



Monitoring in AWS
---
- why is monitoring impt?
    - need to know how to deploy apps
        - safely
        - automatically
        - using infra as code
        - leveraging the best aws components
    - ensure app is always working at best of ability
        - app latency
            - inc over time?
        - app outages
            - customer experience shld not be degraded
    - troubleshooting and remediation
    - internal monitoring
        - can we prevent issues before it happens?
        - perf and cost
        - trends
            - scaling patterns
        - learning and improvement

### AWS Monitoring
- aws cloudwatch
    - metrics - collect and track key metrics
    - logs - collect, monitor, analyse and store log files
    - events
        - send notifs when certain events happen in your aws
    - alarms
        - react in realtime to metrics/events
- aws x-ray
    - troubleshooting app perf and errors
    - distributed tracing of microservices
- aws cloudtrial
    - internal monitoring of api calls made
    - audit changes to aws res by users


AWS CloudWatch
---
### Metrics
- cloudwatch provides metrics for __every__ service in aws
- __metric__ is a var to monitor
    - Eg. cpu utilisation, network bandwidth etc.
- metrics belong to namespaces
- __dimension__ is attribute of metric
    - Eg. instance id, env etc.
    - up to 10 dimensions per metric
- metrics have __timestamps__
- can create cloudwatch dashboard of metrics


#### CloudWatch EC2 Detailed Monitoring
- ec2 instance metrics have metrics every 5 mins
    - with details monitoring, u get data every 1 min at a cost
    - use detailed monitoring if want to more promptly scale your asg
- aws free tier allows for 10 detailed monitoring metrics
- NOTE
    - ec2 memory usage is by default not pushed
        - must be pushed from inside the instance as custom metric

#### Custom Metrics
- can define and send own custom metrics to cloudwatch
- abi to use dimensions (attr) to segment metrics
    - instance.id
    - env.name
- metric resolution - `StorageResolution` API param has 2 possible vals
    - standard
        - 1 min 
        - default
    - high resolution
        - 1 second
        - higher cost
        - have very fine-grained metrics
- use api called `PutMetricData`
    - to send metric to cloudwatch
- use exponential back off incase of throttle errors
    - eg. send too many metrics to cloudwatch

### Alarms
- alarms used to trigger notifs for any metric
    - can go to auto scaling, ec2 actions, sns notifs
- various options
    - sampling, %, max, min etc.
- alarm states
    - OK
    - INSUFFICIENT_DATA
    - ALARM
- period
    - length of time in seconds to evaluate the metric
    - high resolution custom metrics - can only choose 10 or 30 sec

### Logs
- apps can send logs to cloudwatch using sdk
- cloudwatch can collect logs from
    - elastic beanstalk
        - collection of logs from app
    - ecs
        - collection from containers
    - aws lambda
        - collection from func logs
    - vpc flow logs
        - vpc specific logs
    - api gateway
    - cloudtrial based on filter
    - cloudwatch log agents
        - Eg. ec2 machines
        - these agents will get log files and send to cloudwatch
    - route53
        - log dns queries
- logs can go to
    - batch explorer to s3 for archival
    - stream to elastisearch cluster for further analytics
- logs can use filter expressions
- logs storage architecture
    - log grps
        - arbitrary name, usually representing an app
    - log stream
        - instances within app/log files/containers
- can define log expiration policies
    - nvr expire, 30 days etc.
    - nvr expire by default
    - defined at log grp lvl
- using aws cli, we can tail cloudwatch logs
- to send logs to cloudwatch, ensure iam perms are correct
- security
    - encryption of logs using kms at grp lvl

#### Logs for EC2
- by default no logs from ec2 will go to cloudwatch
    - need to run cloudwatch agent on ec2 to push log files u want
    - make sure iam perms correct
        - role that can send logs to cloudwatch
- cloudwatch log agent can be set on-premise servers too

![](https://i.imgur.com/KBNgdcj.png)



### Log Agents and Unified Agent
- both for virtual servers 
    - Eg. ec2 instances, on-premise servers
- cloudwatch log agents
    - old ver of agent
    - can only send to cloudwatch logs
- cloudwatch unified agent (new)
    - collect extra system-lvl metrics like ram, processes etc.
    - collect logs to send to cloudwatch logs
        - can do both metrics and logs (hence unified)
    - centralised config using SSM param store
        - log agent dont have this

#### Unified Agent - Metrics
- collect directly on linux server/ec2 machine
    - cpu
        - active
        - guest
        - idle
        - system
        - user
        - steal
    - disk metrics
        - free, used, total
        - disk IO
            - writes, reads, bytes, iops
    - ram
        - free, inactive, used, total, cached
    - netstat
        - num of tcp and udp conns, net packets, bytes
    - processes
        - total, dead, bloqued, idle, running, sleep
    - swap space
        - free, used, used %
- reminder
    - out of box metrics for ec2 - disk, cpu, network (high lvl)
- cloudwatch unified agent allows you to get a lot more metrics and more granular details than normal monitoring of ec2 instances

#### Logs Metric Filter
- cloudwatch logs can filter expressions
    - eg find specific ip inside log or count occurences of error in logs
    - metric filters can be used to trigger alarms
- filters dont retroactively filter data
    - only publish metric data pts for events that happen after filter created

![](https://i.imgur.com/bRzkHvq.png)

### Cloudwatch Events
- schedule events - cron jobs
- event pattern
    - event rules to react to service doing sth
    - Eg. codepipeline state change
- triggers to lambda funcs, sqs, sns, kinesis msgs
- cloudwatch event creates small json doc to give info abt change



### Console
#### Metrics
![](https://i.imgur.com/HcM51JK.png)
![](https://i.imgur.com/1jTiESL.png)
- 801 metrics grped by category


![](https://i.imgur.com/5W7s9Za.png)
- can get graph representation of metric when selected
    - can choose duration of data shown - 1h, 3d, 1w etc

![](https://i.imgur.com/8nMUnv8.png)
- can add graphs to your own dashboard

![](https://i.imgur.com/IbzDOV2.png)

![](https://i.imgur.com/95IHTkJ.png)
- in asg, have option to enable group metrics collection
    - by default not enabled

![](https://i.imgur.com/OM8xnkp.png)
- for ec2 can also go to monitoring tab to view all metrics

#### Alarms
![](https://i.imgur.com/40oIwvT.png)
- shown here is default alarms created by beanstalk

![](https://i.imgur.com/UErcq7M.png)
- if app gets less than 2mb of data out for 5mins, decrease the size of our asg

![](https://i.imgur.com/nZvjxPA.png)
- can also view scaling policies in asg for a brief descrip of alarms

![](https://i.imgur.com/DvsbTU5.png)
- create alarm by first choosing a metric

![](https://i.imgur.com/1tP8VlA.png)
- set threshold for metric
- can also set alarm actions
    - sending notif
    - sending to an email
        - need to cfm your email

![](https://i.imgur.com/S7AqxVt.png)
- new alarm created
    - wait for state to become OK

#### Logs
![](https://i.imgur.com/W58UOix.png)
- default log grp created by codebuild

![](https://i.imgur.com/xRU7oAL.png)
![](https://i.imgur.com/gYJERoI.png)
- by clicking search logs, can filter your logs

![](https://i.imgur.com/aMjHyP4.png)
- edit retention setting
    - when does event expire?
- export data to amazon s3
- stream logs into elastisearch/lambda subscription filter

![](https://i.imgur.com/QTWWCC6.png)
![](https://i.imgur.com/TAyLV1i.png)


##### Configuring Beanstalk to Stream Logs to Cloudwatch
![](https://i.imgur.com/o1QRS6w.png)
- beanstalk env > configuration > edit software config

![](https://i.imgur.com/PcKMrvt.png)
- can stream logs to s3 or cloudwatch

![](https://i.imgur.com/cXdaIKk.png)

![](https://i.imgur.com/wVJjWQT.png)

- alternative can also scroll to monitoring config and edit (above)

#### Metric Filter
![](https://i.imgur.com/XQjcUew.png)
- create metric filter

![](https://i.imgur.com/1CYVer9.png)
- state pattern

![](https://i.imgur.com/wmBiWy0.png)
- can send custom log data or get directly from logs to test

![](https://i.imgur.com/YGYGFxJ.png)

![](https://i.imgur.com/QQ9H8Jd.png)
- assign metric attributes
    - metric name and namespace
    - filter name

![](https://i.imgur.com/VaJxMVK.png)
- state metric val
    - what val it returns when filter matches Eg. 1
    - default val - if no filter matches what does it return

![](https://i.imgur.com/56kpMgC.png)
- metric created wont appear in metric tab yet
    - since there arent any probs to generate logs to filter yet

![](https://i.imgur.com/p1PIaRj.png)
- go to one of your services and restart to generate a some logs

![](https://i.imgur.com/omI5jME.png)
- our custom metric shld appear now
    - click on it to generate a graph view

![](https://i.imgur.com/FrL5VW8.png)
- graph is empty as metric filters dont backfill data
    - only consists of data after filter is created

![](https://i.imgur.com/B4NAb2U.png)
- can click create alarm to do some automations

![](https://i.imgur.com/PNvwziC.png)
![](https://i.imgur.com/Gukwsgb.png)
- state conditions for alarm

![](https://i.imgur.com/u5bu9gX.png)
- set alarm trigger
    - and also whr to send notif to

![](https://i.imgur.com/XJpcywZ.png)

#### Events
![](https://i.imgur.com/ZhkTpC8.png)
- set event schedule
    - happen every 5mins etc.
    - can also setup cron expressions

![](https://i.imgur.com/iNz9X94.png)
- can view sample json obj of the event
    - can also edit event directly from its json obj

![](https://i.imgur.com/kKsHBBO.png)

- define target to invoke event on

![](https://i.imgur.com/YTsuAk7.png)
- can also invoke event based on event pattern

![](https://i.imgur.com/wghNpuZ.png)
- example
    - event invoke on codepipeline state change to FAILED

![](https://i.imgur.com/drVIU5z.png)
- finalise rule details 

Amazon EventBridge
---
- eventbridge is next evolution of cloudwatch events
- event buses
    - __default event bus__
        - generated by aws services
        - eg. cloudwatch events
    - __partner event bus__
        - receive events from saas service or apps
            - other parties can send events to your aws acc and u can react to it
        - Eg. zendesk, datadog, segment, auth0 etc.
    - __custom event buses__
        - for own apps to publish own events and have other apps react to it
- event buses can be accessed by other aws accs
    - allows for cross acc event buses
- rules
    - how to process the events
    - similar to cloudwatch events

### Schema Registry
- eventbridge can analyse the events in your bus and infer __schema__
    - schema - how the data is structured
- the schema registry allows you to generate code for your app that will know in advance how data is structured in the event bus
    - save time
    - add safety
- schema can be versioned
    - can make schema/events evolve over time

![](https://i.imgur.com/OJvNoHb.png)



### EventBridge VS CloudWatch Events
- eventbridge builds upon and extends cloudwatch events
    - uses same service API and endpt and same underlying service infrastructure
- eventbridge allows extension to add event buses for your custom apps and 3rd party saas apps
- eventbridge has schema registry capabilty
- event bridge has diff name to mark the new capabiities
    - over time, cloudwatch events will be replaced by eventbridge

### Console
![](https://i.imgur.com/6ff8ZG3.png)
- alr have default event bus

![](https://i.imgur.com/NApmLF0.png)
- create custom event bus
    - can give access to other accs or orgs

![](https://i.imgur.com/4lSnF2M.png)
- custom event bus created
    - own apps can publish event to

![](https://i.imgur.com/UoBr8Ag.png)
- partners with eventbridge
    - list will grow in future
    - ea partner will have setup instructions

![](https://i.imgur.com/9sRsYvF.png)
- example symantec
    - copy aws acc info
    - create event bus for symantec

![](https://i.imgur.com/s08bHUw.png)
- default event bus
    - to view its rules, click on the bus and go to the rules tab

![](https://i.imgur.com/9vBGrnU.png)
- 2 rules in eventbridge default bus is exact same rules in cloudwatch events
    - hence highlighting fact that they're built on same infra

![](https://i.imgur.com/duR3lIq.png)
- creating new rule for your bus

![](https://i.imgur.com/WGZVVwy.png)
- define pattern

![](https://i.imgur.com/LuzzDuq.png)
- many configs for service provider of rule

![](https://i.imgur.com/c7tWifZ.png)
- select event bus to invoke event on and select target

![](https://i.imgur.com/aTMyzdr.png)
- schema registry tab

![](https://i.imgur.com/S4qPwBA.png)
- many schemas included
    - can search through all
    - get info for schemas by aws
    - or self discovered schemas by our own event buses
    - or create own custom schema registry

![](https://i.imgur.com/pP3JdSM.png)
- example codepipeline schema for actionexecutionstatechange
    - have entire schema of ver 1
    - defines what you can expect in every field out of event coming from codepipeline for the event type

![](https://i.imgur.com/Hn1Vc9X.png)
- can download code bindings


AWS X-Ray
---
#### Limitations of Debugging
- debugging in production the good old way
    - test locally
    - add log statements everywhr
    - redeploy in prod
- log formats differ across apps using cloudwatch
    - makes analytics hard
    - have to centralise insights
- debugging
    - monolith - easy
    - distributed services - hard
        - hundred of microservices running
- no common views of your architecture

### Intro to X-Ray
- provides a visual analysis of our apps

![](https://i.imgur.com/vGSE22H.png)

### Advantages
- troubleshooting performance
    - bottlenecks
- understand dependencies in microservice architecture
- pinpoint service issues
- review request behaviour
- find errors and exceptions
- give insights
    - meeting time SLA (service lvl agreement)?
    - where are you throttled?
- identify users impacted


### Compatibility
- aws lambda
- elastic beanstalk
- ecs
- elb
- api gateway
- ec2 instances or app server
    - even on premise

### Leverages Tracing
- tracing is end to end way to following a request
    - ea component dealing with req adds its own trace
    - made up of segments and sub segments
- annotations can be added to traces to provide extra info
- ability to trace
    - every req
    - sample req - as % for example or rate per min
- x-ray security
    - iam for authorisation
    - kms for encryption at rest

### Enabling X-Ray
- through code
    - Eg. java, python, go, nodejs, .net
    - must import aws xray sdk
        - very little modification needed
        - app sdk will then capture
            - calls to aws services
            - http/https requests
            - database calls - mysql, postgresql, dynamodb
            - queue calls - sqs
- install x-ray daemon or enable x-ray aws integration
    - xray daemon works as low lvl udp packet interceptor
        - linux/windows/macs
    - aws lambda/other aws services alr run xray daemon for you
    - ea app must have iam rights to write data to xray

![](https://i.imgur.com/Ynirr1P.png)

- xray collects data from all diff services
    - service map computed from all segments and traces
- xray is graphical so even non tech people can help troubleshoot

### Troubleshooting
- if xray not working on ec2
    - ensure ec2 iam role has proper perms
    - ensure ec2 instance is running xray daemon
- to enable on aws lambda
    - ensure it has iam execution role with proper policy
        - `AWSX-RayWriteOnlyAccess`
    - ensure xray is imported in code
        - and xray integration enabled on lambda

### X-Ray Instrumentation
- __instrumentation__ - measure of product's perf, diagnose errors and to write trace info
    - to instrument your app code, use xray sdk
    - many sdk need config changes
- can modify app code to customise and annotate data that xray sends to xray using __interceptors, filters, handlers, middleware etc.__

![](https://i.imgur.com/A5jD1BP.png)

### X-Ray Concepts
- segments
    - ea app/service will send them
- subsegments
    - if need more details in segments
- trace
    - segments collected tgt to form end-to-end trace
- sampling
    - decrease amt of reqs sent to xray
    - reduce cost
- annotations
    - key val pairs used to index traces and use with filters
    - be able to search traces with new indexes vs metadatas
- metadata
    - key val pairs
    - not indexed
    - not used for searching
- xray daemon/agent has config to send traces cross acc
    - ensure iam perms correct - agent will assume the role
    - allows to have central acc for all the app tracing

### Sampling Rules
- with sampling rules, can control the amt of data u record
    - can modify sampling rules w/o changing code
    - more reqs, more u need to pay
- by default, xray sdk records 1st req ea second and 5% of any extra reqs
    - 1 req per sec is __reservoir__
        - this ensures at least 1 trace is recorded ea sec as long as service is serving reqs
        - reservoir set the least amount of reqs that has to be sent to xray
    - 5% is the __rate__ at which extra reqs beyond the reservoir size is sampled
        - rate is % of extra reqs out of total reqs to send to xray if reqs sent per sec reaches reservoir amt
- Eg. reservoir - 50, fixed rate - 10%
    - 100 reqs per second matches the rule - 50 + 5 reqs per sec sent
    - 5 is 10% of reservoir's 50
    - NOT SURE IF THIS CORRECT THOUGH
- [Read the Docs](https://docs.aws.amazon.com/xray/latest/devguide/xray-console-sampling.html)


#### Custom Sampling Rules
- can create own rules with reservoir and rate

![](https://i.imgur.com/UItEfu1.png)
- reservoir 10 so abt 10 reqs per sec is sent to xray


![](https://i.imgur.com/vWp1FiS.png)
- rate is 1 so all reqs are sent
    - since its debugging
    - though very expensive
- NOTE
    - if change sampling rules in xray console, dont need to restart apps

### X-Ray APIs
#### Write APIs
- used by xray daemon to write to xray
- `PutTraceSegments` - uploads segment documents to xray
    - necessary to write to xray
- `PutTelemetryRecords` - used by aws xray daemon to upload telemetry
    - `SegmentsReceivedCount`, `SegmentsRejectedCounts`, `BackendConnectionErrors` etc.
    - for xray daemon to upload info about how many segments were received, rejected and backend conn errors
    - helps with metrics
- `GetSamplingRules` - retrieve all sampling rules
    - to know what/when to send
    - for xray daemon to know what changed in rules when we update it
- `GetSamplingTargets` & `GetSamplingStatisticSummaries` - advanced
    - related to get APIS
- xray daemon needs to have iam policy authorising the correct api calls to func correctly

![](https://i.imgur.com/yoRj6uc.png)

#### Read APIs
- `GetServiceGraph` - main graph
- `BatchGetTraces`
    - retrieves list of traces specified by id
    - ea trace is collection of segments documents that originates from single req
- `GetTraceSummaries`
    - retrieves IDs and annotations for traces avail for specified time frame using optional filter
    - to get full traces, pass trace ids to `BatchGetTraces`
- `GetTraceGraph`
    - retrieves a service graph for 1 or more specific trace ids

![](https://i.imgur.com/i5EzDvI.png)


### X-Ray with Elastic Beanstalk
- aws elastic beanstalk platforms include xray daemon
    - can run daemon by setting option in beanstalk console or with config file
        - config file in `.ebextensions/xray-daemon.config` (see below it's just 1 line)
    - make sure to give instance profile the correct iam perms so xray daemon can function correctly
    - make sure app code is instrumented with xray sdk
- NOTE
    - xray daemon is not provided for mulicontainer docker

![](https://i.imgur.com/N05jUAM.png)

### ECS + X-Ray
#### Integration Options
- ecs cluster
    - xray container as daemon itself
    - xray daemon container running on every instance

![](https://i.imgur.com/f0qKHTP.png)

- ecs cluster
    - xray container as side car
    - 1 xray daemon alongside ea app container
        - they will connect from a networking standpt
        - xray daemon run side to side with app

![](https://i.imgur.com/pQtmkoC.png)

- fargate cluster
    - xray container as side car

![](https://i.imgur.com/9XD64hO.png)


#### Example Task Definition
![](https://i.imgur.com/9hrrT1f.png)
- port mapping mapped to udp port 2000 for container
    - app name - scorekeep-api
- env name called `AWS_XRAY_DAEMON_ADDRESS`
    - need to set so xray will know how to find xray daemon
- links section links the port mapping and xray daemon tgt from a networking standpoint


### Console
![](https://i.imgur.com/YgfN7uo.png)
- getting started
    - sample app or instrument own app

![](https://i.imgur.com/gAyUNpA.png)
![](https://i.imgur.com/K7qABQQ.png)

- add xray sdk to your app

![](https://i.imgur.com/kswsLmZ.png)
- xray sdk dont send traces directly so need to setup daemon too
    - they want to avoid throttling

![](https://i.imgur.com/9B4Qru9.png)
- when creating sample app will launch u into cloudformation to create a new stack

![](https://i.imgur.com/QaacxqM.png)
- a bunch of xray res created alongside

![](https://i.imgur.com/uT0a3jZ.png)
- xray sample app
    - start btn will generate signups every 6 seconds
    - creates duplicate signups every min which will generate error

![](https://i.imgur.com/rPDOmZ8.png)
- xray service map generated
- 169.254.169.254 is the metadata service from aws
    - can see that there's errors happening in front end dev app which is coming from dynamodb

![](https://i.imgur.com/vQlmqGL.png)
- error is highlighted in yellow

![](https://i.imgur.com/TgajPw3.png)
- can click error > view traces to view all traces of errors
    - 66% is signup while rest is favico

![](https://i.imgur.com/Zz0csVQ.png)
- more details on a trace
    - frontend made a req to dynamodb
    - POST > put item

![](https://i.imgur.com/3jPpJ5P.png)
- can even show what goes on in dynamodb

![](https://i.imgur.com/aA9rvz1.png)
- this is an example of an OK trace


#### Sampling Rules
![](https://i.imgur.com/H8tpTft.png)

![](https://i.imgur.com/4fcgxkj.png)
- default sampling rule
    - priority high
    - default rate is 5%

![](https://i.imgur.com/EYxRVnK.png)
- matching criteria to match reqs
    - now its set to match all

![](https://i.imgur.com/JDh2XJ5.png)

#### Xray with Beanstalk
- go to beanstalk env > configuration > software edit

![](https://i.imgur.com/63VAZEI.png)
- just enable xray daemon

![](https://i.imgur.com/0dfI5LA.png)
- make sure ec2 instance of beanstalk has correct roles

![](https://i.imgur.com/zmcssj8.png)
- elasticbeanstalk web tier for ec2 iam role

![](https://i.imgur.com/tCsxNyq.png)
- alr has read/write perms for xray

![](https://i.imgur.com/sdaxHzb.png)
- json ver for that



AWS CloudTrial
---
- provides governance, compliance and audit for your aws acc
    - enabled by default
- get history of events/API calls made within your aws acc
    - console
    - sdk
    - cli
    - aws services
- can put logs from cloudtrial into cloudwatch logs
    - so u can keep the events
- if res deleted in aws, look in cloudtrial first


### Console
![](https://i.imgur.com/euKHU2s.png)
![](https://i.imgur.com/zRcxi9o.png)
- example event

![](https://i.imgur.com/L1YufTu.png)

![](https://i.imgur.com/ocFqX1A.png)
- can filter event history



CloudTrial VS CloudWatch VS X-Ray
---
- cloudtrial
    - audit api calls made by users/services/aws console
    - useful to detect unauthorised calls or root cause of changes
- cloudwatch
    - cloudwatch metrics over time for monitoring
    - cloudwatch logs for storing app log
    - cloudwatch alarms to send notifs in case of unexpected metrics
- xray
    - automated trace analysis & central service map visualisation
    - latency, errors and fault analysis
    - request tracking across distributed systems



###### tags: `AWS Developer Associate` `Notes`