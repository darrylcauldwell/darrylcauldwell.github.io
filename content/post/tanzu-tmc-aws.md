+++
title = "Deploying Tanzu for AWS Using Tanzu Misson Control"
date = "2021-12-14"
description = "Deploying vSphere with Tanzu"
tags = [
    "tanzu",
    "kubernetes",
    "aws"
]
categories = [
    "homelab"
]
thumbnail = "clarity-icons/code-144.svg"
+++
Fourth in a series of posts which build on each other looking Tanzu.

1. [Deploying Tanzu for vSphere with NSX-T](/post/tanzu-basic-nsx)
2. [Managing Tanzu for vSphere Clusters Using ClusterAPI](/post/tanzu-tkc)
3. [Managing Tanzu for vSphere Clusters Using Tanzu Misson Control](/post/tanzu-tmc)
4. [Deploying Tanzu for AWS Using Tanzu Misson Control](/post/tanzu-tmc-aws)

Tanzu Mission Control hosts a TKG management cluster on AWS that you can use to provision workload clusters into your AWS account. The TKG management cluster is pre-registered within TMC with name aws-hosted.

![Tanzu Mission Control Provisioner](/images/tanzu-tmc-aws-prov.png)

Deploying a TKG cluster to AWS creates and configures EC2 instances in a region of your choosing. In order that TMC can create these requires AWS IAM credentials. The process to create these is initiated from within TMC and first step is associate to provisoner and specify a name.

![Tanzu Mission Control Credential](/images/tanzu-tmc-aws-cred-name.png)

The UI outputs an AWS CloudFormation stack file which can be used to create IAM resources.

![AWS CloudFormation IAM Input](/images/tanzu-tmc-aws-cloudform-in.png)

When AWS CloudFormation stack is created the role ARN is output.

![AWS CloudFormation IAM Output](/images/tanzu-tmc-aws-cloudform-out.png)

This ARN is then input fed back into TMC to complete the association of IAM Role with TMC Credential.

![Tanzu Mission Control Credential ARN](/images/tanzu-tmc-aws-cred-arn.png)

The IAM role will give TMC the ability to create EC2 instances. In addition to this an EC2 SSH key pair is required in each region. I will be looking to deploy cluster to eu-west-1 Ireland.

![Tanzu Mission Control Credential ARN](/images/tanzu-tmc-aws-cred-ssh.png)

With these in place we can now look to create cluster using the aws-hosted with the provider and credentials we just created.

![Tanzu Mission Control Cluster Create](/images/tanzu-tmc-aws-clus-create.png)

Then specifying the region, SSH key and Kubernetes version to deploy. I do not already have a VPC in place so I'll look to have TMC create one to house this cluster. On-premises I had started allocating NSX-T subnets from 172.16.0.0/17 so am using the other half for AWS VPC so they do not overlap.

![Tanzu Mission Control Cluster Configuration](/images/tanzu-tmc-aws-clus-configure.png)

This is just a test so I want to keep costs low so choose single node control plane and single node node pool.

![Tanzu Mission Control Cluster Control Plane](/images/tanzu-tmc-aws-clus-control.png)

![Tanzu Mission Control Cluster Node Pool](/images/tanzu-tmc-aws-clus-nodepool.png)

After a short time the cluster is deployed and cluster health is visible in TMC.

![Tanzu Mission Control Cluster Health](/images/tanzu-tmc-aws-clus-health.png)

In addition to the two cluster nodes a third EC2 instance is deployed. Note this has external IP address and can be used as bastion host into VPC to connect onwards.

![Tanzu Mission Control Cluster Bastion](/images/tanzu-tmc-aws-clus-bastion.png)
