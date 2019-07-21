---
title: \#Cloud Computing\# Introduction to Cloud Infrastructure Technologies(9) Software-Defined Networking and Networking for Containers
date: 2019-07-18 15:13:21
categories: Cloud Computing
tags:
- Computer Network
thumbnail: /images/cloud_computing.png
---



**Welcome to LFS151x: Introduction to Cloud Infrastructure Technologies**

By the end of this chapter, you should be able to:

- Define Software-Defined Networking.
- Discuss the basics of Software-Defined Networking.
- Discuss different networking options with containers using Docker, Kubernetes and Cloud Foundry.



<!-- more -->



## Software-Defined Networking

### Concept

*“To connect devices, applications, VMs, and containers, we need a similar level of virtualization in networking. This will allow us to meet the end-to-end business requirements. Software-Defined Networking (SDN) decouples the network control layer from the layer which forwards the traffic. This allows SDN to program the control layer to create custom rules in order to meet the networking requirements.”*

### Features

- **Ingress and egress packets** 
  These are done at the lowest layer, which decides what to do with ingress packets and which packets to forward, based on forwarding tables. These activities are mapped as Data Plane activities. All routers, switches, modem, etc. are part of this plane. 
- **Collect, process, and manage the network information** 
  By collecting, processing, and managing the network information, the network device makes the forwarding decisions, which the Data Plane follows. These activities are mapped by the Control Plane activities. Some of the protocols which run on the Control Plane are routing and adjacent device discovery. 
- **Monitor and manage the network** 
  Using the tools available in the Management Plane*,* we can interact with the network device to configure it and monitor it with tools like SNMP (Simple Network Management Protocol).  

### Components

- **Data Plane**
  The Data Plane, also called the Forwarding Plane, is responsible for handling data packets and apply actions to them based on rules which we program into lookup-tables.

- **Control Plane**

  The Control Plane is tasked with calculating and programming the actions for the Data Plane. This is where the forwarding decisions are made and where services (e.g. Quality of Service and VLANs) are implemented. 

- **Management Plane** 
  The Management Plane is the place where we can configure, monitor, and manage the network devices.

### Architecture

In [Software-Defined Networking](https://en.wikipedia.org/wiki/Software-defined_networking), we decouple the Control Plane with the Data Plane. The Control Plane has a centralized view of the overall network, which allows it to create *forwarding tables* of interest. These tables are then given to the Data Plane to manage network traffic.

![SDN-Framework](/images/SDN-Framework.png)

The Control Plane has well-defined APIs that take requests from applications to configure the network. After preparing the desired state of the network, the Control Plane communicates that to the Data Plane (also known as the Forwarding Plane*),* using a well-defined protocol like [OpenFlow](http://en.wikipedia.org/wiki/OpenFlow). We can use configuration tools like Ansible or Chef to configure SDN, which brings lots of flexibility and agility on the operations side as well.

## Networking for Containers

### Concept

*“The host kernel uses the [network namespace](https://lwn.net/Articles/580893/) feature of the Linux kernel to isolate the network from one container to another on the system. Network namespaces can be shared as well.”*

- On a single host, when using the virtual Ethernet (vEth) feature with Linux bridging, we can give a virtual network interface to each container and assign it an IP address. With technologies like [Macvlan](https://docs.docker.com/network/macvlan/) and [IPVlan](https://www.kernel.org/doc/Documentation/networking/ipvlan.txt) we can configure each container to have a unique and world-wide routable IP address.
- If we want to do multi-host networking with containers, the most common solution is to use some form of Overlay network driver, which encapsulates the Layer 2 traffic to a higher layer. Examples of this type of implementation are the Docker Overlay Driver, Flannel, Weave, etc. Project Calico allows multi-host networking on Layer 3 using BGP (Border Gateway Protocol).

### Standards

- [**Container Network Model**](https://github.com/docker/libnetwork/blob/master/docs/design.md) (**CNM**)
  Docker, Inc. is the primary driver for this networking model. It is implemented using the [libnetwork](https://github.com/docker/libnetwork/blob/master/docs/design.md) project, which has the following utilizations:

  - **Null**: NOOP implementation of the driver. It is used when no networking is required.
  - **Bridge**: It provides a Linux-specific bridging implementation based in Linux Bridge.
  - **Overlay**: It provides a [multi-host communication](https://github.com/docker/libnetwork/blob/master/docs/overlay.md)[ ](https://courses.edx.org/courses/course-v1:LinuxFoundationX+LFS151.x+2T2018/courseware/b5d5959248e24467b640307bff6a2890/10c130094667494c8bbbc0262cb8ae7d/?child=first#)over VXLAN.
  - **Remote**: It does not provide a driver. Instead, it provides a means of supporting drivers over a remote transport, by which we can write third-party drivers.

- **Container Networking Interface** (**CNI**) 

  It is a Cloud Native Computing Foundation project which consists of specifications and libraries for writing plugins to configure network interfaces in Linux containers, along with a number of supported plugins. It is limited to provide network connectivity of containers and removing allocated resources when the container is deleted. As such, it has a wide range of support. It is used by projects like Kubernetes, OpenShift, Cloud Foundry, etc. 

### Service Discovery

#### Concept

*“Service discovery is a mechanism by which processes and services can find each other automatically and talk. With respect to containers, it is used to map a container name with its IP address, so that we can access the container without worrying about its exact location (IP address). ”*

#### Components

- **Registration**
  When a container starts, the container scheduler registers the mapping in some key-value store like **etcd** or **Consul**. And, if the container restarts or stops, then the scheduler updates the mapping accordingly.
- **Lookup**
  Services and applications use *Lookup* to get the address of a container, so that they can connect to it. Generally, this is done using some kind of DNS (Domain Name Server), which is local to the environment. The DNS used resolves the requests by looking at the entries in the key-value store, which is used for *Registration*. SkyDNS and Mesos-DNS are examples of such DNS services.

### Listing the Available Networks

#### Commands

If we list the available networks after installing the Docker daemon, we should see something like the following:

```powershell
$ docker network ls
NETWORK ID          NAME          DRIVER
6f30debc5baf        bridge        bridge
a1798169d2c0        host          host
4eb0210d40bb        none          null
```

**bridge, null,** and **host** are different network drivers available on a single host.

```powershell
$ ifconfig
docker0   Link encap:Ethernet  HWaddr 02:42:A9:DB:AF:39
          inet addr:172.17.0.1  Bcast:0.0.0.0  Mask:255.255.0.0
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```

#### Bridge

Like the hardware bridge, we can emulate a software bridge on a Linux host. It can forward traffic between two networks based on MAC (hardware address) addresses. By default, Docker creates a **docker0** Linux bridge. All the containers on a single host get an IP address from this bridge, unless we specify some other network with the **--net=** option. Docker uses theLinux's virtual Ethernet (**vEth**) feature to create two virtual interfaces, one end of which is attached to the container and the other end to the **docker0** bridge.

```powershell
$ docker container run -it --name=c1 busybox /bin/sh
/ # ip a
1: lo: <LOOPBACK,UP,LOWE_UP> mtu 65536 qdisc noqueue qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
7: eth0@if8: <BROADCAST,MULTICAST,UP, LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.2/16 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:acff:fe11:2/64 scope link
       valid_lft forever preferred_lft forever
```

- Inspecting a Bridge Network

  ```powershell
  $ docker network inspect bridge
  [
       {
          "Name": "bridge",
          "Id": "6f30debc5baff467d437e3c7c3de673f21b51f821588aca2e30a7db68f10260c",
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
          "Containers": {
              "613f1c7812a9db597e7e0efbd1cc102426edea02d9b281061967e25a4841733f": {
                   "Name": "c1",
                   "EndpointID": "80070f69de6d147732eb119e02d161326f40b47a0cc0f7f14ac7d207ac09a695",
                   "MacAddress": "02:42:ac:11:00:02",
                   "IPv4Address": "172.17.0.2/16",
                   "IPv6Address": ""
               }
           },
           "Options": {
               "com.docker.network.bridge.default_bridge": "true",
               "com.docker.network.bridge.enable_icc": "true",
               "com.docker.network.bridge.enable_ip_masquerade": "true"
               "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
               "com.docker.network.bridge.name": "docker0",
               "com.docker.network.driver.mtu": "1500"
           },
           "Labels": {}
        }
   ]
  ```

- Creating a Bridge Network

  ```powershell
  $ docker network create --driver bridge my_bridge
  $ docker container run --net=my_bridge -itd --name=c2 busybox
  ```

A **bridge** network does not support automatic service discovery, so we have to rely on the [legacy **--link** option](https://docs.docker.com/network/links/).

![Creating-a-Bridge-Network](/images/Creating-a-Bridge-Network.png)

#### Null

As the name suggests, **NULL** means no networking. If we attach a container to a **null**driver, then it would just get the loopback interface. It would not be accessible from the outside.

```powershell
$ docker container run -it --name=c3 --net=none busybox /bin/sh
/ # ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 1disc noqueue qlen 1
         link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet ::1/128 scope host
       valid_lft forever preferred_lft forever
```

#### Host

Using the **host** driver, we can share the host machine's network namespace with a container. By doing so, the container would have full access to the host's network. We can see below that running an **ifconfig** command inside the container lists all the interfaces of the host system:

```powershell
$ docker container run -it --name=c4 --net=host  busybox /bin/sh
/ # ifconfig
docker0   Link encap:Ethernet  HWaddr 02:42:A9:DB:AF:39

          inet addr:172.17.0.1  Bcast:0.0.0.0  Mask:255.255.0.0
          inet6 addr: fe80::42:a9ff:fedb:af39/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:8 errors:0 dropped:0 overruns:0 frame:0
          TX packets:8 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:536 (536.0 B)  TX bytes:648 (648.0 B)

eth0      Link encap:Ethernet  HWaddr 08:00:27:CA:BD:10
          inet addr:10.0.2.15  Bcast:10.0.2.255  Mask:255.255.255.0
          inet6 addr: fe80::a00:27ff:feca:bd10/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:3399 errors:0 dropped:0 overruns:0 frame:0
          TX packets:2050 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:1021964 (998.0 KiB)  TX bytes:287879 (281.1 KiB)

eth1      Link encap:Ethernet  HWaddr 08:00:27:00:42:F9
          inet addr:192.168.99.100  Bcast:192.168.99.255  Mask:255.255.255.0
          inet6 addr: fe80::a00:27ff:fe00:42f9/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:71 errors:0 dropped:0 overruns:0 frame:0
          TX packets:46 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:13475 (13.1 KiB)  TX bytes:7754 (7.5 KiB)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:16 errors:0 dropped:0 overruns:0 frame:0
          TX packets:16 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1
          RX bytes:1021964376 (1.3 KiB)  TX bytes:1376 (1.3 KiB)

vethb3bb730 Link encap:Ethernet  HWaddr 4E:7C:8F:B2:2D:AD
          inet6 addr: fe80::4c7c:8fff:feb2:2dad/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:8 errors:0 dropped:0 overruns:0 frame:0
          TX packets:16 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:648 (648.0 B)  TX bytes:1296 (1.2 KiB)
```

### Sharing Network Namespaces Among Containers

#### Commands

Similar to **host**, we can share network namespaces among containers. So, two or more containers can have the same network stack and reach each other by referring to **localhost**.

- Let's take a look at its IP address:

  ```powershell
  $ docker container run -it --name=c5 busybox /bin/sh
  / # ip a
  1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1
      link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
      inet 127.0.0.1/8 scope host lo
          valid_lft forever preferred_lft forever
      inet6 ::1/128 scope host
          valid_lft forever preferred_lft forever
  
  10: eth0@if11: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue
      link/ether 02:42:ac:11:00:03 brd ff:ff:ff:ff:ff:ff
      inet 172.17.0.3/16 scope global eth0
          valid_lft forever preferred_lft forever
      inet6 fe80::42:acff:fe11:3/64 scope link
          valid_lft forever preferred_lft forever
  ```

- Now, if we start a new container with the **--net=container:CONTAINER** option, we can see that the other container has the same IP address.

  ```powershell
  $ docker container run -it --name=c6 --net=container:c5 busybox /bin/sh
  / # ip a
  1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1
      link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
      inet 127.0.0.1/8 scope host lo
          valid_lft forever preferred_lft forever
      inet6 ::1/128 scope host
          valid_lft forever preferred_lft forever
  
  12: eth0@if13: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue
      link/ether 02:42:ac:11:00:03 brd ff:ff:ff:ff:ff:ff
      inet 172.17.0.3/16 scope global eth0
          valid_lft forever preferred_lft forever
      inet6 fe80::42:acff:fe11:3/64 scope link
          valid_lft forever preferred_lft forever
  ```

### Docker Multi-Host Networking

#### Components

- **Docker Overlay Driver**
  With the Overlay driver, Docker encapsulates the container's IP packet inside a host's packet while sending it over the wire. While receiving, Docker on the other host decapsulates the whole packet and forwards the container's packet to the receiving container. This is accomplished with **libnetwork**, a built-in VXLAN-based overlay network driver.

  ![packetwalk](/images/packetwalk.png)

- **Macvlan Driver**

  With the Macvlan driver, Docker assigns a MAC (physical) address for each container, and makes it appear as a physical device on the network. As the containers appears in the same physical network as the Docker host, we can assign them an IP from the network subnet as host. As a result, we can do direct container-to-container communication between different hosts. Containers can also directly talk to hosts. We need hardware support to implement the Macvlan driver. For more information about Macvlan, please visit its [documentation available on the Docker website](https://docs.docker.com/network/macvlan/). 

#### Plugins

- **Weave Network Plugin**
  Weave Net provides multi-host container networking for Docker. It also provides service discovery and does not require any external cluster store to save the networking configuration. Weave Net has a Docker Networking Plugin which we can use with Docker deployment.

- **Kuryr Network Plugin**
  It is a part of the OpenStack Kuryr project, which also implements libnetwork's remote driver API by utilizing Neutron, which is OpenStack's networking service.

- In addition, we can write our own driver with Docker remote driver APIs.

### Kubernetes Networking

#### Concept

*“As we know, the smallest deployment unit in Kubernetes is a Pod, which can have one or more containers. Kubernetes assigns a unique IP address to each Pod. Containers in a Pod share the same network namespace and can refer to each other by localhost. We have seen an example of network namespaces sharing in the previous section. Containers in a Pod can expose unique ports, using which we can connect to individual containers using the same Pod IP.”*

#### Features

As each Pod gets a unique IP, Kubernetes assumes that Pods should be able to communicate with each other, irrespective of the nodes they get scheduled on. There are different ways to achieve this. Kubernetes uses the Container Network Interface (CNI) for container networking and has put down the following rules if someone wants to write drivers for Kubernetes networking:

- All pods can communicate with all other pods without NAT
- All nodes running pods can communicate with all pods (and vice versa) without NAT
- The IP that a pod sees itself as is the same IP that other pods see it as.

#### Components

- **Flannel**
  Flannel uses the overlay network, as we have seen with Docker, to meet the Kubernetes networking requirements.

- **Calico**
  Calico, or Project Calico, uses the BGP protocol to meet the Kubernetes networking requirements.

### Cloud Foundry: Container-to-Container Networking

#### Concept

*“Container-to-container also allows network policy creation and enforcement using which sets routing rules based on app, destination app, protocol and ports, without going through the Gorouter, a load balancer, or a firewall. ”*

#### Gorouter

By default, the *Gorouter* routes the external and internal traffic to different Cloud Foundry components. Even the application- to-application communication happens via the *Gorouter*. To simplify the app-to-app communication, we can enable container-to-container networking with Cloud Foundry. It implements it using Overlay Networking, which we discussed earlier. With Overlay Networking, each container gets its own unique IP address which is reachable from other application instances.

