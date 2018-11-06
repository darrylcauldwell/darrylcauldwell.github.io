---
layout: post
title:  "VMware Certified Advanced Professional - Data Center Virtualization Design"
date:   2017-05-15 22:20:56 +0100
tags:
    - VMware
permalink: vcap-dcv-design/
---
Having passed the VCAP5-DCD exam with little to no studying only eighteen months ago and VCIX6-NV since. All things considered I was pretty confident of doing well in the VCAP6-DCV Design exam, and when I completed exam I was pretty sure I'd passed. To my surprise I failed the exam with a 285, to make matters worse during the drive home I couldn't fathom in my mind where it had gone wrong.
Luckily on readin the score report it highlights the blueprint objectives relating the area's of weakness. For me that was,

* Objective 1.2 - Gather and Analyze Application Requirements
* Objective 1.3 - Determine risks, requirements, constraints and assumptions
* Objective 3.4 - Determine Appropriate Compute Resources for a vSphere 6.x Physical Design

Here I focus on these three objectives, and how I revisited my study and revision to fill my knowledge gap.

# Objective 1.2 - Gather and Analyze Application Requirements

Blueprint Defined Skills and Abilities

* Gather and analyze application requirements for a given scenario.
* Determine the requirements for a set of applications that will be included in the design.
* Collect information needed in order to identify application dependencies.
* Given one or more application requirements, determine the impact of the requirements on the design.

Resources I used

[vBrownBag VCAP6-DCV Design Objective 1.2 with Mark Gabryjelski](https://www.youtube.com/watch?v=ilUrNRY_foI)

# Objective 1.3 - Determine risks, requirements, constraints and assumptions

Blueprint Defined Skills and Abilities

* Differentiate between the concepts of risks, requirements, constraints, and assumptions.
* Given a statement, determine whether it is a risk, requirement, constraint, or an assumption.
* Analyze impact of VMware best practices to identified risks, constraints, and assumptions.

Resources I used

[vBrownBag VCAP6-DCV Design Objective 1.3 with Rebecca Fitzhugh](https://www.youtube.com/watch?v=S_9Al_Tvm5U)

# Objective 3.4 - Determine Appropriate Compute Resources for a vSphere 6.x Physical Design

Blueprint Defined Skills and Abilities

* Analyze best practices with respect to CPU family choices.
* Evaluate NUMA and vNUMA with ESXi hosts and Virtual machines in a given design.
* Analyze the following in a vSphere design:
  * Enhanced vMotion compatibility
  * Implications of vSMP in virtual machines
  * Transparent Page Sharing (TPS) and large pages
  * Resource overcommitment
* Based on the service catalog and given functional requirements:
  * Determine the most appropriate compute technologies for the design
  * Implement the service based on the required infrastructure qualities.
* Determine the impact of a technical design on the choice of server density:
  * Scale Up/Out
  * Auto Deploy
* Calculate the required number of nodes in an HA cluster based upon host failures and resource guarantees
* Evaluate the implications of using reservations, limits, and shares on the physical design.
* Specify the resource pool and vApp configuration based upon resource requirements.
* Size the following resources appropriately:
  * Memory
  * CPU
  * I/O devices
  * Internal storage
* Given a constraint to use existing hardware, determine suitability of the hardware for the design.

Resources I used  
[RAID](https://www.prepressure.com/library/technology/raid)  
[vSphere Flash Read Cache aka vFlash](http://www.yellow-bricks.com/2013/08/26/introduction-to-vsphere-flash-read-cache-aka-vflash/)  
[vSphere Flash Read Cache aka vFlash - FAQs](http://www.yellow-bricks.com/2013/09/11/frequently-asked-questions-vsphere-flash-read-cache/)
[VMware Infrastructure Architecitecture Overview](https://www.vmware.com/pdf/vi_architecture_wp.pdf)

# General

[Site Recovery Manager Maximums](https://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=2105500)