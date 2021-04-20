+++
title = "Installing vSphere Integrated Containers In Five Minutes"
date = "2016-07-13"
description = "Installing vSphere Integrated Containers In Five Minutes"
tags = [
    "docker",
    "vsphere"
]
thumbnail = "clarity-icons/code-144.svg"
+++
As it looks like vSphere Integrated Containers will feature a lot at the upcoming VMworld I took the chance to install this and find out more.

The [documentation available](https://github.com/vmware/vic/tree/master/doc/user_doc/vic_installation) on isn't extensive, but, contains all I needed to know and I think benefits from being to the point.

The files needed to set this up are provided as a gzip file 
[hosted on bintray](https://bintray.com/vmware/vic-repo/build#files). Just checking the file dates you can see the development is iterating at a great pace with around six releases per day. I chose to download the latest package which contains installers to be run from different operating systems.

|**Platform**|**Supported Versions**|
|---|---|
|Windows|7, 10|
|Mac OS X |10.11 (TBC)|
|Linux|Ubuntu 15.04, others TBD|

My homelab is vCenter, single cluster of two ESX hosts, storage VSAN and networking NSX. Security is not important in my homelab and TLS adds complication so I chose not to add this. There are great syntaxt examples to follow in the documentation, but here are options I used.

```
./vic-machine-darwin create --target 192.168.1.13 --user Administrator@vsphere.local --password VMware1! --compute-resource ElectricChair --image-datastore vsanDatastore --bridge-network DPortGroup --name vch
```

![vSphere Integrated Containers Install](/images/vic-install.jpg)

In order to use vSphere Integrated Containers you'll need a Docker client, for Mac there is the choice of [Docker Toolbox](https://www.docker.com/products/docker-toolbox) or the newer [Beta Docker for Mac](https://docs.docker.com/docker-for-mac/). I had the beta installed, and found I needed to remove this and replace with Docker Toolbox.

The first thing to do is ensure that you can connect and that it looks healthy, to do this run

```bash
docker -H ipaddress-vch:2375 info
```

![vSphere Integrated Containers Info](/images/vic-info.jpg)

Its always useful to look in the log files when your setting something up for the first time, however SSH is disabled on the VCH host so I enabled SSH for current session.  Logon via console using *root* with password of *password*.

```bash
systemctl start sshd
```

There are five log files I found so far

```bash
/var/log/vic/docker-personality.log
/var/log/vic/imagec.log
/var/log/vic/init.log
/var/log/vic/port-layer.log
/var/log/vic/vicadmin.log
```

So now we're running and can access logs the next thing to do is pull a container

```bash
docker -H paddress-vch:2375 pull vmwarecna/nginx
```

![vSphere Integrated Containers NGINX](/images/vic-nginx.jpg)

Once its local we can start it up,  we'll start it in the background and put a port mapping of port 80 in place and then view it.

```
docker -H ipaddress-vch:2375 run -d -p 80:80 vmwarecna/nginx
docker -H ipaddress-vch:2375 ps
```

![vSphere Integrated Containers NGINX Go](/images/vic-nginx-go.jpg)

We can then also see that a new VM has been created in vCenter with VM name matching the UID of the docker container.

![vSphere Integrated Containers NGINX VM](/images/vic-nginx-vm.jpg)

We can then stop this and tidy up our test container

```bash
docker -H ipaddress-vch:2375 stop container-id
docker -H ipaddress-vch:2375 rm container-id
```

As its so easy to install and configure its probably worth while removing VCH and when you come to use it pull the latest to remove is as easy as installing.

```bash
./vic-machine-darwin delete --target vcenter.darrylcauldwell.local --user Administrator@vsphere.local --password VMware1! --name vch --force
```

![vSphere Integrated Containers Bye!](/images/vic-bye.jpg)

There is a little bug just now and the image files don't seem to get removed by uninstall process but these are easy enough to delete along with the VIC folder from the datatore.

I'm disappointed in myself for waiting so long to look at this, what was putting me off was the thought it might be complex but it proved to be very much straight forwards.