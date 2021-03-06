+++
title = "Ansibile NSX Configuration - Infrastructure As Code"
date = "2016-06-07"
description = "Ansibile NSX Configuration - Infrastructure As Code"
tags = [
    "nsx",
    "python",
    "ansible"
]
categories = [
    "networking",
    "automation"
]
+++
With the rise of DevOps culture and the ethos of automate all the things. Using test driven development techniques to apply application configuration declaratively has become widespread. The ever closer working relations of developers and infrastructure operations staff to become an application centric delivery and management unit has lead to many benefits.  One of the benefits is the shared empathy for each others pain points and understanding of how each approaches tackling these. It is clear that managing configuration as code has many benefits for consistent repeated delivery of applications.

Tools which are commonly used for deploying applications with declarative configuration include Puppet, Chef and Ansible. The processes for distributed source control are also mature and with Git, Bitbucket and Apache Subversion there is a DVCS for most use-cases. As we work as one delivery team sharing tools and repositories is natural as with this we can look to deploy our infrastructure and application configuration with a single tool and avoid issues with interoperability and hand off.

VMware NSX offers many things to many people with this rich feature set comes complexity in configuration.  At VMworld Europe 2015 I was introduced to Yves Fauser and attended his session [‘NET4972 – Incorporating VMware NSX in your DevOps Toolchain – Network Programmability with Python and Ansible‘.](https://vmworld2015.lanyonevents.com/connect/sessionDetail.ww?SESSION_ID=4972&tclass=popup)  He had created an [NSX RAML specification]((http://github.com/vmware/nsxraml), a [Python NSX RAML client](http://github.com/vmware/nsxramlclient.) he then brings these altogether into a usable form by way of an [NSX Ansible module.](https://github.com/vmware/nsxansible)  The [Ansible module](https://github.com/vmware/nsxansible) offers examples which give the ability to NSX Manager, configure NSX to vCenter integration, configure VXLAN, deploy NSX Controllers and then deploy some logical switches.

I explorered this capability in [this follow up post.]({{ site.url }}/deploy-nsx-from-ansible)