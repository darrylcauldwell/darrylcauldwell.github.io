+++
title = "Managing Tanzu for vSphere Clusters Using ClusterAPI"
date = "2021-12-03"
description = "Managing Tanzu for vSphere Clusters Using ClusterAPI"
tags = [
    "tanzu",
    "kubernetes",
    "vsphere",
    "nsx-t"
]
categories = [
    "homelab"
]
thumbnail = "clarity-icons/code-144.svg"
+++
Second in a series of posts which build on each other looking Tanzu.

1. [Deploying Tanzu for vSphere with NSX-T](/post/tanzu-basic-nsx)
2. [Managing Tanzu for vSphere Clusters Using ClusterAPI](/post/tanzu-tkc)
3. [Managing Tanzu for vSphere Clusters Using Tanzu Misson Control](/post/tanzu-tmc)
4. [Deploying Tanzu for AWS Using Tanzu Misson Control](/post/tanzu-tmc-aws)

Clusters are created within organisational construct known as a namespace. The namespace configuration can be used to map user/groups to role/permission it uses the same users/groups available as vSphere Identity source, these can be aligned to one of three roles view, edit and owner. The namespace configuration also defines which vSphere storage policies can be aligned to namespace and the amount of available CPU, Memory and Storage can be limited. I have a single cluster prepared with the following Namespace configuration:

| Namespace | User | Role | Storage Policy | Limit |
| --- | --- | --- | --- | --- |
| Hogwarts | hermione@vsphere.local | Owner | vSAN Default Storage Policy | Unlimited |
| Hogwarts | harry@vsphere.local | Edit | vSAN Default Storage Policy | Unlimited |
| Hogwarts | ron@vsphere.local | View | vSAN Default Storage Policy | Unlimited |
| Hogsmead | hermione@vsphere.local | Edit | vSAN Default Storage Policy | Unlimited |
| Hogsmead | harry@vsphere.local | Owner | vSAN Default Storage Policy | Unlimited |
| Hogsmead | ron@vsphere.local | View | vSAN Default Storage Policy | Unlimited |

With these in place I can attempt to connect as Hermione and check access to Hippogriff.

```bash
set KUBECTL_VSPHERE_PASSWORD=VMware1!
kubectl vsphere login --server=172.16.1.2 --vsphere-username hermione@vsphere.local --insecure-skip-tls-verify 

Logged in successfully.
You have access to the following contexts:
   172.16.1.2
   hogsmead
   hogwarts
If the context you wish to use is not in this list, you may need to try
logging in again later, or contact your cluster administrator.
To change context, use `kubectl config use-context <workload name>`

kubectl config use-context hogwarts
```

To deploy a cluster you need to specify certain attributes such as which image and Kubernetes version to deploy and the dimensions of the nodes. The node images are stored in vSphere Content Library and some standard ones are available via a publiclu content library subscription. The VM hardware specification dimensions of node are defined as virtual machine classes. The VM storage location are defined as storage classes. The names of available images, vmclasses and storageclasses available to be deployed from supervisor can be easily checked.

```bash
kubectl get tanzukubernetesreleases
NAME                                VERSION                          READY   COMPATIBLE   CREATED   UPDATES AVAILABLE
v1.16.12---vmware.1-tkg.1.da7afe7   1.16.12+vmware.1-tkg.1.da7afe7   True    True         6m28s     [1.17.17+vmware.1-tkg.1.d44d45a 1.16.14+vmware.1-tkg.1.ada4837]
v1.16.14---vmware.1-tkg.1.ada4837   1.16.14+vmware.1-tkg.1.ada4837   True    True         6m25s     [1.17.17+vmware.1-tkg.1.d44d45a]
v1.16.8---vmware.1-tkg.3.60d2ffd    1.16.8+vmware.1-tkg.3.60d2ffd    False   False        6m33s
v1.17.11---vmware.1-tkg.1.15f1e18   1.17.11+vmware.1-tkg.1.15f1e18   True    True         6m29s     [1.18.19+vmware.1-tkg.1.17af790 1.17.17+vmware.1-tkg.1.d44d45a]
v1.17.11---vmware.1-tkg.2.ad3d374   1.17.11+vmware.1-tkg.2.ad3d374   True    True         6m28s     [1.18.19+vmware.1-tkg.1.17af790 1.17.17+vmware.1-tkg.1.d44d45a]
v1.17.13---vmware.1-tkg.2.2c133ed   1.17.13+vmware.1-tkg.2.2c133ed   True    True         6m30s     [1.18.19+vmware.1-tkg.1.17af790 1.17.17+vmware.1-tkg.1.d44d45a]
v1.17.17---vmware.1-tkg.1.d44d45a   1.17.17+vmware.1-tkg.1.d44d45a   True    True         6m27s     [1.18.19+vmware.1-tkg.1.17af790]
v1.17.7---vmware.1-tkg.1.154236c    1.17.7+vmware.1-tkg.1.154236c    True    True         6m24s     [1.18.19+vmware.1-tkg.1.17af790 1.17.17+vmware.1-tkg.1.d44d45a]
v1.17.8---vmware.1-tkg.1.5417466    1.17.8+vmware.1-tkg.1.5417466    True    True         6m28s     [1.18.19+vmware.1-tkg.1.17af790 1.17.17+vmware.1-tkg.1.d44d45a]
v1.18.10---vmware.1-tkg.1.3a6cd48   1.18.10+vmware.1-tkg.1.3a6cd48   True    True         6m30s     [1.19.14+vmware.1-tkg.1.8753786 1.18.19+vmware.1-tkg.1.17af790]
v1.18.15---vmware.1-tkg.1.600e412   1.18.15+vmware.1-tkg.1.600e412   True    True         6m27s     [1.19.14+vmware.1-tkg.1.8753786 1.18.19+vmware.1-tkg.1.17af790]
v1.18.15---vmware.1-tkg.2.ebf6117   1.18.15+vmware.1-tkg.2.ebf6117   True    True         6m29s     [1.19.14+vmware.1-tkg.1.8753786 1.18.19+vmware.1-tkg.1.17af790]
v1.18.19---vmware.1-tkg.1.17af790   1.18.19+vmware.1-tkg.1.17af790   True    True         6m27s     [1.19.14+vmware.1-tkg.1.8753786]
v1.18.5---vmware.1-tkg.1.c40d30d    1.18.5+vmware.1-tkg.1.c40d30d    True    True         6m30s     [1.19.14+vmware.1-tkg.1.8753786 1.18.19+vmware.1-tkg.1.17af790]
v1.19.11---vmware.1-tkg.1.9d9b236   1.19.11+vmware.1-tkg.1.9d9b236   True    True         6m33s     [1.20.9+vmware.1-tkg.1.a4cee5b 1.19.14+vmware.1-tkg.1.8753786]
v1.19.14---vmware.1-tkg.1.8753786   1.19.14+vmware.1-tkg.1.8753786   True    True         6m29s     [1.20.9+vmware.1-tkg.1.a4cee5b]
v1.19.7---vmware.1-tkg.1.fc82c41    1.19.7+vmware.1-tkg.1.fc82c41    True    True         6m26s     [1.20.9+vmware.1-tkg.1.a4cee5b 1.19.14+vmware.1-tkg.1.8753786]
v1.19.7---vmware.1-tkg.2.f52f85a    1.19.7+vmware.1-tkg.2.f52f85a    True    True         6m26s     [1.20.9+vmware.1-tkg.1.a4cee5b 1.19.14+vmware.1-tkg.1.8753786]
v1.20.2---vmware.1-tkg.1.1d4f79a    1.20.2+vmware.1-tkg.1.1d4f79a    True    True         6m26s     [1.21.2+vmware.1-tkg.1.ee25d55 1.20.9+vmware.1-tkg.1.a4cee5b]
v1.20.2---vmware.1-tkg.2.3e10706    1.20.2+vmware.1-tkg.2.3e10706    True    True         6m33s     [1.20.9+vmware.1-tkg.1.a4cee5b]
v1.20.7---vmware.1-tkg.1.7fb9067    1.20.7+vmware.1-tkg.1.7fb9067    True    True         6m24s     [1.21.2+vmware.1-tkg.1.ee25d55 1.20.9+vmware.1-tkg.1.a4cee5b]
v1.20.8---vmware.1-tkg.2            1.20.8+vmware.1-tkg.2            True    True         6m25s
v1.20.9---vmware.1-tkg.1.a4cee5b    1.20.9+vmware.1-tkg.1.a4cee5b    True    True         6m31s     [1.21.2+vmware.1-tkg.1.ee25d55]
v1.21.2---vmware.1-tkg.1.ee25d55    1.21.2+vmware.1-tkg.1.ee25d55    True    True         6m31s

kubectl get vmclass
NAME                  CPU   MEMORY   AGE
best-effort-2xlarge   8     64Gi     80s
best-effort-4xlarge   16    128Gi    80s
best-effort-8xlarge   32    128Gi    80s
best-effort-large     4     16Gi     80s
best-effort-medium    2     8Gi      78s
best-effort-small     2     4Gi      80s
best-effort-xlarge    4     32Gi     80s
best-effort-xsmall    2     2Gi      77s
guaranteed-2xlarge    8     64Gi     79s
guaranteed-4xlarge    16    128Gi    74s

kubectl get storageclass -n hogwarts
NAME                          PROVISIONER              RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
vsan-default-storage-policy   csi.vsphere.vmware.com   Delete          Immediate           true                   19m
```

Using the output from previous commands we can copy paste the appropriate storage class, vm class and Kuberneters version values from what is available in the namespace into a yaml file definition for the cluster.

```yaml
apiVersion: run.tanzu.vmware.com/v1alpha2
kind: TanzuKubernetesCluster
metadata:
  name: hogwarts-cluster01
  namespace: hogwarts
spec:
  topology:
    controlPlane:
      replicas: 3
      vmClass: best-effort-small
      storageClass: vsan-default-storage-policy
      tkr:  
        reference:
          name: v1.21.2---vmware.1-tkg.1.ee25d55
    nodePools:
    - name: forbidden-forest
      replicas: 3
      vmClass: best-effort-small
      storageClass: vsan-default-storage-policy
      tkr:  
        reference:
          name: v1.21.2---vmware.1-tkg.1.ee25d55
  settings:
    network:
        cni:
          name: antrea
        services:
          cidrBlocks: ["10.96.0.0/12"]
        pods:
          cidrBlocks: ["172.16.128.0/17"]
        serviceDomain: hogwarts.local
```

With all of this in place we can deploy the cluster,  we can check it has been deployed correctly.

```bash
kubectl apply -f hogwarts-cls1.yml
tanzukubernetescluster.run.tanzu.vmware.com/hogwarts-cluster01 created

kubectl get TanzuKubernetesCluster
NAMESPACE   NAME                 CONTROL PLANE   WORKER   TKR NAME                           AGE   READY   TKR COMPATIBLE   UPDATES AVAILABLE
hogwarts    hogwarts-cluster01   3               3        v1.21.2---vmware.1-tkg.1.ee25d55   32m   True    True

kubectl get virtualmachines
NAME                                                         POWERSTATE   AGE
hogwarts-cluster01-control-plane-9tljd                       poweredOn    35m
hogwarts-cluster01-control-plane-cn89v                       poweredOn    37m
hogwarts-cluster01-control-plane-wfnjg                       poweredOn    41m
hogwarts-cluster01-forbidden-forest-hdqms-7b9bf6bb44-fqw2d   poweredOn    38m
hogwarts-cluster01-forbidden-forest-hdqms-7b9bf6bb44-n2r5s   poweredOn    38m
hogwarts-cluster01-forbidden-forest-hdqms-7b9bf6bb44-vbnnj   poweredOn    38m

kubectl get service
NAME                                               TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE
service/hogwarts-cluster01-control-plane-service   LoadBalancer   10.96.0.231   172.16.1.3    6443:32176/TCP   44m
```

If I swing over to take a look what has happened in NSX-T a second segment has been added to the T1 router which was created during the supervisor cluster enablement. The segment has connected the three control plane and three node pool VMs. An NSX Load Balancer is deployed to represent the Kubernetes API service resource.  There are also NAT Rules to represent cluster Egress via 172.16.2.2 IP address. All of the created NSX entities have the namespace name in this case 'hogwarts' in their description they are really easy to identify via search.

So all looks good and can try and connect to the subordinate cluster and interact with it.

```bash
kubectl vsphere login --server=172.16.1.2 --tanzu-kubernetes-cluster-name hogwarts-cluster01 --tanzu-kubernetes-cluster-namespace hogwarts --vsphere-username hermione@vsphere.local --insecure-skip-tls-verify

Logged in successfully.
You have access to the following contexts:
   172.16.1.2
   hogsmead
   hogwarts
   hogwarts-cluster01
If the context you wish to use is not in this list, you may need to try logging in again later, or contact your cluster administrator.
To change context, use `kubectl config use-context <workload name>`

kubectl config use-context hogwarts-cluster01

kubectl get nodes
NAME                                                         STATUS   ROLES                  AGE   VERSION
hogwarts-cluster01-control-plane-9tljd                       Ready    control-plane,master   58m   v1.21.2+vmware.1
hogwarts-cluster01-control-plane-cn89v                       Ready    control-plane,master   61m   v1.21.2+vmware.1
hogwarts-cluster01-control-plane-wfnjg                       Ready    control-plane,master   64m   v1.21.2+vmware.1
hogwarts-cluster01-forbidden-forest-hdqms-7b9bf6bb44-fqw2d   Ready    <none>                 62m   v1.21.2+vmware.1
hogwarts-cluster01-forbidden-forest-hdqms-7b9bf6bb44-n2r5s   Ready    <none>                 62m   v1.21.2+vmware.1
hogwarts-cluster01-forbidden-forest-hdqms-7b9bf6bb44-vbnnj   Ready    <none>                 62m   v1.21.2+vmware.1
```

Next step is to deploy first Pod into the cluster, but this got stuck ContainerCreating and Pod seems to show an Antrea socket error.

```bash
kubectl run -i --tty busybox --image=quay.io/quay/busybox --restart=Never -- sh

kubectl get pods
NAME      READY   STATUS              RESTARTS   AGE
busybox   0/1     ContainerCreating   0          3m12s

kubectl describe pod busybox
Events:
  Type     Reason                  Age                From               Message
  ----     ------                  ----               ----               -------
  Normal   Scheduled               3m55s              default-scheduler  Successfully assigned default/busybox to hogwarts-cluster01-forbidden-forest-8g2s2-66d6bfdd-8wzbr
  Warning  FailedCreatePodSandBox  3m52s              kubelet            Failed to create pod sandbox: rpc error: code = Unknown desc = failed to setup network for sandbox "354f63b2c94a0fb5a88c9611adb3008c7dd564deb157a39ebeca78f1b717316c": rpc error: code = Unavailable desc = connection error: desc = "transport: Error while dialing dial unix /var/run/antrea/cni.sock: connect: connection refused"
```

So first thing to look at I guess is Antrea Pods and can see one is broken.  Checking logs and it looks like OVS timed out on startup.

```bash
kubectl get pods -A
NAMESPACE                      NAME                                                                       READY   STATUS              RESTARTS   AGE
default                        busybox                                                                    0/1     ContainerCreating   0          12m
kube-system                    antrea-agent-8pf5t                                                         0/2     CrashLoopBackOff    26         96m
kube-system                    antrea-agent-bv9rq                                                         2/2     Running             1          102m
kube-system                    antrea-agent-cth2b                                                         2/2     Running             0          102m
kube-system                    antrea-agent-p6qfq                                                         2/2     Running             1          96m
kube-system                    antrea-agent-wh8k4                                                         2/2     Running             0          91m
kube-system                    antrea-agent-wsrtg                                                         2/2     Running             0          99m

kubectl logs antrea-agent-8pf5t -n kube-system antrea-ovs
[[1;34m2021-12-03T20:08:00Z [0;32mINFO [0;36mantrea-ovs[0m]: Starting ovsdb-server
Starting ovsdb-server.
2021-12-03T20:09:05Z|00002|timeval|WARN|Unreasonably long 10026ms poll interval (7883ms user, 82ms system)
2021-12-03T20:09:05Z|00003|timeval|WARN|faults: 57 minor, 0 major
2021-12-03T20:09:05Z|00004|timeval|WARN|context switches: 0 voluntary, 9309 involuntary
2021-12-03T20:09:17Z|00002|timeval|WARN|Unreasonably long 8797ms poll interval (7858ms user, 89ms system)
2021-12-03T20:09:17Z|00003|timeval|WARN|faults: 60 minor, 0 major
2021-12-03T20:09:17Z|00004|timeval|WARN|context switches: 0 voluntary, 9264 involuntary
Configuring Open vSwitch system IDs.
/usr/share/openvswitch/scripts/ovs-ctl: line 42: hostname: command not found
Enabling remote OVSDB managers.
[[1;34m2021-12-03T20:09:19Z [0;32mINFO [0;36mantrea-ovs[0m]: Stopping OVS before quit
[[1;34m2021-12-03T20:09:19Z [0;32mINFO [0;36mantrea-ovs[0m]: Stopping OVS
ovs-vswitchd is not running.
2021-12-03T20:09:21Z|00001|fatal_signal|WARN|terminating with signal 14 (Alarm clock)
```

As this was a timeout and that the cluster is very easy to remove and redeploy I tried this and with exact same configuration this issue resolved. And we can finally run our pod and run some commands!

```bash
kubectl config use-context hogwarts
kubectl delete -f hogwarts-cls1b.yml
kubectl apply -f hogwarts-cls1b.yml

kubectl vsphere login --server=172.16.1.2 --tanzu-kubernetes-cluster-name hogwarts-cluster01 --tanzu-kubernetes-cluster-namespace hogwarts --vsphere-username hermione@vsphere.local --insecure-skip-tls-verify
Logged in successfully.
You have access to the following contexts:
   172.16.1.2
   hogsmead
   hogwarts
   hogwarts-cluster01
If the context you wish to use is not in this list, you may need to try logging in again later, or contact your cluster administrator.
To change context, use `kubectl config use-context <workload name>`

kubectl config use-context hogwarts-cluster01

kubectl run -i --tty busybox --image=quay.io/quay/busybox --restart=Never -- sh
If you don't see a command prompt, try pressing enter.
/ # nslookup bbc.co.uk
Server:         10.96.0.10
Address:        10.96.0.10:53

Non-authoritative answer:
Name:	bbc.co.uk
Address: 151.101.128.81
Name:	bbc.co.uk
Address: 151.101.0.81
Name:	bbc.co.uk
Address: 151.101.192.81
Name:	bbc.co.uk
Address: 151.101.64.81

/ #
```
