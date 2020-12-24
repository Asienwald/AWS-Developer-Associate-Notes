
22 Step Functions & AppSync
===




## Table of Contents

- [22 Step Functions & AppSync](#22-step-functions---appsync)
  * [Table of Contents](#table-of-contents)
  * [AWS Step Functions](#aws-step-functions)
    + [Visual Workflow](#visual-workflow)
    + [Error Handling](#error-handling)
    + [Standard vs Express](#standard-vs-express)
    + [Console](#console)
  * [AWS AppSync](#aws-appsync)
    + [AppSync Security](#appsync-security)
    + [Console](#console-1)
  * [Quiz](#quiz)



AWS Step Functions
---
- build serverless visual workflow to orchestrate lambda funcs
    - represent flow as json state machine
- features
    - sequence
        - 1 func then another
    - parallel
        - 2 funcs at once 
    - conditions
    - timeouts
    - error handling
        - retry and catch
- can also integrate with ec2, ecs, on premise servers, api gateway
- max execution time of 1 year
- possibility to implement human approval feature
    - have human review before proceed with workflow
- use cases
    - order fulfillment
    - data processing
    - web apps
    - any workflow

### Visual Workflow
![](https://i.imgur.com/Q3zv4Nj.png)
- get graph like this after designing state machine in json

### Error Handling
- any state can encounter runtime errors for various reasons
    - state machine definition issues
        - eg. no matching rule in choice state
    - task failures
        - eg. exception in lambda func
    - transient issues
        - eg. network partition events
- by default, when a state reports an error, aws step funcs causes the execution to fail entirely
- retrying failures - `Retry`
    - `IntervalSeconds`, `MaxAttempts`, `BackoffRate`
- moving on - `Catch`
    - `ErrorEquals`, `Next`
- best prac to include data in error msgs

### Standard vs Express
![](https://i.imgur.com/taidn8A.png)
- express workflow is much cheaper
    - intended for large num of workflow with small duration
- standard workflows for longer workflows with slower execution rate

### Console
![](https://i.imgur.com/GT9dYSe.png)
![](https://i.imgur.com/rCRkxJ2.png)
- sequence batch func job

![](https://i.imgur.com/r3GweFe.png)
- hello world example in console

![](https://i.imgur.com/pItYBL5.png)
![](https://i.imgur.com/la9s1JM.png)
![](https://i.imgur.com/QiNed3H.png)
![](https://i.imgur.com/eBzhLBE.png)
- can see entire execution event history and how long it took
    - can also see input req/resp


![](https://i.imgur.com/IR44gfK.png)

- when u start new execution, can see visual workflow of it

![](https://i.imgur.com/5Wvwmiv.png)
- can see all execution from your parent state machine
    - only applies to standard state machines

![](https://i.imgur.com/u6VCUUp.png)
- when creating state machines, can choose from templates too

![](https://i.imgur.com/PgZmdam.png)
- eg. retry catch example


- eg. catch example
    - falls to specific state based on error


AWS AppSync
---
- appsync is managed service that uses graphql
    - graphql makes it easy for apps to get exactly the data they need
        - new way of writing api
- includes combining data from 1 or more sources
    - sources include nosql data stores, relational dbs, http apis
    - integrates with dynamodb, aurora, elastisearch and others
    - custom sources with aws lambda
- retrieve data in realtime with websocket or MQTT on websocket
- for mobile apps
    - local data access
    - data sync
    - replacement for cookie to sync
- all starts with uploading 1 graphql schema

![](https://i.imgur.com/igmLKYU.png)
- graphql schema uploaded into appsync
    - client does graphql query on appsync
    - appsync runs it's own resolver
        - resolver might be dynamodb so it does a fetch
    - appsync sends graphql response in json back


![](https://i.imgur.com/77CkhjL.png)
- appsync can work with a multitude of apps
- at it's core, you have
    - graphql schema
    - resolver so it knows how to fetch data
        - have direct integration with
            - dynamodb
            - aurora
            - elastisearch
            - lambda
            - http
- cloudwatch metrics/logs to get info from appsync

### AppSync Security
- 4 ways u can authorise apps to interact with your aws appsync graphql api
    - `API_KEY`
    - `AWS_IAM`
        - iam users/roles/cross acc access
    - `OPENID_CONNECT`
        - openid connect provider/json web token
    - `AMAZON_COGNITO_USER_POOLS`
- for custom doamin and https, use cloudfront in front of appsync

### Console
![](https://i.imgur.com/xOVm7oC.png)
![](https://i.imgur.com/2DogpX2.png)
![](https://i.imgur.com/XoPHChU.png)
![](https://i.imgur.com/hUFLzRE.png)

![](https://i.imgur.com/jiTF8Tg.png)
- graphql schema example

![](https://i.imgur.com/XwuEVM1.png)
- uses 2 dynamodb tables auto created

![](https://i.imgur.com/hXuDOwO.png)
 
![](https://i.imgur.com/QYC9HcE.png)
- queries is how we can start using our apis
    - can test with the orange play btn
- mutations define the diff apis that can be used here

![](https://i.imgur.com/aoeJEW4.png)
- reduce num of queries to backend


![](https://i.imgur.com/Z2p3EgQ.png)

![](https://i.imgur.com/yUz5LUT.png)

- default auth mode
    - can use iam to have user enroll or cross acc access into appsync api

![](https://i.imgur.com/s5G8Nem.png)
![](https://i.imgur.com/PZPhprY.png)


Quiz
---
![](https://i.imgur.com/AAeSh30.png)




###### tags: `AWS Developer Associate` `Notes`