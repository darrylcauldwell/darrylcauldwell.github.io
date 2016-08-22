---
layout: post
title:  "NSX Activity Monitoring"
date:   2016-05-20 16:59:56 +0100
tags:
    - Software Defined Networking
permalink: nsx-activity-monitoring/
---
As part of study for VCIX-NV I've given myself task of exploring all parts of NSX which I don't use at work. One of 
these things is NSX Activity Monitoring,  to investigate this I came up with a test scenario and then worked through 
the steps to achieve a solution to meet the scenario design.

<H3>Scenario Design</H3>
Gather traffic data between Activity Monitored virtual machines.

<H3>Virtual Machine Configuration</H3>
I created a Windows 2012 R2 virtual machine template previously,  deploy two of these to a network with DHCP enabled 
for example in my lab the OOB network.

The Windows template I created has the VMware Tools typical installation performed. To use the NSX Activity Monitoring 
the Network File Introspection Driver (vnetflt.sys) is required.  To add this we need to re-run the VMware tools 
installer,  select modify and then add NSX Network File Introspection Driver.

<center><img src="/images/WindowsDriver.jpg" width="50%"></center>

Add IIS role to one of the hosts.

Add server to Active Directory domain and reboot.

In vSphere web client select each virtual machine in turn and enable NSX Activity Monitoring.

<center><img src="/images/EnableMonitoring.jpg" width="50%"></center>

<h3>Deploy Activity Monitor Appliances</h3>
From the Networking and Security menu, select Installation. Navigate to the Service Deployments tab and glick green 
add button.

In order to gather activity data virtual appliances are required to be deployed.

<center><img src="/images/DeployActivityServices.jpg" width="50%"></center>

<H3>Register a Windows Domain with NSX Manager</H3>
In vSphere web client click Networking &amp; Security and then click NSX Managers. Click an NSX Manager in the Name 
column and then click the Manage tab. Click the Domain tab and then click the Add domain (Add domain) icon.

Complete the domain details as per your Active Directory.

<center><img src="/images/Add-Domain.jpg" width="50%"></center>

Wait for synchronization status to show SUCCESS.

<H3>Generate Activity</H3>
Connect to test virtual machine which <strong>does not</strong> have IIS installed and open browser to virtual machine with IIS installed.

Test data is being gathered by Activity Monitor.

<center><img src="/images/ActivityMonitorActivity.jpg" width="50%"></center>

Another easy test is to copy a file between the two virtual machines.

<center><img src="/images/ActivityMonitorSMB-CIFS.jpg" width="50%"></center>
