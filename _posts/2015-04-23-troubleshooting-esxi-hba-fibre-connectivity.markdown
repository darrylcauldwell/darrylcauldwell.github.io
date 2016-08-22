---
layout: post
title:  "Troubleshooting ESXi HBA Fibre Connectivity"
date:   2015-04-23 16:59:56 +0100
tags:
    - Storage
permalink: troubleshooting-esxi-hba-fibre-connectivity/
---
To see physical NIC connectivity in vSphere is nice and easy as its displayed in 
vCenter console.  I found however checking the link state of fibre channel HBAs 
is possible but not so easy so I thought I’d detail the steps I used here.

I had a simple issue that the storage target was not being discovered by my 
newly built ESXi 5.5 host, normally a couple of reboots or HBA resets are required 
get the array visible,  but this host wasn’t having it.  The first thing to 
establish is if the cabling engineer had connected it correctly as the server is 
many hundreds of miles away physically checking the cabling is non trivial.

I first explored the esxcli storage namespace and while from here I can verify 
that the adapters are discovered using either 

{% highlight bash %}
esxcfg-scsidevs -a
{% endhighlight %}
or
{% highlight bash %}
esxcli storage core adapter list
{% endhighlight %}

Once we are happy the adapters are discovered,  we can see if packets are going through the hba’s using 

{% highlight bash %}
esxcli storage core adapter stats get
{% endhighlight %}

Unfortunatley there is no part of *esxcli storage* namespace for checking fibre connection state.  

Up to and including vSphere 5.1 the running HBA driver populates */proc/scsi/*  with a file which 
records fibre channel link state.

For an active link:
{% highlight bash %}
Host adapter:Loop State = <READY>, flags = 0xaa68
Link speed = <8 Gbps>
{% endhighlight %}

For an inactive link:
{% highlight bash %}
Host adapter:Loop State = <DEAD>, flags = 0x1a268
Link speed = <Unknown>
{% endhighlight %}

For vSphere 5.5 and later, you do not see native drivers in the /proc nodes. To view native driver 
information, run the command: 

{% highlight bash %}
/usr/lib/vmware/vmkmgmt_keyval/vmkmgmt_keyval -a”
{% endhighlight %}

This command looks to give you some great information from the running driver including link state,  
in my case this was down and it was a cabling fault.