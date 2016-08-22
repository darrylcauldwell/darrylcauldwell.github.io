---
layout: post
title:  "NSX-V, vRealize, All-Flash VSAN Homelab"
date:   2016-05-16 16:59:56 +0100
tags:
    - Storage
    - Homelab
    - Software Defined Networking
permalink: all-flash-vsan-homelab/
---

As VMware add more products to its suite its more difficult to run a suitable homelab to host them. 
With this I aim to describe the steps I followed to create my current homelab. 

<H3>High Level Hosting Requirements</H3>
Some core products will permanently available in the homelab.

vCenter Appliance (Tiny)            2 vCPU, 8GB vRAM, 120GB vHDD  
Windows, DNS, Active Directory      2 vCPU, 4GB vRAM, 100GB vHDD  
Ansible                             1 vCPU, 2GB vRAM, 16GB vHDD  
Core Total                          5 vCPU, 14GB vRAM, 256GB vHDD  

Other products which will be transient in the environment.

NSX Manager                         4 vCPU, 16GB vRAM, 60GB vHDD  
NSX Controller                      4 vCPU, 4GB vRAM, 20GB vHDD  
NSX Total                           8 vCPU, 20GB vRAM, 80GB vHDD  

vRealize Operations (Extra Small)   2 vCPU, 8GB vRAM, 122GB vHDD  
vRealize Log Insight (Extra Small)  2 vCPU, 4GB vRAM, 144GB vHDD  
Management Total                    4 vCPU, 12GB VRAM, 266GB vHDD  

vRealize Orchestrator               2 vCPU, 4GB vRAM, 12GB vHDD  
vRealize Automation (Small)         4 vCPU, 18GB vRAM, 60GB vHDD  
vRealize Automation IaaS            2 vCPU, 8GB vRAM, 30GB vHDD  
vRA Total                           8 vCPU, 30GB vRAM, 102GB vHDD  

<h3>Physical Hosts Design</h3>
The requirements are to concurrently host VMs with 54GB of RAM and 16 vCPU and 600GB of vHDD. 
As they will have a single user (me) I expect the get away with a high over commit ratio, however 
memory will be committed by the applications as they run.

On researching what low power, small footprint, All Flash VSAN capable hosts are available I came across 
<a href="http://www.virtuallyghetto.com/2016/03/vsan-6-2-vsphere-6-0-update-2-homelab-on-6th-gen-intel-nuc.html" target="_blank">
William Lam article on 6th Generation NUC homelab</a>. These seemed to meet most of the requirement. 
As I'm potentially CPU constrained I chose the best CPU available so got two i5 Intel NUCs. At the tine 
of purchase the 1TB Samsung 850 EVO were only a little more expensive so I opted for these.

<h3>Physical Network Design</h3>
The internet connection terminates in my lounge, the lab will be run in a remote location which has no structured cabling. My internet 
connectivity is provided by UK provider BT and they supply 
<a href="https://www.shop.bt.com/products/bt-home-hub-4--bt-broadband-customers--068340-8P1M.html" target="_blank">
BT Home Hub 4</a> while being a providing great WiFi this provides 3x 10/100 Ethernet port and 1x Gigabit Ethernet 
port.

The Gigabit Ethernet port will be extended to the remote location over the power line by use of a 
<a href="http://www.tp-link.com/lk/products/details/cat-18_TL-WPA4220KIT.html" target="_blank">
TP-LINK AV500 Wi-Fi Powerline Extender</a>. While this will provide a 500MB connection it only has 2x 10/100 
Ethernet Ports on the remote end. The North-South traffic to internet will therefore be limited to 100MB,  
as my connection only runs at 20MB I don't foresee this as an issue.

Most traffic will be East-West within the lab and 
<a href="http://www.cisco.com/c/en/us/products/collateral/switches/small-business-200-series-smart-switches/data_sheet_c78-634369.html" target="_blank">
Cisco SG200-08</a> switch will be attached to the AV500 to provide Gigabit Ethernet between hosts and 
network attached storage.

The network switch provides 8x 1GbE ports one of which uplinks to internet, one  will connect the NAS. 
The other six ports will be presented evently to the two ESXi hosts by way of the two onboard NICs and 
four StarTech USB3 to GbE adapters.

<h3>Storage Design</h3>
The two ESXi hosts will host a All-Flash VSAN of 1.8TB, this will have deduplication and compression enabled.

For a few years now I've had a Synology DS213j NAS with 2x 500GB HDD this has a single 1GbE NIC and 
can present iSCSI and/or NFS.

<h3>Bill Of Materials</h3>
2 x Intel NUC 6th Gen NUC6i5SYH (<a href="http://www.ebuyer.com/product/729217" target="_blank">eBuyer unit cost £357.99</a>)  
2 x Crucial 32GB Kit (2x16GB) DDR4 (<a href="http://www.ebuyer.com/product/727316" target="_blank">eBuyer unit cost £85.98</a>)  
2 x Samsung SM951 NVMe 128GB M.2 (<a href="http://www.ebuyer.com/743528-samsung-pm951-128gb-m-2-nvme-pcie-ssd-mzvlv128hcgr-00000" target="_blank">eBuyer unit cost £42.98</a>)  
2 x Samsung 850 EVO 1TB SATA3 (<a href="http://www.ebuyer.com/product/695894" target="_blank">eBuyer unit cost £247.98</a>)  
2 x Toshiba microSDHC UHS-I 8GB Card (<a href="http://www.ebuyer.com/product/727736" target="_blank">eBuyer unit cost £2.39</a>)  
4 x StarTech USB 3.0 to Gigabit Ethernet (<a href="http://www.ebuyer.com/product/483489" target="_blank">eBuyer unit cost £16.98</a>)  
1 x Cisco SG200-08 8-port Gigabit (<a href="http://www.ebuyer.com/product/264288" target="_blank">eBuyer unit cost £72.96</a>)  
8 x 0.5m Cat6 Cables (<a href="http://www.ebuyer.com/product/130640" target="_blank">eBuyer unit cost £0.82</a>)  

<h3>Management IP Address Scheme</h3>
The switch I purchased supports VLANs but does not include a routing capability this will be done where required with
NSX Edge devices. All core virtual machines will sit on the normal home network so they can NAT out to internet. The IP 
in use on my homework is 192.168.1.0/24, the gateway is 192.168.1.254 and 192.168.1.64 - 192.168.1.253 are in use for DHCP. 
I will use the 192.168.1.10 - 192.168.1.50 for static IPv4 lab out of band management addressing.

<h3>Core Network Configuration</h3>
The TP-Link AV500 extended installation was simple case of connecting a cable at each end and pressing a button to 
mirror the WiFi settings to improve connectivity in the remote location.

The Cisco SG200-08 comes set with IP address 192.168.1.254/24 which would clash with the BT Home Hub. The first 
task is to connect the switch directly to a PC with Ethernet cable,  configure the Ethernet port with a temporary 
IP on the 192.168.1.0/24 network.  Using a web browser connect to 192.168.1.254 and connect using username: cisco 
and password: cisco. I then updated this to static IP address 192.168.1.10/24 with gateway 192.168.1.254.

<center><img src="/images/Cisco.jpg" width="50%"></center>

Once the switch has correct IP address we can connect this to the AV500 I connected this to port #8 from a WiFi client.  We can test connectivity and do something like changing to SNTP and enabling the three SNTP servers.

<center><img src="/images/Cisco-SNTP.jpg" width="50%"></center>

Connect the NAS to port #7.

Ports #1 - #6 will be used for ESXi host connectivity, VSAN and NSX require so we need to increase the MTU size for these ports to there maximum size of 9216.

Save the running configuration to be the startup configuration before exiting or it will be lost when the switch restarts.

<center><img src="/images/Cisco-Save.jpg" width="50%"></center>

<h3>ESXi Host Installation</h3>
I followed the 
<a href="http://www.virtuallyghetto.com/2016/03/vsan-6-2-vsphere-6-0-update-2-homelab-on-6th-gen-intel-nuc.html" target="_blank">
Installation section of Willam Lam's guide</a> which worked well. 
<a href="http://www.virtuallyghetto.com/2016/03/functional-usb-3-0-ethernet-adapter-nic-driver-for-esxi-5-5-6-0.html">
William had also found StarTech USB3 1GbE NICs</a> could be added to 6th Generation NUCs. Following his guide and 
supplied driver and these should get detected.

Once installed first task is to configure IPv4 addressing we will be using 192.168.1.11 and 192.168.1.12.  Set host 
names to esx1.darrylcauldwell.local and esx2.darrylcauldwell.local and while we have not setup DNS yet set primary 
DNS to the IP which will be Active Directory DNS 192.168.1.14.  This lab will not use IPv6 so at this stage I disable 
this too.

You should now be able to use a browser to see the ESXi Embedeed Host Client  
  https://192.168.1.11/ui/  
  https://192.168.1.12/ui/  

<center><img src="/images/ESXi-Web.jpg" width="50%"></center>

<H3>Single Node VSAN</h3>
Will create VSAN on single node to deploy DNS and vCenter,  using a method based on this 
<a href="http://www.virtuallyghetto.com/2013/09/how-to-bootstrap-vcenter-server-onto_9.html">William Lam article</a>.

The configuration of VSAN without vCenter is done via the command line on ESX host. As such 
first task is to enable SSH and the console.

<center><img src="/images/SSH-Console.png" width="50%"></center>

In order to run a single VSAN node we need to update the default VSAN storage policy to allow us to force VM provisioning and FTT,

{% highlight cmd %}
esxcli vsan policy setdefault -c vdisk -p "((\"hostFailuresToTolerate\" i1) (\"forceProvisioning\" i1))"
esxcli vsan policy setdefault -c vmnamespace -p "((\"hostFailuresToTolerate\" i1) (\"forceProvisioning\" i1))"
{% endhighlight %}


So at this stage we can check if VSAN will identify the M.2 storage for cache tier and SSD for capacity by running the following and checking the IsCapacityFlash attribute

{% highlight bash %}
vdq -q
{% endhighlight %}

We will find that it is not marked correctly and also find the device name so we can use this output to to configure this attribute by running a command similar to

{% highlight bash %}
esxcli vsan storage tag add -d t10.ATA_____Samsung_SSD_850_EVO_1TB_________________S2RFNXAH317049Z_____ -t capacityFlash
{% endhighlight %}

We can then check the attribute is updated correctly by running,
{% highlight bash %}
vdq -q
{% endhighlight %}

<center><img src="/images/VDQ.jpg" width="50%"></center>

We can then add both disks to the create a disk group,  by running a command similar to below substituting the disk name with output of vdq -q command,
{% highlight bash %}
esxcli vsan storage add -s t10.ATA_____SAMSUNG_MZHPV128HDGM2D00000______________S1X3NYAH201722______ -d t10.ATA_____Samsung_SSD_850_EVO_1TB_________________S2RFNXAH317049Z_____
{% endhighlight %}

We can then look to create the VSAN cluster by running,
{% highlight bash %}
esxcli vsan cluster new
{% endhighlight %}

This should now provision a VSAN datastore on single node to be used to deploy first VMs.

<h3>Active Directory and DNS</h3>
In order to deploy vCenter 6 we require DNS to be in place,  this will be hosted on a Windows 2012 Server.
<ul>
 	<li>Create a folder called ISOs on VSAN datastore</li>
 	<li>Upload Windows Server 2012 R2 ISO files to ISOs folder on VSAN datastore</li>
 	<li>Create new VM named AD with hardware config 2x vCPU, 4GB</li>
 	<li>vRAM, 1x 60GB vHDD, attach Windows ISO as vCD-ROM</li>
 	<li>Apply MSDN License Key and Active Windows</li>
 	<li>Disable IE Enhanced Security</li>
 	<li>Use Windows Update to apply all current patches</li>
 	<li>As some Updates will update .net we should <a href="https://support.microsoft.com/en-us/kb/2570538">force the Assemblies to get updated</a></li>
</ul>

{% highlight cmd %}
%windir%\Microsoft.NET\Framework\v4.0.30319\ngen.exe update /force
%windir%\Microsoft.NET\Framework64\v4.0.30319\ngen.exe update /force
{% endhighlight %}

<ul>
 	<li>Update Windows computername to 'ad'</li>
 	<li>Configure IPv4 address 192.168.1.14, net mask 255.255.255.0, gateway 192.168.1.254 DNS server 192.168.1.14</li>
 	<li>Add Active Directory, DNS, Desktop Experience and .net Framework 3.5 using Roles and Features</li>
 	<li>Create a DNS Forward lookup zone for darrylcauldwell.local</li>
 	<li>Create a DNS Reverse lookup zone for 192.168.1.0</li>
 	<li>Create A &amp; PTR record in DNS for ‘ad’ with IP ‘192.168.1.14’</li>
 	<li>Add a new Active Directory forest named darrylcauldwell.local</li>
</ul>
As well as hosting AD and DNS this will be used as a hopbox at least initially we will also perform the following 
extra steps.
<ul>
 	<li>Allow Remote Desktop access</li>
 	<li>Install Google Chrome</li>
 	<li>Install Google Chrome PostMAN Rest Client App</li>
 	<li>Install putty and WinSCP</li>
 	<li>Add Port Forward record on the BT HomeHub Router</li>
</ul>
<h3>vCenter</h3>
Follow the <a href="https://pubs.vmware.com/vsphere-60/topic/com.vmware.vsphere.install.doc/GUID-F06BA415-66D8-42CD-9151-701BBBCE8D65.html">VMware guide</a> for installing a Tiny VCSA with Embedded PSC to the newly formed VSAN datastore, give IP address 192.168.1.13 and hostname vcenter.darrylcauldwell.local.

Create a Datacenter and add the ESXi hosts. Enable vMotion and VSAN traffic on vmk0. Add VSAN Enterprise, 
vCenter, NSX and ESXi license keys.

<h3>VSAN Cluster</h3>
Create a cluster in the Datacenter holding the two physical hosts. Enable VSAN .  Move the two esx hosts into 
the cluster.  Edit VSAN configuration and enable Deduplication and Compression.

For some reason the VSAN cluster configuration from the single node VSAN cluster is not picked up by vCenter. 
So we need to manually set this again via the GUI to show FTT=0 and to force provisioning.

<center><img src="/images/VSAN-Policy.jpg" width="50%"></center>

<h3>vRealize Log Insight</h3>
As ESX is installed to USB rather than physical disk the log files are not persistent, in home lab I'll be 
trying things which will cause errors so retaining the log files would be useful. As such I add Log Insight 
3.3 OVF at this point, it comes with vSphere content pack included so I just configure this to the correct 
FQDN, I'll be shortly adding NSX so add this plugin at this point.

In order to view the network switch logs as part of troubleshooting I also configure the Log Insight IP 
address as a remote syslog receiver.

<h3>vRealize Operations Manager</h3>
While this won't be production its useful to see what information is recorded in vROPs so I deployed 6.2.1 
OVF and then configured the Log Insight integration.

<H3>NSX for vSphere</H3>
Create new VDS with both hosts added, assign both USB3 NICs as uplinks for each host.  Update MTU size to 
9000 and enable LLDP to both listen and advertise.

Deploy NSX Manager and give it an IP address on out of band management network (192.168.1.17).  Register 
with vCenter, and update remote syslog to point to Log Insight.

Create a NSX Controller IP Pool with 192.168.1.20-192.168.2.25.

<center><img src="/images/ControllerIPpool.jpg" width="50%"></center>

Create a VTEP IP Pool with 192.168.1.26-192.168.2.35.

<center><img src="/images/VTEPs.jpg" width="50%"></center>

Use Host Preparation to install the VIBs to the ESX hosts in the cluster. Configure VXLAN to use VTEPs IP Pool.

<center><img src="/images/VXLAN.jpg" width="50%"></center>

As this is a lab only and we don't need high availability deploy a single NSX Controller to our cluster.

<center><img src="/images/nsxController.jpg" width="50%"></center>

Add segment IDPool 5001-6000

Create a Transport Zone for the cluster.

<center><img src="/images/Transport.jpg" width="50%"></center>