---
layout: post
title:  "VMware NSX-T 2.0 Homelab"
date:   2017-12-04 22:20:56 +0100
tags:
    - Homelab
    - Software Defined Networking
permalink: nsx-t-homelab/
---
NSX-T that can provide network virtualization for a multi-cloud and multi-hypervisor environment. NSX-V (NSX for vSphere) Manager integrates into vCenter and leverages the vSphere dvSwitch. NSX-T Manager can be used with vSphere but does not integrate with vCenter or dvSwitch, instead NSX Manager is managed its API and the Transport Zone creates a  hostswitch on each host.

Current homelab, 2x Intel NUC running ESXi, All-Flash VSAN, VCSA deployed, each NUC has 2x pNIC a single pNIC from each host is connected to a dvSwitch the other is available for use with NSX-T Hostswitch.

<center><img src="/images/NSX-T-Homelab-Before.jpeg" width="50%"></center>

My goal here is to deploy NSX Manager, an NSX Controller and get a functioning NSX Transport Zone.

<center><img src="/images/NSX-T-Homelab-Target.jpeg" width="50%"></center>

## Step 1 - Deploy NSX-T Manager & Controller
The total resources required for this deployment are 32GB vRAM, 8x vCPU and 380GB vHDD.

I deployed a small NSX-T Manager it is a VM sized 8GB vRAM, 2x vCPU and 140GB of vHDD, an NSX-T Controller it is also a VM but sized 16GB vRAM, 4x vCPU and 120GB of vHDD. I connect both of these to an the same existing dvPortgroup on the existing dvSwitch as which vCenter and AD (DNS and NTP) are connected. I configure the appliances to use the IP of AD for DNS and NTP.

I deployed a small NSX-T Edge it is a VM sized 8GB vRAM, 2x vCPU and 120GB of vHDD. I connect both of these to an the same existing dvPortgroup on the existing dvSwitch as which vCenter and AD (DNS and NTP) are connected. I configure the appliances to use the IP of AD for DNS and NTP.



## Step 2 - Connect NSX-T Controller with NSX-T Manager
Open an SSH session to NSX Manager and run
```bash
get certificate api thumbprint
```

Open an SSH session to NSX Controller and run
```bash
join management-plane <NSX-Manager> username admin thumbprint <NSX-Managers-thumbprint>

set control-cluster security-model shared-secret

initialize control-cluster
```

<center><img src="/images/NSX-T-Homelab-Mgr-Ctrl-Thumb.jpeg" width="50%"></center>

## Step 3 - Configure vCenter Connection
Open a web browser to https://(nsx-mgr), navigate menu to select Compute Managers > and click Add

Enter details for vCenter

<center><img src="/images/NSX-T-Homelab-Compute-Manager.jpeg" width="50%"></center>

## Step 4 - Configure A NSX-T Fabric
Open a web browser to https://(nsx-mgr), navigate menu to select Fabric > Nodes > Hosts and click Add,  enter details for both hosts.

<center><img src="/images/NSX-T-Homelab-Add-Hosts.jpeg" width="50%"></center>

Navigate menu to Select Fabric > Transport Zones and click Add, give Transport Zone and Host Switch a name, ensure Overlay selected (Default)

<center><img src="/images/NSX-T-Homelab-Transport-Zone.jpeg" width="50%"></center>

Navigate menu to Select Fabric > Transport Nodes and click Add, give Transport Node and select first ESX host,  select Transport Zone created just before.  Switch to Host Switches tab and select transport zoine for Host Switch a name, select Uplink profile already created,  select Use IP Pool and create a new IP Pool large enough for all hosts (for me 2x)

<center><img src="/images/NSX-T-Homelab-Hostswitch-IP-Pool.jpeg" width="50%"></center>



## Step 5 - Deploy Logical Switches and link with a Logical Router
Once we have a functioning NSX Transport Zone we can begin deploying Network Virtualization objects. Here I deploy a pair of Logical Switches and join these with a Logical Router.

Navigate menu to Select Switching and click Add,  enter name and ensure the transport zone you created is populated. Repeat for 2nd switch.

<center><img src="/images/NSX-T-Homelab-Logical-Switches.jpeg" width="50%"></center>

Navigate menu to Select Routing and click Add,  select Tier-1 router,  enter name. 

This will create the Logical Router, once created select it from the list and in right pane select Configuration > Router Ports and click Add,  enter name,  select Logical Switch to connect to and specify a IP Address and mask. Repeat for 2nd switch.

<center><img src="/images/NSX-T-Homelab-Logical-Router-Port.jpeg" width="50%"></center>

If all has gone to plan we now see the two Logical Switches in vCenter.

<center><img src="/images/NSX-T-Homelab-vSphere-Networks.jpeg" width="50%"></center>

We can then build some test VMs and join VMs to these two networks.

<center><img src="/images/NSX-T-Homelab-VM-Logical-Switch.jpeg" width="50%"></center>

If all has gone to plan these can ping each other between Distributed Logical Switeches across the Distributed Router.

<center><img src="/images/NSX-T-Homelab-VM-Pings.jpeg" width="50%"></center>

These VMs are now consuming the Transport Node (Hostswith) on each host and seemlessly communicating across the overlay network.

<center><img src="/images/NSX-T-Homelab-Inter-VM-Logical-.jpeg" width="50%"></center>

## Step 5 - Deploy Edge
Open a web browser to https://(nsx-mgr), navigate menu to select Fabric > Edges > and click Add Edge VM
* Specify name and FQDN of the appliance, select small form factor,  and click Next
* Specify passwords,  and click Next
* Select vCenter Compute Manager created in previous step,  and populate with appropriate information
* At configure ports,  specify Management IP/Gateway and Interface on same network as NSX Manager, Controller and vCenter. For datapath repeat network MOID 3*

The form requires supplying MOID for datastore and portgroup.  For finding MOID's browse to https://(vcenter)/mob then follow links ServiceContent = content -> rootFolder = datacenters -> childEntity = datacenter.