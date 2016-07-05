---
layout: post
title:  "Handling VSAN Ineligible Disks"
date:   2015-07-16 16:59:56 +0100
tags:
    - Storage
permalink: vsan-ineligible-disks
---
If you are reusing a server and \ or disks which will become a VSAN cluster you may find some of 
your disks show as ineligible. In the Cluster,  Manage tab,  Virtual SAN,  Disk Management section.

<center><img src="/images/vsan_disk11.png" width="50%"></center>

Disks in this state cannot be used for adding to VSAN cluster, the most common reason is pre-existing 
partition data on the disks.  If you not down all of the device ids listed as ineligible with issue 
‘Existing partition found on disk…’,  then open a SSH connectoin to ESXi.

First you will need to identify which of these devices is in use as the ESXi boot partition,  to do 
this we issue a command to give information about the root volume

{% highlight bash %}
ls -l /
{% endhighlight %}

This should return assorted information but the important part for us is the path starting /vmfs/volumes/

<center><img src="/images/vsan_disk4.png" width="50%"></center>

Once we know the volume UID we can then check which device it is located on by issuing this command

{% highlight bash %}
vmkfstools -P /vmfs/volumes/<volume id>
{% endhighlight %}

This returns the device ID and partition this volume is housed.

<center><img src="/images/vsan_disk5.png" width="50%"></center>

We want to ensure that we don’t remove the partitions from this ESXi boot disk,  but for all of the 
other devices we can go ahead and check the partition table.

{% highlight bash %}
partedUtil get /dev/disks/naa.<device id>
{% endhighlight %}

On attempting to get the partition you may get an error “Error: Can’t have a partition outside 
the disk!” if this occurs create a simple gpt partition.

{% highlight bash %}
partedUtil setptbl /vmfs/devices/disks/naa.<device id> gpt
{% endhighlight %}

If this still errors with “Error: Can’t have a partition outside the disk!” then you can look to 
fix the gpt table

{% highlight bash %}
partedUtil fixgpt /vmfs/devices/disks/naa.<device id>
{% endhighlight %}

<center><img src="/images/vsan_disk6.png" width="50%"></center>

This should fix the partition which you then get and remove.

This should return a list of the partitions on the disk for us to remove.

<center><img src="/images/vsan_disk2.png" width="50%"></center>

You can then use the following command to remove the partitions,  you run this once for every partition

{% highlight bash %}
partedUtil delete /dev/disks/naa.<device id> <partition number>
{% endhighlight %}

These changes are not quick to populate into VSAN,  to encourage this to be picked up quicker refreshing 
the storage adapter

<center><img src="/images/vsan_disk3.png" width="50%"></center>

Repeat this until the only ineligible Disk is the one you use for ESXi boot device,  obviously if you 
boot from SD Card then you should show with zero ineligible Disk.