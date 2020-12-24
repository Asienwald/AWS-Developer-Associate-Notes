

11 ECS, ECR & Fargate + Docker in AWS
===




## Table of Contents

- [11 ECS, ECR & Fargate + Docker in AWS](#11-ecs--ecr---fargate---docker-in-aws)
  * [Table of Contents](#table-of-contents)
  * [Docker](#docker)
    + [Where are Docker Images Stored?](#where-are-docker-images-stored-)
    + [Docker VS Virtual Machines](#docker-vs-virtual-machines)
    + [Getting Started with Docker](#getting-started-with-docker)
    + [Docker Containers Management](#docker-containers-management)
  * [AWS ECS (Elastic Container Service)](#aws-ecs--elastic-container-service-)
    + [ECS Clusters](#ecs-clusters)
    + [ECS Task Definitions](#ecs-task-definitions)
    + [ECS Service](#ecs-service)
      - [ECS Service with Load Balancer](#ecs-service-with-load-balancer)
    + [Console](#console)
      - [Task Definitions](#task-definitions)
      - [Service](#service)
      - [ECS Service with Load Balancer](#ecs-service-with-load-balancer-1)
  * [ECR (Elastic Container Registry)](#ecr--elastic-container-registry-)
    + [Console](#console-1)
  * [Fargate](#fargate)
    + [Console](#console-2)
  * [More ECS](#more-ecs)
    + [ECS IAM Roles Deep Dive](#ecs-iam-roles-deep-dive)
    + [ECS Tasks Placement](#ecs-tasks-placement)
      - [ECS Task Placement Process](#ecs-task-placement-process)
      - [ECS Task Placement Constraints](#ecs-task-placement-constraints)
    + [ECS Task Placement Strategies](#ecs-task-placement-strategies)
      - [Binpack](#binpack)
      - [Random](#random)
      - [Spread](#spread)
      - [Hybrid](#hybrid)
    + [ECS Service Auto Scaling](#ecs-service-auto-scaling)
      - [ECS Cluster Capacity Provider](#ecs-cluster-capacity-provider)
    + [Console](#console-3)
      - [ECS IAM Roles](#ecs-iam-roles)
      - [Task Placements](#task-placements)
      - [Capacity Provider](#capacity-provider)
  * [Summary](#summary)
  * [Quiz](#quiz)




Docker
---
- docker is software dev platform to deploy apps
    - apps packaged in containers that can be run on any os
- apps run the same regardless whr they're run
    - any machine
    - no compat issues
    - predictable behaviour
    - less work
    - easier to maintain and deploy
        - cuz standardised
    - works with any lang/OS/tech

![](https://i.imgur.com/2XIFJbc.png)
- can run multiple docker containers
    - even tho run on same machine, they dont interact with ea other unless we tell them to
        - dont impact ea other
    - can scale them since on same server


### Where are Docker Images Stored?
- docker imgs stored in docker repositories
    - 2 famous repos
- public - docker hub https://hub.docker.com/
    - find base imgs for many tech or os
        - ubuntu
        - mysql
        - nodejs, java etc.
- private - __amazon ECR (elastic container registry)__

### Docker VS Virtual Machines
- docker is sort of virtualisation tech but not exactly
    - res are shared with host
        - can run many containers on 1 server

![](https://i.imgur.com/QQXvCkn.png)
- docker has no hypervisor
    - once installed docker daemon, can just run a container
- since docker is lightweight and well made, can run a lot more containers than VMs on your machine


### Getting Started with Docker
- download docker at https://www.docker.com/get-started

![](https://i.imgur.com/htZ4JrO.png)
- build dockerfile
    - run docker img to get docker container


### Docker Containers Management
- to manage containers, need container management platform
- 3 choices
    - ecs
        - amazon's own platform
    - fargate
        - amazon's own serverless platform
    - eks
        - amazon's managed kubernetes (open source)
        - not in exam


AWS ECS (Elastic Container Service)
---
### ECS Clusters
- ecs clusters are logical grping of ec2 instances
- ec2 instances run ecs agent (docker container)
    - ecs agent registers instance to ecs cluster
- ec2 instances run special AMI, made specifically for ecs

![](https://i.imgur.com/9liDgL8.png)


### ECS Task Definitions
- task definitions are metadata in json form to tell ecs how to run docker container
- contains crucial info around
    - img name
    - port binding for container and host
    - memory and cpu required
    - env variables
    - networking info
    - iam role
    - logging configuration
        - Eg. cloudwatch

![](https://i.imgur.com/aS27e79.png)

### ECS Service
- ecs services help define how many tasks shld run and how they shld be run
- ensure that num of tasks desired is running across fleet of ec2 instances
    - will also help if do updates to tasks
- can be linked to elb/nlb/alb if needed
- created at cluster lvl

#### ECS Service with Load Balancer
![](https://i.imgur.com/JeSEwHF.png)
- change task def to specify container port and have random host port
    - do this many time
    - everytime httpd service is launched, new random port num created
- app load balancer will route traffic to random ports and spread load to diff containers
    - has feature called dynamic port forwarding





### Console
![](https://i.imgur.com/eTXRx7N.png)

- do not press get started cuz it will auto use fargate (fargate it gr8 but v new so might not come out in exam?)
    - press create cluster

![](https://i.imgur.com/aoet1eX.png)
![](https://i.imgur.com/n8CjGRI.png)
- shld create a keypair for ecs too

![](https://i.imgur.com/Mt4z5oG.png)
- add vpc, subnets and sec grp

![](https://i.imgur.com/sgjbML5.png)
- need iam role so ec2 instance can register to ecs

![](https://i.imgur.com/er9bRro.png)
- when created ecs cluster, asg auto created too

![](https://i.imgur.com/AWSFduG.png)
- AMI id
- IAM profile
    - created by ecs, called `ecsInstanceRole`

![](https://i.imgur.com/9R87PwX.png)
- ec2 user data
    - ecs config file contains var ecs cluster = cluster demo
    - when instance starts, register cluster named cluster demo
    - change this bit if have multiple clusters

![](https://i.imgur.com/0Q3pWds.png)
- security grp auto created too

![](https://i.imgur.com/cvEm8a3.png)
- iam role on ec2 created for ecs

![](https://i.imgur.com/nx7JmOZ.png)
- docker agent helps register

![](https://i.imgur.com/N9MEsyv.png)
- this specific process is what's registering our agent 

![](https://i.imgur.com/5RW7Xqa.png)
- use `docker logs <container id>` to see logs

#### Task Definitions
![](https://i.imgur.com/sE2oZsk.png)

![](https://i.imgur.com/91umaLH.png)
- task assigned can have an iam role
- network mode
    - bridged on linux
    - NAT on windows

![](https://i.imgur.com/a48icQ7.png)
- task execution role
    - role created to allow ecs to create the task
    - is just info

![](https://i.imgur.com/VqFNGLn.png)
- mem and ram assigned to task

![](https://i.imgur.com/ms6g2Iv.png)

- need to choose docker container from docker hub
- if specify image as `<name>:<ver tag>`, know to take img directly from dockerhub

![](https://i.imgur.com/k88bIkf.png)
- port mapping v impt
    - must go dockerhub and find what port the img is using
- there's still a lot of options left to choose from

![](https://i.imgur.com/7FxskLV.png)
- can view json form of task def

#### Service
![](https://i.imgur.com/gTP1iwE.png)
![](https://i.imgur.com/5Rh2DDf.png)
- set min healthy percent to 0
    - enable us to do rolling restarts

![](https://i.imgur.com/7nz1Gsn.png)
- codedeploy can be used to deploy to ecs too

![](https://i.imgur.com/z1mJ4ZI.png)
- task placement
    - defining strategy of how to place tasks on diff ecs instances

![](https://i.imgur.com/5nsXBRn.png)


![](https://i.imgur.com/2Mkhika.png)
- add inbound rule in cluster sec grp for docker container

![](https://i.imgur.com/97HNWMh.png)
- can set to scale ec2 instances for cluster
    - or if dont see option can just access asg and change desired count


#### ECS Service with Load Balancer
![](https://i.imgur.com/RVKxuYH.png)
- leave host port empty to randomise it

![](https://i.imgur.com/xMolACN.png)

![](https://i.imgur.com/y39LFQY.png)
- alb
    - allows dynamic port mapping - feature that is not in clb (static port mapping)

![](https://i.imgur.com/Bpa2Xe0.png)
- though need to create load balancer in ec2 console to link to ecs

![](https://i.imgur.com/MMBd7vj.png)
- security grp must allow all traffic from ecs load balancer
- setup p complicated, might want to rewatch the vid ECS Service with Load Balancers



ECR (Elastic Container Registry)
---
- so far we using docker imgs from docker hub (public)
    - ecr is private docker img repo
    - access controlled through iam
        - perms errors => policy
- authenticating ecr using CLI
    - aws cli v1 login cmd
        - may be asked in exam
        - `$(aws ecr get-login --no-include-email --region eu-west-1)`
            - have `$()` cuz output of cmd has to be executed
    - aws cli v2 login cmd
        - newer, may also be asked in exam
        - relies on pipes `|`
        - `aws ecr get-login-password --region eu-west-1 | docker login --username AWS --password-stdin 1234567890.dkr.ecr.eu-west-1.amazonaws.com`
            - 1st part give login passwd and pass into second part
- docker push & pull
    - `docker push 1234567890.dkr.ecr.eu-west-1.amazonaws.com/demo:latest`
        - just push full img url
    - `docker pull 1234567890.dkr.ecr.eu-west-1.amazonaws.com/demo:latest`

### Console
- install docker first
    - `docker --version` to check install

![](https://i.imgur.com/7xPDrny.png)
- breakdown of above docker file
    - use `httpd:2.4` img from docker hub
    - gonna install a few things on it
    - add some html to it
    - add a script
    - launch that script after giving it executing perms

![](https://i.imgur.com/dSI9BOW.png)
- bootstrap script
    - going to run and pickup some metadata info arnd ecs and display to us

![](https://i.imgur.com/gVGfK5d.png)
- build the docker img with this cmd
    - `docker build -t <img> .`
- before we can use it in ecs, we have to push it to ecr

![](https://i.imgur.com/ebemjgs.png)
- create new repo in ecr

![](https://i.imgur.com/6BWF1d0.png)

![](https://i.imgur.com/Mhu8SoY.png)
- can view push commands from repo console

![](https://i.imgur.com/12TmIsj.png)
- login into docker and ecr

![](https://i.imgur.com/fDHsDtT.png)
- can push img into docker

![](https://i.imgur.com/44mWgMW.png)
- change img name to ecr img name in ecs task def

![](https://i.imgur.com/ZlqAfsY.png)
- when u update task def, it will slowly shutdown the prev task def and replace it with the new one
    - might take a while

![](https://i.imgur.com/EK1nK0l.png)
- alb has a deregistration delay of 300secs (5mins) for the registered targets to be drained
    - or u can just stop it yourself


Fargate
---
- when launching ecs cluster, need to create ec2 instances
    - if need scale, need to add ec2 instances
- with fargate, its all serverless
    - dont provision ec2 instances, just create task definitions & aws will run container for us
    - to scale, just inc task num
        - simple
        - no more ec2 :)
    - if want run docker container in cloud, no need to think abt ec2, just use fargate

### Console
![](https://i.imgur.com/vm2bA3P.png)
- creating cluster with fargate is v simple

![](https://i.imgur.com/Ir0UQ0s.png)
- create 1st service

![](https://i.imgur.com/Ts5fpR5.png)
- create task def with fargate
    - set task mem and cpu sizes
    - add container for task def
    - just like ecs hands on

![](https://i.imgur.com/mgny67z.png)
- only need add container port
    - fargate smart enough to do dynamic port mapping

![](https://i.imgur.com/7znbr7i.png)
- create fargate service
    - process p similar to above

![](https://i.imgur.com/y5pn6G1.png)
- tasks are not linked to any containers cuz its serverless
    - no ec2 instances needed


More ECS
---
### ECS IAM Roles Deep Dive
- ec2 instance profile
    - used by ecs agent
    - ecs agent use role to make api calls to ecs service
    - send container logs to cloudwatch logs
    - pull docker img from ecr
- ecs task role
    - allow ea task to have specific role with min perms
    - use diff roles for diff ecs services u run
    - task role defined in task definition

![](https://i.imgur.com/MTeVjil.png)

### ECS Tasks Placement
- when task of type ec2 launched, ecs must determine whr to place it with constraints of cpu, memory and avail port
    - similarly, when service scales in, ecs needs to determine which task to terminate
    - to assist with this, can define a __task placement strategy__ and __task placement constraints__
- NOTE
    - this is only for ecs and ec2, not fargate

![](https://i.imgur.com/oFhL3BU.png)


#### ECS Task Placement Process
- task placement strategies are best effort
- when amazon ecs places tasks, uses following process to select container instances
    - identify instances that satisfy cpu, memory and port requirements in task def
    - identify instances that satisfy task placement constraints
    - identify instances that satisfy task placement strategies
    - select instances for task placement and place task thr

#### ECS Task Placement Constraints
![](https://i.imgur.com/wPxzr9A.png)

- `distinctInstance` - place ea task on diff container instance
    - no 2 tasks on same instance

![](https://i.imgur.com/zMNsm8S.png)
- `memberOf` - places task on instances that satisfy an expression
    - uses cluster query lang (p advanced)
    - Eg. instancetype must be type t2



### ECS Task Placement Strategies
#### Binpack
- binpack
    - place tasks based on least avail amt of cpu or memory
    - minimises num of instances in use
        - cost savings

![](https://i.imgur.com/VIv1yil.png)
- packs all containers into one ec2 instance until it cannot hold anymore
    - hence called binpack (packing containers tgt)
    - minimises num of ec2 instances in use and maximise utilisation of 1 ec2 instance at a time
![](https://i.imgur.com/3wSluRC.png)

#### Random
- random
    - place task randomly

![](https://i.imgur.com/3tWSceP.png)
- tasks place randomly between the 2 ec2 instances
    - no logic to it
![](https://i.imgur.com/kASGTTa.png)

#### Spread
- spread
    - place task evenly based on specified value
        - Eg. `instanceId`, `attribute:availability-zone`

![](https://i.imgur.com/uR9KqKH.png)
- for this example, it spreads based on ea az then start all over again
![](https://i.imgur.com/IZ5QNXN.png)

#### Hybrid
![](https://i.imgur.com/3Ip7O8C.png)
![](https://i.imgur.com/sfqFgOv.png)


### ECS Service Auto Scaling
- cpu and ram tracked in cloudwatch at ecs service lvl
    - __target tracking__
        - target specific avg cloudwatch metric
        - Eg. cpu transition shld be 60% across ecs service
    - __step scaling__
        - scale based on cloudwatch alarms
    - __scheduled scaling__
        - based on predictable changes
- ecs service scaling (task lvl) __NOT EQUAL__ to ec2 auto scaling (instance lvl)
    - if scale up/down your ecs service, doesnt mean ec2 asg at instance lvl will scale up/down
        - this makes ecs auto scaling p difficult at service lvl
- fargate auto scaling is much easier to setup
    - because serverless


#### ECS Cluster Capacity Provider
- capacity provider is used in association with cluster to determine infrastructure that task runs on
    - for ecs and fargate users, `FARGATE` and `FARGATE_SPOT` capacity providers are added automatically
    - for amazon ecs on ec2, need to associate capacity provider with an asg
        - asg add ec2 instances when needed
- when run task or service, define a capacity provider strat to prioritise which provider to run
    - this allows capacity provider to auto provision infrastructure for u

![](https://i.imgur.com/5K2qd8V.png)
- both ec2 instances are full
    - create new task
        - if task assigned to cluster capacity provider, will auto launch new ec2 instance running on ecs
        - or if use fargate capacity container, will go on fargate container




### Console
#### ECS IAM Roles
![](https://i.imgur.com/3Cm9eAL.png)
![](https://i.imgur.com/eS31Sh4.png)
- choose use case for ecs iam role

![](https://i.imgur.com/YEVOF7u.png)
- choose your iam policy for the role

![](https://i.imgur.com/Apth8Bu.png)
- ecs task role
    - if go to trust r/s tab, it can be endorsed by ecs tasks
        - is the role that gets assigned to specific tasks/services


#### Task Placements
![](https://i.imgur.com/0hhGqJz.png)
- config when creating service

![](https://i.imgur.com/wEj5ADV.png)
- custom task placements + constraints


#### Capacity Provider
![](https://i.imgur.com/TAN5aVV.png)
![](https://i.imgur.com/sgdnMUm.png)
- select asg
    - set target capacity
        - Eg. 70% = if 70% capacity of ec2 instances rched then will launch more instances

![](https://i.imgur.com/zw9Hd7q.png)
- can set capacity provider at service config

![](https://i.imgur.com/laQiJ0H.png)
- set autoscaling for service
    - set scaling policy type
- might want to rewatch the handson again

Summary
---
![](https://i.imgur.com/GzMOkG8.png)
![](https://i.imgur.com/uUTPA2E.png)
![](https://i.imgur.com/efxR1yG.png)
![](https://i.imgur.com/IcWoQ7F.png)
![](https://i.imgur.com/DiX9SW3.png)
- service auto scaling does not mean your cluster will get bigger


Quiz
---
![](https://i.imgur.com/yjYPk6t.png)
![](https://i.imgur.com/rK0pYdh.png)
![](https://i.imgur.com/c6jEwhH.png)
![](https://i.imgur.com/9wn2bC5.png)




###### tags: `AWS Developer Associate` `Notes`