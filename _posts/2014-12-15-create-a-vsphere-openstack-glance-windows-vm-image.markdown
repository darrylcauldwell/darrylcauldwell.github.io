---
layout: post
title:  "Create A VMware Integrated OpenStack Glance Windows VM Image"
date:   2014-12-15 16:59:56 +0100
tags:
    - VMware Integrated OpenStack
permalink: create-a-vsphere-openstack-glance-windows-vm-image
---
While looking at OpenStack as the control plane for vSphere it appears there isn't too much detail and I 
found it tricky to create my first OpenStack image.  Here are the steps I followed.

<H3>Create vSphere Donor Windows Virtual Machine</H3>
<ul>
	<li>Use vSphere web client wizard to create new hardware v10 virtual machine in vSphere with Thin vHDD</li>
	<li>Attach Windows DVD ISO</li>
	<li>Install Windows to virtual machine</li>
	<li>Disconnect DVD ISO</li>
	<li>Install VMware Tools</li>
	<li>Complete VMware Tools Reboot</li>
	<li>Power Down VM</li>
	<li>Export VM as OVF</li>
</ul>
<H3>Create OpenStack Image</H3>
<ul>
	<li>Open OpenStack Console</li>
	<li>Change to your Project in left pane</li>
	<li>Select Manage Compute -&gt; Images &amp; Snapshots</li>
	<li>Click Create Image</li>
	<li>Complete form using this as an example,  for Image File,  open the OVF file and select the VMDK you exported earlier<center><img src="/images/OpenStack-Image-Create.png" width="50%"></center></li>
	<li>Click Create Image</li>
	<li>Wait while file uploads, it takes a while, you'll be returned to console when it completes</li>
Note this updates the file to folder glance_openstack on the datastore glance is configured to use. Once 
created you can cross ref the file UID with the OpenStack Console.
</ul>

<H3>Test OpenStack Image Works As Instance</H3>
<ul>
	<li>Open OpenStack Console</li>
	<li>Change to your Project in left pane</li>
	<li>Select Manage Compute -&gt;Instances</li>
	<li>Click Launch Instance in right pane</li>
	<li>Complete Details Form</li>
    <center><img src="/images/InstanceLaunch.png" width="50%"></center>
	<li>Ensure correct Security Group selected on Access &amp; Security Tab</li>
	<li>Ensure correct Network is selected on Networking tab</li>
	<li>Click Launch</li>
</ul>

<H3>VSAN Abnormality</H3>
If your Image is to go to a VSAN datastore and your using OpenStack Havana the above method will fail, this is because 
VSAN introduces a new disk type [streamOptimized] (http://specs.openstack.org/openstack/nova-specs/specs/kilo/approved/vmware-vsan-support.html) 
which the UI is not aware of (this is fixed in Icehouse and later).

In order to import these images you would need to use the OpenStack command line interface.  First open WebUI then  "Project -&gt; Manage Compute -&gt; Access &amp; Security" and click Download OpenStack RC File.

SCP the RC file to your Linux jump box /var/tmp and then use

{% highlight bash %}
source /var/tmp/<filename>.rc
{% endhighlight %}

Enter password when prompted.

Once you have authenticated,  run script like (substituting name and filename as required.

{% highlight bash %}
glance --insecure --os-endpoint-type internalURL image-create \
--name Windows2008R2-VM10 \
--property vmware_disktype=streamOptimized \
--property vmware_adaptertype=lsiLogic \
--container-format=bare --disk-format=vmdk \
--is-public=True \
--file=/var/tmp/Windows2008R2-VM10-disk1.vmdk</code>
{% endhighlight %}

You should get output like

<center><img src="/images/streamOptimized2.png" width="50%"></center>

If you have uploaded an image already and found that its not streamOptimized you can change the attribute.

{% highlight bash %}
glance image-update <image_name or uuid> --property vmware_disktype=streamOptimized
{% endhighlight %}