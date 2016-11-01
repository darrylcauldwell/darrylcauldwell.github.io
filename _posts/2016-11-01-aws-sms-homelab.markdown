---
layout: post
title:  "AWS Server Migration Services (SMS)"
date:   2016-11-01 15:40:56 +0100
tags:
    - Public Cloud
permalink: aws-sms-homelab/
---
The [AWS Server Migration Service](https://aws.amazon.com/server-migration-service/) simplifies and streamlines the process of migrating existing virtualized applications to Amazon EC2. AWS SMS allows you to automate, schedule, and track incremental replications of live server volumes, making it easier for you to coordinate large-scale server migrations. Presently this only supports migrating from VMware with support for other hypervisors and physical servers is coming soon.

The process is pretty straight forwards,  first task is to download the AWS Server Migration Connector from [S3](https://s3.amazonaws.com/sms-connector/AWS-SMS-Connector.ova).

Then deploy the OVA as normal  
<center><img src="/images/aws-sms-ova-deploy.jpeg" width="50%"></center>

<center><img src="/images/aws-sms-ova-deploy-final.jpeg" width="50%"></center>

The Connector has a web UI, once deployed connect open browser to https://dhcp-addr
<center><img src="/images/aws-sma-cfg-wiz.jpeg" width="50%"></center>

The SMS Connector needst to connect to your AWS account and therefore we need to create a user with the "ServerMigrationConnector" role attached.
<center><img src="/images/aws-sms-account.jpeg" width="50%"></center>

The setup wizard guides you through the following steps,

* Step 1: License Agreement
* Step 2: Create a Password
* Step 3: Network Info
* Step 4: Log Uploads and Upgrades
* Step 5: Server Migration Service

Once configuration is complete the connection to AWS and vCenter should show good in the connector configuration.
<center><img src="/images/aws-sma-cfg-complete.jpeg" width="50%"></center>

If we then connect to AWS we can see the connector.
<center><img src="/images/aws-sma-cfg-complete-console.jpeg" width="50%"></center>

The first task to complete is importing the server catalog from the Connector and inspect the vCenter Server inventory.
<center><img src="/images/aws-sma-import.jpeg" width="50%"></center>

We need a custom role creating to import a virtual machine.  To do this create a local file named trust-policy.json with the following content:  

```  
{  
    "Version": "2012-10-17",  
    "Statement": [  
        {  
            "Sid": "",  
            "Effect": "Allow",  
            "Principal": {  
                "Service": "sms.amazonaws.com"  
            },  
            "Action": "sts:AssumeRole",  
            "Condition": {  
                "StringEquals": {  
                    "sts:ExternalId": "sms"  
                }  
            }  
        }  
    ]  
}  
```   

Create a local file named role-policy.json with the following content:  

```  
{  
    "Version": "2012-10-17",  
    "Statement": [  
        {  
            "Effect": "Allow",  
            "Action": [  
                "ec2:ModifySnapshotAttribute",  
                "ec2:CopySnapshot",  
                "ec2:CopyImage",  
                "ec2:DeleteSnapshot",  
                "ec2:DescribeImages",  
                "ec2:DescribeSnapshots"  
            ],  
            "Resource": "*"  
        }  
    ]  
}  
```  

At a command prompt, go to the directory where you stored the two JSON policy files, and run the following commands to create the SMS service role:  

    aws iam create-role --role-name sms --assume-role-policy-document file://trust-policy.json
    aws iam put-role-policy --role-name sms --policy-name sms --policy-document file:/

<center><img src="/images/aws-policy-create.jpeg" width="50%"></center>

Once the vCenter Server Inventory is imported to AWS SMS, and the role is created we can create our first replication job.
<center><img src="/images/aws-sms-replication.jpeg" width="50%"></center>



