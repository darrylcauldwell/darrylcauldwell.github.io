+++
title = "Creating A Knative Function Using Buildpacks"
date = "2022-3-22"
description = "Creating A Knative Function Using Buildpacks"
tags = [
    "knative",
    "kubernetes",
    "veba",
    "python",
    "docker"
]
categories = [
    "automation"
]
thumbnail = "clarity-icons/code-144.svg"
+++

I'm looking at event-based architectures as a method for building loosely coupled relationships between products. To this end I am creating various functions which I'd like to trigger on an event in a product. Knative is an extension of Kubernetes which enables serverless workloads to run on Kubernetes. Knative is made of various components Knative Eventing provides tools for routing events from event producers to sinks. Eventing uses standard HTTP POST requests to send and receive events between event producers and sinks. The event layouts conform to the CloudEvents specifications. Brokers and Triggers provide an "event mesh" model, which allows an event producer to deliver events to a Broker, which then distributes them uniformly to consumers by using Triggers. In this article, we start from the point where our Knative deployment already has a Broker connected to an event producer. The Knative function we are looking to create will require at least two resources, a trigger and a service.

## Knative Trigger Filter

The [Knative trigger](https://knative.dev/docs/eventing/broker/triggers/#trigger-filtering) resource can be configured with a filter so it only fires when a specific CloudEvent with a specific attribute is received by the Broker. This is also configured with an association to the Knative service resource.

```yaml
apiVersion: eventing.knative.dev/v1
kind: Trigger
metadata:
  name: veba-py-cb-trigger
  labels:
    app: veba-ui
spec:
  broker: default
  filter:
    attributes:
      type: com.vmware.event.router/event
      subject: VmCreatedEvent
  subscriber:
    ref:
      apiVersion: serving.knative.dev/v1
      kind: Service
      name: kn-py-cb
```

## Knative Service Resource

When a [Knative service](https://github.com/knative/specs/blob/main/specs/serving/knative-api-specification-1.0.md#service) is called by a trigger it executes a container image.

```yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: kn-py-cb
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
        - image: ghcr.io/darrylcauldwell/kn-py-cb:1.0
          envFrom:
            - secretRef:
                name: kn-py-cb-secret
```

## Python Flask

The Knative serving component references a container image that executes the code. Knative Eventing makes standard HTTP POST requests to the service. The container image must run a web service to receive the POST. Flask is a lightweight WSGI web application framework written in Python. It is designed to make getting started quick and easy, with the ability to scale up to complex applications. It is a good candidate to present our function logic from.

## Container Image Development Using Buildpacks

Creating container images can be handcrafted artisanal creations. Buildpacks look to simplify container creation and target developers that have source code and simply wish to turn that into a container image.

The Google Cloud Buildpacks project is a language family implementation on top of the Cloud Native Buildpack specification. Google Cloud Buildpacks support Go, Java, Node, Python, and .Net applications. Here I am using this as our practical example of developing our Python Flask Knative service image using a Google Cloud Buildpack.

### Install Pack CLI Tool

Pack is a CLI tool maintained by the Cloud Native Buildpacks project to support the use of buildpacks. The following command will download the latest version of pack from GitHub and install it in /usr/local/bin/.

```bash
(curl -sSL "https://github.com/buildpacks/pack/releases/download/v0.24.0/pack-v0.24.0-linux.tgz" | sudo tar -C /usr/local/bin/ --no-same-owner -xzv pack)
```

### Define The Container Entrypoint

Buildpacks allow applications to define process types using a special file named Procfile located at the root of the application. The Procfile specifies the commands that are executed by the app on startup.

While developing the function logic I'd like to run Flask in DEBUG mode so I specify the FLASK_ENV environment variable as development. I also define the FLASK_APP to be a file named 'handler.py' and bind this locally to the port defined at container runtime.

```Procfile
web: FLASK_ENV=development FLASK_APP=handler.py python3 -m flask run --host=0.0.0.0 --port=$PORT
```

## Function Logic

The Buildpack Procfile in our example runs a Python Flask application 'handler.py'. The Flask framework uses simple syntax and here we listen for HTTP POST made to the website /. The CloudEvent data is passed the POST body so we can get this as an object and use it as input to our logic.

```Python
# handler.py
from flask import Flask, request
from cloudevents.http import from_http
import logging,json,os,time
logging.basicConfig(level=logging.DEBUG,format='%(asctime)s %(levelname)s %(name)s %(threadName)s : %(message)s')

app = Flask(__name__)
@app.route("/", methods=["POST"])

def home():
    # Extract  VM Name From CloudEvent Data
    event = from_http(request.headers, request.get_data(),None)
    app.logger.debug(f"Full event contents {event}")
    data  = event.data
    app.logger.info(f"Found event ID {event['id']} triggered by {event['subject']} event for VM {data['Vm']['Name']}")
```

A Python application typically builds upon existing packages. To ensure the packages within the image the dependant packages and versions should be defined in 'requirements.txt'.

```Python
# requirements.txt
flask==2.0.3
cloudevents==1.2.0
```

### Create Container Image

With the required Buildpack components in place ensure that local docker can pull the buildpack. Set variable for the container image and then use pack CLI to build the container image and which base to build from.

```bash
docker pull gcr.io/buildpacks/builder:v1
IMAGE=ghcr.io/darrylcauldwell/kn-py-cb:1.0
pack build -B gcr.io/buildpacks/builder:v1 ${IMAGE}
```

## Environmental Specific Variables

The created container likely needs to be portable between deployments and any environmentally specific configuration passed at runtime. Here I use a file 'function.env' to define all environment variables.

```bash
# function.env
ENV1 = VALUE1
ENV2 = VALUE2
ENV3 = VALUE3
```

This file can be passed to the container at runtime which means these can be consumed by the code.

```env
app.logger.debug(f"Environment variable ENV1 value is " + os.environ['ENV1'])
app.logger.debug(f"Environment variable ENV2 value is " + os.environ['ENV2'])
app.logger.debug(f"Environment variable ENV3 value is " + os.environ['ENV3'])
app.logger.debug(f"Environment variable ENV4 value is " + os.environ['ENV4'])
```

## Rapid Iteration Development And Testing

Before deploying as Knative serving function it is possible to simulate how it will behave once triggered. We know that Knative is making an HTTP POST with CloudEvent data body which executes the container image. Once the container image is created using pack cli we can simply start the docker image and pass it a valid port to listen on and any environment variables it requires.

```bash
docker run -e PORT=8080 -it --env-file ./function.env --rm -p 8080:8080 ${IMAGE}
```

When the container image with function is running. Create a file with CloudEvent formatted JSON body and use curl to make the post to the locally running container image.

```bash
curl -i -d@testevent.json localhost:8080
```

In case the function does not behave in the expected way and you wish to iterate the logic simply:

1) Stop the container image
2) Edit the function logic eg 'handler.py'
3) Run the pack command
4) Start the new container image
5) Test the functionality by making local POST using cURL

## Software Dependancy Lifecycle

One of the benefits of using Buildpacks as a container builder over artisanal container images using Dockerfiles is lifecycle management. The base buildpack images are maintained to pickup updates you can simply re-run pack cli with no changes and this will recreate your image from the updated buildpack base. Similarly, when you want to update your Python packages you just need to update the requirements.txt and re-run.

## Bulky Buildpacks

Buildpacks are nice as they are simple and just work. But the resulting Container Images created from Buildpacks can be a bit bulky and can be not optimally layered.
