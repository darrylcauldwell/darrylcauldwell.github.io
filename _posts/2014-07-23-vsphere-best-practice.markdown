---
layout: post
title:  "vSphere Best Practice"
date:   2014-07-23 16:59:56 +0100
tags:
    - VMware
permalink: vsphere-best-practice
---
I often get asked the question,  what is the best general configuration for private VMware hosting. Typically this is when an engineer or architect has been reading the multiple best practices and has become confused as much best practice advise is conditional. They just want to stand something up quickly with little detail on what the requirements are and then as solution gets used evolve the configuration based on the requirements which come through usage. Its certainly not ideal but does seem to happen a lot.

As such I compiled this list of what I think are general purpose best practices, these are not hard and fast just a starting point you should over time evaluate each within your environment.

<strong>vSphere</strong>
<ul>
	<li>Most current version ,  so 5.5</li>
	<li>Include <a href="http://www.vmware.com/products/vsphere/features/update-manager.html?src=vmw_so_vex_dcaul_99" target="_blank">VMware Update Manager</a> in design</li>
	<li>Include <a href="http://www.vmware.com/products/vcenter-operations-management/?src=vmw_so_vex_dcaul_99" target="_blank">vCenter Operations Managment Suite</a> in design (even if no fund just have a VM to run evaluation for 60 days and profile environment)</li>
	<li><a href="http://kb.vmware.com/kb/2003322/?src=vmw_so_vex_dcaul_99" target="_blank"> Centralized Syslog</a> and <a href="http://kb.vmware.com/kb/1032051/?src=vmw_so_vex_dcaul_99" target="_blank">Dump Collector</a></li>
	<li>ESXi install preference,  HDD, USB \ SD, <a href="http://kb.vmware.com/kb/2005131/?src=vmw_so_vex_dcaul_99" target="_blank">AutoDeploy</a>,  SAN boot</li>
	<li>Don't use<a href="http://kb.vmware.com/kb/1008077/?src=vmw_so_vex_dcaul_99" target="_blank"> lockdown mode</a> unless specifically required</li>
</ul>
<strong>Monitoring</strong>
<ul>
	<li>Configure <a href="http://kb.vmware.com/kb/1008065/?src=vmw_so_vex_dcaul_99" target="_blank">SNMP trapping</a> for vCenter and any outbound from the server hardware running ESXi</li>
</ul>
<strong>High Availability</strong>
<ul>
	<li>Form clusters and enable HA on clusters</li>
	<li>Configure to use percentage of resources</li>
	<li>Enable <a href="http://pubs.vmware.com/vsphere-50/index.jsp?topic=%2Fcom.vmware.vsphere.avail.doc_50%2FGUID-62B80D7A-C764-40CB-AE59-752DA6AD78E7.html?src=vmw_so_vex_dcaul_99" target="_blank">VM level HA monitoring</a></li>
	<li>Use multiple <a href="http://kb.vmware.com/kb/1030320/?src=vmw_so_vex_dcaul_99" target="_blank">isolation response</a> addresses of HA device IPs such as HSRP address of a network devices and change isolation response to Shutdown</li>
</ul>
<strong>DRS</strong>
<ul>
	<li>Form clusters and enable DRS</li>
	<li>Use larger clusters and use <a href="http://pubs.vmware.com/vsphere-51/index.jsp?topic=%2Fcom.vmware.vsphere.resmgmt.doc%2FGUID-FF28F29C-8B67-4EFF-A2EF-63B3537E6934.html?src=vmw_so_vex_dcaul_99" target="_blank">DRS anti-affinity rules</a> to create smaller groupings within if required for SQL licensing etc</li>
	<li>Set exceptions for application specifics such as MS Lync no vMotion etc</li>
</ul>
<strong>sDRS</strong>
<ul>
	<li>Use <a href="http://www.vmware.com/products/vsphere/features/storage-api.html?src=vmw_so_vex_dcaul_99" target="_blank">VASA </a>profile meta data from the storage array rather than create manually if possible</li>
</ul>
<strong>VAAI Thin Provisioning &amp; Unmap</strong>
<ul>
	<li>Encourage VM guests OS choice to support TRIM\UnMap so Windows 2012+ (be careful of 3am unmap storm)</li>
	<li>Schedule datastore free <a href="http://kb.vmware.com/kb/2057513/?src=vmw_so_vex_dcaul_99" target="_blank">SAN block unmapping</a></li>
	<li>Plan VM template placement to maximise clone offload,  dependant on storage but within same aggregate as target datastore will do an offload rather than a full copy</li>
	<li>As a starting point we generally aim for no more than 15 VMs per datastore,  to note isn't a hard limit just a balance on risk we have formed</li>
</ul>
<strong>CPU</strong>
<ul>
	<li>Enable Turbo Boost (where available)</li>
	<li>Enable Hyperthreading (where available)</li>
	<li>Enable Hardware Support for virtualization</li>
	<li>Intel VT-x or AMD-V</li>
	<li>Intel VT-d or AMD IOMMU</li>
	<li>No eXecute (NX)/eXecute Disable (XD)</li>
	<li>Always right-size your VMs,  be very careful not to oversize with too many vCPU</li>
	<li>Idle vCPU wastes resources</li>
	<li>VMs can always be increased in size if necessary, whereas an oversized VM will not get the attention required to change the size at a later date</li>
	<li>Where possible use 1 vCPU per virtual socket</li>
	<li>Putting multiple cores per virtual socket can impact performance in NUMA systems</li>
</ul>
<strong>NUMA</strong>
<ul>
	<li>Use a <a href="http://blogs.vmware.com/vsphere/tag/vnuma?src=vmw_so_vex_dcaul_99" target="_blank">vNUMA </a>enabled version of vSphere so 5.0+</li>
	<li>Use virtual hardware enabled version 8+ on VMs with NUMA aware OS so like Windows 2008 R2 and newer</li>
	<li>Use virtual machine guest OS version which is NUMA aware such as Windows 2008 or newer</li>
	<li>Use 1 vCPU per virtual socket, putting multiple cores per virtual socket can impact memory latency in NUMA enabled systems.</li>
</ul>
<strong>Networking</strong>
<ul>
	<li>Configure ESXi management portgroup on Standard Switch and others on vDistributed Switch</li>
	<li>SplitRx mode is supported only on vmxnet3 network adapters and is disabled by default. Enable SplitRx Mode in situations where multiple virtual machines share a single physical NIC and receive a lot of multicast or broadcast packets.</li>
	<li>Ensure Portfast is enabled on all network switch ports connected to ESX(i) servers</li>
	<li>Ensure SpanningTree is disables on all network switch ports connected to ESX(i) servers</li>
	<li>Enable Jumbo Frames for network based storage and vMotion network.</li>
	<li>When specifying NICs look for 10g PCIe network cards with NetQueue feature.</li>
	<li>Use multiple uplinks per vSwitch</li>
	<li>Separate vSwitch by network function where possible with NIC quantity</li>
	<li>Use 802.1Q trunked connections with base VLAN set to the ESX management network VLAN</li>
	<li>If there is a NetFlow collector in environment configure vSphere to send information to it if required</li>
</ul>
<strong>Power</strong>
<ul>
	<li>Set server power mode to “OS Control” in BIOS</li>
</ul>
<strong>Storage
</strong>
<ul>
	<li>Work with storage vendor to define which MP to use per array</li>
	<li>Split each VM guest volume into its own VMDK so each can be resized independently and each can be placed on datastore with best suited characteristics</li>
	<li>Use array snapshots where possible for VM guest backups</li>
	<li>Use vscsiStats to profile the workload of your VMs and work with your storage team to form LUNs most suited to the workload profiles</li>
</ul>
<strong>Vendor Packs</strong>
<ul>
	<li>Install and use storage vCentre \ vCOPs plugins</li>
	<li>Install and use server vendor vCenter plugins</li>
</ul>
