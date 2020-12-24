

03 EC2 Storage: EBS & EFS
===




## Table of Contents

- [03 EC2 Storage: EBS & EFS](#03-ec2-storage--ebs---efs)
  * [Table of Contents](#table-of-contents)
  * [Elastic Block Store (EBS)](#elastic-block-store--ebs-)
    + [EBS Volume](#ebs-volume)
    + [EBS Volume Types](#ebs-volume-types)
      - [Use Cases - GP2](#use-cases---gp2)
      - [Use Cases - IO1](#use-cases---io1)
      - [Use Cases - ST1](#use-cases---st1)
      - [Use Cases - SC1](#use-cases---sc1)
      - [Summary of Vol Types](#summary-of-vol-types)
    + [EBS VS Instance Store](#ebs-vs-instance-store)
      - [Local EC2 Instance Store](#local-ec2-instance-store)
    + [Console](#console)
  * [Elastic File System](#elastic-file-system)
    + [Performance & Storage Classes](#performance---storage-classes)
    + [Console](#console-1)
  * [EBS VS EFS](#ebs-vs-efs)
    + [EBS](#ebs)
    + [EFS](#efs)



Elastic Block Store (EBS)
---
- ec2 machine loses root vol when manually terminated
    - unexpected terminations might happen occasionally (aws will email you)
    - need way to store instance data somewhr
- elastic block store (ebs) - network drive u can attach to instances when they run
    - allows instances to persist data

### EBS Volume
- is network drive (not phy drive)
    - uses network to comm with instances
        - might have latency
    - can be detached from ec2 instance & attached to another one quickly
- locked to AZ
    - ebs vol in us-east-1a cannot be attached to us-east1b
    - to move vol across, need to snapshot 
- has provisioned capacity
    - size in GBs & IOPS
    - get billed for all provisioned capacity
        - provisioned is diff from use
        - Eg. provision 1tb but only use 500gb, still pay for 1tb
    - can inc capacity of drive over time

![](https://i.imgur.com/Q2Jsk6X.png)

### EBS Volume Types
- 4 types
    - GP2 (SSD)
        - general purpose ssd vol that balances price & perf for wide variety of workloads
    - IO1 (SSD)
        - highest perf ssd vol for mission-crit low-latency/high-throughput workloads
    - ST1 (HDD)
        - low cost hdd vol designed for frequently accessed, throughput-intensive workloads
    - SC1 (HDD)
        - lowest cost hdd vol designed for less frequently accessed workloads
- characterised in size/throughput/IOPS (I/O Ops per sec)
- when in doubt, consult aws documentation
    - constantly updating
- only GP2 and IO1 can be used as boot vols

#### Use Cases - GP2
- recommended for most workloads
    - system boot vols
    - virtual desktops
    - low latency interactive apps
    - development & test envs
- 1 GB - 16 TB
- small gp2 vols can burst IOPS (input/output operations per second) to 3000
    - max IOPS is 16k
    - 3 IOPS per gb
        - at 5334gb we at max IOPS

![](https://i.imgur.com/iXNZhR2.png)
- IOPS dec/inc as size dec/inc

#### Use Cases - IO1
- crit business apps that require sustained IOPS perf or more than 16k IOPS per vol (gp2 limit)
- large db workloads like
    - mongodb, cassandra, microsoft sql server, mysql, postgresql, oracle
- 4 GB - 16 TB
- IOPS provisioned (PIOPS) - min 100 to max 64k (nitro instances)
    - else max 32k (other instances)
    - max ratio of provisioned IOPS to requested vol in GiB is 50:1
        - anything above that rejected
        - Eg. 10gb = 500 max IOPS

#### Use Cases - ST1
- streaming workloads requiring constant, fast throughput at low price
    - big data, data warehouses, log processing
    - apache kafka
- cannot be boot vol
- 500 GB - 16 TB
- max IOPS is 500
- max throughput of 500mb/s - can burst

#### Use Cases - SC1
- throughput-oriented storage for large vols of data infrequently accessed 
    - scenarios whr lowest storage cost important
- cannot be boot vol
- 500 GB - 16 TB
- max IOPS is 250
    - is fixed
- max throughput 250mb/s - can burst

#### Summary of Vol Types
![](https://i.imgur.com/6ADTJYu.png)
- for gp2,
    - 3000 iops per tb


### EBS VS Instance Store
- some instances dont come with root ebs vols
    - instead come with __instance store__ (= ephemeral storage)
        - ephemeral = lasting for a very short time
- instance store is phy attached to machine
    - whereas ebs is network drive
- pros (cuz no network)
    - better i/o perf
        - ebs gp2 have max iops of 16k, io1 have 64k
    - good for buffer/cache/scratch data/temp content
    - data survives reboots
- cons
    - on stop/termination, instance store lost
    - cant resize instance store
    - backups must be operated by user
- ask yourself, r u okay losing your data/is your data ephemeral?

#### Local EC2 Instance Store
- phy disk attached to phy server whr your ec2 is
    - very high IOPS because phy attached
        - network bandwidth limits iops
    - disks up to 7.5tb (can change over time), stripped to reach 30tb (can change over time)
        - number can change over time
        - BUT once setup disk in local instance store, cannot change its size??
- block storage
    - just like ebs
    - can have file system
- cannot be increased in size
- risk of data loss if hardware fails

![](https://i.imgur.com/AnoOUzL.png)


### Console
![](https://i.imgur.com/dggaeBF.png)
- when launching new instance, add new ebs volume
    - free tier can get up to 30gb of ebs gp2 ssd

![](https://i.imgur.com/ftlio9d.png)
- view your volumes in the volumes tab
    - 8gb is root, 2gb is ebs
- `lsblk` to list mounted volumes in terminal
    - https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-using-volumes.html
        - follow this guide to make vol avail for use
    - must create and mount new file system since new vols are raw block devices

![](https://i.imgur.com/7hnpSvj.png)



Elastic File System
---
- elastic file system (efs) - managed network file system (NFS) that can be mounted on many ec2
- works with ec2 instances in multi-az
    - unlike ebs which is locked to az
- highly available, scalable, expensive (3x gp2) and pay per user
    - might be cheaper than ebs if dont store too much data
- use cases
    - content management
    - web serving
    - data sharing
    - wordpress
- uses NFSv4.1 protocol
- uses security grp to control access to EFS
- compatible with linux based AMI 
    - not windows! windows cannot mount efs onto their file system
- encryption at rest using __key management service (KMS)__
- portable OS interface (POSIX) file system (linux) that has a standard file API
    - EFS only used for POSIX file systems (basically in linux)
- file system scales automatically, pay per use and no capacity planning
    - hence very easy to use

![](https://i.imgur.com/lIRI9xO.png)

### Performance & Storage Classes
- efs scale
    - 1000s of concurrent NFS clients, 10gbps throughput
    - grow to petabyte-scale nfs automatically
        - huge
- performance mode (set at efs creation time)
    - general purpose (default)
        - latency-sensitive use cases
            - Eg. webserver, content management systems (CMS) etc.
    - max i/o
        - higher latency/throughput
        - highly parallel
        - Eg. big data, media processing etc.
- storage tiers - has lifecycle management feature (move file after N days)
    - standard
        - for frequently accessed files
    - infrequent access (EFS-IA)
        - cost to retrieve files
        - lower price to store


### Console
- need search efs console from all aws services

![](https://i.imgur.com/0N3LXZ7.png)
- click customise to customise efs settings 


![](https://i.imgur.com/3IrLUU6.png)
![](https://i.imgur.com/2WYil9H.png)
- lifecycle management
    - move files to EFS infrequent access storage class if not accessed for xx days
    - save costs

![](https://i.imgur.com/zp8OlEl.png)
- throughput mode
    - provisioned allows u to choose throughput from 1-1024mbps
    - bursting depends on size

![](https://i.imgur.com/vvcaezd.png)
- need define sec grp for ea az

![](https://i.imgur.com/8eesZRj.png)

![](https://i.imgur.com/KzyetIB.png)
- attach efs to instance by clicking attach btn at efs console
    - can mount via dns or ip
        - copy paste the commands into your ec2
        - need install amazon efs utils package
            - https://docs.aws.amazon.com/efs/latest/ug/installing-amazon-efs-utils.html


EBS VS EFS
---
### EBS
- ebs volumes
    - can be attached to only 1 instance at a time
    - locked to az lvl
    - gp2 - i/o increases if disk size inc
    - io1 - can inc io independently
- to migrate ebs vol across az
    - take snapshot
    - resotre snapshot to another az
    - ebs backups use io so shouldnt run them while app is handling a lot of traffic
        - else will get perf issues
- root ebs vols of instances get terminated by default if ec2 instance gets terminated
    - can disable this

![](https://i.imgur.com/K2p1dXT.png)


### EFS
- can mount to 100s of instances across az
- efs share website files
    - Eg. wordpress
- only for linux (POSIX)
- has higher price pt than ebs
    - can leverage EFS-IA for cost savings
- EFS only get billed for what you use
    - for ebs, get billed on provisioned size not actual size
- NOTE
    - rmb EFS VS EBS VS Instance Store
        - efs for nfs to mount across multiple instances
        - ebs is for network vol to mount only on 1 instance & locked to az
        - instance store to get max amt of io on instance
            - is ephemeral drive - easy lose data

![](https://i.imgur.com/NAXRKA0.png)



###### tags: `AWS Developer Associate` `Notes`