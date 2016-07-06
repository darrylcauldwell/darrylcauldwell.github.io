---
layout: post
title:  "VMware Validated Design for SDDC"
date:   2015-10-04 16:59:56 +0100
tags:
    - VMware
permalink: vmware-validated-designs-an-architecture-for-the-sddc
---
Historically VMware engineers and architects produce best practice documentation by way of reference architectures and whitepapers which are product and product version focused. As the VMware product portfolio is expanding to more of a suite of products under the banner Software-Defined Data Centre (SDDC) interoperability becomes more important and the best practice settings, testing and supportability becomes more complex.

VMware Validated Designs are architectures created and validated by VMware encompassing the entire product stack used in VMware’s Software-Defined Data Center. Product versions within a validated design are tested together and structured stack migration paths to new versions will be made available. Initially five validated stacks are being detailed,  and two have been released,
<ul>
	<li><a href="http://vmware.com/go/v2d-foundation" target="_blank">Datacenter Foundation</a></li>
	<li><a href="http://vmware.com/go/v2d-itac-single" target="_blank">Single-Region</a></li>
	<li>Dual-region</li>
	<li>QE</li>
	<li>Demo</li>
</ul>
Both designs released so far follow a model where each contains three clusters, each with its own distinct function.  A management plane that is separate from the user workload (compute) virtual machines. They also leverages an edge cluster, which provides dedicated resources for network services such as VMware NSX Edge devices, which provide access to the corporate network and the Internet.  These designs are formed on VMware virtual SAN storage and use vRealize Log Insight and vRealize Operations Manager products for management.

Alongside these 'VMware Validated Design' is launched 'Certified Partner Architecture Program' and these programs work together. Certified Partner Architecture is aso new program, which certifies partners SDDC architectures to ensure they are in compliance with established VMware Validated Designs and best practices. This program will allow partners to:
<ul>
	<li>Formally certify their own Software-Defined Data Center architecture designs and best practices as aligning with the VMware Validated Designs and best practices.</li>
	<li>Ensure that their architectures remain compliant with the VMware Validated Designs and best practices.</li>
	<li>Upon completion of the certification process, the partner’s architecture will then be eligible for the VMware Ready logo in the Certified Partner Architecture category and listing on the VMware Compatibility Guide (VCG).</li>
</ul>
