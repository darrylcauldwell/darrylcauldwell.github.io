+++
title = "Managing Clusters Using Tanzu Mission Control"
date = "2021-12-10"
description = "Managing Clusters Using Tanzu Mission Control"
tags = [
    "tanzu",
    "kubernetes",
    "vsphere",
    "tmc"
]
categories = [
    "homelab"
]
thumbnail = "clarity-icons/code-144.svg"
+++
Third in a series of posts which build on each other looking Tanzu.

1. [Deploying Tanzu for vSphere with NSX-T](/post/tanzu-basic-nsx)
2. [Managing Tanzu Kubernetes Clusters Using ClusterAPI](/post/tanzu-tkc)
3. [Managing Tanzu Kubernetes Clusters Using Misson Control](/post/tanzu-tmc)

I am moving next to look at Tanzu Mission Control. The Mission Control inventory has a hierachy, first I'll create a group to place the clusters in. While adding the group add some metadata to group including a label so other users know who to contact with any queries.

![Tanzu Mission Control Cluster Group](/images/tanzu-tmc-cluster-group.png)

With the group in place we can look at attaching an existing cluster to TMC.  Following the wizard the first step is to enter some metadata for the cluster being attached.

![Tanzu Mission Control Cluster Metadata](/images/tanzu-tmc-cluster-meta.png)

anzu Misson Control does not need to connect inwards to the cluster to be managed.  Instead agents are deployed and configured on the cluster which initiate outbound connection to Tanzu Misson Control. Outbound internet communications are sometimes via proxy, if required add proxy details next.

![Tanzu Mission Control Cluster Proxy](/images/tanzu-tmc-cluster-proxy.png)

When the deployed agents initiate communications with TMC they require to advertise who they are and be associated with the right objects. The agent configuration is supplied via UI as YAML. A URL is supplied to the configuration on internet. The administrator of this cluster is Hermione so I connect with her credentials and apply the yaml file to deploy and configure the agents.

```bash
kubectl vsphere login --server=172.16.1.2 --tanzu-kubernetes-cluster-name hogwarts-cluster01 --tanzu-kubernetes-cluster-namespace hogwarts --vsphere-username hermione@vsphere.local --insecure-skip-tls-verify

kubectl create -f "https://tanzuemea.tmc.cloud.vmware.com/installer?id=f1d706f38d6f3ad834314a941831b190e18508468ade1adf2e105a822c8c0b49&source=attach"
```

![Tanzu Mission Control Cluster kubectl](/images/tanzu-tmc-cluster-kubectl.png)

Once the agent has be deployed can move back to TMC. The agents take a couple of minutes to initialize and gather data but in very short order can then view the cluster and extension health in TMC.

![Tanzu Mission Control Cluster Health](/images/tanzu-tmc-cluster-health.png)

To unlock more power of TMC we can register the TKG management cluster. The process is similar to attaching subordinate cluster but is slightly different as before we start with adding some metadata.

![Tanzu Mission Control Supervisor Cluster Metadata](/images/tanzu-tmc-cluster-sup-meta.png)

Again here I am not using proxy so just skip through next page.

![Tanzu Mission Control Supervisor Cluster Proxy](/images/tanzu-tmc-cluster-sup-proxy.png)

Rather than being presented with a yaml file to apply with kubectl cmd here the wizard outputs a URL link which contains config.

![Tanzu Mission Control Supervisor Cluster Config URL](/images/tanzu-tmc-cluster-sup-URL.png)

We then have to create environmentally specific yaml file which includes namespace and the supplied URL. This needs to be ran in the context of the cluster itself so we switch to that and then get the namespace name prefixed with svc-tmc.

```bash
kubectl vsphere login --server=172.16.1.2 --vsphere-username administrator@vsphere.local --insecure-skip-tls-verify
Logged in successfully.
You have access to the following contexts:
   172.16.1.2
   hogsmead
   hogwarts
If the context you wish to use is not in this list, you may need to try
logging in again later, or contact your cluster administrator.
To change context, use `kubectl config use-context <workload name>`

kubectl config use-context 172.16.1.2
Switched to context "172.16.1.2".

kubectl get ns | grep svc-tmc
NAME                                        STATUS   AGE
svc-tmc-c34                                 Active   11d
```

With the namespace name and URL we can populate the yaml file to apply agent and config.

```yaml
apiVersion: installers.tmc.cloud.vmware.com/v1alpha1
kind: AgentInstall
metadata:
  name: tmc-agent-installer-config
  namespace: svc-tmc-c34 
spec:
  operation: INSTALL
  registrationLink: https://tanzuemea.tmc.cloud.vmware.com/installer?id=0f25b71394ac623867f64138ed9d27bb0db5e8d0436e874e4f65ff179e7be807&source=registration&type=tkgs
```

We can then apply the configuration using kubectl and the agents get created with correct configuration.

```bash
kubectl create -f tmc-registration.yaml
agentinstall.installers.tmc.cloud.vmware.com/tmc-agent-installer-config created
```

![Tanzu Mission Control Supervisor Cluster kubectl](/images/tanzu-tmc-cluster-sup-kubectl.png)

After a couple of minutes when the agents have started and have reported back we can get a view of supervisor cluster health.

![Tanzu Mission Control Supervisor Cluster Health](/images/tanzu-tmc-cluster-sup-health.png)

With the supervisor cluster registered we can start to some interesting things from TMC like provision a new cluster. To recap at this stage my deployment has two namespaces configured. Only Hogwarts has a workload cluster deployed.

* Hogwarts
  * hogwarts-cluster-01
* Hogsmead

Starting the TMC wizard to create cluster first prompt is for the provider. This is analagous to namespace so we'll look at creating cluster under Hogsmead.

![Tanzu Mission Control Cluster Provising Provider](/images/tanzu-tmc-cluster-prov-provider.png)

Then add some cluster metadata.

![Tanzu Mission Control Cluster Provising Metadata](/images/tanzu-tmc-cluster-prov-meta.png)

Can then pick a version of Kubernetes which the cluster will run, the Pod and Service networking configuration and storage class.

![Tanzu Mission Control Cluster Provising Config](/images/tanzu-tmc-cluster-prov-config.png)

Followed by choosing the VM class, hardware specifcation of the cluster nodes.

![Tanzu Mission Control Cluster Provising VM Class](/images/tanzu-tmc-cluster-prov-vmclass.png)

Finally the node pool configuration.

![Tanzu Mission Control Cluster Provising Node Pool](/images/tanzu-tmc-cluster-prov-nodepool.png)

It takes a few minutes to deploy the required VMs and configure the Kubernetes cluster.  You can get a view on how far it is along by looking in vCenter.

![Tanzu Mission Control Cluster Provising vCenter](/images/tanzu-tmc-cluster-prov-vcenter.png)

When deploying cluster from TMC the agents and configuration to register the cluster as with TMC is included.  After a few minutes when VMs are built and configured the cluster shows in UI.

![Tanzu Mission Control Cluster Provising Registered](/images/tanzu-tmc-cluster-prov-registered.png)

As well as provisioning TMC can also help with day two operations. When deploying the Hogsmead cluster I selected an v1.20.12 version of k8s. Upgrading to latest can be initiated very easily. Within the cluster view just hit Upgrade and select from drop down of all newer versions the version you need.

![Tanzu Mission Control Cluster Lifecycle Management](/images/tanzu-tmc-cluster-lcm.png)

A couple of minutes later and can see its now running v1.21.6.

![Tanzu Mission Control Cluster Lifecycle Management Complete](/images/tanzu-tmc-cluster-lcm-version.png)
