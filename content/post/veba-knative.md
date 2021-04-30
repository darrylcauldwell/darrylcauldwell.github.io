+++
title = "VMware Event Broker Appliance (VEBA) - Knative"
date = "2021-04-30"
description = "VMware Event Broker Appliance (VEBA) - Knative"
tags = [
    "kubernetes",
    "knative",
    "veba",
    "vsphere"
]
categories = [
    "vsphere"
]
thumbnail = "clarity-icons/code-144.svg"
+++

The VMware Event Broker Appliance (VEBA) aims to facilitate event-driven automation based on vCenter Server events.

## AWS Lambda and Step Functions

When I worked as an architect working with AWS I used event-driven automation with AWS Lambda to integrate distributed systems. Some more complex requirements orchestration, maintaining state and sequencing of multiple functions achieved with AWS Step Functions. This event-driven automation allowed complex systems to be put in place very simply.

## Knative

Since getting engaged with the Kubernetes community it seemed the biggest barrier to entry for most people was complexity.  Knative looks to obfuscate some of that complexity and provide an extraction that aims to allow more focus on business functionality. Looking a little deeper Knative eventing components appears to provide many parallels with AWS Lambda and Step Functions.

## VMware Event Broker Appliance

VMware provides the VMware Event Broker Appliance as a [fling](https://flings.vmware.com/vmware-event-broker-appliance). The [system architecture](https://vmweventbroker.io/kb/architecture) shows that the appliance is built on a Photon OS running Kubernetes with Contour acting as ingress controller. The event router appears to be composed of two components an event stream source that maintains a connection to vCenter Server and a choice of event stream processor Knative, OpenFaaS or AWS EventBridge. With version 0.6 the default deployment delivers embedded Knative.

### vCenter Event Provider Config

The deployment of the appliance takes vCenter Server details as input and configures the provider. The [vcenter provider specification](https://github.com/vmware-samples/vcenter-event-broker-appliance/tree/development/vmware-event-router#provider-type-vcenter) has mostly intuitive naming. What wasn't immediately obvious to me was the checkpoint parameters it appears this is to control whether at-least-once delivery (true) or at-most-once delivery (false). I hadn't seen an option to configure this during OVA deployment but did some digging and found deployment appears to create a Kubernetes manifest file.

```yaml
## SSH to VEBA
cat config/event-router-config.yml

apiVersion: event-router.vmware.com/v1alpha1
kind: RouterConfig
metadata:
  name: router-config-knative
eventProcessor:
  name: veba-knative
  type: knative
  knative:
    insecureSSL: false
    encoding: binary
    destination:
      ref:
        apiVersion: eventing.knative.dev/v1
        kind: Broker
        name: default
        namespace: vmware-functions
eventProvider:
  name: veba-vc-01
  type: vcenter
  vcenter:
    address: https://<MY VCENTER FQDN>/sdk
    auth:
      basicAuth:
        password: "<MY PASSWORD>"
        username: "administrator@vsphere.local"
      type: basic_auth
    insecureSSL: true
    checkpoint: false
metricsProvider:
  default:
    bindAddress: 0.0.0.0:8082
  name: veba-metrics
  type: default
```

We can see that default appears to be at-most-once event delivery. Once delivered, there is no chance of delivering again. If the consumer is unable to handle the message due to some exception, the message is lost.

## Knative Event Processor Config

Within the same manifest, a knative event processor is also defined.

```yaml
eventProcessor:
  name: veba-knative
  type: knative
  knative:
    insecureSSL: false
    encoding: binary
    destination:
      ref:
        apiVersion: eventing.knative.dev/v1
        kind: Broker
        name: default
        namespace: vmware-functions
```

## Kubernetes Pods

It is worth taking a moment to look at the pods and relative namespaces.

```bash
kubectl get pods --all-namespaces

NAMESPACE            NAME                                          READY   STATUS      RESTARTS   AGE
contour-external     contour-5869594b-m8ttg                        1/1     Running     0          4d4h
contour-external     contour-5869594b-s57c6                        1/1     Running     0          4d4h
contour-external     contour-certgen-v1.10.0-nj5gh                 0/1     Completed   0          4d4h
contour-external     envoy-ccp68                                   2/2     Running     0          4d4h
contour-internal     contour-5d47766fd8-kcnt5                      1/1     Running     0          4d4h
contour-internal     contour-5d47766fd8-p7ztf                      1/1     Running     0          4d4h
contour-internal     contour-certgen-v1.10.0-r672b                 0/1     Completed   0          4d4h
contour-internal     envoy-nt9d7                                   2/2     Running     0          4d4h
knative-eventing     eventing-controller-658f454d9d-4xzcw          1/1     Running     0          4d4h
knative-eventing     eventing-webhook-69fdcdf8d4-mhlh6             1/1     Running     0          4d4h
knative-eventing     rabbitmq-broker-controller-88fc96b44-l6bp2    1/1     Running     0          4d4h
knative-serving      activator-85cd6f6f9-qcvkm                     1/1     Running     0          4d4h
knative-serving      autoscaler-7959969587-v72dr                   1/1     Running     0          4d4h
knative-serving      contour-ingress-controller-6d5777577c-5zvxt   1/1     Running     0          4d4h
knative-serving      controller-577558f799-hktb7                   1/1     Running     0          4d4h
knative-serving      webhook-78f446786-htjfc                       1/1     Running     0          4d4h
kube-system          antrea-agent-q8jxm                            2/2     Running     0          4d4h
kube-system          antrea-controller-849fff8c5d-7g49p            1/1     Running     0          4d4h
kube-system          coredns-74ff55c5b-ljfjq                       1/1     Running     0          4d4h
kube-system          coredns-74ff55c5b-rbpfm                       1/1     Running     0          4d4h
kube-system          etcd-veba.cork.local                          1/1     Running     0          4d4h
kube-system          kube-apiserver-veba.cork.local                1/1     Running     0          4d4h
kube-system          kube-controller-manager-veba.cork.local       1/1     Running     0          4d4h
kube-system          kube-proxy-trt4x                              1/1     Running     0          4d4h
kube-system          kube-scheduler-veba.cork.local                1/1     Running     0          4d4h
local-path-storage   local-path-provisioner-5696dbb894-n5lw9       1/1     Running     0          4d4h
rabbitmq-system      rabbitmq-cluster-operator-7bbbb8d559-cth75    1/1     Running     0          4d4h
vmware-functions     default-broker-ingress-5c98bf68bc-whmj4       1/1     Running     0          4d4h
vmware-functions     sockeye-65697bdfc4-n8ght                      1/1     Running     0          4d4h
vmware-functions     sockeye-trigger-dispatcher-5fff8567fc-9v74l   1/1     Running     0          4d4h
vmware-system        tinywww-dd88dc7db-q57rl                       1/1     Running     0          4d4h
vmware-system        veba-rabbit-server-0                          1/1     Running     0          4d4h
vmware-system        vmware-event-router-ccdfdc67f-t8wzs           1/1     Running     3          4d4h
```

As well as the vmware-functions namespace it is worth taking a look at vmware-system namespace. We can see a pod named vmware-event-router..... so we can take a look at its properties and logs.

```bash
kubectl get pod vmware-event-router-ccdfdc67f-t8wzs --namespace vmware-system -o json
kubectl logs vmware-event-router-ccdfdc67f-t8wzs --namespace vmware-system
```

## Knative Eventing Anatomy

Knative Broker and Trigger objects make it easy to filter events based on event attributes. A Broker provides a bucket of events which can be selected by attribute. It receives events and forwards them to subscribers defined by one or more matching Triggers.

![Broker Trigger Architecture](/images/veba-knative-broker-trigger-overview.svg)

## Consuming Example Knative Function

The GitHub repository contains a folder containing [example Knative functions](https://github.com/vmware-samples/vcenter-event-broker-appliance/tree/development/examples/knative).

One example is a Powershell function which triggers echo response.

### Knative Service
The Knative service definition kn-service.yaml

```yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: kn-ps-echo
  labels:
    app: veba-ui
spec:
  template:
    metadata:
      annotations:
        autoscaling.knative.dev/maxScale: "1"
        autoscaling.knative.dev/minScale: "1"
    spec:
      containers:
        - image: projects.registry.vmware.com/veba/kn-ps-echo:1.0
```

We can see the service definition pulls a container from the VMware public container registry named 'kn-ps-echo' version 1.0.  The Dockerfile to create the container reads:

```dockerfile
FROM photon:3.0
ENV TERM linux
ENV PORT 8080

# Set terminal. If we don't do this, weird readline things happen.
RUN echo "/usr/bin/pwsh" >> /etc/shells && \
    echo "/bin/pwsh" >> /etc/shells && \
    tdnf install -y powershell-7.0.3-2.ph3 unzip && \
    pwsh -c "Set-PSRepository -Name PSGallery -InstallationPolicy Trusted" && \
    find / -name "net45" | xargs rm -rf && \
    tdnf erase -y unzip && \
    tdnf clean all
RUN pwsh  -Command 'Install-Module ThreadJob -Force -Confirm:$false'
RUN pwsh -Command 'Install-Module -Name CloudEvents.Sdk'

COPY server.ps1 ./
COPY handler.ps1 handler.ps1

CMD ["pwsh","./server.ps1"]
```

We can see the Dockerfile defines the Powershell environment but the entrypoint is to run server.ps1 which starts a CloudEvent HTTP listener and if we look within that it calls handler.ps1 which in this case utputs event data.  Both of these Powershell scripts are copied into the container at point of creation.

### Knative Trigger Definition

A Knative Trigger defines the vCenter Server events to subscribe from a given broker. By default, no filter is applied so all events are subscribed. The trigger defines the service as a subscriber based on service name.

```yaml
## kn-trigger.yaml
apiVersion: eventing.knative.dev/v1
kind: Trigger
metadata:
  name: veba-ps-echo-trigger
  labels:
    app: veba-ui
spec:
  broker: default
  subscriber:
    ref:
      apiVersion: serving.knative.dev/v1
      kind: Service
      name: kn-ps-echo
```

If event filtering is required it can be configured like.

```yaml
## kn-trigger.yaml
apiVersion: eventing.knative.dev/v1
kind: Trigger
metadata:
  name: veba-ps-echo-trigger
  labels:
    app: veba-ui
spec:
  broker: default
  filter:
    attributes:
        type: com.vmware.event.router/event
        subject: VmPoweredOffEvent
  subscriber:
    ref:
      apiVersion: serving.knative.dev/v1
      kind: Service
      name: kn-ps-echo
```

### Testing Example Knative Function

Pull and execute the manifest file.

```bash
curl -O https://github.com/vmware-samples/vcenter-event-broker-appliance/blob/development/examples/knative/powershell/kn-ps-echo/function.yaml
kubectl apply --filename function.yaml --namespace vmware-functions

service.serving.knative.dev/kn-ps-echo created
trigger.eventing.knative.dev/kn-ps-echo-trigger created
```

We can then check the vmware-functions namespace to get the pod name. There are two containers in the pod, the user-container runs the function so we can follow its logs and see the flow of vCenter events being echo'd.

```bash
kubectl get pods --namespace vmware-functions

NAME                                             READY   STATUS    RESTARTS   AGE
default-broker-ingress-5c98bf68bc-whmj4          1/1     Running   0          4d6h
kn-ps-echo-00001-deployment-6c9f77855c-ddz8w     2/2     Running   0          18m
kn-ps-echo-trigger-dispatcher-7bc8f78d48-5cwc7   1/1     Running   0          18m
sockeye-65697bdfc4-n8ght                         1/1     Running   0          4d6h
sockeye-trigger-dispatcher-5fff8567fc-9v74l      1/1     Running   0          4d6h

kubectl logs --namespace vmware-functions kn-ps-echo-00001-deployment-6c9f77855c-ddz8w user-container --follow

Cloud Event
  Source: https://vcenter.cork.local/sdk
  Type: com.vmware.event.router/event
  Subject: UserLoginSessionEvent
  Id: caf397c6-3188-4100-882a-982dd6abf239
CloudEvent Data:
```

## Next Steps

Thinking forwards to next steps it should be possible to reuse majority of this Powershell example and simply adjust trigger filter and modify contents of handler.ps1 to perform desired onward action.