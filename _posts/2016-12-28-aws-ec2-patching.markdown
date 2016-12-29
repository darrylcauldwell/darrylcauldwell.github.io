---
layout: post
title:  "How To Patch AWS Windows EC2 Instances"
date:   2016-12-28 11:20:56 +0100
tags:
    - Public Cloud
permalink: aws-ec2-patching/
---
Amazon EC2 Systems Manager is a collection of capabilities that helps you automate management tasks such as collecting system inventory, applying operating system (OS) patches, automating the creation of Amazon Machine Images (AMIs), and configuring operating systems (OSs) and applications at scale.

Amazon EC2 Systems Manager relies on the Amazon Simple Systems Management Service (SSM) agent being installed on the guests. The SSM agent is pre-installed on Windows Server 2016 instances or Windows Server 2003-2012 R2 instances created from AMI's published after November 2016, if you created your own or used earlier AMI you'll need to install. The SSM agent can be installed on other instances by following this [install guide](http://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/sysman-install-ssm-win.html).

Patch Management is always an operational pain point so its welcome that AWS offers a solution. You create groups of computers to patch by applying a tag 'Patch Group' and specifying a group name as the value. You create a group of patches by forming a patch baseline containing and excluding the patches you require. You then specify a maintenance window and specify task like patch this group of servers to this patch baseline.

How To
------
The guest agent requires permissions to connect to EC2 Systems Manager, we give these rights by create an EC2 Service role with the policy document 'AmazonEC2RoleforSSM' attached. We then provision the EC2 instance to be patched with this role attached. If you have instance already deployed you can add the policy document to a role which is added already,  or clone the instance to a new AMI attach role and power on.

Here I create a new role named 'EC2-Systems-Manager-Role' with the policy document 'AmazonEC2RoleforSSM' attached, attached it to a new test Windows 2016 EC2 instance.

    aws iam create-role --role-name EC2-Systems-Manager-Role --assume-role-policy-document '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Principal":{"Service":["ssm.amazonaws.com"]},"Action":["sts:AssumeRole"]}]}'

    aws iam create-instance-profile --instance-profile-name EC2-Systems-Manager-Profile

    aws iam add-role-to-instance-profile --instance-profile-name EC2-Systems-Manager-Profile --role-name EC2-Systems-Manager-Role

    aws ec2 create-security-group --group-name systemsManagerTest --description "EC2 Systems Manager Test Security Group"
    
    aws ec2 run-instances --image-id ami-771b4504 --count 1 --instance-type t2.micro --key-name MyEC2 --security-groups systemsManagerTest --iam-instance-profile Name=EC2-Systems-Manager-Profile

The 'Systems Manager Service' requires a Patch Group tag adding to the EC2 instances to patch,  so here we add a tag to the instance we just created.

    aws ec2 create-tags --resources <ec2-instance-id> --tags Key='Patch Group',Value='Test-Patch'

The 'Systems Manager Service - Maintenance Window' task requires rights on the EC2 instances to apply patches and also to task to the 'Systems Manager Service'.

    aws iam create-user --user-name ssmPatchUser

    aws iam create-policy --policy-name ssmPassRole --policy-document '{"Version": "2012-10-17","Statement": [{"Effect": "Allow","Action": ["iam:PassRole"],"Resource": "*"}]}'

    aws iam attach-user-policy --policy-arn arn:aws:iam::<your-acc-number>:policy/ssmPassRole  --user-name ssmPatchUser

    aws iam attach-user-policy --policy-arn arn:aws:iam::aws:policy/AmazonSSMFullAccess --user-name ssmPatchUser

We can then setup a patch baseline, this baseline auto approves all Critical, Important and Moderate patches of all classifications to be deployed seven days after they released.

    aws ssm create-patch-baseline --name "testBaseline" --approval-rules "PatchRules=[{PatchFilterGroup={PatchFilters=[{Key=MSRC_SEVERITY,Values=[Critical,Important,Moderate]},{Key=CLASSIFICATION,Values=[SecurityUpdates,Updates,UpdateRollups,CriticalUpdates]}]},ApproveAfterDays=7}]" --description "myBaseline"

We then take the baseline ID output and link the patch baseline with the patch group.

    aws ssm register-patch-baseline-for-patch-group --baseline-id <baseline-id> --patch-group "Test-Patch"

Once the baseline and patch groups are created and linked,  we create a SSM Maintenance Window with a task to deploy.  Here we create a schedule which starts every week day 6pm to midnight and stops scheduling tasks at 11pm. To format the cron expression is a little complex but its documented [here](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/sysman-maintenance-cron.html?icmpid=docs_ec2_console) in my example we have a window at 6pm for six hours Monday through Friday.

    aws ssm create-maintenance-window --name myFirstMaintenanceWindow --schedule "cron(0 00 18 ? * MON-FRI)" --duration 6 --cutoff 1 --allow-unassociated-targets

    aws ssm register-target-with-maintenance-window --window-id <maint-window-id> --targets "Key=tag:Patch Group,Values=Test-Patch" --owner-information "Test server" --resource-type "INSTANCE" 

    aws ssm register-task-with-maintenance-window --window-id <your-maintenance-window-id> --targets "Key=WindowTargetIds,Values=<your-target-group-id>" --task-arn "AWS-ApplyPatchBaseline" --service-role-arn "arn:aws:iam::<your-acc-id>:policy/ssmPassRole" --task-type "RUN_COMMAND" --max-concurrency 2 --max-errors 1 --priority 1 --task-parameters "{\"Operation\":{\"Values\":[\"Install\"]}}"

Another useful little tool for making operating EC2 instances that little bit easier.

