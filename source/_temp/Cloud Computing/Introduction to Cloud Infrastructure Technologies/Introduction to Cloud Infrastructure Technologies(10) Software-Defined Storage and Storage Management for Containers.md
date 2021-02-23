---
title: \#Cloud Computing\# Introduction to Cloud Infrastructure Technologies(10) Software-Defined Storage and Storage Management for Containers
date: 2019-07-19 15:13:21
categories: Cloud Computing
tags:
- Database System
thumbnail: /images/cloud_computing.png
---



**Welcome to LFS151x: Introduction to Cloud Infrastructure Technologies**

By the end of this chapter, you should be able to:

- Discuss the basics of Software-Defined Storage.
- Discuss different storage options with containers using Docker, Kubernetes and Cloud Foundry.



<!-- more -->



## Software-Defined Storage

### Concept

*“Software-Defined Storage (SDS) is a form of storage virtualization in which the storage hardware is separated from the software which manages it. By doing this, we can bring hardware from different sources and we can manage them with software. Software can provide different features, like replication, erasure coding, snapshot, etc. on top of the pooled resources. Once the pooled storage is configured in the form of a cluster, SDS allows multiple access methods like File, Block, and Object. ”*

### Ceph

#### Concept

*“Ceph is a distributed object store and file system designed to provide excellent performance, reliability and scalability. Ceph is an open source defined and unified storage solution. It can deliver Object, Block and File system storage.”*

![SoftwareDefinedStorage_SDS](/images/SoftwareDefinedStorage_SDS.png)

#### Features

- It achieves higher throughput by stripping the files/data across the multiple nodes.
- It achieves adaptive load-balancing by replicating frequently accessed objects over multiple nodes.
- 

#### Components

- **Reliable Autonomic Distributed Object Store** (**RADOS**) 
  It is the object store which stores the objects. This layer makes sure that data is always in a consistent and reliable state. It performs operations like replication, failure detection, recovery, data migration, and rebalancing across the cluster nodes. This layer has the following three major components:
  - Object Storage Device (OSD): This is where the actual user content is written and retrieved with read operations. One OSD daemon is typically tied to one physical disk in the cluster.
  - Ceph Monitors (MON): Monitors are responsible for monitoring the cluster state. All cluster nodes report to Monitors. Monitors map the cluster state through the OSD, Place Groups (PG), CRUSH and Monitor maps.
  - Ceph Metadata Server (MDS): It is needed only by CephFS, to store the file hierarchy and metadata for files.

 ![Components_of_Storage_Clusters](/images/Components_of_Storage_Clusters.png)

- **Librados** 
  It is a library that allows direct access to RADOS from languages like C, C++, Python, Java, PHP, etc. Ceph Block Device*,* RADOSGW, and CephFS are implemented on top of Librados.
- **Ceph Block Device** 
  This provides the block interface for Ceph. It works as a block device and has enterprise features like thin provisioning and snapshots.
- **RADOS Gateway** (**RADOSGW**) 
  This provides a REST API interface for Ceph, which is compatible with Amazon S3 and OpenStack Swift.
- **Ceph File System** (**CephFS**) 
  This provides a POSIX-compliant distributed filesystem on top of Ceph. It relies on Ceph MDS to track the file hierarchy.

#### Architecture

Everything in Ceph is stored as objects. Ceph uses the CRUSH (Controlled Replication Under Scalable Hashing) algorithm to deterministically find out, write, and read the location of objects.

![Ceph_Architecture](/images/Ceph_Architecture.png)

### GlusterFS

#### Concept

*“Gluster is a scalable, distributed file system that aggregates disk storage resources from multiple servers into a single global namespace.”*

#### Features

- GlusterFS does not have a centralized metadata server. It uses an elastic [hashing](https://en.wikipedia.org/wiki/Hash_function) algorithm to store files on bricks.

#### Components

The GlusterFS volume can be accessed using one of the following methods:

- Native FUSE mount
- NFS (Network File System)
- CIFS (Common Internet File System).

#### Architectures

To create shared storage, we need to start by grouping the machines in a trusted pool. Then, we group the directories (called *bricks*) from those machines in a GlusterFS volume, using [FUSE (Filesystem in Userspace)](https://en.wikipedia.org/wiki/Filesystem_in_Userspace). GlusterFS supports different kinds of volumes:

- distributed GlusterFS volume
- replicated GlusterFS volume
- distributed replicated GlusterFS volume
- dispersed GlusterFS volume
- distributed dispersed GlusterFS volume.

![Distributed_GlusterFS_Volume](/images/Distributed_GlusterFS_Volume.png)

## Storage Management for Containers

### Background

Containers are ephemeral in nature, which means that whatever is stored inside the container would be gone as soon as the container is deleted. It is best practice to store data outside the container, which would be accessible even after the container is gone.

In a multi-host environment, containers can be scheduled to run on any host. We need to make sure the data volume required by the container is available on the node on which the container would be running. 

### Docker Volume Plugins

#### Backends

Docker uses [copy-on-write](https://en.wikipedia.org/wiki/Copy-on-write) to start containers from images, which means we do not have to copy an image while starting a container. All of the changes done by a container are saved in a filesystem layer. Docker images and containers are stored on the host system and we can choose the storage backend for Docker storage, depending on our requirements. Docker supports the following storage backends on Linux: 

- AUFS (Another Union File System)
- BtrFS
- Device Mapper
- Overlay
- VFS (Virtual File System)
- ZFS
- windowsfilter.

#### Components

Docker primarily supports two options for storing files on a host system:

- **Docker Volumes**

  On Linux, Docker Volumes are stored under the **/var/lib/docker/volumes** folder and they are directly managed by Docker. 

- **Bind Mounts**

  Using Bind Mounts, Docker can mount any file or directory from the *host system* into a container.

In both cases, Docker bypasses the Union Filesystem, which it uses for *copy-on-write.* The writes happen directly to the host directory.  

#### Plugins

On Linux, Docker also supports *tmpfs* mounts, which are available in the host system's memory. Docker also supports third party volume plugins. Some examples of volume plugins are:

- GlusterFS
- Blockbridge
- EMC REX-Ray.

Volume plugins are especially helpful when we migrate a stateful container, like a database, on a multi-host environment. In such an environment, we have to make sure that the volume attached to a container is also migrated to the host where the container is migrated or started afresh.

#### Commands

To create a container with volume, we can use either the **docker container run** or the**docker container create** commands, like the following:

```powershell
$ docker container run -d --name web -v /webapp nkhare/myapp
```

The above command would create a Docker volume inside the Docker working directory (default to **/var/lib/docker/**) on the host system. We can get the exact path via the**docker container inspect** command:

```powershell
$ docker container inspect web
```

We can give a specific name to a Docker volume and then use it for different operations. To create a named volume*,* we can run the following command:

```powershell
$ docker volume create --name my-named-volume
```

and then mount it. To do a bind mount*,* we can run the following command:

```powershell
$ docker container run -d --name web -v /mnt/webapp:/webapp nkhare/myapp
```

which would mount the host's **/mnt/webapp** directory to **/myapp**, while starting the container. 

### Volume Management in Kubernetes

#### Concept

*“Kubernetes uses volumes to attach external storage to the Pods. A volume is essentially a directory, backed by a storage medium. The storage medium and its contents are determined by the volume type.”*

![kubernetes_volumes](/images/kubernetes_volumes.png)

A volume in Kubernetes is attached to a Pod and shared among containers of that Pod. The volume has the same lifetime as the Pod and it can outlive the containers of that Pod, which allows data to be preserved across container restarts.

#### Types

A directory which is mounted inside a Pod is backed by the underlying volume type. A volume type decides the properties of the directory, like size, content, etc. Some of the volume types are:

- **emptyDir**
  An empty volume is created for the Pod as soon as it is scheduled on a worker node. The life of the volume is tightly coupled with the Pod. If the Pod dies, the content of **emptyDir** is deleted forever.

- **hostPath**
  With the **hostPath** volume type, we can share a directory from the host to the Pod. If the Pod dies, the content of the volume is still available on the host.

- **gcePersistentDisk**
  With the **gcePersistentDisk** volume type, we can mount a [Google Compute Engine (GCE)](https://cloud.google.com/compute/docs/disks/) persistent disk into a Pod.

- **awsElasticBlockStore**
  With the **awsElasticBlockStore** volume type, we can mount an [AWS EBS](https://aws.amazon.com/ebs/) volume into a Pod.

- **azureFile**

  With the **azureFile** volume type, we can mount a [Azure](https://docs.azure.cn/zh-cn/virtual-machine-scale-sets/virtual-machine-scale-sets-attached-disks) persistent disk into a Pod.

- **nfs**
  With the **nfs** volume type, we can mount an NFS share into a Pod.

- **persistentVolumeClaim**
  With the **persistentVolumeClaim** type we can attach a persistent volume to a Pod. 

#### Persistent Volumes

##### Background

In a typical IT environment, storage is managed by the storage/system administrators. The end-user gets instructions to use the storage and he/she does not have to worry about the underlying storage management.

##### Static control

In the containerized world, we would like to follow similar rules, but it becomes very challenging given the many volume types we have seen earlier. In Kubernetes, this problem is solved with the persistent volume subsystem, which provides APIs to manage and consume the storage. To manage the volume it uses the *PersistentVolume* (PV) resource type and to consume it, it uses the *PersistentVolumeClaim* (PVC) resource type.

Persistent volumes can be provisioned manually or dynamically. For example, a Kubernetes Administrator has created a few PVs upfront.

![persistent_volumes](/images/persistent_volumes.png)

A *PersistentVolumeClaim* (PVC) is a request for storage by a user. Users request for PV resources based on size, access modes, etc. Once a suitable PV is found, it is bound to PVC. After a successful bound, PVC can be used in a Pod. Once a user is done with work, the attached PVC can be released. Then, the underlying PV can be reclaimed and recycled for future usage.

![persistent_volume_claim](/images/persistent_volume_claim.png)

##### Dynamic Control

For dynamic provisioning of PVs, Kubernetes uses the *StorageClass* resource, which contains pre-defined provisioners and parameters for the PV creation. With *PersistentVolumeClaim* (PVC), a user sends the requests for dynamic PV creation, which gets wired to the *StorageClass* resource.

Some of the volume types that support managing storage using PersistentVolume are:

- GCEPersistentDisk
- AWSElasticBlockStore
- AzureFile
- NFS
- iSCSI.

### Cloud Foundry Volume Service

#### Concept

*“On Cloud Foundry, applications connect to other services via a service marketplace. Each service has a service broker, which encapsulates the logic for creating, managing, and binding services to applications. With volume services, the volume service broker allows Cloud Foundry applications to attach external storage.”*

![cf-volume-service](/images/cf-volume-service.png)

#### Components

With the **volume bind** command, the service broker issues a volume mount instruction which instructs the Diego scheduler to schedule the app instances on cells which have the appropriate volume driver. In the backend, the volume driver gets attached to the device. Cells then mount the device into the container and start the app instance.

Following are some examples of CF Volume Services:

- **Local-volume-release** 
It provides local-volume service for Cloud Foundry applications.
- **nfs-volume-release**
This allows easy mounting of external NFS shares for Cloud Foundry applications.
- **Cephfs-bosh-release** 
It is an open source shared volumes service, built on top of Ceph.
- **Efs-volume-release** 
It provides driver and service broker for provisioning and mounting Amazon EFS volumes.

### Container Storage Interface (CSI)

#### Concept

*“For a storage vendor, it is difficult to manage different volume plugins for different orchestrators. Storage vendors and community members from different orchestrators are working together to standardize the volume interface to avoid duplicate work. This is being referred to as the **Container Storage Interface** (CSI). With CSI, the same volume plugin would work for different container Oorchestrators.”*