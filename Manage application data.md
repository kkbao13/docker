# Manage application data

# Manage data in Docker

By default all files created inside a container are stored on a writable container layer. This means that:

- The data doesn’t persist when that container no longer exists, and it can be difficult to get the data out of the container if another process needs it.
- A container’s writable layer is tightly coupled to the host machine where the container is running. You can’t easily move the data somewhere else.
- Writing into a container’s writable layer requires a [storage driver](https://docs.docker.com/storage/storagedriver/) to manage the filesystem. The storage driver provides a union filesystem, using the Linux kernel. This extra abstraction reduces performance as compared to using *data volumes*, which write directly to the host filesystem.

Docker has two options for containers to store files in the host machine, so that the files are persisted even after the container stops: *volumes*, and *bind mounts*. If you’re running Docker on Linux you can also use a *tmpfs mount*.

Keep reading for more information about these two ways of persisting data.

默认情况下，在容器内创建的所有文件都存储在**可写容器层**中。这意味着：

- 当该容器不再存在时，数据不会持久存在，并且如果另一个进程需要数据，则可能很难从容器中获取数据。
- 容器的可写层紧密耦合到运行容器的主机。您无法轻松地将数据移动到其他位置。
- 写入容器的可写层需要存储驱动程序来管理文件系统。**storage driver**程序使用Linux内核提供联合文件系统。与使用直接写入主机文件系统的数据卷相比，这种额外的抽象降低了性能。

Docker有两个容器选项可以在主机中存储文件，因此即使在容器停止之后文件仍然存在：***volumes*, and*bind mounts***。如果您在Linux上运行Docker，您还可以使用**tmpfs mount**.Keep读取以获取有关这两种持久化数据方式的更多信息。

## Choose the right type of mount

No matter which type of mount you choose to use, the data looks the same from within the container. It is exposed as either a directory or an individual file in the container’s filesystem.

An easy way to visualize the difference among volumes, bind mounts, and `tmpfs` mounts is to think about where the data lives on the Docker host.

无论您选择使用哪种类型的安装，数据在容器内看起来都是相同的。它作为目录或容器文件系统中的单个文件公开。

 **volumes, bind mounts, and `tmpfs`mounts**之间差异的简单方法是考虑数据在Docker主机上的位置。

![types of mounts and where they live on the Docker host](https://docs.docker.com/storage/images/types-of-mounts.png)

- **Volumes** are stored in a part of the host filesystem which is *managed by Docker* (`/var/lib/docker/volumes/` on Linux). Non-Docker processes should not modify this part of the filesystem. Volumes are the best way to persist data in Docker.
- **Volumes** 存储在由Docker管理的主机文件系统的一部分中（Linux上的`/var/lib/docker/volumes/`）。**非Docker进程**不应修改文件系统的这一部分。卷是在Docker中保留数据的**最佳方式**。
- **Bind mounts** may be stored *anywhere* on the host system. They may even be important system files or directories. Non-Docker processes on the Docker host or a Docker container can modify them at any time.
- 安装可以存储在主机系统的任何地方。它们甚至可能是重要的系统文件或目录。 Docker主机或Docker容器上的非Docker进程可以随时修改它们
- **tmpfs mounts** are stored in the host system’s memory only, and are never written to the host system’s filesystem.
- 仅存储在主机系统的内存中，永远不会写入主机系统的文件系统。

### More details about mount types

- **Volumes**: Created and managed by Docker. You can create a volume explicitly using the `docker volume create` command, or Docker can create a volume during container or service creation.

  When you create a volume, it is stored within a directory on the Docker host. When you mount the volume into a container, this directory is what is mounted into the container. This is similar to the way that bind mounts work, except that volumes are managed by Docker and are isolated from the core functionality of the host machine.

  A given volume can be mounted into multiple containers simultaneously. When no running container is using a volume, the volume is still available to Docker and is not removed automatically. You can remove unused volumes using `docker volume prune`.

  When you mount a volume, it may be **named** or **anonymous**. Anonymous volumes are not given an explicit name when they are first mounted into a container, so Docker gives them a random name that is guaranteed to be unique within a given Docker host. Besides the name, named and anonymous volumes behave in the same ways.

  Volumes also support the use of *volume drivers*, which allow you to store your data on remote hosts or cloud providers, among other possibilities.

  创建，绑定，多绑定和删除，命名，远程存储

  由Docker创建和管理。您可以使用docker volume create命令显式创建卷，或者Docker可以在容器或服务创建期间创建卷。

  创建卷时，它存储在Docker主机上的目录中。将卷装入容器时，此目录是装入容器的目录。这类似于绑定挂载的工作方式，除了卷由Docker管理并与主机的核心功能隔离。

  **给定的体积可以同时安装到多个容器中**。当没有正在运行的容器正在使用卷时，该卷仍可供Docker使用，并且不会自动删除。您可以使用docker volume prune删除未使用的卷。

  装入卷时，可能会命名或匿名。匿名卷在首次装入容器时未给出明确的名称，因此Docker为它们提供了一个随机名称，该名称在给定的Docker主机中保证是唯一的。除名称外，命名和匿名卷的行为方式相同。

  卷还支持使用**volume drivers**，这些驱动程序允许您将数据存储在**远程主机**或云提供程序上，以及其他可能性。

- **Bind mounts**: Available since the early days of Docker. Bind mounts have limited functionality compared to volumes. When you use a bind mount, a file or directory on the *host machine* is mounted into a container. The file or directory is referenced by its full path on the host machine. The file or directory does not need to exist on the Docker host already. It is created on demand if it does not yet exist. Bind mounts are very performant, but they rely on the host machine’s filesystem having a specific directory structure available. If you are developing new Docker applications, consider using named volumes instead. You can’t use Docker CLI commands to directly manage bind mounts.

  > Bind mounts allow access to sensitive files
  >
  > One side effect of using bind mounts, for better or for worse, is that you can change the **host** filesystem via processes running in a **container**, including creating, modifying, or deleting important system files or directories. This is a powerful ability which can have security implications, including impacting non-Docker processes on the host system.

- **tmpfs mounts**: A `tmpfs` mount is not persisted on disk, either on the Docker host or within a container. It can be used by a container during the lifetime of the container, to store non-persistent state or sensitive information. For instance, internally, swarm services use `tmpfs` mounts to mount [secrets](https://docs.docker.com/engine/swarm/secrets/) into a service’s containers.

Bind mounts and volumes can both be mounted into containers using the `-v` or `--volume` flag, but the syntax for each is slightly different. For `tmpfs` mounts, you can use the `--tmpfs` flag. However, in Docker 17.06 and higher, we recommend using the `--mount` flag for both containers and services, for bind mounts, volumes, or `tmpfs` mounts, as the syntax is more clear.

## Good use cases for volumes

Volumes are the **preferred way** to persist data in Docker containers and services. Some use cases for volumes include:

- Sharing data among multiple running containers. If you don’t explicitly create it, a volume is created the first time it is mounted into a container. When that container stops or is removed, the volume still exists. Multiple containers can mount the same volume simultaneously, either read-write or read-only. Volumes are only removed when you explicitly remove them.
- When the Docker host is not guaranteed to have a given directory or file structure. Volumes help you decouple the configuration of the Docker host from the container runtime.
- When you want to store your container’s data on a remote host or a cloud provider, rather than locally.
- When you need to back up, restore, or migrate data from one Docker host to another, volumes are a better choice. You can stop containers using the volume, then back up the volume’s directory (such as `/var/lib/docker/volumes/<volume-name>`).
- 在多个运行容器之间共享数据。如果未显式创建它，则会在第一次将其装入容器时创建卷。当该容器停止或被移除时，该卷仍然存在。多个容器可以同时安装相同的卷，可以是读写也可以是只读。仅在您明确删除卷时才会删除卷。
- 当Docker主机不能保证具有给定的目录或文件结构时。卷可帮助您将Docker主机的配置与容器运行时分离。
- 如果要将容器的数据存储在远程主机或云提供程序上，而不是本地存储。
- 当您需要将数据从一个Docker主机备份，还原或迁移到另一个Docker主机时，卷是更好的选择。您可以使用卷停止容器，然后备份卷的目录（例如/ var / lib / docker / volumes / <volume-name>）。

## Good use cases for bind mounts

In general, you should use volumes where possible. Bind mounts are appropriate for the following types of use case:

- Sharing configuration files from the host machine to containers. This is how Docker provides DNS resolution to containers by default, by mounting `/etc/resolv.conf` from the host machine into each container.

- Sharing source code or build artifacts between a development environment on the Docker host and a container. For instance, you may mount a Maven `target/` directory into a container, and each time you build the Maven project on the Docker host, the container gets access to the rebuilt artifacts.

  If you use Docker for development this way, your production Dockerfile would copy the production-ready artifacts directly into the image, rather than relying on a bind mount.

- When the file or directory structure of the Docker host is guaranteed to be consistent with the bind mounts the containers require.

## Good use cases for tmpfs mounts

`tmpfs` mounts are best used for cases when you do not want the data to persist either on the host machine or within the container. This may be for security reasons or to protect the performance of the container when your application needs to write a large volume of non-persistent state data.

## Tips for using bind mounts or volumes

If you use either bind mounts or volumes, keep the following in mind:

- If you mount an **empty volume** into a directory in the container in which files or directories exist, these files or directories are propagated (copied) into the volume. Similarly, if you start a container and specify a volume which does not already exist, an empty volume is created for you. This is a good way to pre-populate data that another container needs.
- If you mount a **bind mount or non-empty volume** into a directory in the container in which some files or directories exist, these files or directories are obscured by the mount, just as if you saved files into `/mnt` on a Linux host and then mounted a USB drive into `/mnt`. The contents of `/mnt` would be obscured by the contents of the USB drive until the USB drive were unmounted. The obscured files are not removed or altered, but are not accessible while the bind mount or volume is mounted.
- 如果将**empty volume**装入容器中存在文件或目录的目录中，则会将这些文件或目录传播（复制）到卷中。同样，如果启动容器并指定尚不存在的卷，则会为您创建一个空卷。这是预先填充另一个容器所需数据的好方法。
- 如果将绑定装载或**非空卷**装入容器中存在某些文件或目录的目录中，则挂载**会掩盖**这些文件或目录，就像在Linux主机上将文件保存到/mnt中一样将USB驱动器安装到/mnt中。在卸载USB驱动器之前，/mnt的内容将被USB驱动器的内容遮挡。隐藏的文件不会被删除或更改，但在安装绑定装载或卷时无法访问。

# Use volumes

Volumes are the preferred mechanism for persisting data generated by and used by Docker containers. While [bind mounts](https://docs.docker.com/storage/bind-mounts/) are dependent on the directory structure of the host machine, volumes are completely managed by Docker. Volumes have several advantages over bind mounts:

**Volumes**是保存Docker容器生成和使用的数据的首选机制。虽然绑定挂载依赖于主机的目录结构，但卷完全由Docker管理。卷绑定安装有几个**优点**：

- Volumes are easier to **back up or migrate** than bind mounts.卷可以比绑定挂载更容易备份或迁移
- You can manage volumes using Docker **CLI** commands or the Docker **API**.可以使用Docker CLI命令或Docker API管理卷 
- Volumes work on both Linux and Windows containers.可以在Linux和Windows容器上运行
- Volumes can be more safely shared among multiple containers.可以更安全地在多个容器之间共享卷
- Volume drivers let you store volumes on remote hosts or cloud providers, to encrypt the contents of volumes, or to add other functionality.使用卷驱动程序可以存储卷在远程主机或云提供程序上，加密卷的内容，或添加其他功能。
- New volumes can have their content pre-populated by a container.新卷可以通过容器预先填充其内容。

In addition, volumes are often a better choice than persisting data in a container’s writable layer, because a volume does not increase the size of the containers using it, and the volume’s contents exist outside the lifecycle of a given container.

![volumes on the Docker host](https://docs.docker.com/storage/images/types-of-mounts-volume.png)

If your container generates non-persistent state data, consider using a [tmpfs mount](https://docs.docker.com/storage/tmpfs/) to avoid storing the data anywhere permanently, and to increase the container’s performance by avoiding writing into the container’s writable layer.

Volumes use `rprivate` bind propagation, and bind propagation is not configurable for volumes.

此外，卷通常是比容器的可写层中的持久数据更好的选择，因为卷不会增加使用它的容器的大小，并且卷的内容存在于给定容器的生命周期之外。

卷使用rprivate绑定传播，并且卷不可配置绑定传播。

## Choose the -v or --mount flag

Originally, the `-v` or `--volume` flag was used for standalone containers and the `--mount` flag was used for swarm services. However, starting with Docker 17.06, you can also use `--mount` with standalone containers. In general, `--mount` is more explicit and verbose. The biggest difference is that the `-v` syntax combines all the options together in one field, while the `--mount` syntax separates them. Here is a comparison of the syntax for each flag.

> New users should try `--mount` syntax which is simpler than `--volume` syntax.

If you need to specify volume driver options, you must use `--mount`.

- `-v` or `--volume`

  : Consists of three fields, separated by colon characters (

  ```
  :
  ```

  ). The fields must be in the correct order, and the meaning of each field is not immediately obvious.     

  - In the case of named volumes, the first field is the name of the volume, and is unique on a given host machine. For anonymous volumes, the first field is omitted.
  - The second field is the path where the file or directory are mounted in the container.
  - The third field is optional, and is a comma-separated list of options, such as `ro`. These options are discussed below.

- `--mount`

  : Consists of multiple key-value pairs, separated by commas and each consisting of a 

  ```
  <key>=<value>
  ```

   tuple. The 

  ```
  --mount
  ```

   syntax is more verbose than 

  ```
  -v
  ```

   or 

  ```
  --volume
  ```

  , but the order of the keys is not significant, and the value of the flag is easier to understand.     

  - The `type` of the mount, which can be [`bind`](https://docs.docker.com/storage/bind-mounts/), `volume`, or [`tmpfs`](https://docs.docker.com/storage/tmpfs/). This topic discusses volumes, so the type is always `volume`.
  - The `source` of the mount. For named volumes, this is the name of the volume. For anonymous volumes, this field is omitted. May be specified as `source` or `src`.
  - The `destination` takes as its value the path where the file or directory is mounted in the container. May be specified as `destination`, `dst`, or `target`.
  - The `readonly` option, if present, causes the bind mount to be [mounted into the container as read-only](https://docs.docker.com/storage/volumes/#use-a-read-only-volume).
  - The `volume-opt` option, which can be specified more than once, takes a key-value pair consisting of the option name and its value.

> Escape values from outer CSV parser
>
> If your volume driver accepts a comma-separated list as an option, you must escape the value from the outer CSV parser. To escape a `volume-opt`, surround it with double quotes (`"`) and surround the entire mount parameter with single quotes (`'`).
>
> For example, the `local` driver accepts mount options as a comma-separated list in the `o` parameter. This example shows the correct way to escape the list.
>
> ```
> $ docker service create \
>      --mount 'type=volume,src=<VOLUME-NAME>,dst=<CONTAINER-PATH>,volume-driver=local,volume-opt=type=nfs,volume-opt=device=<nfs-server>:<nfs-path>,"volume-opt=o=addr=<nfs-address>,vers=4,soft,timeo=180,bg,tcp,rw"'
>     --name myservice \
>     <IMAGE>
> ```

The examples below show both the `--mount` and `-v` syntax where possible, and     `--mount` is presented first.

### Differences between `-v` and `--mount` behavior

As opposed to bind mounts, all options for volumes are available for both `--mount` and `-v` flags.

When using volumes with services, only `--mount` is supported.

## Create and manage volumes

Unlike a bind mount, you can create and manage volumes outside the scope of any container.

**Create a volume**:

```
$ docker volume create my-vol
```

**List volumes**:

```
$ docker volume ls

local               my-vol
```

**Inspect a volume**:

```
$ docker volume inspect my-vol
[
    {
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/my-vol/_data",
        "Name": "my-vol",
        "Options": {},
        "Scope": "local"
    }
]
```

**Remove a volume**:

```
$ docker volume rm my-vol
```

## Start a container with a volume

If you start a container with a volume that does not yet exist, Docker creates the volume for you. The following example mounts the volume `myvol2` into `/app/` in the container.

The `-v` and `--mount` examples below produce the same result. You can’t run them both unless you remove the `devtest` container and the `myvol2` volume after running the first one.

- `--mount`
- `-v`

```
$ docker run -d \
  --name devtest \
  --mount source=myvol2,target=/app \
  nginx:latest
```

Use `docker inspect devtest` to verify that the volume was created and mounted correctly. Look for the `Mounts` section:

```
"Mounts": [
    {
        "Type": "volume",
        "Name": "myvol2",
        "Source": "/var/lib/docker/volumes/myvol2/_data",
        "Destination": "/app",
        "Driver": "local",
        "Mode": "",
        "RW": true,
        "Propagation": ""
    }
],
```

This shows that the mount is a volume, it shows the correct source and destination, and that the mount is read-write.

Stop the container and remove the volume. Note volume removal is a separate step.

```
$ docker container stop devtest

$ docker container rm devtest

$ docker volume rm myvol2
```

### Start a service with volumes

When you start a service and define a volume, each service container uses its own local volume. None of the containers can share this data if you use the `local` volume driver, but some volume drivers do support shared storage. Docker for AWS and Docker for Azure both support persistent storage using the Cloudstor plugin.

The following example starts a `nginx` service with four replicas, each of which uses a local volume called `myvol2`.

```
$ docker service create -d \
  --replicas=4 \
  --name devtest-service \
  --mount source=myvol2,target=/app \
  nginx:latest
```

Use `docker service ps devtest-service` to verify that the service is running:

```
$ docker service ps devtest-service

ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
4d7oz1j85wwn        devtest-service.1   nginx:latest        moby                Running             Running 14 seconds ago
```

Remove the service, which stops all its tasks:

```
$ docker service rm devtest-service
```

Removing the service does not remove any volumes created by the service. Volume removal is a separate step.

#### Syntax differences for services

The `docker service create` command does not support the `-v` or `--volume` flag. When mounting a volume into a service’s containers, you must use the `--mount` flag.

### Populate a volume using a container

If you start a container which creates a new volume, as above, and the container has files or directories in the directory to be mounted (such as `/app/` above), the directory’s contents are copied into the volume. The container then mounts and uses the volume, and other containers which use the volume also have access to the pre-populated content.

To illustrate this, this example starts an `nginx` container and populates the new volume `nginx-vol` with the contents of the container’s `/usr/share/nginx/html` directory, which is where Nginx stores its default HTML content.

The `--mount` and `-v` examples have the same end result.

- `--mount`
- `-v`

```
$ docker run -d \
  --name=nginxtest \
  --mount source=nginx-vol,destination=/usr/share/nginx/html \
  nginx:latest
```

After running either of these examples, run the following commands to clean up the containers and volumes.  Note volume removal is a separate step.

```
$ docker container stop nginxtest

$ docker container rm nginxtest

$ docker volume rm nginx-vol
```

## Use a read-only volume

For some development applications, the container needs to write into the bind mount so that changes are propagated back to the Docker host. At other times, the container only needs read access to the data. Remember that multiple containers can mount the same volume, and it can be mounted read-write for some of them and read-only for others, at the same time.

This example modifies the one above but mounts the directory as a read-only volume, by adding `ro` to the (empty by default) list of options, after the mount point within the container. Where multiple options are present, separate them by commas.

The `--mount` and `-v` examples have the same result.

- `--mount`
- `-v`

```
$ docker run -d \
  --name=nginxtest \
  --mount source=nginx-vol,destination=/usr/share/nginx/html,readonly \
  nginx:latest
```

Use `docker inspect nginxtest` to verify that the readonly mount was created correctly. Look for the `Mounts` section:

```
"Mounts": [
    {
        "Type": "volume",
        "Name": "nginx-vol",
        "Source": "/var/lib/docker/volumes/nginx-vol/_data",
        "Destination": "/usr/share/nginx/html",
        "Driver": "local",
        "Mode": "",
        "RW": false,
        "Propagation": ""
    }
],
```

Stop and remove the container, and remove the volume. Volume removal is a separate step.

```
$ docker container stop nginxtest

$ docker container rm nginxtest

$ docker volume rm nginx-vol
```

## Share data among machines

When building fault-tolerant applications, you might need to configure multiple replicas of the same service to have access to the same files.

![shared storage](https://docs.docker.com/storage/images/volumes-shared-storage.svg)

There are several ways to achieve this when developing your applications. One is to add logic to your application to store files on a cloud object storage system like Amazon S3. Another is to create volumes with a driver that supports writing files to an external storage system like NFS or Amazon S3.

Volume drivers allow you to abstract the underlying storage system from the application logic. For example, if your services use a volume with an NFS driver, you can update the services to use a different driver, as an example to store data in the cloud, without changing the application logic.

## Use a volume driver

When you create a volume using `docker volume create`, or when you start a container which uses a not-yet-created volume, you can specify a volume driver. The following examples use the `vieux/sshfs` volume driver, first when creating a standalone volume, and then when starting a container which creates a new volume.

### Initial set-up

This example assumes that you have two nodes, the first of which is a Docker host and can connect to the second using SSH.

On the Docker host, install the `vieux/sshfs` plugin:

```
$ docker plugin install --grant-all-permissions vieux/sshfs
```

### Create a volume using a volume driver

This example specifies a SSH password, but if the two hosts have shared keys configured, you can omit the password. Each volume driver may have zero or more configurable options, each of which is specified using an `-o` flag.

```
$ docker volume create --driver vieux/sshfs \
  -o sshcmd=test@node2:/home/test \
  -o password=testpassword \
  sshvolume
```

### Start a container which creates a volume using a volume driver

This example specifies a SSH password, but if the two hosts have shared keys configured, you can omit the password. Each volume driver may have zero or more configurable options. **If the volume driver requires you to pass options, you must use the --mount flag to mount the volume, rather than -v.**

```
$ docker run -d \
  --name sshfs-container \
  --volume-driver vieux/sshfs \
  --mount src=sshvolume,target=/app,volume-opt=sshcmd=test@node2:/home/test,volume-opt=password=testpassword \
  nginx:latest
```

### Create a service which creates an NFS volume

This example shows how you can create an NFS volume when creating a service. This example uses `10.0.0.10` as the NFS server and `/var/docker-nfs` as the exported directory on the NFS server. Note that the volume driver specified is `local`.

#### NFSv3

```
$ docker service create -d \
  --name nfs-service \
  --mount 'type=volume,source=nfsvolume,target=/app,volume-driver=local,volume-opt=type=nfs,volume-opt=device=:/var/docker-nfs,volume-opt=o=addr=10.0.0.10' \
  nginx:latest
```

#### NFSv4

```
docker service create -d \
    --name nfs-service \
    --mount 'type=volume,source=nfsvolume,target=/app,volume-driver=local,volume-opt=type=nfs,volume-opt=device=:/,"volume-opt=o=10.0.0.10,rw,nfsvers=4,async"' \
    nginx:latest`
```

## Backup, restore, or migrate data volumes

Volumes are useful for backups, restores, and migrations. Use the `--volumes-from` flag to create a new container that mounts that volume.

### Backup a container

For example, in the next command, we:

- Launch a new container and mount the volume from the `dbstore` container
- Mount a local host directory as `/backup`
- Pass a command that tars the contents of the `dbdata` volume to a `backup.tar` file inside our `/backup` directory.

```
$ docker run --rm --volumes-from dbstore -v $(pwd):/backup ubuntu tar cvf /backup/backup.tar /dbdata
```

When the command completes and the container stops, we are left with a backup of our `dbdata` volume.

### Restore container from backup

With the backup just created, you can restore it to the same container, or another that you made elsewhere.

For example, create a new container named `dbstore2`:

```
$ docker run -v /dbdata --name dbstore2 ubuntu /bin/bash
```

Then un-tar the backup file in the new container`s data volume:

```
$ docker run --rm --volumes-from dbstore2 -v $(pwd):/backup ubuntu bash -c "cd /dbdata && tar xvf /backup/backup.tar --strip 1"
```

You can use the techniques above to automate backup, migration and restore testing using your preferred tools.

## Remove volumes

A Docker data volume persists after a container is deleted. There are two types of volumes to consider:

- **Named volumes** have a specific source from outside the container, for example `awesome:/bar`.
- **Anonymous volumes** have no specific source so when the container is deleted, instruct the Docker Engine daemon to remove them.

### Remove anonymous volumes

To automatically remove anonymous volumes, use the `--rm` option. For example, this command creates an anonymous `/foo` volume. When the container is removed, the Docker Engine removes the `/foo` volume but not the `awesome` volume.

```
$ docker run --rm -v /foo -v awesome:/bar busybox top
```

### Remove all volumes

To remove all unused volumes and free up space:

```
$ docker volume prune
```