+++
title = "Raspberry Pi Kubernetes Cluster"
date = "2021-05-22"
description = "Raspberry Pi Kubernetes Cluster"
tags = [
    "raspberry",
    "kubernetes"
]
categories = [
    "homelab",
    "raspberry"
]
thumbnail = "clarity-icons/code-144.svg"
+++
Here I am looking at setting up my Raspberry Pi cluster to host the various Docker containers I run at home. I am using [four Raspberry Pi 4B](/post/homelab-pi/) which are all build with Ubuntu 21.04 and configured as per [Ubuntu Base configuration](/post/homelab-pi-ubuntu/).

There are various ways to setup a Kubernetes cluster including:
* Manually
* Kubeadm
* Rancher K3S
* microk8s

## Manual
When learning Kubernetes it is useful practise to configure all components manually. Often people follow the tutorial [Kubernetes The Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way). For this installation I want to get up and running quickly so will be looking at automated build solution.

## Kubeadm
The official CNCF tool for provisioning Kubernetes clusters in a variety of shapes and forms e.g. single-node, multi-node, HA, self-hosted.  It's as close to a "vanilla" Kubernetes you get in production but consumes a lot of resources.

## Rancher K3S OR Canonical Microk8s
K3s and Microk8s are both lightweight implementation of Kubernetes they have various differences but a key one to understand is high availablity approach.

| Function | MicroK8s | K3s |
| --- | --- | --- |
| Kubernetes Node Roles | Every node is a worker node | Agent Node(s) |
| Kubernetes API Server | Every node is an API server | Server Node (s) |
| Kubernetes Datastore | Dqlite | etcd |
| Datastore HA | Embedded Dqlite when >3 nodes | Requires External datastore |

I want my four node cluster to be highly available as easily as possible so for this deployment I chose Mircok8s.

## Visual Studio Code Remote SSH
The four Raspberry Pi which make up my cluster are headless. The Visual Studio Code [Remote Development extension pack](https://code.visualstudio.com/docs/remote/remote-overview) makes configuring headless systems very easy.

## Enable c-groups
Edit `/boot/firmware/cmdline.txt` and add following options:

```
cgroup_enable=memory cgroup_memory=1
```

![Microk8s cgroup](/images/homelab-pi-microk8s-cgroup.png)

Reboot `sudo reboot` for changes to take effect.

## Install Microk8s
MicroK8s is supplied as a snap, there are various Kubernetes releases these are available as snap channels.  To see all available versions we can query what channels are available. When I'm running this the current Kubernetes version is 1.21 and I filter results on this we can see all.


```
snap info microk8s | grep 1.21

latest/stable:    v1.21.1  2021-05-17 (2215) 168MB classic
latest/candidate: v1.21.1  2021-05-13 (2214) 168MB classic
latest/beta:      v1.21.1  2021-05-13 (2214) 168MB classic
latest/edge:      v1.21.1  2021-05-19 (2227) 168MB classic
1.21/stable:      v1.21.1  2021-05-17 (2215) 168MB classic
1.21/candidate:   v1.21.1  2021-05-14 (2215) 168MB classic
1.21/beta:        v1.21.1  2021-05-14 (2215) 168MB classic
1.21/edge:        v1.21.1  2021-05-20 (2232) 168MB classic
1.14/stable:      v1.14.10 2019-12-20 (1121) 164MB classic
```

I'll look to install the stable release of 1.21 on each using:

```
sudo snap install microk8s --classic --channel=1.21/stable
```

When the snap is loaded and running on all nodes we can look to form them into a cluster.  On first node run following and it will generate a token to run on remote node to add it.

```
## From first node in cluster generate token  
sudo microk8s add-node

From the node you wish to join to this cluster, run the following:
microk8s join 192.168.1.100:25000/73d707e38ddb7c7bbcf29a328a505179/7e8d2c9a15af
```

Then on remote node run command with generated token.

```
sudo microk8s join 192.168.1.100:25000/73d707e38ddb7c7bbcf29a328a505179/7e8d2c9a15af

Contacting cluster at 192.168.1.100
Waiting for this node to finish joining the cluster. ..  
```

Repeat this process (generate a token, run it from the joining node) for the third and forth nodes.  When all done from any node we can query the cluster state see all four nodes and check high availability status.

```
sudo microk8s status --wait-readymicrok8s is running

high-availability: yes
  datastore master nodes: 192.168.1.100:19001 192.168.1.101:19001 192.168.1.102:19001
  datastore standby nodes: 192.168.1.103:19001
addons:
  enabled:
    ha-cluster           # Configure high availability on the current node
  disabled:
    dashboard            # The Kubernetes dashboard
    dns                  # CoreDNS
    helm                 # Helm 2 - the package manager for Kubernetes
    helm3                # Helm 3 - Kubernetes package manager
    host-access          # Allow Pods connecting to Host services smoothly
    ingress              # Ingress controller for external access
    linkerd              # Linkerd is a service mesh for Kubernetes and other frameworks
    metallb              # Loadbalancer for your Kubernetes cluster
    metrics-server       # K8s Metrics Server for API access to service metrics
    portainer            # Portainer UI for your Kubernetes cluster
    prometheus           # Prometheus operator for monitoring and logging
    rbac                 # Role-Based Access Control for authorisation
    registry             # Private image registry exposed on localhost:32000
    storage              # Storage class; allocates storage from host directory
    traefik              # traefik Ingress controller for external access
```

## Install kubectl
The Microk8s install deploys its own client and to execute have to remember to prefix everything with microk8s. I have to switch between systems often and to avoid confusion prefer to install normal kubectl. Again this is available as a snap in multiple versions.

```
snap info kubectl | grep 1.21

latest/stable:    1.21.1         2021-05-14 (1970)  9MB classic
latest/candidate: 1.21.1         2021-05-14 (1970)  9MB classic
latest/beta:      1.21.1         2021-05-14 (1970)  9MB classic
latest/edge:      1.21.1         2021-05-14 (1970)  9MB classic
1.21/stable:      1.21.1         2021-05-13 (1970)  9MB classic
1.21/candidate:   1.21.1         2021-05-13 (1970)  9MB classic
1.21/beta:        1.21.1         2021-05-13 (1970)  9MB classic
1.21/edge:        1.21.1         2021-05-13 (1970)  9MB classic
```

We want to install version to match Kubernetes and copy the Microk8s kubectl config to be kubectl default config.

```
sudo snap install kubectl --classic --channel=1.21/stable
mkdir ~/.kube
sudo microk8s config > ~/.kube/config
```

## Memory Footprint
One of the benefits I hoped to realize by using Microk8s was low memory footprint.  We can see here our 8GB Raspberry Pi4B with Microk8s running it still shows 6GB available. 

```
free -m
              total        used        free      shared  buff/cache   available
Mem:           7809        1173        4718           4        1918        6200
Swap:             0           0           0
```

So all in all very easy to setup and get a low footprint highly available Kubernetes cluster running on Raspberry Pi cluster with Microk8s.