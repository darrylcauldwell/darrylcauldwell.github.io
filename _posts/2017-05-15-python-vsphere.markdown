---
layout: post
title:  "Controlling vSphere With pyVmomi"
date:   2017-05-15 18:20:56 +0100
tags:
    - VMware
permalink: vsphere-python/
---
I am tasked with working with VMware again for now, it seems like a good opportunity to try and manage the infrastructrure configuration as code as we do with AWS. My former colleagues had made me aware of pyVmomi an open source library which VMware provide and mostly maintain for managing vSphere, so its here I shall start.

I am performing this learning exercise in my home lab  using is vSphere 6.5, vSAN 6.5, with Python 2.7.10, although this should work the same with other versions.

Install pyVmomi and open vCenter connection and then initiate an interactive python environment

```bash
git clone https://github.com/vmware/pyvmomi.git
sudo pip install -r ~/pyvmomi/requirements.txt
cd ~/pyvmomi
python
```

Import the pyVim, pyVmomi & SSL libraries we are using,

```python
import ssl
from pyVim import connect
from pyVmomi import vim, vmodl
```

Open connection to vCenter then gather contents as an object

```python
vcenter = connect.SmartConnect(
    host='192.168.1.13',
    user='administrator@vsphere.local',
    pwd='VMware1!',
    port=443,
    sslContext=ssl._create_unverified_context()
)
content = vcenter.RetrieveContent()
```

# Basic get of information

Once we have vCenter Object Model as content object we can output any part of this data

```bash
content.about.fullName
'VMware vCenter Server 6.5.0 build-5178943'
```

We can also explore the Object Model which is well descrived here in the [vSphere SDK API Docs](http://pubs.vmware.com/vsphere-65/topic/com.vmware.wssdk.apiref.doc/right-pane.html) and when we know what we want to look for we can search and display anything we like, for example the list of virtual machines.

```python
objView = content.viewManager.CreateContainerView(content.rootFolder,[vim.VirtualMachine],True)
vmList = objView.view
objView.Destroy()
for  vm in vmList:
    vm.summary.config.name
```

# Basic put of configuration information

As well as getting information from the Object Model we can just as easily apply configuration to items within (assuming the account we connect with has sufficient rights),  for example if we gather the list of hosts and set a advanced option on all of them.

```python
objView = content.viewManager.CreateContainerView(content.rootFolder,[vim.HostSystem],True)
hostList = objView.view
objView.Destroy()
for host in hostList:
    optionManager = host.configManager.advancedOption
    option = vim.option.OptionValue(key='VSAN.ClomRepairDelay', value=long(120))
    optionManager.UpdateOptions(changedValue=[option])
```

It looks like with our new found friend the Python toolkit we can easily create and deploy a configuration.