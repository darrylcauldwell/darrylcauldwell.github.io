---
layout: post
title:  "Install NetApp VAAI For NFX On vSphere 5.5"
date:   2014-02-26 16:59:56 +0100
tags:
    - Storage
permalink: install-netapp-vaai-for-nfs-on-vsphere-5-5
---

Data ONTAP 8.0.1 and later supports certain VMware vStorage APIs for Array Integration (VAAI) 
features when the ESX host is running ESX 4.1 or later. These features help offload operations 
from the ESX host to the storage system and increase the network throughput. The ESX host 
enables the features automatically in the correct environment. You can determine the extent 
to which your system is using the VAAI features by checking the statistics contained in the 
VAAI counters.

The VAAI feature set consists of the following:

<h3>Extended copy</h3>
This feature enables the host to initiate the transfer of data between the source and destination 
without involving the host in the data transfer. This results in saving ESX CPU cycles and increasing 
the network throughput. The extended copy feature is used in scenarios such as cloning a virtual 
machine. When invoked by the ESX host, the extended copy feature copies the data within the Data 
ONTAP storage system rather than going through the host network. 

Copy offload transfers data in the following ways:
<h4>Within a LUN</h4>
Between LUNs within a volume. If this feature cannot be invoked, the ESX host automatically uses 
the standard ESX copy operation.
<h4>Write Same</h4>
This feature offloads the work of writing a repeated pattern, such as all zeros, to a storage 
array. The ESX host uses this feature in  scenarios such as zero-filling a file.

<h3>Verify and Write</h3>
This feature bypasses certain file access concurrency limitations, which speeds up operations 
such as booting up a virtual machine.

<H3>Pre-requisits</H3>
NFSv3 must be enabled on the storage system.
VMware vSphere 5.0 or later must be available.
Data ONTAP 8.0.1 and later supports

<H3>Check current VAAI state</H3>
To check the current state of VAAI on block storage systems we can run

{% highlight bash %}
esxcli storage core device vaai status get
{% endhighlight %}

and on NFS storage systems

{% highlight bash %}
esxcli storage nfs list
{% endhighlight %}

<H3>If you have NetApp VSC installed</H3>
Download the offline bundle in .zip format and then extract the VMware Installation Bundle *.vib*. 
Copy the plug-in .vib file, NetAppNasPlugin.vib, to the VSC installation directory. The path name is 
VSC_installation_directory\etc\vsc\web, where VSC_installation_directory is the directory where you 
installed VSC. If using the default installation location, the path is 
C:\Program Files\NetApp\Virtual Storage Console\etc\vsc\web\NetAppNasPlugin.vib. 

Check that the file name of the .vib file that you downloaded matches the predefined name that VSC 
uses: NetAppNasPlugin.vib. If the file name does not match the predefined name, rename the .vib file. 
You do not need to restart the VSC client or the NetApp vSphere Plug-in Framework (NVPF) service.

Go to the NFS Plug-in for VMware VAAI section of the Tools panel of Monitoring and Host Configuration, 
and then click Install to Host this can be done to all hosts in one go,  however to take effect each 
must be and host must be rebooted.

Once the reboot complete confirm the storage is now enabled for VAAI using the method described above.

<H3>If you do not have NetApp VSC installed</H3>
If you do not have NetApp VSC installed or it is not working,  you can still install the vib.  To do 
this upload a copy of the extracted file NetAppNasPlugin.vib to a datastore which host has access to.

Open secure shell session with the host using PuTTy or similar,  confirm you can see file on the datastore

{% highlight bash %}
ls /vmfs/volumes/<datastore name>/NetAppNasPlugin.vib
{% endhighlight %}

Once you have confirmed file can be seen,  you can run

{% highlight bash %}
esxcli software vib install -v /vmfs/volumes/<datastore name>/NetAppNasPlugin.vib
{% endhighlight %}

Once installed reboot host, and once the reboot complete confirm the storage is now enabled for 
VAAI using the method described above.