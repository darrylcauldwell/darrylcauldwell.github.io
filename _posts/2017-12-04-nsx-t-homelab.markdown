---
layout: post
title:  "VMware NSX-T 2.0 Homelab"
date:   2017-05-15 22:20:56 +0100
tags:
    - Homelab
    - Software Defined Networking
permalink: nsx-t-homelab/
---
NSX-T that can provide network virtualization for a multi-cloud and multi-hypervisor environment. NSX-V (NSX for vSphere) Manager integrates into vCenter and leverages the vSphere dvSwitch. NSX-T Manager can be used with vSphere but does not integrate with vCenter or dvSwitch, instead NSX Manager is managed its API and the Transport Zone creates a  hostswitch on each host.

Current homelab, 2x Intel NUC running ESXi, All-Flash VSAN, VCSA deployed, each NUC has 2x pNIC a single pNIC from each host is connected to a dvSwitch.
<center><img src="/images/NSX-T-Homelab-Before.jpeg" width="50%"></center>
Our goal here is to deploy NSX Manager, an NSX Controller and get a functioning NSX Transport Zone.
<center><img src="/images/NSX-T-Homelab-Target.jpeg" width="50%"></center>

## Step 1 - Deploy NSX-T Manager
The NSX-T Manager is a VM sized 16GB vRAM, 2x vCPU and 140GB of vHDD.

I connect its single vNIC to an the same existing dvPortgroup on the existing dvSwitch as which vCenter and AD (DNS and NTP) are connected. I configure the appliance to use the IP of AD for DNS and NTP.

As I will be connecting to the API I ensure A and PTR records are in place for NSX Manager.

## Step 2 - Deploy NSX-T Controller
Here I will deploy a single NSX-T Controller as I am really testing connectivity and not availability.

The NSX-T Controller is a VM sized 16GB vRAM, 2x vCPU and 120GB of vHDD.

I connect its single vNIC to an the same existing dvPortgroup on the existing dvSwitch as which NSX Manager is connected to. I configure the appliance to use the IP of AD for DNS and NTP.

## Step 3 - Connect NSX-T Controller with NSX-T Manager
Open an SSH session to NSX Manager and run
```bash
get certificate api thumbprint
```

Open an SSH session to NSX Controller
```bash
join management-plane NSX-Manager username admin thumbprint <NSX-Managers-thumbprint>

set control-cluster security-model shared-secret

initialize control-cluster
```

## Step 4 - Deploy NSX-T Transport Zone
Open a web browser to https://(nsx-mgr), navigate menu to Select Fabric > Nodes > Hosts and click Add.

Add both ESXi hosts accepting SSL thumprints when prompted that they invalid.

Open an SSH session to NSX Manager and run
```bash
get certificate api thumbprint
```

Open an SSH session to each ESXi Host and run
```bash
/opt/vmware/nsx-cli/bin/scripts/nsxcli

join management-plane NSX-Manager1 username admin thumbprint <NSX-Managers-thumbprint>
```

## Step 5 - Deploy Logical Switches and Logical Router
Once we have a functioning NSX Transport Zone we can begin deploying Network Virtualization objects. Here we deploy a pair of Logical Switches and join these with a Logical Router.

