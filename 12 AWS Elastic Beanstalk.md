

12 AWS Elastic Beanstalk
===




## Table of Contents

- [12 AWS Elastic Beanstalk](#12-aws-elastic-beanstalk)
  * [Table of Contents](#table-of-contents)
  * [Overview](#overview)
    + [Typical Architecture: WebApp 3 Tier](#typical-architecture--webapp-3-tier)
    + [Developer Problems on AWS](#developer-problems-on-aws)
  * [Elastic Beanstalk](#elastic-beanstalk)
    + [Beanstalk Components](#beanstalk-components)
    + [Supported Languages](#supported-languages)
    + [Deployment Modes](#deployment-modes)
    + [Deployment Options for Updates](#deployment-options-for-updates)
      - [All at Once](#all-at-once)
      - [Rolling](#rolling)
      - [Rolling with Additional Batches](#rolling-with-additional-batches)
      - [Immutable](#immutable)
      - [Extra - Blue/Green](#extra---blue-green)
      - [Summary](#summary)
    + [Elastic Beanstalk CLI](#elastic-beanstalk-cli)
    + [Elastic Beanstalk Deployment Process](#elastic-beanstalk-deployment-process)
    + [Beanstalk Lifecycle Policy](#beanstalk-lifecycle-policy)
    + [Elastic Beanstalk Extensions](#elastic-beanstalk-extensions)
    + [Beanstalk and CloudFormation](#beanstalk-and-cloudformation)
    + [Beanstalk Cloning](#beanstalk-cloning)
    + [Beanstalk Migration - Load Balancer](#beanstalk-migration---load-balancer)
    + [RDS with Beanstalk](#rds-with-beanstalk)
      - [Beanstalk Migration - Decouple RDS](#beanstalk-migration---decouple-rds)
    + [Elastic Beanstalk with Docker](#elastic-beanstalk-with-docker)
      - [Beanstalk - Single Docker](#beanstalk---single-docker)
      - [Beanstalk - Multi Docker Container](#beanstalk---multi-docker-container)
    + [Beanstalk and HTTPS](#beanstalk-and-https)
    + [Web Server VS Worker Environment](#web-server-vs-worker-environment)
    + [Beanstalk Custom Platform (Advanced)](#beanstalk-custom-platform--advanced-)
    + [Console](#console)
      - [Creating a new Environment](#creating-a-new-environment)
      - [Extra Configuration Options](#extra-configuration-options)
      - [Deployment Modes](#deployment-modes-1)
        * [Blue/green env](#blue-green-env)
      - [Lifecycle Policy](#lifecycle-policy)
      - [Extensions](#extensions)
      - [Cloudformation](#cloudformation)
      - [Cloning](#cloning)
      - [Beanstalk with Docker](#beanstalk-with-docker)
  * [Quiz](#quiz)



Overview
---
### Typical Architecture: WebApp 3 Tier
![](https://i.imgur.com/6xUWnfY.png)

### Developer Problems on AWS
- managing infrastructure
- deploying code
- configuring all db, load balancers etc.
- scaling concerns
- NOTE
    - most webapp have similar architecture (ALB + ASG)
    - enggoal is for code to run consistently across diff apps and environments


Elastic Beanstalk
---
- developer centric view of deploying an app on aws
- uses all component's we've seen
    - EC2, ASG, ELB, RDS etc.
    - all in one view so easy to make sense of
- still have full control over the config
- is free but pay for underlying instances
-  managed service
    -  instance config/os handled by beanstalk
    -  deployment strategy is configurable but performed by beanstalk
    -  just app code is responsibility of developer
-  3 architecture models
    -  single instance deployment - good for dev
    -  LB + ASG - great for production or pre-prod webapps
    -  ASG only - great for non-web apps in prod
        -  Eg. workers etc.
        -  AKA worker tier

### Beanstalk Components
-  3 components
    -  application
    -  application version
        -  ea deployment gets assigned a ver
    -  environment name
        -  Eg. dev, test, prod etc. (free naming)
-  you deploy app versions to envs & can promote app vers to next env
    -  rollback feature to prev app ver
    -  full control over lifecycle of envs

![](https://i.imgur.com/EjMEQ14.png)

### Supported Languages
- Go
- Java SE
- java with tomcat
- .NET on windows server with IIS
- nodejs
- php
- python
- ruby
- packer builder
- single container docker
- multicontainer docker
- preconfigured docker
- NOTE
    - if not supported, you can write your own custom platform (advanced)

### Deployment Modes
![](https://i.imgur.com/o8SdEBD.png)
- single instance as get 1 elastic ip and 1 asg in 1 az
    - very easy to reason out and dns name maps straight to elastic ip
- high avail with load balancer
    - multiple ec2 instances span across multiple az
- NOTE
    - very common exam qns - which deployment mode is btr for which situation


### Deployment Options for Updates
- all at once - deploy all in 1 go
    - fastest
    - though instances arent avail to serve traffic for a bit
        - downtime
- rolling
    - update a few instances (AKA bucket) at a time then move onto next bucket once 1st bucket healthy
- rolling with extra batches
    - like rolling but spins new instances to move batch so old app is still avail
- immutable
    - spins new instances in new ASG
    - deploys ver to these instances then swaps all instances when everything is healthy

#### All at Once
- fastest deployment
- app has downtime
    - Eg. in below, middle is all grey
- great for quick iterations in dev env
- no extra cost

![](https://i.imgur.com/keGBv86.png)

#### Rolling
- app running below capacity
- can set bucket size (to be updated)
- app running both vers simultaneously
- no extra cost
- long deployment

![](https://i.imgur.com/8fP9ost.png)

#### Rolling with Additional Batches
- app running at capacity
    - before not running at capacity since half taken out based on bucket size
- can set bucket size
- app running both vers simultaneously
    - extra batch removed at end of deployment
- small extra cost
- longer deployment
- good for prod

![](https://i.imgur.com/M4vkmjI.png)
- instances running at anytime is always at least 4
    - sometimes running over capacity (hence extra cost)

#### Immutable
- 0 downtime
- new code deployed to new instances on temp ASG
- high cost, double capacity
- longest deployment
- quick rollback in case of failure
    - just terminate new ASG
- great for prod

![](https://i.imgur.com/zBiuLc1.png)
- for new temp asg, beanstalk will 1st take one v2 instance to do initial health check
    - if init health check passes, it continues loading the other instances into the asg
    - then move all instances to main asg

#### Extra - Blue/Green
- not direct feature of elastic beanstalk
- 0 downtime and release facility
    - allows for more testing
- create new stage env & deploy v2 there
    - before all deployment is in same env
    - new env (green) can be validated independently and rollback if have issues
- route53 can be setup using weighted policies to redirect a little bit of traffic to stage env
    - using beanstalk, swap urls when done with env test
- NOTE
    - manual task (not embedded into beanstalk)

![](https://i.imgur.com/n9J6kCT.png)


#### Summary
![](https://i.imgur.com/lO87MgS.png)
- https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.deploy-existing-version.html


### Elastic Beanstalk CLI
- can install extra CLI called EB cli which makes working with beanstalk from cli easier
- basic commands
    - `eb create`
    - `eb status`
    - `eb health`
    - `eb events`
    - `eb logs`
    - `eb open`
    - `eb deploy`
    - `eb config`
    - `eb terminate`
- helpful for automated deployment pipelines
- NOTE
    - not necessary to know commands in exam (more necessary in devops)

### Elastic Beanstalk Deployment Process
- describe dependencies
    - requirements.txt for python
    - package.json for nodejs
- package code as zip
- console - upload zip file
    - this creates new app ver and deploys it
- CLI - create new app ver using CLI (uploads zip)
    - then deploy
    - this is the same as console but in cli
- elastic beanstalk will deploy the zip on ea ec2 instance, resolve dependencies and start app
- NOTE
    - the uploaded zip also gets uploaded into amazon s3, then refers the s3 bucket from beanstalk interface

### Beanstalk Lifecycle Policy
- elastic beanstalk can store at most 1000 app vers
    - if dont remove old vers, wont be able to deploy anymore
- to phase out old vers, use __lifecycle policy__
    - based on time
        - old vers removed
    - based on space
        - when too many vers
- vers that currently used wont be deleted
- option to not delete the src bundle in s3 to prevent data loss

### Elastic Beanstalk Extensions
- zip file containing our code must be deployed into elastic beanstalk
- all params set in UI can be configured with code using files
- requirements
    - in `.ebextensions/` dir in root src of src code
    - must be in YAML/JSON format
    - must end with .config extensions 
        - Eg. logging.config
    - able to modify some default settings using `option_settings`
    - ability to add res such as RDS, elasticache, dynamodb etc.
- res managed by `.ebextensions` get deleted if env goes away


### Beanstalk and CloudFormation
- under the hood, beanstalk relies on cloudformation
    - cloudformation used to provision other aws services
- use case
    - define cloudformation res in `.ebextensions` to provision elasticache, s3 bucket or anything else


### Beanstalk Cloning
- clone env into new env with exact same config
- useful for deploying test ver of app
- all res and config preserved
    - load balancer type and config
    - rds db type
        - but data not preserved
    - env vars
- after cloning env, can change settings


### Beanstalk Migration - Load Balancer
- after creating elastic beanstalk env, cannot change elb type
    - can only change its config
- to migrate
    - create new env with same config except lb
        - you CANNOT clone for this as it also copies lb config
    - deploy your app onto new env
    - perform CNAME swap or route53 update

![](https://i.imgur.com/x5iAv88.png)

### RDS with Beanstalk
- rds can be provisioned with beanstalk, which is great for dev/test
    - __not great for prod__ as db lifecycle is tied to beanstalk env lifecycle
- best for prod is to separately create rds db and provide our eb app with connection string

![](https://i.imgur.com/LNqERIx.png)

#### Beanstalk Migration - Decouple RDS
- create snapshot of rds db as a safeguard
- go to rds console and protect rds db from deletion
    - prevent it from being deleted no matter what
- create new beanstalk env w/o rds
    - pt app to existing rds using env var (for example)
- perform CNAME swap (blue/green) or route53 update
    - confirm if working
- terminate old env
    - rds wont be deleted since enabled deletion protection
- delete cloudformation stack in `DELETE_FAILED` state

![](https://i.imgur.com/qiBqozt.png)

### Elastic Beanstalk with Docker
#### Beanstalk - Single Docker
- run app as single docker container
- either provide
    - dockerfile
        - beanstalk will build and run docker container
    - dockerrun.aws.json (v1)
        - describe whr alrdy built docker img is (can be on ecr repo/docker hub etc.)
            - image
            - ports
            - volumes
            - logging
            - etc.
- beanstalk in single docker container dont use ecs
    - just use docker on ec2

#### Beanstalk - Multi Docker Container
- multi docker helps run multiple containers per ec2 instance in eb
- will create
    - ecs cluster
    - ec2 instances configured to use ecs cluster
    - load balancer in high availability mode
    - task definitions and execution
- requires config `dockerrun.aws.json (v2)` at root of src code
- `dockerrun.aws.json` is used to generate the ecs task definition
- your docker imgs must be pre-built and stored in ecr for example

![](https://i.imgur.com/kw5hbtb.png)


### Beanstalk and HTTPS
- beanstalk with https
    - idea - load ssl cert onto lb
        - can be done from console
            - eb console, lb config
        - can be done from code
            - .ebextensions, securelistener-alb.config
    - ssl cert can be provisioned using ACM (aws cert manager) or cli
    - must config security grp rule to allow incoming port 443 (https port)
- beanstalk redirect http to https
    - config instances to redirect http to https
        - https://github.com/awsdocs/elastic-beanstalk-samples/tree/master/configuration-files/aws-provided/security-configuration/https-redirect
    - OR config app load balancer (ALB only) with rule for redirecting
    - make sure health checks not redirected so they keep giving 200 OK

### Web Server VS Worker Environment
- if app performs tasks that are too long to complete, offload these tasks to dedicated __worker env__
- decoupling your app into 2 tiers is common
    - eg. processing vid, generating zip file etc.
- can define periodic tasks in a file `cron.yaml`

![](https://i.imgur.com/XFwKtIE.png)
- send msgs to sqs queue and worker reading from queue

### Beanstalk Custom Platform (Advanced)
- custom platforms very advanced, they allow to define from scratch
    - OS
    - additional software
    - scripts that beanstalk runs on these platforms
- use case
    - app lang is incompatible with beanstalk and dont use docker
- to create own platform
    - define AMI using `platform.yaml` file
    - build that platform using packer software (open src tool to create AMIs)
- custom platform VS custom image (AMI)
    - custom img is to tweak existing beanstalk platform
        - Eg. python, nodejs, java
    - custom platform is to create entirely new beanstalk platform





### Console
![](https://i.imgur.com/MebiDHI.png)
![](https://i.imgur.com/syEbm4k.png)
- choose platform and platform branch

![](https://i.imgur.com/e1ncV12.png)


![](https://i.imgur.com/GReZLvH.png)
- option to upload your own code

![](https://i.imgur.com/36v1N5X.png)
- click on URL to access webapp

![](https://i.imgur.com/Fyj4663.png)
- events tab - show all events that has happened in your env

![](https://i.imgur.com/PFGufor.png)

![](https://i.imgur.com/Dt6E5my.png)
- s3 bucket auto created that holds app code and configs
    - also created EIP, sec grp and ec2

![](https://i.imgur.com/RXV4KSw.png)
- show configs

![](https://i.imgur.com/6mS9ACt.png)
![](https://i.imgur.com/d8xoXcc.png)

- logs tab

![](https://i.imgur.com/9j6Lwks.png)
- health tab - show info on all running ec2 instances
    - Eg. cpu utilisation etc.

![](https://i.imgur.com/1OtBwaq.png)
- monitoring tab

![](https://i.imgur.com/qBzJT5T.png)
- beanstalk env (separate from beanstalk webapp)

![](https://i.imgur.com/TAmHbQv.png)
![](https://i.imgur.com/cJaHAfR.png)
- create more envs from app lvl

![](https://i.imgur.com/TLqU7lH.png)
- view app vers and saved configs from app lvl

#### Creating a new Environment
![](https://i.imgur.com/DZ8DmTA.png)
- 2 types of env
    - web server
    - worker

![](https://i.imgur.com/uuBAuOI.png)
- append env mode at back (Eg. prod)

![](https://i.imgur.com/vGO8jSv.png)
- choose platform and app code (like above)


#### Extra Configuration Options
![](https://i.imgur.com/hq3qC57.png)
![](https://i.imgur.com/nSmoOXG.png)


- extra configurations for env 
    - click the extra config button instead of create new env at bottom

![](https://i.imgur.com/xXp1i6u.png)


![](https://i.imgur.com/VvnLCL5.png)
- software
    - aws xray daemon, log storage, cloudwatch logs etc.

![](https://i.imgur.com/d0zjd7u.png)
- instances
    - specify root vol
    - type of vol
    - sizes
    - sec grps

![](https://i.imgur.com/ZSAGn6x.png)
- capacity
    - config ASGs
    - instance types of asg etc.
    - scaling triggers

![](https://i.imgur.com/voamq8B.png)
- load balancer
    - config load balancer
        - alb, clb, nlb etc.
    - lb processes, rules, logs
- NOTE
    - once you choose your lb, you cannot change it for the entire lifecycle of your env 

![](https://i.imgur.com/IIBSxOh.png)
- security
    - defines service role for beanstalk
    - keypairs for ec2 instances

![](https://i.imgur.com/TmIGR1N.png)
- database
    - create and config rds db
- NOTE
    - if delete your beanstalk env, rds db will get deleted too
        - sometimes good to have rds outside of beanstalk

#### Deployment Modes
![](https://i.imgur.com/NVcLjoF.png)
- under configuration > rolling updates and deployments

![](https://i.imgur.com/ScwgfU5.png)
- deployment policy to choose deployment mode
- batch size - x amt of instances taken down at a time

![](https://i.imgur.com/jUBWFcS.png)
- out of scope for this exam

![](https://i.imgur.com/pTTxoK0.png)

- https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/tutorials.html
    - can use sample code here to do update demo

![](https://i.imgur.com/IcdidDd.png)
- upload new code in a zip file

![](https://i.imgur.com/0yvoF6D.png)
- see recent events for logs

![](https://i.imgur.com/CRm4ZUs.png)
- go to health tab to check health of instances

![](https://i.imgur.com/GFYyqHB.png)
- new asg created for immutable deployment mode

##### Blue/green env
![](https://i.imgur.com/0hfktvt.png)
- swap env urls between url


![](https://i.imgur.com/sLehSKO.png)
- swap might take awhile especially if prev dns record is cached

#### Lifecycle Policy
![](https://i.imgur.com/pwtk3u7.png)
- go app vers > settings

![](https://i.imgur.com/cJJtGyG.png)

![](https://i.imgur.com/4Vl7b7V.png)
- can delete or retain ver in s3
- specify service role to allow beanstalk to actually remove the lifecycle, vers and file from s3


#### Extensions
![](https://i.imgur.com/XIY9Nuq.png)
- even if ext is .config, it still has a YAML format
- define env vars in option settings
    - env vars can be used for rds etc.


![](https://i.imgur.com/SeVXftZ.png)
- go to configuration > software > edit and scroll down to see env vars

![](https://i.imgur.com/5fiU4Wk.png)


#### Cloudformation
![](https://i.imgur.com/nKsBzSo.png)
- view your stacks

![](https://i.imgur.com/qoaXIJ7.png)
- 1 stack for env, other for prod

![](https://i.imgur.com/AbSO4Um.png)
- view cloudformation stack template
    - covered ltr in course

![](https://i.imgur.com/1x5lESa.png)
- see res created for us in res tab

#### Cloning
![](https://i.imgur.com/Gp9Ubd7.png)
- env actions > clone

![](https://i.imgur.com/7Tb3XLn.png)
- can change stuff like env name and url
    - options of change are limited as we're just recloning env


#### Beanstalk with Docker
![](https://i.imgur.com/Venz4S5.png)
- when creating new env, choose docker in platform
    - can use docker sample app zip to demo

![](https://i.imgur.com/rDOfjJp.png)
- is a task definition

![](https://i.imgur.com/BBkMUq2.png)
- ecs cluster auto created if choose multi docker

![](https://i.imgur.com/MoeD9L7.png)


Quiz
---
![](https://i.imgur.com/kcEqHHr.png)
![](https://i.imgur.com/Gc9VdRG.png)
![](https://i.imgur.com/2NDQ3Zt.png)
- https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features-managing-env-tiers.html

![](https://i.imgur.com/Dx6EwSk.png)




###### tags: `AWS Developer Associate` `Notes`