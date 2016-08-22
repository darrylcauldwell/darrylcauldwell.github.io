---
layout: post
title:  "Create VMware Virtual Machine Templates"
date:   2016-05-17 16:59:56 +0100
tags:
    - VMware
permalink: create-vmware-virtual-machine-templates/
---
When creating virtual machine templates I'd like to do this consistently as such I'll try and keep this post 
up to date with the settings I am including.

<h3>CentOS 7</h3>
New custom VM named CentOS7, hardware version 11, guest operating system Linux version 'CentOS 4/5/6/7 (64-bit)'. 
We'd like a small template which we expand if required, so single virtual socket with single core, 2GB virtual 
memory, one VMXNET3 NIC, default LSI Logic Parallel SCSI controller with a thin 16GB hard disk and mount CentOS7 
ISO as CD-ROM.

Power on VM and open console, ensure you change NIC to connected and enter password otherwise leave all as default 
and install. Now I install common tools to use within lab, so once installed reboot and logon still using remote 
console.

{% highlight cmd %}
yum install open-vm-tools net-tools epel-release gcc git
yum update
cp -f /etc/sysconfig/network-scripts/ifcfg-eth0 /tmp/eth0
sed "/^HWADDR/d" /tmp/eth0 &gt; /etc/sysconfig/network-scripts/ifcfg-eth0
{% endhighlight %}

Shutdown the virtual machine and convert to template using vCenter.

<H3>Windows 2012 R2</H3>
New custom VM named Win2012R2, hardware version 11, guest operating system Windows version 'Microsoft Windows Server 2012 (64-bit)'. We'd like a small template which we expand if required, so single virtual socket with single core, 4GB virtual memory, one VMXNET3 NIC, default LSI Logic SAS SCSI controller with a thin 40GB hard disk and mount Windows 2012 R2 ISO as CD-ROM.

Power on VM and open console, select Datacenter Edition with GUI and enter password otherwise leave all as default and install. Once installed rebooted logon and perform Typical VMware tools install.
<ul>
 	<li>Apply MSDN License Key and Active Windows</li>
 	<li>Disable IE Enhanced Security</li>
 	<li>Use Windows Update to apply all current patches, repeat until all dependant patches are installed.</li>
 	<li>Copy \Sources\SxS folder from CD-ROM to C:\</li>
 	<li>As some Updates will update .net we should force the Assemblies to get updated</li>
</ul>
{% highlight cmd %}
%windir%\Microsoft.NET\Framework\v4.0.30319\ngen.exe update /force
%windir%\Microsoft.NET\Framework64\v4.0.30319\ngen.exe update /force</code>
{% endhighlight %}

Shutdown the virtual machine and convert to template using vCenter.