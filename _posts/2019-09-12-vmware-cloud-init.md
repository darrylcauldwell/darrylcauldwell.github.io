---
layout: post
title:  "Understanding VMware Guest OS Customization"
date:   2019-09-11 22:20:56 +0100
tags:
    - DevOps
    - VMware
permalink: vmware-cloud-init/
---
I recently began looking more closely at cloud-init for customizing VMware guest VMs. This caused me some heachaches! I took various notes while troubleshooting and researching around this.

## Why Guest OS Customization Is Needed

When you clone a virtual machine or deploy a virtual machine from a template it clones the virtual hard disk and all guest operating system settings. In many deployment scenarios if virtual machines with identical settings are deployed conflicts can occur, such as duplicate computer names. Typically for this deployment scenario during deployment guest operating system customization is required to be performed to give them uniqueness.

## VM Guest OS Customization Approach

VMware has long provided the facility to perform guest operating system customization through a combination of vCenter Server and VMware Tools. This gives the option to choose to launch the Guest Customization wizard during the cloning or deployment process,  or can create standard customization specifications and apply these during the cloning or deployment process.

When the virtual machine template is Linux the VMware guest operating system customization is performed using a combination of cloud-init and a collection of perl scripts packaged with open-vm-tools.

## Logfiles and Log Configuration

Guest OS can be complex and things can go wrong during guest OS customization. When they do its useful to understand where to look for clues as to cause. 

If there were any problems with the extraction and deployment of the customization package onto the guest operating system these are written to:

```
/var/log/vmware-imc/toolsDeployPkg.log
````

VMware guest operating system customization steps performed using the perl scripts output to default location of /var/log/messages. There are many other things in this log file so can use grep to view only the customization steps:

```
grep "customize-guest" /var/log/messages 
```

There are many options for configuring cloud-init logging. These can be configured these within your template virtual machine. The configuration settings are stored in:

```
/etc/cloud/cloud.cfg.d/05_logging.cfg
```

Within this configuration file we can identify that by default cloud-init directs the main handler output to a file:

```
/var/log/cloud-init.log
```

We can also see that by default the stdout and stderr of the specific we tell it to perform is written to:

```
/var/log/cloud-init-output.log
```

## Customization Guest Customatization Configuration

There are many options which cloud-init can be configured with. These can be configured these within your template virtual machine. The configuration settings are stored in:

```
/etc/cloud/cloud.cfg
```

One of the options which can be configured is whether VMware tools perl scripts are ran. How to enable or disable this is documented in this article:

```
https://kb.vmware.com/s/article/59557
```

It is useful to note here that there is currently a known issue when cloud-init calls the open-vm-tools perl scripts for 'Ubuntu 18.04'. The boot sequence and bring up order can cause the perl scripts to fail. A high level cause and workaround are described here:

```
https://kb.vmware.com/s/article/56409 
```

A more detailed description of the bug including some very interesting detail about how cloud-init and open-vm-tools perl scripts are called is described here:

```
https://bugs.launchpad.net/ubuntu/+source/open-vm-tools/+bug/1793715
```

## Cloud-Init User-Data

One of the more useful enhancements of using cloud-init is the ability to instruct the guest operating system to run things such as custom scripts during boot.

User-Data content supports the following types of content 'gzip compressed' or 'mime multi-part archive'. With a mime multi-part file, the user can specify more than one type of data within the content.  For full details on the User-Data content format see here:

```
https://cloudinit.readthedocs.io/en/latest/topics/format.html
```

The two most common data types I use at the moment are Cloud Config Data or simple shell scripts. To differentiate sections when using a 'mime multi-part archive' begin each section with data type for example  ``` #! ``` for shell script or ``` #cloud-config ``` for cloud config data.

Cloud-init can be fussy about syntax of the file contents once created you can validate the syntaxt using a command like this:

```
cloud-init devel schema --config-file /tmp/user-data
```

## vCenter Deploy OVA Import

Some operating systems packages which include cloud-init expose options to confiure during deployment.  For example Ubuntu which can be found here:

```
https://cloud-images.ubuntu.com/releases/18.04/release/ubuntu-18.04-server-cloudimg-amd64.ova
```

The Ubunutu image includes property 'Encoded user-data' this has property description *"In order to fit into a xml attribute, this value is base64 encoded . It will be decoded, and then processed normally as user-data"*.

I like to keep a file copy of the un-encoded and encoded user-data. To do this I use text editior to create a file with the required cloud-init user-data contents.  I then use [base64](https://linux.die.net/man/1/base64) to output contents into a file appended with .b64.

```
base64 --input /tmp/user-data --output /tmp/user-data.b64
```

If encoded user-data is passed during deployment of OVA file in this way this will run when the VM boots.

## Cloud Assembly

Other VMware products such as Cloud Assembly can utilize open-vm-tools and cloud-init to perform guest customization.

```
https://docs.vmware.com/en/VMware-Cloud-Assembly/services/Using-and-Managing/GUID-70EA052D-FABF-4CE5-875D-9B52FED08AA3.html
```
```
https://blogs.vmware.com/management/2018/11/customizing-cloud-assembly-deployments-with-cloud-init.html
```