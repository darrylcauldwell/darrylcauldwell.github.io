---
layout: post
title:  "VMware Data Encryption At Rest"
date:   2018-04-13 22:20:56 +0100
tags:
    - VMware
    - Storage
permalink: vm-encryption/
---
Encryption of data at rest is a requirement for many customers,  with VMware hosted Virtual Machines (VMs) there are two ways to achieve this. VM data can be encrypted using vSAN whole-datastore encryption or using Storage Policy (VMcrypt). Both VM Encryption and vSAN Encryption require a Key Management Interoperability Protocol (KMIP) 1.1 compliant Key Management Server (KMS), the same KMS provider can be used for both types of encryption.

There are important differences between these two encryption methods.

Feature/Function  | VSAN Encryption | VMcrypt Encryption |
------------ | :-----------: | :-----------: |
External key-management server (KMS) | √ | √ |
Per VM Encryption | x | √ |
Datastore Encryption | √ | x |
Data At Rest Encryption | √ | √ |
End To End Encryption | x | √ |
VMs Encrypted By | Placement on datastore | Storage Policy |
Encryption Occurs | After Deduplication | Before Deduplication |

## VM Encryption (VMcrypt)
Encryption is done in the hypervisor, “beneath” the virtual machine. As I/O comes out of the virtual disk controller in the VM it is immediately encrypted by a module in the kernel before being send to the kernel storage layer. Because encryption happens at the hypervisor level and not in the VM, the Guest OS and datastore type are not a factor. The encyrption is hardware accelerated when host CPU support [Intel Advanced Encryption Standard Instructuctions (AES-NI)](https://software.intel.com/en-us/articles/intel-advanced-encryption-standard-instructions-aes-ni). The encryption option is exposed via Storage Policy, therefore it can be applied to only those VMs or groups of VMs which require it. 

VM Encryption is implemented through VAIO (vSphere APIs for IO Filters). The VAIO framework allows a filter driver to do intercept IO that a VM sends down to a device. Ecryption occurs before any data is send across the wire.
<center><img src="https://c2.staticflickr.com/6/5759/30748768661_7114cb408b_z.jpg" width="50%"></center>

## VMcrypt VSAN Efficiency Consideration
An important consideration when using VSAN is that, VMcrypt writes an encrypted data stream whereas vSAN encryption receives an unencrypted data stream and encrypts it during the write process. As the encrypted data written by VMcrypt appears to be random, it does not deduplicate well. If using VMcrypt with VSAN deduplication, expect deduplication efficiency to approach zero for encrypted VMs.

## VSAN Encyption
With vSAN Encryption we can achieve “data encryption at rest” however the data travels to the destination unencrypted then when it reaches its destination it is encrypted, and it will be encrypted after it is deduplicated and/or compressed again. This allows the benefit of deduplicated and/or compressed space saving functionality.

Thie data encyption at rest method is a cluster wide option, which means that every VM will be encrypted.
