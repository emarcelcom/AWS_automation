# AWS automation
AWS EC2, S3 and some other (lambda, SNS, CloudWatch, CloudTrail etc) automation scripts (deployment, cleaning, maintenance). the idea is to: run a script and have already fully defined, working environment with all needed services, and appropriately configured policies and secure groups... On the other hands to have a tool, to fully clean AWS AccoundID env.

Because it is a cloud environemnt and some things takes time (deployment of instance, instance being actually up, creating snapshot or image) - all commands are checking if resource is already available.

It works both: in hybrid clouds, and in public one either (I tested it against [CloudGuru sandboxes](https://learn.acloud.guru/cloud-playground/cloud-sandboxes) and it works fine (despite some restriction which is described on CloudGuru site).

Scripts takes care for permissions, relations (to entities) and support error handling; the funniest is that from the scripts and CLI you may collect much more information then AWS console...

## prerequisites:
- some more-or-less obvious tools: ip, ifconfig, bc, and [grepcidr](http://www.pc-tools.net/unix/grepcidr/)
- already created VPC, IGW and subnet (usually defined by default, tested on newly created free tier account) - there is an assumptions they exist
- created IAM named profile - scripts won't run without providing profile name
- few initial mandatory VARIABLES (they are mostly covered, but they are personal):
  - ```AWS_DEFAULT_NAME``` - for prefixing security groups, policies, roles, instance profiles etc
  - ```AWS_COMMON_NAME``` - for prefixing something "bigger", and for more then one person/instance, like S3 buckets
  - ```AWS_NETWORK_TAG``` - in case there are some TAGs/Names of particular networks (like: private), if empty, then all taken
  - ```AWS_REPO_ADDRESS="https://kamoyl.github.io/AWS_automation/"``` (it is already pointing [here](https://kamoyl.github.io/AWS_automation/), because this repo is cloned on newly created instance and from that repo, and that instance there is a possibility to manage the AWS env also (policy is already properly created)
  - ```SLACK_CHANNEL_AWS```
  - ```SLACK_WEB_HOOK``` (for sending notifications about status of volumes and instances etc)
  - ```SLACK_TOKEN```
- parameters needed:
  - ```DEPLOY_USER``` (-u) - user which have permissions to do what we need to be done on created instance (optional parameter, default: "jenkins")
  - ```AWS_PROFILE``` (-p) - profile with which all commands (AWS CLI) will be run (mandatory parameter)
  - ```AWS_AMI_ID```  (-A) - AMI ID (when there is no local AMI(s)) (optional parameter, by default scripts are trying to figure it out from local AMI(s) which is the newest (created as the latest)
  - ```AWS_INSTANCE_TYPE``` (-I) - optional parameter (default is free tier: "t3.small")
  
## purpose

The main purpose of this repo is to - using only CLI, and as less as possible manual work - create automatically an environemnt for doing some tasks on the fly:
- create tight security group which only ads deployed IP and ssh protocol to access to EC2 instance(s) (only when instance is created)
- create common policy group which adds a group of networks with ssh protocol to access to EC2 instance(s) created by lambda
- create policy
- create role
- attach role to policy
- create an instance profile
- add that role to instance profile
- create an EC2 instance based on an image (own images, but it is very easy to change it to standard one) - choosing the newest one if more
  - each instance when started notifies itself on slack channel
  - each instance when shut down - notifies itself on slack channel (services definitions are in the repo already)
- assign created instance profile to that instance
- assign security policy to the instance 
- create S3 bucket(s) with secure access and encrypted by default
- create a KeyPair - checking if there is already an ssh key of deployer to import it in case
  - use it in case there is something to do remotely
  - remote script functions are prepared to be deployed in background logging output, which is presented with the rest of output
- very simmilar process is done when creating a lambda function which deployes EC2 instance based on delivered image ID (or pick up own newest) and runs appropriate code (of course with different policies, groups etc)
- present details in nice output, with all monthly and annualy costs so far
- CloudTrail (ToDo)
- events (busses, rules etc) - for appropriately deploy and run EC2 with a code, and clean environment afterword
- send appropriate notifications to slack channel - also nicely formatted
  - about costs
  - about own AMIs
  - about working/running instances
  - about volumes
  - S3 buckets (and their features: lifecycle, encryption, secure access etc)
- created instances (doesn't matter if by Lambda or directly) are deployed with 

And also:
- clean the whole environment - if needed:
  - all instance profiles, roles, policies
  - all security groups
  - all instances
  - all volumes
  - all keypairs
  - leaving only own AMIs, and S3 buckets
  
## AWS extra addons

- SSM agent manual installation:
  - **[SSM rpm](https://s3.eu-west-1.amazonaws.com/amazon-ssm-eu-west-1/latest/linux_amd64/amazon-ssm-agent.rpm)**
  - ```aws ssm start-session --region=eu-west-1 --target [instance_id]```
- packages and configuration management: *cloudformation* or *terraform*
- reconfiguration ssh port to 2222:
  - (SELinux) ```semanage port -a -t ssh_port_t -p tcp 2222```

### AWS related links
- [AWS cli installation](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)
- *[AWS info about policy versioning](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_version.html)*
- *[automatic backup process EC2 -> S3](https://aws.amazon.com/blogs/startups/how-to-back-up-workloads-with-amazon-s3-ebs/)*
- *[Running commands on your Linux instance at launch](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html)*
- *[How to use jq in oneliners AWS CLI](https://medium.com/circuitpeople/aws-cli-with-jq-and-bash-9d54e2eabaf1)*
- **[creating/attaching role to existing EC2 Instance](https://aws.amazon.com/blogs/security/new-attach-an-aws-iam-role-to-an-existing-amazon-ec2-instance-by-using-the-aws-cli/)**
- **[AWS Lambda - Launch EC2 Instances](https://medium.com/appgambit/aws-lambda-launch-ec2-instances-40d32d93fb58)**
- *[AWS Lambda - nice guide](https://stackify.com/aws-lambda-with-python-a-complete-getting-started-guide/)
- [s3fs - mounting s3 bucket as a filesystem under linux](https://github.com/s3fs-fuse/s3fs-fuse)
- [goofys - one of a method of mounting s3 as filesystem to linux](https://github.com/kahing/goofys)
- [sharing encrypted AMIs between accounts](https://aws.amazon.com/blogs/security/how-to-share-encrypted-amis-across-accounts-to-launch-encrypted-ec2-instances/)
### other
- *[slack emoji](https://www.webfx.com/tools/emoji-cheat-sheet)*
- *[markdown guide](https://www.markdownguide.org/basic-syntax)*

* __* all scripts are prepared now to be run outside AWS instance, and from it either - it means that cleaning is safe (current instance will be untouched, so as policied, secure groups, etc), profile is taken properly frmo inside EC2 instance and from autiside__

    [Kamil Czarnecki](kamoyl@outlook.com) - Wed, 9 Dec 2020 16:49:09 +0100
    
    * added lifecycle to S3 buckets
    
    * optimize a bit - run in parallel with shortened time about half
    

* __* updated gitignore__

    [Kamil Czarnecki](kamoyl@outlook.com) - Wed, 9 Dec 2020 10:52:52 +0100
    
    * added some - useful I think - scripts for EC2 instance to deploy - it is even
    easy to do it by lambda when creating an instance of by deploying a script when
    deploying instance on the fly
    
    * added: zram configuration (useful with small instances), and slack
    notification WHEN vm is up, and when it is going down - based on systemd
    

* __* adding changelog__

    [Kamil Czarnecki](kamoyl@outlook.com) - Wed, 9 Dec 2020 10:12:23 +0100
    
    

* __* small correction of static value when sending slack__

    [Kamil Czarnecki](kamoyl@outlook.com) - Wed, 9 Dec 2020 10:11:08 +0100
    
    

* __* added checking of S3 buckets features like secure access instead of apply it each time__

    [Kamil Czarnecki](kamoyl@outlook.com) - Tue, 8 Dec 2020 16:39:53 +0100
    
    * Lambda and Evens are now taken from one filke, with schedule, so it shhould
    be easier then it was
    

* __* lambda is migrated to bash function, which let&#39;s now deploy more of them, related to evens by name__

    [Kamil Czarnecki](kamoyl@outlook.com) - Mon, 7 Dec 2020 16:57:07 +0100
    
    

* __* Added changelog__

    [Kamil Czarnecki](kamoyl@outlook.com) - Fri, 4 Dec 2020 16:11:24 +0100
    
    

* __* Added changelog__

    [Kamil Czarnecki](kamoyl@outlook.com) - Fri, 4 Dec 2020 16:09:57 +0100
    
    

* __* eventbridge attached - it is deployed right after lambda, and with lambda related__

    [Kamil Czarnecki](kamoyl@outlook.com) - Fri, 4 Dec 2020 16:04:15 +0100
    
    * few correction to lambda deployment
    
    * changed picking up lambda scripts - now each one in lambda_functions with .py
    extension will be picked up, and based on it, appropriate event will be created
    

* __* added checking mandatory aws cli__

    [Kamil Czarnecki](kamoyl@outlook.com) - Fri, 4 Dec 2020 14:38:22 +0100
    
    * aws cli version is also checked
    
    * added an aws cli installation link to README
    

* __Add scripts - init of a branch__

    [Kamil Czarnecki](kamoyl@outlook.com) - Fri, 4 Dec 2020 12:40:51 +0100
    
    

* __Update README.md__

    [Kamil Czarnecki](kamoyl@outlook.com) - Wed, 2 Dec 2020 13:10:55 +0100
    
    

* __Update README.md__

    [Kamil Czarnecki](kamoyl@outlook.com) - Mon, 16 Nov 2020 12:14:11 +0100
    
    

* __Update README.md__

    [Kamil Czarnecki](kamoyl@outlook.com) - Mon, 16 Nov 2020 10:29:22 +0100
    
    

* __Update README.md__

    [Kamil Czarnecki](kamoyl@outlook.com) - Fri, 13 Nov 2020 10:43:50 +0100
    
    

* __Update README.md__

    [Kamil Czarnecki](kamoyl@outlook.com) - Thu, 12 Nov 2020 16:38:02 +0100
    
    

* __Update README.md__

    [Kamil Czarnecki](kamoyl@outlook.com) - Thu, 12 Nov 2020 16:36:41 +0100
    
    

* __Update README.md__

    [Kamil Czarnecki](kamoyl@outlook.com) - Thu, 12 Nov 2020 14:22:06 +0100
    
    

* __Update README.md__

    [Kamil Czarnecki](kamoyl@outlook.com) - Thu, 12 Nov 2020 14:21:31 +0100
    
    

* __Update README.md__

    [Kamil Czarnecki](kamoyl@outlook.com) - Thu, 12 Nov 2020 11:55:12 +0100
    
    

* __Update README.md__

    [Kamil Czarnecki](kamoyl@outlook.com) - Thu, 12 Nov 2020 11:50:45 +0100
    
    

* __Update README.md__

    [Kamil Czarnecki](kamoyl@outlook.com) - Thu, 12 Nov 2020 11:36:15 +0100
    
    

* __Update README.md__

    [Kamil Czarnecki](kamoyl@outlook.com) - Thu, 12 Nov 2020 10:13:56 +0100
    
    

* __Update README.md__

    [Kamil Czarnecki](kamoyl@outlook.com) - Thu, 12 Nov 2020 09:42:19 +0100
    
    

* __Update README.md__

    [Kamil Czarnecki](kamoyl@outlook.com) - Thu, 12 Nov 2020 09:26:48 +0100
    
    

* __Update README.md__

    [Kamil Czarnecki](kamoyl@outlook.com) - Wed, 11 Nov 2020 13:07:38 +0100
    
    

* __Update README.md__

    [Kamil Czarnecki](kamoyl@outlook.com) - Wed, 11 Nov 2020 09:32:03 +0100
    
    

* __Update README.md__

    [Kamil Czarnecki](kamoyl@outlook.com) - Wed, 11 Nov 2020 09:24:59 +0100
    
    

* __Update README.md__

    [Kamil Czarnecki](kamoyl@outlook.com) - Wed, 11 Nov 2020 09:24:42 +0100
    
    

* __Update README.md__

    [Kamil Czarnecki](kamoyl@outlook.com) - Tue, 10 Nov 2020 12:41:11 +0100
    
    

* __Update README.md__

    [Kamil Czarnecki](kamoyl@outlook.com) - Tue, 10 Nov 2020 12:35:31 +0100
    
    

* __Update README.md__

    [Kamil Czarnecki](kamoyl@outlook.com) - Fri, 6 Nov 2020 13:56:52 +0100
    
    

* __Update README.md__

    [Kamil Czarnecki](kamoyl@outlook.com) - Thu, 5 Nov 2020 15:47:50 +0100
    
    

* __Update README.md__

    [Kamil Czarnecki](kamoyl@outlook.com) - Thu, 5 Nov 2020 15:42:08 +0100
    
    

* __Update README.md__

    [Kamil Czarnecki](kamoyl@outlook.com) - Thu, 5 Nov 2020 12:33:53 +0100
    
    

* __Update README.md__

    [Kamil Czarnecki](kamoyl@outlook.com) - Thu, 5 Nov 2020 12:25:49 +0100
    
    

* __Update README.md__

    [Kamil Czarnecki](kamoyl@outlook.com) - Thu, 5 Nov 2020 12:18:15 +0100
    
    

* __Initial commit__

    [Kamil Czarnecki](kamoyl@outlook.com) - Thu, 5 Nov 2020 12:13:10 +0100
    
    
    

