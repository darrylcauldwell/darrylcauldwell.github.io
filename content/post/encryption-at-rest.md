+++
title = "VMware Data Encryption At Rest"
date = "2018-04-13"
description = "VMware Data Encryption At Rest"
tags = [
    "encryption",
    "vsan",
    "vsphere"
]
thumbnail = "clarity-icons/code-144.svg"
+++
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

![VMware Encryption](/images/encryption.jpg)

VM Encryption is implemented through [vSphere APIs for IO Filters (VAIO)](https://code.vmware.com/programs/vsphere-apis-for-io-filtering). The VAIO framework allows a filter driver to intercept IO that a VM sends down to a storage device. Ecryption occurs before any data is send across the wire.

![vSphere APIs for IO Architecture](/images/encryption-vaio-architecture.png)

## VMcrypt VSAN Efficiency Consideration
An important consideration when using VSAN is that, VMcrypt writes an encrypted data stream whereas vSAN encryption receives an unencrypted data stream and encrypts it during the write process. As the encrypted data written by VMcrypt appears to be random, it does not deduplicate well. If using VMcrypt with VSAN deduplication, expect deduplication efficiency to approach zero for encrypted VMs.

## VSAN Encyption
With vSAN Encryption we can achieve “data encryption at rest” however the data travels to the destination unencrypted then when it reaches its destination it is encrypted, and it will be encrypted after it is deduplicated and/or compressed again. This allows the benefit of deduplicated and/or compressed space saving functionality.

Thie data encyption at rest method is a cluster wide option, which means that every VM will be encrypted.

![VMware Encryption](/images/encryption-vsan-hytrust.png)

## Key Management Server (KMS) Topologies

Key Management Server (KMS) availability is a critical component when deploying encryption as without it all data access is not allowed. If you lose the keys, you’ve lost the data (unless you’ve backed it up).

Key Managers today are usually set up in a way that they replicate keys to one another. If I have three instances of a key manager, KMS-A, KMS-B and KMS-C, they replicate the keys between them. If I create a key on KMS-A it will show up in KMS-B & KMS-C at some point. Using this example, in vCenter I would create a key manager cluster/alias. In this example I’ll call it “KMSCluster” and add KMS-A, KMS-B and KMS-C into that KMS Cluster. I would then establish a trust with each of the key managers.

KMS can either be purchased as hardware appliances or virtual appliances. When using KMS virtual appliances avoid circular dependancy where the datastore hosting the appliance is encrypted using itself. The same is true for vCenter and PSC’s in a VM Encryption scenario. You shouldn’t encrypt them using VM Encryption because they would then need to boot up to get their encryption key to boot up.

If you have a single site then you probably want to have, at minimum, two replicating key managers. For multi-site,  you want all the KMS servers running and replicating across the sites.

![VMware Encryption](/images/encryption-multisite.png)