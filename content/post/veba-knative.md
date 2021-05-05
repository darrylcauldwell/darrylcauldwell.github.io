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

When I worked as an architect working with AWS I used event-driven automation with AWS Lambda to integrate distributed systems. This event-driven automation allowed me to put complex systems in place very simply. The VMware Event Broker Appliance (VEBA) aims to facilitate event-driven automation based on vCenter Server events.

## VMware Event Broker Appliance

VMware provides the VMware Event Broker Appliance as a [fling](https://flings.vmware.com/vmware-event-broker-appliance). The [system architecture](https://vmweventbroker.io/kb/architecture) shows that the appliance is built on a Photon OS running Kubernetes with Contour acting as ingress controller. The event broker appliance is composed of two components an event router and a choice of event stream processor Knative, OpenFaaS or AWS EventBridge.

### Knative Eventing Configuration

Since getting engaged with the Kubernetes community it seemed the biggest barrier to entry for most people was complexity.  Knative looks to obfuscate some of that complexity and provide an abstraction that allows more focus on business functionality. It offers two core functions:

* Serving - Run serverless containers on Kubernetes
* Eventing - Universal subscription, delivery, and management of events

Knative Eventing is composed of Knative Broker and Trigger objects which make it easy to filter events based on event attributes. A Broker provides a bucket of events which can be selected by attribute. It receives events and forwards them to subscribers defined by one or more matching Triggers.

![Broker Trigger Architecture](/images/veba-knative-broker-trigger-overview.svg)

The v0.6 default install generates configuration for binding to vCenter Server with embedded Knative eventing 'config/event-router-config.yml':

```yaml
## SSH to appliance
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

## Consuming Example Knative Function

The GitHub repository contains a folder containing [example Knative functions](https://github.com/vmware-samples/vcenter-event-broker-appliance/tree/development/examples/knative).

If we look at the example function which triggers each of each all events to Pod console. To achieve this it defines a Knative eventing resource which has an unfiltered trigger on the default broker and defines Service named 'kn-ps-echo' as subscriber.

```yaml
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

To perform action the example defines a Knative servicing Service resource which calls container image. We can see the service definition pulls a container from the VMware public container registry named 'kn-ps-echo' version 1.0.

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

Within the example folder is the Dockerfile used to create the container. We can see this defines a Powershell runtime environment with the [CloudEvents SDK](https://www.powershellgallery.com/packages/CloudEvents.Sdk) and [ThreadJob](https://www.powershellgallery.com/packages/ThreadJob) modules installed. When running the container executes server.ps1 which starts a CloudEvent HTTP listener and if we look within that it calls handler.ps1 which in this case is what outputs event contents.  Both of these Powershell scripts are copied into the container at point of creation.

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

To test the example we can first pull and execute the manifest file.

```bash
curl -O https://github.com/vmware-samples/vcenter-event-broker-appliance/blob/development/examples/knative/powershell/kn-ps-echo/function.yaml
kubectl apply --filename function.yaml --namespace vmware-functions

service.serving.knative.dev/kn-ps-echo created
trigger.eventing.knative.dev/kn-ps-echo-trigger created
```

We see the service and eventing resources are created and we can check the vmware-functions namespace to get the kn-ps-echo function pod names. There are two containers in the pod, the user-container runs the function so we can follow its logs and see the flow of vCenter events being echo'd.

```bash
kubectl get pods --namespace vmware-functions

NAME                                             READY   STATUS    RESTARTS   AGE
default-broker-ingress-5c98bf68bc-whmj4          1/1     Running   0          4d6h
kn-ps-echo-00001-deployment-6c9f77855c-ddz8w     2/2     Running   0          18m
kn-ps-echo-trigger-dispatcher-7bc8f78d48-5cwc7   1/1     Running   0          18m
sockeye-65697bdfc4-n8ght                         1/1     Running   0          4d6h
sockeye-trigger-dispatcher-5fff8567fc-9v74l      1/1     Running   0          4d6h

kubectl logs --namespace vmware-functions kn-ps-echo-00001-deployment-6c9f77855c-ddz8w user-container --follow

Server start listening on 'http://*:8080/'
Cloud Event
  Source: https://vcenter.cork.local/sdk
  Type: com.vmware.event.router/event
  Subject: UserLogoutSessionEvent
  Id: b2cb5b99-baf2-4b0b-93e7-33795e56ec88
CloudEvent Data:



Cloud Event
  Source: https://vcenter.cork.local/sdk
  Type: com.vmware.event.router/event
  Subject: UserLoginSessionEvent
  Id: 4256ead8-b86d-4bc0-96ac-92ccaae02605
CloudEvent Data:



Cloud Event
  Source: https://vcenter.cork.local/sdk
  Type: com.vmware.event.router/event
  Subject: UserLogoutSessionEvent
  Id: a160d7bd-542d-4729-98bd-bbb14d505373
CloudEvent Data:
```

So we can see all events are of the same Type: but the Subject: is populated with descriptive name. The subject contents maps to the vCenter Server event description a list of descriptions by vCenter Server version can be found [here](https://github.com/lamw/vcenter-event-mapping).

## Creating A Knative Function

So we can see it is easy to consume a pre-built function but I wonder how hard it is to create one to meet a bespoke need.  Pleased to report that it turns out that is also pretty easy.

If we start off by defining problem,  maybe maintaining the synchronicity of state between two systems. When performing ESXi host lifecycle operations it is useful to mark this state in multiple systems. Setting object state to maintenance mode in vCenter Server can trigger vMotion work away from host and prevent scheduling of new workload on host. Setting object state to maintenance mode in vRealize Operations helps reduce amount of false positive issues relating to lifecycle operations. Host lifecycle operations like patching are typically initiated via vCenter Server so its likely maintenance mode will be set enabled and disabled correctly. It might be easy to miss mirroring this operation in vRealize Operations.

So the first thing we need to do is identify the vCenter Server event created when a host is placed in maintenance mode. Checking the event documentaion we can find the two events are:

* [EnteredMaintenanceModeEvent](https://vdc-repo.vmware.com/vmwb-repository/dcr-public/fe08899f-1eec-4d8d-b3bc-a6664c168c2c/7fdf97a1-4c0d-4be0-9d43-2ceebbc174d9/doc/vim.event.EnteredMaintenanceModeEvent.html)
* [ExitMaintenanceModeEvent](https://vdc-repo.vmware.com/vmwb-repository/dcr-public/fe08899f-1eec-4d8d-b3bc-a6664c168c2c/7fdf97a1-4c0d-4be0-9d43-2ceebbc174d9/doc/vim.event.ExitMaintenanceModeEvent.html)

If we look first at EnteredMaintenanceModeEvent we can create a container image. We can reuse the example Dockerfile and server.ps1 without change.

```bash
## Create folders and pull down reusable example files
mkdir veba-knative-mm
mkdir veba-knative-mm/enter
mkdir veba-knative-mm/exit
cd veba-knative-mm/enter
curl -O https://raw.githubusercontent.com/vmware-samples/vcenter-event-broker-appliance/master/examples/knative/powershell/kn-ps-echo/Dockerfile
curl -O https://raw.githubusercontent.com/vmware-samples/vcenter-event-broker-appliance/master/examples/knative/powershell/kn-ps-echo/server.ps1
```

The [vRealize Operations Manager Suite API](https://code.vmware.com/apis/364/vrealize-operations) shows the two API calls which control Maintenance Mode.

```bash
## Enter Maintenance Mode
PUT /api/resources/{id}/maintained
DELETE /suite-api/api/resources/{id}/maintained
```

If we look at the object definition for EnteredMaintenanceModeEvent we can see it has properties of event object and extends this with additional maintenance mode related properties. With these details we can update the handler.ps1 script to call vROps API. The [blog post from vMAN.ch](https://vman.ch/vrops-maintenance-mode-for-resources/) heavily influenced the following Powershell logic to control maintenance mode state.

```powershell
cat <<EOF > handler.ps1
Function Process-Handler {
   param(
      [Parameter(Position=0,Mandatory=$true)][CloudNative.CloudEvents.CloudEvent]$CloudEvent
   )

   Write-Host "Cloud Event"
   Write-Host "  Source: $($cloudEvent.Source)"
   Write-Host "  Type: $($cloudEvent.Type)"
   Write-Host "  Subject: $($cloudEvent.Subject)"
   Write-Host "  Id: $($cloudEvent.Id)"
   Write-Host "  Host: $($cloudEvent.host)"

   # Decode CloudEvent
   $cloudEventData = $cloudEvent | Read-CloudEventJsonData -ErrorAction SilentlyContinue -Depth 10
   if($cloudEventData -eq $null) {
      $cloudEventData = $cloudEvent | Read-CloudEventData
   }

   Write-Host "CloudEvent Data:"
   Write-Host "$($cloudEventData | Out-String)"
EOF
```

With the Dockerfile and scripts it copies in ready we can look to build the container image locally and then push this to a public container registry.

```bash
# Build local image with tag for GitHub Container Registry
docker build --tag ghcr.io/darrylcauldwell/veba-ps-enter-mm:0.1 .

# Generate GitHub Personal Access Token
# Connect to GitHub Container Registry
# Use Personal Access Token when prompted for password
docker login ghcr.io -u darrylcauldwell

# Push local image to GitHub Container Registry
docker push ghcr.io/darrylcauldwell/veba-ps-enter-mm:0.1
```

Once the container is available in public repository we can look to create a Knative service resource which links to container image:

```yaml
cat <<EOF > enter-mm-service.yml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: enter-mm
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
        - image: ghcr.io/darrylcauldwell/veba-ps-enter-mm:0.1
EOF
```

Finally we can create a Knative trigger resource with filter:

```yaml
cat <<EOF > enter-mm-trigger.yml
apiVersion: eventing.knative.dev/v1
kind: Trigger
metadata:
  name: veba-ps-enter-mm-trigger
  labels:
    app: veba-ui
spec:
  broker: default
  filter:
    attributes:
        type: com.vmware.event.router/event
        subject: EnteredMaintenanceModeEvent
  subscriber:
    ref:
      apiVersion: serving.knative.dev/v1
      kind: Service
      name: enter-mm
EOF
```

