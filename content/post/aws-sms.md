+++
title = "Migrate VMware VMs to AWS EC2 using Server Migration Services (SMS)"
date = "2016-11-01"
description = "PMigrate VMware VMs to AWS EC2 using Server Migration Services (SMS)"
tags = [
    "aws",
    "windows",
    "migration"
]
categories = [
    "automation"
]
thumbnail = "clarity-icons/code-144.svg"
+++
The [AWS Server Migration Service](https://aws.amazon.com/server-migration-service/) simplifies and streamlines the process of migrating existing virtualized applications to Amazon EC2. AWS SMS allows you to automate, schedule, and track incremental replications of live server volumes, making it easier for you to coordinate large-scale server migrations. Presently this only supports migrating from VMware with support for other hypervisors and physical servers is coming soon.

Amazon provide a virtual appliance (OVA) which can be imported into your existing vCenter, once booted this is configured to connect to vCenter and your AWS account. Once configured this appliance is controlled by the AWS SMS service to take a snapshot of VMware virtual machines and facilitate the upload of the snapshot copy into an S3 bucket.  Once the VMware snapshot is uploaded into the S3 bucket the SMS service then updates the disk format and then prepare this as an AMI.

The size of virtual machine images is such that the upload process might well take a long time, and during this time you might well want to keep the virtual machine running. The changes made during this time therefore will not be reflected in the initial AMI created. As such AWS SMS offers the option of running the replication job again, but rather than the job creating a full new virtual machine it takes an incremental snapshot.  Once the upload of this is completed the SMS service processes the initial upload with the incremental to form a new AMI.

## How To: Configure Server Migration Services (SMS)

The process is pretty straight forwards,  first task is to download the AWS Server Migration Connector from [S3](https://s3.amazonaws.com/sms-connector/AWS-SMS-Connector.ova), then deploy the OVA.  

![Deploy OVF Template](/images/aws-sms-ova-deploy.jpeg)

![Deploy OVF Ready to complete](/images/aws-sms-ova-deploy-final.jpeg)

The SMS Connector needst to connect to your AWS account and therefore we need to create a user with the "ServerMigrationConnector" role attached.

![MigrationUser Role Mapping](/images/aws-sms-account.jpeg)

Once the SMS connector appliance is deployed connect to the web UI by opening the browser to https://dhcp-addr

![AWS Server Migration Service Splash](/images/aws-sms-cfg-wiz.jpeg)

Work through the wizard to configure all,

* Step 1: License Agreement
* Step 2: Create a Password
* Step 3: Network Info
* Step 4: Log Uploads and Upgrades
* Step 5: Server Migration Service

Once configuration is complete the connection to AWS and vCenter should show good in the connector configuration.

![AWS Server Migration Service Configuration Complete](/images/aws-sms-cfg-complete.jpeg)

If we then connect to AWS we can see the connector.

![AWS Server Migration Service Complete Console](/images/aws-sms-cfg-complete-console.jpeg)

In order to create the first replication job we need to import the list of vCenter VMs by using the 'Import server catalog' function.

![AWS Server Migration Service Import Server Catalog](/images/aws-sms-import.jpeg)

There is not a default role supplied which the SMS services can use to form AMI's from the uploaded VMware snapshots. To do this download this file [trust-policy.json](/attachments/trust-policy.json) and this file [role-policy.json](/attachments/role-policy.json). Then at a command prompt, go to the directory where you stored the two JSON files, and run the following commands to create the SMS service role:  

```bash
aws iam create-role --role-name sms --assume-role-policy-document file://trust-policy.json
aws iam put-role-policy --role-name sms --policy-name sms --policy-document file://role-policy.json
```

## How To: Migrate Virtual Machine

Once the vCenter Server Inventory is imported to AWS SMS, and the role is created we can create our first replication job by using the SMS service wizard.

* Step 1: Select the servers

Select the virtual machine(s) you would like to migrate.

* Step 2: Configure server-specific settings

Select the license type for the guest operating system of the virtual server(s) being migrated.

* Step 3: Configure replication job settings

Schedule the replication job, assuming you used the two files about to create the SMS role for IAM service role leave as default 'sms'.

* Step 4: Review

Once the job schedules,  the first task the job performs is to create a VMware snapshot, it's important to remember to have enough disk capacity to hold snapshots.

![AWS Server Migration Service Seeding Snapshot](/images/aws-sms-seed-snapshot.jpeg)

The AWS console doesn't update very well, so its best to view the progress via the AWS CLI. First list all of the replication jobs and from this you can find the JobID then to make it easier to read target the output to the specific JobID.

```bash
aws sms get-replication-jobs  
aws sms get-replication-jobs --replication-job-id sms-job-2ca54045  
```

A replication job refers to the server being migrated,  as we mentioned earlier multiple replications can occur for example the initial seed and an incremental.  Therefore within a replication job there might be various replication runs.

```bash
aws sms get-replication-runs --replication-job-id sms-job-2ca54045  
```

In the following example we can see this has three runs in the runlist, the initial seed which has completed, an incrememental which has completed, it also has a pending job as I had left the default replication job values to schedule a daily incremental.

![AWS Server Migration Service Replication Runs](/images/aws-sms-repl-runs.jpeg)

When a job is running you would see the state roll through the various stages

* Pending  
* Active  
* Complete  

While the job state is in the Active state the statusMessage rolls through the various stages  

* Uploading  
* Converting  
* Preparing  
* Completed  

Each run forms a new AMI, each newly created AMIs can then be launched in ec2, so you can for example start from the initial seed replication, one containing all the incrementals or anywhere in between.

## Offical SMS Documentation Links

[Marketing Page](https://aws.amazon.com/server-migration-service/)  

[AWS Blog](https://aws.amazon.com/blogs/aws/new-aws-server-migration-service/)  

[Tech Documentation](http://docs.aws.amazon.com/ServerMigration/latest/userguide/server-migration.html)