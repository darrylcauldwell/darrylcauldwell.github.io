---
layout: post
title:  "Cloud Assembly VM Guest Management With Ansible"
date:   2019-08-01 20:20:56 +0100
tags:
    - Ansible
    - DevOps
permalink: cloud-assembly-ansible/
---
VMware [Cloud Automation Services](https://cloud.vmware.com/cloud-automation-services) is a product suite comprising three products, [Cloud Assembly](https://cloud.vmware.com/cloud-assembly), [Service Broker](https://cloud.vmware.com/service-broker) and [Code Stream](https://cloud.vmware.com/code-stream). Cloud Assembly provides a blueprinting engine and manages connectivity to multiple cloud endpoints. Service Broker provides a unified catalog for publishing blueprints with role-based policies. Code Stream provides a release pipeline with analytics.

The Cloud Assembly blueprinting engine can be used to consistently deliver VMs and applications.  To deliver VM guest configuration Cloud Assembly natively uses [cloud-init](https://cloudinit.readthedocs.io/en/latest/). This native VM guest configuration capability can be extended with desired state management tools [Puppet](https://puppet.com/) and [Ansible](https://www.ansible.com/). A major difference between the Puppet and Ansible is that Puppet requires an in-guest agent where Ansible operates agentless.

## Ansible
In this blog post I am going to focus on Ansible.  The Ansible engine is [fully open source](https://github.com/ansible/ansible), as well as OSS RedHat offers a [license model and support package](https://www.ansible.com/products/engine) for Ansible engine. As well as Ansible engine RedHat offer [Ansible Tower](https://www.ansible.com/products/tower) which offers many enterprise grade features. Cloud Assembly supports integration directly with Ansible engine and does not depend on Ansible Tower.

## Ansible Headless
Ansible does not require a single controlling machine, any Linux machine can run Ansible. The absence of a central management server requirement can greatly increase availability, simplify disaster-recovery planning and increased security (no single point of control).

An simplified example of how to deploy an application using this deployment architecture using Cloud Assembly is described [here](https://github.com/darrylcauldwell/titoAnsibleHeadless).

## Ansible Server
While running Ansible without a central management server can be useful for some deployment scenarios, others are better suited to having a central Ansible server controlling multiple clients.

Typical factors which would drive a deployment architecture with a central management server would be,

* Windows guests as Ansible engine only runs on Linux
* Large deployments require centralized control for day two operations

An simplified example of how to deploy an application using this deployment architecture using Cloud Assembly is described [here](https://github.com/darrylcauldwell/titoAnsible).  This also includes a simplified example of how Ansible server can be used for day two operations for servers of a specific type.