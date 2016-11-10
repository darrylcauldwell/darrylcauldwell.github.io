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

I included the example Ansible playbook in the github repository with the example files in which we pulled in previous step.

    ansible-playbook /home/ec2-user/aws-ansible/my-other-test-play.yml
