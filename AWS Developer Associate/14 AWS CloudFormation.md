

14 AWS CloudFormation
===




## Table of Contents

- [14 AWS CloudFormation](#14-aws-cloudformation)
  * [Table of Contents](#table-of-contents)
  * [AWS CloudFormation](#aws-cloudformation)
    + [Benefits](#benefits)
    + [How it Works](#how-it-works)
    + [Deploying CloudFormation Templates](#deploying-cloudformation-templates)
    + [Cloudformation Building Blocks](#cloudformation-building-blocks)
    + [CloudFormation Resources](#cloudformation-resources)
    + [CloudFormation Parameters](#cloudformation-parameters)
      - [Parameter Settings](#parameter-settings)
      - [Referencing a Param](#referencing-a-param)
      - [Pseudo Parameters](#pseudo-parameters)
    + [Mappings](#mappings)
      - [Mappings VS Parameters](#mappings-vs-parameters)
      - [Fn::FindInMap Accessing Mapping Values](#fn--findinmap-accessing-mapping-values)
    + [Outputs](#outputs)
      - [Example](#example)
      - [Cross Stack Reference](#cross-stack-reference)
    + [Conditions](#conditions)
      - [Defining a Condition](#defining-a-condition)
      - [Using a Condition](#using-a-condition)
    + [Must Know Intrinsic Functions](#must-know-intrinsic-functions)
      - [Fn::Ref](#fn--ref)
      - [Fn::GetAtt](#fn--getatt)
      - [Fn::FindInMap Accessing Map Values](#fn--findinmap-accessing-map-values)
      - [Fn::ImportValue](#fn--importvalue)
      - [Fn::Join](#fn--join)
      - [Fn::Sub](#fn--sub)
      - [Condition Functions](#condition-functions)
    + [CloudFormation Rollbacks](#cloudformation-rollbacks)
    + [ChangeSets](#changesets)
    + [Nested Stacks](#nested-stacks)
      - [Cross VS Nested Stacks](#cross-vs-nested-stacks)
    + [StackSets](#stacksets)
    + [Console](#console)
      - [Creating Stack](#creating-stack)
      - [Updating and Deleting Stack](#updating-and-deleting-stack)
      - [Rollbacks](#rollbacks)
  * [YAML Crash Course](#yaml-crash-course)



AWS CloudFormation
---
- currently we doing alot of manual work
    - tough to reproduce
        - in another region/acc
        - same region if everything deleted
    - what if all our infrastructure is code?
        - code can be deployed and create/update/delete our infrastructure
- cloudformation - declarative way of outlining your aws infrastructure for any res
    - most are supported
    - Eg. in cloudformation template you can state
        - security grp
        - 2 ec2 machines with this sec grp
        - 2 elastic IPs for the ec2
        - s3 bucket
        - lb in front of the ec2 machines
    - cloudformation creates those for you in right order with exact config specified

### Benefits
- infrastructure as code
    - no res manually created
        - excellent for control
    - code can be version controlled
        - Eg. using git
    - changes to infrastructure reviewed through code
- cost
    - ea res within stack tagged with identifier so can easily see how much a stack costs you
    - can estimate costs of res using cloudformation template
    - savings strategy
        - Eg. in dev, can automate deletion of templates at 5pm and recreated at 8am safely
- productivity
    - abi to destroy and recreate infra on cloud on fly
    - automated generation of diagram for templates
    - declarative programming
        - dont need figure out ordering and orchestration
- separation of concern - separate many stacks for many apps and layers
    - Eg.
        - vpc stacks
        - network stacks
        - app stacks
- dont re-invent the wheel
    - leverage existing template on internet
    - leverage documentation

### How it Works
- templates have to be uploaded in s3 and referenced in cloudformation
- to update template, cant edit prev ones
    - need to reupload new ver to aws
- stacks identified by name
- deleting stack deletes every single artifact created by cloudformation


### Deploying CloudFormation Templates
- manual way
    - editing templates in cloudformation designer
    - using console to input params
- automated way
    - editing templates in yaml file
    - using aws cli to deploy templates
    - recommended way when you want to fully automate your flow

### Cloudformation Building Blocks
- template components
    - resources
        - aws res declared in template
        - mandatory
        - Eg. ec2 machines, sec grps, lb etc.
    - parameters
        - dynamic inputs for templates
    - mappings
        - static vars for template
    - outputs
        - references to what has been created
        - eg. export some stuff from our template for other templates to refer
    - conditionals
        - list of conditions to perform res creation
            - eg. if statements
    - metadata
- template helpers
    - references
    - functions
- NOTE
    - mastering cloudformation will take a hella long time
        - this is just a high lvl overview
    - exam dont need to write cloudformation
        - exam expects you to understand how to read cloudformation

### CloudFormation Resources
- resources are core of your cloudformation template
    - mandatory
    - they represent the diff aws components that will be created and configured
- res are declared and can reference each other
- aws figures out creation, updates and deletes res for us
- there's over 224 types of res
- res types identifiers are of the form
    - `AWS::aws-product-name::data-type-name`
- guides
    - [List of Resources in CF](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html)
    - [EC2 instance CF example](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-instance.html)
- NOTE
    - you cannot create a dynamic amt of res
        - everything in cloudformation template has to be declared
        - cant perform code generation thr
    - not ALL aws services are supported
        - a few select niches are not thr yet
        - can work arnd using aws lambda custom res

### CloudFormation Parameters
- params are a way to provide inputs to aws cf template
- impt to know if
    - u want to reuse your templates across the company
    - some inputs cannot be determined ahead of time
- params are extremely powerful, controlled and can prevent errors in your templates thanks to __types__
    - dont have to reupload a template to change its content
- when to use?
    - is cf res config likely to change in the future?
        - if yes, make a param


![](https://i.imgur.com/NG2aPqJ.png)

#### Parameter Settings
![](https://i.imgur.com/mdQ6QfQ.png)
- type
    - string
    - number
    - comma delimited list
    - list\<type>
    - aws param
- description
- constraints
- constraintdescription
- min/max length for string
    - min/max val for numbers
- defaults
- allowed values
    - if want to restrict num of values a user can pick
- allowed pattern
    - verify inputs of user using regex
- no echo
    - if want to pass secrets

#### Referencing a Param
- the `Fn::Ref` func can be leveraged for reference params
    - shorthand in yaml is `!Ref`
    - func can also reference other elements within the template
- params can be used anywhr in template

![](https://i.imgur.com/rJaSSHJ.png)
![](https://i.imgur.com/TnLpC5y.png)

#### Pseudo Parameters
- can use pseudo params in cf template
    - can be used at any time 
    - enabled by default
- dont need to know abt this too much
    - maybe just 1st one
        - eg. accountId to try to construct complicated arn val in cf template

![](https://i.imgur.com/0MZ4r07.png)


### Mappings
- mappings are fixed vars within your cf template
    - very handy to differentiate between diff envs (dev vs prod), regions (aws regions), AMI types etc.
    - all values hardcoded within template
- example

![](https://i.imgur.com/raKe803.png)

#### Mappings VS Parameters
- mappings are great when know in advance all vals that can be taken and can be deduced from vars like
    - region
    - avail zone
    - aws acc
    - env (dev vs prod)
    - etc.
- allow safer control over template
- use params when vals rly user specific

#### Fn::FindInMap Accessing Mapping Values
- use `Fn::FindInMap` to return named val from specific key
- shorthand
    - `!FindInMap [ MapName, TopLevelKey, SecondLevelKey ]`

![](https://i.imgur.com/WNE58PD.png)


### Outputs
- outputs section declares optional outputs vals that we can import into other stacks
    - if you export them first
    - can also view outputs in aws console or cli
- very useful
    - Eg. if define network cloudformation and output vars like vpc id and subnet ids
- best way to perform some collab cross stack
    - as let expert handle own part of parts of vpc and subnet
    - app dev just reference these vals out of the box
- cant delete cf stack if its output are being referenced by another cf stack

#### Example
- create ssh sec grp as part of 1 template
- create output that references that sec grp

![](https://i.imgur.com/Dl4IVFD.png)
- export block is optional
    - sshsecuritygrp id we created is going to be exported with the name "SSHSecurityGroup"

    
#### Cross Stack Reference
- create 2nd template that leverages sec grp
    - use `Fn::ImportValue` func
- cant delete the underlying stack until all references deleted too

![](https://i.imgur.com/LISIRiI.png)


### Conditions
- conditions are used to control the creation of res or outputs based on a condition
- can be whatever you want them to be, but common ones are based on
    - env (dev/test/prod)
    - aws region
    - any param val
- ea condition can reference another condition, param val or mapping

#### Defining a Condition
![](https://i.imgur.com/8MRALcU.png)

- logical id is for you to choose
    - is how you name your condition
- intrinsic func (logical) can be any of following
    - `Fn::And`
    - `Fn::Equals`
    - `Fn::If`
    - `Fn::Not`
    - `Fn::Or`
- in the example
    - ref EnvType must be equal to string prod
        - apparently yaml dont need quotes for strings
    - `CreateProdResources` is the logical id

#### Using a Condition
- conditions can be applied to res/outputs etc.
    - mountpoint only gets created if condition is true
![](https://i.imgur.com/VgGvcqZ.png)


### Must Know Intrinsic Functions
- `Ref`
- `Fn::getAtt`
- `Fn::FindInMap`
- `Fn::ImportValue`
- `Fn::Join`
- `Fn::Sub`
- condition funcs
    - covered above

#### Fn::Ref
- fn::ref func can be leveraged to reference
    - params - returns val of param
    - resources - returns phy id of underlying res 
        - Eg. ec2 id
- shorthand in yaml is `!Ref`
![](https://i.imgur.com/pMLKEgV.png)


#### Fn::GetAtt
- attributes are attached to any res you create
- to know attr of your res, look at documentation
    - Eg. az of an ec2 machine

![](https://i.imgur.com/1TTmguN.png)

![](https://i.imgur.com/fmDTuYg.png)

#### Fn::FindInMap Accessing Map Values
- use `Fn::FindInMap` to return named val from specific key
- `!FindInMap [MapName, TopLevelKey, SecondLevelKey]`
    - [docs](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-findinmap.html)

![](https://i.imgur.com/kR4o1Ck.png)
![](https://i.imgur.com/0FVVBhn.png)


#### Fn::ImportValue 
- import values exported in other templates

![](https://i.imgur.com/Q6ReBuW.png)

#### Fn::Join
- join vals with delimiter

![](https://i.imgur.com/ZK1ZHBF.png)
![](https://i.imgur.com/dOooVjT.png)
- creates `a:b:c`

#### Fn::Sub
- `!Sub` is shorthand
- used to substitute vars from text
    - allows you to customise your templates
    - Eg. can combine !Sub with refrences or aws pseudo vars
- string must contain `${VariableName}` and will sub them

![](https://i.imgur.com/4lbORp4.png)

#### Condition Functions
- see above


### CloudFormation Rollbacks
- when stack creation fails
    - default - everything gets rollback (deleted)
        - look at the log
    - option to disable rollback and troubleshoot
        - have more insight to what's created
- when stack update fails
    - stack automatically rolls back to prev known working state
    - ability to see in log what happened and error msgs

### ChangeSets
- when update a stack, need to know what changes before it happens for greater confidence
- changesets wont say if update will be successful

![](https://i.imgur.com/3O8oguB.png)

### Nested Stacks
- nested stacks are stacks as part of other stacks
    - allow you to isolate repeated patterns/common components in separate stacks and call from other stacks
- example
    - load balancer config is reused
    - sec grp is reused
- nested stack considered best practice
- to update nested stack, always update parent (root stack)

#### Cross VS Nested Stacks
- cross stacks
    - helpful when stacks have diff lifecycles
    - use outputs export and  `Fn::ImportValue`
    - when need to pass export vals to many stacks
        - vpc id etc.
- nested stacks
    - helpful when components must be reused
        - eg. reuse how to properly config an app lb
    - nested stack only impt to higher lvl stack
        - its not shared

![](https://i.imgur.com/SP1OIsz.png)

### StackSets
- create, update or delete stacks across multiple accs and regions with single op
- admin acc to create stacksets
    - trusted acc to create, update, delete stack instances from stacksets
- when you update a stackset, __all__ associated stack instances are updated throughout all accs and regions

![](https://i.imgur.com/eU3tmNY.png)





### Console
- creating simple ec2 instance
    - create elastic ip
    - add 2 security grps
    - NOTE
        - make sure to look at code syntax
        - and file structure

![](https://i.imgur.com/icdUUQg.png)
- list of stacks

![](https://i.imgur.com/hWBZLIz.png)
- see details for ea stack

![](https://i.imgur.com/3uQiybg.png)
- see template of our stack
    - can open in designer

![](https://i.imgur.com/5skbkQO.png)
- designer view
    - view everything in template and how they relate to each other

![](https://i.imgur.com/Im4gow8.png)
- click on a node in the graph view to view more details of its template
    - can switch between json and yaml format in designer

#### Creating Stack
![](https://i.imgur.com/eT5NWbp.png)
- creating stack
    - can use template or create own

![](https://i.imgur.com/OX5ngLB.png)
- upload template or specify s3
    - file shld be yaml or json

![](https://i.imgur.com/M6bghYf.png)
- add params if needed
- modify extra stack configs if needed
    - but for now leave as default

![](https://i.imgur.com/aCcOZsS.png)
- can see all events being created when u create your 1st stack
    - first 2 is creation of ec2 machines

![](https://i.imgur.com/88lJS2w.png)
- created ec2 machines have tags automatically added to them

#### Updating and Deleting Stack
![](https://i.imgur.com/ciGohN1.png)
- example of cloudformation config in yaml

![](https://i.imgur.com/kdZMx30.png)
- new template to update in cloudformation

![](https://i.imgur.com/QyCBulC.png)
- can replace and upload new template

![](https://i.imgur.com/XBReFvf.png)
- need to enter val of param
    - need to define new section in template

![](https://i.imgur.com/456xuDr.png)

- view change set preview
    - cloudformation compares old and new template and shows diff
- eg. here elastic ip needs to be added, ec2 has to be modified etc.
    - ec2 replacement set to true - prev instance has to be terminated and new one created
- NOTE
    - cloudformation will automatically detect the order of creation of res in your template even if it's not in right order
        - Eg. security grp create before ec2 then create elastic ip

![](https://i.imgur.com/ZjAa7fj.png)
- deleting stack
    - will delete all res created in that stack too
        - hence deletion has to happen through cloudformation
    - also knows what to delete first

#### Rollbacks
![](https://i.imgur.com/20UJNCg.png)
- when sth goes wrong, cf auto rollbacks based on whether creation/update

![](https://i.imgur.com/p0EeRrN.png)
![](https://i.imgur.com/3x1Wtpb.png)
- change option in stack creation options when creating stack





YAML Crash Course
---
- yaml and json are langs u can use for cloudformation
    - json is horrible for cf
        - unreadable
        - unwriteable
    - yaml is great in many ways
- yaml has/can (compared to json)
    - key value pairs
    - nested objs
    - support arrays
        - denoted by `-` in front of the obj
    - multiline strings
        - denoted by `|`
    - can include comments

![](https://i.imgur.com/UBRFZgn.png)


###### tags: `AWS Developer Associate` `Notes`