+++
title = "Solar Powered Raspberry Pi Cluster"
date = "2021-07-09"
description = "Solar Powered Raspberry Pi Cluster"
tags = [
    "raspberry",
    "power"
]
categories = [
    "homelab",
    "raspberry",
    "sustainability'
]
thumbnail = "clarity-icons/code-144.svg"
+++
I am interested in technology and run various compute workloads from home. I have been running a [cluster of three Intel NUCs](/post/homelab-nuc/) running as a vSphere cluster hosting VMs. Each Intel NUC has peak power draw of 60W, the switch has peak power draw 28W and NAS has peak power draw 60W so total peak power draw 268W.

Most of the workload I was running could run on Arm or Intel. I looked at optimizing power draw by building a [found host Raspberry Pi cluster](/post/homelab-pi/) running [Kubernetes hosting containers](/post/homelab-pi-microk8s/). Each Raspberry Pi has peak power draw 15W,  these are powered by a PoE switch so total including Pi's is constrained to its peak power draw of 60W.  As the power comes via the switch it is easy to meaure the total draw with a power meter,  interestingly my peak draw does 33W which looks to occurs at boot and then when all containers are scheduled runs ~22W.

![Homelab Power Draw](/images/homelab-pi-solar-draw.png)

The surprisingly lower than planned power draw is excellent but also made me consider if I could optimize further and have it run from solar energy!

## Solar Solution

After a little reading the main components I need are:

* Solar Panel
* Solar Charger
* Battery
* DC AC Inverter

![Homelab Solar Solution](/images/homelab-pi-solar-overview.drawio.png)

Victron was a brand which kept cropping up as a platform which offers and API and rich data stream.


