---
layout: post
title:  "NSX Distributed Firewall Under The Covers"
date:   2015-07-21 16:59:56 +0100
tags:
    - Software Defined Networking
permalink: nsx-distributed-firewall-under-the-covers
---
An NSX distributed firewall (dFW) runs as an ESXi host as a kernel module added as a VMware Installation Bundle 
(VIB). The dFW rules operate on Layer 2 through Layer 4; although this can be extended through Layer 7 by 
integrating with a 3-Party vendor.
<ul>
	<li>L2 rules are based on MAC address L2 protocols like ARP, RARP and LLDP etc.</li>
	<li>L3 &amp; 4 rules are based on IP source/destination and L4 uses a TCP or UDP ports.</li>
</ul>

<center><img src="/images/dFW_TCP_OSI.png" width="50%"></center>

The NSX dFW enforces a state full firewall service for VMs using the vNIC as the enforcement point. Every packet 
that leaves the VM (before VTEP encapsulation) or enters the VM (After VTEP de-encapsulation) can be inspected 
with a firewall policy.

The ruleset is created and managed via NSX Manager either API or UI. The ESXi host has two dFW specific modules 
vShield Statefull Firewall and VMware Internetworking Service Insertion Platform (vSIP). vShield Statefull 
Firewall performs the following roles.
<ul>
	<li>Interact with NSX Manager to retrieve DFW policy rules.</li>
	<li>Gather DFW statistics information and send them to the NSX Manager.</li>
	<li>Send audit logs information to the NSX Manager.</li>
	<li>Receive configuration from NSX manager to create (or delete) DLR Control VM, create (or delete) ESG.</li>
	<li>Part of the host preparation process SSL related tasks from NSX manager</li>
</ul>
VMware Internetworking Service Insertion Platform is the distributed firewall kernel space module core component. 
The vSIP receives firewall rules via vShield State-full Firewall and downloads them down to each VMware-sfw. When 
VM connect to Logical switch there are security services each packet transverse which are implemented as IOChains 
processed at the Kernel level.

<center><img src="/images/dFW_Slots.png" width="50%"></center>

<ul>
	<li>Slot 0: Distributed Virtual Filter (DVFilter): It monitors the incoming and outgoing traffic on the protected virtual NIC and performs stateless filtering.</li>
	<li>Slot 1: Switch Security module (SW-sec): Learns VMs IP and MAC address. sw-sec is critical component capture DHCP Ack and ARP broadcast message and forward this info as unicast to NSX Controller to perform the ARP suppression feature. This is also the layer where NSX IP spoof guard is implemented.</li>
	<li>Slot 2: NSX Distributed Firewall (VMware-sfw): This is the place where DFW firewall rules are stored and enforced; VMware-sfw contains rules table and connections table received via vShield State-full Firewall</li>
</ul>
The effect of the processing of these packets is that packet leaving the VM which doesn’t match firewall rules get 
removed before arriving at the vSphere Switch.
