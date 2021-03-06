+++
title = "What is Virtustream xStream Cloud"
date = "2015-07-21"
description = "What is Virtustream xStream Cloud"
tags = [
    "virtustream"
]
+++
If you are an IT consumer with Enterprise workload it is a risk moving this workload to the cloud,  how can you guarantee your potential future cloud provider is delivering you the service you require and once there how can you measure how well they are achieving delivering that service.

## What Is A Micro VM (uVM)

A Micro VM (uVM) is a measure mechanism used to define the resource consumption of a collection of running virtual machine,  it does not matter if the the virtual machine could be running on vSphere or KVM,  and it does not mater what operating systems the virtual machine is running be it Windows \ Linux.

The virtual machine resource usage is measured in the Micro VM unit,  the standard unit size used here is

* 1 CPU Unit = 200MHz
* 1 RAM Unit = 768MB
* 1 IO Unit (Storage or Network) = 40 IO
* 1 Bandwidth Unit = 2 Mb/s

If you take a collection of servers running differing workloads which combine to provide an application service you can use a consistent measurement metric across these to measure the resources required to deliver that service. So whether it is a a single app and single database server,  or it maybe a hundred different servers running a hundred different pieces of different software, the running capacity it consumes can be given as a consistent metric.

## What Is Virtustream xStream
Virtustream is a IaaS cloud provider as well they have produce software called xStream which they use to monitor and measure there own IaaS cloud,  but they also offer this product to measure and monitor other clouds (private, public or hybrid).

With a consistent independent metric of a uVM established you can create boundaries around normal usage range and factor in peak workload to form logical application resource groups, these resource groups can then be measured against service level agreements to ensure the cloud provider is delivering the resources.