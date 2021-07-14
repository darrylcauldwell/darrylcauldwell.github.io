+++
title = "Maintaining Container Images Within GitHub"
date = "2021-07-07"
description = "GitHub CI Pipeline"
tags = [
    "github",
    "gitops",
    "devops",
    "pipeline"
]
categories = [
    "automation",
    "pipeline"
]
thumbnail = "clarity-icons/code-144.svg"
+++

I have been using a collection of products to maintain docker images. GitHub now provides suite of products which encompass all I require. This post documents the process I followed to create a CI pipeline for a container running static web site built with Jekyll. The static web site content is created from a fork of [chowdown](https://chowdown.io/). I maintain recipes markdown files using Visual Studio code and when I push changes to the 'publish' branch of my GitHub Repository this triggers a GitHub Action to create the container image and publish this to the GitHub Container Registry.

![GitHub Package Process Overview](/images/git-action-container-overview.drawio.png)

## Fork and clone chowdown

I am more interested in the process so I started by forking chowdown.  I look to automate where possible so used this opportunity to look at using the API to perform the fork.

```bash
## Create personal access token with permissions on repo
curl -u 'darrylcauldwell' https://api.github.com/repos/clarklab/chowdown/forks -d ''
## When promoted paste personal access token
```

With the fork in place I can look to make local clone and run the website under Jekyll.

```bash
git clone https://github.com/darrylcauldwell/chowdown.git
cd chowdown
## if running first time from form edit _config.yml and remove the value for key baseurl: otherwise it doesn't start properly
jekyll serve
## brower to http://127.0.0.1:4000
```

## Rename Main Branch

The repository forked from looks like it is hosted from GitHub Pages and contains single gh-pages branch.  I would like  main branch to be called publish to reflect that this contains the source for current HEAD image.

```bash
git branch --list --all
* gh-pages
  remotes/origin/HEAD -> origin/gh-pages
  remotes/origin/gh-pages

git branch -m gh-pages publish
git push origin :gh-pages publish
git push origin –u publish
git branch --list --all
* publish
  remotes/origin/HEAD -> origin/publish
  remotes/origin/publish
```

## Static File Web Site

Using 'jekyll serve' provide UX testing during development which is useful. The chowdown docker-compose.yml builds fro jekyll/jekyll image and runs jekyll serve to host the site. I would like to generate static file website where I can choose serving engine later. I can use 'jekyll build' which outputs the site to a folder (./_site by default). The chowdown repository lists folder /_site is in the .gitignore, this prevents the static file website being upstream. The GitHub Action will look to create docker image from upstream so I will remove /_site from the .gitignore.

```bash
jekyll build
git add .
git commit -m "added static file web site files"
git push
```

## Docker Image

I'll be looking to use NGINX to serve the site and would like the image to be as slim as possible so looking Alpine Linux. Copying the contents of /_site to the NGINX default path. In case of troubleshooting Pods being able to exec bash is useful so I look to install this.

```Dockerfile
# Dockerfile
FROM nginx:1.21.1-alpine
COPY _site /usr/share/nginx/html
# COPY nginx.conf /etc/nginx/nginx.conf
RUN apk add --no-cache bash
```

With the Dockerfile in place we can use this to build the container image locally. Once this is build we can run it and ensure that its working as we expect. When we're happy its working can push this to the GitHub Container Registry.

```bash
docker build --tag ghcr.io/darrylcauldwell/chowdown:1.0 .
docker run -d -p 80:80 ghcr.io/darrylcauldwell/chowdown:1.0
# browse http://127.0.0.0:80 to ensure 
docker push ghcr.io/darrylcauldwell/chowdown:1.0
```

## Event Driven Build

I would like a new version of the docker image be built every time a branch is merged to the main 'publish' branch. GitHub Actions offer me capability on occurence of an event (merge branch) trigger an action (build image). These particular container images will be ran on Raspberry Pi so need to use docker buildx to create for Arm platform

The GitHub Action is created and runs in context of the GitHub Repository,  the GitHub Container Registry is under context of user or organisation. In order the GitHub Action can write images the package needs configuring so Actions have write access.

![GitHub Package Access Writes](/images/git-action-container-access.png)

Once permissions are in place create a custom action using similar yaml to this:

```yaml
# .github/workflows/create.yml
name: Create chowdown image

on:
  push:
    branches: [ publish ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout
      uses: actions/checkout@v2

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v1
      
    - name: Log in to the Container registry
      uses: docker/login-action@f054a8b539a109f9f41c372932f1ae047eff08c9
      with:
        registry: ghcr.io
        username: darrylcauldwell
        password: ${{ secrets.GITHUB_TOKEN }}
          
    - name: Build and push image
      id: docker_build
      uses: docker/build-push-action@v2
      with:
        platforms: linux/amd64,linux/arm64
        push: true
        tags: |
          ghcr.io/darrylcauldwell/chowdown:latest
          ghcr.io/darrylcauldwell/chowdown:1.0

    - name: Image digest
      run: echo ${{ steps.docker_build.outputs.digest }}
```

The actions workflow definitio is within the repository so upon committing the changes the action should trigger and build the image.

## Deploy Container To Kubernetes

We can look to test this by creating a Kubernetes deployment and expose this as a service.

```bash
kubectl create namespace chowdown
kubectl create deployment chowdown --image=ghcr.io/darrylcauldwell/chowdown:1.0 --namespace=chowdown
kubectl expose deployment chowdown --port=80 --type=ClusterIP --namespace=chowdown
```

This service will be exposed to public internet. I have a single static IP but expose multiple services to achieve this I use Kubernetes Ingress rules which allow routing based on URL and path. Save the following as file named chowdown-ingress.yaml.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: chowdown-ingress
  namespace: chowdown
  annotations:
    kubernetes.io/ingress.class: public
spec:
  rules:
  - host: www.theveggiechef.uk
    http:
      paths:
      - backend:
          service:
            name: chowdown
            port: 
              number: 80
        path: /
        pathType: Prefix
```

Once created apply the file to create the Ingress route and test using cURL. 

```
kubectl apply -f chowdown-ingress.yaml

curl www.theveggiechef.uk
```