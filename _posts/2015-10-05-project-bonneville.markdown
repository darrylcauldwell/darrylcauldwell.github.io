---
layout: post
title:  "Project Bonneville - vSphere Integrated Containers"
date:   2015-10-06 16:59:56 +0100
tags:
    - Docker
permalink: project-bonneville/
---
Working for a company which develops software for business I've been keen on following the story of containers unfold. The explosion of containers and microservices in the software development space has been a revolution in the way applications can be architected and deployed. This has put a great deal of power in the hands of the developer but with great power comes great responsibility for operational management.

Working in the operational side the key words to my working life are Availability, Reliability, Maintainability, Supportability and Security. The operational model to effectively run containers in production requires a paradigm shift where the approach and underlying assumptions are revisited. Working in a company promoting DevOps culture and values we in the operational side are beginning to gain greater empathy to the challenges a developer faces and how they believe containers will help. Gaining a technical understanding of Docker has therefore become a priority so I can ensure the infrastructure I design and engineer can help the developer deliver not only velocity but help them to deliver Availability, Reliability, Maintainability, Supportability and Security.

During <a href="http://darrylcauldwell.com/vmware-projects-fargo-vmfork-meteor/" target="_blank">VMworld 2014 project Fargo (VMfork)</a> was described which has become the Instant Clone feature of vSphere 6. At the time I didn't take too much notice as this was signed to virtual desktop space.  During April this year VMware released project Photon which is essentially a minimal Linux container host,  at the time this was confusing to me as RedHat Atomic, Ubuntu Snappy and CoreOS seemed to have all bases covered in this space. That is until two months later when Ben Corrie of VMware announced project Bonneville to the world.

The fundamental to this solution is a 1:1 model between container host and containers rather than one host with many containers. Project Bonneville is a Docker daemon that has custom drivers for networks and execution that is compatible with the Docker Client APIs, and that means a vSphere VM is treated exactly like a Docker container. With this it redefined a Docker container as a x86 hardware virtualised VM, (not tied to host running the Linux kernel), a Bonneville VM can theoretically run containers of any kernel version of any operating system and can do it fast and efficiently.

‘Instant Clone’ also known as VMfork or Project Fargo, gives the ability to clone and deploy virtual machines, as much as 10x faster than what is currently possible before. It does this by using rapid in-memory cloning of running virtual machines and copy-on-write to quickly deploy clones of a parent virtual machine. Using this clone method of the container host ensures the only changed blocks and memory pages are those used by container itself this.

A fantastic architecture flow of how Bonneville might work is described in this diagram as posted by George Hicken @hickeng.

<center><img src="/images/vSphere-Integrated-Containers.png" width="50%"></center>

The benefits this technology appears to offer appear fantastic and I am really looking forwards to attending Ben Corrie's session 'INF5229 — Docker and Fargo: Exploding the Linux Container Host' to solidify my understanding and learn more about how we can begin exploiting this to help our developers deliver Availability, Reliability, Maintainability, Supportability and Security alongside the awesome features they write.
