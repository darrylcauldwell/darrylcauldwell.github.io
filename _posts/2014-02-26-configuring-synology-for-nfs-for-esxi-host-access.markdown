---
layout: post
title:  "Configuring Synology NFS For ESXi"
date:   2014-02-26 16:59:56 +0100
tags:
    - Homelab
    - Storage
permalink: configuring-synology-for-nfs-esxi-host-access/
---
The function of my NAS within homelab is to provide shared storage for my ESXi hosts, as I'm studying for VCAP it 
would be great to use both iSCSI and NFS. By default NFS is disabled on the DS213J,  first step is to enable it.

Open Control Panel -&gt; Win/Mac/NFS -&gt; Check the box Enable NFS

<center><img src="/images/SynologyNFS1.png" width="50%"></center>

Once NFS is enabled,  we can create a folder and share this out via NFS,

Open Control Panel -&gt; Shared Folder -&gt; Create

<center><img src="/images/SynologyNFS2.png" width="50%"></center>

Still within "Control Panel -&gt; Shared Folder" click the folder you created and click Privileges -&gt; NFS 
Privileges use Create Edit Delete to add the IP address of all your ESXi hosts.

<center><img src="/images/SynologyNFS3.png" width="50%"></center>

Ignore part which says you may specify a host in three ways,  the only method I found which works presently is adding each IP address for each host.  This is a bit annoying as my home network is all DHCP so while the ESXi host addresses are relatively stable they can change which means I have have to revisit this configuration.  If network segment was working then I would in theory never need to return here.

To then add this as an NFS mounted datastore use vSphere client -&gt; Inventory -&gt; Hosts and Clusters, and select 
the host you want to add this to,  then in right pane change to Configuration tab,  Storage section and click Add 
Storage button.

<center><img src="/images/SynologyNFS4.png" width="50%"></center>

Add the details of your NAS and the shared folder you just created

<center><img src="/images/SynologyNFS5.png" width="50%"></center>

Your Synology NFS datastore should now be mounted.

One of the nice things about the Synology operating system is that it allows easy transfer of files using 
any method you please.  For example if I have a VM OVF ISO on USB which I want to transfer to my homelab I 
just plug in the USB direct to the NAS and file copy to my NFS folder. Or if I want to transfer a file from 
Windows to my NFS datastore,  rather than mount the NFS volume I can use WinSCP to copy the files. When working 
in homelab especially in my portable laptop based lab where 2x ESX and 1x vCenter are bridged using VMware 
workstation to my wireless network,  network throughput for file copies is obviously very poor the flexibility 
of the Synology means I can work around this easily.