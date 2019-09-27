---
layout: post
title:  "Cloudbased-init For vSphere"
date:   2019-09-17 22:20:56 +0100
tags:
    - DevOps
    - VMware
permalink: cloudbased-init/
---
vRealize Automation Cloud allows you to deploy and configure cloud agnostic virtual machines. To achieve agnostic in-guest OS customization capability across for on-premises vSphere, AWS, GCP and Azure endpoints [cloud-init](https://cloud-init.io/) is used for Linux guests and [cloudbase-init](https://cloudbase.it/cloudbase-init/) is used for Windows guests.

## Inital Attempt All Defaults
In vSphere I created a Windows 2019 VM with defaults except for mapping ISO as CD/ROM. I then ran through installer to install Standard edition with Desktop experience, installed VMware Tools, disabled firewall and enabled Remote Desktop. I installed Cloudbased- init 0.9.12.dev76 and optional Carbon PowerShell module to default location. For configuration options I left the default Admin user be created into Administrators group, in addition I selected option to ‘Run Cloudbased-Init service as LocalSystem’. I chose options to 'Run Sysprep to create a generalized image' and 'Shutdown when Sysprep terminates'. When complete I change virtual machine to be a virtual machine template.
 
I connect to vRealize Automation Cloud > Infrastructure > Cloud Accounts > {my account} and run 'Sync Images'.  Once image sync is completed vRealize Automation Cloud > Infrastructure > Image Mappings and create 'New Image Mapping'. Create a Blueprint with single Cloud Agnostic Machine linked to the image mapping.

When I provision a vRA Cloud blueprint to deploy a VM
    formatVersion: 1
    inputs: {}
    resources:
    Cloud_Machine_1:
        type: Cloud.Machine
        properties:
        image: dc-win2019
        flavor: small
        cloudConfig: |
          #cloud-config
          hostname: i-got-a-new-name
 
The VM powers on and cloud-data gets passed by vRealize Automation Cloud as an OVF which is mounted as CDROM with CDROM contents ovf-env.xml in its root.
 
Cloud-init runs on startup but appears to fail to do what it is asked, Cloudbase-init.log contains error.
```
2019-09-26 16:00:15.041 4360 ERROR cloudbaseinit.metadata.services.base [-] HTTPConnectionPool(host='169.254.169.254', port=80): Max retries exceeded with url: /openstack/latest/meta_data.json
```
 
## Adding OVF 
Looking at the error suggested to me that Cloudbase-Init was not looking on the CDROM. I read through some of the Cloudbase-Init documentation and found that metadata service configuration was specified in,

```
C:\Program Files\Cloudbase Solutions\Cloudbase-Init\conf\cloudbase-init-unattend.conf
```

Checking the default configuration it does not include the [OvfService metadata service](https://cloudbase-init.readthedocs.io/en/latest/services.html):

```
metadata_services=cloudbaseinit.metadata.services.configdrive.ConfigDriveService,cloudbaseinit.metadata.services.httpservice.HttpService,cloudbaseinit.metadata.services.ec2service.EC2Service,cloudbaseinit.metadata.services.maasservice.MaaSHttpService
```

If I follow similar steps to before and install Cloudbase-init but DO NOT chose wizard options to 'Run Sysprep to create a generalized image' and 'Shutdown when Sysprep terminates'.  Instead I update entry in the cloudbase-init-unattend.conf file include OvfService like:

```
metadata_services=cloudbaseinit.metadata.services.ovfservice.OvfService,cloudbaseinit.metadata.services.configdrive.ConfigDriveService,cloudbaseinit.metadata.services.httpservice.HttpService,cloudbaseinit.metadata.services.ec2service.EC2Service,cloudbaseinit.metadata.services.maasservice.MaaSHttpService
```

Then manually run sysprep calling the Cloudbase-Init unattend files:

```
cd “C:\Program Files\Cloudbase Solutions\Cloudbase-Init\conf\”
C:\Windows\System32\Sysprep\Sysprep.exe /quiet /generalize /oobe /shutdown /unattend:unattend.xml
```

When I now deploy it finds the OVF mounted as CDROM and file upon it.