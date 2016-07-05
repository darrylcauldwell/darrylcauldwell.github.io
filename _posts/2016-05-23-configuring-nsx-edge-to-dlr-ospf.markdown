---
layout: post
title:  "Configuring NSX Edge to DLR OSPF"
date:   2016-05-23 16:59:56 +0100
tags:
    - Software Defined Networking
    - VCIX-NV
permalink: configuring-nsx-edge-to-dlr-ospf
---
As part of study for VCIX-NV I’ve given myself task of exploring in my new home lab all parts of NSX which 
I'm still not fully comfortable.  One of these things is OSPF,  to investigate this I came up with a test 
scenario and then worked through the steps to achieve a solution to meet the scenario design.

<H3>Scenario Design</H3>
We will replicate a typical development environment secured with an Edge Gateway.  Within the environment 
the developers will have ability to add and remove logical switches to the DLR and the automatic updating 
of the Edge Gateway routing table.

<center><img src="/images/OSPF-Test.jpg" width="50%"></center>

<H3>Deploy Edge, Transit Logical Switch and Distributed Logical Router</H3>
The lab has VXLAN capability during my NSX core home lab configuration, so has hosts prepared, VNI pool and deployed Controller.

We'll first deploy the Transit logical switch.

<center><img src="/images/ospfLS.jpg" width="50%"></center>

Create Edge Gateway with an Uplink interface on the OOB network  (192.168.1.43) and on the Transit network 
(192.168.2.1) .

<center><img src="/images/OSPFedge.jpg" width="50%"></center>

Create DLR initially with a single uplink on Transit network (192.168.2.254) , do not configure a default gateway.

<center><img src="/images/OSPFdlr.jpg" width="50%"></center>

<H3>Configure OSPF Area</H3>
An OSPF area is a logical collection of OSPF networks, routers, and links that have the same area identification. 
The simulated development environment we are using RFC1918 address space in order that the IP address schema can 
overlap as such our OSPF routing will be internal so we will create an OSPF area which is type Stub.

On Edge Gateway Enable Dynamic Routing Configuration on the Transit interface.

<center><img src="/images/ospfDynamic.jpg" width="50%"></center>

Then we need to enable OSPF

<center><img src="/images/ospfEnable.jpg" width="50%"></center>

We require a stub OSPF area, and usefully Area ID 51 is configured already so we'll use this.  We just 
need to associate this with the Transit interface in order it can communicate with DLR.

<center><img src="/images/ospfMapping.jpg" width="50%"></center>

Repeat the steps used for configuring OSPF on the Edge on the DLR, when enabling OSPF enter the forwarding 
address to match the Transit network IP interface and the protocol address to be another available IP 
address on the Transit network.

<center><img src="/images/ospfProFwd.jpg" width="50%"></center>

<H3>Route Distribution</H3>
The default Route Redistribution configuration for DLR is to distribute information about networks 
it is connected to.

<center><img src="/images/ospfRouteDistribute.jpg" width="50%"></center>

If we now check the Edge Gateway routing table we can see all routing is of type C (directly 
connected) or S (static entry for default gateway).

<center><img src="/images/edgeRouteTable.jpg" width="50%"></center>

If I now simulate a developer creating a new Logical Switch for network 192.168.3.0/24 which 
is attached to DLR as internal interface 192.168.3.1.

<center><img src="/images/test1DLR.jpg" width="50%"></center>

We can then check the Edge Gateway routing table again.

<center><img src="/images/test1Edge.jpg" width="50%"></center>

We see a new route added type O (ospf derived) N2 (OSPF NSSA external type 2).  Which directs 
this traffic to the DLR interface on Transit network.

If we then simulate the developer adding the test2 and test3 networks. We see the other routes populated.

<center><img src="/images/ospfTest2-3.jpg" width="50%"></center>