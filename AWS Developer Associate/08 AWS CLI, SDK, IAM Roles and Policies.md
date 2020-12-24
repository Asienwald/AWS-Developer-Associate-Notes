

08 AWS CLI, SDK, IAM Roles and Policies
===




## Table of Contents

- [08 AWS CLI, SDK, IAM Roles and Policies](#08-aws-cli--sdk--iam-roles-and-policies)
  * [Table of Contents](#table-of-contents)
  * [Developing on AWS](#developing-on-aws)
    + [Introduction](#introduction)
  * [AWS CLI and IAM Roles/Policies](#aws-cli-and-iam-roles-policies)
    + [AWS CLI Setup](#aws-cli-setup)
      - [CLI Installation Troubleshooting](#cli-installation-troubleshooting)
    + [AWS CLI Configuration](#aws-cli-configuration)
      - [Bad Config of AWS CLI on EC2](#bad-config-of-aws-cli-on-ec2)
      - [Good Config for AWS CLI on EC2](#good-config-for-aws-cli-on-ec2)
    + [AWS CLI Dry Runs](#aws-cli-dry-runs)
      - [AWS CLI STS Decode Errors](#aws-cli-sts-decode-errors)
    + [AWS EC2 Instance Metadata](#aws-ec2-instance-metadata)
    + [MFA with CLI](#mfa-with-cli)
    + [Console and CLI](#console-and-cli)
      - [AWS CLI](#aws-cli)
      - [IAM Roles on EC2](#iam-roles-on-ec2)
      - [Creating Own Policy](#creating-own-policy)
      - [AWS Policy Simulator](#aws-policy-simulator)
      - [CLI Dry Runs](#cli-dry-runs)
      - [EC2 Metadata](#ec2-metadata)
      - [AWS CLI Profiles - Multiple AWS Accounts on CLI](#aws-cli-profiles---multiple-aws-accounts-on-cli)
      - [MFA with CLI](#mfa-with-cli-1)
  * [CLI/API Resources](#cli-api-resources)
    + [S3](#s3)
    + [Generic CLI](#generic-cli)
  * [AWS SDK](#aws-sdk)
    + [AWS Limits (Quotas)](#aws-limits--quotas-)
      - [Exponential Backoff](#exponential-backoff)
  * [CLI/SDK Credentials Provider Chain](#cli-sdk-credentials-provider-chain)
    + [AWS CLI Credentials Provider Chain](#aws-cli-credentials-provider-chain)
    + [AWS SDK Default Credentials Provider Chain](#aws-sdk-default-credentials-provider-chain)
    + [Credentials Scenario](#credentials-scenario)
    + [AWS Credentials Best Practices](#aws-credentials-best-practices)
  * [Signing AWS API Requests](#signing-aws-api-requests)
    + [SigV4 Request Examples](#sigv4-request-examples)
  * [Quiz](#quiz)



Developing on AWS
---
### Introduction
- so far we've interacted with services manually and they exposed standard info to clients
    - ec2 exposes standard linux machine we can use any way we want
    - rds exposes standard db we can connect using url
    - elasticache exposes cache url we can conn using url
    - asg/elb automated and dont have to program against them
    - route53 was setup manually
- dev against aws has 2 components
    - how to perform interactions with aws w/o online console
        - doing stuff manually is bad
    - how to interact with aws proprietary services
        - Eg. s3, dynamodb etc.
- developing and performing aws tasks can be done in several ways
    - using aws cli on local comp
    - using aws cli on ec2 machines
    - using aws sdk on local comp/ec2 machines
    - using aws instance metadata service for ec2
- we'll learn
    - how to do all those in right & most secure way with best practices


AWS CLI and IAM Roles/Policies
---

### AWS CLI Setup
- windows
    - https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-windows.html
- mac os x
    - https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-mac.html
- linux
    - https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-linux.html

#### CLI Installation Troubleshooting
- if get error `aws: command not found` after installing cli
    - or `aws executable is not in the PATH env var` on linux/mac os
- PATH allows system to know whr aws exe is

### AWS CLI Configuration
- how to properly config cli
    - learn how to get access credentials & protect them
- dont share your aws access key and secret key with anyone

![](https://i.imgur.com/ZtXIb8P.png)

#### Bad Config of AWS CLI on EC2
- could run `aws configure` on ec2 and it'll work
    - but super insecure
- never put your personal credentials on ec2
    - personal creds only belong on personal comp
    - if ec2 compromised, so is your personal acc
        - if ec2 shared, other people may perform aws actions while impersonating you
- for ec2 there's better way - aws iam roles

#### Good Config for AWS CLI on EC2
- IAM roles can be attached to ec2 instances
    - iam roles can come with policy authorising exactly what ec2 instance shld be able to do
- ec2 instances can use these profiles automatically w/o additional configs
    - best practice on aws
    - DO NOT ever put your credentials on ec2

![](https://i.imgur.com/vhA9pCh.png)


### AWS CLI Dry Runs
- sometimes we want the perms but dw to actually run the commands
    - some aws cli cmds (Eg. ec2) can be expensive if they succeed
    - some aws cli cmds (not all) contain a `--dry-run` option to simulate api calls

#### AWS CLI STS Decode Errors
- when your run api calls and they fail, can get long error msg
    - this error msg can be decoded using STS cmd line
    - `sts decode-authorization-message`


### AWS EC2 Instance Metadata
- aws ec2 instance metadata is powerful but one of least known features to devs
- it allows ec2 instances to "learn about themselves" __w/o using IAM role__
    - URL is `http://<instance IP>/latest/meta-data`
    - can retrieve IAM role name from metadata but CANNOT retrieve IAM policy
- metadata = info abt ec2 instance
    - userdata = launch script of ec2 instance

### MFA with CLI
- to use MFA with CLI, must create temporary session
- must run `STS GetSession Token` API call
    - `aws sts get-session-token --serial-number <arn of mfa device> --token-code <code from token> --duration-seconds 3600`

![](https://i.imgur.com/RitiRX8.png)
- result given from cmd above




### Console and CLI
#### AWS CLI
![](https://codimd.s3.shivering-isles.com/demo/uploads/upload_089011404f91d818f109f8299a3e4f3b.png)


- IAM > users > security credentials
    - we will put access key into our aws config
        - need to create access key

![](https://codimd.s3.shivering-isles.com/demo/uploads/upload_6aaf3a9dbd1d67174351004107b5a2f7.png)

![](https://codimd.s3.shivering-isles.com/demo/uploads/upload_8da389ba9be3833876c83b50179ad0d0.png)
- input your access keys into your aws configure

#### IAM Roles on EC2
![](https://i.imgur.com/JOCfPSN.png)
- above `s3 ls` cmd will not work without the proper iam roles attached
- do not put credentials
    - attach IAM role instead

![](https://i.imgur.com/Y9uNi7n.png)
- can attach iam role to a lot of services

![](https://i.imgur.com/YhbqD13.png)
- after role created need attach to ec2 instance

![](https://i.imgur.com/U3lwcY8.png)
- now with role configed, `aws s3 ls` works
- NOTE
    - ec2 instances can only have 1 iam role at a time
        - role help ec2 perform api calls

#### Creating Own Policy
![](https://i.imgur.com/WCb2jCN.png)

![](https://i.imgur.com/j8ghQzk.png)
- inline policies
    - added on top of wtv policy you have alr chosen
    - these policies are not possible to be added to other roles
        - this policy is just for that specific role u configuring
    - not recommended, btr to manage policies globally for better management view

![](https://i.imgur.com/zDBFuTO.png)
- aws managed policy can be expanded to show policy summary or in json format

![](https://i.imgur.com/D6MWp1o.png)
- can use visual editor to create policy

![](https://i.imgur.com/QJSjFNZ.png)
- shows all avail api

![](https://i.imgur.com/ix9ZQAn.png)
- apply policy for all res or specific res through ARN

![](https://i.imgur.com/xt0BTwj.png)

![](https://i.imgur.com/ci0NHkS.png)
- alternatively can use aws policy generator too
    - but just use visual editor like same same eh

#### AWS Policy Simulator
- https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_testing-policies.html
    
    - tool to test your policies

![](https://i.imgur.com/lKJVxpL.png)
- select service, and perms included
    - https://policysim.aws.amazon.com/
    - can test against simulated actions
- alternatively can test directly using CLI (but why)

#### CLI Dry Runs
![](https://i.imgur.com/ZPkH84n.png)
- dry run cmd call to write instance example
- use sts decode msg to make sense of that long chunk of error msg

![](https://i.imgur.com/Rn2GHKL.jpg)
- need to give sts authorisation in iam policy too

![](https://i.imgur.com/VgICRn9.png)
- api `sts:decodeauthorizationmessage`

![](https://i.imgur.com/FtunC6O.png)
- final decoded result
    - paste in vscode, specify its json and format it to make it look nicer
        - open command palatte `F1` and choose `format document` OR `shift + alt + F` on windows


#### EC2 Metadata
![](https://i.imgur.com/EESk21C.png)
- get iam role name from `security-credentials`

![](https://i.imgur.com/tinj6TM.png)
- when iam role query API, it first queries the metadata url first to get impt info like access keys/tokens etc.
    - is shortlived credentials - temporary credentials

#### AWS CLI Profiles - Multiple AWS Accounts on CLI
- `aws configure --profile <profile name>`
    - input your access keys

![](https://i.imgur.com/aVZN3iv.png)

![](https://i.imgur.com/hqmA9Ft.png)

- `aws s3 ls --profile <profile name>`
    - to execute exe with specific profile (not default profile)

#### MFA with CLI
![](https://i.imgur.com/PuXQwG8.png)
- in IAM user console must assign mfa device
    - can use authy app or qr code to assign device

![](https://i.imgur.com/w3yoTxQ.png)
![](https://i.imgur.com/01e4XG0.png)
- get the arn and input into the sts command

![](https://i.imgur.com/4qijaFe.png)


![](https://i.imgur.com/D4j5yLu.png)
- result from command
    - this is temp credentials

![](https://i.imgur.com/yHdNeDY.png)
- create new profile mfa
    - open the credential file at `~/.aws/credentials`

![](https://i.imgur.com/YswhKue.png)

- add `aws_session_token` clause in credentials file from the result of sts command earlier
    - API calls will use this session token given


CLI/API Resources
---
### S3
- s3 cli
    - https://docs.aws.amazon.com/cli/latest/reference/s3/
- s3 API
    - https://docs.aws.amazon.com/AmazonS3/latest/API/API_control_GetBucket.html

### Generic CLI
- sts decode authorisation msg
    - https://docs.aws.amazon.com/cli/latest/reference/sts/decode-authorization-message.html


AWS SDK
---
- perform actions on aws from app code w/o using cli
    - use __software development kit (SDK)__
- official sdks are
    - java
    - .NET
    - node.js
    - php
    - python (named boto3/botocore)
    - go
    - ruby
    - c++
- have to use aws sdk when coding against aws services like dynamodb
    - aws cli uses python sdk (boto3)
- NOTE
    - exam expects you to know when u shld use sdk
    - if dont specify/config a default region, `us-east-1` will be chosen by default

### AWS Limits (Quotas)
- api rate limits - how many times u can call an alias API in a row
    - `DescribeInstances`
        - api for ec2 has a limit of 100 calls per second
    - `GetObject` on s3
        - limit of 5500 GET per sec per prefix
    - for intermittent errors when we go over the limit, implement exponential backoff
    - for consistent errors from heavy usage of app (consistently go over the limits), request an api throttling limit increase
        - must ask aws for this
- service quotas/limits - how many res we can run with something
    - running on-demand standard instances
        - 1152 vCPU
    - can request service limit inc by __opening a ticket__
    - can request service quota inc by using __service quota API__

#### Exponential Backoff
- for any aws service
- if get `ThrottlingException` (means going over limit) intermittently, use exponential backoff
- exponential backoff - is a retry mechanism included in sdk api calls
    - must implement yourself if using the API w/o any sdk as is or in specific cases
    - everytime u retry sth, you double the amt of time to wait before retrying
        - slow down the amt of load on system for it to go back to normal
- NOTE
    - very common exam qns when asked abt throttling exception

![](https://i.imgur.com/tPBg2Cn.png)
- 1st call, wait 1sec, 2nd call, wait 2 sec etc. 

CLI/SDK Credentials Provider Chain
---
- https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html#config-settings-and-precedence
### AWS CLI Credentials Provider Chain
- cli will look for credentials in this order
    - command line options
        - `--region`, `--output` and `--profile`
        - you input with the cmd
    - env variables
        - `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` AND `AWS_SESSION_TOKEN`
    - CLI credentials file
        - `~/.aws/credentials` on linux/mac
        - `C:\Users\user\.aws\credentials` on windows
    - CLI config file
        - `~/.aws/config` on linux/mac
        - `C:\Users\USERNAME\.aws\config` on windows
            - USERNAME is diff from user
    - container credentials
        - for ecs tasks
        - ecs covered later
    - instance profile credentials
        - for ec2 instance profiles
- NOTE
    - impt in scenario qns in exam

### AWS SDK Default Credentials Provider Chain
- java sdk (example) will look for credentials in this order
    - env variables
        - `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`
    - java system properties
        - `aws.accessKeyId` and `aws.secretKey`
    - default credential profiles file
        - Eg. at `~/.aws/credentials` shared by many SDKs
    - amazon ecs container credentials
        - for ecs containers
    - instance profile credentials
        - used on ec2 instances

### Credentials Scenario
![](https://i.imgur.com/12oabp3.png)
- iam user can do anything he wants on s3 buckets
- still giving priority to env vars from earlier
    - must unset env vars
    - will then leverage ec2 instance profile and perms

### AWS Credentials Best Practices
- NEVER store aws credentials in code
    - best prac to inherit from credentials chain
- if using working within aws, use IAM roles
    - ec2 instances roles for ec2 instances
    - ecs roles for ecs tasks
    - lambda roles for lambda funcs
- if working outside aws, use env vars/named profiles
    - in cli

Signing AWS API Requests
---
- when call aws http api, sign req using aws credentials so aws can identify you
    - access key and secret key
    - though some reqs to aws s3 dont need to be signed
    - if use sdk or cli, http requests signed for you
- shld sign aws http req using __signature v4 (SigV4)__
    - this is aws protocol
    - sigv4 signs reqs against your aws creds so you are authenticated

![](https://i.imgur.com/tLjhAxo.png)

### SigV4 Request Examples
- http header option

![](https://i.imgur.com/gA3f7Ns.png)

- query string option (Eg. s3 pre-signed urls)

![](https://i.imgur.com/NwS5ma4.png)


Quiz
---
![](https://i.imgur.com/YhDak9V.png)
- cannot attach iam role to on-premise server (local)
    - use env vars

![](https://i.imgur.com/9jbJthl.png)
- meta data credentials are temp


###### tags: `AWS Developer Associate` `Notes`