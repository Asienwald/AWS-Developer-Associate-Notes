

10 AWS Cloudfront
===




## Table of Contents

- [10 AWS Cloudfront](#10-aws-cloudfront)
  * [Table of Contents](#table-of-contents)
  * [AWS Cloudfront](#aws-cloudfront)
    + [Cloudfront at High Level](#cloudfront-at-high-level)
    + [Cloudfront Origins](#cloudfront-origins)
      - [S3 as an Origin](#s3-as-an-origin)
      - [ALB or EC2 as an Origin](#alb-or-ec2-as-an-origin)
    + [Cloudfront Geo Restriction](#cloudfront-geo-restriction)
    + [Cloudfront VS S3 Cross Region Replication](#cloudfront-vs-s3-cross-region-replication)
    + [Cloudfront Caching](#cloudfront-caching)
      - [Maximise Cache Hits by Separating Static and Dynamic Distributions](#maximise-cache-hits-by-separating-static-and-dynamic-distributions)
    + [Cloudfront and HTTPS](#cloudfront-and-https)
    + [Cloudfront Signed URL/Signed Cookies](#cloudfront-signed-url-signed-cookies)
    + [Cloudfront Signed URL VS S3 Pre-Signed URL](#cloudfront-signed-url-vs-s3-pre-signed-url)
    + [Console](#console)
      - [Cloudfront Caching](#cloudfront-caching-1)
      - [Cloudfront Security](#cloudfront-security)
  * [Quiz](#quiz)




AWS Cloudfront
---
- is content delivery network (CDN)
    - improves read perf as content distributed and cached at edge locations around world
    - improve latency
- 216 pt of presence globally
    - edge locations
    - always growing
- DDos protection, integration with shield, AWS web app firewall
    - very protected
    - good way to front apps when u deploy them globally
- can expose external https by loading certs & can talk to internal https backends
- https://aws.amazon.com/cloudfront/features/?nc=sn&loc=2
    - everything here is edge
        - client access edge through pub network, edge access bucket through aws priv network

![](https://i.imgur.com/rrOFqZO.png)

### Cloudfront at High Level
![](https://i.imgur.com/UKKzu8U.png)
- edge location caches response based on cache settings defined and return resp to client
    - when client makes similar req, edge location will 1st look in cache before forwarding req to origin (whole purpose of using CDN)


### Cloudfront Origins
- s3 buckets
    - for distributing files globally and caching them at edge
    - enhanced security with cloudfront __Origin access identity (OAI)__
        - allow s3 bucket to only allow communication from cloudfront and nowhr else
    - cloudfront can be used as an ingress to upload files to s3 from anywhr in world
- custom origin (must be HTTP endpt)
    - app load balancer
    - ec2 instance
    - s3 website
        - must 1st enable bucket as static s3 website
    - any http backend you want


#### S3 as an Origin
![](https://i.imgur.com/eHjKuiE.png)
- for edge location to access your s3 origin bucket, must use OAI which is an iam role for your cloudfront origin

#### ALB or EC2 as an Origin
![](https://i.imgur.com/qqqOY70.png)
- sec grp of ec2 must allow ip of cloudfront edge locations into it
    - can get list of pub ip of edge locations here
        - http://d7uri8nf7uskq.cloudfront.net/tools/list-cloudfront-ips
    - same for alb
        - sec grp of ec2 can be priv though

### Cloudfront Geo Restriction
- is a security measure
- can restrict who can access your distribution
    - whitelist
        - allow users to access content only if they're in 1 of approved countries
    - blacklist
        - prevent user from accessing content based on list of banned countries
- the country is determined using a 3rd party geo-ip database
- use case
    - copyright laws to control access to content


### Cloudfront VS S3 Cross Region Replication
- cloudfront
    - global edge network
    - files attached for ttl
        - maybe a day
    - great for static content that must be avail everywhr
- s3 cross region replication
    - must be setup for ea region u want replication to happen
    - files updated in near real-time
    - read only
        - help with read perf
    - great for dynamic content that needs to be avail at low-latency in few regions


### Cloudfront Caching
- cache based on
    - headers
    - session cookies
    - query string params
- cache lives at ea cloudfront edge location
- u want to maximise cache hit rate to minimise requests to origin
    - control the ttl (0 secs to 1 year)
        - can be set by origin using the `Cache-Control` header, `Expires` header
    - can invalidate part of the cache using `CreateInvalidation` API

![](https://i.imgur.com/28lFUHY.png)

#### Maximise Cache Hits by Separating Static and Dynamic Distributions
![](https://i.imgur.com/ut70AYs.png)
- for static traffic, no headers and no session caching rules must apply
    - all content is cached in cloudfront to maximise cache hits
- for dynamic content need to be more careful abt how we want to cache based on the values of headers and cookies

### Cloudfront and HTTPS
- __viewer protocol policy__
    - redirect http to https
    - or use https only
- __origin protocol policy (http or s3)__
    - https only
    - or match viewer
        - HTTP => HTTP & HTTPS -> HTTPS
- NOTE
    - s3 bucket websites dont support https
        - only can use http

![](https://i.imgur.com/RRMjyf8.png)


### Cloudfront Signed URL/Signed Cookies
- want to distribute paid shared content to premium users over world
    - to restrict viewer access, can create cloudfront signed url/cookie
- validity length of url
    - shared content (movie, music)
        - short - few mins
    - priv content (priv to user)
        - can make it last for years
- signed url = access to indiv files
    - 1 signed url per file
- signed cookies = access to multiple files
    - 1 signed cookie for many files

![](https://i.imgur.com/g37I7En.png)
- objs in bucket can only be accessed by cloudfront
- client authorise and auth through app
    - app use aws sdk to generate signed url to clients
    - clients use signed url to get data directly from cloudfront


### Cloudfront Signed URL VS S3 Pre-Signed URL
![](https://i.imgur.com/4Oa6XXe.png)

- cloudfront signed url
    - allow access to path no matter the origin
        - not just s3 but http etc.
    - acc wide key-pair
        - only root can manage it
    - can filter by ip, path, date and expiration
    - can leverage caching features

![](https://i.imgur.com/OsguSAj.png)

- s3 presigned url
    - issue request as person who presigned url
    - uses iam key of signing iam principal
    - limited lifetime
- NOTE
    - if have cloudfront distro infront of bucket, can only use cloudfront signed url as thr's bucket policy restricting access to OAI
        - use presigned url if dw use cloudfront





### Console
![](https://i.imgur.com/CRHSvg8.png)
- create web distribution
    - will take awhile to create

![](https://i.imgur.com/kWZRE7m.png)
- restrict bucket access option
    - if select yes, will have more options appear
    - restrict so users must use cloudfront url not s3 url to access s3 contents
- need to create or use an existing origin access identity OAI

![](https://i.imgur.com/L1zdHIT.png)

![](https://i.imgur.com/FCVrHcE.png)
![](https://i.imgur.com/l2EKoXa.png)
- auto add bucket policy saying that OAI can getobject from s3 bucket
    - only cloudfront user can access bucket

![](https://i.imgur.com/T9Y6xFM.png)
- if get redirected to s3 url instead of cloudfront, is dns issue and must wait 3/4 hours
    - dns to propagate properly
    - https://stackoverflow.com/questions/38735306/aws-cloudfront-redirecting-to-s3-bucket
    - not a bug
   
![](https://i.imgur.com/uosgwCU.png)


![](https://i.imgur.com/tq6mlbo.png)
- also need make s3 bucket files public


#### Cloudfront Caching
![](https://i.imgur.com/cwa6PZ3.png)
- edit ttl in behaviour settings of cloudfront distribution

![](https://i.imgur.com/WD9d5YM.png)
- by invalidating an obj, u remove them from the cloudfront edge caches

![](https://i.imgur.com/yUC8Uzs.png)
- `*` mean everything
- cloudfront talk to all edge in the world and tell them to flush their cache based on this setting


#### Cloudfront Security
- OAI security
    - only specific OAI can access s3 bucket based on bucket policy

![](https://i.imgur.com/rioc5IW.png)
- viewer protocol policy in distribution behaviour
- dont see anything for origin protocol policy cuz we linked to s3 bucket
    - will have option if we have a http origin

![](https://i.imgur.com/U8ElI2j.png)
![](https://i.imgur.com/velVumM.png)
- set geo restriction for distro
    - access from restrictions tab in distro


Quiz
---
![](https://i.imgur.com/q1KkapV.png)
- i dont get why cloudfront is not the ans
    - get from pub edge location > retrieve frem s3 using priv subnet = faster



###### tags: `AWS Developer Associate` `Notes`