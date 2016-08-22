---
layout: post
title:  "Configuring Network Core Dump With PowerCLI"
date:   2015-06-17 16:59:56 +0100
tags:
    - Powershell
permalink: configuring-network-core-dump-powercli/
---
The VMware vSphere Network Dump Collector service enables a host to transmit diagnostic information via the 
network to a remote netdump service, which stores it on disk. Network-based coredump collection can be configured 
in addition to or instead of disk-based coredump collection. This may be useful in stateless environments with 
no local disk usable for a diagnostic partition.

vSphere ESXi Dump Collector service pre-packaged with the vSphere vCenter Server Virtual Appliance, and if 
vCenter on Windows vSphere ESXi Dump Collector typically is installed on same server.

It is a little bit of a pain to configure this on every server, so I wrote this little script to get all 
hosts and then configure each ESXi host registered with vCenter to point its core dumps to the vCenter.

{% highlight cmd %}
Add-PSSnapin VMware.VimAutomation.Core
$vcenter = ''
Connect-VIServer $vcenter
foreach($vmhost in Get-VMHost){
$esxcli = Get-EsxCli -VMHost $vmhost.Name
$esxcli.system.coredump.network.set($null,"vmk0",$vcenter,6500)
$esxcli.system.coredump.network.set(1)
$esxcli.system.coredump.network.get()
}
{% endhighlight %}
