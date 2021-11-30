+++
title = "Tanzu Kubernetes Grid Integrated (TKGi) with HAProxy"
date = "2021-05-10"
description = "Tanzu Kubernetes Grid Integrated (TKGi) with HAProxy"
tags = [
    "tanzu",
    "kubernetes",
    "vsphere"
]
categories = [
    "homelab"
]
thumbnail = "clarity-icons/code-144.svg"
+++
VMware Tanzu is offered in three editions Basic, Standard and Advanced. Each edition is a superset of the one before it along a spectrum.

* [Tanzu Basic](https://tanzu.vmware.com/tanzu/basic) offers capabilities to  run Kubernetes in vSphere.
* [Tanzu Standard](https://tanzu.vmware.com/tanzu/standard) offers capabilities to run and manage Kubernetes across multiple clouds.
* [Tanzu Advanced](https://tanzu.vmware.com/tanzu/advanced) offers capabilities to simplify and secure the container lifecycle at scale-and speed app delivery

Tanzu Kubernetes cluster networking offers various deployment options. The clusters can be deployed with CNI defined as Calico or Antrea (default). The load balancer service resource can be configured as NSX-T load balancer, NSX Advanced Load Balancer or HAProxy. It is not oppinionated on ingress controller so any can be used. Here I am looking at setting up and understanding the lowest cost option of Tanzu Basic edition with "bring your own" networking. The summary of component choices I made for this deployment:

| Endpoint | Provider | Description |
| --- | --- | --- |
| Pod connectivity | Antrea | Container network interface for pods |
| Service type: ClusterIP | Antrea | Only accessible from within the cluster |
| Service type: NodePort | Antrea | External access through a port opened on each worker node |
| Service type: LoadBalancer | HAProxy | External access through a port with virtual IP address |
| Cluster ingress | Traefik | Routing for inbound pod traffic |

## Content Libraries

To Tanzu Kubernetes requires software respository to store source files required for the build. Tanzu Kubernetes with "bring your own" networking requires HAProxy source files. Both of these source file storage requirements can be met with Content Libary.

Create a Subscribed Content Library named TanzuKubernetesRelease on the lab vCenter which is a subscriber of https://wp-content.vmware.com/v2/latest/lib.json. Create a Local Content Library named named InstallSource on the lab vCenter. Download the latest version of the VMware HAProxy OVA file from the [VMware-HAProxy site](https://github.com/haproxytech/vmware-haproxy#download). Import the downloaded HAProxy OVA to the InstallSource Content Library.

## Network Topology

One kay aspect to getting the solution working is having a clear understanding of its networking.

* Management network is used for administrative SSH access ( 192.168.10.0/24 VLAN 150 )
* HA Proxy Management - 192.168.10.185
* HA Proxy Dataplane - 192.168.1.25
* Loadbalancer Address 192.168.1.48/28 (usable 192.168.1.49-62)
* Supervisor VM Range 192.168.10.192 /28 (usable 192.168.10.193-206)
* Worker network is used for workload access  (192.168.1.0 /24 VLAN 152 )

![Tanzu lab  with 'Bring your own networking'](/images/tanzu-basic-byo.drawio.svg)

## HAProxy

The Supervisor Cluster is a highly available configuration to faciliate this a LoadBalancer service is consumed. To deploy the Supervisor Cluster the chosen LoadBalancer needs to be available. To this end I deployed HAProxy appliance from the OVA imported to Content Library with the following settings:

| Parameter | Value |
| --- | --- |
| OVA Version | 0.2.0 |
| VM Name | haproxy |
| Configuration | Default 2 NIC |
| Management Network | VLAN-150 |
| Workload Network | VLAN-152 |
| Frontend Network | VLAN-152 |
| Permit Root Login | True |
| Hostname | haproxy.cork.local |
| DNS Server | 192.168.10.10 |
| Management IP Addr | 192.168.10.185/24 |
| Management Gateway | 192.168.10.254 |
| Workload IP Addr | 192.168.1.25/24 |
| Workload Gateway | 192.168.1.254 |
| Load Balancer IP Range | 192.168.1.33/28 |
| UserID | admin |

Once deployed power on the applliance. SSH to the HAProxy VM as root and make a copy of /etc/haproxy/ca.crt contents.

## Supervisor Cluster

The Supervisor Cluster feature adds a capability to vCenter to deploy Tanzu Kubernetes clusters. To enable the feature navigate to the Workload Management feature and start the wizard. In this lab I configured with following values:

| Parameter | Sub-parameter| Sub-parameter |  Value |
| --- | --- | --- | --- |
| vCenter Server Version ||| 7.0 Update 2a |
| vCenter Server and Network | Network stack option || vCenter Server Network |
| Control Plane Size ||| Medium |
| Storage | Control Plane Nodes || vSAN Default Storage Policy |
| Load Balancer | Name || haproxy |
| Load Balancer | Type || HAProxy |
| Load Balancer | Data Plane API Addr || 192.168.10.185:5556 |
| Load Balancer | User name || admin |
| Load Balancer | Virtual IP Range || 192.168.1.33 - 192.168.1.46 |
| Load Balancer | Server Certificate Authority || (copied contents of HAProxy /etc/haproxy/ca.crt ) |
| Mangement Network | Network || VLAN-150 |
| Mangement Network | Starting IP Address || 192.168.10.193 |
| Mangement Network | Subnet Mask || 255.255.255.0 |
| Mangement Network | Gateway || 192.168.10.254 |
| Mangement Network | DNS Server || 192.168.10.10 |
| Mangement Network | DNS Search Domains || cork.local |
| Mangement Network | NTP Server || 192.168.10.10 |
| Workload Network | IP Address for Services || 10.96.0.0/23 |
| Workload Network | DNS Server || 192.168.10.10 |
| Workload Network | Workload Network # 1 | Name | network-1 |
| Workload Network | Workload Network # 1 | Port Group | VLAN-152 |
| Workload Network | Workload Network # 1 | Gateway | 192.168.1.254 |
| Workload Network | Workload Network # 1 | Subnet | 255.255.255.0 |
| Workload Network | Workload Network # 1 | IP Addr Range | 192.168.1.33 - 192.168.1.46 |
| Tanzu Kubernetes Grid | Content Library || TanzuKubernetesRelease |

While the installation runs progress and any errors can be viewed in log files on the vCenter appliance.

```bash
tail -f /var/log/vmware/wcp/wcpsvc.log | grep error
```

Viewing the newly created cluster I can see the 'Control Plane Node IP Address' allocated by Loadbalancer is 192.168.1.52.

## Namespace

Once the supervisor cluster is in place we can create a namespace 'application1'. With the namespace in place enables UI link to download vSphere Plugin for kubectl. For me the link returned error 'An error occurred during a connection to 192.168.1.52. PR_END_OF_FILE_ERROR'.

HAProxy is a simple load balancer which stores its configuration in a file /etc/haproxy/haproxy.cfg.

```bash
haproxy -c -f /etc/haproxy/haproxy.cfg

Configuration file is valid

cat /etc/haproxy/haproxy.cfg

....
frontend domain-c8:53e819c5-f517-46c0-996d-90a73ea84310-kube-system-kube-apiserver-lb-svc
  mode tcp
  bind 192.168.1.52:443 name domain-c8:53e819c5-f517-46c0-996d-90a73ea84310-kube-system-kube-apiserver-lb-svc-192.168.1.52:nginx
  bind 192.168.1.52:6443 name domain-c8:53e819c5-f517-46c0-996d-90a73ea84310-kube-system-kube-apiserver-lb-svc-192.168.1.52:kube-apiserver
  log-tag domain-c8:53e819c5-f517-46c0-996d-90a73ea84310-kube-system-kube-apiserver-lb-svc
  option tcplog
  use_backend domain-c8:53e819c5-f517-46c0-996d-90a73ea84310-kube-system-kube-apiserver-lb-svc-nginx if { dst_port 443 }  use_backend domain-c8:53e819c5-f517-46c0-996d-90a73ea84310-kube-system-kube-apiserver-lb-svc-kube-apiserver if { dst_port 6443 }

backend domain-c8:53e819c5-f517-46c0-996d-90a73ea84310-kube-system-kube-apiserver-lb-svc-kube-apiserver
  mode tcp
  balance roundrobin
  option tcp-check
  log-tag domain-c8:53e819c5-f517-46c0-996d-90a73ea84310-kube-system-kube-apiserver-lb-svc-kube-apiserver
  server domain-c8:53e819c5-f517-46c0-996d-90a73ea84310-kube-system-kube-apiserver-lb-svc-192.168.1.49:6443 192.168.1.49:6443 check-ssl weight 100 verify none
  server domain-c8:53e819c5-f517-46c0-996d-90a73ea84310-kube-system-kube-apiserver-lb-svc-192.168.1.50:6443 192.168.1.50:6443 check-ssl weight 100 verify none
  server domain-c8:53e819c5-f517-46c0-996d-90a73ea84310-kube-system-kube-apiserver-lb-svc-192.168.1.51:6443 192.168.1.51:6443 check-ssl weight 100 verify none

backend domain-c8:53e819c5-f517-46c0-996d-90a73ea84310-kube-system-kube-apiserver-lb-svc-nginx
  mode tcp
  balance roundrobin
  option tcp-check
  log-tag domain-c8:53e819c5-f517-46c0-996d-90a73ea84310-kube-system-kube-apiserver-lb-svc-nginx
  server domain-c8:53e819c5-f517-46c0-996d-90a73ea84310-kube-system-kube-apiserver-lb-svc-192.168.1.49:443 192.168.1.49:443 check-ssl weight 100 verify none
  server domain-c8:53e819c5-f517-46c0-996d-90a73ea84310-kube-system-kube-apiserver-lb-svc-192.168.1.50:443 192.168.1.50:443 check-ssl weight 100 verify none
  server domain-c8:53e819c5-f517-46c0-996d-90a73ea84310-kube-system-kube-apiserver-lb-svc-192.168.1.51:443 192.168.1.51:443 check-ssl weight 100 verify none
```

The configuration file apepars valid and shows my HTTPS request via port 443 round robins around the three cluster node member addresses. Browsing any node addresses https://192.168.1.49-51 directly took me to the vSphere Plugin download page.

HAProxy runs as a service so I queried its state and found it was not running.

```bash
systemctl status haproxy.service -l --no-pager

● haproxy.service - HAProxy Load Balancer
   Loaded: loaded (/lib/systemd/system/haproxy.service; enabled; vendor preset: enabled)
  Drop-In: /etc/systemd/system/haproxy.service.d
           └─cloud-init.conf, slice.conf
   Active: failed (Result: exit-code) since Wed 2021-08-04 14:05:26 UTC; 7min ago
  Process: 854 ExecStart=/usr/sbin/haproxy -Ws -f $CONFIG -p $PIDFILE $EXTRAOPTS (code=exited, status=1/FAILURE)
  Process: 851 ExecStartPre=/usr/sbin/haproxy -f $CONFIG -c -q $EXTRAOPTS (code=exited, status=0/SUCCESS)
 Main PID: 854 (code=exited, status=1/FAILURE)

Aug 04 14:05:26 haproxy.cork.local systemd[1]: Failed to start HAProxy Load Balancer.
Aug 04 14:05:26 haproxy.cork.local systemd[1]: haproxy.service: Service RestartSec=100ms expired, scheduling restart.
Aug 04 14:05:26 haproxy.cork.local systemd[1]: haproxy.service: Scheduled restart job, restart counter is at 5.
Aug 04 14:05:26 haproxy.cork.local systemd[1]: Stopped HAProxy Load Balancer.
Aug 04 14:05:26 haproxy.cork.local systemd[1]: haproxy.service: Start request repeated too quickly.
Aug 04 14:05:26 haproxy.cork.local systemd[1]: haproxy.service: Failed with result 'exit-code'.
Aug 04 14:05:26 haproxy.cork.local systemd[1]: Failed to start HAProxy Load Balancer.
Aug 04 14:05:56 haproxy.cork.local systemd[1]: haproxy.service: Unit cannot be reloaded because it is inactive.
Aug 04 14:09:26 haproxy.cork.local systemd[1]: haproxy.service: Unit cannot be reloaded because it is inactive.
Aug 04 14:10:56 haproxy.cork.local systemd[1]: haproxy.service: Unit cannot be reloaded because it is inactive.
```

Checking the log it looks like the bindings of sockets cannot be completed.

```bash
journalctl -u haproxy.service --since=today | grep bind

Aug 04 14:05:25 haproxy.cork.local haproxy[829]: [ALERT] 215/140525 (829) : Starting frontend domain-c8:53e819c5-f517-46c0-996d-90a73ea84310-kube-system-kube-apiserver-lb-svc: cannot
bind socket [192.168.1.35:443]
Aug 04 14:05:25 haproxy.cork.local haproxy[829]: [ALERT] 215/140525 (829) : Starting frontend domain-c8:53e819c5-f517-46c0-996d-90a73ea84310-kube-system-kube-apiserver-lb-svc: cannot
bind socket [192.168.1.35:6443]
Aug 04 14:05:26 haproxy.cork.local haproxy[847]: [ALERT] 215/140526 (847) : Starting frontend domain-c8:53e819c5-f517-46c0-996d-90a73ea84310-kube-system-kube-apiserver-lb-svc: cannot
bind socket [192.168.1.35:443]
Aug 04 14:05:26 haproxy.cork.local haproxy[847]: [ALERT] 215/140526 (847) : Starting frontend domain-c8:53e819c5-f517-46c0-996d-90a73ea84310-kube-system-kube-apiserver-lb-svc: cannot
bind socket [192.168.1.35:6443]
Aug 04 14:05:26 haproxy.cork.local haproxy[854]: [ALERT] 215/140526 (854) : Starting frontend domain-c8:53e819c5-f517-46c0-996d-90a73ea84310-kube-system-kube-apiserver-lb-svc: cannot
bind socket [192.168.1.35:443]
Aug 04 14:05:26 haproxy.cork.local haproxy[854]: [ALERT] 215/140526 (854) : Starting frontend domain-c8:53e819c5-f517-46c0-996d-90a73ea84310-kube-system-kube-apiserver-lb-svc: cannot
bind socket [192.168.1.35:6443]
```

This issue appears specific to OVA Version 0.2.0,  reverting to older 0.1.10 with identical configuration works.

The vSphere Plugin for kubectl allows authentication with vCenter Single Sign-On credentials. We can test this is working by adding SSO user with rights to namespace. I added administrator@vsphere.local to the namespace role Owner.


```bash
set KUBECTL_VSPHERE_PASSWORD=<use your password>
kubectl vsphere login --server=https://192.168.1.36/ --insecure-skip-tls-verify --vsphere-username=administrator@vsphere.local
```

Interestingly here this fails I receive another unexpected occurance.

```bash
time="2021-08-04T18:10:56+01:00" level=warning msg="Error occurred during HTTP request: Post \"https://192.168.1.52/wcp/login\": EOF"
time="2021-08-04T18:10:56+01:00" level=error msg="Login failed: Post \"https://192.168.1.52/wcp/login\": EOF"
time="2021-08-04T18:10:56+01:00" level=warning msg="Login failed for workload/cluster {application1 192.168.1.52}\n"
Not all cluster/workload sessions were established. Login failed for the following clusters/workloads:
   application1

You have access to the following contexts:
   192.168.1.49

If the context you wish to use is not in this list, you may need to try
logging in again later, or contact your cluster administrator.

To change context, use `kubectl config use-context <workload name>`
```

