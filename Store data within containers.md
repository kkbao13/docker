#　Store data within containers

# About storage drivers

   Estimated reading time:            14 minutes    

To use storage drivers effectively, it’s important to know how Docker builds and stores images, and how these images are used by containers. You can use this information to make informed choices about the best way to persist data from your applications and avoid performance problems along the way.

Storage drivers allow you to create data in the writable layer of your container. The files won’t be persisted after the container is deleted, and both read and write speeds are low.

[Learn how to use volumes](https://docs.docker.com/storage/volumes/) to persist data and improve performance.

要有效地使用**storage drivers**存储驱动程序，了解Docker如何**构建和存储**图像以及容器如何使用这些图像非常重要。您可以使用此信息做出明智的选择，以确定从应用程序中保留数据的最佳方法，并避免在此过程中出现**性能问题**。存储驱动程序允许您在容器的**可写层**中创建数据。删除容器后，文件将**不会保留**，并且**读取和写入速度都很低**。



## Images and layers

A Docker image is built up from a series of layers. Each layer represents an instruction in the image’s Dockerfile. Each layer except the very last one is read-only. Consider the following Dockerfile:

Docker镜像是从一系列图层构建的。每个图层代表图像Dockerfile中的指令。除最后一层之外的每一层都是只读的。考虑以下Dockerfile：

```dockerfile
FROM ubuntu:15.04
COPY . /app
RUN make /app
CMD python /app/app.py
```

This Dockerfile contains four commands, each of which creates a layer. The `FROM` statement starts out by creating a layer from the `ubuntu:15.04` image. The `COPY` command adds some files from your Docker client’s current directory. The `RUN` command builds your application using the `make` command. Finally, the last layer specifies what command to run within the container.

此Dockerfile包含四个命令，每个命令都创建一个层。 FROM语句首先从ubuntu：15.04映像创建一个层。 COPY命令添加Docker客户端当前目录中的一些文件。 RUN命令使用make命令构建应用程序。最后，最后一层指定在容器中运行的命令。

Each layer is only a set of differences from the layer before it. The layers are stacked on top of each other. When you create a new container, you add a new writable layer on top of the underlying layers. This layer is often called the “container layer”. All changes made to the running container, such as writing new files, modifying existing files, and deleting files, are written to this thin writable container layer. The diagram below shows a container based on the Ubuntu 15.04 image.

每个图层只是与之前图层的一组差异。这些层堆叠在彼此之上。创建新容器时，可以在基础图层的顶部添加新的可写层。该层通常称为“容器层”。对正在运行的容器所做的所有更改（例如，写入新文件，修改现有文件和删除文件）都将写入此可写容器层。下图显示了基于Ubuntu
15.04映像的容器。

![Layers of a container based on the Ubuntu image](https://docs.docker.com/storage/storagedriver/images/container-layers.jpg)

A *storage driver* handles the details about the way these layers interact with each other. Different storage drivers are available, which have advantages and disadvantages in different situations.

存储驱动程序处理有关这些层彼此交互方式的详细信息。可以使用不同的存储驱动程序，它们在不同情况下具有优点和缺点。

## Container and layers

The major difference between a container and an image is the top writable layer. All writes to the container that add new or modify existing data are stored in this writable layer. When the container is deleted, the writable layer is also deleted. The underlying image remains unchanged.

Because each container has its own writable container layer, and all changes are stored in this container layer, multiple containers can share access to the same underlying image and yet have their own data state. The diagram below shows multiple containers sharing the same Ubuntu 15.04 image.

![Containers sharing same image](https://docs.docker.com/storage/storagedriver/images/sharing-layers.jpg)

> **Note**: If you need multiple images to have shared access to the exact same data, store this data in a Docker volume and mount it into your containers.

Docker uses storage drivers to manage the contents of the image layers and the writable container layer. Each storage driver handles the implementation differently, but all drivers use stackable image layers and the copy-on-write (CoW) strategy.

## Container size on disk

To view the approximate size of a running container, you can use the `docker ps -s` command. Two different columns relate to size.

- `size`: the amount of data (on disk) that is used for the writable layer of each container.
- `virtual size`: the amount of data used for the read-only image data used by the container plus the container’s writable layer `size`. Multiple containers may share some or all read-only image data. Two containers started from the same image share 100% of the read-only data, while two containers with different images which have layers in common share those common layers. Therefore, you can’t just total the virtual sizes. This over-estimates the total disk usage by a potentially non-trivial amount.

The total disk space used by all of the running containers on disk is some combination of each container’s `size` and the `virtual size` values. If multiple containers started from the same exact image, the total size on disk for these containers would be SUM (`size` of containers) plus one image size (`virtual size`- `size`).

This also does not count the following additional ways a container can take up disk space:

- Disk space used for log files if you use the `json-file` logging driver. This can be non-trivial if your container generates a large amount of logging data and log rotation is not configured.
- Volumes and bind mounts used by the container.
- Disk space used for the container’s configuration files, which are typically small.
- Memory written to disk (if swapping is enabled).
- Checkpoints, if you’re using the experimental checkpoint/restore feature.

## The copy-on-write (CoW) strategy

Copy-on-write is a strategy of sharing and copying files for maximum efficiency. If a file or directory exists in a lower layer within the image, and another layer (including the writable layer) needs read access to it, it just uses the existing file. The first time another layer needs to modify the file (when building the image or running the container), the file is copied into that layer and modified. This minimizes I/O and the size of each of the subsequent layers. These advantages are explained in more depth below.

### Sharing promotes smaller images

When you use `docker pull` to pull down an image from a repository, or when you create a container from an image that does not yet exist locally, each layer is pulled down separately, and stored in Docker’s local storage area, which is usually `/var/lib/docker/` on Linux hosts. You can see these layers being pulled in this example:

```
$ docker pull ubuntu:15.04

15.04: Pulling from library/ubuntu
1ba8ac955b97: Pull complete
f157c4e5ede7: Pull complete
0b7e98f84c4c: Pull complete
a3ed95caeb02: Pull complete
Digest: sha256:5e279a9df07990286cce22e1b0f5b0490629ca6d187698746ae5e28e604a640e
Status: Downloaded newer image for ubuntu:15.04
```

Each of these layers is stored in its own directory inside the Docker host’s local storage area. To examine the layers on the filesystem, list the contents of `/var/lib/docker/<storage-driver>/layers/`. This example uses the `aufs`  storage driver:

```
$ ls /var/lib/docker/aufs/layers
1d6674ff835b10f76e354806e16b950f91a191d3b471236609ab13a930275e24
5dbb0cbe0148cf447b9464a358c1587be586058d9a4c9ce079320265e2bb94e7
bef7199f2ed8e86fa4ada1309cfad3089e0542fec8894690529e4c04a7ca2d73
ebf814eccfe98f2704660ca1d844e4348db3b5ccc637eb905d4818fbfb00a06a
```

The directory names do not correspond to the layer IDs (this has been true since Docker 1.10).

Now imagine that you have two different Dockerfiles. You use the first one to create an image called `acme/my-base-image:1.0`.

```
FROM ubuntu:16.10
COPY . /app
```

The second one is based on `acme/my-base-image:1.0`, but has some additional layers:

```
FROM acme/my-base-image:1.0
CMD /app/hello.sh
```

The second image contains all the layers from the first image, plus a new layer with the `CMD` instruction, and a read-write container layer. Docker already has all the layers from the first image, so it does not need to pull them again. The two images share any layers they have in common.

If you build images from the two Dockerfiles, you can use `docker image ls` and `docker history` commands to verify that the cryptographic IDs of the shared layers are the same.

1. Make a new directory `cow-test/` and change into it.

2. Within `cow-test/`, create a new file with the following contents:

   ```
   #!/bin/sh
   echo "Hello world"
   ```

   Save the file, and make it executable:

   ```
   chmod +x hello.sh
   ```

3. Copy the contents of the first Dockerfile above into a new file called `Dockerfile.base`.

4. Copy the contents of the second Dockerfile above into a new file called `Dockerfile`.

5. Within the `cow-test/` directory, build the first image. Don’t forget to include the final `.` in the command. That sets the `PATH`, which tells Docker where to look for any files that need to be added to the image.

   ```
   $ docker build -t acme/my-base-image:1.0 -f Dockerfile.base .
   
   Sending build context to Docker daemon  4.096kB
   Step 1/2 : FROM ubuntu:16.10
    ---> 31005225a745
   Step 2/2 : COPY . /app
    ---> Using cache
    ---> bd09118bcef6
   Successfully built bd09118bcef6
   Successfully tagged acme/my-base-image:1.0
   ```

6. Build the second image.

   ```
   $ docker build -t acme/my-final-image:1.0 -f Dockerfile .
   
   Sending build context to Docker daemon  4.096kB
   Step 1/2 : FROM acme/my-base-image:1.0
    ---> bd09118bcef6
   Step 2/2 : CMD /app/hello.sh
    ---> Running in a07b694759ba
    ---> dbf995fc07ff
   Removing intermediate container a07b694759ba
   Successfully built dbf995fc07ff
   Successfully tagged acme/my-final-image:1.0
   ```

7. Check out the sizes of the images:

   ```
   $ docker image ls
   
   REPOSITORY                         TAG                     IMAGE ID            CREATED             SIZE
   acme/my-final-image                1.0                     dbf995fc07ff        58 seconds ago      103MB
   acme/my-base-image                 1.0                     bd09118bcef6        3 minutes ago       103MB
   ```

8. Check out the layers that comprise each image:

   ```
   $ docker history bd09118bcef6
   IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
   bd09118bcef6        4 minutes ago       /bin/sh -c #(nop) COPY dir:35a7eb158c1504e...   100B                
   31005225a745        3 months ago        /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B                  
   <missing>           3 months ago        /bin/sh -c mkdir -p /run/systemd && echo '...   7B                  
   <missing>           3 months ago        /bin/sh -c sed -i 's/^#\s*\(deb.*universe\...   2.78kB              
   <missing>           3 months ago        /bin/sh -c rm -rf /var/lib/apt/lists/*          0B                  
   <missing>           3 months ago        /bin/sh -c set -xe   && echo '#!/bin/sh' >...   745B                
   <missing>           3 months ago        /bin/sh -c #(nop) ADD file:eef57983bd66e3a...   103MB      
   ```

   ```
   $ docker history dbf995fc07ff
   
   IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
   dbf995fc07ff        3 minutes ago       /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "/a...   0B                  
   bd09118bcef6        5 minutes ago       /bin/sh -c #(nop) COPY dir:35a7eb158c1504e...   100B                
   31005225a745        3 months ago        /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B                  
   <missing>           3 months ago        /bin/sh -c mkdir -p /run/systemd && echo '...   7B                  
   <missing>           3 months ago        /bin/sh -c sed -i 's/^#\s*\(deb.*universe\...   2.78kB              
   <missing>           3 months ago        /bin/sh -c rm -rf /var/lib/apt/lists/*          0B                  
   <missing>           3 months ago        /bin/sh -c set -xe   && echo '#!/bin/sh' >...   745B                
   <missing>           3 months ago        /bin/sh -c #(nop) ADD file:eef57983bd66e3a...   103MB  
   ```

   Notice that all the layers are identical except the top layer of the second image. All the other layers are shared between the two images, and are only stored once in `/var/lib/docker/`. The new layer actually doesn’t take any room at all, because it is not changing any files, but only running a command.

   > **Note**: The `<missing>` lines in the `docker history` output indicate that those layers were built on another system and are not available locally. This can be ignored.

### Copying makes containers efficient

When you start a container, a thin writable container layer is added on top of the other layers. Any changes the container makes to the filesystem are stored here. Any files the container does not change do not get copied to this writable layer. This means that the writable layer is as small as possible.

When an existing file in a container is modified, the storage driver performs a copy-on-write operation. The specifics steps involved depend on the specific storage driver. For the `aufs`, `overlay`, and `overlay2` drivers, the  copy-on-write operation follows this rough sequence:

- Search through the image layers for the file to update. The process starts at the newest layer and works down to the base layer one layer at a time. When results are found, they are added to a cache to speed future operations.
- Perform a `copy_up` operation on the first copy of the file that is found, to copy the file to the container’s writable layer.
- Any modifications are made to this copy of the file, and the container cannot see the read-only copy of the file that exists in the lower layer.

Btrfs, ZFS, and other drivers handle the copy-on-write differently. You can read more about the methods of these drivers later in their detailed descriptions.

Containers that write a lot of data consume more space than containers that do not. This is because most write operations consume new space in the container’s thin writable top layer.

> **Note**: for write-heavy applications, you should not store the data in the container. Instead, use Docker volumes, which are independent of the running container and are designed to be efficient for I/O. In addition, volumes can be shared among containers and do not increase the size of your container’s writable layer.

A `copy_up` operation can incur a noticeable performance overhead. This overhead is different depending on which storage driver is in use. Large files, lots of layers, and deep directory trees can make the impact more noticeable. This is mitigated by the fact that each `copy_up` operation only occurs the first time a given file is modified.

To verify the way that copy-on-write works, the following procedures spins up 5 containers based on the `acme/my-final-image:1.0` image we built earlier and examines how much room they take up.

> **Note**: This procedure doesn’t work on Docker Desktop for Mac or Docker Desktop for Windows.

1. From a terminal on your Docker host, run the following `docker run` commands. The strings at the end are the IDs of each container.

   ```
   $ docker run -dit --name my_container_1 acme/my-final-image:1.0 bash \
     && docker run -dit --name my_container_2 acme/my-final-image:1.0 bash \
     && docker run -dit --name my_container_3 acme/my-final-image:1.0 bash \
     && docker run -dit --name my_container_4 acme/my-final-image:1.0 bash \
     && docker run -dit --name my_container_5 acme/my-final-image:1.0 bash
   
     c36785c423ec7e0422b2af7364a7ba4da6146cbba7981a0951fcc3fa0430c409
     dcad7101795e4206e637d9358a818e5c32e13b349e62b00bf05cd5a4343ea513
     1e7264576d78a3134fbaf7829bc24b1d96017cf2bc046b7cd8b08b5775c33d0c
     38fa94212a419a082e6a6b87a8e2ec4a44dd327d7069b85892a707e3fc818544
     1a174fc216cccf18ec7d4fe14e008e30130b11ede0f0f94a87982e310cf2e765
   ```

2. Run the `docker ps` command to verify the 5 containers are running.

   ```
   CONTAINER ID      IMAGE                     COMMAND     CREATED              STATUS              PORTS      NAMES
   1a174fc216cc      acme/my-final-image:1.0   "bash"      About a minute ago   Up About a minute              my_container_5
   38fa94212a41      acme/my-final-image:1.0   "bash"      About a minute ago   Up About a minute              my_container_4
   1e7264576d78      acme/my-final-image:1.0   "bash"      About a minute ago   Up About a minute              my_container_3
   dcad7101795e      acme/my-final-image:1.0   "bash"      About a minute ago   Up About a minute              my_container_2
   c36785c423ec      acme/my-final-image:1.0   "bash"      About a minute ago   Up About a minute              my_container_1
   ```

3. List the contents of the local storage area.

   ```
   $ sudo ls /var/lib/docker/containers
   
   1a174fc216cccf18ec7d4fe14e008e30130b11ede0f0f94a87982e310cf2e765
   1e7264576d78a3134fbaf7829bc24b1d96017cf2bc046b7cd8b08b5775c33d0c
   38fa94212a419a082e6a6b87a8e2ec4a44dd327d7069b85892a707e3fc818544
   c36785c423ec7e0422b2af7364a7ba4da6146cbba7981a0951fcc3fa0430c409
   dcad7101795e4206e637d9358a818e5c32e13b349e62b00bf05cd5a4343ea513
   ```

4. Now check out their sizes:

   ```
   $ sudo du -sh /var/lib/docker/containers/*
   
   32K  /var/lib/docker/containers/1a174fc216cccf18ec7d4fe14e008e30130b11ede0f0f94a87982e310cf2e765
   32K  /var/lib/docker/containers/1e7264576d78a3134fbaf7829bc24b1d96017cf2bc046b7cd8b08b5775c33d0c
   32K  /var/lib/docker/containers/38fa94212a419a082e6a6b87a8e2ec4a44dd327d7069b85892a707e3fc818544
   32K  /var/lib/docker/containers/c36785c423ec7e0422b2af7364a7ba4da6146cbba7981a0951fcc3fa0430c409
   32K  /var/lib/docker/containers/dcad7101795e4206e637d9358a818e5c32e13b349e62b00bf05cd5a4343ea513
   ```

   Each of these containers only takes up 32k of space on the filesystem.

Not only does copy-on-write save space, but it also reduces start-up time. When you start a container (or multiple containers from the same image), Docker only needs to create the thin writable container layer.

If Docker had to make an entire copy of the underlying image stack each time it started a new container, container start times and disk space used would be significantly increased. This would be similar to the way that virtual machines work, with one or more virtual disks per virtual machine.

# Docker storage drivers

Ideally, very little data is written to a container’s writable layer, and you use Docker volumes to write data. However, some workloads require you to be able to write to the container’s writable layer. This is where storage drivers come in.

Docker supports several different storage drivers, using a pluggable architecture. The storage driver controls how images and containers are stored and managed on your Docker host.

理想情况下，将非常少的数据写入容器的可写层，并使用Docker卷来写入数据。但是，某些工作负载要求您能够写入容器的可写层。这是存储驱动程序的用武之地。

Docker使用可插拔架构支持多种不同的存储驱动程序。存储驱动程序控制在Docker主机上存储和管理映像和容器的方式。

After you have read the [storage driver overview](https://docs.docker.com/storage/storagedriver/), the next step is to choose the best storage driver for your workloads. In making this decision, there are three high-level factors to consider:

If multiple storage drivers are supported in your kernel, Docker has a prioritized  list of which storage driver to use if no storage driver is explicitly configured,  assuming that the storage driver meets the prerequisites.

Use the storage driver with the best overall performance and stability in the most  usual scenarios.

阅读完存储驱动程序概述后，下一步是为工作负载选择最佳存储驱动程序。在做出此决定时，需要考虑三个高级别因素：

如果内核支持多个存储驱动程序，则假设存储驱动程序满足先决条件，则Docker会在没有显式配置存储驱动程序的情况下列出要使用哪个存储驱动程序的优先级列表。

在最常见的情况下，使用具有最佳整体性能和稳定性的存储驱动程序。

Docker supports the following storage drivers:

- `overlay2` is the preferred storage driver, for all currently supported  Linux distributions, and requires no extra configuration.
- overlay2是所有当前支持的Linux发行版的首选存储驱动程序，无需额外配置。
- `aufs` is the preferred storage driver for Docker 18.06 and older, when  running on Ubuntu 14.04 on kernel 3.13 which has no support for `overlay2`.
- aufs是Docker 18.06及更早版本的首选存储驱动程序，在内核3.13上运行Ubuntu 14.04时不支持overlay2。
- `devicemapper` is supported, but requires `direct-lvm` for production  environments, because `loopback-lvm`, while zero-configuration, has very  poor performance. `devicemapper` was the recommended storage driver for  CentOS and RHEL, as their kernel version did not support `overlay2`. However,  current versions of CentOS and RHEL now have support for `overlay2`,  which is now the recommended driver.
- 支持devicemapper，但生产环境需要direct-lvm，因为loopback-lvm虽然是零配置，但性能非常差。 devicemapper是CentOS和RHEL的推荐存储驱动程序，因为它们的内核版本不支持overlay2。但是，当前版本的CentOS和RHEL现在支持overlay2，现在推荐使用它。
- The `btrfs` and `zfs` storage drivers are used if they are the backing filesystem (the filesystem of the host on which Docker is installed). These filesystems allow for advanced options, such as creating “snapshots”, but require more maintenance and setup. Each of these relies on the backing filesystem being configured correctly.
- 如果btrfs和zfs存储驱动程序是后备文件系统（安装Docker的主机的文件系统），则使用它们。这些文件系统允许高级选项，例如创建“快照”，但需要更多维护和设置。其中每个都依赖于正确配置的后备文件系统。
- The `vfs` storage driver is intended for testing purposes, and for situations where no copy-on-write filesystem can be used. Performance of this storage driver is poor, and is not generally recommended for production use.
- vfs存储驱动程序用于测试目的，适用于无法使用copy-on-write文件系统的情况。此存储驱动程序的性能很差，通常不建议用于生产。

Docker’s source code defines the selection order. You can see the order at [the source code for Docker Engine - Community 18.09](https://github.com/docker/docker-ce/blob/18.09/components/engine/daemon/graphdriver/driver_linux.go#L50)

If you run a different version of Docker,  you can use the branch selector at the top of the file viewer to choose a  different branch.

Some storage drivers require you to use a specific format for the backing filesystem. If you have external  requirements to use a specific backing filesystem, this may limit your choices. See [Supported backing filesystems](https://docs.docker.com/storage/storagedriver/select-storage-driver/#supported-backing-filesystems).

After you have narrowed down which storage drivers you can choose from, your choice is determined by the  characteristics of your workload and the level of stability you need. See [Other considerations](https://docs.docker.com/storage/storagedriver/select-storage-driver/#other-considerations) for help in making the final decision.

Docker的源代码定义了选择顺序。您可以在Docker Engine  -  Community 18.09的源代码中查看订单。

如果运行不同版本的Docker，则可以使用文件查看器顶部的分支选择器来选择其他分支。

某些存储驱动程序要求您使用特定格式作为后备文件系统。如果您有使用特定支持文件系统的外部要求，则可能会限制您的选择。请参阅支持的后备文件系统

在缩小了可供选择的存储驱动程序之后，您的选择取决于工作负载的特性和所需的稳定性级别。请参阅其他注意事项，以帮助您做出最终决定。

> **NOTE**: Your choice may be limited by your Docker edition, operating system, and distribution.  For instance, `aufs` is only supported on Ubuntu and Debian, and may require extra packages  to be installed, while `btrfs` is only supported on SLES, which is only supported with Docker Enterprise. See [Support storage drivers per Linux distribution](https://docs.docker.com/storage/storagedriver/select-storage-driver/#supported-storage-drivers-per-linux-distribution)  for more information.
>
> 注意：您的选择可能受Docker版本，操作系统和分发版的限制。例如，aufs仅在Ubuntu和Debian上受支持，并且可能需要安装额外的软件包，而btrfs仅在SLES上受支持，SLES仅支持Docker
> Enterprise。有关更多信息，请参阅每个Linux发行版的支持存储驱

## Supported storage drivers per Linux distribution

At a high level, the storage drivers you can use is partially determined by the Docker edition you use.

In addition, Docker does not recommend any configuration that requires you to disable security features of your operating system, such as the need to disable `selinux` if you use the `overlay` or `overlay2` driver on CentOS.

在较高的层次上，您可以使用的存储驱动程序部分取决于您使用的Docker版本。此外，Docker不建议任何需要您禁用操作系统安全功能的配置，例如需要禁用selinux您在CentOS上使用overlay或overlay2驱动程序。

### Docker Engine - Enterprise and Docker Enterprise

For Docker Engine - Enterprise and Docker Enterprise, the definitive resource for which storage drivers are supported is the [Product compatibility matrix](https://success.docker.com/Policies/Compatibility_Matrix). To get commercial support from Docker, you must use a supported configuration.

对于Docker Engine  -  Enterprise和Docker Enterprise，支持存储驱动程序的权威资源是产品兼容性矩阵。要从Docker获得商业支持，您必须使用支持的配置。

### Docker Engine - Community

For Docker Engine - Community, only some configurations are tested, and your operating system’s kernel may not support every storage driver. In general, the following configurations work on recent versions of the Linux distribution:

对于Docker Engine  -  Community，只测试了一些配置，并且您的操作系统内核可能不支持每个存储驱动程序。通常，以下配置适用于最新版本的Linux发行版：

| Linux distribution                  | Recommended storage drivers                                  | Alternative drivers                       |
| :---------------------------------- | :----------------------------------------------------------- | :---------------------------------------- |
| Docker Engine - Community on Ubuntu | `overlay2` or `aufs` (for Ubuntu 14.04 running on kernel 3.13) | `overlay`¹, `devicemapper`², `zfs`, `vfs` |
| Docker Engine - Community on Debian | `overlay2` (Debian Stretch), `aufs` or `devicemapper` (older versions) | `overlay`¹, `vfs`                         |
| Docker Engine - Community on CentOS | `overlay2`                                                   | `overlay`¹, `devicemapper`², `zfs`, `vfs` |
| Docker Engine - Community on Fedora | `overlay2`                                                   | `overlay`¹, `devicemapper`², `zfs`, `vfs` |

¹) The `overlay` storage driver is deprecated in Docker Engine - Enterprise 18.09, and will be removed in a future release. It is recommended that users of the `overlay` storage driver  migrate to `overlay2`.

²) The `devicemapper` storage driver is deprecated in Docker Engine 18.09, and will be removed in a future release. It is recommended that users of the `devicemapper` storage driver  migrate to `overlay2`.

When possible, `overlay2` is the recommended storage driver. When installing Docker for the first time, `overlay2` is used by default. Previously, `aufs` was used by default when available, but this is no longer the case. If you want to use `aufs` on new installations going forward, you need to explicitly configure it, and you may need to install extra packages, such as `linux-image-extra`. See [aufs](https://docs.docker.com/storage/storagedriver/aufs-driver/).

On existing installations using `aufs`, it is still used.

When in doubt, the best all-around configuration is to use a modern Linux distribution with a kernel that supports the `overlay2` storage driver, and to use Docker volumes for write-heavy workloads instead of relying on writing data to the container’s writable layer.

The `vfs` storage driver is usually not the best choice. Before using the `vfs` storage driver, be sure to read about [its performance and storage characteristics and limitations](https://docs.docker.com/storage/storagedriver/vfs-driver/).

> **Expectations for non-recommended storage drivers**: Commercial support is not available for Docker Engine - Community, and you can technically use any storage driver that is available for your platform. For instance, you can use `btrfs` with Docker Engine - Community, even though it is not recommended on any platform for Docker Engine - Community, and you do so at your own risk.
>
> The recommendations in the table above are based on automated regression testing and the configurations that are known to work for a large number of users. If you use a recommended configuration and find a reproducible issue, it is likely to be fixed very quickly. If the driver that you want to use is not recommended according to this table, you can run it at your own risk. You can and should still report any issues you run into. However, such issues have a lower priority than issues encountered when using a recommended configuration.

### Docker Desktop for Mac and Docker Desktop for Windows

Docker Desktop for Mac and Docker Desktop for Windows are intended for development, rather than production. Modifying the storage driver on these platforms is not possible.

## Supported backing filesystems

With regard to Docker, the backing filesystem is the filesystem where `/var/lib/docker/` is located. Some storage drivers only work with specific backing filesystems.

| Storage driver        | Supported backing filesystems |
| :-------------------- | :---------------------------- |
| `overlay2`, `overlay` | `xfs` with ftype=1, `ext4`    |
| `aufs`                | `xfs`, `ext4`                 |
| `devicemapper`        | `direct-lvm`                  |
| `btrfs`               | `btrfs`                       |
| `zfs`                 | `zfs`                         |
| `vfs`                 | any filesystem                |

## Other considerations

### Suitability for your workload

Among other things, each storage driver has its own performance characteristics that make it more or less suitable for different workloads. Consider the following generalizations:

除此之外，每个存储驱动程序都有自己的性能特征，使其或多或少适合不同的工作负载。请考虑以下概括：

- `overlay2`, `aufs`, and `overlay` all operate at the file level rather than the block level. This uses memory more efficiently, but the container’s writable layer may grow quite large in write-heavy workloads.
- Block-level storage drivers such as `devicemapper`, `btrfs`, and `zfs` perform better for write-heavy workloads (though not as well as Docker volumes).
- overlay2，aufs和overlay都在文件级而不是块级操作。这样可以更有效地使用内存，但容器的可写层在写入较多的工作负载中可能会变得非常大。诸如devicemapper，btrfs和zfs之类的块级存储驱动程序对于大量写入的工作负载表现更好（尽管不如Docker卷那么好）。
- For lots of small writes or containers with many layers or deep filesystems, `overlay` may perform better than `overlay2`, but consumes more inodes, which can lead to inode exhaustion.
- `btrfs` and `zfs` require a lot of memory.
- `zfs` is a good choice for high-density workloads such as PaaS.

More information about performance, suitability, and best practices is available in the documentation for each storage driver.

### Shared storage systems and the storage driver

If your enterprise uses SAN, NAS, hardware RAID, or other shared storage systems, they may provide high availability, increased performance, thin provisioning, deduplication, and compression. In many cases, Docker can work on top of these storage systems, but Docker does not closely integrate with them.

Each Docker storage driver is based on a Linux filesystem or volume manager. Be sure to follow existing best practices for operating your storage driver (filesystem or volume manager) on top of your shared storage system. For example, if using the ZFS storage driver on top of a shared storage system, be sure to follow best practices for operating ZFS filesystems on top of that specific shared storage system.

### Stability

For some users, stability is more important than performance. Though Docker considers all of the storage drivers mentioned here to be stable, some are newer and are still under active development. In general, `overlay2`, `aufs`, `overlay`, and `devicemapper` are the choices with the highest stability.

### Test with your own workloads

You can test Docker’s performance when running your own workloads on different storage drivers. Make sure to use equivalent hardware and workloads to match production conditions, so you can see which storage driver offers the best overall performance.

## Check your current storage driver

The detailed documentation for each individual storage driver details all of the set-up steps to use a given storage driver.

To see what storage driver Docker is currently using, use `docker info` and look for the `Storage Driver` line:

```
$ docker info

Containers: 0
Images: 0
Storage Driver: overlay2
 Backing Filesystem: xfs
<output truncated>
```

To change the storage driver, see the specific instructions for the new storage driver. Some drivers require additional configuration, including configuration to physical or logical disks on the Docker host.

> **Important**: When you change the storage driver, any existing images and containers become inaccessible. This is because their layers cannot be used by the new storage driver. If you revert your changes, you can access the old images and containers again, but any that you pulled or created using the new driver are then inaccessible.



# Use the OverlayFS storage driver

   Estimated reading time:            18 minutes    

OverlayFS is a modern *union filesystem* that is similar to AUFS, but faster and with a simpler implementation. Docker provides two storage drivers for OverlayFS: the original `overlay`, and the newer and more stable `overlay2`.

This topic refers to the Linux kernel driver as `OverlayFS` and to the Docker storage driver as `overlay` or `overlay2`.

> **Note**: If you use OverlayFS, use the `overlay2` driver rather than the `overlay` driver, because it is more efficient in terms of inode utilization. To use the new driver, you need version 4.0 or higher of the Linux kernel, or RHEL or CentOS using version 3.10.0-514 and above.
>
> For more information about differences between `overlay` vs `overlay2`, check [Docker storage drivers](https://docs.docker.com/storage/storagedriver/select-storage-driver/).

## Prerequisites

OverlayFS is supported if you meet the following prerequisites:

- The `overlay2` driver is supported on Docker CE, and Docker EE 17.06.02-ee5 and up, and is the recommended storage driver.

- Version 4.0 or higher of the Linux kernel, or RHEL or CentOS using version 3.10.0-514 of the kernel or higher. If you use an older kernel, you need to use the `overlay` driver, which is not recommended.

- The `overlay` and `overlay2` drivers are supported on `xfs` backing filesystems, but only with `d_type=true` enabled.

  Use `xfs_info` to verify that the `ftype` option is set to `1`. To format an   `xfs` filesystem correctly, use the flag `-n ftype=1`.

  > **Warning**: Running on XFS without d_type support now causes Docker to skip the attempt to use the `overlay` or `overlay2` driver. Existing installs will continue to run, but produce an error. This is to allow users to migrate their data. In a future version, this will be a fatal error, which will prevent Docker from starting.

- Changing the storage driver makes existing containers and images inaccessible on the local system. Use `docker save` to save any images you have built or push them to Docker Hub or a private registry before changing the storage driver, so that you do not need to re-create them later.

## Configure Docker with the `overlay` or `overlay2` storage driver

It is highly recommended that you use the `overlay2` driver if possible, rather than the `overlay` driver. The `overlay` driver is **not** supported for Docker EE.

To configure Docker to use the `overlay` storage driver your Docker host must be running version 3.18 of the Linux kernel (preferably newer) with the overlay kernel module loaded. For the `overlay2` driver, the version of your kernel must be 4.0 or newer.

Before following this procedure, you must first meet all the [prerequisites](https://docs.docker.com/storage/storagedriver/overlayfs-driver/#prerequisites).

The steps below outline how to configure the `overlay2` storage driver. If you need to use the legacy `overlay` driver, specify it instead.

1. Stop Docker.

   ```
   $ sudo systemctl stop docker
   ```

2. Copy the contents of `/var/lib/docker` to a temporary location.

   ```
   $ cp -au /var/lib/docker /var/lib/docker.bk
   ```

3. If you want to use a separate backing filesystem from the one used by `/var/lib/`, format the filesystem and mount it into `/var/lib/docker`. Make sure add this mount to `/etc/fstab` to make it permanent.

4. Edit `/etc/docker/daemon.json`. If it does not yet exist, create it. Assuming that the file was empty, add the following contents.

   ```
   {
     "storage-driver": "overlay2"
   }
   ```

   Docker does not start if the `daemon.json` file contains badly-formed JSON.

5. Start Docker.

   ```
   $ sudo systemctl start docker
   ```

6. Verify that the daemon is using the `overlay2` storage driver. Use the `docker info` command and look for `Storage Driver` and `Backing filesystem`.

   ```
   $ docker info
   
   Containers: 0
   Images: 0
   Storage Driver: overlay2
    Backing Filesystem: xfs
    Supports d_type: true
    Native Overlay Diff: true
   <output truncated>
   ```

Docker is now using the `overlay2` storage driver and has automatically created the overlay mount with the required `lowerdir`, `upperdir`, `merged`, and `workdir` constructs.

Continue reading for details about how OverlayFS works within your Docker containers, as well as performance advice and information about limitations of its compatibility with different backing filesystems.

## How the `overlay2` driver works

If you are still using the `overlay` driver rather than `overlay2`, see [How the overlay driver works](https://docs.docker.com/storage/storagedriver/overlayfs-driver/#how-the-overlay-driver-works) instead.

OverlayFS layers two directories on a single Linux host and presents them as a single directory. These directories are called *layers* and the unification process is referred to as a *union mount*. OverlayFS refers to the lower directory as `lowerdir` and the upper directory a `upperdir`. The unified view is exposed through its own directory called `merged`.

The `overlay2` driver natively supports up to 128 lower OverlayFS layers. This capability provides better performance for layer-related Docker commands such as `docker build` and `docker commit`, and consumes fewer inodes on the backing filesystem.

### Image and container layers on-disk

After downloading a five-layer image using `docker pull ubuntu`, you can see six directories under `/var/lib/docker/overlay2`.

> **Warning**: Do not directly manipulate any files or directories within `/var/lib/docker/`. These files and directories are managed by Docker.

```
$ ls -l /var/lib/docker/overlay2

total 24
drwx------ 5 root root 4096 Jun 20 07:36 223c2864175491657d238e2664251df13b63adb8d050924fd1bfcdb278b866f7
drwx------ 3 root root 4096 Jun 20 07:36 3a36935c9df35472229c57f4a27105a136f5e4dbef0f87905b2e506e494e348b
drwx------ 5 root root 4096 Jun 20 07:36 4e9fa83caff3e8f4cc83693fa407a4a9fac9573deaf481506c102d484dd1e6a1
drwx------ 5 root root 4096 Jun 20 07:36 e8876a226237217ec61c4baf238a32992291d059fdac95ed6303bdff3f59cff5
drwx------ 5 root root 4096 Jun 20 07:36 eca1e4e1694283e001f200a667bb3cb40853cf2d1b12c29feda7422fed78afed
drwx------ 2 root root 4096 Jun 20 07:36 l
```

The new `l` (lowercase `L`) directory contains shortened layer identifiers as symbolic links. These identifiers are used to avoid hitting the page size limitation on arguments to the `mount` command.

```
$ ls -l /var/lib/docker/overlay2/l

total 20
lrwxrwxrwx 1 root root 72 Jun 20 07:36 6Y5IM2XC7TSNIJZZFLJCS6I4I4 -> ../3a36935c9df35472229c57f4a27105a136f5e4dbef0f87905b2e506e494e348b/diff
lrwxrwxrwx 1 root root 72 Jun 20 07:36 B3WWEFKBG3PLLV737KZFIASSW7 -> ../4e9fa83caff3e8f4cc83693fa407a4a9fac9573deaf481506c102d484dd1e6a1/diff
lrwxrwxrwx 1 root root 72 Jun 20 07:36 JEYMODZYFCZFYSDABYXD5MF6YO -> ../eca1e4e1694283e001f200a667bb3cb40853cf2d1b12c29feda7422fed78afed/diff
lrwxrwxrwx 1 root root 72 Jun 20 07:36 NFYKDW6APBCCUCTOUSYDH4DXAT -> ../223c2864175491657d238e2664251df13b63adb8d050924fd1bfcdb278b866f7/diff
lrwxrwxrwx 1 root root 72 Jun 20 07:36 UL2MW33MSE3Q5VYIKBRN4ZAGQP -> ../e8876a226237217ec61c4baf238a32992291d059fdac95ed6303bdff3f59cff5/diff
```

The lowest layer contains a file called `link`, which contains the name of the shortened identifier, and a directory called `diff` which contains the layer’s contents.

```
$ ls /var/lib/docker/overlay2/3a36935c9df35472229c57f4a27105a136f5e4dbef0f87905b2e506e494e348b/

diff  link

$ cat /var/lib/docker/overlay2/3a36935c9df35472229c57f4a27105a136f5e4dbef0f87905b2e506e494e348b/link

6Y5IM2XC7TSNIJZZFLJCS6I4I4

$ ls  /var/lib/docker/overlay2/3a36935c9df35472229c57f4a27105a136f5e4dbef0f87905b2e506e494e348b/diff

bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```

The second-lowest layer, and each higher layer, contain a file called `lower`, which denotes its parent, and a directory called `diff` which contains its contents. It also contains a `merged` directory, which contains the unified contents of its parent layer and itself, and a `work` directory which is used internally by OverlayFS.

```
$ ls /var/lib/docker/overlay2/223c2864175491657d238e2664251df13b63adb8d050924fd1bfcdb278b866f7

diff  link  lower  merged  work

$ cat /var/lib/docker/overlay2/223c2864175491657d238e2664251df13b63adb8d050924fd1bfcdb278b866f7/lower

l/6Y5IM2XC7TSNIJZZFLJCS6I4I4

$ ls /var/lib/docker/overlay2/223c2864175491657d238e2664251df13b63adb8d050924fd1bfcdb278b866f7/diff/

etc  sbin  usr  var
```

To view the mounts which exist when you use the `overlay` storage driver with Docker, use the `mount` command. The output below is truncated for readability.

```
$ mount | grep overlay

overlay on /var/lib/docker/overlay2/9186877cdf386d0a3b016149cf30c208f326dca307529e646afce5b3f83f5304/merged
type overlay (rw,relatime,
lowerdir=l/DJA75GUWHWG7EWICFYX54FIOVT:l/B3WWEFKBG3PLLV737KZFIASSW7:l/JEYMODZYFCZFYSDABYXD5MF6YO:l/UL2MW33MSE3Q5VYIKBRN4ZAGQP:l/NFYKDW6APBCCUCTOUSYDH4DXAT:l/6Y5IM2XC7TSNIJZZFLJCS6I4I4,
upperdir=9186877cdf386d0a3b016149cf30c208f326dca307529e646afce5b3f83f5304/diff,
workdir=9186877cdf386d0a3b016149cf30c208f326dca307529e646afce5b3f83f5304/work)
```

The `rw` on the second line shows that the `overlay` mount is read-write.

## How the `overlay` driver works

This content applies to the `overlay` driver only. Docker recommends using the `overlay2` driver, which works differently. See [How the overlay2 driver works](https://docs.docker.com/storage/storagedriver/overlayfs-driver/#how-the-overlay2-driver-works) for `overlay2`.

OverlayFS layers two directories on a single Linux host and presents them as a single directory. These directories are called *layers* and the unification process is referred to as a *union mount*. OverlayFS refers to the lower directory as `lowerdir` and the upper directory a `upperdir`. The unified view is exposed through its own directory called `merged`.

The diagram below shows how a Docker image and a Docker container are layered. The image layer is the `lowerdir` and the container layer is the `upperdir`. The unified view is exposed through a directory called `merged` which is effectively the containers mount point. The diagram shows how Docker constructs map to OverlayFS constructs.

![overlayfs lowerdir, upperdir, merged](https://docs.docker.com/storage/storagedriver/images/overlay_constructs.jpg)

Where the image layer and the container layer contain the same files, the container layer “wins” and obscures the existence of the same files in the image layer.

The `overlay` driver only works with two layers. This means that multi-layered images cannot be implemented as multiple OverlayFS layers. Instead, each image layer is implemented as its own directory under `/var/lib/docker/overlay`.  Hard links are then used as a space-efficient way to reference data shared with lower layers. The use of hardlinks causes an excessive use of inodes, which is a known limitation of the legacy `overlay` storage driver, and may require additional configuration of the backing filesystem. Refer to the [overlayFS and Docker performance](https://docs.docker.com/storage/storagedriver/overlayfs-driver/#overlayfs-and-docker-performance) for details.

To create a container, the `overlay` driver combines the directory representing the image’s top layer plus a new directory for the container. The image’s top layer is the `lowerdir` in the overlay and is read-only. The new directory for the container is the `upperdir` and is writable.

### Image and container layers on-disk

The following `docker pull` command shows a Docker host downloading a Docker image comprising five layers.

```
$ docker pull ubuntu

Using default tag: latest
latest: Pulling from library/ubuntu

5ba4f30e5bea: Pull complete
9d7d19c9dc56: Pull complete
ac6ad7efd0f9: Pull complete
e7491a747824: Pull complete
a3ed95caeb02: Pull complete
Digest: sha256:46fb5d001b88ad904c5c732b086b596b92cfb4a4840a3abd0e35dbb6870585e4
Status: Downloaded newer image for ubuntu:latest
```

#### The image layers

Each image layer has its own directory within `/var/lib/docker/overlay/`, which contains its contents, as shown below. The image layer IDs do not correspond to the directory IDs.

> **Warning**: Do not directly manipulate any files or directories within `/var/lib/docker/`. These files and directories are managed by Docker.

```
$ ls -l /var/lib/docker/overlay/

total 20
drwx------ 3 root root 4096 Jun 20 16:11 38f3ed2eac129654acef11c32670b534670c3a06e483fce313d72e3e0a15baa8
drwx------ 3 root root 4096 Jun 20 16:11 55f1e14c361b90570df46371b20ce6d480c434981cbda5fd68c6ff61aa0a5358
drwx------ 3 root root 4096 Jun 20 16:11 824c8a961a4f5e8fe4f4243dab57c5be798e7fd195f6d88ab06aea92ba931654
drwx------ 3 root root 4096 Jun 20 16:11 ad0fe55125ebf599da124da175174a4b8c1878afe6907bf7c78570341f308461
drwx------ 3 root root 4096 Jun 20 16:11 edab9b5e5bf73f2997524eebeac1de4cf9c8b904fa8ad3ec43b3504196aa3801
```

The image layer directories contain the files unique to that layer as well as hard links to the data that is shared with lower layers. This allows for efficient use of disk space.

```
$ ls -i /var/lib/docker/overlay/38f3ed2eac129654acef11c32670b534670c3a06e483fce313d72e3e0a15baa8/root/bin/ls

19793696 /var/lib/docker/overlay/38f3ed2eac129654acef11c32670b534670c3a06e483fce313d72e3e0a15baa8/root/bin/ls

$ ls -i /var/lib/docker/overlay/55f1e14c361b90570df46371b20ce6d480c434981cbda5fd68c6ff61aa0a5358/root/bin/ls

19793696 /var/lib/docker/overlay/55f1e14c361b90570df46371b20ce6d480c434981cbda5fd68c6ff61aa0a5358/root/bin/ls
```

#### The container layer

Containers also exist on-disk in the Docker host’s filesystem under `/var/lib/docker/overlay/`. If you list a running container’s subdirectory using the `ls -l` command, three directories and one file exist:

```
$ ls -l /var/lib/docker/overlay/<directory-of-running-container>

total 16
-rw-r--r-- 1 root root   64 Jun 20 16:39 lower-id
drwxr-xr-x 1 root root 4096 Jun 20 16:39 merged
drwxr-xr-x 4 root root 4096 Jun 20 16:39 upper
drwx------ 3 root root 4096 Jun 20 16:39 work
```

The `lower-id` file contains the ID of the top layer of the image the container is based on, which is the OverlayFS `lowerdir`.

```
$ cat /var/lib/docker/overlay/ec444863a55a9f1ca2df72223d459c5d940a721b2288ff86a3f27be28b53be6c/lower-id

55f1e14c361b90570df46371b20ce6d480c434981cbda5fd68c6ff61aa0a5358
```

The `upper` directory contains the contents of the container’s read-write layer, which corresponds to the OverlayFS `upperdir`.

The `merged` directory is the union mount of the `lowerdir` and `upperdir`, which comprises the view of the filesystem from within the running container.

The `work` directory is internal to OverlayFS.

To view the mounts which exist when you use the `overlay` storage driver with Docker, use the `mount` command. The output below is truncated for readability.

```
$ mount | grep overlay

overlay on /var/lib/docker/overlay/ec444863a55a.../merged
type overlay (rw,relatime,lowerdir=/var/lib/docker/overlay/55f1e14c361b.../root,
upperdir=/var/lib/docker/overlay/ec444863a55a.../upper,
workdir=/var/lib/docker/overlay/ec444863a55a.../work)
```

The `rw` on the second line shows that the `overlay` mount is read-write.

## How container reads and writes work with `overlay` or `overlay2`

### Reading files

Consider three scenarios where a container opens a file for read access with overlay.

- **The file does not exist in the container layer**: If a container opens a file for read access and the file does not already exist in the container (`upperdir`) it is read from the image (`lowerdir)`. This incurs very little performance overhead.
- **The file only exists in the container layer**: If a container opens a file for read access and the file exists in the container (`upperdir`) and not in the image (`lowerdir`), it is read directly from the container.
- **The file exists in both the container layer and the image layer**: If a container opens a file for read access and the file exists in the image layer and the container layer, the file’s version in the container layer is read. Files in the container layer (`upperdir`) obscure files with the same name in the image layer (`lowerdir`).

### Modifying files or directories

Consider some scenarios where files in a container are modified.

- **Writing to a file for the first time**: The first time a container writes to an existing file, that file does not exist in the container (`upperdir`). The `overlay`/`overlay2` driver performs a *copy_up* operation to copy the file from the image (`lowerdir`) to the container (`upperdir`). The container then writes the changes to the new copy of the file in the container layer.

  However, OverlayFS works at the file level rather than the block level. This means that all OverlayFS copy_up operations copy the entire file, even if the file is very large and only a small part of it is being modified. This can have a noticeable impact on container write performance. However, two things are worth noting:

  - The copy_up operation only occurs the first time a given file is written to. Subsequent writes to the same file operate against the copy of the file already copied up to the container.
  - OverlayFS only works with two layers. This means that performance should be better than AUFS, which can suffer noticeable latencies when searching for files in images with many layers. This advantage applies to both `overlay` and `overlay2` drivers. `overlayfs2` is slightly less performant than `overlayfs` on initial read, because it must look through more layers, but it caches the results so this is only a small penalty.

- **Deleting files and directories**:

  - When a *file* is deleted within a container, a *whiteout* file is created in the container (`upperdir`). The version of the file in the image layer (`lowerdir`) is not deleted (because the `lowerdir` is read-only). However, the whiteout file prevents it from being available to the container.
  - When a *directory* is deleted within a container, an *opaque directory* is created within the container (`upperdir`). This works in the same way as a whiteout file and effectively prevents the directory from being accessed, even though it still exists in the image (`lowerdir`).

- **Renaming directories**: Calling `rename(2)` for a directory is allowed only when both the source and the destination path are on the top layer. Otherwise, it returns `EXDEV` error (“cross-device link not permitted”). Your application needs to be designed to handle `EXDEV` and fall back to a “copy and unlink” strategy.

## OverlayFS and Docker Performance

Both `overlay2` and `overlay` drivers are more performant than `aufs` and `devicemapper`. In certain circumstances, `overlay2` may perform better than `btrfs` as well. However, be aware of the following details.

- **Page Caching**. OverlayFS supports page cache sharing. Multiple containers accessing the same file share a single page cache entry for that file. This makes the `overlay` and `overlay2` drivers efficient with memory and a good option for high-density use cases such as PaaS.

- **copy_up**. As with AUFS, OverlayFS performs copy-up operations whenever a container writes to a file for the first time. This can add latency into the write operation, especially for large files. However, once the file has been copied up, all subsequent writes to that file occur in the upper layer, without the need for further copy-up operations.

  The OverlayFS `copy_up` operation is faster than the same operation with AUFS, because AUFS supports more layers than OverlayFS and it is possible to incur far larger latencies if searching through many AUFS layers. `overlay2` supports multiple layers as well, but mitigates any performance hit with caching.

- **Inode limits**. Use of the legacy `overlay` storage driver can cause excessive inode consumption. This is especially true in the presence of a large number of images and containers on the Docker host. The only way to increase the number of inodes available to a filesystem is to reformat it. To avoid running into this issue, it is highly recommended that you use `overlay2` if at all possible.

### Performance best practices

The following generic performance best practices also apply to OverlayFS.

- **Use fast storage**: Solid-state drives (SSDs) provide faster reads and writes than spinning disks.
- **Use volumes for write-heavy workloads**: Volumes provide the best and most predictable performance for write-heavy workloads. This is because they bypass the storage driver and do not incur any of the potential overheads introduced by thin provisioning and copy-on-write. Volumes have other benefits, such as allowing you to share data among containers and persisting your data even if no running container is using them.

## Limitations on OverlayFS compatibility

To summarize the OverlayFS’s aspect which is incompatible with other filesystems:

- **open(2)**: OverlayFS only implements a subset of the POSIX standards. This can result in certain OverlayFS operations breaking POSIX standards. One such operation is the *copy-up* operation. Suppose that  your application calls `fd1=open("foo", O_RDONLY)` and then `fd2=open("foo", O_RDWR)`. In this case, your application expects `fd1` and `fd2` to refer to the same file. However, due to a copy-up operation that occurs after the second calling to `open(2)`, the descriptors refer to different files. The `fd1` continues to reference the file in the image (`lowerdir`) and the `fd2` references the file in the container (`upperdir`). A workaround for this is to `touch` the files which causes the copy-up operation to happen. All subsequent `open(2)` operations regardless of read-only or read-write access mode reference the file in the container (`upperdir`).

  `yum` is known to be affected unless the `yum-plugin-ovl` package is installed. If the `yum-plugin-ovl` package is not available in your distribution such as RHEL/CentOS prior to 6.8 or 7.2, you may need to run `touch /var/lib/rpm/*` before running `yum install`. This package implements the `touch` workaround referenced above for `yum`.

- **rename(2)**: OverlayFS does not fully support the `rename(2)` system call. Your application needs to detect its failure and fall back to a “copy and unlink” strategy.