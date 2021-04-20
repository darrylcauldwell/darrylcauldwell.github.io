+++
title = "Controlling vSphere & NSX-V With Python"
date = "2017-05-15"
description = "Controlling vSphere & NSX-V With Python"
tags = [
    "nsx",
    "python"
]
categories = [
    "networking",
    "automation"
]
thumbnail = "clarity-icons/code-144.svg"
+++
My former colleagues had made me aware of pyVmomi an open source library which VMware provide and mostly maintain for managing vSphere, so its here I shall start. Since then NSX for vSphere has also an open source library NSX RAML Client provided by VMware so I'll then move to that.

I am performing this learning exercise in my home lab  using is vSphere 6.5, vSAN 6.5, NSX6.3, with Python 2.7.10, although this should work the same with other versions.

Install pyVmomi and open vCenter connection and then initiate an interactive python environment

```bash
git clone https://github.com/vmware/pyvmomi.git
sudo pip install -r ~/pyvmomi/requirements.txt
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

# NSX for vSphere
So we have the pyVmomi library for vSphere, in addition to this VMware have provided open source library for [NSX for vSphere](https://github.com/vmware/nsxramlclient).

We'll first make sure the packages are installed along with the additional packages for managing NSX.

```bash
sudo pip install nsxramlclient pyvim pyvmomi lxml requests
```

The NSX for vSphere REST API changes with each version, so in order to use the nsxramlclient library we will need a RAML file specific to version of NSX-V we are connecting to. The RAML file also produces nice [dynamic documentation of the NSX APIs](https://htmlpreview.github.io/?https://github.com/vmware/nsxraml/blob/6.3/html-version/nsxvapi.html).

```bash
git clone https://github.com/vmware/nsxraml
cd nsxraml
git checkout 6.3
```

So now we can try and connect and get some information about anything described in the API document, like NSX Controllers.

```python
from nsxramlclient.client import NsxClient
nsx_manager = "192.168.1.18"
nsx_username = "admin"
nsx_password = "VMware1!VMware1!"
nsxraml_file = 'nsxvapi.raml'
nsx_manager_session = NsxClient(nsxraml_file, nsx_manager, nsx_username, nsx_password)
nsx_controllers = nsx_manager_session.read('nsxControllers', 'read')
nsx_controllers
```

When it comes to putting and posting information getting the formatting right can be a challenge. To this end with the library it is possible to create a template python dictionary using extract_resource_body_example.  Once we have this we can display the output structure but more usefully we can also substitute values into the template.

```python
new_ls = nsx_manager_session.extract_resource_body_example('logicalSwitches', 'create')
nsx_manager_session.view_body_dict(new_ls)
new_ls['virtualWireCreateSpec']['name'] = 'TestLogicalSwitch1'
new_ls['virtualWireCreateSpec']['description'] = 'TestLogicalSwitch1'
new_ls['virtualWireCreateSpec']['tenantId'] = 'Tenant1'
new_ls['virtualWireCreateSpec']['controlPlaneMode'] = 'UNICAST_MODE'
nsx_manager_session.view_body_dict(new_ls)
```

Once we have out body template correctly describing what we want we can post this and if all goes to plan create a new Logical Switch. In this example I am passing in the scopeId (transport zone) manually to keep it a simple example.

```python
new_ls_response = nsx_manager_session.create('logicalSwitches', uri_parameters={'scopeId': 'vdnscope-1'}, request_body_dict=new_ls)
nsx_manager_session.view_response(new_ls_response)
```

# Summary
It looks like with our new found friend the VMware Python libraries we can easily create and deploy a VMware configuration 'infrastructure as code'.