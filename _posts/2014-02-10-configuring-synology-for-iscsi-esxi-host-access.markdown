---
layout: post
title:  "Configuring Synology iSCSI For ESXi"
date:   2014-02-26 16:59:56 +0100
tags:
    - Homelab
permalink: configuring-synology-for-iscsi-esxi-host-access
---
To configure a Synology disk station to present LUNs as iSCSI for ESXi hosts is straight forwards. 
As unlike NFS, iSCSI support is enabled by default.

The first step to is to use Storage Manager to form the LUN,  to do this open Storage Manager and 
change to iSCSI LUN and click create

<center><img src="/images/SynologyISCSI1.png" width="50%"></center>

Use wizard to give the LUN a name,  specify which volume it will reside, whether you 
want it thick, and its size, if this is your first iSCSI LUN ensure iSCSI target mapping is Create 
a new iSCSI target

<center><img src="/images/SynologyISCSI2.png" width="50%"></center>

Leave all defaults for target

<center><img src="/images/SynologyISCSI3.png" width="50%"></center>

Once completed,  edit the new target and ensure all LUNs are mapped which you would want to present 
to ESXi.

<center><img src="/images/SynologyISCSI4.png" width="50%"></center>

If you now create a iSCSI adapater and point it to the Synology IP to dynamically discover the LUNs 
as shown [here.](https://www.youtube.com/v/xAxE5LOAXSE)