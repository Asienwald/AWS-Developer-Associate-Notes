

24 AWS Security and Encryption
===



## Table of Contents

- [24 AWS Security and Encryption](#24-aws-security-and-encryption)
  * [Table of Contents](#table-of-contents)
  * [AWS KMS](#aws-kms)
    + [Why Encryption?](#why-encryption-)
      - [Encryption in Flight (SSL)](#encryption-in-flight--ssl-)
      - [Server Side Encryption at Rest](#server-side-encryption-at-rest)
      - [Client Side Encryption](#client-side-encryption)
    + [AWS KMS Introduction](#aws-kms-introduction)
      - [More KMS](#more-kms)
    + [Customer Master Key (CMK) Types](#customer-master-key--cmk--types)
    + [Copying Snapshots across Regions](#copying-snapshots-across-regions)
    + [KMS Key Policies](#kms-key-policies)
    + [Copying Snapshots across Accounts](#copying-snapshots-across-accounts)
    + [KMS API - Encrypt & Decrypt](#kms-api---encrypt---decrypt)
    + [Envelope Encryption](#envelope-encryption)
      - [Deep Dive](#deep-dive)
      - [Encryption SDK](#encryption-sdk)
    + [API Summary](#api-summary)
    + [KMS Request Quotas](#kms-request-quotas)
    + [Console](#console)
      - [CLI usage of Key](#cli-usage-of-key)
      - [Encryption SDK CLI](#encryption-sdk-cli)
      - [KMS and Lambda](#kms-and-lambda)
  * [S3 Security](#s3-security)
    + [S3 Encryption for Objects](#s3-encryption-for-objects)
    + [SSE-KMS](#sse-kms)
      - [Deep Dive](#deep-dive-1)
    + [S3 Bucket Policy - Force SSL](#s3-bucket-policy---force-ssl)
    + [S3 Bucket Policy - Force Encryption of SSE-KMS](#s3-bucket-policy---force-encryption-of-sse-kms)
  * [SSM Parameter Store](#ssm-parameter-store)
      - [SSM Param Store Hierarchy](#ssm-param-store-hierarchy)
      - [Standard and Advanced Param Tiers](#standard-and-advanced-param-tiers)
      - [Parameters Policies for Advanced Params](#parameters-policies-for-advanced-params)
    + [Console](#console-1)
      - [With Lambda](#with-lambda)
  * [AWS Secrets Manager](#aws-secrets-manager)
    + [SSM Param Store vs Secrets Manager](#ssm-param-store-vs-secrets-manager)
    + [Console](#console-2)
  * [CloudWatch Logs Encryption](#cloudwatch-logs-encryption)
    + [Console](#console-3)
  * [CodeBuild Security](#codebuild-security)



AWS KMS
---
### Why Encryption?
#### Encryption in Flight (SSL)
- data encrypted before sending and decrypted after receiving
- ssl certs help with encryption (https)
    - green lock on browser
- encryption in flight ensures no MITM can happen

![](https://i.imgur.com/bdm9Bqp.png)

#### Server Side Encryption at Rest
- data encrypted after being received by server
    - decrypted before sent
- stored in encrypted form using a data key
- encryption/decryption keys must be managed somewhr and server must have access to it

![](https://i.imgur.com/QoUB4tN.png)

#### Client Side Encryption
- data encrypted by client and nvr decrypted by server
    - data will be decrypted by receiving client
    - data stored on server but server dont know what it means
- server shldnt be able to decrypt data
- can leverage envelope encryption

![](https://i.imgur.com/AVoQNkL.png)


### AWS KMS Introduction
- anything related to encryption, it's kms
- easy way to control access to data
    - aws manages keys for us
- fully integrated with iam for authorisation
- seamlessly integrated into a lot of services
    - amazon ebs
        - encrypt volumes
    - amazon s3
        - server side encryption of objs
    - amazon redshift
        - encryption of data
    - amazon rds
        - encryption of data
    - amazon ssm
        - param store
    - etc
- can also use cli/sdk
- able to fully manage keys and policies
    - create
    - rotate policies
    - disable
    - enable
- able to audit key usage using cloudtrial
    - see who use keys and whenz
- 3 types of CMK
    - aws managed service default cmk
        - free
    - user keys created in kms
        - $1/month
    - user keys imported
        - must be 256bit symmetric key
        - $1/month
    - + pay for api call to kms
        - $0.03 / 10000 calls

#### More KMS
- use kms anytime u need to share sensitive info
    - db passwords
    - creds to external services
    - priv key for ssl certs
- value in kms is that the cmk used to encrypt data can nvr be retrieved by the user
    - cmk can also be rotated for extra security
- never ever store secrets in plaintxt especially in code
    - however encrypted secrets can be stored in the code/env vars
- kms only help in encrypting up to 4kb of data per call
    - if data > 4kb, use envelope encryption
- to give access to kms to someone,
    - ensure key policy allows user
    - ensure iam policy allows api calls

### Customer Master Key (CMK) Types
- symmetric (AES-256 keys)
    - 1st offering of kms, single encryption key used to encrypt and decrypt
    - aws services integrated with kms use symmetric cmks
    - needed for envelope encryption
    - nvr get access to the key unencrypted
        - must call kms api to use
- asymmetric (RSA & ECC key pairs)
    - public (encrypt) and private key (decrypt) pair
        - pub key is downloadable but can access priv key unencrypted
    - used for encrypt/decrypt or sign/verify operations
    - use case
        - encryption outside aws by users who cant call kms api


### Copying Snapshots across Regions
![](https://i.imgur.com/WnGaugV.png)
- key in region a cannot be transmitted to region b
- hence first create snapshot of volume
    - copy snapshot to new region but specify a new kms key to reencrypt the data with


### KMS Key Policies
- control access to kms keys similar to s3 bucket policies
    - diff - cannot control access w/o them
    - if u dont specify key policy then nobody can access your key
- default kms key policy
    - created if u dont provide specific kms key policy
        - is very permissive
    - complete access to key to root user = entire aws acc can use kms key
    - give access to iam policy to kms key to give user access to the key
- custom kms key policy
    - define users, roles that can access kms key
    - define who can administer key
    - useful for cross-account access of kms key


### Copying Snapshots across Accounts
- create snapshot encrypted with own cmk
- attach kms key policy to authorise cross acc access
- share encrypted snapshot
- in target, create copy of snapshot with kms key in its acc
- create vol from snapshot

![](https://i.imgur.com/yX0HNqZ.png)


### KMS API - Encrypt & Decrypt
![](https://i.imgur.com/TSJTHtR.png)
- encrypting
    - secret file less than 4kb
    - sent into kms service using encrypt apt
        - specify cmk 
    - kms then check with iam for permissions
    - kms sends encrypted secret
- decrypting
    - similar to encrypting
- though we are limited by size of file to be encrypted (4kb)

### Envelope Encryption
- kms encrypt api has limit of 4kb
    - if want to encrypt >4kb, use envelope encryption
- main api is `GenerateDataKey` api
- NOTE
    - for exam, anything >4kb must use envelope encryption == GenerateDataKey API

#### Deep Dive
![](https://i.imgur.com/5QFA3yI.png)
- big file 10mb
    - use gendatakey api to send file to kms
        - specify cmk
- kms check with iam perms if we can generate the key
- if yes, key generated and returned 
    - key returned in .dek (data encryption key) format
- key sent to client side - can encrypt the big file on client side uing our own cpu
    - returns both plaintxt key and encrypted key
- we build envelope around the encrypted file using the .dek key
    - includes encrypted file and encrypted dek key

![](https://i.imgur.com/I6ddqmR.png)
- encrypted envelope file is big, decrypt api only takes 4kb of data
    - hence only send encrypted dek
- kms decrypts dek and returns plaintxt dek key
    - use it to decrypt the encrypted file

#### Encryption SDK
- aws encryption sdk implements envelope encryption for us
    - encryption sdk also exists as cli tool to install
- implementations for java, python, C, js
- has feature - **Data Key Caching**
    - reuse data keys instead of creating new ones for ea encryption
    - reduces num of calls to kms with security trade-off
        - since u reuse keys
        - sec concern - using same key for many files
    - uses `LocalCryptoMaterialsCache`
        - max age, max bytes, max num of msgs

### API Summary
- `Encrypt`
    - encrypt up to 4kb of data through kms
- `GenerateDataKey`
    - generates unique symmetric data key (dek)
    - returns plaintxt copy of data key and encrypted copy of same cmk specified
- `GenerateDataKeyWithoutPlaintext`
    - generate dek to use at some pt (not immediately)
    - dek encrypted under the cmk that you specify
        - must use decrypt ltr to use it
    - envelope encryption uses the prev api not this one
        - exam might try to trick you
- `Decrypt`
    - decrypt up to 4kb of data
        - includes dek keys
- `GenerateRandom`
    - returns random byte string

### KMS Request Quotas
- when u exceed a req quota, u get `ThrottlingException`
    - to respond, use exponential backoff
        - backoff and retry
- for crypto operations, they share a quota
    - includes reqs made aws on your behalf
        - eg. SSE-KMS
- 2 other options to counter throttling
    - for gendatakey, consider using dek caching from encryption sdk
    - can request quota increase through api or aws support

![](https://i.imgur.com/6FPoKzE.png)
- symmetric cmk quota differs by region






### Console
![](https://i.imgur.com/PB9DiLU.png)
- all aws managed keys here
    - cuz they start with aws/


- if want to use key, wil ref alias
- the key cannot be used outside of its service

![](https://i.imgur.com/9GhrVuF.png)
- visaservice clause decides what service can use this key

![](https://i.imgur.com/TA1vvKr.png)
- out our own keys here
    - need to pay $1 per month to do that

![](https://i.imgur.com/Ra6iNEp.png)
- for aws cloudhsm cluster
    - out of scope for exam

![Uploading file..._qu4urvbud]()
- kms will generate key for us or we provide own key

![](https://i.imgur.com/ZTF0JP5.png)

![](https://i.imgur.com/LXxSK49.png)
- define key's admin
    - if leave this blank, will use default policy

![](https://i.imgur.com/7UTc15H.png)
- define who can use the key

![](https://i.imgur.com/oDBXox4.png)
- can also specify accounts for cross acc key usage instead of just using users

![](https://i.imgur.com/1Fl0mpm.png)

![](https://i.imgur.com/HB7fX4E.png)
- key actions

![](https://i.imgur.com/17FrNwK.png)
- crypto info of key

![](https://i.imgur.com/HrnYnXG.png)
- key rotation
    - rotate key every year

#### CLI usage of Key
![](https://i.imgur.com/ryQ7PYr.png)

![](https://i.imgur.com/2N1LOM4.png)
- encryption, specify
    - key alias
    - file to encrypt
    - output format
    - crypto algo
    - region
- the cmd returns a base64 file wiith your key


![](https://i.imgur.com/3RV8qdl.png)
- windows use 1st, linux 2nd


![](https://i.imgur.com/2Yz7YGU.png)
- decrypt the file (after u decoded base64)

#### Encryption SDK CLI
![](https://i.imgur.com/wCAfvaD.png)
- refer to doc to see installation guide
    - https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/crypto-cli-install.html

![](https://i.imgur.com/hWL7w14.png)

![](https://i.imgur.com/i0qqQmi.png)
- metadata file generated
    - can use `jq` command to process the json

![](https://i.imgur.com/4sYOuvR.png)
![](https://i.imgur.com/D4AzpJ4.png)

#### KMS and Lambda
![](https://i.imgur.com/BRkgcg2.png)
- defining env var in lambda
    - if user has access to env var, can still see our secret var
    - hence can use encryption console within the lambda console

![](https://i.imgur.com/ykBjDqW.png)
- specify key to encrypt var with
- must also copy snippet of code and paste into lambda func code

![](https://i.imgur.com/X7veyaG.png)
![](https://i.imgur.com/BUXv3sX.png)
- lambda func might timeout if encrypt/decrypt taking too much time
    - change your func timeout in its basic settings 

![](https://i.imgur.com/jwxoPfP.png)
- accessdenied error if iam role dont have perms to call decrypt api

![](https://i.imgur.com/3FJNkxF.png)
- attach inline policy
    - give specific arn the abi to call decrypt api


![](https://i.imgur.com/9ykUv96.png)



S3 Security
---
### S3 Encryption for Objects
- 4 methods of encrypting objs in s3
    - SSE-S3
        - encrypts s3 objs using keys handled & managed by aws
    - SSE-KMS
        - leverage aws kms to manage encrypted keys
    - SSE-C
        - manage own encryption keys
    - client side encryption
- NOTE
    - need to understand which ones to use for which situation

### SSE-KMS
- encryption keys handled and managed by kms
    - kms advantages
        - user control
        - audit trial
- obj encrypted server side
- must set header `"x-amz-server-side-encryption":"awskms"`

![](https://i.imgur.com/EANuE85.png)

#### Deep Dive
- sse-kms leverages the `GenerateDataKey` and `Decrypt` kms api calls
    - these kms api calls will show up in cloudtrial
        - helpful for logging
- to perform sse-kms, u need
    - kms key policy that authorises the user/role
    - iam policy that authorises access to kms
        - else will get access denied error
- s3 calls kms for sse-kms count against your kms limits
    - if throttling, try exponential backoff
        - or request inc in kms limits if throttling way too much/often
    - service throttled is kms, not s3

### S3 Bucket Policy - Force SSL
- to force ssl, create s3 bucket policy with deny on the condition `aws:SecureTransport = false`
    - using allow on that condition will allow anonymous `GetObject` if using ssl
    - [read more](https://aws.amazon.com/premiumsupport/knowledge-center/s3-bucket-policy-for-config-rule/)

![](https://i.imgur.com/p1y0nDp.png)

### S3 Bucket Policy - Force Encryption of SSE-KMS
- deny incorrect encryption header
    - make sure includes `aws:kms` == SSE-KMS
- deny no encryption header to ensure objs not uploaded unencrypted
    - or can also config default encryption to SSE-KMS

![](https://i.imgur.com/4KpYSZ5.png)


SSM Parameter Store
---
- secure storage for config and secrets
- optional seamless encryption using kms
- serverless, scalable, durable, easy sdk
- ver tracking of configs/secrets
- config management using path and iam
- notifs with cloudwatch events
- integration with cloudformation

![](https://i.imgur.com/ge4FOMN.png)

#### SSM Param Store Hierarchy
![](https://i.imgur.com/DHQ6RFj.png)

#### Standard and Advanced Param Tiers
![](https://i.imgur.com/K0A6gEi.png)

#### Parameters Policies for Advanced Params
- allow to assign ttl to param (exp date) to force updating/deleting sensitive data like passwords
- can assign multiple policies at once

![](https://i.imgur.com/Gzy3FLa.png)


### Console

![](https://i.imgur.com/X6B8H8a.png)

- param store can be found in secrets manager service

![](https://i.imgur.com/VsHltxx.png)

![](https://i.imgur.com/gH59fXZ.png)
![](https://i.imgur.com/vcKh4rE.png)
![](https://i.imgur.com/LW5cncz.png)
- encrypt param val using kms key

![](https://i.imgur.com/esdiPKc.png)

![](https://i.imgur.com/IVVo0pv.png)
- use ssl to get params from param store
- secure string returns encrypted value when `--with-decryption` flag is on

![](https://i.imgur.com/0dwe0Rz.png)

![](https://i.imgur.com/4xYLW5u.png)
- get param value based on path
- gives abi to organise our secrets

#### With Lambda
![](https://i.imgur.com/nvgQBKq.png)
- change lambda func to include boto and call ssm store from code

![](https://i.imgur.com/NfdwpPX.png)
- can also use env vars in code to call param from param store


AWS Secrets Manager
---
- newer service for storing secrets
- can force rotation of secret every x days
- automate generation of secrets on rotation using lambda
- integrate with amazon rds
    - eg. mysql, postgresql, aurora
- secrets encrypted using kms
- mostly meant for rds integration

### SSM Param Store vs Secrets Manager
- secrets manager
    - automatic rotation of secrets with aws lambda
    - integration with rds, redshift, documentdb
    - kms encryption mandatory
    - can integrate with cloudformation
- ssm param store
    - simple api
    - no secret rotation
    - kms encryption optional
    - can integrate with cloudformation
    - can pull secrets manager using ssm param store api






### Console
![](https://i.imgur.com/5dIs9Gq.png)
- can have creds and key value pairs

![](https://i.imgur.com/ZAI3qQD.png)

![](https://i.imgur.com/AEp5f32.png)

![](https://i.imgur.com/vwaiIn5.png)
- sample code to get your secret value

![](https://i.imgur.com/UR59OzQ.png)
- for other secret type, you can specify db to integrate with


CloudWatch Logs Encryption
---
- can encrypt cloudwatch logs with kms keys
- encryption enabled at log grp lvl
    - associate cmk with log grp when creating log grp or after it exists
- cannot associate cmk with log grp using cloudwatch console
    - must use cloudwatch logs api
        - `associate-kms-key` - if log grp alr exists
        - `create-log-grp` - if log grp doesnt exist yet


### Console
![](https://i.imgur.com/0a9sRP5.png)
- kms key id is blank

![](https://i.imgur.com/XaLZgAO.png)
- key policy need to be applied to allow access to it

![](https://i.imgur.com/AgiboyR.png)
- allow cloudwatch logs with specific action

![](https://i.imgur.com/DAj4h4V.png)


CodeBuild Security
---
- to access res in your vpc, make sure u specify vpc config for your codebuild
- secrets in codebuild
    - dont store as plaintxt in env vars
    - have env var reference param store params
        - or ref secrets manager secrets

### Console
![](https://i.imgur.com/04QNmzp.png)

- in codebuild specify param type
    - or secrets manager
- ensure iam policy allows access to these 2 services






###### tags: `AWS Developer Associate` `Notes`