+++
title = "Deploying NSX-V With Ansible"
date = "2016-06-08"
description = "Deploying NSX-V With Ansible"
tags = [
    "nsx",
    "python",
    "ansible",
    "ovftool"
]
categories = [
    "networking",
    "automation"
]
+++
Here I will describe the steps taken to deploy NSX Manager using the [Ansible NSX Module](https://github.com/vmware/nsxansible). The NSX Ansible module is written by VMware and is provided opensource on GitHub. To work properly this depends on the [NSX RAML Specification](https://github.com/vmware/nsxraml), [NSX RAML Python Client](https://github.com/vmware/nsxramlclient), [vSphere API Python Bindings](https://github.com/vmware/pyvmomi) and the [OVF Tool](https://www.vmware.com/support/developer/ovf/) all being installed on the Ansible server.

The following assumes you have a working Ansible installation already, and a vSphere environmentto install NSX to. If you don't yet have these you can see how I performed my [Ansible Installation]({{site.url}}/how-to-setup-an-ansible-test-lab-for-windows-managed-nodes-custom-windows-modules/) in this earlier blog post.

## NSX RAML Specification
As the NSX REST API changes with each release the REST API Markup Language (RAML) specification for NSX is provided as a different branch of the GitHub repository.  In my environment I will be using NSX 6.2.2 so first I ensure the git client is installed to my Ansible server and then I use this to take a clone of the correct branch.

```bash
yum install git-all
git clone -b 6.2.2 https://github.com/vmware/nsxraml.git /nsxraml
```

## NSX RAML Python Client
The NSX RAML python client is series of functions which can be used standalone or in our case called by the Ansible NSX module.  To install these we need to ensure the 'Python Package Manger' and some other tools are installed and available on our Ansible server.

```bash
yum install python-pip gcc libxslt-devel python-devel pyOpenSSL
pip install --upgrade pip
pip install lxml
```

Once thes pre-requiste components are in place we can then install the NSX RAML python client. Similar to the RAML specification, the client functions are also dependant on version. The version which works with NSX RAML Specification 6.2.2 is Python client 1.0.4.  To install this version from Python package manager we use.

```bash
pip install nsxramlclient=1.0.4
```

If you have NSX Manager already deployed you can create a session to this using the python client.  First start python by running

```bash
python
```

then you can run the commands similar to this to create a session.

```python
from nsxramlclient.client import NsxClient
nsxraml_file = '/nsxraml/nsxvapi.raml'
nsxmanager = 'nsx.darrylcauldwell.local'
nsx_username = 'admin'
nsx_password = 'VMware!'
client_session = NsxClient(nsxraml_file, nsxmanager, nsx_username, nsx_password, debug=False)
```

you can then see the session by running the following command in python session

```python
client_session
```

![NSX RAML Client](/images/nsx-install-ansible-RAMLclient.jpg)

## VMware vSphere API Python Bindings

As well as the NSX Python client the Ansible NSX Module also depends on the VMware vSphere API Python Bindings (pyVmomi. pyVmomi is the Python SDK for the VMware vSphere API that allows you to manage ESX, ESXi, and vCenter. This is similarly installed with the python package manager using command like.

```bash
pip install pyvmomi
```

## OVF Tool
The final thing to be installed for the NSX Module to operate correctly is the VMware OVF Tool. The OVF tool for Linux version 4.1.0 is [available here](https://my.vmware.com/group/vmware/details?downloadGroup=OVFTOOL410&productId=491#), please note a VMware login is required to get this.

Once downloaded to the Ansbile server, we need to ensure it has execute attribute and then execute it to start the install, the commands to do this for the current version 4.1.0 are.

```bash
chmod +x VMware-ovftool-4.1.0-2459827-lin.x86_64.bundle
./VMware-ovftool-4.1.0-2459827-lin.x86_64.bundle
```

![OVFTool](/images/nsx-install-ansible-ovftool.jpg)

Once the install is running you have to agree to EULA, to get to the end of the text, hold down the Space bar. When prompted *Do you agree?* type yes.

![OVFTool](/images/nsx-install-ansible-ovftool2.jpg)

Once OVF Tool is installed we can use SCP to copy the NSX OVA, VMware-NSX-Manager-6.2.2-3604087.ova, to Ansible server I placed this in folder named /OVAs.

## Ansible NSX Module

We already have Git installed for dowlnloading the NSX RAML Specification so we can use this to clone the NSX Ansible repository.

```bash
git clone https://github.com/vmware/nsxansible.git /nsxansible
```

## Deploy NSX Manager

The NSX Module comes supplied with some example playbooks for performing common tasks, we’ll first take a copy of the example to deploy NSX Manager

```bash
cp /nsxansible/test_deploynsxova.yml /nsxansible/darryl_deploynsxova.yml
```

We can then edit the contents to match the variables we plan to deploy in environment. While most of playbook contents are environmental specific varibles its worth noting that we run this module against the Ansible server itself as this is where the OVA and ovftool are located so hosts: value will always be localhost.

![Deploy NSX Manager](/images/nsx-install-ansible-NSXmgr.jpg)

Once we have our environmental specific entries set we can execute the playbook with Ansible.

```bash
ansible-playbook darryl_deploynsxova.yml
```

![Deploy NSX Manager Playbook](/images/nsx-install-ansible-playbook.jpg)

We see this deploys ‘NSX Manager’ and configures the setting specified in the playbook.

## Deploy NSX Manager and register with vCenter and SSO

As with any Ansible playbook we can put common variables in a central location and call these from playbooks. An example is provided called answerfile-deployLab.yml. Variable names are overlapped between parts of the NSX Modules, in order that they are unique in the central answer file the names vary but its very easy to match these.

An example of my playbook to deploy NSX Manager and then register this with vCenter and SSO.

![Deploy and Configure NSX Manager Playbook](/images/nsx-install-ansible-mgr-cfg.jpg)
