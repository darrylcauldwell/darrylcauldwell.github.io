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

When I worked as an architect working with AWS I used event-driven automation with AWS Lambda to integrate distributed systems. This event-driven automation allowed me to put complex systems in place very simply. The VMware Event Broker Appliance (VEBA) aims to facilitate event-driven automation based on vCenter Server events. The VMware Event Broker Appliance is made available as a [fling](https://flings.vmware.com/vmware-event-broker-appliance). The [system architecture](https://vmweventbroker.io/kb/architecture) shows that the appliance is built on a Photon OS running Kubernetes with Contour acting as ingress controller. The event broker appliance is composed of two components an event router and a choice of event stream processor Knative, OpenFaaS or AWS EventBridge.

The appliance OVA installation takes parameters which configure the event router with a binding to vCenter Server. Once installed we can take a look at the resource description.

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

## Knative

Knative is an open source community project which extends a Kubernetes deploymemnt with components for deploying, running, and managing serverless applications. It consists of three core components, build, serving and eventing. The build component offers a flexible approach to building source code into containers. The serving component enables rapid deployment and automatic scaling of containers through a request-driven model for serving workloads based on demand. The eventing component enabkles universal subscription, delivery, and management of events. Knative eventing is composed of Knative Broker and Trigger objects which make it easy to filter events based on event attributes. A Broker provides a bucket of events which can be selected by attribute. It receives events and forwards them to subscribers defined by one or more matching Triggers.

![Broker Trigger Architecture](/images/veba-knative-broker-trigger-overview.svg)

## Consuming Example Knative Function

The vmware-samples GitHub repository contains a folder containing [example Knative functions](https://github.com/vmware-samples/vcenter-event-broker-appliance/tree/development/examples/knative). The most simplistic example of a function is kn-ps-echo this recieves all events from the event router and outputs them to Stdout.

A Knative function requires a container image the example function is in a publicly available repository projects.registry.vmware.com/veba/kn-ps-echo. Within the example folder is the Dockerfile used to create the container. We can see this defines a Powershell runtime environment with the [CloudEvents SDK](https://www.powershellgallery.com/packages/CloudEvents.Sdk) and [ThreadJob](https://www.powershellgallery.com/packages/ThreadJob) modules installed. We can see two Powershell scripts are copied in server.ps1 and handler.ps1 and that server.ps1 is ran when the container is started. The server.ps1 acts as a System.Net.HttpListener which recieves event and on message reciept passes this on to handler.ps1 to perform an action.

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

As the manifest file and dependant container image are publicly available to test we can just execute the manifest file and it will create the Knative kservice and trigger resources.

```bash
kubectl -n vmware-function -f https://github.com/vmware-samples/vcenter-event-broker-appliance/blob/development/examples/knative/powershell/kn-ps-echo/function.yaml

service.serving.knative.dev/kn-ps-echo created
trigger.eventing.knative.dev/kn-ps-echo-trigger created
```

vCenter Server constantly generates events and as the example has an unfiltered trigger events are output to Stdout. We can view the output by looking at the log file of container executing the script.  If we list the pods in the vmware-functions namespace and filter using function name we can see two pods and the deployment pod has two containers.

```bash
kubectl -n vmware-functions get pods | grep kn-ps-echo

NAME                                             READY   STATUS    RESTARTS   AGE
kn-ps-echo-00001-deployment-6c9f77855c-ddz8w     2/2     Running   0          18m
kn-ps-echo-trigger-dispatcher-7bc8f78d48-5cwc7   1/1     Running   0          18m
```

The scripts are ran in a container named user-container in the deployment pod so we can follow its logs and see the flow of vCenter events being echo'd.

```
kubectl logs -na vmware-functions kn-ps-echo-00001-deployment-6c9f77855c-ddz8w user-container --follow

Server start listening on 'http://*:8080/'
Cloud Event
  Source: https://<MY VCENTER FQDN>/sdk
  Type: com.vmware.event.router/event
  Subject: UserLogoutSessionEvent
  Id: b2cb5b99-baf2-4b0b-93e7-33795e56ec88
CloudEvent Data:



Cloud Event
  Source: https://<MY VCENTER FQDN>/sdk
  Type: com.vmware.event.router/event
  Subject: UserLoginSessionEvent
  Id: 4256ead8-b86d-4bc0-96ac-92ccaae02605
CloudEvent Data:



Cloud Event
  Source: https://<MY VCENTER FQDN>/sdk
  Type: com.vmware.event.router/event
  Subject: UserLogoutSessionEvent
  Id: a160d7bd-542d-4729-98bd-bbb14d505373
CloudEvent Data:
```

So we can see all events are of the same Type: but the Subject: is populated with descriptive name. The subject contents maps to the vCenter Server event description a list of descriptions by vCenter Server version can be found [here](https://github.com/lamw/vcenter-event-mapping).

Note: At the time of running the example didn't seem to be working correctly as it did not output the CloudEvent Data.

## Creating A Knative Function

So we can see it is very easy to consume a pre-built function but I wonder how hard it is to create one to meet a bespoke need.  Pleased to report that it turns out that is also pretty easy.

If we start off by thinking about a use case for event driven architecture. I got a lot of value from eventing in AWS in forming loosely coupled relationship between discreet systems and maintaining synchronicity of state between the systems. vCenter Server and vRealize Operations Manager both have concept of maintenance mode and when performing lifecycle management tasks its common to enable this on both products. Here I'll look at the usecase where when I set vCenter Server maintenance mode state the event triggers functions which mirror the state in vRealize Operations Manager.

The first thing we need to do is identify the vCenter Server event created when a host is placed in maintenance mode. Checking the event documentaion we can find the two events are [EnteredMaintenanceModeEvent](https://vdc-repo.vmware.com/vmwb-repository/dcr-public/fe08899f-1eec-4d8d-b3bc-a6664c168c2c/7fdf97a1-4c0d-4be0-9d43-2ceebbc174d9/doc/vim.event.EnteredMaintenanceModeEvent.html) and [ExitMaintenanceModeEvent](https://vdc-repo.vmware.com/vmwb-repository/dcr-public/fe08899f-1eec-4d8d-b3bc-a6664c168c2c/7fdf97a1-4c0d-4be0-9d43-2ceebbc174d9/doc/vim.event.ExitMaintenanceModeEvent.html). Next we need to establish a API call to set state in vRealize Operations Manager checking the APIU documentation the [markResourceAsBeingMaintained](https://vdc-download.vmware.com/vmwb-repository/dcr-public/1e2150d8-8682-4213-a6f0-03fb3b1dc410/b10f698a-21c4-4194-84a5-d3aca9002a07/index.html#markResourceAsBeingMaintained) can be used to mark and unmark resourde maintenance mode.

I'll start by creating a container image for the enter event. The kn-ps-echo example Dockerfile and server.ps1 contain nothing I need to change so can be reused.

```bash
## Create folders and pull down reusable example files
mkdir veba-knative-mm-enter
cd veba-knative-mm-enter
curl -O https://raw.githubusercontent.com/vmware-samples/vcenter-event-broker-appliance/master/examples/knative/powershell/kn-ps-echo/Dockerfile
curl -O https://raw.githubusercontent.com/vmware-samples/vcenter-event-broker-appliance/master/examples/knative/powershell/kn-ps-echo/server.ps1
```

What I do need to do though is create a handler.ps1 which takes input from event and calls REST API of vROps.

```powershell
Function Process-Handler {
   param(
      [Parameter(Position=0,Mandatory=$true)][CloudNative.CloudEvents.CloudEvent]$CloudEvent
   )

# Form cloudEventData object and output to console for debugging
$cloudEventData = $cloudEvent | Read-CloudEventJsonData -ErrorAction SilentlyContinue -Depth 10
if($cloudEventData -eq $null) {
   $cloudEventData = $cloudEvent | Read-CloudEventData
   }
Write-Host "Full contents of CloudEventData`n $(${cloudEventData} | ConvertTo-Json)`n"

# Perform onward action

## vROps REST API documentation https://code.vmware.com/apis/364/vrealize-operations

## Check secret in place which supplies vROps environment variables
Write-Host "vropsFqdn:" ${env:vropsFqdn}
Write-Host "vropsUser:" ${env:vropsUser}
Write-Host "vropsPassword:" ${env:vropsPassword}

## Form unauthorized headers payload
$headers = @{
   "Content-Type" = "application/json";
   "Accept"  = "application/json"
   }

## Acquire bearer token
$uri = "https://" + $env:vropsFqdn + "/suite-api/api/auth/token/acquire"
$basicAuthBody = @{
    username = $env:vropsUser;
    password = $env:vropsPassword;
    }
$basicAuthBodyJson = $basicAuthBody | ConvertTo-Json -Depth 5
Write-Host "Acquiring bearer token ..."
$bearer = Invoke-WebRequest -Uri $uri -Method POST -Headers $headers -Body $basicAuthBodyJson -SkipCertificateCheck | ConvertFrom-Json
Write-Host "Bearer token is" $bearer.token

## Form authorized headers payload
$authedHeaders = @{
   "Content-Type" = "application/json";
   "Accept"  = "application/json";
   "Authorization" = "vRealizeOpsToken " + $bearer.token
   }

## Get host ResourceID
$uri = "https://" + $env:vropsFqdn + "/suite-api/api/adapterkinds/VMWARE/resourcekinds/HostSystem/resources?name=" + $esxiHost
Write-Host "Acquiring host ResourceID ..."
$resource = Invoke-WebRequest -Uri $uri -Method GET -Headers $authedHeaders -SkipCertificateCheck
$resourceJson = $resource.Content | ConvertFrom-Json
Write-Host "ResourceID of host is " $resourceJson.resourceList[0].identifier

## Mark host as maintenance mode
$uri = "https://" + $env:vropsFqdn + "/suite-api/api/resources/" + $resourceJson.resourceList[0].identifier + "/maintained"
Write-Host "Marking host as vROps maintenance mode ..."
Invoke-WebRequest -Uri $uri -Method PUT -Headers $authedHeaders -SkipCertificateCheck

## Get host maintenance mode state
$uri = "https://" + $env:vropsFqdn + "/suite-api/api/adapterkinds/VMWARE/resourcekinds/HostSystem/resources?name=" + $esxiHost
Write-Host "Acquiring host maintenance mode state ..."
$resource = Invoke-WebRequest -Uri $uri -Method GET -Headers $authedHeaders -SkipCertificateCheck
$resourceJson = $resource.Content | ConvertFrom-Json
Write-Host "Host maintenence mode state is " $resourceJson.resourceList[0].resourceStatusStates[0].resourceState
Write-Host "Note: STARTED=Not In Maintenance | MAINTAINED_MANUAL=In Maintenance"
}
```

In order to be environmentally agnositic I have the script use Environment Variables. These can be storedm in a Kubernetes Secret resource which can be associated with the containers and available at script runtime.

```
kubectl -n vmware-functions create secret generic veba-knative-mm-vrops \
  --from-literal=vropsFqdn=<MY VCENTER FQDN> \
  --from-literal=vropsUser=admin \
  --from-literal=vropsPassword='VMware1!'
```

With the Dockerfile and scripts in place we can look to build the container image locally and then push this to a public container registry.

```bash
# Build local image with tag for GitHub Container Registry
docker build --tag ghcr.io/darrylcauldwell/veba-ps-enter-mm:1.0 .

# Generate GitHub Personal Access Token
# Connect to GitHub Container Registry
# Use Personal Access Token when prompted for password
docker login ghcr.io -u darrylcauldwell

# Push local image to GitHub Container Registry
docker push ghcr.io/darrylcauldwell/veba-ps-enter-mm:1.0
```

Once the container is available in public repository I can look to create a Knative service and trigger resource which links to container image and has association with the secret:

```yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: veba-ps-enter-mm-service
  labels:
    app: veba-ui
  namespace: vmware-functions
spec:
  template:
    metadata:
      annotations:
        autoscaling.knative.dev/maxScale: "1"
        autoscaling.knative.dev/minScale: "1"
    spec:
      containers:
        - image: ghcr.io/darrylcauldwell/veba-ps-enter-mm:1.0
          envFrom:
            - secretRef:
                name: veba-knative-mm-vrops
---
apiVersion: eventing.knative.dev/v1
kind: Trigger
metadata:
  name: veba-ps-enter-mm-trigger
  labels:
    app: veba-ui
  namespace: vmware-functions
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
      name: veba-ps-enter-mm-service
```

The exit function is logically laid out the same but with slightly different trigger filter and handler action. With the manifest files created both can now be applied.

```bash
kubectl apply -f https://raw.githubusercontent.com/darrylcauldwell/veba-knative-mm-enter/main/veba-knative-mm-enter.yml

service.serving.knative.dev/veba-ps-enter-mm-service created
trigger.eventing.knative.dev/veba-ps-enter-mm-trigger created

kubectl apply -f https://raw.githubusercontent.com/darrylcauldwell/veba-knative-mm-exit/master/veba-knative-mm-exit.yml

service.serving.knative.dev/veba-ps-exit-mm-service created
trigger.eventing.knative.dev/veba-ps-exit-mm-trigger created
```

Once the container image has pulled down from repository can check the created resources.

```bash
kubectl -n vmware-functions get kservice | grep mm

NAME                       URL                                                                LATESTCREATED                    LATESTREADY                      READY   REASON
veba-ps-enter-mm-service   http://veba-ps-enter-mm-service.vmware-functions.veba.cork.local   veba-ps-enter-mm-service-00001   veba-ps-enter-mm-service-00001   True
veba-ps-exit-mm-service    http://veba-ps-exit-mm-service.vmware-functions.veba.cork.local    veba-ps-exit-mm-service-00001    veba-ps-exit-mm-service-00001    True

kubectl -n vmware-functions get triggers | grep mm

NAME                       BROKER    SUBSCRIBER_URI                                                       AGE    READY   REASON
veba-ps-enter-mm-trigger   default   http://veba-ps-enter-mm-service.vmware-functions.svc.cluster.local   61s    True
veba-ps-exit-mm-trigger    default   http://veba-ps-exit-mm-service.vmware-functions.svc.cluster.local    45s    True

kubectl -n vmware-functions get pods | grep mm

veba-ps-enter-mm-service-00001-deployment-d689d7fbd-9gtlv   2/2     Running   0          87s
veba-ps-enter-mm-trigger-dispatcher-848ff8c858-qnxg8        1/1     Running   0          76s
veba-ps-exit-mm-service-00001-deployment-b98b6f795-chqpx    2/2     Running   0          71s
veba-ps-exit-mm-trigger-dispatcher-5fc8cbc978-6n2nf         1/1     Running   0          65s
```

With the functions in place we can follow the logs on the container and place a host into maintenance mode to check it works.

```bash
kubectl -n vmware-functions logs veba-ps-enter-mm-service-00001-deployment-d689d7fbd-9gtlv user-container --follow

Server start listening on 'http://*:8080/'
Full contents of CloudEventData
 {
  "ChangeTag": "",
  "ChainId": 8122827,
  "Host": {
    "Name": "<MY HOST>",
    "Host": {
      "Value": "host-19",
      "Type": "HostSystem"
    }
  },
  "ComputeResource": {
    "Name": "Virtual-SAN-Cluster-5425b3e6-6e38-4221-8804-500f1360c7a3",
    "ComputeResource": {
      "Value": "domain-c9",
      "Type": "ClusterComputeResource"
    }
  },
  "Net": null,
  "Datacenter": {
    "Name": "Datacenter",
    "Datacenter": {
      "Value": "datacenter-3",
      "Type": "Datacenter"
    }
  },
  "Vm": null,
  "Dvs": null,
  "UserName": "VSPHERE.LOCAL\\Administrator",
  "CreatedTime": "2021-05-06T16:20:05.137999Z",
  "FullFormattedMessage": "Host <MY HOST> in Datacenter has entered maintenance mode",
  "Key": 8122856,
  "Ds": null
}

vropsFqdn: vrops.cork.local
vropsUser: admin
vropsPassword: VMware1!
Acquiring bearer token ...
Bearer token is b3898f1c-3a94-4dff-8b80-2ab835bd53bb::1c46ab4d-0e5e-44e4-ba4b-9898811bc645
Acquiring host ResourceID ...
ResourceID of host is  8f07b6de-9918-4849-af0f-7a1cca3ff5c7
Marking host as vROps maintenance mode ...
Acquiring host maintenance mode state ...
Host maintenence mode state is  MAINTAINED_MANUAL
Note: STARTED=Not In Maintenance | MAINTAINED_MANUAL=In Maintenance
```