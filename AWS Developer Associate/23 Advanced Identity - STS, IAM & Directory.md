

23 Advanced Identity - STS, IAM & Directory
===




## Table of Contents

- [23 Advanced Identity - STS, IAM & Directory](#23-advanced-identity---sts--iam---directory)
  * [Table of Contents](#table-of-contents)
  * [AWS STS - Security Token Service](#aws-sts---security-token-service)
    + [Using STS to Assume a Role](#using-sts-to-assume-a-role)
    + [Cross Account Access with STS](#cross-account-access-with-sts)
    + [STS with MFA](#sts-with-mfa)
  * [Advanced IAM](#advanced-iam)
    + [Authorisation Model Evaluation of Policies](#authorisation-model-evaluation-of-policies)
    + [IAM Policies and S3 Bucket Policies](#iam-policies-and-s3-bucket-policies)
      - [Examples](#examples)
    + [Dynamic Policies with IAM](#dynamic-policies-with-iam)
    + [Inline vs Managed Policies](#inline-vs-managed-policies)
    + [Perms to Pass Role to AWS Service](#perms-to-pass-role-to-aws-service)
    + [Console](#console)
      - [Trust Policy](#trust-policy)
  * [Active Directory](#active-directory)
    + [AWS Active Directory Services](#aws-active-directory-services)
    + [Console](#console-1)
  * [Quiz](#quiz)



AWS STS - Security Token Service
---
- allow to grant limited and temp access to aws res
    - up to 1h
- common apis
    - `AssumeRole`
        - assume roles within your acc/cross acc
    - `AssumeRoleWithSAML`
        - return creds for users logged with SAML
    - `AssumeRoleWithWebIdentity`
        - return creds for users logged with IDP
            - eg. fb login, google login, OIDC compatible
        - aws recommends against using this and use cognito identity pools instead
    - `GetSessionToken`
        - for mfa, from user or aws acc root user
    - `GetFederationToken`
        - obtain temp creds for federated user
    - `GetCallerIdentity`
        - return details abt iam user or role used in api call
    - `DecodeAuthorizationMessage`
        - decode error msg when aws api denied
- NOTE
    - assumerole, getsessiontoken, getcalleridentity and decodeauthmsg shld be focused for dev associate exam

### Using STS to Assume a Role
- define iam role within acc or cross acc
- define which principals can access the iam role
    - then authorise everything using iam policies
- use aws sts to retrieve creds and impersonate iam role u have access to
    - AssumeRole API
- temp creds can be valid between 15mins and 1h

![](https://i.imgur.com/GEoviY7.png)
- user use assumerole api on sts
- sts check if perms correct
    - if yes, return aws creds to allow user to act as if he had the role

### Cross Account Access with STS
![](https://i.imgur.com/D3qfTOB.png)
- create role in another acc
- write correct perms into own acc and target acc
- finally run AssumeRole api to access target accs
- https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_common-scenarios_aws-accounts.html

### STS with MFA
- use `GetSessionToken` from sts to get session token
    - returns
        - access id
        - secret key
        - session token
        - expiration date
- appropriate iam policy using iam conditions
    - add `aws:MultiFactorAuthPresent:true` in iam policy
    

![](https://i.imgur.com/VxztTeG.png)



Advanced IAM
---
### Authorisation Model Evaluation of Policies
- if there's explicit DENY, end decision and deny
- if there's an ALLOW, end decison with allow
- else 
- NOTE
    - if there's both explicit deny and allow, deny wins

![](https://i.imgur.com/207LCIu.png)

### IAM Policies and S3 Bucket Policies
- iam policies attached to users, roles and grps
    - s3 bucket policies are attached to buckets
- when evaluating if iam principal can perform operation on a bucket, the **union** of its assigned iam and bucket policy will be evaluated

![](https://i.imgur.com/Le8y3Db.png)

#### Examples
![](https://i.imgur.com/1i626YH.png)
![](https://i.imgur.com/mqBchHc.png)
![](https://i.imgur.com/e0VCVNI.png)
![](https://i.imgur.com/Pn402BD.png)

### Dynamic Policies with IAM
- how do u assign ea user a `/home/<user>` folder in s3 bucket?
- option 1
    - create iam policy allowing ea user to have access to their own home dir
        - 1 policy per user
    - not scalable
- option 2
    - create 1 dynamic policy with iam
    - leverage special policy var `${aws:username}`
    - have policy that is customised per user basis

![](https://i.imgur.com/W70G3VJ.png)

### Inline vs Managed Policies
- aws managed policy
    - maintained by aws
    - good for power users and admins
    - updated in case of new services/apis
- customer managed policy
    - created by you
    - best practice by aws documentation
        - reusable
        - can be applied to many principals
    - version controlled and rollback
    - central change management
        - see who did what and whom
- inline
    - directly within a principal
    - strict one-to-one r/s between policy and principal
    - not version controlled, cannot be rollback
        - cannot be altered easily
    - policy deleted if iam principal deleted

### Perms to Pass Role to AWS Service
- to config many aws services, must pass an iam role to service
    - happens once during setup
- service will ltr resume the role and perform actions
- example of passing a role
    - to ec2/lambda/ecs task/codepipeline to allow it to invoke other services
- to grant user permissions to pass a role to an aws service, you need `iam:PassRole`
    - often comes with `iam:GetRole` to view role being passed

![](https://i.imgur.com/VYgITYI.png)
- allow u to passrole s3 access
- NOTE
    - roles can only be passed to what their trust allows
        - a trust policy for the role that allows the service to assume the role

![](https://i.imgur.com/nqnPa5V.png)
- only ec2 service has the trust to allow to assume that role 



### Console
![](https://i.imgur.com/pSjs1pI.png)

![](https://i.imgur.com/4iDIOCV.png)
- filter policies based on type
    - inline policies not found here

![](https://i.imgur.com/85RcwyC.png)

- go to iam > users > add inline policy

![](https://i.imgur.com/424kjHY.png)
- error means u selected too many attributes for your policy

#### Trust Policy
![](https://i.imgur.com/v9DBabT.png)
- trusted entities col
    - allow service to assume the role
 
![](https://i.imgur.com/y7b4w4r.png)
- eg. aws codepipeline has trust r/s tab

![](https://i.imgur.com/YPPAjMo.png)


Active Directory
---
- microsoft active dir
    - found on any windows server with ad domain services
    - is db of objs
        - eg. user accs, comps, printers, file shares, sec grps
        - objs organised in **trees**
        - grp of trees is **forest**
    - is centralised security management, acc creation and perms assigning

![](https://i.imgur.com/Kl9NW9V.png)
- eg. if login with john on one machine, it'll look it up in dc to verify

### AWS Active Directory Services
- aws managed microsoft ad
    - create your own ad in aws
        - manage users locally
        - support mfa
    - establish trust conns with your on-premise ad
        - if users create acc on on-premise ad, it will be trusted to look it up on other ad and vice versa
        - users shared between both ad
- ad connector
    - dir gateway (proxy) to redirect to on-premise ad
        - user auths on ad conn will proxy to on premise ad and look it up
    - users managed on on-premise ad
- simple ad
    - ad-compatible managed dir on aws
        - can create ad with ec2 instances running windows
            - these ec2 instances can join the domain controllers for your network and share all logins and creds
    - cannot be joined with on-premise ad

![](https://i.imgur.com/rYk1e9n.png)

### Console
![](https://i.imgur.com/G7wIpLB.png)
![](https://i.imgur.com/6pxYQtt.png)
![](https://i.imgur.com/fhqV5Bz.png)


Quiz
---
![](https://i.imgur.com/MWRh9zW.png)
![](https://i.imgur.com/aWhibpa.png)



###### tags: `AWS Developer Associate` `Notes`