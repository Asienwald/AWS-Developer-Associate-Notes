

05 AWS Route 53
===



## Table of Contents

- [05 AWS Route 53](#05-aws-route-53)
  * [Table of Contents](#table-of-contents)
  * [Route 53](#route-53)
    + [DNS Records TTL](#dns-records-ttl)
    + [CNAME vs Alias](#cname-vs-alias)
    + [Health Checks](#health-checks)
    + [Route53 as a Registrar](#route53-as-a-registrar)
      - [3rd Party Registrar with Route53](#3rd-party-registrar-with-route53)
    + [Console](#console)
      - [TTL](#ttl)
      - [Health Checks](#health-checks-1)
  * [Route53 Routing Policies](#route53-routing-policies)
    + [Simple Routing Policy](#simple-routing-policy)
    + [Weighted Routing Policy](#weighted-routing-policy)
    + [Latency Routing Policy](#latency-routing-policy)
    + [Failover Routing Policies](#failover-routing-policies)
    + [Geolocation Routing Policy](#geolocation-routing-policy)
    + [Multi Value Routing Policy](#multi-value-routing-policy)




Route 53
---
- route53 is managed dns (domain name system)
    - dns is collection of rules and records which helps clients understand how to rch server through URLs
- in aws most common records are
    - A
        - hostname to ipv4
    - AAAA
        - hostname to ipv6
    - CNAME
        - hostname to hostname
    - alias
        - hostname to aws res
- route53 can use
    - public domain names you own/bought
    - priv domain names that can be resolved by your instances in your VPCs
- has advanced features like
    - load balancing
        - through dns
        - AKA client lb
    - health checks
        - but limited
    - routing policy
        - simple
        - failover
        - geolocation
        - latency
        - weighted
        - multi value
- pay $0.50 per month per hosted zone

![](https://i.imgur.com/kUsdtjU.png)

### DNS Records TTL
- high ttl
    - Eg. 24h
    - less traffic on dns
    - possibly outdated records
        - ttl need to end before change applied
- low ttl
    - Eg. 60s
    - more traffic on dns
    - records outdates for less time
    - easy to change record
- ttl is mandatory for ea dns record


![](https://i.imgur.com/15GoM37.png)

### CNAME vs Alias
- aws res (Eg. load balancerm cloudfront etc.) exposes an aws hostname
- CNAME
    - points hostname to any other hostname
        - Eg. app.mydomain.com => blabla.anything.com
    - __only for non root domain__
- alias
    - points hostname to aws res
        - Eg. app.mydomain.com => blabla.amazonaws.com
    - works for root/non root domains
    - free
    - native health check
- NOTE
    - exam ask diff between cname and alias
        - root domain = alias, non root = cname or alias
        - but shld be alias anyways cuz pt to aws service and is free

### Health Checks
- have x health checks failed = unhealthy
    - default 3
- after x health checks passed = healthy
    - default 3
- default health check interval = 30s
    - can set to 10s
        - but higher cost
        - AKA fast health check
- about 15 health checkers checks endpt health
    - 1 req every 2 secs on average
        - if interval lesser, avg go up
- can have http, tcp and https health checks
    - no ssl verification
- possibility of integrating health check with cloudwatch
- can be linked to route53 dns queries


### Route53 as a Registrar
- domain name registrar - org that manages reservation of internet domain names
- famous names
    - godaddy
    - google domains
    - etc.
- NOTE
    - domain registrar != DNS
        - but ea registrar usually comes with some DNS features

#### 3rd Party Registrar with Route53
- if buy domain on 3rd pt website, can still use route53
    - create hosted zone in route53
    - update NS records on 3rd pt website to use route53 name servers


### Console
![](https://i.imgur.com/U67pykw.png)
- is global service

![](https://i.imgur.com/qT5iecD.png)
- need buy avail domain name
    - check personal info to do purchase

![](https://i.imgur.com/TdUBSkI.png)
- after create domain name, can click on it to add some records

![](https://i.imgur.com/tOCeTul.png)
- create record set

#### TTL
![](https://i.imgur.com/fGTdUdB.png)
- default 300sec (5mins)


#### Health Checks
![](https://i.imgur.com/cLJiSdk.png)
![](https://i.imgur.com/d1beM40.png)
- what to monitor
- specify endpt

![](https://i.imgur.com/qvhZH7u.png)




Route53 Routing Policies
---
### Simple Routing Policy
- maps hostname to another hostname
    - use when need to redirect to single res
- cannot attach health checks to simple routing policy
- if multiple values returned, random one chosen __by client__

![](https://i.imgur.com/QJMyzss.png)


### Weighted Routing Policy
- control % of requests that go to specific endpt
    - Eg. helpful to test 1% of traffic on new app ver
    - helpful to split traffic between 2 regions
- can be associated with health checks
    - if 1 instance not working properly, no traffic sent to it

![](https://i.imgur.com/lNuwUZs.png)
![](https://i.imgur.com/GqEUTWN.png)


### Latency Routing Policy
- redirect to server that has least latency close to us
    - super helpful when latency of users is priority
        - latency is evaluated in terms of user to designated aws region
    - Eg. germany can be directed to US if its lowest latency

![](https://i.imgur.com/1PTjq42.png)


### Failover Routing Policies
![](https://i.imgur.com/0PrRpE0.png)
- secondary only used when pri fails
- route53 will have mandatory health check for pri associated with pri record
    - auto failover to sec if check fails

![](https://i.imgur.com/bn55mV5.png)
- must associate with health check if pri
    - will give error if put no



### Geolocation Routing Policy
- diff from latency based
    - routing based on user location
    - Eg. specify traffic from UK go to xx ip
- shld create default policy in case no match on location

![](https://i.imgur.com/aQDXQSS.png)



### Multi Value Routing Policy
- use when routing traffic to multiple res
- want to associate route53 health checks with records
    - up to 8 healthy records returned for ea multi value query
- multi value is not substitute for ELB
    - but still do some kind of lb on client side
- NOTE
    - some sort of improvement from simple routing policy

![](https://i.imgur.com/orXtAQJ.png)

![](https://i.imgur.com/YOE36Qv.png)
- ttl will update for all multi value

![](https://i.imgur.com/vWwfXHr.png)
- if u dig the dns record it should return more than 1 answers



###### tags: `AWS Developer Associate` `Notes`