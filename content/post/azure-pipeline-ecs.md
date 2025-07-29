+++
title = "Orchestrating AI Deployment: Azure Pipelines Meets VMware ECS"
date = "2024-10-02"
description = "Discover how to seamlessly integrate Azure Pipelines with VMware ECS to automate and streamline the deployment of AI applications, ensuring control, efficiency and reliability across multiple edge environments."
tags = [
    "ecs",
    "vsphere",
    "kubernetes",
    "devops"
    ]
categories = [
    "azure",
    "pipeline",
    "automation"
    ]
thumbnail = "clarity-icons/code-144.svg"
+++

## Introduction

kkk

## Edge AI Challenge

AI applications at the edge offer significant advantages, including local data processing for real-time analysis, reduced latency, offline operation, and scalability without overwhelming central infrastructure. Regular retraining of models allows contiunal incremental accuracy improvements. Regular quantization, pruning, and knowledge distillation also helps reduce model size without loss and improve model performance. However the software lifecycle management of applications across a large estate of distributed edge compute environments presents several significant challenges:

* Ensuring compatibility across different operating systems and software environments is challenging.
* Edge environments can have limited or unreliable network connectivity.
* There's a shortage of IT expertise in edge sites.

To address these challenges, organizations are turning to specialized edge platforms and tools that offer features like:

* Pull-based architectures for efficient management of distributed sites.
* Zero-touch provisioning and lifecycle management.
* Edge-optimized runtimes for both virtualized and containerized applications.
* Integrated telemetry for visibility into application and platform behaviors.

It is a risk to deploy large-scale AI model updates to thousands of edge sites in an uncontrolled fashion. Potential widespread system failures or performance degradation if the updates contain bugs. Without proper testing, updates could introduce new biases or errors in AI decision-making. Without rollback capabilities in an uncontrolled process makes it difficult to quickly address problems. To address these challenges, organizations are looking to deploy continous delivery pipelines and employing testing stage gates.

## GitOps

GitOps is an operational model for cloud-native applications that uses Git as the single source of truth for declarative infrastructure and applications its principles:

* Infrastructure-as-Code (IaC) stored in Git repositories
* Automated synchronization between Git state and cluster state
* Declarative descriptions of the desired system state
* Continuous deployment and reconciliation

## Azure DevOps Pipeline

Azure [Pipelines](https://azure.microsoft.com/en-us/products/devops/pipelines/) is a cloud-based service provided by Microsoft as part of the Azure DevOps suite. It is designed to automate the building, testing, and deployment of code projects. It enables controlled, stagegated deployments with approvals and gates allowing thorough testing in controlled environments before wider rollout. It provides visibility into the deployment process, which is essential for managing and troubleshooting across distributed edge sites.

## VMware Edge Compute Stack (ECS)

[VMware Edge Compute Stack (ECS)](https://www.vmware.com/products/software-defined-edge/edge-compute/stack) is an edge-optimized runtime and orchestration platform designed for managing edge applications and infrastructure across multiple distributed sites. The edge-optimized runtime is managed by VMware Edge Cloud Orchestrator (VECO). VECO enables a GitOps approach to managing edge infrastructure and applications. This approach enables version control and auditing of changes to edge environments. VECO implements a zero-touch provisioning with a pull-based model suitable for edge sites with limited IT skills and intermittent connectivity.

In VECO sites can be used to manage a number of edge hosts in the same location, these hosts can share a common set of configuration files and also be visualized and troubleshooted as a single location. A site can be associated with a GitHub repository, branch and or folder. In my environment I associate each site with a site folder one each for qa, uat and production environments. Within each folder I have:

* configureHost.yaml - Defines the configuration to be applied to the ESXi host
* face-detector.yaml - Defines the configuration of the AI inference application
* vm.yaml - Defines the configuration of other VM workload to be deployed to the site
* metallb.yaml - Defines the configuration of load balancer to expose AI inference application kubernetes service to the outside network

The file and folder structure for the sites and associated declarative configuration files looks something like:

```bash
├── sites
    ├── qa
    │   ├── configureHost.yaml
    │   ├── face-detector.yaml
    │   ├── vm.yaml
    │   └── metallb.yaml
    ├── uat
    │   ├── configureHost.yaml
    │   ├── face-detector.yaml
    │   ├── vm.yaml
    │   └── metallb.yaml
    └── production
        ├── configureHost.yaml
        ├── face-detector.yaml
        ├── vm.yaml
        └── metallb.yaml
```

## Computer Vision Pipeline

A while ago I created a computer vision demonstration application the approach is documented [here](https://darrylcauldwell.github.io/post/computer-vision/). This was designed to make simple changes to the code which reflected visually in the running application. There is the ability to simulate a change the AI model to detect a draw a green box around either full face or just the eyes. There is also the ability to simulate other changes to the app such as changing web app background colour or text.  The source code for this application is stored in a [GitHub repository](https://github.com/darrylcauldwell/keswick). There are various other assets in this repository,  all face detector application files are within folder [face-detector](https://github.com/darrylcauldwell/keswick/tree/main/face-detector). As such the first part of the pipeline watches for commits to files in the face-detector folder in the main repository branch.

```yaml
trigger:
  branches:
    include:
    - main
  paths:
    include:
    - face-detector/*
```

An Azure Pipeline agent is a software component that runs on virtual server in Azure cloud or on a physical device. The agent is responsible for executing the tasks defined in an Azure DevOps pipeline. There is a cost to using Azure cloud agents and I have perfectly good laptop which could run the pipeline agent software.

Any software required by the pipeline to run needs to be available on the agent. I need my pipeline to build docker imaages and so I need to have the Docker Desktop engine running. I have an Arm based Mac in which the docker engine socket location changes from standard. In this standard configuration the Azure pipeline fails as it cannot find docker engine socket.  In this case it is possible to set environment variable to its correct location before starting the agent.

```bash
export DOCKER_HOST=unix://$HOME/.docker/run/docker.sock
```

yq is a lightweight and portable command-line tool for processing YAML, JSON, XML, CSV, and other structured data formats. It is particularly useful for updating YAML files. My pipeline uses yq to update the infrastructure as code yaml files. As such I needed to install this on MacOS using command like:

```bash
brew install yq
```

The next section of the pipeline defines agent pool to use my laptop rather than a cloud instance.

```yaml
pool:
  name: Default
  demands:
    - agent.name -equals Darryls-MacBook-Pro
```

With the base pipeline attributes defined we can look at the stages. A stage is a logical boundary in an Azure DevOps pipeline. Stages can be used to group actions, each stage contains one or more jobs. In this pipeline I have the following stages:

* Build and push multi-arch docker image
* Update the QA environment
* Approval stagegate for QA
* Update the UAT environment
* Approval stagegate for UAT
* Update the production environment


Introduce your approach using Azure Pipelines to manage VMware ECS site YAML files

Emphasize the benefits of this method (automation, consistency, control)

Azure Pipelines: Explain its role in orchestrating the deployment process

VMware ECS: Briefly describe how it's used as the deployment target

YAML Configuration: Discuss the importance of YAML files in defining ECS site configurations

Stage Gates: Explain how these control the flow between environments

## Pipeline Structure

Describe the overall structure of your pipeline

Explain the stages: QA, UAT, and Production

Discuss how you implemented stage gates between environments

## Desired State - YAML Configuration Management

Explain how you manage different YAML configurations for each environment

Discuss any templating or parameterization techniques used