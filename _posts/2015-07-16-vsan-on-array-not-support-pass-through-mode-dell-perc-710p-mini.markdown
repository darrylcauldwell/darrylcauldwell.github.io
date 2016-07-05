---
layout: post
title:  "VSAN On Array Not Support Pass Through Mode – Dell Perc 710P Mini"
date:   2015-07-16 16:59:56 +0100
tags:
    - Storage
permalink: vsan-on-array-not-support-pass-through-mode-dell-perc-710p-mini
---
Several array controllers do not support pass through mode,  as such to use this will VSAN, 
we need to create a single disk RAID0 group for every SSD and HDD.

An example of this is the Dell Perc 710P Mini here is a How To guide on how to configure 
RAID0 groups.

When the server is starting up, press Ctrl-R to launch the PowerEdge Expandable RAID Controller 
BIOS. Press Ctrl-R when it is displaying the following message on the console.

<center><img src="/images/RAID0_Config1.png" width="50%"></center>

This will launch the H700 Integrated BIOS Configuration Utility. This utility will have the following 
three TABs on the top.
<ul>
	<li>VD Mgmt – Virtual Disk Management, which is selected by default</li>
	<li>PD Mgmt – Physical Disk Management</li>
	<li>Ctrl Mgmt – Controller Management</li>
</ul>
You’ll see the all disks as “Unconfigured Physical Disks” section under “VD Mgmt” tab as shown below:

<center><img src="/images/RAID0_Config2.png" width="50%"></center>

From the Virtual Disk Management, use arrow key and select ‘PERC H710PIntegrated (Bus 0x03, Dev 0x00)’. 
Press F2 to show available operations. This will display a pop-up menu with following choices. Select 
“Create New VD”.

Press Enter on the RAID option, which will display the available RAID choices, choose RAID-0.

Use Tab key and scroll to the “Physical Disks” section, which will display all the new disks available. 
Press space-bar to select the first disk, a “X” in front of the disk indicates that it is selected to 
be included in this particular virtual disk creation. The “01:00:02″ and “01:00:03″ that is displayed 
in the front of the disks, the last two digits represent the slot number. For hard disk give "VD Name" 
as HDD_Sxx where xx is slot number,  for solid state disk give "VD Name" as SSD_Sxx where xx is slot number.

Keep all the following options to the default values,
<ul>
	<li>Element Size: 64KB</li>
	<li>Read Policy: Adaptive Read</li>
	<li>Write Policy: Write Back</li>
	<li>Force WB with no battery unchecked</li>
	<li>Initialize unchecked</li>
	<li>Configure HotSpare unchecked</li>
</ul>

<center><img src="/images/RAID0_Config3.png" width="50%"></center>

Press enter when ‘OK’ is selected.

Repeat this procedure for all HDD and SSD which will be used for VSAN.