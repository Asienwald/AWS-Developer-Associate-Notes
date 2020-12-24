

25 Other AWS Services - ACM, SWF, SES, DBs
===




## Table of Contents

- [25 Other AWS Services - ACM, SWF, SES, DBs](#25-other-aws-services---acm--swf--ses--dbs)
  * [Table of Contents](#table-of-contents)
  * [AWS Certificate Manager (ACM)](#aws-certificate-manager--acm-)
    + [Console](#console)
  * [AWS Simple Workflow Service (SWF)](#aws-simple-workflow-service--swf-)
  * [AWS SES](#aws-ses)
  * [Database Summary](#database-summary)



AWS Certificate Manager (ACM)
---
- to host public ssl certs in aws, you can
    - buy own and upload using cli
    - have acm provision and renew public ssl certs for you
        - free
- acm loads ssl certs on following integrations
    - load balancers
        - including ones created by EB
    - cloudfront distributions
    - apis on api gateways
- manually managing ssl certs is a pain so acm is a great service to leverage
    - eg. when certs expire need to reload and transfer security

![](https://i.imgur.com/8ewbSb0.png)
- users connect to our alb
    - alb has ssl termination
        - when users using public internet talking via https terminate, the request is transformed into http
- ACM provisions and maintain cert on the alb
- when https req made to alb, alb makes priv aws http request (not https) to ec2
    - thanks to ssl termination, only alb deals with https not our ec2 instances
    - less cpu costs on ec2 from decrypting/encrypting
    - also dont have to manage ssl certs on ec2 instances

### Console
![](https://i.imgur.com/6umr18c.png)
- private CA is out of scope for exam

![](https://i.imgur.com/wlAmbtJ.png)
![](https://i.imgur.com/PXEBOOk.png)
![](https://i.imgur.com/92ZG67Y.png)
![](https://i.imgur.com/hBDfvYn.png)

![](https://i.imgur.com/vYlhrhe.png)

- can click create record in route 53

![](https://i.imgur.com/LlvSoBP.png)
- wait awhile for cert to be issued

![](https://i.imgur.com/xU9v3RA.png)
- can config alb listener with the ssl cert in beanstalk

![](https://i.imgur.com/cODiNd2.png)


AWS Simple Workflow Service (SWF)
---
- coordinate work amongst apps
- code runs on ec2
    - not serverless
- 1 year max runtime
- concept of activity step and decision step
    - has builtin human intervention step
    - eg. order fulfillment from web to warehouse to delivery
- step funcs is recommended to be used for new apps except
    - if need external signals to intervene in processes
    - if need child processes that returns values to parent processes





AWS SES
---
- send emails using
    - smtp interface
    - aws sdk
- abi to send email, integrates with
    - s3
    - sns
    - lambda
- integrated with iam for allowing to send emails


Database Summary
---
- rds - relational databases, OLTP (online transation processing)
    - postgresql, mysql, oracle etc.
    - aurora + aurora serverless
    - provisioned db
- dynamodb - nosql db
    - managed, key value, document
    - serverless
    - can provison rcu and wcu with auto scaling or use on-demand and not care about provisioning
- elasticache - in memory db
    - redis/memcached
    - cache capability
- redshift - OLAP, analytics processing
    - data warehousing/data lake
    - analytic queries
- neptune - graph db
- db migration service (DMS)
- documentdb - managed mongodb for aws




###### tags: `AWS Developer Associate` `Notes`