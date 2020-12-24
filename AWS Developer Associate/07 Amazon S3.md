

07 Amazon S3
===




## Table of Contents

- [07 Amazon S3](#07-amazon-s3)
  * [Table of Contents](#table-of-contents)
  * [Amazon S3](#amazon-s3)
    + [Buckets](#buckets)
    + [Objects](#objects)
    + [Versioning](#versioning)
    + [S3 Encryption for Objects](#s3-encryption-for-objects)
      - [SSE-S3](#sse-s3)
      - [SSE-KMS](#sse-kms)
      - [SSE-C](#sse-c)
      - [Client Side Encryption](#client-side-encryption)
      - [Encryption in Transit (SSL/TLS)](#encryption-in-transit--ssl-tls-)
    + [S3 Security](#s3-security)
      - [S3 Bucket Policies](#s3-bucket-policies)
      - [4 Bucket Settings for Block Public Access](#4-bucket-settings-for-block-public-access)
      - [Other Security](#other-security)
    + [S3 Websites](#s3-websites)
    + [CORS (Cross-Origin Resource Sharing)](#cors--cross-origin-resource-sharing-)
      - [CORS in S3](#cors-in-s3)
    + [S3 Consistency Model](#s3-consistency-model)
    + [Console](#console)
      - [Versioning](#versioning-1)
      - [Encryption Settings](#encryption-settings)
      - [Security Settings](#security-settings)
      - [Host Websites](#host-websites)
      - [CORS](#cors)


Amazon S3
---
- s3 is 1 of main building blks of aws
    - advertised as infinitely scaling storage
    - widely popular & deserves own section
- many websites uses aws s3 as backbone
    - many aws services uses s3 as integration too
- s3 service is global but buckets are region specific


### Buckets
- amazon s3 allows people to store objects (files) in __buckets__ (directories)
- buckets must have __globally unique name__
    - defined at region lvl
- naming convention
    - no uppercase
    - no underscore
    - 3-63 chars long
    - not an ip
    - must start with lowercase letter/num

### Objects
- objects (files) have a key
- key is FULL path
    - ![](https://i.imgur.com/lOeIgGc.png)
- key composed of prefix + obj name
    - ![](https://i.imgur.com/f9Cnk9E.png)
- no concepts of directories within buckets
    - though UI will trick you to think otherwise
    - just keys with long names with slashes
- obj values are content of body
    - max obj size is 5tb
    - if uploading more than 5gb at a time, must use __multi-part upload__
        - divide obj into parts less than 5gb
- ea s3 object can have
    - metadata 
        - list of txt key/value pairs
        - system or user metadata
    - tags
        - unicode key/value pair
        - up to 10
        - useful for security/lifecycle
    - version id
        - if versioning enabled

### Versioning
- can version files in amazon s3
- enabled at bucket lvl
- same key overwrite will increment version
    - ver 1, 2, 3 etc.
- best practice to version buckets
    - protect against unintended deletes
        - abi to restore ver
    - easy rollback to prev version
- NOTE
    - any file not versioned prior to enabling versioning will have version null
    - suspending versioning does not delete prev versions


### S3 Encryption for Objects
- 4 methods of encrypting objs in s3
    - SSE-S3
        - encrypts s3 objs using keys handled & managed by aws
    - SSE-KMS
        - leverage aws key management service to manage encryption keys
    - SSE-C
        - when u want to manage your own encryption keys
    - client side encryption
- impt to uderstand which ones adapted to which situation in exam

#### SSE-S3
- encryption using keys handled & managed by amazon s3
- obj encrypted server side
- AES-256 encryption type
- must set header
    - `"x-amz-server-side-encryption":"AES256"`

![](https://i.imgur.com/vZ2bbKJ.png)


#### SSE-KMS
- encryption using keys handled & managed by KMS
    - KMS advantages
        - user control
        - audit trial
- obj encrypted server side
- must set header
    - `"x-amz-server-side-encryption":"aws:kms"`

![](https://i.imgur.com/SEg27er.png)


#### SSE-C
- server-side encryption using data keys fully managed by customer outside of aws
    - amazon s3 dont store encryption key you provide
- HTTPS must be used
    - since u sending a secret to aws 
- encryption key must provided in HTTP headers for every http request made
    - since its going to be discarded every single time
    - s3 wont save your key

![](https://i.imgur.com/EdjNLUP.png)


#### Client Side Encryption
- client lib like amazon s3 encryption client
- client encrypt data themselves before sending to s3
    - decrypt data themselves when retrieving from s3
- customer fully manages keys & encryption cycle

![](https://i.imgur.com/wGxecBC.png)

#### Encryption in Transit (SSL/TLS)
- amazon s3 is http service
- amazon s3 exposes
    - http endpt
        - non encrypted
    - https endpt
        - encrypted in flight
        - is encrypted & relies on ssl/tls certs
- free to use endpt u want, but https is recommended
    - most clients use https endpt by default
- https is mandatory for SSE-C
- encryption in flight is also called SSL/TLS
    - traffic between client and s3 is fully encrypted
    - uses ssl/tls certs

### S3 Security
- user based
    - IAM policies - which api calls allowed for specific user from iam console
- resource based
    - bucket policies
        - bucket wide rules from s3 console
        - allows cross account
        - say what principals can or cannot do in s3 bucket
    - object access control list (ACL)
        - finer grain
        - set at obj lvl the access rule
    - bucket access control list (ACL)
        - less common
- NOTE
    - IAM principal can access s3 obj if
        - user iam perms allow it OR res policy ALLOWS it
        - AND there's no explicit deny
    - obj and bucket acl dont rly come out in exam

#### S3 Bucket Policies
- json based policies
    - resources
        - buckets and objs
    - actions
        - set of api to allow/deny
    - effect
        - allow/deny
    - principal
        - acc/user to apply policy to
- use s3 bucket for policy to
    - grant pub access to bucket
    - force objs to be encrypted at upload
    - grant access to another acc
        - cross acc

![](https://i.imgur.com/OIiWr7O.png)
- this eg allows pub read

#### 4 Bucket Settings for Block Public Access
- blk pub access to buckets and objs granted through
    - new acls
    - any acls
    - new pub bucket/access pt policies
- block pub & cross acc access to buckets & objs through any pub bucket or access pt policies
- these settings were created to prevent company data leaks
    - if know bucket shldnt ever by public, leave these on
    - can be set on acc lvl
- NOTE
    - exam wont test on ea of these settings
        - just need to know that there's a way to blk pub access through these settings

#### Other Security
- networking
    - supports vpc endpts (for instances in vpc w/o www internet)
        - can access s3 privately
- logging & audit
    - s3 access logs can be stored in other s3 bucket
    - api calls can be logged in aws cloudtrial
- user security
    - MFA delete
        - MFA (multi factor auth) can be required in versioned buckets to delete objs
    - pre-signed URLs
        - urls valid only for limited time
            - Eg. premium vid service for logged in users

### S3 Websites
- s3 can host static websites and have them accessible on internet
- welsite url will be
    - `<bucket name>.s3-website-<aws region>.amazonaws.com` 
- if get 403 forbidden err, make sure bucket policy allows pub read

### CORS (Cross-Origin Resource Sharing)
- an origin is a scheme (protocol), host (domain) and port
- CORS (cross origin res sharing)
- web browser based mechanism to allow requests to other origins while visiting main origin
    - Eg. same origin
        - `http://example.com/app1` and `http://example.com/app2`
    - Eg. diff origins
        - `http://www.example.com` and `http://other.example.com`
    - browser based security
        - when visit website, can only make reqs to other origins only if these origins allow you to
        - defend against XSS 
- reqs wont be fulfilled unless other origin allows for reqs using __CORS Headers (Eg. Access-Control-Allow-Origin)__

![](https://i.imgur.com/DNpBRzF.png)
- preflight req ask cross origin if allowed to do req on it
    - preflight resp respond with that methods authorised

#### CORS in S3
- if client does cross origin req on s3 bucket, need to enable correct CORS headers
    - is popular exam qns
- can allow for specific origin or for * (all origins)

![](https://i.imgur.com/n4NIQFD.png)


### S3 Consistency Model
- s3 made of multiple servers
    - when write to s3, other servers will replicate data
        - leads to diff consistency issues
- read after write consistency for PUTS of new objs
    - as soon as new obj written, can retrieve it
        - Eg. PUT 200 => GET 200
    - this is true except if we did a GET before to see if obj existed
        - Eg. GET 404 => PUT 200 => GET 404
            - eventually consistent
- eventual consistency for DELETES & PUTS of existing objs
    - if read an obj after updating, might get older ver
        - Eg. PUT 200 => PUT 200 => GET 200
            - might be older ver
    - if delete an obj, might still be able to retrieve it for short time
        - Eg. DELETE 200 => GET 200
- NOTE
    - there's no way to req strong consistency
        - only get eventual consistency
            - means if overwrite obj, need to wait a bit before GET returns newest ver of obj





### Console
![](https://i.imgur.com/4RCVNq7.png)

![](https://i.imgur.com/rLGKDMC.png)
- bucket name cannot be used by other peeps

![](https://i.imgur.com/3UtiYU2.png)
- prevent bucket from being used by pub

![](https://i.imgur.com/X7xPcDd.png)
- upload file on bucket


![](https://i.imgur.com/0A8REvn.png)
- perms of who can see the file

![](https://i.imgur.com/Ofk5rAs.png)
- set properties of file
    - how its stored etc.
- NOTE
    - for non publicly accessible files, special URL created (pre-signed url) signed with own aws credentials to access the file


#### Versioning
![](https://i.imgur.com/JBu0mki.png)
- click enable it

![](https://i.imgur.com/9PTGTBE.png)
![](https://i.imgur.com/T8J7s8S.png)
- shows all versions

![](https://i.imgur.com/r5eE361.png)
- even if u delete a file, if u show vers, the file appears with a delete marker
    - 0 file size
    - can still restore

#### Encryption Settings
![](https://i.imgur.com/vXC7zEz.png)
- right now no encryption

![](https://i.imgur.com/JWXi6Pp.png)

![](https://i.imgur.com/a8HdF9I.png)
- for SSE-KMS, aws kms alr made your own s3 key to use
    - but can use your own custom one

![](https://i.imgur.com/UJOZd2Y.png)
- can set default encryption in bucket properties
- only 3 options though
    - aws chose not to implement SSE-C in their console
        - cannot do through console but can through CLI 
    - also no client side encryption option cuz thats client responsibility

#### Security Settings
![](https://i.imgur.com/zHzseEY.png)
- set bucket policies
    - is in json format
    - can use policy generator

![](https://i.imgur.com/BUFOy2v.png)
- ARN has to be taken from bucket policy page
    - add a `/*` behind it

![](https://i.imgur.com/HDMfBBU.png)
- can add conditions to deny/allow

![](https://i.imgur.com/SNm2az3.png)
- can set acls
    - this is for bucket lvl, but can set at obj lvl too

![](https://i.imgur.com/7s7fFVp.png)
- blk pub access
    - can do from acc lvl or bucket lvl too (this is acc)

#### Host Websites
![](https://i.imgur.com/QmYC2h4.png)
- set your index and error document
    - endpt will be created
- NOTE
    - need to make sure s3 bucket is pub not priv for u to access pub endpt
        - change blk pub access
        - create bucket policy to allow anyone to view website
            - api - getobject

#### CORS
![](https://i.imgur.com/zrcwq0S.png)
- under perms > cors config
    - in code shown above need to add url sending req from



###### tags: `AWS Developer Associate` `Notes`