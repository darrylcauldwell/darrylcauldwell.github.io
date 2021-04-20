+++
title = "NSX Activity Monitoring"
date = "2016-05-19"
description = "NSX Activity Monitoring"
tags = [
    "nsx",
    "vcix"
]
categories = [
    "networking",
    "certification"
]
+++
As part of study for VCIX-NV I've given myself task of exploring all parts of NSX which I don't use at work. One of these things is NSX Activity Monitoring,  to investigate this I came up with a test scenario and then worked through the steps to achieve a solution to meet the scenario design.

## Scenario Design

Gather traffic data between Activity Monitored virtual machines.

## Virtual Machine Configuration

I created a Windows 2012 R2 virtual machine template previously,  deploy two of these to a network with DHCP enabled for example in my lab the OOB network.

The Windows template I created has the VMware Tools typical installation performed. To use the NSX Activity Monitoring the Network File Introspection Driver (vnetflt.sys) is required.  To add this we need to re-run the VMware tools installer,  select modify and then add NSX Network File Introspection Driver.

![Install Windows Driver](/images/nsx-activity-WindowsDriver.jpg)

Add IIS role to one of the hosts.

Add server to Active Directory domain and reboot.

In vSphere web client select each virtual machine in turn and enable NSX Activity Monitoring.

![Enable Monitoring](/images/nsx-activity-EnableMonitoring.jpg)

## Deploy Activity Monitor Appliances

From the Networking and Security menu, select Installation. Navigate to the Service Deployments tab and glick green add button.

In order to gather activity data virtual appliances are required to be deployed.

![Deploy Virtual Appliances](/images/nsx-activity-DeployActivityServices.jpg)

## Register a Windows Domain with NSX Manager

In vSphere web client click Networking and Security and then click NSX Managers. Click an NSX Manager in the Name column and then click the Manage tab. Click the Domain tab and then click the Add domain (Add domain) icon.

Complete the domain details as per your Active Directory.

![Add Domain](/images/nsx-activity-Add-Domain.jpg)

Wait for synchronization status to show SUCCESS.

## Generate Activity

Connect to test virtual machine which <strong>does not</strong> have IIS installed and open browser to virtual machine with IIS installed.

Test data is being gathered by Activity Monitor.

![Activity Monitor Activity](/images/nsx-activity-ActivityMonitorActivity.jpg)

Another easy test is to copy a file between the two virtual machines.

![Activity Monitor Activity](/images/nsx-activity-ActivityMonitorSMB-CIFS.jpg)