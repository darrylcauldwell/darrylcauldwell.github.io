+++
title = "Setup VPC Peering With Ansible"
date = "2016-11-14"
description = "Setup VPC Peering With Ansible"
tags = [
    "aws",
    "vpc",
    "ansible"
]
categories = [
    "automation"
]
thumbnail = "clarity-icons/code-144.svg"
+++
In this post I look at setting up [AWS VPC peering](http://docs.aws.amazon.com/AmazonVPC/latest/PeeringGuide/vpc-peering-overview.html) using Ansible. To do this we will start simple and add complexity to our configuration, we will start with peering within a single account and then move to script across accounts.

## Local Peering

I'll start by configuring two VPCs within the same account a single account, and configure peering between them. The Ansible host will sit within the same account which we create the new VPCs and peering. In order that Ansible can manage the AWS VPC services create an IAM Role named Ansible and assign it to the AdmistratorAccess policy. Once the IAM role is created we can then create a RHEL7 EC2 instance with this IAM role attached.

 The Ansible AWS modules manages AWS via the API by use of the [Python boto library](http://boto.cloudhackers.com/), presently the boto project is migrating from v2 to v3, the Ansible VPC module relies on both versions.  In order for boto to function correctly we also need locally installed AWSCLI. Once the RHEL instance is running connect and run the following commands to install Ansible and the boto and AWS CLI python library.

```bash
sudo yum install wget -y
wget http://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-8.noarch.rpm
sudo rpm -ivh epel-release-7-8.noarch.rpm
sudo yum install ansible git python python-devel python-pip -y
sudo pip install boto boto3 awscli
sudo yum update -y
```

What we'll do is use the [ec2_vpc module](http://docs.ansible.com/ansible/ec2_vpc_module.html) to create two new VPCs and capture the output of these as variables.  We can then use pass the output from VPC creation into the [ec2_vpc_peer module](http://docs.ansible.com/ansible/ec2_vpc_peer_module.html) to configure peering.

Once Ansible is installed you can clone example repository.

```bash
git clone https://github.com/darrylcauldwell/aws-ansible.git
cd aws-ansible
```

Inside the repository is an example playbook.

```bash
local-vpc-peering.yml
```

The contents of which are shown here.
 
 ```yml
---
- hosts: "localhost"
    connection: "local"

    tasks:
    - name: "First VPC"
    ec2_vpc:
        state: present
        cidr_block: 10.0.0.0/16
        resource_tags: { "Name":"First VPC" }
        subnets:
        - cidr: 10.0.1.0/24
            az: eu-west-1a
            resource_tags: {"Location":"SiteA"}
        - cidr: 10.0.2.0/24
            az: eu-west-1b
            resource_tags: {"Location":"SiteB"}
        - cidr: 10.0.3.0/24
            az: eu-west-1c
            resource_tags: {"Location":"SiteC"}
        region: eu-west-1
        wait: "yes" 
    register: first_vpc

    - debug: msg="{{ first_vpc }}" 

    - name: "Second VPC"
    ec2_vpc:
        state: present
        cidr_block: 10.1.0.0/16
        resource_tags: { "Name":"Second VPC" }
        subnets:
        - cidr: 10.1.1.0/24
            az: eu-west-1a
            resource_tags: {"Location":"SiteA"}
        - cidr: 10.1.2.0/24
            az: eu-west-1b
            resource_tags: {"Location":"SiteB"}
        - cidr: 10.1.3.0/24
            az: eu-west-1c
            resource_tags: {"Location":"SiteC"}
        region: eu-west-1
        wait: "yes" 
    register: second_vpc

    - debug: msg="{{ second_vpc }}" 

    - name: "Create local VPC peering connection"
    ec2_vpc_peer:
        region: eu-west-1
        vpc_id: "{{ first_vpc.vpc_id }}"
        peer_vpc_id: "{{ second_vpc.vpc_id }}"
        state: present
    register: vpc_peer

    - debug: msg="{{ vpc_peer }}"

    - name: "Accept local VPC peering connection"
    ec2_vpc_peer:
        region: eu-west-1
        peering_id: "{{ vpc_peer.peering_id }}"
        state: accept
    register: accept_peer

    - debug: msg="{{ accept_peer }}"
```

I've added debugging of output, if you run this you will see the two VPCs get created and then the peering configured between them.

## Cross Account VPC Peering Using Access Keys

As well as confguring peering within a single account, we can also use Ansible across AWS accounts. The steps we use are very similar but we begin to use [boto configuration profiles](http://boto.cloudhackers.com/en/latest/boto_config_tut.html) once we have a configuration profile for each account in place we can then target each task in the play to a different account. As we are using boto we'll authenticate using AWS Access Key and Secret rather than role based permissions applied to the EC2 instance, we cannot remove a role from a EC2 instance so terminate the old Ansible server and create a new one as above but without the IAM Role attached.

Within each account your managing create an IAM User called AnsibleAdministratorAccess and attach the AdmistratorAccess policy, add the Access Key ID and Secret Access Key to the boto2 and boto3 configuration files. I create a profile for each account named the account number.

```bash
sudo vi /etc/boto.cfg

[profile 843555617105]
aws_access_key_id = <your_access_key_here>
aws_secret_access_key = <your_secret_key_here>

[profile 664710917345]
aws_access_key_id = <your_access_key_here>
aws_secret_access_key = <your_secret_key_here>

mkdir ~/.aws/
cp /etc/boto.cfg ~/.aws/config
```

Once Ansible and boto are installed and configured you can clone example repository.

```
git clone https://github.com/darrylcauldwell/aws-ansible.git
cd aws-ansible
```

Inside the repository is an example playbook.

```
cross-account-vpc-peering.yml 
```

You'll notice this playbook looks very similar to the one for local peering. The key differences are the addition of the profile parameter for the ec2_vpc and ec2_vpc_peer tasks, and the addition of account number hosting VPCs.

```yml
---
- hosts: "localhost"
    connection: "local"

    tasks:
    - name: "First VPC"
    ec2_vpc:
        state: present
        cidr_block: 10.0.0.0/16
        profile: 843555617105
        resource_tags: { "Name":"First VPC" }
        subnets:
        - cidr: 10.0.1.0/24
            az: eu-west-1a
            resource_tags: {"Location":"SiteA"}
        - cidr: 10.0.2.0/24
            az: eu-west-1b
            resource_tags: {"Location":"SiteB"}
        - cidr: 10.0.3.0/24
            az: eu-west-1c
            resource_tags: {"Location":"SiteC"}
        region: eu-west-1
        wait: "yes" 
    register: first_vpc

    - debug: msg="{{ first_vpc }}" 

    - name: "Second VPC"
    ec2_vpc:
        state: present
        cidr_block: 10.1.0.0/16
        profile: 664710917345
        resource_tags: { "Name":"Second VPC" }
        subnets:
        - cidr: 10.1.1.0/24
            az: eu-west-1a
            resource_tags: {"Location":"SiteA"}
        - cidr: 10.1.2.0/24
            az: eu-west-1b
            resource_tags: {"Location":"SiteB"}
        - cidr: 10.1.3.0/24
            az: eu-west-1c
            resource_tags: {"Location":"SiteC"}
        region: eu-west-1
        wait: "yes" 
    register: second_vpc

    - debug: msg="{{ second_vpc }}" 

    - name: "Create local VPC peering connection"
    ec2_vpc_peer:
        profile: 843555617105
        region: eu-west-1
        vpc_id: "{{ first_vpc.vpc_id }}"
        peer_vpc_id: "{{ second_vpc.vpc_id }}"
        peer_owner_id: 664710917345
        state: present
    register: vpc_peer

    - debug: msg="{{ vpc_peer }}"

    - name: "Accept local VPC peering connection"
    ec2_vpc_peer:
        profile: 664710917345
        region: eu-west-1
        peering_id: "{{ vpc_peer.peering_id }}" 
        state: accept
    register: accept_peer

    - debug: msg="{{ accept_peer }}"
```

I've added debugging of output, if you run this 

```bash
ansible-playbook aws-ansible/cross-account-vpc-peering.yml
```

You will see the two VPCs in different accounts and get created and then the peering configured between them.

Cross Account VPC Peering Using IAM Assume Role Provider
--------------------------------------------------------
It is Amazon best practise is to [delegate access using roles instead of sharing credentials.](http://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#delegate-using-roles) You can define a role that specifies what permissions the IAM users in the other account are allowed, and from which AWS accounts the IAM users are allowed to assume the role. Up to now we've used IAM User and Access Keys to authenticate across multiple accounts, here we will look at configuring Ansible using AssumeRole.

Unfortunatly ec2_vpc does not yet support boto3 and this is required to use AssumeRole, ec2_vpc_peer however does support this. What this means though is we need to configure boto2 config file with access key in both accounts.

```bash
sudo vi /etc/boto.cfg

[profile 843555617105]
aws_access_key_id = <your_access_key_here>
aws_secret_access_key = <your_secret_key_here>

[profile 664710917345]
aws_access_key_id = <your_access_key_here>
aws_secret_access_key = <your_secret_key_here>
```

In order to configure boto3 to use AssumeRole we first, create an IAM User called AnsibleAdminUser in your first account. Then create an IAM Role your second account called 'AnsibleAdministrator' for role type select 'Role for Cross-Account Access \ Provide access between AWS accounts you own' then enter the Account ID of your first account and attach policy AdmistratorAccess. Once created view your new role in IAM and copy the Role ARN eg arn:aws:iam::664710917345:role/AnsibleAdministrator

Configure boto3 credentials

```bash
mkdir ~/.aws
sudo vi ~/.aws/credentials 

[664710917345-AnsibleAdminUser]
aws_access_key_id = <your_access_key_here>
aws_secret_access_key = <your_secret_key_here>

sudo vi ~/.aws/config

[profile 664710917345]
aws_access_key_id = <your_access_key_here>
aws_secret_access_key = <your_secret_key_here>

[profile 843555617105] 
role_arn=arn:aws:iam::843555617105:role/AnsibleAdministrator
source_profile=664710917345-AnsibleAdminUser
```

Once the credentials files are completed we can clone the example git repo and run the playbook.

```
git clone https://github.com/darrylcauldwell/aws-ansible.git
ansible-playbook aws-ansible/cross-account-vpc-peering.yml
```

We should see the same behaviour where two VPCs are created and VPC Peering is established between them.