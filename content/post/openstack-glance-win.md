+++
title = "OpenStack Windows Image"
date = "2015-04-23"
description = "Create A VMware Integrated OpenStack Glance Windows VM Image"
tags = [
    "openstack",
    "vsphere",
    "vm guest customization"
]
categories = [
    "automation"
]
+++
While looking at OpenStack as the control plane for vSphere it appears there isn't too much detail and I found it tricky to create my first OpenStack image.  Here are the steps I followed.

## Create vSphere Donor Windows Virtual Machine
1. Use vSphere web client wizard to create new hardware v10 virtual machine in vSphere with Thin vHDD
2. Attach Windows DVD ISO
3. Install Windows to virtual machine
4. Disconnect DVD ISO
5. Install VMware Tools
6. Complete VMware Tools Reboot
7. Power Down VM
8. Export VM as OVF

## Create OpenStack Image

1. Open OpenStack Console
2. Change to your Project in left pane
3. Select Manage Compute, Images and Snapshots
4. Click Create Image
5. Complete form using this as an example,  for Image File,  open the OVF file and select the VMDK you exported earlier

![Openstack Image](/images/openstack-glance-win-Image-Create.png)

6. Click Create Image
7. Wait while file uploads, it takes a while, you'll be returned to console when it completes

Note: this updates the file to folder glance_openstack on the datastore glance is configured to use. Once created you can cross ref the file UID with the OpenStack Console.


## Test OpenStack Image Works As Instance

1. Open OpenStack Console
2. Change to your Project in left pane
3. Select Manage Compute Instances
4. Click Launch Instance in right pane
5. Complete Details Form

![Openstack Instance Launch](/images/openstack-glance-win-InstanceLaunch.png)

6. Ensure correct Security Group selected on Access and Security Tab
7. Ensure correct Network is selected on Networking tab
8. Click Launch

## VSAN Abnormality
If your Image is to go to a VSAN datastore and your using OpenStack Havana the above method will fail, this is because VSAN introduces a new disk type [streamOptimized] (http://specs.openstack.org/openstack/nova-specs/specs/kilo/approved/vmware-vsan-support.html) which the UI is not aware of (this is fixed in Icehouse and later).

In order to import these images you would need to use the OpenStack command line interface.  First open WebUI then  "Project -&gt; Manage Compute -&gt; Access &amp; Security" and click Download OpenStack RC File.

SCP the RC file to your Linux jump box /var/tmp and then use

```bash
source /var/tmp/<filename>.rc
```

Enter password when prompted.

Once you have authenticated,  run script like (substituting name and filename as required.

```bash
glance --insecure --os-endpoint-type internalURL image-create \
--name Windows2008R2-VM10 \
--property vmware_disktype=streamOptimized \
--property vmware_adaptertype=lsiLogic \
--container-format=bare --disk-format=vmdk \
--is-public=True \
--file=/var/tmp/Windows2008R2-VM10-disk1.vmdk</code>
```

You should get output like

![Stream Optimized](/images/openstack-glance-win-streamOptimized2.png)

If you have uploaded an image already and found that its not streamOptimized you can change the attribute.

```bash
glance image-update <image_name or uuid> --property vmware_disktype=streamOptimized
```