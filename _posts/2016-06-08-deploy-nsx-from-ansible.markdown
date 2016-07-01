---
layout: post
title:  "Deploying NSX With Ansible"
date:   2016-06-08 16:59:56 +0100
tags:
    - VMware
    - Ansible
    - Software Defined Networking
permalink: deploy-nsx-from-ansible
---
Here I will describe the steps taken to deploy NSX Manager using the 
[Ansible NSX Module](https://github.com/vmware/nsxansible). The NSX Ansible module is written by 
VMware and is provided opensource on GitHub. To work properly this depends on the 
[NSX RAML Specification](https://github.com/vmware/nsxraml), 
[NSX RAML Python Client](https://github.com/vmware/nsxramlclient), 
[vSphere API Python Bindings](https://github.com/vmware/pyvmomi) and the 
[OVF Tool](https://www.vmware.com/support/developer/ovf/) all being installed on the Ansible server.

The following assumes you have a working Ansible installation already, and a vSphere environment
to install NSX to. If you don't yet have these you can see how I performed my 
[Ansible Installation]({{site.url}}/how-to-setup-an-ansible-test-lab-for-windows-managed-nodes-custom-windows-modules/)
 in this earlier blog post.

<H3>NSX RAML Specification</H3>
As the NSX REST API changes with each release the REST API Markup Language (RAML) specification for NSX 
is provided as a different branch of the GitHub repository.  In my environment I will be using NSX 6.2.2
so first I ensure the git client is installed to my Ansible server and then I use this to take a clone of the 
correct branch.

{% highlight bash %}
yum install git-all
git clone -b 6.2.2 https://github.com/vmware/nsxraml.git /nsxraml
{% endhighlight %}

<H3>NSX RAML Python Client</H3>
The NSX RAML python client is series of functions which can be used standalone or in our case called by 
the Ansible NSX module.  To install these we need to ensure the 'Python Package Manger' and some other tools
are installed and available on our Ansible server.

{% highlight bash %}
yum install python-pip gcc libxslt-devel python-devel pyOpenSSL
pip install --upgrade pip
pip install lxml
{% endhighlight %}

Once thes pre-requiste components are in place we can then install the NSX RAML python client. Similar to the RAML 
specification, the client functions are also dependant on version. The version which works with NSX RAML Specification 
6.2.2 is Python client 1.0.4.  To install this version from Python package manager we use.

{% highlight bash %}
pip install nsxramlclient=1.0.4
{% endhighlight %}

If you have NSX Manager already deployed you can create a session to this using the python client.  First start python
by running

{% highlight bash %}
python
{% endhighlight %}

then you can run the commands similar to this to create a session.

{% highlight python %}
from nsxramlclient.client import NsxClient
nsxraml_file = '/nsxraml/nsxvapi.raml'
nsxmanager = 'nsx.darrylcauldwell.local'
nsx_username = 'admin'
nsx_password = 'VMware!'
client_session = NsxClient(nsxraml_file, nsxmanager, nsx_username, nsx_password, debug=False)
{% endhighlight %}

you can then see the session by running the following command in python session
{% highlight python %}
client_session
{% endhighlight %}

<H3>VMware vSphere API Python Bindings</H3>
As well as the NSX Python client the Ansible NSX Module also depends on the VMware vSphere API Python Bindings 
(pyVmomi. pyVmomi is the Python SDK for the VMware vSphere API that allows you to manage ESX, ESXi, and vCenter. 
This is similarly installed with the python package manager using command like.

{% highlight bash %}
pip install pyvmomi
{% endhighlight %}

<H3>OVF Tool</H3>
The final thing to be installed for the NSX Module to operate correctly is the VMware OVF Tool. 
The OVF tool for Linux version 4.1.0 is 
[available here](https://my.vmware.com/group/vmware/details?downloadGroup=OVFTOOL410&productId=491#), 
please note a VMware login is required to get this.

Once downloaded to the Ansbile server, we need to ensure it has execute attribute and then execute it to
start the install, the commands to do this for the current version 4.1.0 are.

{% highlight bash %}
chmod +x VMware-ovftool-4.1.0-2459827-lin.x86_64.bundle
./VMware-ovftool-4.1.0-2459827-lin.x86_64.bundle
{% endhighlight %}

Once the install is running you have to agree to EULA, to get to the end of the text, hold down the Space bar.
When prompted *Do you agree?* type yes.

Once OVF Tool is installed we can use SCP to copy the NSX OVA, VMware-NSX-Manager-6.2.2-3604087.ova, to Ansible 
server I placed this in folder named /OVAs.

<H3>Ansible NSX Module</H3>
We already have Git installed for dowlnloading the NSX RAML Specification so we can use this to clone the NSX 
Ansible repository.

{% highlight bash %}
git clone https://github.com/vmware/nsxansible.git /nsxansible
{% endhighlight %}

<H3>Deploy NSX Manager</H3>

The NSX Module comes supplied with some example playbooks for performing common tasks, we’ll first take a copy 
of the example to deploy NSX Manager

{% highlight bash %}
cp /nsxansible/test_deploynsxova.yml /nsxansible/darryl_deploynsxova.yml
{% endhighlight %}

We can then edit the contents to match the variables we plan to deploy in environment. While most of playbook 
contents are environmental specific varibles its worth noting that we run this module against the Ansible server 
itself as this is where the OVA and ovftool are located so hosts: value will always be localhost.

Once we have our environmental specific entries set we can execute the playbook with Ansible.

{% highlight bash %}
ansible-playbook darryl_deploynsxova.yml
{% endhighlight %}

We see this deploys ‘NSX Manager’ and configures the setting specified in the playbook.