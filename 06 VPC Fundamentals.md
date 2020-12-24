

06 VPC Fundamentals
===



## Table of Contents

- [06 VPC Fundamentals](#06-vpc-fundamentals)
  * [Table of Contents](#table-of-contents)
  * [Virtual Private Cloud (VPC)](#virtual-private-cloud--vpc-)
    + [VPC & Subnets](#vpc---subnets)
      - [VPC Diagram](#vpc-diagram)
    + [Internet Gateway & NAT Gateways](#internet-gateway---nat-gateways)
    + [Network ACL & Security Groups](#network-acl---security-groups)
    + [VPC Flow Logs](#vpc-flow-logs)
    + [VPC Peering](#vpc-peering)
    + [VPC Endpoints](#vpc-endpoints)
    + [Site to Site VPN & Direct Connect](#site-to-site-vpn---direct-connect)
    + [VPC Summary](#vpc-summary)
  * [3 Tier Architecture](#3-tier-architecture)
    + [Typical 3 Tier Solution Architecture](#typical-3-tier-solution-architecture)
    + [LAMP Stack on EC2](#lamp-stack-on-ec2)
    + [Wordpress on AWS](#wordpress-on-aws)
      - [More Complicated](#more-complicated)
  * [VPC Quiz Notes](#vpc-quiz-notes)



Virtual Private Cloud (VPC)
---
__Note on Exam__
- need to know vpc in depth for SAA & sysops admin certs
- for associate dev, shld know
    - vpc, subnets, internet gateways and NAT gateways
    - security grps, network ACL (NACL), VPC flow logs
    - VPC peering, vpc endpts
    - sit to site vpn and direct connect

### VPC & Subnets
- vpc
    - priv network to deploy your res in it
    - regional res
- subnets
    - allow you to partition network inside vpc
    - availability zone res
- public subnet
    - subnet accessible from internet
- private subnet
    - not accessible from internet
- route tables
    - used to define access to internet & between subnets

![](https://i.imgur.com/HdIp0W0.png)

#### VPC Diagram
![](https://i.imgur.com/DPzyrqM.png)
- cidr range = range of ip allowed in your vpc
- this is common for a vpc that is created for u
    - only have pub subnets when using aws cloud
        - dh priv subnets
    - 1 pub subnet per AZ
        - 1 vpc in ea region created for u
            - AKA default vpc


### Internet Gateway & NAT Gateways
- internet gateways helps vpc instances connect with internet
    - public subnets have route to internet gateway
- NAT gateways (aws-managed) & NAT instances (self-managed) allow instances in __priv subnets__ to access internet while remaining private

![](https://i.imgur.com/dazxiEz.png)


### Network ACL & Security Groups
- network acl (NACL)
    - firewall which controls traffic from/to subnet
    - can have ALLOW & DENY rules
    - attached at subnet lvl
    - rules only include ip addr
- security grps
    - firewall that controls traffic from/to an ENI (elastic network interface)/EC2 instance
    - can only have ALLOW rules
    - rules include ip addr & other sec grps
- NOTE
    - default vpc, default nacl allows everything in and out
        - hence dn change nacl in this course and wont in the future



![](https://i.imgur.com/sWO9l0O.png)

![](https://i.imgur.com/7oKy6BY.png)
- [Read more on diff](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Security.html)

### VPC Flow Logs
- capture info abt ip traffic going into your interfaces
    - vpc flow logs
    - subnet flow logs
    - elastic network interface flow logs
- helps monitor & troubleshoot connectivity issues
    - Eg. subnet to internet, subnet to subnet, internet to subnets
- captures network info from aws managed interfaces too
    - elastic load balancers
    - elasticache
    - rds
    - aurora
    - etc.
- vpc flow logs data can go to s3/cloudwatch logs for storage

### VPC Peering
- connect 2 vpc privately using aws' network
    - make them behave as if on same network
    - must have overlapping CIDR (ip addr range)
- vpc peering conn is not transitive
    - must be established for ea vpc that need to comm with ea other
    - Eg. if only c and a have vpc peering, c cannot talk to b even though a connect to b
        - need to create own peering to 

![](https://i.imgur.com/ETm3llH.png)

### VPC Endpoints
- endpts allow you to connect aws using priv network instead of pub network
    - gives enhanced security & lower latency to access aws services
- vpc endpt gateway
    - s3 & dynamodb
- vpc endpt interface
    - the rest
    - can create vpc endpt interface to have priv access to cloudwatch
- only used within your vpc

![](https://i.imgur.com/TF3A3xS.png)
- eg. use vpc endpt to access public realm services (s3, dynamodb etc.) from within priv vpc subnet
    - is only for these services (traffic dont go through internet)


### Site to Site VPN & Direct Connect
- site to site vpn
    - connect at on-premises vpn to aws
    - conn auto encrypted
    - goes over pub internet
    - very easy to setup
- direct connect (DX)
    - establish phy conn between on-premises & aws
    - conn priv, secure and fast
    - goes over priv network
    - takes at least a month to establish
        - because is priv line to your vpc
- NOTE
    - site to site vpn & direct connect cannot access vpc endpts

![](https://i.imgur.com/eDjOKg8.png)


### VPC Summary
- vpc
    - virtual priv cloud
- subnets
    - tied to az
    - network partition of vpc
- internet gateway
    - at vpc lvl, provide internet access
- nat gateway/instances
    - give internet to priv subnets
- nacl
    - stateless
    - subnet rules for inbound & outbound
- security grps
    - stateful
    - operate at ec2 instance lvl or ENI
- vpc peering
    - connect 2 vpc with non overlapping ip ranges
    - non transitive
- vpc endpts
    - provide priv access to aws within vpc
- vpc flow logs
    - network traffic logs
    - for debugging and viewing network traffic
- site to site vpn
    - vpn over pub internet between on-premises dc and aws
- direct connect
    - direct priv conn to aws


3 Tier Architecture
---
### Typical 3 Tier Solution Architecture
![](https://i.imgur.com/FDknEjH.png)
- 3 tier
    - pub subnet
    - priv subnet
    - data subnet
- elb to access web server
    - ec2 instances in auto scaling grp
    - elb spread traffic between asg
- dns query through route53

### LAMP Stack on EC2
- linux
    - os for ec2 instances
- apache
    - web server that run on linux (ec2)
- mysql
    - db on rds
- php
    - app logic (running on ec2)
- NOTE
    - can add redis/memcached (elasticache) to include caching tech
    - to store local app data & software, use ebs drive (root)

### Wordpress on AWS
![](https://i.imgur.com/s5Swx6D.png)
- elb
- efs to store images

#### More Complicated
![](https://i.imgur.com/BxZSFM1.png)
- shld be able to understand everything by now


VPC Quiz Notes
---
![](https://i.imgur.com/8t2vWHi.png)



###### tags: `AWS Developer Associate` `Notes`