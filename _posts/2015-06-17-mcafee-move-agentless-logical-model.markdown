---
layout: post
title:  "McAfee MOVE Agentless Logical Model"
date:   2015-06-17 16:59:56 +0100
tags:
    - VMware
permalink: mcafee-move-agentless-logical-model/
---
McAfee Management for Optimized Virtual Environments AntiVirus (McAfee MOVE AntiVirus) is an anti-virus solution 
for virtual environments. It removes the need to install an anti-virus application on every virtual machine (VM), 
yet provides the protection and performance adequate for your organization requirements.

Component parts of McAfee MOVE and what they do.

<center><img src="/images/McAfee-MOVE.jpg" width="50%"></center>

Each McAfee MOVE component performs specific functions to keep your environment protected.
<ul>
	<li><strong>McAfee ePolicy Orchestrator (ePO)</strong> — Allows you to configure policies to manage McAfee MOVE AV Agentless and provides reports on malware discovered within your virtual environment.</li>
	<li><strong>File Quarantine</strong> — Remote quarantine system, where quarantined files are stored on an administrator-specified network share.</li>
	<li><strong>GTI (Global Threat Intelligence)</strong> — Classifies suspicious files that are found on the file system. When the real-time malware defense detects a suspicious program, it sends a DNS request for analysis to a central database server hosted by McAfee Labs.</li>
	<li><strong>Security Virtual Appliance (SVA)</strong> — Provides anti-virus protection for VMs and communicates with the loadable kernel module on the hypervisor, ePolicy Orchestrator, and the GTI servers. The SVA is the only system directly managed by ePolicy Orchestrator.</li>
	<li><strong>VMware NSX Manager</strong> — Console that allows you to configure, provision, and automate the protection on the endpoints in a data center.</li>
</ul>
The McAfee MOVE SVA is imported into McAfee EPO,  NSX Manager is then registered with McAfee EPO,  the McAfee MOVE SVA can then be deployed via NSX Manager. There are four main parts of the SVA,  the AV scanner the Engine,  a Shared Cache and the DAT virus definition file.  The AV scanner within the MOVE SVA communicates with vShield Endpoint kernel module on each ESXi host.

<center><img src="/images/McAfee-MOVE-Flow.jpg" width="50%"></center>