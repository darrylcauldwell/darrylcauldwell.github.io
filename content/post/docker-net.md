+++
title = "Docker Networking"
date = "2016-11-08"
description = "Docker Networking"
tags = [
    "docker"
]
categories = [
    "networking"
]
thumbnail = "clarity-icons/code-144.svg"
+++
Hear I describe Docker networking from the very basic through to extending it by pluggin in alternative drivers. In order to follow along with this post you will require Docker capability on your laptop to do this follow [these instructions](https://docs.docker.com/engine/installation/).

Mostly will be using [Alpine Linux](https://www.alpinelinux.org/) as it is very small so downloads quickly and doesn't consume many resources.

```bash
docker pull alpine
docker images
```

Output from images command should list all images you hold locally including Alpine Linux  

```
REPOSITORY  TAG     IMAGE ID        CREATED         SIZE
alpine      latest  baa5d63471ea    2 weeks ago     4.803 MB
```

Once we have the Alpine Linux image local we can then launch it,  if we do this with interactive and simulated TTY options we get presented with the Linux shell

```bash
docker run --interactive --tty alpine sh
```

From here we can view the docker containers view of the networking,  we see here it has allocated an rfc1918 address and that it can ping outbound.

``` bash
/ # ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
    valid_lft forever preferred_lft forever
16: eth0@if17: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP 
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.2/16 scope global eth0
    valid_lft forever preferred_lft forever
    inet6 fe80::42:acff:fe11:2/64 scope link 
    valid_lft forever preferred_lft forever
/ # ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8): 56 data bytes
64 bytes from 8.8.8.8: seq=0 ttl=61 time=43.770 ms
```

To detach the tty without exiting the shell, use the escape sequence Ctrl+p + Ctrl+q more details [here](https://docs.docker.com/engine/reference/commandline/attach/).  Once exited you should still be able to see the container in the running state.

```bash
docker ps -a

CONTAINER ID    IMAGE   COMMAND     CREATED             STATUS          PORTS   NAMES
5099910cc9d0    alpine  "sh"        7 minutes ago       Up 7 minutes            elegant_torvalds
```

If you need to get back to the running shell you can use either the container id or the name, these two commands are equivelant.

```bash
docker attach 5099910cc9d0
docker attach elegant_torvalds
```

So now if we start a second instance of Apline Linux in container

```bash
docker run --interactive --tty alpine sh
docker ps -a

CONTAINER ID    IMAGE   COMMAND     CREATED             STATUS          PORTS   NAMES
e96c3e67410c    alpine  "sh"        6 seconds ago       Up 6 seconds            suspicious_pare
5099910cc9d0    alpine  "sh"        9 minutes ago       Up 9 minutes            elegant_torvalds
```

We can attach and detach to each,  we notice that the second instance has been assigned next IP 172.17.0.3 and that it can ping the first instance 172.17.0.2.  

If we then detach the containers and view what docker networking shows, we see that Docker uses the concept of Drivers and that we are using the default bridge driver.

```bash
docker network ls

NETWORK ID          NAME                DRIVER
64481f7454d1        bridge              bridge              
a8f3b81d9991        host                host                
6ccbf074e447        none                null         
```

We can explore the properties of the bridge network a little more, and see the IP addresses it has issued to the containers.

```bash
docker network inspect bridge

[
{
    "Name": "bridge",
    "Id": "64481f7454d138e43558359d1e41bb96564769bfcccc547b42b2a37b873c3a73",
    "Scope": "local",
    "Driver": "bridge",
    "EnableIPv6": false,
    "IPAM": {
        "Driver": "default",
        "Options": null,
        "Config": [
            {
                "Subnet": "172.17.0.0/16"
            }
        ]
    },
    "Internal": false,
    "Containers": {}
    },
    "Options": {
        "com.docker.network.bridge.default_bridge": "true",
        "com.docker.network.bridge.enable_icc": "true",
        "com.docker.network.bridge.enable_ip_masquerade": "true",
        "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
        "com.docker.network.bridge.name": "docker0",
        "com.docker.network.driver.mtu": "1500"
    },
    "Labels": {}
}
]
```

So we can see that docker created a shared bridge, but what happens if you want to isolate containers on the same host well we can create a different bridge network we see it allocates the 172.18.0.0/16 CIDR.

```bash
docker network create --driver bridge my-bridge-network
docker network inspect my-bridge-network

[
    {
        "Name": "my-bridge-network",
        "Id": "5fec8c172a6c323a616c54938815b42d6baab04e816d7a5bb9113a0e914c52a4",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1/16"
                }
            ]
        },
        "Internal": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
```    

So if we create a new container now on this new network, we see it gets IP address on 172.18.0.0/16 and it cannot ping containers on 172.17.0.0/16.

``` bash
docker run --interactive --tty --net=my-bridge-network alpine sh
/ # ipaddr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
    valid_lft forever preferred_lft forever
25: eth0@if26: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP 
    link/ether 02:42:ac:12:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.2/16 scope global eth0
    valid_lft forever preferred_lft forever
    inet6 fe80::42:acff:fe12:2/64 scope link 
    valid_lft forever preferred_lft forever
/ # ping 172.17.0.3
PING 172.17.0.3 (172.17.0.3): 56 data bytes
^C
--- 172.17.0.3 ping statistics ---
8 packets transmitted, 0 packets received, 100% packet loss
```

We have focussed on the shipped bridge driver, however since [LibNetwork project](https://github.com/docker/libnetwork) got integrated into docker the driver became plugable. Some of the Docker networking ecosystems, include Weave, Nuage, Cisco, Microsoft, Calico, Midokura, and VMware.

Upto now we have looked at networking between containers on the same host by bridging to the network interface card. You will probably want to extend the layer 2 networking capacibility so containers on different hosts can communicate. To do this we would use [Docker Swarm](https://docs.docker.com/swarm/networking/) which includes the multi-host networking feature and allows the creation of custom container networks which span multiple Docker hosts. By default it does this by use of the 'overlay' network driver 

An interesting option to use as a Docker networking driver is [Project Calico for Containers](https://github.com/projectcalico/calico-containers) as the overlay networking expands across more and more hosts it can experience performance issues. Project Calico aims to over come this issue but allow container connectivity across hosts by having each container route directly at Layer 3 rather than Layer 2.  It does this by having a calico-node container installed and running on each host which manages the network routing etc.

As in my previous post about [vSphere Integrated Containers](/install-vic/) its interesting to see VMware integrating with Docker and I'd expect it won't be too long until we discover how NSX will plug into Docker Networking.