

13 AWS Continuous Integration and Continuous Delivery  (CICD)
===




## Table of Contents

- [13 AWS Continuous Integration and Continuous Delivery  (CICD)](#13-aws-continuous-integration-and-continuous-delivery---cicd-)
  * [Table of Contents](#table-of-contents)
  * [AWS CICD](#aws-cicd)
    + [Introduction](#introduction)
    + [Continous Integration](#continous-integration)
    + [Continuous Delivery](#continuous-delivery)
    + [CICD Tech Stack](#cicd-tech-stack)
  * [AWS CodeCommit](#aws-codecommit)
    + [CodeCommit Security](#codecommit-security)
    + [CodeCommit VS Github](#codecommit-vs-github)
    + [CodeCommit Notifications](#codecommit-notifications)
    + [Console](#console)
      - [Pushing files via CLI](#pushing-files-via-cli)
  * [AWS CodePipeline](#aws-codepipeline)
    + [CodePipeline Artifacts](#codepipeline-artifacts)
    + [CodePipeline Troubleshooting](#codepipeline-troubleshooting)
    + [Console](#console-1)
  * [CodeBuild](#codebuild)
    + [Overview of what CodeBuild Does](#overview-of-what-codebuild-does)
    + [Supported Environments](#supported-environments)
    + [How Codebuild Works](#how-codebuild-works)
    + [CodeBuild BuildSpec.yml](#codebuild-buildspecyml)
    + [CodeBuild Local Build](#codebuild-local-build)
    + [CodeBuild in VPC](#codebuild-in-vpc)
    + [Console](#console-2)
      - [buildspec.yml](#buildspecyml)
      - [Adding VPC](#adding-vpc)
  * [CodeDeploy](#codedeploy)
    + [Working with CodeDeploy](#working-with-codedeploy)
    + [CodeDeploy Primary Components](#codedeploy-primary-components)
    + [AppSpec.yml](#appspecyml)
    + [CodeDeploy Deployment Config](#codedeploy-deployment-config)
      - [Half at a Time](#half-at-a-time)
      - [Blue Green Deployment](#blue-green-deployment)
    + [CodeDeploy to EC2](#codedeploy-to-ec2)
    + [CodeDeploy to ASG](#codedeploy-to-asg)
    + [CodeDeploy Rollbacks](#codedeploy-rollbacks)
    + [Console](#console-3)
  * [AWS CodeStar](#aws-codestar)
    + [Console](#console-4)
  * [Quiz](#quiz)




AWS CICD
---
### Introduction
- what we know now
    - create res in aws manually (fundamentals)
    - interact with aws programmatically (CLI)
    - deploy code to aws using elastic beanstalk
    - BUT these manual steps makes it very easy to make mistakes
- want to push our code into a repo and have it deployed onto aws
    - automatically
    - right way
    - make sure tested before deploying
    - with possibility to go into diff stages
        - Eg. dev, test, pre-prod, prod
    - with manual approval whr needed
- NEED LEARN AWS CICD
    - to automate deployment done so far whilst adding extra safety
- learn about
    - aws codecommit
        - store code
    - aws codepipeline
        - automating pipeline from code to elastic beanstalk
    - aws codebuild
        - building and testing code
    - aws codedeploy
        - deploying code to ec2 fleets (not beanstalk)

### Continous Integration
- process
    - devs push code to code repo often
        - Eg. github, codecommit, bitbucket etc.
    - testing/build server checks code as soon as it's pushed
        - Eg. codebuild, jenkins CI etc.
    - dev gets feedback abt tests and checks that passed/failed
- advantages/goal
    - find/fix bugs early
    - deliver faster as code is tested
    - deploy often
    - happier devs as they're unblocked in their workflow
        - Eg. dont have to wait for code to be tested, there's a build server to do it for them
    - increase productivity

![](https://i.imgur.com/FQ74vSd.png)

### Continuous Delivery
- ensure software can be released reliably whenever needed
- ensures deployments happen often and quick
    - shift away from 1 release every 3 months to 5 releases a day
- usually means automated deployment
    - codedeploy
    - jenkins CD
    - spinnaker
    - etc.

![](https://i.imgur.com/5mbDzkU.png)

### CICD Tech Stack
![](https://i.imgur.com/Gq3uQGF.png)
- overview of what services to use for each stage of deployment and alternatives to aws services


AWS CodeCommit
---
- __version control__ is ability to understand various changes that happens to code over time and the ability to rollback
    - enabled by using ver control system like Git
    - git repo can live on one's machine but usually lives on central online repo
- benefits
    - collab with other devs
    - ensure code is backed up somewhr
    - make sure it's fully viewable and auditable
- git repos can be expensive
    - industry includes
        - github - pub repos, paid priv repos
        - bitbucket
        - etc.
    - AND aws codecommit
        - priv git repos
        - no size limit on repos
            - scales seamlessly
        - fully managed, highly avail
        - code only in aws cloud acc
            - increased security and compliance
        - secure
            - encrypted
            - access control etc.
        - integrated with jenkins/codebuild/other CI tools

### CodeCommit Security
- interactions done using git
    - standard commands (git push/clone etc.)
- auth in git
    - ssh keys
        - aws users can config ssh keys in IAM console
    - https
        - done through aws cli auth helper or generating https credentials
    - mfa
        - enabled for extra safety
- authorisation in git
    - iam policies manage user/roles right to repos
- encryption
    - repo automatically encrypted at rest using KMS
    - encrypted in transit
        - can only use https or SSH (both secure)
- cross acc access
    - dont share ssh keys
    - dont share aws creds
    - use iam role in aws acc and use AWS STS (with AssumeRole API)

### CodeCommit VS Github
- similarities
    - are git repos
    - support code review/pull requests
    - can be integrated with aws codebuild
    - support https and ssh auth
- differences
    - security
        - github - github users
        - codecommit - aws iam users and roles
    - hosted
        - github - hosted by github
        - github enterprise - self hosted on your servers
        - codecommit - managed and hosted by aws
    - UI
        - github UI fully featured
        - codecommit UI minimal

### CodeCommit Notifications
- can trigger notifs in codecommit using AWS SNS (simple notif service) or aws lambda or aws cloudwatch event rules
- use cases for notif SNS/lambda notifs
    - deletion of branches
    - trigger for pushes that happens in master branch
    - notify external build system
    - trigger aws lambda func to perform codebase analysis
        - Eg. check for commited creds in code
- use cases for cloudwatch event rules
    - trigger for pull request updates
        - created/updated/deleted/commented
        - cloudwatch is more arnd pull reqs (code review)
            - or commenting on commits
    - commit comment events
    - cloudwatch event rules goes into SNS topic

### Console
![](https://i.imgur.com/0mqimrY.png)
- create repo in codecommit

![](https://i.imgur.com/k7GRa9j.png)
- if dont see ssh, means connected using root instead of iam user

![](https://i.imgur.com/OBbSCKT.png)
- upload file into repo
    - also write commit details

![](https://i.imgur.com/52BdAsP.png)
- pull req allow devs to marge changes from diff branch into master

![](https://i.imgur.com/TdjaKgB.png)
- can also view multiple commits and compare commits

![](https://i.imgur.com/6mPXZZy.png)

![](https://i.imgur.com/Q3ArieV.png)
- settings for repo

![](https://i.imgur.com/YQM7Sza.png)
- create notif rules for code repo in notif tab under settings

![](https://i.imgur.com/XezzU2V.png)
- full for all info or basic info

![](https://i.imgur.com/87CVAqp.png)

![](https://i.imgur.com/0UlI2qE.png)
- create target to send notif to

![](https://i.imgur.com/VRNvVqZ.png)
- eg. sns topic can trigger email alert when receives notifs from codecommit

![](https://i.imgur.com/2YAd6iV.png)

- can also create trigger in settings of repo
    - triggers are more on specific code events

![](https://i.imgur.com/VJTKxm0.png)
- can trigger to sns or lambda

#### Pushing files via CLI
![](https://i.imgur.com/wS3NRwe.png)
- need go iam roles and see ssh keys for codecommit or git
    - can just upload ssh pub key in thr and git integration will work
- over here has ssh key support or username and password support (https git credentials)

![](https://i.imgur.com/7BHOCuL.png)
![](https://i.imgur.com/ELS0Ktm.png)

- basically use standard git cli commands
    - `git status add commit push fetch pull clone`


AWS CodePipeline
---
- cont delivery
- visual workflow
- source - github, codecommit, amazon s3
    - build - codebuild, jenkins etc.
    - load testing - 3rd party tools
    - deploy - aws codedeploy, beanstalk, cloudformation, ecs etc.
- made of __stages__
    - ea stage can have sequential actions and/or parallel actions
        - can customise pipeline
            - Eg. load testing all at same time or 2 types of build one after another
    - stage examples
        - build > test > deploy > load test etc.
    - manual approval can be defined at any stage
        - except on source pull


### CodePipeline Artifacts
- ea pipeline stage can create artifacts
    - artifacts are files passed and stored in amazon s3 and passed onto next stage

![](https://i.imgur.com/jo3eyOb.png)
- after trigger, source code in codecommit all sent to s3 (AKA source output artifacts)
    - the artifacts slowly come back up through s3 and into the build stage
        - build stage get code from source and build it
        - after building, may also generate more artifacts
        - Eg. generated binaries, zip files
    - output of build stage then piped to deploy stage

### CodePipeline Troubleshooting
- codepipeline states that changes happen in aws __cloudwatch events__ which in return create sns notifs
    - any state change in pipeline generated a cloudwatch event
    - Eg. can create events for failed pipelines/cancelled stages
- if codepipeline fails a stage, your pipeline stops and u can get info in the console
    - can also send sns, cloud watch event and go in console to understand what happened
- aws cloudtrial can be used to audit aws API calls
    - cloudtrial is used to audit any api calls done across your amazon infrastructure
- if pipeline cant perform an action, make sure IAM service role attached have enough perms
    - check iam policy

### Console
![](https://i.imgur.com/3lOzJnA.png)
- pipeline is gonna be talking to a lot of things (s3, codecommit, beanstalk etc.) hence need service role with enough perms 

![](https://i.imgur.com/vvTQNp7.png)
- specify location for artifacts


![](https://i.imgur.com/06FpgIS.png)
- choose source provider - whr you store your input artifacts
    - for now choose codecommit
        - can also integrate with ecr, s3 or github

![](https://i.imgur.com/1EpOH8N.png)
- choose repo name and branch

![](https://i.imgur.com/kTye7Uf.png)
- 2 ways for codecommit to be tracked
    - cloudwatch events
        - cloudwatch event rule created automatically to trigger pipeline on event
    - else can use codepipeline to check periodically for changes
        - but wont be as fast and not recommended

![](https://i.imgur.com/CaL75A4.png)
- leaving blank for now but ideally use codebuild

![](https://i.imgur.com/Dw1xjCD.png)
![](https://i.imgur.com/irqbOTA.png)
- if choose beanstalk must also provide app and env names

![](https://i.imgur.com/YG5vbCN.png)

![](https://i.imgur.com/j7FrJwy.png)
- new codepipeline ver created in beanstalk

![](https://i.imgur.com/2WuROdE.png)
- can edit pipeline and add stages

![](https://i.imgur.com/6eunbgX.png)
- in stage, can add action grp
    - choose action provider

![](https://i.imgur.com/lxn3a6k.png)
- theres many action providers other than manual approval
    - like jenkins, github etc.

![](https://i.imgur.com/NffVMse.png)

- ea stage can have multiple action grps
    - either sequential or parallel

![](https://i.imgur.com/0dTS6fY.png)
- eg. another action grp to deploy to beanstalk

![](https://i.imgur.com/NymKc5q.png)
- for manual approval, need to click review and add a review comment if needed

![](https://i.imgur.com/0tsztO0.png)


CodeBuild
---
- fully managed build service
- alternative to other build tools like jenkins
- continuous scaling
    - no servers needed to manage provision
    - no build queue
- pay for usage
    - time taken to complete builds
    - compared to jenkins, if do one build per day (Eg. only take a min) for jenkins you would be wasting money
- leverages docker under the hood for reproducible builds
    - possibility to extend capabilities leveraging own docker images
        - eg. instead of using docker images provided by amazon for codebuild, can create own base docker imgs for codebuild
        - enables for more customization
- secure
    - integration with
        - kms for encryption of build artifacts (build outputs)
        - iam for build perms
        - vpc for network security
        - cloudtrial for api calls logging

### Overview of what CodeBuild Does
- src code from github/codecommit/codepipeline/s3 etc.
- build instructions can be defined in code
    - `buildspec.yml`
- output logs to amazon s3 & aws cloudwatch logs
    - can config
    - Eg. when codebuild finishes, whole container that ran build goes away and only things left to troubleshoot is logs in s3/cloudwatch
- metrics to monitor codebuild stats
- use cloudwatch events to detect failed builds and trigger notifs
- use cloudwatch alarms to notify if you need thresholds for failures
- cloudwatch events/aws lambda as glue
- trigger sns notifs
- ability to reproduce codebuild locally to troubleshoot in case of errors
- builds can be defined within codepipeline/codebuild itself

### Supported Environments
- java
- ruby
- python
- go
- nodejs
- android
- .net core
- php
- docker
    - extend any env you like

### How Codebuild Works
![](https://i.imgur.com/kvuFjxz.png)
- src code from codecommit
    - need buildspec.yml file which will be the root of your code
        - what will the container do to build our code
    - also build docker img either from aws or custom made
- codebuild container starts when codebuild triggered 
    - uses docker img we set before as src img for run
    - optional aws s3 cache bucket 
        - can cache dependencies/artifacts as you build
        - to increase performance
- when codebuild passes, it sends output to s3 artifacts bucket while logs go to cloudwatch or s3
    - cache file within codebuild will be back into s3 within the feedback loop


### CodeBuild BuildSpec.yml
- `buildspec.yml` file must be at root of code
- define env vars
    - plaintext vars
    - secure secrets
        - use SSM param store to pull in secrets
        - dont store in code
- phases - specify commands to run
    - install
        - install dependencies you may need for build
    - prebuild
        - final commands to execute before build
    - build
        - actual build cmds
    - post build
        - finishing touches
        - Eg. create zip output
- artifacts
    - what to upload to s3
    - encrypted with kms
- cache
    - files to cache to s3 for future build speedup
        - usually dependencies
        - performance improvement

### CodeBuild Local Build
- in case you need deep troubleshooting beyond logs, can run codebuild locally on desktop after installing docker
    - can leverage [codebuild agent](https://docs.aws.amazon.com/codebuild/latest/userguide/use-codebuild-agent.html)

### CodeBuild in VPC
- by default, codebuild containers launched outside vpc
    - hence cant access res in your vpc
- can specify vpc configuration
    - vpc id
    - subnet ids
    - security grps ids
- build can then access res in your vpc
    - Eg. rds, elasticache, ec2, alb etc.
- use cases
    - integration tests
    - data query
    - internal load balancers

![](https://i.imgur.com/E3uJOp8.png)



### Console
![](https://i.imgur.com/LeXD1uo.png)
- when creating build, specify source

![](https://i.imgur.com/eR1d6u1.png)

- choose docker img
    - also config your docker img
        - os
        - runtime
        - image version
        - etc.

![](https://i.imgur.com/im9Q3Zr.png)
- iam service role

![](https://i.imgur.com/1U6xNJG.png)
![](https://i.imgur.com/4AxC0Ju.png)

- extra configs
    - timeout - how long build runs of nothing's happening
    - queued timeout - how long for build to be queued
    - certs
    - vpc
        - if codebuild needs to access res in vpc
    - select how big docker compute will be
    - set env vars

![](https://i.imgur.com/Cej43IS.png)
- buildspec file
    - by default looks for file in src code

![](https://i.imgur.com/Z4cHJg9.png)
![](https://i.imgur.com/ykXph65.png)

- what artifacts to use Eg. s3

![](https://i.imgur.com/7Pf4ZIV.png)
- extra artifacts config - caching

![](https://i.imgur.com/cm4Qj1o.png)
- logs by default pushed to cloudwatch

![](https://i.imgur.com/qPHl8k4.png)
- able to see build details
    - logs
    - details
    - env vars
    - details
- eg. here is due to missing .yml file

#### buildspec.yml
![](https://i.imgur.com/lvgjjGX.png)
- can use console editor to create the .yml file
    - make sure to name it buildspec.yml
    - also have to include some personal details when commiting like
        - author name
        - email
        - commit msg

![](https://i.imgur.com/Zw4TAJR.png)
- use indents to separate sections
- 4 phases
    - install
        - specify runtime ver for codebuild env
            - nodejs ver 10
        - commands to run when installing
    - pre_build (before build happens)
    - build
        - `grep -Fq ... index.html`
            - look to see if congratulations string is present in the html file
            - if not in there, the command will crash and fail
    - post_build

![](https://i.imgur.com/QWjSeJH.png)
- this shows all phases of build

![](https://i.imgur.com/dnGwjcY.png)
- can click view all logs in the console to be brought to cloudwatch logs

![](https://i.imgur.com/6xQ7Qku.png)
- can add new build which tests for congrats into the pipeline

![](https://i.imgur.com/2ZTbzJJ.png)
- error msg when the grep cmd fails

#### Adding VPC
![](https://i.imgur.com/UAuTOq5.png)

- in edit env, go to extra configs > vpc

![](https://i.imgur.com/B6WuzfH.png)
- vpc with this id might not have internet conn as provided subnets is pub
    - so u cant have internet conn with codebuild if u launch in the public subnets
    - recommendation to launch in priv subnet and add NAT gateway
        - its fine if dont need internet for your build though


CodeDeploy
---
- want to deploy app automatically to many ec2 instances
    - these instances not managed by elastic beanstalk but rather by us
- several ways to handle deployments using open src tools
    - Eg. ansible, terraform, chef, puppet etc.
    - though can use managed service aws codedeploy instead of the open src ones

### Working with CodeDeploy
- ea ec2 machine or on-premise machine must be running codedeploy agent
- agent is continuously polling aws codedeploy for work to do
- codedeploy sends `appspec.yml` file
    - or app pulled from github or s3
- ec2 runs deployment instructions
- codedeploy agent will report success/failure of deployment on instance

![](https://i.imgur.com/3FX8qzf.png)

- NOTE
    - ec2 instances grped by deployment grp
        - dev/test/prod
    - lots of flexibility to define any kinds of deployments
    - codedeploy can be chained into codepipeline and use artifacts from thr
    - codedeploy can reuse existing setup tools, works with any app and has autoscaling integration
        - codedeploy is basically a bit more powerful than beanstalk
            - beanstalk forces u to use a set of diff apps and platforms whereas codedeploy can be anything u want
            - though will be more complex
    - blue/green only works with ec2 instances
        - not on premise
    - support for aws lambda deployments
        - wont be asked in exam
    - codedeploy does not provision res
        - it assumes your ec2 instances alr exists

### CodeDeploy Primary Components
- app
    - unique name
- compute platform
    - ec2/on-premise or lambda
- deployment config - deployment rules for sucess/failure
    - ec2/on-premise
        - can specify min num of healthy instances for deployment
        - Eg. 95% of deployment to succeed
    - aws lambda
        - specify how traffic is routed to your updated lambda func vers
- deployment grp
    - grp of tagged instances
    - allows to deploy gradually
- deployment type
    - in-place deployment or blue/green deployment
- iam instance profile
    - need to give ec2 the perms to pull from s3/github
- app revision
    - app code + appspec.yml
- service role
    - role for codedeploy to perform what it needs
- target revision
    - target deployment app ver

### AppSpec.yml
- file section
    - how to src and copy from s3/github to filesystem
- hooks - set of instructions to do to deploy new ver
    - hooks can have timeouts
        - Eg. shldnt run for >2mins else fails
    - order of timeouts
        - ApplicationStop
            - stop current app to deploy new app
        - DownloadBundle
            - how to download new app
        - BeforeInstall
            - do prep before installing app ver
        - AfterInstall
            - cleanup after install or launch server
        - ApplicationStart
            - how to start app
        - ValidateService
            - very impt
            - once app started, how to ensure its rly working?
            - AKA health check to ensure app correctly deployed

![](https://i.imgur.com/AGm32b2.png)

### CodeDeploy Deployment Config
- configs
    - one at a time
        - one instance at a time
        - if one instance fails, deployment stops
    - half at time
    - all at once
        - quick but no healthy host
        - downtime
        - good for dev
    - custom
        - Eg. min healthy host = 75%
- failures
    - instances stay in failed state
    - new deployments first deployed to failed state instances
        - guarantee that wont bring down your whole app
    - to rollback
        - redeploy old deployment or enable automated rollback for failures
- deployment targets
    - set of ec2 instances with tags
    - directly to asg
    - mix of asg/tags that can build deployment segments
    - customisation in scripts with `DEPLOYMENT_GROUP_NAME` env vars
        - Eg. if in dev do this, if in prod do this
        - big control

#### Half at a Time
![](https://i.imgur.com/V9UsHDZ.png)

#### Blue Green Deployment
![](https://i.imgur.com/Lbbd0QH.png)

### CodeDeploy to EC2
- define how to deploy app using appspec.yml + deployment strat
- will do in-place update to your fleet of ec2 instances
- can use hooks to verify deployment after ea deployment phase

![](https://i.imgur.com/urXBulO.png)

### CodeDeploy to ASG
- in place updates
    - updates current existing ec2 instances
    - instances newly created by asg also get automated deployments
- blue/green deployment
    - new auto-scaling grp created 
        - settings copied
    - choose how long to keep old instances
    - must be using elb

![](https://i.imgur.com/N5Qm8Jr.png)

### CodeDeploy Rollbacks
- can specify automated rollback options
    - rollback when deployment fails
    - rollback when alarm thresholds met
    - disable rollbacks
        - dont perform rollbacks for deployment
- if rollback happens, codedeploy redeploys last known good revision as new deployment
    - new deployment has new ver id




### Console
![](https://i.imgur.com/Hjb3cE8.png)
- firstly need to create 2 roles in iam for codedeploy
    - service role
    - ec2 service role
        - for ec2 instance running the codedeploy agent
        - ec2 instance must be able to pull data/code directly from amazon s3
            - hence need s3 access

![](https://i.imgur.com/XV6XWwM.png)
![](https://i.imgur.com/XRWXPdU.png)

![](https://i.imgur.com/cQkBXyb.png)

![](https://i.imgur.com/rNI7ZaH.png)
- need to manual create ec2 instance for codedeploy
    - they dont do it for you automatically
    - attach your ec2 instance role to it

![](https://i.imgur.com/QbxUiOq.png)

- ssh into your ec2 and install codedeploy agent
- install ruby and latest install of aws codedeploy

![](https://i.imgur.com/BAfL77J.png)
- need to create deployment grp
    - are sets of ec2 instances you deploying app to

![](https://i.imgur.com/oPqB6il.png)

- need to first tag ec2 instances in order to add to deployment grp

![](https://i.imgur.com/6CMHHi3.png)
- attach role created earlier
- 2 deployment types

![](https://i.imgur.com/hzHr4Tr.png)
![](https://i.imgur.com/XI2RxCT.png)
- select your tag created earlier
    - all instances that have tag will be part of the deployment grp

![](https://i.imgur.com/aYGqKgS.png)
- deployment settings

![](https://i.imgur.com/JRc9QUx.png)
- can even create own deployment settings

![](https://i.imgur.com/2l2mMK6.png)

![](https://i.imgur.com/yyHQcoQ.png)
- create deployment and specify deployment grp

![](https://i.imgur.com/LBb3EKs.png)
- specify code from s3 or github
    - need to create s3 bucket for your code

![](https://i.imgur.com/KrrGF0E.png)
- sample appspec.yml file for codedeploy
    - has 3 bash scripts that have to be launched

![](https://i.imgur.com/NUZ4C4g.png)
- run hooks based on these series of events


AWS CodeStar
---
- codestar is integrated solution that regroups:
    - github
    - codecommit
    - codebuild
    - codedeploy
    - cloudformation
    - codepipeline
    - cloudwatch
- helps quickly create CICD-ready projects for ec2, lambda, beanstalk
- supported langs
    - c#
    - go
    - html5
    - java
    - nodejs
    - php
    - python
    - ruby
- issue tracking integration with 
    - jira/github issues
- ability to integrate with cloud9 to obtain web IDE env to edit code
    - though not avail on all regions
- 1 dashboard to view all components
    - just a wrapper arnd everything
- free service, pay only for underlying usage of other services
- limited customisation
    - cannot edit every single service
    - meant to be simple to get your started quickly

### Console
![](https://i.imgur.com/jETKbgX.png)
- after creating service role

![](https://i.imgur.com/tb7mYpa.png)
![](https://i.imgur.com/PK3zl4A.png)
- everything handled by aws pipeline

![](https://i.imgur.com/rKgz9Ya.png)
![](https://i.imgur.com/7EtJa5r.png)
- if in us east virginia, will see cloud9 too

![](https://i.imgur.com/3kZabxa.png)
![](https://i.imgur.com/Ox5S2MO.png)
![](https://i.imgur.com/RZ0FYxe.png)
- clicking ea tab in codestar will redirect to the underlying services

![](https://i.imgur.com/sgL7N9M.png)
- displays info on other underlying services


Quiz
---
![](https://i.imgur.com/5sabN3N.png)
![](https://i.imgur.com/0A6fOiN.png)
![](https://i.imgur.com/4ltbgz1.png)
![](https://i.imgur.com/Mt8keao.png)
![](https://i.imgur.com/dBe4KR4.png)



###### tags: `AWS Developer Associate` `Notes`