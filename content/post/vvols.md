+++
title = "VMware Virtual Volumes Deep Dive"
date = "2014-10-24"
description = "VMware Virtual Volumes Deep Dive"
tags = [
    "vvols",
    "vsphere"
]
categories = [
    "storage"
]
thumbnail = "clarity-icons/code-144.svg"
+++
## How vVols Will Help
The constraints and issues which VVOLs address include, LUNs introduce constrains such as the quantity of LUNs we can present,  having lots of LUNs each with unused space is in efficient as such we currently shared VMs on data stores and this  gives a lack of granular control for passing storage attribute on a per-VM basis.

Virtual Volumes addresses these challenges by rather than using the construct of LUNs on the storage array which is then formatted with VMFS filesystem.  Instead we would create on the storage array a Storage Container, the capabilities of the  Storage Container are exposed via the VMware Aware Storage API (VASA) provider and the data path is exposed via a Protocol Endpoint (PE). While the PE exposes the data path traffic is not funneled through it it provides a pointer, the ESXi to array IO is direct. The removal of the LUN construct and replacement the storage container removes the constraints of using LUNs.

The capabilities of the VMware Aware Storage API (VASA) provider needs to be extended for VVOLs so will increment to version 2. The storage array vendor will provide this extended VASA provider, the current deployment of VASA providers is most typically on a management server, as this is now more critical, it is likely vendors will move its hosting to the array, by way of firmware update. By moving the VASA provider to the array will give the solution the inherent resilience built in to the array.

The Virtual Volume is analogous to the file object which makes up a virtual machine. The other key thing which VVOLs gives us is that by directly exposing the storage we get more full access to offload work to the storage array.  For example snapshots traditionally are managed snapshots within vSphere but now these are performed directly as storage array operations as Un-(vSphere)managed snapshots which allows this to be done as efficiently as possible. As well as snapshotting all SCSI operations, ATS, XCOPY, UNMAP, TRIM etc are also performed natively. One thing to note is that snapshotting of a running virtual machine while being much fantastically improved is not instantaneous for a running virtual machine as the memory bitmap is still required to be written.

![Virtual Volumes](http://www.wooditwork.com/wp-content/uploads/2014/08/image_thumb18.png)

## VVOLs Deep Dive
The Protocol Endpoint (PE) is compatible with all SAN and NAS protocols and each can support any one of the protocols at a given time. The PE is still a SCSI device and as such is managed by the Pluggable Storage Architecture (PSA) which applies the multi-path policies, the PSA is modified slightly to be aware of the PE construct. There can be multiple PEs defined an SCSI PE is picked up during a rescan. While you may at first thing that VVOLs is purely for block storage it is also a construct which can be used with NFS, a NFS PE is maintained as an IP address or file path, ESX identifies and maintains all discovered PEs within a database. Each Virtual volumes has a UID, the data paths are established through a VASA bind request, the bind request is processed by VASA and the mapping of UID and paths can be many to many and all bindings are stored within the database.

![Protocol Endpoint](http://datacenterdude.com/wp-content/uploads/2012/10/vmware-vvol-1.png)

The storage container holds the capabilities of the storage, capabilities would be things like performance tier (Diamond, Gold, Silver, Bronze), backup, snapshots, whether it is de-duplicated. We can therefore use storage containers as logical partitions to apply  storage needs and requirements. There would be a minimum of one storage container per array and maximum would depend on vendor. There is no direct mapping between storage container and PE, a PE can manage multiple storage containers and multiple PEs can manage a single storage container. These storage container capabilities are then advertised for use by ESX through the VMware Aware Storage API (VASA) provider. Surprisingly at first we still need to form Storage Containers into datastores within vSphere. This requirement is to allow all the traditional features of vSphere HA, SDRS etc, which need to interact with the datastore construct.

A virtual volume is per virtual machine but within it is broken up by capability requirement into five sub-types,

* Config-VVOL
* Data-VVOL
* Mem-VVOL
* Swap-VVOL
* Other-VVOL

With this granularity we can now place each VVOL sub-type onto a Storage Container with the best match for its capability requirement.

To consume VVOLs effectively and on a per object (per virtual machine) basis we need to move towards Storage Policy Based Management (SPBM) so we can consume the storage capabilities presented with VVOLs by VMware Aware Storage API (VASA) provider.

In this current release mentioned as 2015 SRM is not supported, although it was mentioned that that is on radar for 2016. The use of RDMs is also not supported within VVOLs, you can however present these as normal outside of the VVOL container.

## Try VVOLs At Home
Up until recently you couldn't realistically configure a VVOLs solution unless you had a spare array running beta firmware which supported VASA2. That is until NetApp shipped [OnTap simulator 8.2.1](http://mysupport.netapp.com/NOW/download/tools/simulator/ontap/8.X/), the NetApp Simulator is run as a nested VM under Fusion, Workstaion or ESX and provides the interface for vSphere 6 to connect to.