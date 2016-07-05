---
layout: post
title:  "vSphere 6, VSAN & NSX 6.2 Homelab In The Public Cloud"
date:   2016-04-16 16:59:56 +0100
tags:
    - Homelab
permalink: vsphere-6-vsan-nsx-6-2-homelab-in-the-public-cloud
---
Over the last few years the suite of products which VMware offer has grown its made it less practical to obtain 
and run enough hardware for a home learning lab. At the same time the cloud promises access to resources on demand 
and paid for as consumed. I came across Ravello who offer a unique proposition, namely running nested ESXi on
 public cloud.  They are able to achieve this as they have written an abstraction layer called HPX which offers 
 nested virtualisation engine, a software defined network and a storage overlay. The solution is pay as you use, 
 and so seems potentially the ideal way to run my <del>homelab</del> cloudlab.

<h3>Windows Management Host</h3>  
The first thing to build is a server to host all management tools and build out the environment. I chose Windows 
Server 2012 R2 as this will also host Active Directory. The high level steps I followed,
<ul>
	<li>Upload Windows Server 2012 R2 and ESXi 6.0 Update 2 ISO files to Ravello Library -&gt; Disk Images</li>
	<li>Create new Ravello Application called 'vSphere 6'</li>
	<li>Change to Canvas tab within application and add VM named 'Empty' from library</li>
	<li>Select the 'Empty' VM and its specification will be displayed in right pane and change following
<ul>
	<li>General -&gt; Name to 'ad'</li>
	<li>General  -&gt; Description to 'Active Directory'</li>
	<li>General  -&gt; Hostnames to 'ad'</li>
	<li>System -&gt; CPUs to '2'</li>
	<li>System -&gt; Mem Size to 4GB</li>
	<li>Disks -&gt; Disk 1 Sized to 60GB</li>
	<li>Disks -&gt; CD to the uploaded Windows 2012 R2 ISO</li>
	<li>Networks - Network 1 to have IP Configuration
<ul>
	<li>Static IP 172.16.20.5</li>
	<li>Netmask 255.255.255.0</li>
	<li>Gateway 172.16.20.254</li>
	<li>DNS 172.16.20.5</li>
</ul>
</li>
	<li>Services Remove ssh</li>
	<li>Services Add rdp Port 3389</li>
</ul>
</li>
	<li>Power on the VM and open the VNC console</li>
	<li>Unbind IPv6</li>
	<li>Configure IPv4 address 172.16.20.5, net mask 255.255.255.0, gateway 172.16.20.254 DNS server 172.16.20.5</li>
	<li>Give computer name as 'ad'</li>
	<li>Disable IE Enhanced Security</li>
	<li>Enable Remote Desktop connection,  find public IP from VM Summary tab and connect using mstsc</li>
	<li>Install all Windows Updates</li>
	<li>Add Active Directory, DNS, DHCP and .net Framework 3.5 using Roles and Features</li>
	<li>Install Google Chrome</li>
	<li>Install Google Chrome PostMAN Rest Client App</li>
	<li>Install putty and WinSCP</li>
	<li>Create a folder C:\ISOs</li>
	<li>Download vCenter appliance 6.0 Update 2 ISO file</li>
	<li>Create a DNS Forward lookup zone for darrylcauldwell.home</li>
	<li>Create a DNS Reverse lookup zone for 172.16.20.0</li>
	<li>Create A &amp; PTR record in DNS for 'ad' with IP '172.16.20.5'</li>
	<li>Add a new Active Directory forest named darrylcauldwell.home</li>
	<li>Install License Key and Activate Windows</li>
</ul>

<h3>ESXi Hosts</h3>
From ravello web application open the vSphere 6 application and change to canvas view. Add three new VMs from the 
'Empty ESX' library object.  For each esx VM change the following.
<ul>
	<li>General -&gt; Name to 'esx1, esx2 OR esx3'</li>
	<li>General  -&gt; Description to 'esx1, esx2 OR esx3'</li>
	<li>General  -&gt; Hostnames to 'esx1, esx2 OR esx3'</li>
	<li>System -&gt; CPUs to '4'</li>
	<li>System -&gt; Mem Size to 16GB</li>
	<li>Disks -&gt; Disk 1 Sized to 5GB (Make bootable)</li>
	<li>Disks -&gt; Disk 2 Sized to 4GB</li>
	<li>Disks -&gt; Disk 3 Sized to 2048GB</li>
	<li>Disks -&gt; CD to the uploaded ESXi ISO</li>
	<li>Networks - Network 1 named mgmt to have IP Configuration
<ul>
	<li>Static IP 172.16.20.11, 172.16.20.12 OR 172.16.20.13</li>
	<li>Netmask 255.255.255.0</li>
	<li>Gateway 172.16.20.254</li>
	<li>DNS 172.16.20.5</li>
</ul>
</li>
	<li>Networks - Network 2 named vmotion to have IP Configuration
<ul>
	<li>Static IP 172.16.21.11, 172.16.21.12 OR 172.16.21.13</li>
	<li>Netmask 255.255.255.0</li>
</ul>
</li>
	<li>Networks - Network 3 named vsan to have IP Configuration
<ul>
	<li>Static IP 172.16.22.11, 172.16.22.12 OR 172.16.22.13</li>
	<li>Netmask 255.255.255.0</li>
</ul>
</li>
	<li>Networks - Network 4 named nsx to have IP Configuration
<ul>
	<li>Static IP 172.16.23.11, 172.16.23.12 OR 172.16.23.13</li>
	<li>Netmask 255.255.255.0</li>
</ul>
</li>
</ul>
Power on all three ESX VMs and perform installation of ESX to the 5GB disk.  Using console configure IPv4 address, 
DNS to 172.16.20.5, hostname, disable IPv6, enable SSH and Shell.

<h3>Configure ESXi Hosts</h3>
All disk we have is hdd but in order to form VSAN we need to  have some SSD,  we should mark our 4GB disk with 
the enable_ssd option using the method described in <a href="https://kb.vmware.com/kb/2013188" target="_blank">
kb2013188</a>. For how my ESX detected devices the command syntax was

{% highlight cmd %} 
esxcli storage nmp satp rule add --satp=VMW_SATP_LOCAL --device mpx.vmhba2:C0:T0:L0 --option "enable_ssd"
{% endhighlight %}

In order to run VMs on these we need to add the ‘vmx.allowNested’ flag on each ESX host.  Run 

{% highlight bash %} 
vi /etc/vmware/config 
{% endhighlight %}

and add the line
{% highlight cmd %}
vmx.allowNested = "TRUE"
{% endhighlight %}

Run
{% highlight bash %}
/sbin/auto-backup.sh
{% endhighlight %}

Ignore any warnings.

<h3>VSAN Bootstrap</h3>
In order to deploy vCenter we require some storage,  as we require vCenter to form VSAN we need to form a VSAN 
bootstrap with one host so we can deploy VCSA then form VSAN between all hosts.  The method is described in this 
William Lam <a href="http://www.virtuallyghetto.com/2013/09/how-to-bootstrap-vcenter-server-onto_9.html" target="_blank">guide</a>.

<h3>vCenter Appliance</h3>
Mount the vCenter Appliance ISO file to AD server and from this install \vcsa\VMware-ClientIntegrationPlugin-6.0.0.exe 
then open vcsa-setup.html in Chrome and allow it to access the client integration plugin.
<ul>
	<li>point installer to the ESXi host you created the bootstrap VSAN on</li>
	<li>name appliance 'vcenter'</li>
	<li>use embdeed psc</li>
	<li>create new sso domain called vsphere.local</li>
	<li>ensure Tiny size is selected</li>
	<li>select vsan datastore</li>
	<li>use embdedded PostgresSQL DB</li>
	<li>network address 172.16.20.10</li>
	<li>system name vcenter.darrylcauldwell.home</li>
	<li>netmask 255.255.255.0</li>
	<li>gateway 172.16.20.254</li>
	<li>dns server 172.16.20.5</li>
	<li>ntp server 172.16.20.5</li>
	<li>enable ssh</li>
</ul>
During the deployment it powers on the VM,  due to a limitation in how Ravello presents hardware to ESX as vCenter 
boots it presents a warning and doesn't boot automatically.  As its deploying connect to C# client console and press 
any key to continue past warning,  if you don't do this the installation process fails due to timeout.

<h3>Configure vCenter Cluster</h3>
Create a Datacenter and a Cluster then add the three ESXi hosts using FQDN.

Update the networking on each ESXi host and add a vStandard Switch,  linked to vmnic1,  with IP address (172.16.21.11, 
172.16.21.12 OR 172.16.21.13) and enabled for vMotion.

Update the networking on each ESXi host and add a vStandard Switch,  linked to vmnic2,  with IP address (172.16.22.11, 
172.16.22.12 OR 172.16.22.13) and enabled for VSAN.

Enable VSAN, DRS and HA for the cluster.

<h3>NSX</h3>
Create a distributed switch, portgroup and add all three hosts using vmnic3 for uplink. Deploy NSX Manager OVA give 
name nsx and IP address 172.16.20.8. Do not check box to power on after deployment.

As our ESX hosts only have 16GB we need to reduce the configured size from 16GB to 8GB.  First we need to remove the 
memory reservation of VM object and as this is a very small environment reduce the assigned memory to 8GB.

The NSX Manager virtual machine deploys a VMXNet3 NIC this doesn't play nicely with Ravello so remove this NIC and 
add a e1000 NIC.  If you do this prior to powering on then the installation scripts use this.

Now you can power on NSX Manager.

<h3>Proxy</h3>
The Ravello networking allows internet incoming and outgoing traffic to VMs running in Ravello via NAT rules. 
This means VMs nested into vSphere don't get internet access,  as such its useful to deploy a Squid Proxy.

Download CentOS ISO and upload to Ravello library,  deploy a Empty VM and attach the ISO, configure Static Networking 
for IP 172.16.20.250.  During CentOS setup wizard,  enable network and configure IP address as in ravello console. 
Once installed open console and run

{% highlight bash %}
yum -y update
yum -y install squid
{% endhighlight %}

The default configuration works with no changes needed and offers working proxy to your nested VMs (vCenter and NSX) 
on 172.16.20.250:3128.