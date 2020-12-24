


01 AWS Fundamentals P1
===


## Table of Contents

- [01 AWS Fundamentals P1](#01-aws-fundamentals-p1)
  * [Table of Contents](#table-of-contents)
  * [Services Covered in Course Cert](#services-covered-in-course-cert)
  * [Regions and Zones](#regions-and-zones)
    + [AWS Regions](#aws-regions)
    + [AWS Availability Zones](#aws-availability-zones)
  * [IAM Introduction](#iam-introduction)
    + [3 Main Entities](#3-main-entities)
    + [IAM Federation](#iam-federation)
    + [Best Practices](#best-practices)
    + [IAM Console](#iam-console)
      - [MFA](#mfa)
      - [Creating Users](#creating-users)
      - [IAM Password Policy](#iam-password-policy)
  * [EC2 Introduction](#ec2-introduction)
    + [EC2 Console](#ec2-console)
      - [Creating Instance](#creating-instance)
      - [Viewing Instance](#viewing-instance)
    + [SSH-ing into your instance](#ssh-ing-into-your-instance)
      - [SSH from Terminal (Mac/Linux/Win10)](#ssh-from-terminal--mac-linux-win10-)
      - [SSH from Putty](#ssh-from-putty)
      - [Troubleshooting SSH](#troubleshooting-ssh)
      - [EC2 Instance Connect](#ec2-instance-connect)
  * [Introduction to Security Groups](#introduction-to-security-groups)
    + [Configuring Security Groups in EC2 Console](#configuring-security-groups-in-ec2-console)
  * [Private VS Public IP](#private-vs-public-ip)
    + [Diff between Private & Public IP](#diff-between-private---public-ip)
      - [Diff in AWS EC2](#diff-in-aws-ec2)
    + [Elastic IPs](#elastic-ips)
      - [Configuring Elastic IP](#configuring-elastic-ip)
  * [More on EC2](#more-on-ec2)
    + [Launching Apache Server on EC2](#launching-apache-server-on-ec2)
    + [EC2 User Data](#ec2-user-data)
      - [Example](#example)
    + [EC2 Instance Launch Types](#ec2-instance-launch-types)
      - [On-Demand](#on-demand)
      - [Reserved Instances](#reserved-instances)
      - [Spot Instances](#spot-instances)
      - [Dedicated Instances](#dedicated-instances)
      - [Dedicated Hosts](#dedicated-hosts)
      - [Analogy](#analogy)
      - [Price Comparison](#price-comparison)
    + [Elastic Network Interfaces (ENI)](#elastic-network-interfaces--eni-)
    + [EC2 Pricing](#ec2-pricing)
    + [Amazon Machine Images (AMI)](#amazon-machine-images--ami-)
      - [Why use custom AMI?](#why-use-custom-ami-)
    + [EC2 Instance Overview](#ec2-instance-overview)
      - [Burstable Instances (T2)](#burstable-instances--t2-)
      - [CPU Credit](#cpu-credit)
      - [T2 Unlimited](#t2-unlimited)
    + [EC2 Checklist](#ec2-checklist)


Services Covered in Course Cert
---
![](https://i.imgur.com/cJ6B1uH.png)

Regions and Zones
---
### AWS Regions
![](https://i.imgur.com/OPnx8PF.png)
- ea region is cluster of data centers
- most services region-scoped
    - same service from diff regions is diff
- some services are global
    - don't run on specific region

### AWS Availability Zones
- ea region has many avail zones (AZ) (max 6, usually 2/3)
    - Eg. sp-southeast-2a and 2b
- ea az is 1/more data centeres with redundant power, networking and connectivity
    - separate from ea other - isolated from disasters
    - connected with high bandwidth, ultra-low latency networking

![](https://i.imgur.com/Pktp6Oo.png)


IAM Introduction
---
- identity and access management (IAM)
    - manages whole aws security
    - center of aws
- root acc shld nvr be used or shared
    - also best to give users minimal amt of perms (least privilege)
- uses policies written in json to set permissions
    - has predefined managed policies
        - prebuilt policy templates by AWS so you dont have to craft from scratch
        - for ease of management
- is global (no specific region)
- MFA (multi-factor auth can be setup)

### 3 Main Entities
- users
    - for people
- grps
    - contains grp of users usually for specific functions (admin func, devops func etc.)
- roles
    - for machines

### IAM Federation
- enterprises usually integrate own repo of users with IAM
    - like active dir
    - login into aws using company credentials
- identity federation uses SAML standard
    - active dir uses this too

### Best Practices
- 1 iam user per person
- 1 iam role per app
- do not share iam credentials
- dont write iam credentials in code
    - people steal and spend your aws credits
    - dont commit them too
- nvr use root acc except for initial setup
- nvr use root iam credentials

### IAM Console
![](https://i.imgur.com/Ty7YHZ7.png)
- security status
    - follow this to secure your iam
    - delete root access keys - do not ever use root credentials
- customise your iam signin link with your own alias so people dont sign in to your root account
    - use this link to signin as one of your iam users
        - new password set for first login

#### MFA
![](https://i.imgur.com/XlwtynW.png)
- go to security credentials page to activate mfa
- choose virtual or hardware mfa
    - virtual
        - can use apps like google auth
        - scan qr code and input auth codes to verify yourself

#### Creating Users
![](https://i.imgur.com/dFXjpNu.png)
- set
    - access type
    - password
        - manual or auto generate
    - permissions
        - attaching policies or adding to grp
            - can use prebuilt policies
    - set permissions boundary 
- once created,
    - can download security credentials.csv or save password to txt file
- creating grp similar to creating user
    - can add user to grp then attach policy to grp

#### IAM Password Policy
![](https://i.imgur.com/2tcC403.png)
- ensure iam users have strong passwords


EC2 Introduction
---
- most popular service in aws
- capabilities consists of
    - renting VMs (EC2)
    - storing data on virtual drives (EBS)
    - load balancing (ELB)
    - scaling services with auto-scaling grp (ASG)

### EC2 Console
![](https://i.imgur.com/Dsdem8K.png)

- make sure your region correct
    - is not global like iam

#### Creating Instance
![](https://i.imgur.com/KJxyEcH.png)
- choose __amazon machine image (AMI)__
    - basically software and OS to be launched on server
    - many OS to choose from (windows, linux etc.)
        - but course use amazon linux 2 ami as it comes with many amazon specific stuff 
![](https://i.imgur.com/asOlCjO.png)
- Choose Instance Type
    - determine machine specs
![](https://i.imgur.com/zyJSUnH.png)
- config instance details 
    - subnet determine which AZ to use
- adding storage
![](https://i.imgur.com/vzG9MeJ.png)
- adding tags
    - classify instances
![](https://i.imgur.com/WNvGC7s.png)
- config security grp
    - basically set of firewall rules
- review settings
    - review all your settings
- when launching, have to create key pair
    - this gives you access to login/ssh into the machine


#### Viewing Instance
![](https://i.imgur.com/bpOYwks.png)
- can stop/reboot/terminate instance by right clicking instance state


### SSH-ing into your instance
![](https://i.imgur.com/cg2dlkV.png)
- just use instance connect to ssh from your browser

#### SSH from Terminal (Mac/Linux/Win10)
- use `ec2-user` as ssh user
    - use public ip of instance to ssh
    - use keypair .pem file with `-i` flag in ssh command
        - `ssh -i key.pem ec2-user@<pub ip>`
    - __you're gonna have warning - unprotected private key file__
        - very common exam qns
        - downloaded file uses perms 0644 which is not secure
            - anyone can read
            - do `chmod 0400` on key file
                - only owner can read 
                - [1st digit in chmod](https://serverfault.com/questions/344544/what-is-the-first-digit-for-in-4-digit-octal-unix-file-permission-notation#:~:text=From%20man%20chmod%20%3A,and%20sticky%20(1)%20attributes.)
- NOTE
    - win10 powershell dont have chmod command
        - go to properties of .pem key and set advanced security settings
        - ensure owner is set to yourself
        - remove other users
            - disable inheritance if needed
        - give yourself full control

![](https://i.imgur.com/O11JEPL.png)


#### SSH from Putty
- can use putty or mobaxterm to ssh from windows
- in puttygen, load private file to convert .pem key to .ppk which putty uses
    - then save priv key
        - dont need protect with password
- in putty
    - enter `ec2-user@<ip>` in hostname
        - rmb to save instance so no need retype all the time
        - under connection > ssh > auth
            - load priv key file for auth to ssh


#### Troubleshooting SSH
- ensure ssh port 22 allowed in your security grp (which is the firewall)
- if show conn refused error, restart instance or create new if needed
    - make sure using amazon linux 2
- make sure pub ip of instance correct
    - new ip allocated if u restart instance

#### EC2 Instance Connect
- just select your instance in the console and click connect 
    - only works with amazon linux 2

![](https://i.imgur.com/t1n2ePu.png)
- choose ec2 instance connect to connect from browser
    - will open a browser terminal
    - dont need worry abt keys


Introduction to Security Groups
---
- fundamental of network security in aws
    - like firewall but for ec2
    - controls how traffic is allowed in/out of ec2 machines
- regulate
    - access to ports
    - authorised ip ranges (ipv4 & ipv6)
    - control inbound/outbound traffic
- NOTE
    - can attach to multiple instances
    - locked to region/vpc config
        - new grp if diff region/vpc
    - block traffic by sec grp wont be accessible from ec2
    - good to maintain separate security grp just for ssh access
        - ssh access is complex
    - if ssh timeout, is sec grp issue
        - if conn refused, is app err/not launched
    - by default, all inbound traffic blocked
        - all outbound traffic authorised
    - you can reference other security grps when allowing/blocking traffic
        - Eg. all instances from sec grp 2 is allowed
        - good for load balancers
    - security grps cannot reference dns names
        - cuz no dns to resolve?

![](https://i.imgur.com/Mr6bDmy.png)


### Configuring Security Groups in EC2 Console
- inbound
    - traffic going into ec2 machine
    - by default shld have ssh rule
- outbound
    - traffic going out of machine
    - by default all traffic to anywhr allowed
- tags
    - can add name tag to your security grp
- * adding rules are just like adding firewall rules
    - type, protocol (TCP/UDP), port range, source and description


Private VS Public IP
---
- 2 types of IP IPv4 & IPv6
    - ipv4 - 32bits
        - ipv4 is most common format used online
        - allows 3.7b addr
    - ipv6 - 128bits
        - ipv6 is newer & solve probs for IoT
- this course only uses ipv4

![](https://i.imgur.com/AImyAi0.png)

### Diff between Private & Public IP
- public ip
    - machine can be identified on internet
    - must be unique across internet
    - can be geo-located easily
- private ip
    - machine only identified on priv network
    - must be unique across priv network
        - though 2 diff priv network can have same IP
    - machine connect to internet using NAT + internet gateway (proxy)
    - only specified range of IPs can be used as private IP

#### Diff in AWS EC2
- by default, ec2 machine comes with
    - priv ip for internal aws network
    - public ip for internet
- when sshing into ec2, 
    - cant use priv ip as not in same network
    - use pub ip
- pub ip changes when machine stopped & started


### Elastic IPs
- when restarting ec2 instance, it changes its public ip
    - if need fixed pub ip for instance, need elastic ip
- __elastic ip__ - public ipv4 ip you own as long as you dont delete it
    - can attach to 1 instance at a time
- with elastic ip, can mask failure of instance/software by rapidly remapping addr to another instance
- can only have 5 elastic ip in acc
    - though can ask aws to increase that, but you shldnt have a need for it
- NOTE
    - avoid using elastic ip
        - often reflect poor architectural decisions
        - use random public IP & register DNS name to it
        - OR use load balancer
            - best option
   
#### Configuring Elastic IP
![](https://i.imgur.com/mTubVzw.png)
- can choose to use aws ipv4 or your own

![](https://i.imgur.com/3Qldu7l.png)
- select new elastic ip and go to actions > associate elastic ip addr


More on EC2
---
### Launching Apache Server on EC2
- ssh into instance
- `yum updates -y`
    - force machien to update itself
- `yum install -y httpd.x86_64`
- `systemctl start httpd.service`
    - enable it too (start on reboot)
- `curl localhost:80`
    - load whatever page u specify
- rmb to allow port 80 on security grp to access webpage
- `echo "bruh $(hostname -f)"` > index.html
    - string formatting with echo

### EC2 User Data
- can bootstrap instance using __ec2 user data script__
    - __bootstrapping__ - launch commands when machine starts
    - script only runs once at instance first start
- used to automate boot tasks like
    - installing updates/software
    - downloading common files from internet
    - anything else
- ec2 user data script runs with root user
- NOTE
    - as you add more stuff, the more the instance have to do at boot time

#### Example
- write userdata script to ensure instance have apache http server installed
    - executed at first boot of instance
- first terminate our instance
    - create new instance

![](https://i.imgur.com/jRIbV5P.png)

- when creating new instance, at step 3 config instance details, expand advanced details > user data
- NOTE 
    - script is basically a bash file

```bash
#!/bin/bash
sudo su

# install httpd (linux 2 ver)
yum update -y
yum install -y httpd.x86_64
systemctl start httpd.service
systemctl enable httpd.service
echo "Hello from $(hostname -f)" > /var/www/html/index.html
```
- once u start your new instance, apache shld be installed and working

### EC2 Instance Launch Types
- on demand instances - create from beginning; running and on demand
    - short workload
    - predictable pricing
- reserved (min 1 year usage)
    - reserved instances
        - long workloads
    - convertible reserved instances
        - long workloads with flexible instances
        - Eg. instead of saying you want xx instance, you just say u want sth and can convert that sth
    - scheduled reserved instances
        - Eg. every thurs 3-6pm
- spot instances
    - short workloads
    - cheap
    - can lose instances
        - less reliable
- dedicated instances
    - other customers wont share hardware
- dedicated hosts
    - book entire phy server
    - control instance placement

#### On-Demand
- pay for what you use
    - billing per sec after 1st min
- highest cost but no upfront payment
- no long term commitment
- recommended for short-term & uninterrupted workloads whr u cant predict how app will behave
    - good for elastic workloads


#### Reserved Instances
- up to 75% discount compared to on-demand
    - pay upfront for what u use with long term commitment
- reservation period can be 1/3 years
    - reserve specific instance type
- recommended for steady state usage apps
    - like db
- convertible reserved instance
    - can change ec2 instance type
    - up to 54% discount
- scheduled reserved instance
    - launch within time window you reserve
    - when you need fraction of day/week/month
    - only avail in select regions

![](https://i.imgur.com/tI9xCRD.png)
![](https://i.imgur.com/NgDFDNf.png)


#### Spot Instances
- discount up to 90% compared to on-demand
- can lose instance at anytime if max price is less than current spot price
    - can view price history under instances > spot requests > pricing history in console
    - spot prices usually p stable
- most cost-efficient instance in aws
- useful for workloads resilient to failure (can retry)
    - batch jobs
    - data analysis
    - image processing
    - etc.
- bad for critical jobs/db
- great combo
    - reserved instances for baseline + on-demand & spot for peaks

![](https://i.imgur.com/IH2Ihvc.png)
- can define based on task needed

![](https://i.imgur.com/5ulvhiW.png)
![](https://i.imgur.com/erNwgWM.png)
- there's a shit ton of configs
- NOTE
    - can also request spot instance in step 3 of launching instance

![](https://i.imgur.com/cWSmZhN.png)
- can set max price
    - aws reclaim when spot price exceed max price


#### Dedicated Instances
- instance run on dedicated hardware
    - can share hardware with other instances only in same acc
- no control over instance placement
    - can move hardware after stop/start

![](https://i.imgur.com/BgWhfhg.png)



#### Dedicated Hosts
- phy dedicated ec2 server
    - full control of ec2 instance placement
    - visibility of underlying sockets/phy cores of hardware
        - great for licensing purposes
- allocated for acc for 3 year period reservation
- more expensive
- useful for software that have complicated licensing model
    - Bring your own license (BYOL)
        - some licenses might bill based on cores or sockets
    - or for companies with strong regulatory/compliance needs
        - dw share with customers
![](https://i.imgur.com/A0WcWhZ.png)

![](https://i.imgur.com/zva26Mo.png)


#### Analogy
![](https://i.imgur.com/0Nh05OB.png)

#### Price Comparison
![](https://i.imgur.com/vy4SHAt.png)
- reserved scheduled so high but prob lesser since u using instance lesser
- exam wont ask exact pricing


### Elastic Network Interfaces (ENI)
- ENI - logical component in VPC (virtual priv cloud) that represents __virtual network card__
- ENI can have following attributes
    - pri private ipv4, 1 or more secondary ipv4
    - 1 elastic ipv4 ip per priv ipv4
    - 1 pub ipv4
    - 1 or more sec grps
    - mac addr
- can create ENI independently & attach/move them on fly to ec2 instances for failover
- bound to specific AZ
- [Read more here](https://aws.amazon.com/blogs/aws/new-elastic-network-interfaces-in-the-virtual-private-cloud/)

![](https://i.imgur.com/MasNYuT.png)
- Eg. pri ENI attached to eth0 interface on your 1st ec2 instance
    - can add sec ENI to eth2
- ENI can be moved to another instance's interface

![](https://i.imgur.com/lZzKMhN.png)
![](https://i.imgur.com/qMs53gw.png)
- make sure subnet chosen is same subnet whr instances are in
- can attach sec grps too
- under network interface u can right click and click attach to attach ENI to an instance

### EC2 Pricing
- instance prices (per hour) varies based on
    - region
    - instance type
    - on-demand vs spot vs reserved vs dedicated host
    - linux vs windows vs priv OS (RHEL, SLES, Windows SQL)
- billed by sec with min of 60 secs
- pay for other factors like
    - storage
    - data transfer
    - fixed ip public addr
    - load balancing
- dont pay when instance is stopped
- consult pricing page
    - https://aws.amazon.com/ec2/pricing/on-demand/

### Amazon Machine Images (AMI)
- aws comes with base imgs like ubuntu, fedora, redhat, windows etc.
    - imgs can be customised at runtime using ec2 user data
- AMI - img used to create our instances
    - we can create our own image ready to go
    - can be built for linux/windows
    - built for specific regions

#### Why use custom AMI?
- pre-installed packages needed
- faster boot time
    - dont need long ec2 user data at boot time
- machine configed with monitoring/enterprise software
- security concerns
    - control over machines in network
- control of maintenance & updates of AMIs over time
- active dir integration out of box
- installing app ahead of time
    - faster deploys when auto-scaling
- use someone else's AMI optimised for running an app, db etc.


### EC2 Instance Overview
- instances have 5 distinct characteristics
    - ram
        - type
        - amt 
        - generation
    - cpu
        - type
        - make
        - freq
        - generation
        - num of cores
    - I/O
        - disk perf
        - EBS optimisations
    - network 
        - bandwidth
        - network latency
    - GPU
    - https://aws.amazon.com/ec2/instance-types/
        - summary of types
            - https://ec2instances.info/
- R/C/P/G/H/X/I/F/Z/CR specialised in ram, cpu, I/O, network & GPU
    - Eg. R instance a lot of ram, C instance CPU etc..
- M types balanced
    - good at everything but not great
- T2/T3 types are burstable
    - most common is t2.micro cuz free
- NOTE
    - exam wont ask qns like which instance best for xxx

#### Burstable Instances (T2)
- burst means overall, instance has OK CPU perf
    - when machine needs to process sth unexpected, it can burst (CPU become VERY good)
- if machine bursts, it uses __burst credits__
    - if all credits gone, CPU becomes BAD
        - if load too long, cpu becomes bad
    - if machine stops bursting, credits accumulated over time
- amazing to handle unexpected traffic & getting insurance that it'll be handled correctly
- if instance consistently runs low on credit, need to move to diff kind of non-burstable instance

#### CPU Credit
![](https://i.imgur.com/hS6U9qr.png)

#### T2 Unlimited
- nov 2017
    - possible to have unlimited burst credit balance
    - pay extra money if go over credit balance but dont lose perf
- be careful as costs can go high if not monitoring health of instances
- https://aws.amazon.com/blogs/aws/new-t2-unlimited-going-beyond-the-burst-with-high-performance/


### EC2 Checklist
![](https://i.imgur.com/6UVhbIH.png)






###### tags: `AWS Developer Associate` `Notes`