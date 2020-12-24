

20 AWS Serverless Application Model (SAM)
===




## Table of Contents

- [20 AWS Serverless Application Model (SAM)](#20-aws-serverless-application-model--sam-)
  * [Table of Contents](#table-of-contents)
  * [AWS SAM](#aws-sam)
    + [What it Looks Like](#what-it-looks-like)
    + [SAM Deployment](#sam-deployment)
    + [SAM Policy Templates](#sam-policy-templates)
    + [SAM with CodeDeploy](#sam-with-codedeploy)
    + [Summary](#summary)
    + [Console](#console)
      - [Init Project](#init-project)
      - [Adding API Gateway](#adding-api-gateway)
      - [Adding DynamoDB](#adding-dynamodb)
      - [CloudFormation Designer and App Repository](#cloudformation-designer-and-app-repository)
      - [SAM with CodeDeploy](#sam-with-codedeploy-1)




AWS SAM
---
- serverless app model (sam)
- framework for developing and deploying serverless apps
    - all config in yaml
    - generate complex cloudformation from simple SAM YAML file
- supports anything from cloudformation
    - outputs, mappings, params, res etc.
- only 2 commands to deploy to aws
- sam can use codedeploy to deploy lambda funcs
    - sam can help you run lambda, api gateway, dynamodb locally
        - dont need to deploy lambda func to test it

### What it Looks Like
- transform header indicating SAM template
    - `Transform: 'AWS::Serverless-2016-10-31'`
- write code
    - `AWS::Serverless::Function`
        - lambda
    - `AWS::Serverless::Api`
        - api gateway
    - `AWS:Serverless::SimpleTable`
        - dynamodb
- package and deploy
    - `aws cloudformation package / sam package`
    - `aws cloudformation deploy / sam deploy`

### SAM Deployment
![](https://i.imgur.com/QOVDssI.png)
- aws cloudformation package
    - upload code zip file to s3
    - also transforms sam template into cloudformation template
    - generated templates will have a reference to s3
- aws cloudformation deploy
    - create and execute a change set
        - change set is figuring out how cloudformation shld take its existing state and move it to the next state based on modifications generated
    - cloudformation then applies it to our stack
        - stack may comprise of all services

### SAM Policy Templates
- list of templates to apply perms to your lambda funcs
    - full list [here](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-policy-templates.html)
- important examples
    - `S3ReadPolicy`
        - gives read only perms to objs in s3
    - `SQSPollerPolicy`
        - allows to poll on sqs queue
    - `DynamoDBCrudPolicy`
        - create read update delete

![](https://i.imgur.com/aHrfWlq.png)
- instead of creating iam role, attach policy in sam template instead

### SAM with CodeDeploy
- sam framework natively uses codedeploy to update lambda functions
- leverage traffic shifting feature using aliases
- can also define pre and post traffic hooks features to validate deployment
    - before traffic shift starts and after it ends
- easy & automated rollback using cloudwatch alarms

![](https://i.imgur.com/Qn0Lu3M.png)
- trigger deployment in codedeploy
    - runs pre traffic hook test using another lambda func
        - optional
    - do traffic shifting with alias
    - then monitor cloudwatch alarm
        - optional
        - ensure everything goes well during deployment
    - once deployment and traffic shifting done, run post-traffic hook lambda func
        - also optional
        - runs some tests on your alias
    - if everything goes well, v1 func of alias goes away
        - only left with v2

### Summary
- sam is built on cloudformation
- sam requires `Transform` and `Resources` section
- commands to know
    - `sam build`
        - fetch dependencies and create local deployment artifacts
    - `sam package`
        - package and upload to amazon s3
        - generate CF template
    - `sam deploy`
        - deploy to cloudformation
- sam policy templates for easy iam policy definition
- sam is integrated with codedeploy to do deploy to lambda aliases





### Console
- https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install-windows.html
    - follow this
    - windows just download a sam installer
    - use brew for mac and linux

#### Init Project
- use `sam init` to generate a sam template for your specific runtime
    - but can also create from scratch on your own

![](https://i.imgur.com/atXek0f.png)
- can go to sam examples github for dummy code
    - https://github.com/aws/serverless-application-model

![](https://i.imgur.com/jxYiTII.png)

- example template yaml file
    - `Transform` means sam template
    - resource list
        - 1st res is function
            - handler shld be `<file name>.<function name>`
        - codeuri shld point to dir with the code

![](https://i.imgur.com/jQruvMr.png)
- create cli commands to generate your transformed cloudformation template
    - first create s3 bucket
    - next upload code and do transformation from aws cloudformation package
        - can also use `sam package` since sam is just shorthand for aws cloudformation 

![](https://i.imgur.com/6YxSIcZ.png)
- generated template
    - codeuri now points to s3
![](https://i.imgur.com/nqGKlgq.png)
- running package will also output a sample deploy cli command
![](https://i.imgur.com/hBaZrC0.png)

![](https://i.imgur.com/IVO9jSz.png)
- failed as didnt add `CAPABILITY_IAM` into your cli cmd

![](https://i.imgur.com/jqbWB4y.png)


![](https://i.imgur.com/9Zd7nUK.png)

- once deployed, can go to cloudformation to check your stack out

#### Adding API Gateway
- look for api gateway in the examples github for ref

![](https://i.imgur.com/N9rf4bB.png)
- example lambda func for api gateway demo

![](https://i.imgur.com/dXZKo8b.png)
- add events segment in lambda func to create new api
    - func is invoked everytime a get req is called to /hello
- rerun your cloudformation commands to create the new api gateway


![](https://i.imgur.com/u1lhSGe.png)

- more resources created for your cloudformation
    - like api gateway and iam roles

#### Adding DynamoDB
- look for dynamodb example in github
    - actually prev api gateway example alr used dynamodb so we'll just use that

![](https://i.imgur.com/13uN9uS.png)
- example lambda func for adding of dynamodb
    - get region from os environment vars


![](https://i.imgur.com/paRhsB6.png)
- read doc for full list of properties u can pass into your simpletable segment
    - set provisioned throughput to save costs for this demo

![](https://i.imgur.com/JBKPjvr.png)
- add your simpletable section in the yaml file

![](https://i.imgur.com/wQhc5xn.png)
- can add env vars in your yaml template
    - table name refs the simpletable created
    - region refs a pseudo param `AWS::Region`

![](https://i.imgur.com/wAbC2eb.png)
- rmb to add iam policy to your func
    - this basically gives a dynamodbcrudpolicy with ref to the table created to your lambda func

![](https://i.imgur.com/sIY8ccW.png)
- sample function scans the dynamodb and returns it

#### CloudFormation Designer and App Repository
![](https://i.imgur.com/lINuNhR.png)
- can see details of what was created in cloudformation from viewing the stack
    - pic above shows the template code

![](https://i.imgur.com/ZFj1Zw2.png)
- can go actions > view designer

![](https://i.imgur.com/qdOeHCE.png)
- high lvl view of what was created

![](https://i.imgur.com/Iu4aoZl.png)
- also when creating your lambda func, u have option to use aws serverless app repo
    - is basically sam templates created by a lot of people
        - can search through


#### SAM with CodeDeploy
![](https://i.imgur.com/6TZeZ3a.png)
- using sam hello world python example

![](https://i.imgur.com/53FazTL.png)
- codeploy yaml to be integrated with your template yaml

![](https://i.imgur.com/MdSlswX.png)
- add `AutoPublishAlias` in your lambda func section in template yaml
    - create new alias "live"
- also deployment pref specific canary 10% 10 mins
    - 10% of traffic go to new ver for 10mins then shift 100%

![](https://i.imgur.com/A2PyL8k.png)
- can also use `sam deploy --guided` instead of package and deploy 
    - run `sam build` before

![](https://i.imgur.com/aqsHPHQ.png)
- live alias created

![](https://i.imgur.com/8ThfOvM.png)
- can view codedeploy traffic shift from codedeploy console







###### tags: `AWS Developer Associate` `Notes`