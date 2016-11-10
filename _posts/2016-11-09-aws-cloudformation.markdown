---
layout: post
title:  "How To Use AWS CloudFormation With Ansible"
date:   2016-11-09 14:40:56 +0100
tags:
    - Ansible
    - Public Cloud
permalink: aws-cloudformation/
---
AWS CloudFormation gives developers and systems administrators an easy way to create and manage a collection of related AWS resources, provisioning and updating them in an orderly and predictable fashion. Ansible is a radically simple IT automation engine that automates cloud provisioning, configuration management, application deployment, intra-service orchestration, and many other IT needs. Ansible uses no agents and no additional custom security infrastructure, so it's easy to deploy, it uses a very simple language which allows you to describe your automation jobs in a way that approaches plain English.

Install Ansible For AWS Management
----------------------------------
In order that Ansible can manage AWS services create an IAM Role named Ansible and assign it to the AdmistratorAccess policy, then create a RHEL7 EC2 instance with this IAM role attached. The Ansible AWS modules manages AWS via the API by use of the [Python boto library](http://boto.cloudhackers.com/) and locally installed AWSCLI. Once the RHEL instance is running connect and run the following commands to install Ansible and the boto and AWS CLI python library.

    sudo yum install wget -y
    wget http://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-8.noarch.rpm
    sudo rpm -ivh epel-release-7-8.noarch.rpm
    sudo yum install ansible git python python-devel python-pip -y
    sudo pip install boto awscli
    sudo yum update -y

Deploy An EC2 Instance Using Ansible
------------------------------------
Once installed we can test Ansible can communicate correctly with AWS by creating a security group and EC2 instance. I've prepared a short playbook for my AWS account, in order to use yourself modify the five variables at the top of the playbook.

    git clone https://github.com/darrylcauldwell/aws-ansible.git
    ansible-playbook /home/ec2-user/aws-ansible/my-test-play.yml

Deploy An EC2 Instance Using CloudFormation
-------------------------------------------
Amazon provide various [CloudFormation Sample Templates](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-sample-templates.html). In this example I'll use the sample template EC2InstanceWithSecurityGroupSample: this creates an Amazon EC2 instance and a security group.

I included the template file in the github repository with the example files in which we pulled in previous step. The sample template takes upto three parameters,

* KeyName : Name of an existing EC2 KeyPair to enable SSH access to the instance
* InstanceType (Optional) : The size of EC2 instance if not specified defaults to t2.small
* SSHLocation (Optional) : The range of IP addresses which is allowed to connect to SSH defaults to 0.0.0.0/0

The example AWS provide requires a default region to be set on the AWS CLI, to do this use [AWS Configure](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html). We can then ask CloudFormations to deploy the template and include the parameters we want.

    aws configure
    aws cloudformation create-stack --stack-name startmyinstance --template-body file:///home/ec2-user/aws-ansible/EC2InstanceWithSecurityGroupSample.template --parameters  ParameterKey=KeyName,ParameterValue=MyEC2 ParameterKey=InstanceType,ParameterValue=t1.micro 

Deploy An EC2 Instance Using Ansible & CloudFormation
-----------------------------------------------------
Taking the scenrio one step further we'd like to drive the deployment of AWS infrastructure from Ansible therefore the parameters required by the CloudFormation templates we should use Ansible CloudFormation module and variables.

    ---
    - hosts: "localhost"
    connection: "local"
    gather_facts: false

    vars:
        KeyName: "MyEC2"
        InstanceType: "t2.micro"
        SSHLocation: "0.0.0.0/0"

    tasks:
        - name: Run my CloudFormation stack
        cloudformation:
            stack_name: "MyEC2Stack"
            region: "eu-west-1"
            state: "present"
            template: "EC2InstanceWithSecurityGroupSample.template"
            template_parameters:
            KeyName: "{{ KeyName }}"
            InstanceType: "{{ InstanceType }}"
            SSHLocation: "{{ SSHLocation }}"
            tags:
            tool: "ansible"
            env: "test"
        register: my_ec2_stack

        - debug: msg="{{ my_ec2_stack.stack_resources}}"
        - debug: msg="{{ my_ec2_stack.stack_outputs}}"

I included the example Ansible playbook above in the github repository with the example files in which we pulled in previous step.

    ansible-playbook /home/ec2-user/aws-ansible/my-other-test-play.yml

    PLAY [localhost] ***************************************************************
    TASK [Run my CloudFormation stack] *********************************************
    changed: [localhost]
    TASK [debug] *******************************************************************
    ok: [localhost] => {
        "msg": [
            {
                "last_updated_time": null, 
                "logical_resource_id": "EC2Instance", 
                "physical_resource_id": "i-040228bd8fcb5c81a", 
                "resource_type": "AWS::EC2::Instance", 
                "status": "CREATE_COMPLETE", 
                "status_reason": null
            }, 
            {
                "last_updated_time": null, 
                "logical_resource_id": "InstanceSecurityGroup", 
                "physical_resource_id": "MyEC2Stack-InstanceSecurityGroup-3YAAZV42DEPF", 
                "resource_type": "AWS::EC2::SecurityGroup", 
                "status": "CREATE_COMPLETE", 
                "status_reason": null
            }
        ]
    }
    TASK [debug] *******************************************************************
    ok: [localhost] => {
        "msg": {
            "AZ": "eu-west-1a", 
            "InstanceId": "i-040228bd8fcb5c81a", 
            "PublicDNS": "ec2-54-171-78-90.eu-west-1.compute.amazonaws.com", 
            "PublicIP": "54.171.78.90"
        }
    }
    PLAY RECAP *********************************************************************
    localhost                  : ok=3    changed=1    unreachable=0    failed=0   

You will see that if we run this it creates the stack with parameters we defined as variables.  Notice at the end of the first task we register the output as an object. For the example we output this to the screen by using a debug task, and it includes the EC2 instance details. You could just as easily use this to continue configuration of the EC2 instance guest operating system.

The Ansible playbook is idempotent so if you re-run the playbook whith state attribute as 'present' it checks it is in place and makes no changes. If you would like to remove the CloudFormation Stack then you can change the state attribute to 'absent' and when you re-run the playbook the CloudFormation Stack will be removed.
