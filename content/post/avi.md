+++
title = "Avi Load Balancer"
date = "2020-08-12"
description = "Avi Load Balancer"
tags = [
    "avi",
    "load balancer"
]
categories = [
    "networking"
]
thumbnail = "clarity-icons/code-144.svg"
featured = true
+++
Here we will explore an example of how an application can be secured using NSX Advanced Load Balancer. The application is deployed to Cloud Foundation on-premises and is extended to other geographic regions using both Amazon Web Services and Azure public clouds. The application is two tier, Ant Media servers provide a user facing presentation tier and MooseFS servers provide storage.

![Multi-Cloud Ant Media](/images/avi-1.jpg)

# What is a layer 7 load balancer?

Traditional load balancers operate at layer 4 and offers traffic management of transactions at the network protocol layer (TCP/UDP). A layer 4 load balancer distributes traffic to pool members with a load balancing algorithm (i.e. round-robin) and by calculating the best server based on fewest connections and fastest server response time.

The Avi Load Balancer operates at layer 7 which means you can make routing decisions on various characteristics of the HTTP/HTTPS header, the content of the message, the URL type, and information in cookies. A device that performs Layer 7 load balancing is sometimes referred to as a reverse proxy server or in Kubernetes terminology an ingress controller.

# Avi Load Balancer Architecture

The Avi Load Balancer (ALB) uses a software-defined architecture that separates the central control plane (Controller) from the distributed data plane (Service Engines).

![Avi Load Balancer Architecture](/images/avi-2.jpg)

The Controller acts as a single point of intelligence, management, and control for the data plane. This control plane can be self-hosted or consumed as managed software-as-a-service (SaaS). When deploying the software-as-a-service architecture model the data plane remains deployed in each environment.

The Service Engines represent the data-plane and provide full-featured, enterprise-grade load balancers, WAF, or analytics that manage and secure application traffic, and collect real-time telemetry from the traffic flows. Virtual Services are deployed to the data-plane which provide virtual IP address to a pool of application servers. Pools maintain the list of servers assigned to them and perform health monitoring, load balancing, persistence, web application firewall and SSL offload. A typical virtual service will point to one pool; however, more advanced configurations may have a virtual service content switching across multiple pools.

The Controllers need to continually exchange information securely with Service Engines. The Service Engines hold communications channel with controller over TCP port 22 (SSH), 8443 (HTTPS) and UDP port 123 (NTP).

![Service Engine to Cntroller Communications](/images/avi-3.jpg)

# Web Application Firewall

A web application firewall (or WAF) filters, monitors, and blocks HTTP traffic to and from a web application. A network firewall filters traffic at Layer 4 a web application firewall filters traffic at Layer 7. A web application firewall is a type of reverse-proxy, protecting the server from exposure by having clients pass through the WAF before reaching the server. The capability to inspecting HTTP traffic allows filtering rules which can prevent attacks stemming from web application security flaws, such as SQL injection, cross-site scripting (XSS), file inclusion, and security misconfigurations.

# Avi Load Balancer iWAF

The Avi Load Balancer provides capability to configure Virtual Services with Intelligent Web Application Firewall (iWAF) capabilities.

The WAF configuration is stored as profile and policy objects. These objects can be logically associated to one or many virtual services. Using this approach, an application specific profile and policy can be created and maintained which is logically to any Virtual Services in any cloud.

The WAF can be set to either detect or enforce a ruleset. In detection mode the policy will evaluate the request and log request matching ruleset, in enforcement mode the policy will evaluate the request, block and log request matching ruleset. Paranoia mode can be set for each WAF policy which defines its rigidity. Specific rules are enabled or disabled based on the set paranoia mode.

![Avi Load Balancer iWAF](/images/avi-4.jpg)

The iWAF comes with a ruleset which provides signatures to provide Core Rule Set (CRS) protection, support for compliance regulations such as PCI DSS, HIPAA, and GDPR.

# Demonstrating WAF Ruleset

Core Rule Set protection primarily consists of regular expressions, and it decides for each request whether it is legitimate, an attack, or an information leak. There are various commercial and open source tools to test if WAFs are working. The testing tools can target many aspects of the WAF ruleset. It is possible to perform basic test using a web browser and look to target CRS rule 920350 ‘Host header is numeric IP address’. To be valid the Host header field must be sent in all HTTP/1.1 request messages. When the rule is enabled any traffic to IP address would be caught and only access via FQDN would be allowed.

![CRS rule 920350](/images/avi-5.jpg)

1. Within the default ‘System-WAF-Policy’ this signature is disabled by default. We cannot edit the default policy but if we duplicate it and name like ‘ANT-Media-WAF-Policy’ and change policy mode to Enforcement. Ensure the CRS 920350 signature check is disabled (default).

2. With this policy applied to a Virtual Service we can direct browser to website via IP or FQDN and web page is displayed.

3. When we update the policy and enable the CRS 920350 signature check requests via IP are blocked and web page can only be accessed via FQDN.

# Demonstrating WAF Logging and Exception

When a WAF policy is attached to a virtual service, specific WAF logs are generated. When WAF blocks traffic the request details get logged.

![WAF Block Detected Log Graph](/images/avi-6.jpg)

It maybe that a rule is implemented which has unintended consequences and is blocking traffic we want to allow. Clicking on the + sign at the end of each log entry will expand the panel to provide more details. If we look through the extended details, we can see specific WAF signature which caused the log entry. There is also a button to add an WAF policy exception directly from the log view.

![WAF Block Detected Log Detail](/images/avi-7.jpg)

This opens a wizard where we can confirm the details of the exception.

![Add Exception](/images/avi-8.jpg)

Making an exception in this way alters the behaviour of policy immediately, in our example to allow access via IP address. In the policy itself the exception is clearly marked which can allow retrospective security change records to be created.

![Exception Highlighted](/images/avi-9.jpg)

# Lab Hybrid Cloud Networking

The lab configuration used for testing was comprised of a Cloud Foundation on-premises cloud,  AWS and Azure. The Cloud Foundation deployment hosts an Ubuntu VM with SSL CPN client installed and is configured to maintain VPN to both AWS and Azure. Static routing is configured, and Virtual Machines and Virtual Services on the Cloud Foundation network can communicate with Virtual Machines and Virtual Services in both public clouds.

![Multi-Cloud Site-to-Site VPNs](/images/avi-10.jpg)

Multi-cloud and hybrid cloud are closely related but are not the same. A multi-cloud approach would be hosting some services on-premises and other services in public cloud and the services operate independently. A hybrid-cloud approach would be hosting services across multiple clouds but having them integrated, communicating and exchanges data with each other. When integrating services applying consistent configuration is key, integrating services across multiple clouds is challenging.

# Demonstrating Hybrid Cloud

Another use case we can explorer is where an on-premises application is made available in its local geography but also made available in other geographies. Provide application proxy services adjacent to clients to improve performance and security.

![Edge Proxy](/images/avi-11.jpg)

For this scenario we can deploy Virtual Service on-premises which has business application as pool members. We can deploy additional Virtual Services in public cloud which have on-premises Virtual Service as pool member. The network routing across private and public clouds is all within VPN tunnel.

The separation of NSX Advanced Load Balancer control and data plane allows us to manage the Virtual Services and Web Application Firewalls deployed to multiple clouds as a hybrid-cloud. The WAF profile and policy are defined as logical objects within the control plane. These objects then have logical association to Virtual Services deployed to private and public cloud data planes. We can apply the same policy as in previous test to the Virtual Services across clouds and test the WAF rules function exactly the same.

# Application Latency Analysis

When operating web applications, it is essential to understand where network latency is introduced to the user experience. Out of the box when a Virtual Service is deployed you can a very easy to understand view of end-to-end latency.

![Application Latency Analysis](/images/avi-12.jpg)

# Traffic Capture

When operating web applications, it good to understand the traffic flowing towards it. Typically to achieve this packet capture is required to be turned on at network side,  for whole VM or specific VM NIC. For NSX Advanced Load Balancer any Virtual Service can have packet capture turned on and packets captured as they flow through the Virtual Service which can be used to analysed to get a security insight.

![Traffic Capture](/images/avi-13.jpg)

The packets are captured in PCAP Next Generation file format ( *.pcapng files ) so any analyser can be used to open and view.

![Traffic Capture Analysis](/images/avi-14.jpg)
