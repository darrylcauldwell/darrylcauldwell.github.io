+++
title = "Intel NUC Cluster Hardware"
date = "2015-04-16"
description = "Intel NUC Cluster Hardware"
tags = [
    "nuc"
]
categories = [
    "homelab"
]
featured = true
thumbnail = "clarity-icons/code-144.svg"
+++
I have worked most of my professional life with systems built with Intel x86, x64. Often there is little hardware available in the work environment to learn and try out new things.

I enjoy learning by doing and to begin to better understand the technology looked to build a cluster of Intel NUCs. My goals of cluster include understanding more about:

* VMware vSphere, NSX and vSAN
* Kubernetes, OpenShift and Tanzu
* Configuring infrastructure with Python, Ansible and Terraform

## Compute

The NUC6i5SYH was latest model when I purchased. My purchasing decision was primarily lead by power as these draw 60watt at peak load. I also wanted to try vSAN which drove need for at least two disk bays and the NUC has an M.2 and a 2.5" Drive bay.

## Storage

To form vSAN required the two internal disks be fully available which drove the addition of USB Flash Drive. The NUCs sit on and around desk so its very easy for them to get knocked the SanDisk Cruzer Fit formfactor keeps the chance of damage to a minimum.

I was working to a budget but wanted good performance. When cluster operating with vSAN all writes go via a cache disk and then get destaged to capacity tier. For the vSAN cache tier I chose the 128GB Samsung SM951 NVMe M.2 as it is rated for ~2,150MiB/sec read and 1550MB/s MiB/sec write. For the vSAN capacity tier I chose the 1TB Samsung 850 EVO SATA3 as it is rated for ~500MiB/sec read and write. The lab does not need a lot of capacity so only two of the three NUCs are populated with disks and contribute to vSAN the third contributes only compute capacity to the cluster.

As well as internal storage I have a Synology DiskStation DS213j a  "budget-friendly" 2-bay NAS server. It isn't the fastest but offers iSCSI and/or NFS with 100MiB/sec read and 70MiB/sec write.

## Network

One of usecases I wanted to explore with this hardware was Software Defined Networking. To this end I wanted network switch to support configuring multiple VLANs. To connect three NUCs each with two 1GbE NICs would consume six ports, to uplink to wider home network and the Synolofy DiskStation drove my choice to the eight 1GbE port Cisco SG200-08.

To simulate redundant networking scenarios at the ESXi side I wanted to add a second NIC albeit connecting to same switch. To allow for USB NIC on NUCs VMware released a unsupported driver by way of a Fling. This was a little prescritive and this lead me to the StarTech USB 3.0 to Gigabit Ethernet NICs.

My home office is in a cabin in the garden while it has power cabling it has no network cabling. To give a cabled network connection I use TP-Link AV500/AV600s. As these are very specific to my scenario I've not included them in the BOM.

## Bill Of Materials

* 3x Intel NUC 6th Gen NUC6i5SYH (3x £358=) £1,074
* 3x Crucial 32GB Kit (2x16GB) DDR4 (3x £86=) £258
* 2x Samsung SM951 NVMe 128GB M.2 (2x £43=) £86
* 2x Samsung 850 EVO 1TB SATA3 (2x £248=) £496 
* 3x SanDisk Cruzer Fit 16GB USB Flash Drive (3x £5=) £15
* 3x StarTech USB 3.0 to Gigabit Ethernet (3x £17=) £51
* 1x Cisco SG200-08 8-port Gigabit £73
* 7x 0.5m Cat6 Cables £7

Total cluster parts cost ~£2060

Note: Prices at time of purchase in 2014/2015 