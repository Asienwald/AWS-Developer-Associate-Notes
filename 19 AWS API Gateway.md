

19 AWS API Gateway
===




## Table of Contents

- [19 AWS API Gateway](#19-aws-api-gateway)
  * [Table of Contents](#table-of-contents)
  * [AWS API Gateway](#aws-api-gateway)
    + [Integrations High Level](#integrations-high-level)
    + [Endpoint Types](#endpoint-types)
    + [Deployment Stages](#deployment-stages)
      - [Stage Variables](#stage-variables)
    + [Canary Deployment](#canary-deployment)
    + [Integration Types](#integration-types)
      - [Mapping Templates (AWS & HTTP Integration)](#mapping-templates--aws---http-integration-)
      - [Mapping Example](#mapping-example)
    + [API Gateway Swagger/Open API Spec](#api-gateway-swagger-open-api-spec)
    + [Caching](#caching)
      - [Caching API Responses](#caching-api-responses)
      - [Cache Invalidation](#cache-invalidation)
    + [Usage Plans and API Keys](#usage-plans-and-api-keys)
      - [Correct Order for API Keys](#correct-order-for-api-keys)
    + [Logging and Tracing](#logging-and-tracing)
    + [API Gateway Throttling](#api-gateway-throttling)
      - [Errors](#errors)
    + [CORS](#cors)
    + [API Gateway Security](#api-gateway-security)
      - [IAM Permissions](#iam-permissions)
      - [Resource Policies](#resource-policies)
      - [Cognito User Pools](#cognito-user-pools)
      - [Lambda Authoriser (formerly Custom Authorisers)](#lambda-authoriser--formerly-custom-authorisers-)
      - [Security Summary](#security-summary)
    + [HTTP vs REST vs Websocket API](#http-vs-rest-vs-websocket-api)
    + [Console](#console)
      - [Stages](#stages)
      - [Stage Options](#stage-options)
      - [Canary](#canary)
      - [Mapping Templates](#mapping-templates)
      - [Import/Export Swagger/OpenAPI](#import-export-swagger-openapi)
      - [Caching](#caching-1)
      - [Usage Plans](#usage-plans)
      - [CORS](#cors-1)
      - [Security](#security)
  * [Quiz](#quiz)



AWS API Gateway
---
![](https://i.imgur.com/bU2nyM3.png)
- api gateway provides restapi endpts for clients to talk to and invoke lambda functions
- aws lambda + api gateway = no infrastructure to manage
- support for websocket protocol
    - can do real time streaming through the api gateway in 2 diff ways
- handle api versioning
    - v1, v2 etc.
- handle diff environments
    - dev, test, prod
- handle security
    - auth and authoristion
- create api keys and handle request throttling
- swagger/open api import/export to quickly define apis
- transform and validate requests and responses
    - to ensure invocations are correct
- generate sdk and api specifications
- cache api responses

### Integrations High Level
- lambda func
    - invoke lambda func
    - easy way to expose rest api backed by aws lambda
- http
    - expose http endpts in backend
    - eg. internal http api on premise, alb...
    - why? add rate limiting, caching, user auth, api keys etc.
        - useful to have layer of api gateway on top of http endpt
- aws service
    - can expose any aws api through api gateway
    - eg. start aws step func workflow, post msg to sqs
    - why? add auth, deploy publicly, rate control etc.

### Endpoint Types
- edge-optimised (default)
    - for global clients
    - requests routed through cloudfront edge locations
        - improve latency
    - api gateway still lives in only 1 region but accessible from every cloudfront edge location
- regional
    - for clients within same region 
    - could manually combine with cloudfront
        - similar to edge optimise more control over caching strategies and distribution
- private
    - can only be accessed from vpc using interface vpc endpt (ENI)
    - use res policy to define access

### Deployment Stages
- making changes in api gateway dont mean they're effective
    - need to make deployment for them to be in effect
- changes are deployed to stages
    - can have as many as u want
- use naming you like for stages
    - eg. dev, test, prod
- ea stage has own config params
- stages can be rolled back as histoy of deployments is kept
- eg. stages v1 and v2
![](https://i.imgur.com/2Aloosg.png)
- v1 clients accessing v1 stage api gateway and v1 lambda func
- v2 api gateway will have new url
    - clients can just change url and use new ver
        - slowly migrate to v2

#### Stage Variables
- stage vars are like env vars for api gateway
    - use to change often changing config values w/o redeploying api
- can be used in
    - lambda func arn
    - http endpoint
    - param mapping templates
- use cases
    - config http endpts your stages talk to
        - dev, test, prod
    - pass config params to lambda through mapping templates
- stage vars passed to context obj in lambda
    - can get stage vars directly from func

__API Gateway Stage Vars and Lambda Aliases__
- can create stage var to indicate corresponding lambda alias
    - api gateway automatically invoke right lambda func

![](https://i.imgur.com/SIBK3hl.png)

### Canary Deployment
- possibility to enable canary deployments for any stage
    - usually prod
- choose % of traffic the canary channel receives
- metrics and logs are separate
    - for better monitoring
- possibility to override stage vars for canary
- is blue/green deployment with aws lambda and api gateway

![](https://i.imgur.com/HFQV52m.png)
- % of clients visit the canary stage

### Integration Types
- integration type `MOCK`
    - api gateway returns resp w/o sending req to backend
- integration type `HTTP/AWS` (lambda and aws services)
    - api gateway forwards a req but we can modify it
    - must config both integration req and integration resp
    - setup data mapping using __mapping templates__ for req and resp

![](https://i.imgur.com/UedCz7E.png)

- integration type `AWS_PROXY` (lambda proxy)
    - incoming req from client is input to lambda
        - cannot modify req/resp as we're a proxy
    - func is responsible for logic of req/resp
    - no mapping template
        - cannot change headers, query string params etc.
        - they are passed as args to funcs directly
- eg. below
    - req from api gateway passed like json on the left
    - lambda func responsible to take this req, process it and return resp like on the right
    - all the work is on the backend and api gateway is just here to proxy the request through

![](https://i.imgur.com/1gO3ESr.png)


- integration type `HTTP_PROXY`
    - no mapping template cuz proxy
    - http req is passed to backend
    - http resp from backend is forwarded by api gateway

![](https://i.imgur.com/rBBnNYZ.png)

#### Mapping Templates (AWS & HTTP Integration)
- mapping templates can be used to modify req/resp
- rename/modify query string params
    - modify body content
    - add headers
    - use velocity template lang (VTL)
        - for loops, if etc.
        - is scripting lang to modify requests
    - filter output results
        - remove unnecessary data

#### Mapping Example
- eg. json to xml with SOAP
    - soap api are xml based whereas rest api are json based
- hence, api gateway should
    - extract data from req
        - either path, payload or header
    - build SOAP msg based on req data (mapping template)
    - call SOAP service and receive xml response
    - transform xml resp to desired format (like json) and respond to user
- api gateway thanks to mapping template can transform json payload into an xml payload

![](https://i.imgur.com/VyORjgW.png)

- eg. query string params
    - map json vals passed by client into sth lambda can make sense of

![](https://i.imgur.com/7eVt1Pd.png)

### API Gateway Swagger/Open API Spec
- common way of defining rest apis is using api definition as code
- import existing swagger/openapi 3.0 spec to api gateway
    - method
    - method req
    - integration request
    - method resp
    - + aws extensions for api gateway and setup every single action
- can export current api as swagger/openapi spec
    - can be written in yaml/json
    - using this we can generate sdk for our apps

### Caching
#### Caching API Responses
- caching reduces num of calls made to backend
    - default ttl is 300 seconds
        - min 0 seconds, max 3600 seconds
- caches are defined per stage
    - possible to override cache settings per method
- cache encryption option
- cache capacity between 0.5gb to 237gb
- cache is expensive
    - makes sense in prod but not in dev/test

![](https://i.imgur.com/rqjVjnN.png)

#### Cache Invalidation
- able to flush/invalidate entire cache immediately
- clients can invalidate cache with `header: Cache-Control: max-age=0`
    - needs proper iam perms
- if u dont impose an `InvalidateCache` policy or choose the require authorisation checkbox in console, any client can invalidate the api cache

![](https://i.imgur.com/qY4AS68.png)

### Usage Plans and API Keys
- if u want to make an api avail as an offering to your customers
- usage plan
    - who can access 1 or more deployed api stages and methods
    - how much and how fast they can access them
    - use api keys to identify api clients and meter access
    - config throttling limits and quota limits that are enforced on indiv client
- api keys
    - alphanumeric string vals to distribute to your customers to securely use your api gateway
        - eg. WBjHxNtoAb4WPKBC7cGm64CBibIb24b4jt8jJHo9
    - can use usage plans to control access
    - throttling limits applied to api keys
    - quota limits is overall num of max requests

#### Correct Order for API Keys
- to config usage plan
    - create 1 or more apis, config methods to require api key and deploy the api to stages
    - generate/import api keys to distribute to app devs (customers) who are using your api
    - create usage plan with desired throttle and quota limits
    - associate api stages and keys with usage plan
- callers of api must supply an assigned key in `x-api-key` header in requests to api

### Logging and Tracing
- cloudwatch logs
    - enable cloudwatch logging at stage lvl
        - with log lvl
    - can override settings on a per API basis
        - eg.ERROR, DEBUG, INFO
    - log contains info abt request/response body
- xray
    - enable tracing to get extra info abt requests in api gateway
    - xray api gateway + aws lambda gives u full pic
- cloudwatch metrics
    - metrics are by stage
        - possible to enable detailed metrics
    - `CacheHitCount` and `CacheMissCount`
        - efficiency of cache
    - `Count`
        - total num of api reqs in a given period
    - `IntegrationLatency`
        - time between when api gateway relays req to backend and when it receives a resp from the backend
    - `Latency`
        - time between when api gateway receives req from client and when it returns a resp to client
        - the latency includes integration latency and other api gateway overhead
    - `4XXError` (client side) and `5XXError` (server side)

### API Gateway Throttling
- account limit
    - api gateway throttles reqs at 10,000 rps across all api
    - soft limit that can be increased upon req
- in case of throttling => 429 too many reqs
    - retriable error
    - is client error when they do too many reqs
- can set stage limit and method limits to improve performance
    - to ensure ea stage dont use all quotas of reqs if it's under atk
    - or can define usage plans to throttle per customer
- just like lambda concurrency, 1 api if overloaded or limited can cause other apis to be throttled

#### Errors
- 4xx means client errors
    - 400 - bad request
    - 403 - access denied, waf filtered (firewall dont accept your request)
    - 429 - quota exceeded, throttle
- 5xx means server errors
    - 502
        - bad gateway exception
        - usually for incompatible output returned from a lambda proxy integration backend and occasionally for out-of-order invocations due to heavy loads
    - 503
        - service unavailable exception
    - 504
        - integration failure
            - eg. endpt request timeout exception
        - api gateway requests timeout after 29 seconds maximum

### CORS
- cors must be enabled when u receive api calls from another domain
- OPTIONS pre-flight req must contain following headers
    - `Access-Control-Allow-Methods`
    - `Access-Control-Allow-Headers`
    - `Access-Control-Allow-Origin`
- cors can be enabled through the console

![](https://i.imgur.com/IWgZusJ.png)
- browser goes to s3 to get static content
    - js in content from s3 bucket makes api calls to cross origin
    - hence browser as security will make OPTIONS pre-flight req to api gateway
        - api gateway gives us preflight resp if the origin is allowed to make a cross origin request
        - if everything is well, browser and gateway can talk to one another

### API Gateway Security
#### IAM Permissions
- create iam policy authorisation and attach to user/role
    - iam - auth
    - iam policy - authorisation
- good to provide access within aws
    - eg. ec2, lambda, iam users
- leverages sigv4 capability whr iam credentials are signed and placed in headers

![](https://i.imgur.com/jdLe10j.png)
- client first do restapi call and pass sigv4 headers to api gateway
    - api gateway decrypts those and check with iam if the user is authorised (iam policy check)
    - if done, apigateway is authorised and talks to lambda in backend

#### Resource Policies
- res policies
    - similar to lambda res policy
    - allow u to set adj policy on api gateway to define who and what can access it
- allow for cross acc access (is main use case) combined with iam security
    - allow for specific src ip addr
    - allow for vpc endpt

![](https://i.imgur.com/gEJKqyw.png)

#### Cognito User Pools
- cognito fully manages user lifecycle
    - token expires automatically
- api gateway verifies identity automatically from aws cognito
    - no custom implementation required
- auth - cognito user pools
    - authorisation - api gateway methods

![](https://i.imgur.com/enZWr3y.png)
- client first auths with cognito user pools to retrieve a conn token
    - passes token in api call to our api gateway
    - api gateway with direct integration with cognito user pools will evaluate the token with user pool
    - if token correct, allow access to your backend

#### Lambda Authoriser (formerly Custom Authorisers)
- token-based authoriser (bearer token)
    - eg. jwt or oauth
- request param based lambda authoriser
    - passed with headers, query string, stage var
- lambda must return an iam policy for the user
    - result policy is cached
- auth - external
    - authorisation - lambda func

![](https://i.imgur.com/d1JbcbY.png)
- client auths with 3rd party auth system eg. auth0
    - get token from 3rd party and pass to api gateway
        - either through header or request biometrics
    - api gateway integrated with lambda authoriser (which is a lambda func)
        - authoriser retrieve info around the context and token
        - the func is up to us to code so we'll decide how it communicates with the 3rd party to verify the token
    - if token valid, authoriser creates an iam principal and iam policy
    - policy cached inside a policy cache

#### Security Summary
- iam
    - great for users/roles alr within aws acc
        - and res policy for cross acc
    - handle auth and authorisation
    - leverages sigv4
- custom authoriser
    - great for 3rd party tokens
    - very flexible in terms of what iam policy is returned
    - handle auth verification and authorisation in lambda func
    - pay per lambda invocation
        - results cached
- cognito user pools
    - manage own user pool
        - can be backed by facebook, google login etc.
    - no need write any custom code
    - must implement authorisation in backend

### HTTP vs REST vs Websocket API
- http apis
    - low latency
    - cost effective aws lambda proxy, http proxy apis and private integration
        - no data mapping (all proxies)
    - support OIDC and OAuth 2.0 authorisation
    - built-in support for cors
    - no usage plans and api keys
    - is low cost alternative and only support proxy integration
- rest apis
    - all features except native OpenID Connect/OAuth 2.0
- https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-vs-rest.html

![](https://i.imgur.com/9usA6GV.png)

- websocket api
    - is 2 way interactive communication between a user's browser and a server
        - server can push info to client
        - this enables stateful app use cases
    - websocket apis are often used in realtime apps like chat apps, collab platforms, multiplayer games and financial trading platforms
    - works with aws services (lambda, dynamodb) or http endpts






### Console
![](https://i.imgur.com/dO2Fk59.png)
- http api
    - works with lambda or http backends
- websocket api
    - works with lambda, http and aws services
- rest api
    - and also restapi private for within vpc


![](https://i.imgur.com/tstujYT.png)
- rest api example
- choos endpt type
    - regional
        - deployed within this region
    - edge optimised
        - using cloudfront edge network
    - private
        - within vpc

![](https://i.imgur.com/hPA8ID8.png)
- use actions to create method

![](https://i.imgur.com/JNTPm7C.png)
- choose integration type
    - aws service can expose any service in any region
    - vpc link
        - when we have vpc inside of vbc (out of scope for this exam)
- lambda proxy integration
- also have to create lambda func for this endpt
- set timeout
    - lambda func has timeout of up to 15mins, but this default is 29 seconds max
        - can set custom timeout

![](https://i.imgur.com/mbokAjB.png)
![](https://i.imgur.com/X8caoBD.png)
- lambda res policy allows api gateway

![](https://i.imgur.com/bl2Q0Ih.png)
- there's method, integration request

![](https://i.imgur.com/i5thTu4.png)

- can click on endpt and test it
    - see resp body, headers and logs

![](https://i.imgur.com/qFNeyiy.png)
- example request sent to lambda from api gateway seen in cloudwatch logs

![](https://i.imgur.com/EfaEdsz.png)
- can create resource

![](https://i.imgur.com/uMrrKKX.png)
- choose http verb to use for endpt

![](https://i.imgur.com/RkAV3IH.png)
- to deploy api, go to actions

![](https://i.imgur.com/0JyIlyi.png)
- indicate stage name eg. dev

![](https://i.imgur.com/lgmcoYP.png)
- invoke url created for your use

#### Stages
![](https://i.imgur.com/0KsDsQF.png)
- create lambda alias

![](https://i.imgur.com/BmBBHEz.png)
- create method for your endpt

![](https://i.imgur.com/UubYPYp.png)
- paste your lambda func name into your method
    - but append to name `:${stageVariables.lambdaAlias}`

![](https://i.imgur.com/FT5K51e.png)
- need to use cli to add res based policy for each func version/alias to allow api gateway to invoke the corresponding func
    - replace function name param's last section after the `:` with the aliases of the function
    - currently the console doesnt do this for us

![](https://i.imgur.com/QcbttbS.png)
- test endpt with your stage variable

![](https://i.imgur.com/pYjCDdC.png)
- deploy api

![](https://i.imgur.com/coRdX9f.png)
- can go into your deployed api and set your stage variable depending on which deployment stage it's in

#### Stage Options
![](https://i.imgur.com/WPCpNKz.png)
- for ea stage can config
    - cache settings
    - method throttling
        - so not called too often
    - webapp firewall settings
    - client certs
        - for ssl login

![](https://i.imgur.com/QLDeE0b.png)
- logs and tracing
    - cloudwatch logs
    - details metrics
    - access logging
    - xray tracing

![](https://i.imgur.com/to7hsb2.png)
- generate code based on what u want (lang)

![](https://i.imgur.com/vFBGl7b.png)
- export api as either swagger or open api 3

![](https://i.imgur.com/Mojo60Z.png)
- deployment history which shows all deployments

![](https://i.imgur.com/Qlnq6oS.png)
- for stage documentation

![](https://i.imgur.com/G5f309e.png)
- canary


#### Canary
![](https://i.imgur.com/PgpWnHv.png)
- create canary in canary tab of your deployment stage
    - edit % directed to canary and prod

![](https://i.imgur.com/97hbJQv.png)
- can deploy api to only canary of a stage
    - clients directed based on %

#### Mapping Templates
![](https://i.imgur.com/RFxXAnE.png)
- click on integration response > mapping template > json

![](https://i.imgur.com/VlHpnaM.png)
- return `renamedExample` with val of `inputRoot.example` which is var in the lambda func
    - can change response however we want


#### Import/Export Swagger/OpenAPI
![](https://i.imgur.com/FCIKlma.png)
- when creating gateway, can choose to import as rest api

![](https://i.imgur.com/s2oF3BX.png)
![](https://i.imgur.com/Jegs8Ca.png)
- example api that describes your api

![](https://i.imgur.com/mezGyLn.png)
- can go to sdk generation tab of gateway to generate an sdk depending on lang
 
![](https://i.imgur.com/7WWju2y.png)
- export api
    - can choose json or yaml

![](https://i.imgur.com/XnFYjz7.png)

#### Caching
![](https://i.imgur.com/wPU1qfl.png)
- go to console settings and click enable api cache
- caching is not in free tier

![](https://i.imgur.com/c8Pv97Y.png)

- stop at 118gb but max is still 237gb

#### Usage Plans
![Uploading file..._a6i3s412p]()
- in method execution of method, can set api key required field to true

![](https://i.imgur.com/hHlBaWb.png)
- go to usage plan tab on sidebar


![](https://i.imgur.com/chD2t4X.png)

- config throttling
    - how many reqs per second i want to get max
- burst
    - how much i allow my customers to go over
- quota
    - eg. cannot do more than 10,000 requests per month
    - can change month to week or day
    
![](https://i.imgur.com/KSTNqYX.png)
![](https://i.imgur.com/s55S3KF.png)
 
- click add api stage to associate a stage

![](https://i.imgur.com/7MtBExG.png)
![](https://i.imgur.com/QuUUGXB.png)
 
- next create api keys

![](https://i.imgur.com/bMn4XEa.png)
- usage plan created

![](https://i.imgur.com/51HIfZA.png)
- can see usage of a specific customer
    - click on usage under api key tyab

![](https://i.imgur.com/ubxBaYf.png)
- can grant an extension of reqs to specific api key

![](https://i.imgur.com/IumdQr2.png)

- api key panel

![](https://i.imgur.com/r6Cl4pl.png)

![](https://i.imgur.com/OEsFQk7.png)
- can enable method throttling to specific method of gateway through your usage plan console

![](https://i.imgur.com/G6gdlvy.png)
- can sell api key directly on aws marketplace

![](https://i.imgur.com/o7DyDh2.png)
- will see message forbidden if your api key is not passed in the headers

![](https://i.imgur.com/XBI4GzH.png)
- get status 200 if key passed

#### CORS
![](https://i.imgur.com/526ZiVr.png)
- js in static content fetch sth from your api gateway url

![](https://i.imgur.com/NsDumZY.png)

![](https://i.imgur.com/dkLmq1w.png)
- go actions > enable cors for specific method u want

![](https://i.imgur.com/aqoA2nH.png)
- if leave access control allow origin to \*, it'll work for every website
    - can just change to the domain you'll be calling the fetch from
- also need to redeploy api for it to work

![](https://i.imgur.com/KTfLY0s.png)
- enabling cors might not work if your method is a proxy
    - can be seen from integration response window

![](https://i.imgur.com/jsmU8Y6.png)
- OR u can just manually add the header in your lambda func

#### Security
![](https://i.imgur.com/J8UTTOR.png)
- under method execution can set authorisation
    - currently only has iam

![](https://i.imgur.com/cK41bGd.png)
- use tgt with res policy
    - has 3 examples
        - aws acc whitelist
            - cross acc res policy
            - can replace with another acc id

![](https://i.imgur.com/N8D6Al3.png)
- ip range blacklist

![](https://i.imgur.com/Ipjokpu.png)
- src vpc whitelist
    - authorise another vpc into our gateway

![](https://i.imgur.com/Fm3iP23.png)
- go to authorisers on sidebar to create new authorisers

![](https://i.imgur.com/oYG3nII.png)

Quiz
---
![](https://i.imgur.com/fyZWxKL.png)
- i thought latency was a part of cloudwatch metrics instead of xray but ok


###### tags: `AWS Developer Associate` `Notes`