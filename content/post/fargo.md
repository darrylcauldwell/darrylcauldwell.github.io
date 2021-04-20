+++
title = "VMware Projects Fargo (VMFork) & Meteor"
date = "2015-10-06"
description = "VMware Projects Fargo (VMFork) & Meteor"
tags = [
    "docker",
    "vsphere"
]
thumbnail = "clarity-icons/code-144.svg"
+++
There was announced during VMworld a technology preview for Project Fargo (formerly VMFork) it is likely this will be launched with vSphere6. The aim of Fargo is to provides a fast, scalable differential clone of a running VM.  I see this as very similar to Redirect-on-Write (RoW) methodology used by NetApp snapshots where at the point of snap the existing blocks are frozen and any writes (creations/changes/deletions) are redirected to new blocks. However with Fargo rather than than a snapshot we are creating a Copy-on-Write(CoW) the difference being that with CoW the original data that is being written to is copied into a new file that is set aside for the snapshot before original data is overwritten. So before a write is allowed to a block, copy-on-write moves the original data block to the snapshot storage.

![Copy On Write](/images/fargo-cow.gif)

The key benefits of the use of this method is that its near instantaneous and can be done from a running VM,  so a new VM spawned would typically take less than 1 second and be in the same running state. As well as this as only changed blocks are written the solution will take up dramatically less disk space.

While there are many potential use cased for Fargo it was presented with virtual desktop in mind where providing an instant clone of running non-persistent desktop would avoid the boot storms to the storage subsystem.

So now you have quick provisioning of operating system you then need to deliver the right user applications to it quickly and this is I assume to be done by [CloudVolumes](http://blogs.vmware.com/euc/2014/08/cloudvolumes.html) where applications are abstracted and layered onto the users operating system.  The combination of these two tools appears to be known as Project Meteor.