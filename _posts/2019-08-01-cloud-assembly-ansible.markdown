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
In this blog post I am going to focus on Cloud Assembly integration with Ansible.  The Ansible engine is [fully open source](https://github.com/ansible/ansible), but OSS RedHat also offers a [license model and support package](https://www.ansible.com/products/engine) for Ansible engine. In addition to the Ansible engine RedHat provides support for [Ansible Tower](https://www.ansible.com/products/tower) which offers many enterprise grade features. A useful comparison can be found in the blog [Red Hat Ansible Automation: Engine, Tower or Both](https://www.ansible.com/blog/red-hat-ansible-automation-engine-vs-tower). Cloud Assembly supports integration directly with Ansible engine and does not depend on Ansible Tower.

## Ansible Headless
Ansible does not require a single controlling machine. Any Linux machine can run Ansible. The absence of a central management server requirement can greatly increase availability, simplify disaster-recovery planning and enhance security (no single point of control).

When we use this deployment model each server has Ansible installed and has inventory file with a single entry named localhost. We create a Cloud Assembly blueprint which includes cloud-init configuration to pull an Ansible playbook from a repository and execute it locally. Here the Ansible playbook is specific to the server role on which it will be ran.

An simplified example of how to deploy an application using this deployment architecture using Cloud Assembly is described [here](https://github.com/darrylcauldwell/titoAnsibleHeadless).

## Ansible Server
While running Ansible without a central management server can be useful for some deployment scenarios, others are better suited to having a central Ansible server controlling multiple clients.

Typical factors which would drive a deployment architecture with a central management server would be the following:

* Windows guests as Ansible engine only runs on Linux
* Large deployments that require centralized control for day two operations

When we use this deployment model we deploy an Ansible central management server and this we create SSH authorization private public key pair. We then configure this to be integrated with Cloud Assembly.

We create a Cloud Assembly blueprint which includes on each VM resource cloud-init configuration to create a local account named ansible that can be accessed using the public authorization key. The Cloud Assembly blueprint then has Ansible resource added with details of which playbook to execute which are linked to the VM resources. When the blueprint is deployed the VM resources get added dynamically to the Ansible server inventory file and then the playbooks executed. In this model the Ansible playbook can be application rather than server role specific.

An simplified example of how to deploy an application with this deployment architecture using Cloud Assembly is described [here](https://github.com/darrylcauldwell/titoAnsible).  This also includes a simplified example of how Ansible server can be used for day two operations for servers of a specific type.
