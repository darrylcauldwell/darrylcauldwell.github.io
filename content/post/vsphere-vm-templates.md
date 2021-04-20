+++
title = "Create VMware Virtual Machine Templates"
date = "2016-05-17"
description = "Create VMware Virtual Machine Templates"
tags = [
    "vm guest customization",
    "vsphere"
]
categories = [
    "automation"
]
thumbnail = "clarity-icons/code-144.svg"
+++
When creating virtual machine templates I'd like to do this consistently as such I'll try and keep this post up to date with the settings I am including.

## CentOS 7
New custom VM named CentOS7, hardware version 11, guest operating system Linux version 'CentOS 4/5/6/7 (64-bit)'. We'd like a small template which we expand if required, so single virtual socket with single core, 2GB virtual memory, one VMXNET3 NIC, default LSI Logic Parallel SCSI controller with a thin 16GB hard disk and mount CentOS7 ISO as CD-ROM.

Power on VM and open console, ensure you change NIC to connected and enter password otherwise leave all as default and install. Now I install common tools to use within lab, so once installed reboot and logon still using remote console.

```bash
yum install open-vm-tools net-tools epel-release gcc git
yum update
cp -f /etc/sysconfig/network-scripts/ifcfg-eth0 /tmp/eth0
sed "/^HWADDR/d" /tmp/eth0 &gt; /etc/sysconfig/network-scripts/ifcfg-eth0
```

Shutdown the virtual machine and convert to template using vCenter.

## Windows 2012 R2
New custom VM named Win2012R2, hardware version 11, guest operating system Windows version 'Microsoft Windows Server 2012 (64-bit)'. We'd like a small template which we expand if required, so single virtual socket with single core, 4GB virtual memory, one VMXNET3 NIC, default LSI Logic SAS SCSI controller with a thin 40GB hard disk and mount Windows 2012 R2 ISO as CD-ROM.

Power on VM and open console, select Datacenter Edition with GUI and enter password otherwise leave all as default and install. Once installed rebooted logon and perform Typical VMware tools install.

* Apply MSDN License Key and Active Windows
* Disable IE Enhanced Security
* Use Windows Update to apply all current patches, repeat until all dependant patches are installed.
* Copy \Sources\SxS folder from CD-ROM to C:\
* As some Updates will update .net we should force the Assemblies to get updated

```powershell
%windir%\Microsoft.NET\Framework\v4.0.30319\ngen.exe update /force
%windir%\Microsoft.NET\Framework64\v4.0.30319\ngen.exe update /force</code>
```

Shutdown the virtual machine and convert to template using vCenter.