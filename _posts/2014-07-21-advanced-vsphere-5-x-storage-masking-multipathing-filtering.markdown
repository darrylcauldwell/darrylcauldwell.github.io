---
layout: post
title:  "Advanced vSphere 5.x Storage - Masking, Multipathing & Filtering"
date:   2014-07-21 16:59:56 +0100
tags:
    - Storage
permalink: advanced-vsphere-5-x-storage-masking-multipathing-filtering/
---
The relationship between a <a href="http://www.vmware.com/products/esxi-and-esx/?src=vmw_so_vex_dcaul_99" target="_blank">vSphere ESXi</a> host and its shared storage is very important for the solution to work effectively. The storage architecture within <a href="http://www.vmware.com/products/vsphere/?src=vmw_so_vex_dcaul_99" target="_blank">vSphere </a>is extensible and this extensibility is known as the <a href="http://kb.vmware.com/kb/1011375/?src=vmw_so_vex_dcaul_99" target="_blank">Pluggable Storage Architecture</a> often abbreviated to <a href="http://kb.vmware.com/kb/1011375/?src=vmw_so_vex_dcaul_99" target="_blank">PSA</a>.

The <a href="http://kb.vmware.com/kb/1011375/?src=vmw_so_vex_dcaul_99" target="_blank">PSA </a>is an open modular framework that coordinates the simultaneous operation of multiple multi pathing plugins (MPPs). <a href="http://kb.vmware.com/kb/1011375/?src=vmw_so_vex_dcaul_99" target="_blank">PSA </a>is a collection of VMkernel APIs that allow third party hardware vendors to insert code directly into the <a href="http://www.vmware.com/products/esxi-and-esx/?src=vmw_so_vex_dcaul_99" target="_blank">ESX </a>storage I/O path. This allows 3rd party software developers to design their own load balancing techniques and failover mechanisms for particular storage array. The <a href="http://kb.vmware.com/kb/1011375/?src=vmw_so_vex_dcaul_99" target="_blank">PSA </a>coordinates the operation of the <a href="http://kb.vmware.com/kb/1011375/?src=vmw_so_vex_dcaul_99" target="_blank">NMP </a>and any additional 3rd party MPP.

<center><img src="/images/1.3.3.jpg" width="50%"></center>

To view what <a href="http://kb.vmware.com/kb/1011375/?src=vmw_so_vex_dcaul_99" target="_blank">PSA</a> plugins are loaded,
{% highlight cmd %}
esxcli storage core plugin list
{% endhighlight %}

<center><img src="/images/default_plugins.gif" width="50%"></center>

By default only <a href="http://kb.vmware.com/kb/1011375/?src=vmw_so_vex_dcaul_99" target="_blank">NMP </a>and <a href="http://kb.vmware.com/kb/1009449/?src=vmw_so_vex_dcaul_99" target="_blank">MASK_PATH</a> plugins are installed. To install a third-party PSA plugin your vendor would provide you with a vSphere Installation Bundle <a href="http://blogs.vmware.com/vsphere/2011/09/whats-in-a-vib.html" target="_blank">vib </a>file.  An example of how to install a VIB we can see for the <a href="https://library.netapp.com/ecmdocs/ECMP1237939/html/html/GUID-735E5961-E3FB-4105-A8F8-37F6444B68BC.html" target="_blank">Netapp NFS VAAI plugin</a>.

{% highlight cmd %}
esxcli software vib install -n NetAppNasPlugin -d file:///NetAppNasPlugin.v20.zip
Message: The update completed successfully, but the system needs to be rebooted for the changes to be effective.
Reboot Required: true VIBs Installed:
NetApp_bootbank_NetAppNasPlugin_1.0-020
VIBs Removed:
VIBs Skipped:
{% endhighlight %}

As there are multiple plugins supplied and others can be plugged in there is a rule list which defines which device gets managed by which plugin. You can view existing claim rules.

{% highlight cmd %}
esxcli storage core claimrule list
{% endhighlight %}

<center><img src="/images/1.3.4.jpg" width="50%"></center>

The rules are applied by rule number lowest to high,  so if you have a complex claimrule set and your devices are picking up incorrectly check the order. You will see a catch all at the bottom 65535 which assigns any without specific match to <a href="http://kb.vmware.com/kb/1011375/?src=vmw_so_vex_dcaul_99" target="_blank">NMP </a>.

One MP rule you will notice is listed twice with Class as being file and runtime.  The file which these rules apply is

{% highlight cmd %}
/etc/vmware/esx.conf
{% endhighlight %}

<center><img src="/images/esx.gif" width="50%"></center>

Using this we can if we wish we can tell PSA to detect devices and apply the <a href="http://kb.vmware.com/kb/1009449/?src=vmw_so_vex_dcaul_99" target="_blank">MASK_PATH</a> plugin. You might want to consider doing this at the ESX layer if some issue with the storage which causes host to loose contact with it and the LUN (or LUNs) enter an <a href="http://kb.vmware.com/kb/1016626/?src=vmw_so_vex_dcaul_99" target="_blank">all-paths-down condition</a> by temporarily masking the path while the storage engineer fixes the fabric.

Obtain ESXi Device ID of the LUN you would like to mask

{% highlight cmd %}
esxcli storage core device list
{% endhighlight %}

<center><img src="/images/device-id.gif" width="50%"></center>

Once you have the device ID you can then obtain :C Channel :T Target :L LUN and vmhba of the device you want to mask

{% highlight cmd %}
esxcfg-mpath -b -d {device-id}
{% endhighlight %}

<center><img src="/images/CTL.gif" width="50%"></center>

Paths can be viewed and changed using this namespace use this to find a claim rule number not in use

{% highlight cmd %}
esxcli storage core claimrule
{% endhighlight %}

Using all the above information assign the device to <a href="http://kb.vmware.com/kb/1009449/?src=vmw_so_vex_dcaul_99" target="_blank">MASK_PATH</a>

{% highlight cmd %}
esxcli storage core claimrule add -r 500 -t location -A vmhba35 -C &lt;x&gt; -T &lt;y&gt; -L &lt;z&gt; -P MASK_PATH
{% endhighlight %}

Once rule is added you should be able to see it in the claimrule list but you will notice this is file state not runtime
{% highlight cmd %}
esxcli storage core claimrule list
{% endhighlight %}

<center><img src="/images/list-file.gif" width="50%"></center>

You now need to load the file into rules and then run the new ruleset

{% highlight cmd %}
esxcli storage core claimrule load
esxcli storage core claimrule run
{% endhighlight %}

Once loaded and in running configuration you should see

{% highlight cmd %}
esxcli storage core claimrule list
{% endhighlight %}

<center><img src="/images/load-and-run.gif" width="50%"></center>

Even though the new rule is in place for new devices to pickup,  the current device still has an active claimrule,  you can either reboot,  or run a reclaim on the LUN so it picks up its new rule

{% highlight cmd %}
esxcli storage core claiming reclaim -d {device id}
{% endhighlight %}

<center><img src="/images/unclaim.gif" width="50%"></center>

If your in lab and want to get your LUN back

{% highlight cmd %}
esxcli storage core claimrule remove -r 500
esxcli storage core claimrule load
esxcli storage core claiming unclaim -t location -A vmhba33 -C 0 -T 0 -L 0
esxcli storage core adapter rescan -A vmhba33
{% endhighlight %}

You may want to mask out a whole vendor
{% highlight cmd %}
esxcli storage core claimrule add -r 501 -t vendor -V SYNOLOGY -P MASK_PATH
esxcli storage core claimrule load
esxcli storage core claimrule run
esxcli storage core claiming reclaim -d &lt;device id of device&gt;
{% endhighlight %}

To undo mask a whole vendor

{% highlight cmd %}
esxcli storage core claimrule remove -r 501
esxcli storage core claimrule load
esxcli storage core claiming unclaim -t vendor -v SYNOLOGY
esxcli storage core claiming unclaim -t location -A vmhba33
esxcli storage core adapter rescan -A vmhba33
{% endhighlight %}

Once you have correct plugin's installed,  and claimrules in place so correct devices get picked up by correct plugin,  you might want to then configure the plugin to say how it should react for path failover and how to choose correct path.

Within the storage plugin are two submodules
<ul>
	<li>Storage Array Type Plugin (SATP) – Used for path failover</li>
	<li>Path Selection Plugin (PSP) – Used for selecting the path</li>
</ul>
An SATP plugin monitors physical path health, reports to the NMP changes in physical paths, and executes array-specific actions for activating and deactivating paths. The <a href="http://kb.vmware.com/kb/1011340/?src=vmw_so_vex_dcaul_99" target="_blank">Path Selection Plug-ins</a> (PSP) selects the path for I/O requests.  The flow of how these two components operate is nicely shown in this short video.

[youtube=http://www.youtube.com/watch?v=cEp69JYEx30]

The pathing policies which can be used with VMware <a href="http://kb.vmware.com/kb/1011375/?src=vmw_so_vex_dcaul_99" target="_blank">NMP </a>can be managed via the namespace

{% highlight cmd %}
esxcli storage nmp psp
{% endhighlight %}

You can list all available PSP's using

{% highlight cmd %}
esxcli storage nmp psp list
{% endhighlight %}

<center><img src="/images/psp-list.gif" width="50%"></center>

*VMW_PSP_MRU* – Most Recently Used, on path failure recovery stays load stays on same path  
*MW_PSP_RR* – Round Robin, rotates through available paths  
*VMW_PSP_FIXED* – Fixed Pathing, on path failure recovery load moves back to preferred path  

The SATP level path selection can be viewed

{% highlight cmd %}
esxcli storage nmp satp list
{% endhighlight %}

<center><img src="/images/satp-list.gif" width="50%"></center>

The final thing to cover with regards how storage can be seen and managed within ESXi would be device filtering. 
There are four storage filters all of which are applied by default,  these filters define what can be seen 
within the cli and gui.
<ul>
	<li>VMFS Filter: filters out storage devices or LUNs that are already used by a VMFS datastore</li>
	<li>RDM Filter: filters out LUNs that are already mapped as a RDMSame Host and</li>
	<li>Transports Filter: filters out LUNs that can’t be used as a VMFS datastore extend.
<ul>
	<li>Prevents you from adding LUNs as an extent not exposed to all hosts that share the original VMFS datastore.</li>
	<li>Prevents you from adding LUNs as an extent that use a storage type different from the original VMFS datastore</li>
</ul>
</li>
	<li>Host Rescan Filter: Automatically rescans and updates VMFS datastores after you perform datastore management operations</li>
</ul>
vSphere Client -> Administration -> vCenter Server -> Settings -> Advanced Settings

To disable add the following key(s) if not already there and set it to false.

{% highlight cmd %}
config.vpxd.filter.vmfsFilter (VMFS Filter)
config.vpxd.filter.rdmFilter (RDM Filter)
config.vpxd.filter.SameHostAndTransportsFilter (Same Host and Transports Filter)
config.vpxd.filter.hostRescanFilter (Host Rescan Filter)
{% endhighlight %}