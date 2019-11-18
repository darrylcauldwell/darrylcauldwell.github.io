---
layout: post
title:  "Introduction To Docker Container Networking"
date:   2019-11-18 20:20:56 +0100
tags:
    - Docker
    - DevOps
permalink: basic-container-networking/
---
When developing a cloud native application using Docker containers understanding network connectivity between containers and non-Docker workloads such as other servers and cloud APIs is essential. Docker’s networking subsystem is pluggable using drivers, several drivers exist by default, and others can be added easily.

## Linking Containers (legacy)

Linking containers by name is a simple method of enabling communications between containers. To link a an app server to a DB we simply reference the db container name in the command we execute to run app server container.

```
docker run -d --name my_mongodb mongo
docker run -d --link my_mongodb --name my_nodeapp -it node
docker ps -l

    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
    5dc2aa4d8542        node                "docker-entrypoint.s…"   7 seconds ago       Up 6 seconds                            my_nodeapp
```

In order to facilitate communications Docker automatically creates environment variables in the target container. It also exposes all environment variables originating from Docker from the source container. If we view the IP address of the two containers and then view the environment variables of the app server we can see the details for the db server.

```
docker inspect --format '{{ .ID }} - {{ .Name }} - {{ .NetworkSettings.IPAddress }}' my_mongodb
docker inspect --format '{{ .ID }} - {{ .Name }} - {{ .NetworkSettings.IPAddress }}' my_nodeapp

docker exec my_nodeapp env

    PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    HOSTNAME=a88e6e3f1107
    MY_MONGODB_PORT=tcp://172.17.0.2:27017
    MY_MONGODB_PORT_27017_TCP=tcp://172.17.0.2:27017
    MY_MONGODB_PORT_27017_TCP_ADDR=172.17.0.2
    MY_MONGODB_PORT_27017_TCP_PORT=27017
    MY_MONGODB_PORT_27017_TCP_PROTO=tcp
    MY_MONGODB_NAME=/my_nodeapp/my_mongodb
    MY_MONGODB_ENV_GOSU_VERSION=1.11
    MY_MONGODB_ENV_JSYAML_VERSION=3.13.0
    MY_MONGODB_ENV_GPG_KEYS=E162F504A20CDF15827F718D4B7C549A058F8B6B
    MY_MONGODB_ENV_MONGO_PACKAGE=mongodb-org
    MY_MONGODB_ENV_MONGO_REPO=repo.mongodb.org
    MY_MONGODB_ENV_MONGO_MAJOR=4.2
    MY_MONGODB_ENV_MONGO_VERSION=4.2.1
    NODE_VERSION=13.1.0
    YARN_VERSION=1.19.1
    HOME=/root
```

Container linking is a legacy method of connecting containers it is much more likely today that communications on single host will use a bridge network.

## Container Networking With A Bridge Driver

A bridge network can be used on Docker host to enable communications between containers. The first step is to create a custom bridge network on the Docker host which the containers will connect to. Each Docker network gets assigned a CIDR range and has a IPAM capability, we can view these by inspecting the network.

The default bridge network is present on all Docker hosts, if you do not specify a different network, new containers are automatically connected to the default bridge network. More than likely you will want to separate containers to different networks and so will want to create multiple networks and attach different containers to each.

```
docker network create --driver bridge my_isolated_network
docker network ls
docker network inspect my_isolated_network

    [
        {
            "Name": "my_isolated_network",
            "Id": "76e440d68a136ffe1ab9a4dd4977a7dc0499167814de14bf4bedfaffb646197b",
            "Created": "2019-11-18T09:59:09.8310474Z",
            "Scope": "local",
            "Driver": "bridge",
            "EnableIPv6": false,
            "IPAM": {
                "Driver": "default",
                "Options": {},
                "Config": [
                    {
                        "Subnet": "172.18.0.0/16",
                        "Gateway": "172.18.0.1"
                    }
                ]
            },
            "Internal": false,
            "Attachable": false,
            "Ingress": false,
            "ConfigFrom": {
                "Network": ""
            },
            "ConfigOnly": false,
            "Containers": {},
            "Options": {},
            "Labels": {}
        }
    ]
```

When networks are created you can start containers in those bridge networks.  The containers in same isolated network can then communicate by referencing name. When containers are running and connected we can inspect the network again and get the container IP address assignments etc.

```
docker run -d --net=my_isolated_network --name my_mongodb mongo
docker run -d --net=my_isolated_network --name my_nodeapp -it node
docker network inspect my_isolated_network

    [
        {
            "Name": "my_isolated_network",
            "Id": "76e440d68a136ffe1ab9a4dd4977a7dc0499167814de14bf4bedfaffb646197b",
            "Created": "2019-11-18T09:59:09.8310474Z",
            "Scope": "local",
            "Driver": "bridge",
            "EnableIPv6": false,
            "IPAM": {
                "Driver": "default",
                "Options": {},
                "Config": [
                    {
                        "Subnet": "172.18.0.0/16",
                        "Gateway": "172.18.0.1"
                    }
                ]
            },
            "Internal": false,
            "Attachable": false,
            "Ingress": false,
            "ConfigFrom": {
                "Network": ""
            },
            "ConfigOnly": false,
            "Containers": {
                "996b590f206dded3db3a653e554167e046fe3852f55a78d6d983eaa0f7f7fc3d": {
                    "Name": "my_mongodb",
                    "EndpointID": "e78b23529f29b2a6f5291f06fb1875cac9958888ca0eb95cae814dc8b631613a",
                    "MacAddress": "02:42:ac:12:00:02",
                    "IPv4Address": "172.18.0.2/16",
                    "IPv6Address": ""
                },
                "eebe0a3bf7116eab4731ef68da445f24c15b185efc8d7cea20937736702b93f8": {
                    "Name": "my_nodeapp",
                    "EndpointID": "6903d08b5942c71b5afd09caf535d4abb818ded97742fe324d54bb6396b595a5",
                    "MacAddress": "02:42:ac:12:00:03",
                    "IPv4Address": "172.18.0.3/16",
                    "IPv6Address": ""
                }
            },
            "Options": {},
            "Labels": {}
        }
    ]

```

The docker daemon implements an embedded DNS server (127.0.0.11) which provides built-in service discovery for any container created with a valid name. The containers are able to use this by docker overlay three crucial /etc files inside the container with virtual files. This arrangement allows Docker to do clever things like keep resolv.conf up to date across all containers.

```
docker exec my_nodeapp mount

    ...
    /dev/sda1 on /etc/resolv.conf type ext4 (rw,relatime,data=ordered)
    /dev/sda1 on /etc/hostname type ext4 (rw,relatime,data=ordered)
    /dev/sda1 on /etc/hosts type ext4 (rw,relatime,data=ordered)
    ...
```

If we view the DNS settings of one of the containers we see that it is set by default to 127.0.0.11.

```
docker exec -it my_nodeapp cat /etc/resolv.conf

    nameserver 127.0.0.11
    options ndots:0
```

We can therefore ping the db by container name from the app server.

```
docker exec -it my_nodeapp ping -c 1 my_mongodb

    PING my_mongodb (172.18.0.2) 56(84) bytes of data.
    64 bytes from my_mongodb.my_isolated_network (172.18.0.2): icmp_seq=1 ttl=64 time=0.177 ms

    --- my_mongodb ping statistics ---
    1 packets transmitted, 1 received, 0% packet loss, time 0ms
    rtt min/avg/max/mdev = 0.177/0.177/0.177/0.000 ms
```

It can be a relatively common requirement for me to have different container names for same application function in different environments e.g. dev_mongodb, qa_mongodb, prod_mongodb. While I could configuring connection strings in the app server it might be more useful to give the database a connection alias e.g. mongodb. Docker run command has flag --network-alias so we can use a common alias between environments.

```
docker run -d --net=my_isolated_network --name my_mongodb --network-alias mongodb mongo 
docker run -d --net=my_isolated_network --name my_nodeapp -it node
docker exec -it my_nodeapp ping -c 1 mongodb

    PING mongodb (172.18.0.2) 56(84) bytes of data.
    64 bytes from my_mongodb.my_isolated_network (172.18.0.2): icmp_seq=1 ttl=64 time=0.136 ms

    --- mongodb ping statistics ---
    1 packets transmitted, 1 received, 0% packet loss, time 0ms
    rtt min/avg/max/mdev = 0.136/0.136/0.136/0.000 ms
```

