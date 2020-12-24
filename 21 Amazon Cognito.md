

21 Amazon Cognito
===




## Table of Contents

- [21 Amazon Cognito](#21-amazon-cognito)
  * [Table of Contents](#table-of-contents)
  * [Amazon Cognito](#amazon-cognito)
    + [Cognito User Pools](#cognito-user-pools)
      - [User Features](#user-features)
      - [Integrations](#integrations)
      - [Lambda Triggers](#lambda-triggers)
      - [Hosted Auth UI](#hosted-auth-ui)
    + [Cognito Identity Pools (Federated Identities)](#cognito-identity-pools--federated-identities-)
      - [Example](#example)
      - [Example with CUP](#example-with-cup)
      - [Identity Pools and IAM Roles](#identity-pools-and-iam-roles)
      - [CIP IAM Policy Example](#cip-iam-policy-example)
    + [CUP vs CIP](#cup-vs-cip)
    + [Cognito Sync](#cognito-sync)
    + [Console](#console)
      - [Cognito Identity Pools](#cognito-identity-pools)



Amazon Cognito
---
- used to give users an identity so they can interact with our app
    - not iam users but users outside of your cloud
- cognito user pools
    - sign in functionality for app users
    - integrate with api gateway and alb
- cognito identity pools (federated identity)
    - provide aws credentials to users to they can access aws res directly
    - integrate with cognito user pools as identity provider
- cognito sync
    - sync data from mobile device to cognito
    - is deprecated and replaced by AppSync
- cognito vs iam
    - for hundreds of users, mobile users or auth with SAML
    - iam for users u trust within your aws env
        - cognito for all other outsider users

### Cognito User Pools
#### User Features
- way to create serverless db of users for your web and mobile apps
- users can do
    - simple login
        - username or email
        - password
    - password reset
    - email and phone number verification
    - mfa
    - federated identities
        - users from fb, google, SAML etc.
    - feature
        - block users if their credentials compromised elsewhere
        - aws scans web for compromised credentials
- login sends back json web token (jwt)

![](https://i.imgur.com/cvstVZ5.png)
- cognito has own db of users that we can see
    - apps login against cognito user pool
        - returns jwt when login success
    - can also do federation through 3rd party identity providers
        - social identity provider
            - google or fb etc.
        - or more specific identity providers
            - eg. SAML, OpenID Connect

#### Integrations
- cup (cognito user pools) integrates with api gateway and alb natively 

![](https://i.imgur.com/eCWzd5N.png)
- user auth with cup and get jwt token
    - pass token to api gateway
    - api gateway evaluates token with cognito
    - allow access to backend
- can also use alb + listeners and rules
    - auth users against cup
    - once done forward users to backend in target grp
        - which can consist of ec2 instances, lambda funcs, ecs containers etc.

#### Lambda Triggers
- cup can invoke lambda func synchronously on these triggers
    - https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-identity-pools-working-with-aws-lambda-triggers.html

![](https://i.imgur.com/0Ikor8V.png)

#### Hosted Auth UI
- cognito has hosted auth ui that u can add to app to handle signup and signin workflows
- using hosted ui, have foundation for integration with social logins
    - OIDC or SAML
- can customise with a custom logo and custom css

### Cognito Identity Pools (Federated Identities)
- get identities for users so they obtain temp aws creds
- your identity pool can include
    - public providers
        - eg. login with amazon, fb, google, apple etc.
    - users in amazon cup
    - openid connect providers and SAML identity providers
    - developer authenticated identities
        - eg. custom login server
    - unauthenticated guest access
        - yes identity pools allows this
- users then access aws services directly or through api gateway
    - iam policies applied to creds defined in cognito
        - can be customised based on user_id for fine grained control

#### Example
![](https://i.imgur.com/5cvWhYG.png)
- want to allow web/mobile apps to get aws creds but dw to create iam user
- leverage cognito identity pools
    - user first connects to cognito user pool or 3rd pt auth server
    - talk to identity pool service to exchange the token for temp aws creds
        - identity pool first verifies login with provider
        - once validated, talk to STS (security token service) service to get temp creds for the user
    - creds returned to user
    - can access aws services using the creds and associated iam policy

#### Example with CUP
![](https://i.imgur.com/ORoWwBZ.png)
- want user identity to be stored in cup
    - user login and get token from cup so all users centralised in cup db
    - can also enable federated identity providers for cup
- webapp then exchanges token from cup with cip for creds
    - cip talk to sts service
- get creds and access to aws services

#### Identity Pools and IAM Roles
- default iam roles for authenticated and guest users
- define rules to choose role for ea user based on user's id
    - can partition user's access using __policy variables__
        - also customise user's policy
- iam creds obtained by cip through STS (security token service)
- roles must have trust policy of cip

#### CIP IAM Policy Example
__Guest Users__
![](https://i.imgur.com/PjbEBSj.png)
- iam policy that allows any guest user to do get obj on a s3 bucket

__Policy Variable on S3__
![](https://i.imgur.com/aGFs7kG.png)
- define policy var on our s3
    - users are connected but only have access to a prefix in your s3 bucket that represents what the user's identity is
- 1st green rect - policy var
    - allow user to only access within the bucket we've defined (which is anything that starts with the prefix of its user id)

__DynamoDB__
![](https://i.imgur.com/3lurqUW.png)
- allow user to do anything in dynamodb as long as the leading key corresponds to the user id of the user
    - effectively achieving row-based security

### CUP vs CIP
- cognito user pools
    - db of users for your web and mobile app
    - allows federate logins through public social, OIDC, SAML etc.
    - can customise hosted UI for auth
        - includes logo
    - has triggers with aws lambda during auth flow
- cognito identity pools
    - obtain aws creds for your users
    - users can login through public social, OIDC, SAML and cup
    - users can be unauthenticated (guests)
    - users mapped to iam roles and policies, can leverage policy vars
- CUP + CIP = manage user/pwd + access to aws services

### Cognito Sync
- deprecated - use aws appsync now
- is service to store preferences, config and state of app
- cross device sync
    - any platform - ios, android etc.
- office capability
    - sync when back online
- store data in dataset up to 1mb
    - up to 20 datasets to sync
- push sync
    - silently notify across all devices when identity data changes
- cognito stream
    - stream data from cognito into kinesis
- cognito events
    - execute lambda funcs in resp to events






### Console
![](https://i.imgur.com/qRU7rUI.png)
![](https://i.imgur.com/Pt0Acuv.png)
- decide how users signin - email or phone?

![](https://i.imgur.com/EknumW7.png)
- attrs cannot be changed once they selected

![](https://i.imgur.com/4vwEfou.png)
- set password policy

![](https://i.imgur.com/3w2HKEq.png)
- mfa
    - users can use number of mfa device to signin
- how to recover account
- attrs to verify

![](https://i.imgur.com/PJFupeh.png)
- give cognito role if want to send sms

![](https://i.imgur.com/sdAQ6VD.png)
- if send emails whr do we send them from

![](https://i.imgur.com/TTb5CiV.png)
![](https://i.imgur.com/ekBSw2Z.png)
![](https://i.imgur.com/JFBvCsn.png)
![](https://i.imgur.com/tEfUPmk.png)
- need to create app client
    - allow us to login into user pool


![](https://i.imgur.com/5gTd6q2.png)

- go to app client settings on sidebar
    - define how we login into app - use cup
    - provide callback url
        - is url we get redirected to if login successful

![](https://i.imgur.com/bi5WIFV.png)
- can tick all oauth settings

![](https://i.imgur.com/3ZYF0WV.png)
- hosted ui
    - allow users to login through UI that cognito did for us
    - shld have launch hosted ui button but it's not thr currently since we did not specify our domain name yet

![](https://i.imgur.com/pbr2VVg.png)
- go to domain name from sidebar
    - choose domain name

![](https://i.imgur.com/h9Qon3j.png)
- can customise the ui from ui customisation in sidebar

![](https://i.imgur.com/DllrSLv.png)
- can also set custom logo

![](https://i.imgur.com/UG5kUf2.png)

![](https://i.imgur.com/GqS7QKR.png)
- user created from hosted ui

![](https://i.imgur.com/5pJ3dND.png)

- can also create user manually

![](https://i.imgur.com/UA4owrs.png)
- go to identity providers from sidebar to do federated login 

![](https://i.imgur.com/qoKjzNa.png)
- need to provide extra configs for specific identity provider

![](https://i.imgur.com/UHguxlk.png)
- invoking lambda functions triggers from trigger tab in sidebar
    - invoke funcs based on certain cognito events
    - 10 triggers total

#### Cognito Identity Pools
![](https://i.imgur.com/V2f7YFT.png)
- click on federated identities to switch to cip

![](https://i.imgur.com/yNxfYAG.png)
- option to allow guest users (unauth identities)

![](https://i.imgur.com/OOTOaJO.png)

![](https://i.imgur.com/TSHXZn3.png)
- has all listed providers and also direct integration with cup
    - provide cup pool id and app client id
        - get it from your cup console

![](https://i.imgur.com/AQRznVL.png)

![](https://i.imgur.com/iVWUYZx.png)
- 2 iam roles
    - one for auth and unauth identities

![](https://i.imgur.com/HT0s4AU.png)
- need to download sdk for target platform
    - get this from the sample code section in nav bar

![](https://i.imgur.com/sgAui20.png)
- dashboard from navbar
    - can see num of auth and guest users

![](https://i.imgur.com/VvrNttL.png)
- can search by identity id when users login

![](https://i.imgur.com/v2U8x8A.png)
- 2 roles created
    - are the roles we need to customise to decide what our authenticated/guest users have access to in aws

![](https://i.imgur.com/eKyiHiA.png)
- edit identity pool with button on top right of console

![](https://i.imgur.com/u0HLNd8.png)
- edit push sync
    - if user made changes on one device, can silently be pushed to all other devices

![](https://i.imgur.com/QM8Sslg.png)
- cognito streams
    - allow us to push every dataset change in cognito into a kinesis stream in realtime
    - enable realtime processing of these events

![](https://i.imgur.com/VZbbQWh.png)
- cognito events
    - run lambda funcs in resp to events in cognito






###### tags: `AWS Developer Associate` `Notes`