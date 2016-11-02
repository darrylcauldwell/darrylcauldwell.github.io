---
layout: post
title:  "AWS Server Migration Services (SMS) - Seed Upload Issue"
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

In theory this should run the initial seed replication, I started my first replication of a small CentOS7 VM at around 5pm and left it running all night,  on checking at 9am it still had not completed.
<center><img src="/images/aws-sma-import-status.jpeg" width="50%"></center>

While I don't have the worlds fastest internet upload connection this should have completed in 16hrs so I went in search of the log files.  Usefully if you connect to the web UI of the SMS Connector there is a 'Support Link' one of which is 'Download Log Bundle'. Clicking this downloads connector-debug.tar.gz after a lot of trial and error looking through the various files within I found,

    /connector-debug/var/log/connector/sms-replications-poller.log

The SMS Connector replication poller appears to be a Java application which performs the inventory and upload services. The output is left quite verbose so its fairly easy to following the upload process in summary it flows like this,  

* ValidateReplicationJob (Connects to vCenter and confirms permissions)
* CREATE_UPLOAD_BUCKET (Connects to S3 and creates a bucket)
* CREATE_BASE_SNAPSHOT (Connects to vCenter and creates a snapshot)
* UPLOAD_BASE_SNAPSHOT (Starts a InMemoryStreamingMultipartUpload of snapshot to S3)

For my upload I can see it starting well and uploading,

``` 
2016-11-01 17:08:21.447 [INFO ] [RTP:sms-job-75a5401c:sms-run-04a4416d:UploadBaseSnapshot] ReplicationTaskPoller:184 - ---------------------------------------------------------------
2016-11-01 17:08:21.447 [INFO ] [RTP:sms-job-75a5401c:sms-run-04a4416d:UploadBaseSnapshot] ReplicationTaskPoller:185 - 	MESSAGE: UPLOAD_BASE_SNAPSHOT
2016-11-01 17:08:21.451 [INFO ] [RTP:sms-job-75a5401c:sms-run-04a4416d:UploadBaseSnapshot] ReplicationTaskPoller:216 - 	Available slots: 10 LRO, 20 fast. Used LRO slot.
2016-11-01 17:08:21.451 [INFO ] [RTP:sms-job-75a5401c:sms-run-04a4416d:UploadBaseSnapshot] ReplicationTaskPoller:228 - 	STARTED: UPLOAD_BASE_SNAPSHOT
2016-11-01 17:08:21.451 [INFO ] [RTP:sms-job-75a5401c:sms-run-04a4416d:UploadBaseSnapshot] ReplicationTaskPoller:229 - ---------------------------------------------------------------
2016-11-01 17:08:21.632 [INFO ] [sms-job-75a5401c:sms-run-04a4416d:UploadBaseSnapshot] SMSClient:156 - SendMessage: {Type: UPLOAD_BASE_SNAPSHOT_RESULT, ContextToken: sms-job-75a5401c:sms-run-04a4416d:UploadBaseSnapshot, TaskState: STARTING}
2016-11-01 17:08:22.961 [INFO ] [sms-job-75a5401c:sms-run-04a4416d:UploadBaseSnapshot] UploadBaseSnapshotBasicOperation:88 - Successfully responded to SMS to start the task
2016-11-01 17:08:22.961 [INFO ] [sms-job-75a5401c:sms-run-04a4416d:UploadBaseSnapshot] VSphereClientFactory:52 - Logging into vcenter.darrylcauldwell.local as administrator@vsphere.local
2016-11-01 17:08:23.039 [INFO ] [sms-job-75a5401c:sms-run-04a4416d:UploadBaseSnapshot] VSphereClient:90 - Connected: vSphere session, login: 'VSPHERE.LOCAL\Administrator', IP/Hostname: vcenter.darrylcauldwell.local, Product: VMware vCenter Server 6.0.0 build-3634794, apiVersion: 6.0, OS: linux-x64, Server instanceUuid: 25e94de5-86ce-4284-b3f7-004d54883739
2016-11-01 17:08:23.042 [INFO ] [sms-job-75a5401c:sms-run-04a4416d:UploadBaseSnapshot] SmsVSphereClientFactory:69 - Connected to vSphere: VMware vCenter Server 6.0.0 build-3634794
2016-11-01 17:08:23.042 [INFO ] [sms-job-75a5401c:sms-run-04a4416d:UploadBaseSnapshot] UploadBaseSnapshotBasicOperation:376 - vSphere session, login: 'VSPHERE.LOCAL\Administrator', IP/Hostname: vcenter.darrylcauldwell.local, Product: VMware vCenter Server 6.0.0 build-3634794, apiVersion: 6.0, OS: linux-x64, Server instanceUuid: 25e94de5-86ce-4284-b3f7-004d54883739
2016-11-01 17:08:23.313 [INFO ] [sms-job-75a5401c:sms-run-04a4416d:UploadBaseSnapshot] EsxCertificateUtils:22 - ESX certificate of HostSystem:host-20 @ https://vcenter.darrylcauldwell.local/sdk of VirtualMachine:vm-162 @ https://vcenter.darrylcauldwell.local/sdk found, size: 1468
2016-11-01 17:08:23.320 [INFO ] [sms-job-75a5401c:sms-run-04a4416d:UploadBaseSnapshot] UploadBaseSnapshotBasicOperation:99 - Snapshot: snapshot-164
2016-11-01 17:08:23.437 [INFO ] [Thread-7] UploadBaseSnapshotBasicOperation$1:130 - Renewed lease...
2016-11-01 17:08:23.444 [INFO ] [Finalizer] VSphereClient:280 - Already logged out.
2016-11-01 17:08:23.445 [INFO ] [Finalizer] VSphereClient:280 - Already logged out.
2016-11-01 17:08:23.465 [INFO ] [sms-job-75a5401c:sms-run-04a4416d:UploadBaseSnapshot] SnapshotStream:163 - Vmdkurl: https://esx2.darrylcauldwell.local/nfc/52da312e-52f9-ae78-46dd-687261688222/disk-1.vmdk
2016-11-01 17:08:23.465 [INFO ] [sms-job-75a5401c:sms-run-04a4416d:UploadBaseSnapshot] SnapshotStream:176 - Added vmdkurl: https://esx2.darrylcauldwell.local/nfc/52da312e-52f9-ae78-46dd-687261688222/disk-1.vmdk and contentID: 4e3f3d82a59ef77087fee67c67ee9838 to the map
2016-11-01 17:08:23.465 [INFO ] [sms-job-75a5401c:sms-run-04a4416d:UploadBaseSnapshot] SnapshotStream:181 - Got vmdkUrls...
2016-11-01 17:08:23.467 [INFO ] [sms-job-75a5401c:sms-run-04a4416d:UploadBaseSnapshot] UploadBaseSnapshotBasicOperation:283 - Starting progress updater...
2016-11-01 17:08:23.468 [INFO ] [sms-job-75a5401c:sms-run-04a4416d:UploadBaseSnapshot] UploadBaseSnapshotBasicOperation:285 - Progress updater started
2016-11-01 17:08:23.469 [INFO ] [sms-job-75a5401c:sms-run-04a4416d:UploadBaseSnapshot] UploadBaseSnapshotBasicOperation:171 - URL: https://esx2.darrylcauldwell.local/nfc/52da312e-52f9-ae78-46dd-687261688222/disk-1.vmdkDiskInfoDiskInfo(controllerName=VirtualLsiLogicController, controllerKey=1000, busNumber=0, unitNumber=0, longContentId=4e3f3d82a59ef77087fee67c67ee9838, capacityInBytes=17179869184)
2016-11-01 17:08:23.499 [INFO ] [sms-job-75a5401c:sms-run-04a4416d:UploadBaseSnapshot] UploadBaseSnapshotBasicOperation:178 - sms-job-75a5401c/sms-run-04a4416d/4e3f3d82a59ef77087fee67c67ee9838.vmdk
2016-11-01 17:08:23.501 [INFO ] [sms-job-75a5401c:sms-run-04a4416d:UploadBaseSnapshot] InMemoryStreamingMultipartUpload:53 - Initiating upload
2016-11-01 17:08:24.364 [INFO ] [sms-job-75a5401c:sms-run-04a4416d:UploadBaseSnapshot] InMemoryStreamingMultipartUpload:62 - Upload initiated, upload Id:HX5mm4ELRT17CGnsm4a6voAjP4ryyCTnb34qy1C0_1aMKE8zJtuo1Ob7sN56GMbIyFYjTRaB8lFzKUq2AwBOmJFmywRI2zzURdhmKk_yivl7Zzo5LcbvHaOBiabEMHOw
2016-11-01 17:08:24.364 [INFO ] [sms-job-75a5401c:sms-run-04a4416d:UploadBaseSnapshot] InMemoryStreamingMultipartUpload:71 - Attempting to upload part 1
2016-11-01 17:08:28.605 [INFO ] [pool-2-thread-1] SMSClient:156 - SendMessage: {Type: UPLOAD_BASE_SNAPSHOT_RESULT, ContextToken: sms-job-75a5401c:sms-run-04a4416d:UploadBaseSnapshot, TaskState: ONGOING, Progress: 0}
2016-11-01 17:08:31.060 [INFO ] [sms-job-75a5401c:sms-run-04a4416d:UploadBaseSnapshot] InMemoryStreamingMultipartUpload:79 - Uploading part 1
2016-11-01 17:30:37.399 [INFO ] [sms-job-75a5401c:sms-run-04a4416d:UploadBaseSnapshot] InMemoryStreamingMultipartUpload:89 - Bytes Transferred: 104857600
2016-11-01 17:30:37.400 [INFO ] [sms-job-75a5401c:sms-run-04a4416d:UploadBaseSnapshot] InMemoryStreamingMultipartUpload:103 - Part uploaded
2016-11-01 17:30:37.400 [INFO ] [sms-job-75a5401c:sms-run-04a4416d:UploadBaseSnapshot] UploadBaseSnapshotBasicOperation:197 - Upload Progress Percentage: 0
2016-11-01 17:30:37.400 [INFO ] [sms-job-75a5401c:sms-run-04a4416d:UploadBaseSnapshot] InMemoryStreamingMultipartUpload:71 - Attempting to upload part 2
2016-11-01 17:30:46.970 [INFO ] [sms-job-75a5401c:sms-run-04a4416d:UploadBaseSnapshot] InMemoryStreamingMultipartUpload:79 - Uploading part 2
2016-11-01 18:15:48.541 [INFO ] [sms-job-75a5401c:sms-run-04a4416d:UploadBaseSnapshot] UploadBaseSnapshotBasicOperation:197 - Upload Progress Percentage: 1
``` 

However it then looks like Part 2 fails to upload with java.io.IOException

``` 
2016-11-01 18:15:48.599 [ERROR] [sms-job-75a5401c:sms-run-04a4416d:UploadBaseSnapshot] UploadBaseSnapshotBasicOperation:227 - Failed when uploading base disk(s) to S3......
java.io.IOException: Premature EOF
2016-11-01 18:15:48.727 [INFO ] [sms-job-75a5401c:sms-run-04a4416d:UploadBaseSnapshot] SMSClient:156 - SendMessage: {Type: UPLOAD_BASE_SNAPSHOT_RESULT, ContextToken: sms-job-75a5401c:sms-run-04a4416d:UploadBaseSnapshot, TaskState: ERROR, StatusCode: CONNECTOR_ERROR, StatusMessage: Failed to upload base disk(s) to S3. Please try again. If this problem persists, please contact AWS support: Premature EOF, DiagnosticMessage: java.io.IOException: Premature EOF
2016-11-01 18:15:48.727 [INFO ] [sms-job-75a5401c:sms-run-04a4416d:UploadBaseSnapshot] SMSClient:156 - SendMessage: {Type: UPLOAD_BASE_SNAPSHOT_RESULT, ContextToken: sms-job-75a5401c:sms-run-04a4416d:UploadBaseSnapshot, TaskState: ERROR, StatusCode: CONNECTOR_ERROR, StatusMessage: Failed to upload base disk(s) to S3. Please try again. If this problem persists, please contact AWS support: Premature EOF, DiagnosticMessage: java.io.IOException: Premature EOF
``` 

The java job sms-job-75a5401c:sms-run-04a4416d:UploadBaseSnapshot then appears to exit and think its finished upload of base snapshot.

``` 
2016-11-01 18:15:49.590 [INFO ] [sms-job-75a5401c:sms-run-04a4416d:UploadBaseSnapshot] ReplicationTaskPoller:249 - ---------------------------------------------------------------
2016-11-01 18:15:49.590 [INFO ] [sms-job-75a5401c:sms-run-04a4416d:UploadBaseSnapshot] ReplicationTaskPoller:250 - 	FINISHED: UPLOAD_BASE_SNAPSHOT:sms-job-75a5401c:sms-run-04a4416d:UploadBaseSnapshot
2016-11-01 18:15:49.590 [INFO ] [sms-job-75a5401c:sms-run-04a4416d:UploadBaseSnapshot] ReplicationTaskPoller:252 - ---------------------------------------------------------------
``` 



This appears bad,  not because the upload failed as uploads do fail,  but bad that as the AWS Console SMS Service shows that the upload is progressing without error when actually the Connector has failed and is not uploading.
