

02 AWS Fundamentals P2
===



## Table of Contents

- [02 AWS Fundamentals P2](#02-aws-fundamentals-p2)
  * [Table of Contents](#table-of-contents)
  * [Scalability & High Availability](#scalability---high-availability)
    + [Vertical Scalability](#vertical-scalability)
    + [Horizontal Scalability](#horizontal-scalability)
    + [High Availability](#high-availability)
    + [High Availability & Scalability for EC2](#high-availability---scalability-for-ec2)
  * [Elastic Load Balancing](#elastic-load-balancing)
    + [What is Load Balancing?](#what-is-load-balancing-)
    + [Why use ELB?](#why-use-elb-)
    + [Health Checks](#health-checks)
    + [Load Balancer Security Groups](#load-balancer-security-groups)
  * [Types of Load Balancers](#types-of-load-balancers)
    + [Classic Load Balancers (v1)](#classic-load-balancers--v1-)
      - [Console](#console)
    + [Application Load Balancer (v2)](#application-load-balancer--v2-)
      - [Target Groups](#target-groups)
      - [Console](#console-1)
    + [Network Load Balancer (v2)](#network-load-balancer--v2-)
      - [Console](#console-2)
  * [More on Load Balancers](#more-on-load-balancers)
    + [Load Balancer Stickiness](#load-balancer-stickiness)
      - [Console](#console-3)
    + [Cross-Zone Load Balancing](#cross-zone-load-balancing)
      - [Settings for Different LBs](#settings-for-different-lbs)
      - [Console](#console-4)
    + [SSL Certificates](#ssl-certificates)
      - [SSL/TLS Basics](#ssl-tls-basics)
      - [Using SSL Certs in LBs](#using-ssl-certs-in-lbs)
      - [Server Name Indication (SNI)](#server-name-indication--sni-)
      - [Note for Different LBs](#note-for-different-lbs)
      - [Console](#console-5)
    + [Connection Draining](#connection-draining)
  * [Auto Scaling Group (ASG)](#auto-scaling-group--asg-)
    + [ASG Attributes](#asg-attributes)
    + [Auto Scaling Alarms](#auto-scaling-alarms)
    + [Auto Scaling New Rules](#auto-scaling-new-rules)
    + [Auto Scaling Custom Metric](#auto-scaling-custom-metric)
    + [Scaling Policies](#scaling-policies)
    + [Scaling Cooldowns](#scaling-cooldowns)
    + [Console](#console-6)
      - [Scaling Policies + Cooldown](#scaling-policies---cooldown)



Scalability & High Availability
---
- scalability - app/system can handle greater loads by adapting
- 2 kinds
    - vertical scalability
    - horizontal scalability/elasticity
- scalability linked but diff than high availability

### Vertical Scalability
- increase size of instance
    - Eg. app runs on t2.micro. vert scale > run on t2.large
- very common for non distributed systems like db
    - RDS, elasticache are services that can scale vert
- usually limit to how much u can vert scale
    - AKA hardware limit

### Horizontal Scalability
- increase num of instances/systems for app
- implies distributed systems
- very common for web apps/modern apps
- easy to hori scale thanks to cloud offerings like ec2

### High Availability
- goes hand in hand with hori scaling
- means running app/system in at least 2 datacenters (AZs)
- goal is to survive data center loss
- can be passive
    - Eg. for RDS multi AZ
- can be active
    - for hori scaling

### High Availability & Scalability for EC2
- vert scaling - increase instance size
![](https://i.imgur.com/uffh0UY.png)
- hori scaling - increase num of instances
    - auto scaling grp
    - load balancer
- high avail - run instances for same app across multi AZ
    - auto scaling grp multi az
    - load balancer multi az


Elastic Load Balancing
---
### What is Load Balancing?
- load balancers - servers that forward internet traffic to multiple servers (ec2 instances) downstream

![](https://i.imgur.com/78AYjSy.png)

- why use?
    - spread load across many downstream instances
    - expose single point of access (dns) to app
        - dont need know what happening at backend just need know pt of access
    - seamlessly handle failures of downstream instances
    - do regular health checks to instances
    - provides SSL termination (HTTPS for websites
    - enforce stickiness with cookies
    - high avail across zones
        - load balancer can spread across multiple az
    - separate pub traffic from priv traffic

### Why use ELB?
- ELB (EC2 load balancer) is __managed load balancer__
    - aws guarantees it will be working
    - aws takes cares of upgrades, maintenance & high avail
    - aws provides few config knobs
- costs less to setup own load balancer but needs more effort
- integrated with many aws offerings/services
- NOTE
    - lb can scale but not instantaneously
        - contact aws for "warm up"
    - troubleshooting
        - 4xx err is client induced errors
        - 5xx errors are app induced errors
        - lb errors 503 means at capscity or no registered target
        - if lb cant connect to app, check security grps
    - monitoring
        - elb access logs logs all access reqs
            - so can debug per req
        - cloudwatch metrics will give aggregate stats
            - Eg. connections count

### Health Checks
- crucial for load balancers
    - enable load balancer to know if instances it forwards traffic to are avail to reply to requests
- health check done on a port & route
    - `/health` is common
    - if response not 200 (OK), instance is unhealthy

![](https://i.imgur.com/5y2SjLj.png)

### Load Balancer Security Groups
![](https://i.imgur.com/LryNBbi.png)
- take note of sources allowed


Types of Load Balancers
---
- 3 kinds of managed load balancers
    - classic load balancer - v1, old gen 2009
        - support http, https, tcp
    - app load balancer - v2, new gen 2016
        - support http, https, websocket
    - network load balancer - v2 new gen 2017
        - support tcp, tls (secure tcp) & udp
- recommended to use newer/v2 gen load balancers as more features
- can setup __internal__ (private) or __external__ (public) ELBs

### Classic Load Balancers (v1)
- support tcp (layer 4), http & https (layer 7)
- health checks are tcp or http based
- fixed hostname xxx.region.elb.amazonaws.com

![](https://i.imgur.com/DCZ6rYV.png)

#### Console
![](https://i.imgur.com/iyVCXeb.png)
![](https://i.imgur.com/BYk8x2c.png)
- internal means prob not able to publicly access it
    - configed to listen on http port 80
- also config security grp
    - by default allow conn on listening port

![](https://i.imgur.com/GrcmUPP.png)
- NOTE
    - CLB will have its own dns name for people to access
    - rmb to config ec2 sec grp to only allow http port to access from source lb


### Application Load Balancer (v2)
- is layer 7 (http)
- load balancing to multiple http apps across machines
    - target grps
- load balancing to multiple apps on same machine
    - Eg. containers
- support for http/2 & websocket
    - supports redirect 
        - Eg. from http to https
    - routing tables to diff target grps
        - routing based on path in url
            - Eg. example.com/users & /posts
        - routing based on hostname in url
            - Eg. one.example.com vs other.example.com
        - routing based on query string & headers
            - Eg. example.com/users?id=123&order=false
- ALB is great fit for micro services & container-based app
    - Eg. docker & amazon ecs
- has port mapping feature to direct to dynamic port in __ECS (elastic container service)__
- in comparison, we would need multiple classic load balancer per app

![](https://i.imgur.com/BFEJeyt.png)

- NOTE
    - fixed hostname
        - Eg. xxx.region.elb.amazonaws.com
    - app servers dont see ip of client directly
        - true ip of client inserted in header `X-Forwarded-For`
        - also get Port (X-Forwarded-Port) & proto (X-Forwarded-Proto)

![](https://i.imgur.com/Vwc78XS.png)
- ec2 need look at extra headers to see client ip

#### Target Groups
- ec2 instances
    - can be managed by auto scaling grp
    - http
- ec2 tasks
    - managed by ecs itself
    - http
- lambda funcs
    - http req translated into json event
- ip addr
    - must be priv IP
- ALB can route to multiple target grps
    - health checks at target grp lvl

#### Console
![](https://i.imgur.com/uMOmyGT.png)
- need to config targets to instance

![](https://i.imgur.com/9GefTLK.png)
- can create more target grps for more apps

![](https://i.imgur.com/nTTxdMF.png)
- target grps can have specific rules (like a firewall)

![](https://i.imgur.com/0gYNUEc.png)
- if else format


### Network Load Balancer (v2)
- allow to
    - forward tcp & udp traffic to instances
    - handle millions of requests per sec
    - less latency ~100ms VS 400ms for alb
- nlb has 1 static ip per az & supports elastic ip
    - helpful for whitelisting specific ip
- used for extreme perf, tcp or udp traffic
- not included in aws free tier
- NOTE
    - NLB exposes public static ip whereas app/classic lb exposes static dns (URL)

![](https://i.imgur.com/8S7KXpA.png)

#### Console
- similar to alb but now with diff protocol instead of ports
- NOTE
    - traffic coming from nlb is seen not from nlb but from outside in server perspective
        - hence initial health checks will return unhealthy
        - need to config sec grp
            - add rule to allow tcp port 80 from anywhr
    - when choosing az and subnets of lb, cannot change once cfmed

![](https://i.imgur.com/i4xiGN6.png)


More on Load Balancers
---
### Load Balancer Stickiness
- possible to implement stickiness so same client always redirected to same instance behind lb
    - works for clb & alb
    - cookie used for stickiness has expiration date you control
- use case
    - make sure user dont lose session data
        - keep on talking to same instance
    - as long as cookie is same, lb redirect to right instance
- enabling stickiness may bring imbalance to load over backend ec2 instances
    - load not evenly distributed - stickied instance may be overloaded
   
![](https://i.imgur.com/fZhbgat.png)

#### Console
- for clb, can config stickiness in lb console
    - but for alb, config of stickiness will be in target grp lvl

![](https://i.imgur.com/QrskajI.png)
![](https://i.imgur.com/AvAEMfa.png)
- enter duration to maintain stickiness


### Cross-Zone Load Balancing
- cross zone lb
    - ea lb instance distributes evenly across all regsitered instances in __all__ az
    - otherwise ea lb node distributes req evenly across registered instances in its az only

![](https://i.imgur.com/Lu9mHSJ.png)

#### Settings for Different LBs
- clb
    - disabled by default
    - no charges for inter az if enabled
        - usually cross az will charge
- alb
    - always on (cannot disable)
    - no charges for inter az data
- nlb
    - disabled by default
    - pay charges for inter az if enabled

#### Console
![](https://i.imgur.com/itn2W6a.png)


### SSL Certificates
#### SSL/TLS Basics
- ssl cert allows traffic between clients & lb to be encrypted in transit
    - in-flight encryption
- __ssl - secure sockets layer__
    - encrypt conns
- __tls - transport layer security__
    - newer ver of ssl
    - mainly used now, but people still refer as ssl
- public ssl certs issued by certificate authority (ca)
    - Eg. comodo, symantec, godaddy, globalsign, digicert, letsencrypt etc.
- ssl certs have expiration date that you set & must be renewed regularly

#### Using SSL Certs in LBs
- lb uses X.509 digital cert (SSL/TLS server cert)
- can manage certs using __aws cert manager (ACM)__
    - can create/upload your own certs alternatively
- https listener
    - must specify default cert
    - can add optional list of certs to support multiple domains
    - clients can use __server name indication (SNI)__ to specify hostname they reach
    - ability to specify policy to support older vers of ssl/tls (legacy clients)
- NOTE
    - internally lb does __ssl cert termination__
    - lb talks to backend instance using http (not encrypted)
        - but traffic goes over vpc which is priv network so is secure

![](https://i.imgur.com/Jq8P8hU.png)


#### Server Name Indication (SNI)
- sni solves prob of loading multiple ssl certs onto web server to serve multiple sites
    - is newer protocol & requires client to indicate hostname of target server in initial ssl handshake
    - server will then find correct cert or return default one
- NOTE
    - only works for alb & nlb (newer gen) & cloudfront
    - doesnt work for clb (older gen)

![](https://i.imgur.com/8ymnwek.png)
- able to have multiple target grps for diff sites using diff ssl certs

#### Note for Different LBs
- clb (v1)
    - support only 1 ssl cert
    - must use multiple clb for multiple hostname with multiple ssl certs
- alb (v2)
    - supports multiple listeners with multiple ssl certs
    - use sni to make it work
    - same for nlb

#### Console
![](https://i.imgur.com/uaVQGKT.png)
- load balancers > listen > edit > add https listener

![](https://i.imgur.com/QKZ0kw7.png)
- setup cipher for new listener

![](https://i.imgur.com/flsjYex.png)
- select cert to use 
    - CLB can only select one

![](https://i.imgur.com/UNpCmun.png)
- this is adding ssl cert page for alb & nlb


### Connection Draining
- feature naming (diff name for diff lb)
    - clb - conn draining
    - target grp - deregistration delay
        - for alb and nlb
- conn draining  - time to complete in-flight reqs while instance is de-registering or unhealthy
    - stops sending new reqs to instance that is de-registering
    - allow instance to just shutdown anything its doing before doing deregistered
- between 1-3600 secs (3600s = 1h)
    - default is 300 secs
    - set higher if instance take very long to deregister
- can be disabled
    - set value to 0
    - then client just received error and its up to client to retry to be redirected to another instance
- set to low value if reqs are short

![](https://i.imgur.com/9RH3mS4.png)
- will wait for conn draining period (default 300 secs)
    - in meantime redirected to other instances


Auto Scaling Group (ASG)
---
- irl, load on websites and apps can change
    - in cloud, can create/get rid of servers very quickly
- goal of ASG
    - scale out (add ec2 instances) to match increased load
    - scale in (remove ec2 instances) to match decreased load
    - ensure have min & max num of machines running
    - auto register new instances to load balancer

![](https://i.imgur.com/vjmejs0.png)
![](https://i.imgur.com/GpTbQml.png)

### ASG Attributes
- launch configuration
    - AMI (amazon machine img) + instance type
    - ec2 user data
    - elastic block store (ESB) volumes
    - security grps
    - SSH key pair
- min size/max size/initial capacity
    - min/max size, desired capacity will come out very often
- network + subnets info
- load balancer info
- scaling policies
- NOTE
    - scaling policies can be on cpu, network etc.
        - can also be on custom metrics or based on schedule (if know visitor patterns)
    - ASGs use launch configs or __Launch Templates__ (newer)
    - to update ASG, must provide new launch config/template
    - IAM roles attached to ASG will get assigned to ec2 instances
    - ASGs are free
        - pay for underlying res being launched
    - having instances under ASG means if they get terminated, asg will auto create new ones as replacement
        - extra safety
    - asg can terminate instances marked as unhealthy by LB
        - & replace them

### Auto Scaling Alarms
- possible to scale asg based on CloudWatch alarms
    - monitors a metric (Eg. average CPU)
    - metrics computed for overall ASG instances
- based on alarm,
    - create scale-out policies
        - inc num of instances
    - create scale-in policies
        - dec num of instances

![](https://i.imgur.com/1Va5zJT.png)

### Auto Scaling New Rules
- possible to define better auto scaling rules that are directly managed by ec2
    - target average cpu usage
    - num of requests on elb per instance
    - average network in
    - average network out
- rules easier to setup & make more sense

### Auto Scaling Custom Metric
- can autoscale based on custom metric
    - Eg. num of connected users
- steps
    - send custom metric from app on ec2 to cloudwatch
        - PutMetric API
    - create cloudwatch alarm to react to low/high values
    - use cloudwatch alarm as scaling policy for ASG


### Scaling Policies
- target tracking scaling
    - most simple & easy to setup
    - Eg. want average asg cpu to stay at 40%
- simple/step tracking
    - when cloudwatch alarm triggered (Eg. CPU > 70%), then add 2 units
        - remove 1 if cpu < 30%
- scheduled actions
    - anticipate scaling based on known usage patterns
    - Eg. inc min capacity to 10 at 5pm on fridays

### Scaling Cooldowns
- cooldown period ensures asg dont launch/terminate additional instances before prev scaling activity takes effect
    - cooldown to settle before scaling
    - also can create cooldowns that apply to specific __simple scaling policy__ aside from default cooldown
- scaling-specific cooldown period overrides default cooldown period
- common use for scaling specific cooldowns is with scale-in policy
    - since policy terminates instances, ec2 auto scaling needs less time to determine whether to terminate extra instances
    - if default cooldown period of 300secs too long, can reduce costs by applying scaling-specific cooldown period of 180secs to scale-in policy
        - to terminate instances faster
- if app scaling up and down multiple times ea hour, modify auto scaling grp cooldown timers & cloudwatch alarm period that triggers scale-in

![](https://i.imgur.com/LWOK480.png)

### Console
- find auto scaling tab in ec2 and go to auto scaling console
![](https://i.imgur.com/HsCcMfI.png)
- choose launch template/config
    - templates allows you to use a spot fleet of instances
    - config allows u to specify just one instance type

![](https://i.imgur.com/DWN4WTa.png)
- select ami
    - instance type
    - key pair
    - its like creating ec2 instance

![](https://i.imgur.com/NEhqzg3.png)
- can choose on-demand or spot instances or combi or both
    - hybrid better

![](https://i.imgur.com/UskNUAT.png)
![](https://i.imgur.com/nA983k6.png)
- elb health check itself from within target grp doesnt pass
    - alg auto replace terminated instances

![](https://i.imgur.com/DrnRa3F.png)
- asg will try best to meet desired capacity

#### Scaling Policies + Cooldown
![](https://i.imgur.com/LfL0qdo.png)
![](https://i.imgur.com/UklLjiq.png)
- target scaling


![](https://i.imgur.com/s6UZdGU.png)
- step scaling

![](https://i.imgur.com/7k7BcoC.png)
- scheduled scaling



###### tags: `AWS Developer Associate` `Notes`