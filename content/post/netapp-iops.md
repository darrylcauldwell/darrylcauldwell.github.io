+++
title = "NetApp FAS IOPS Theoretical Maximum & Workload Aggregate Load Balancing"
date = "2014-08-26"
description = "NetApp FAS IOPS Theoretical Maximum & Workload Aggregate Load Balancing"
tags = [
    "netapp",
    "performance"
]
categories = [
    "storage"
]
thumbnail = "clarity-icons/code-144.svg"
+++
To achieve good performance for a virtualized workload your Storage Area Network (SAN) should deliver high IOps with a predictable low latency. If you are using a good flash system where IOps (Input/Output Operations Per Second) effectively become an unlimited resource then latency will probably be your limiting factor.

The NetApp FAS family of storage systems offer a series of arrays at various price points and specifications. NetApp also offer the option of adding Performance Acceleration Module (PAM) to improve the performance of random read intensive workloads by way of a tunable cache to reduce latency. the model you have will shape the latency capability for example the [FAS3140 with PAM and FC disk gives a latency at 40,000 IOPs of 4.4ms](https://www.spec.org/sfs2008/results/res2009q1/sfs2008-20081215-00111.html) where the [FAS3160 with PAM and FC disk gives a latency at 40,000 IOPs of 1.7ms.](https://www.spec.org/sfs2008/results/res2009q3/sfs2008-20090727-00126.html)

You can add varying amounts of disk shelves and disks to the various FAS models. The amount of disk and specification of disk and aggregate design will effect the theoretical maximum amount of IOps. Duncan Epping wrote [this piece](http://www.yellow-bricks.com/2009/12/23/iops/) on the calculation of IOps and if we use this in an example. We have 90 disk per storage controller and we form 3 aggregates each with 30 SAN 15K disks. Within the aggregate we follow [NetApp best practice](https://communities.netapp.com/message/2676%20) to form two RAID groups of no more than 16x disks and so form 2x RAID-DP RAID groups within each aggregate which consumes 2×2 disks for parity.  We know from using vscsiStats that today our VMs generate approximatley 60% read and 40% write traffic,  so our formula would read for each aggregate ((26*175)*0.6)+((26*85)*0.4) = 3614 IOps per aggregate.  If we then add a PAM card the cache would potentially absorb some of IOps going as disk reads, this is very worload dependant but if we look on [page 9 of this NetApp powerpoint](https://communities.netapp.com/servlet/JiveServlet/download/3053-2273/TechONTAP_PAM.ppt) we can see the 256GB PAM absorbing 2000 of the 5847 disk reads,  around 30% which would effect our formula to (((26*175)*0.6)*1.3)+((26*85)*0.4) and give 4433 IOps per aggregate.

It is important to understand the relationship between IOps and the disks as we can see in our example each storage controller has 3x islands of IOps.  As such workload can become hot in one aggregate and cool in others,  to ensure your driving your system the most efficient way you should monitor the average IOps in each aggregate and can look to balance these.  You can use either svMotion to move VM workload between datastores in different aggregates or data motion to move volumes between aggregates.