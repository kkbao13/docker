







# 概述



- [1: Orientation](https://docs.docker.com/get-started/part1)

- [2: Containers](https://docs.docker.com/get-started/part2)

- [3: Services](https://docs.docker.com/get-started/part3)

- [4: Swarms](https://docs.docker.com/get-started/part4)

- [5: Stacks](https://docs.docker.com/get-started/part5)

- [6: Deploy your app](https://docs.docker.com/get-started/part6)

  Welcome! We are excited that you want to learn Docker. The *Docker Get Started Tutorial* teaches you how to:

1. Set up your Docker environment (on this page)
2. [Build an image and run it as one container](https://docs.docker.com/get-started/part2/)
3. [Scale your app to run multiple containers](https://docs.docker.com/get-started/part3/)
4. [Distribute your app across a cluster](https://docs.docker.com/get-started/part4/)
5. [Stack services by adding a backend database](https://docs.docker.com/get-started/part5/)
6. [Deploy your app to production](https://docs.docker.com/get-started/part6/)



# Part 1: Orientation and setup


## Docker concepts

Docker is a platform for developers and sysadmins to **develop, deploy, and run** applications with containers. The use of Linux containers to deploy applications is called *containerization*. Containers are not new, but their use for easily deploying applications is.

Docker是开发人员和系统管理员使用容器开发，部署和运行应用程序的平台。使用Linux容器部署应用程序称为容器化。容器不是新的，但它们用于轻松部署应用程序。

Containerization is increasingly popular because containers are:

- Flexible: Even the most complex applications can be containerized.
- Lightweight: Containers leverage and share the host kernel.
- Interchangeable: You can deploy updates and upgrades on-the-fly.
- Portable: You can build locally, deploy to the cloud, and run anywhere.
- Scalable: You can increase and automatically distribute container replicas.
- Stackable: You can stack services vertically and on-the-fly.
- 灵活：即使最复杂的应用程序也可以容器化。
- 轻量级：容器利用并共享主机内核。
- 可交换：您可以即时部署更新和升级。
- 便携：您可以在本地构建，部署到云，并在任何地方运行.
- 可扩展：您可以增加并自动分发容器副本。
- 可堆叠：您可以垂直和即时堆叠服务。

![Containers are portable](https://docs.docker.com/get-started/images/laurel-docker-containers.png)

### Images and containers

A container is launched by running an image. An **image** is an executable package that includes everything needed to run an application--the code, a runtime, libraries, environment variables, and configuration files.

A **container** is a runtime instance of an image--what the image becomes in memory when executed (that is, an image with state, or a user process). You can see a list of your running containers with the command, `docker ps`, just as you would in Linux.

​	通过运行`image`启动`container` 。`image`是一个可执行包，包含运行应用程序所需的所有内容 - **代码，运行环境库，环境变量和配置文件**。

​	容器是图像的运行时实例 - 图像在执行时在内存中变为什么（即具有状态的图像或用户进程）。您可以使用命令docker ps查看正在运行的容器列表，就像在Linux中一样。

### Containers and virtual machines

A **container** runs *natively* on Linux and shares the kernel of the host machine with other containers. It runs a discrete process, taking no more memory than any other executable, making it lightweight.

By contrast, a **virtual machine** (VM) runs a full-blown “guest” operating system with *virtual* access to host resources through a hypervisor. In general, VMs provide an environment with more resources than most applications need.

| ![Container stack example](https://docs.docker.com/images/Container%402x.png) | ![Virtual machine stack example](https://docs.docker.com/images/VM%402x.png) |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |

## Prepare your Docker environment

Install a [maintained version](https://docs.docker.com/engine/installation/#updates-and-patches) of Docker Community Edition (CE) or Enterprise Edition (EE) on a [supported platform](https://docs.docker.com/ee/supported-platforms/).

> For full Kubernetes Integration
>
> - [Kubernetes on Docker Desktop for Mac](https://docs.docker.com/docker-for-mac/kubernetes/) is available in [17.12 Edge (mac45)](https://docs.docker.com/docker-for-mac/edge-release-notes/#docker-community-edition-17120-ce-mac45-2018-01-05) or [17.12 Stable (mac46)](https://docs.docker.com/docker-for-mac/release-notes/#docker-community-edition-17120-ce-mac46-2018-01-09) and higher.
> - [Kubernetes on Docker Desktop for Windows](https://docs.docker.com/docker-for-windows/kubernetes/) is available in [18.02 Edge (win50)](https://docs.docker.com/docker-for-windows/edge-release-notes/#docker-community-edition-18020-ce-rc1-win50-2018-01-26) and higher edge channels only.

[Install Docker](https://docs.docker.com/engine/installation/)

### Test Docker version

1. Run `docker --version` and ensure that you have a supported version of Docker:

   ```
   docker --version
   
   Docker version 17.12.0-ce, build c97c6d6
   ```

2. Run `docker info` (or `docker version` without `--`) to view even more details about your Docker installation:

   ```
   docker info
   
   Containers: 0
    Running: 0
    Paused: 0
    Stopped: 0
   Images: 0
   Server Version: 17.12.0-ce
   Storage Driver: overlay2
   ...
   ```

> To avoid permission errors (and the use of `sudo`), add your user to the `docker` group. [Read more](https://docs.docker.com/engine/installation/linux/linux-postinstall/).

### Test Docker installation

1. Test that your installation works by running the simple Docker image, [hello-world](https://hub.docker.com/_/hello-world/):

   ```
   docker run hello-world
   
   Unable to find image 'hello-world:latest' locally
   latest: Pulling from library/hello-world
   ca4f61b1923c: Pull complete
   Digest: sha256:ca0eeb6fb05351dfc8759c20733c91def84cb8007aa89a5bf606bc8b315b9fc7
   Status: Downloaded newer image for hello-world:latest
   
   Hello from Docker!
   This message shows that your installation appears to be working correctly.
   ...
   ```

2. List the `hello-world` image that was downloaded to your machine:

   ```
   docker image ls
   ```

3. List the `hello-world` container (spawned by the image) which exits after displaying its message. If it were still running, you would not need the `--all` option:

   ```
   docker container ls --all
   
   CONTAINER ID     IMAGE           COMMAND      CREATED            STATUS
   54f4984ed6a8     hello-world     "/hello"     20 seconds ago     Exited (0) 19 seconds ago
   ```

## Recap and cheat sheet

```
## List Docker CLI commands
docker
docker container --help

## Display Docker version and info
docker --version
docker version
docker info

## Execute Docker image
docker run hello-world

## List Docker images
docker image ls

## List Docker containers (running, all, all in quiet mode)
docker container ls
docker container ls --all
docker container ls -aq
```

## Conclusion of part one

Containerization makes [CI/CD](https://www.docker.com/solutions/cicd) seamless. For example:

- applications have no system dependencies
- updates can be pushed to any part of a distributed application
- resource density can be optimized.

With Docker, scaling your application is a matter of spinning up new executables, not running heavy VM hosts.



# Get Started, Part 2: Containers

  Prerequisites

- [Install Docker version 1.13 or higher](https://docs.docker.com/engine/installation/).

- Read the orientation in [Part 1](https://docs.docker.com/get-started/).

- Give your environment a quick test run to make sure you’re all set up:

  ```
  docker run hello-world
  ```

## Introduction

​	It’s time to begin building an app the Docker way. We start at the  bottom of the hierarchy of such app, a container, which this page  covers. Above this level is a service, which defines how containers  behave in production, covered in [Part 3](https://docs.docker.com/get-started/part3/). Finally, at the top level is the stack, defining the interactions of all the services, covered in [Part 5](https://docs.docker.com/get-started/part5/).

​	现在是时候开始使用Docker方式构建应用程序了。

- Stack  最后，在顶层是堆栈，定义了第5部分中介绍的所有**服务的交互**。 
- Services  高于此级别的是**一个服务**，它定义了容器在生产中的行为方式，在第3部分中介绍。
- **Container** (you are here)  我们从这个应用**程序层次结构**的**底部**开始，这个页面包含一个容器。

## Your new development environment

​	In the past, if you were to start writing a Python app, your first order of business was to install a Python runtime onto your machine. But, that creates a situation where the environment on your machine needs to be perfect for your app to run as expected, and also needs to match your production environment.

​	With Docker, you can just grab a portable Python runtime as an image, no installation necessary. Then, your build can include the base Python image right alongside your app code, ensuring that your app, its dependencies, and the runtime, all travel together.

​	These portable images are defined by something called a `Dockerfile`.

​	过去，如果您要开始编写Python应用程序，那么您的首要任务就是在您的计算机上安装Python运行时。但是，这会导致您的计算机上的环境需要非常适合您的应用程序按预期运行，并且还需要与您的生产环境相匹配。

​	使用Docker，您可以有可移植的Python运行时**作为image，无需安装**。然后，您的构建可以在您的应用程序代码旁边包含基本Python映像，确保您的应用程序，其**依赖项和运行时都一起运行**。 

​	 <u>可移植 images 由  `Dockerfile`定义</u>

## Define a container with `Dockerfile`

​	 `Dockerfile` defines what goes on in the environment inside your container. Access to resources like networking interfaces and disk drives is virtualized inside this environment, which is isolated from the rest of your system, so you need to map ports to the outside world, and be specific about what files you want to “copy in” to that environment. However, after doing that, you can expect that the build of your app defined in this `Dockerfile` behaves exactly the same wherever it runs.

​	 `Dockerfile`定义`container`内环境中发生的事情。

- 对`networking interfaces and disk drives`网络接口和磁盘驱动器等资源的访问在此环境中进行虚拟化，
- 该环境与系统的其他部分隔离，因此您需要将`端口映射`到外部世界，并具体说明要“复制”到`哪些文件`那个环境。
- 但是，在执行此操作之后，您可以预期在此`Dockerfile`中定义的应用程序的构建在其运行的任何位置都表现完全相同。 

### `Dockerfile`

Create an empty directory on your local machine. Change directories (`cd`) into the new directory, create a file called `Dockerfile`, copy-and-paste the following content into that file, and save it. Take note of the comments that explain each statement in your new Dockerfile.

在本地计算机上创建一个空目录。将目录（cd）更改为新目录，创建名为Dockerfile的文件，将以下内容复制并粘贴到该文件中，然后保存。记下解释新Dockerfile中每个语句的注释。 

```shell
# Use an official Python runtime as a parent image
FROM python:2.7-slim

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# Install any needed packages specified in requirements.txt
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
```

This `Dockerfile` refers to a couple of files we haven’t created yet, namely `app.py` and `requirements.txt`. Let’s create those next.

## The app itself

Create two more files, `requirements.txt` and `app.py`, and put them in the same folder with the `Dockerfile`. This completes our app, which as you can see is quite simple. When the above `Dockerfile` is built into an image, `app.py` and `requirements.txt` is present because of that `Dockerfile`’s `COPY` command, and the output from `app.py` is accessible over HTTP thanks to the `EXPOSE` command.

再创建两个文件，requirements.txt和app.py，并将它们与Dockerfile放在同一个文件夹中。这完成了我们的应用程序，您可以看到它非常简单。当上面的Dockerfile内置到映像中时，由于Dockerfile的COPY命令，app.py和requirements.txt存在，并且由于EXPOSE命令，app.py的输出可通过HTTP访问。 

### `requirements.txt`

```
Flask
Redis
```

### `app.py`

```python
from flask import Flask
from redis import Redis, RedisError
import os
import socket

# Connect to Redis
redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)

app = Flask(__name__)

@app.route("/")
def hello():
    try:
        visits = redis.incr("counter")
    except RedisError:
        visits = "<i>cannot connect to Redis, counter disabled</i>"

    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>" \
           "<b>Visits:</b> {visits}"
    return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname(), visits=visits)

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)
```

Now we see that `pip install -r requirements.txt` installs the Flask and Redis libraries for Python, and the app prints the environment variable `NAME`, as well as the output of a call to `socket.gethostname()`. Finally, because Redis isn’t running (as we’ve only installed the Python library, and not Redis itself), we should expect that the attempt to use it here fails and produces the error message.

> **Note**: Accessing the name of the host when inside a container retrieves the container ID, which is like the process ID for a running executable.

That’s it! You don’t need Python or anything in `requirements.txt` on your system, nor does building or running this image install them on your system. It doesn’t seem like you’ve really set up an environment with Python and Flask, but you have.

现在我们看到`pip install -r requirements.txt`为Python安装Flask和Redis库，app打印环境变量NAME，以及对socket.gethostname（）的调用输出。最后，因为Redis没有运行（因为我们只安装了Python库，而不是Redis本身），我们应该期望在这里使用它的尝试失败并产生错误消息。

而已！您的系统上不需要Python或requirements.txt中的任何内容，构建或运行此映像也不需要在系统上安装它们。你似乎并没有真正设置Python和Flask的环境，但是你有。

## Build the app

We are ready to build the app. Make sure you are still at the top level of your new directory. Here’s what `ls` should show:

```
$ ls
Dockerfile		app.py			requirements.txt
```

Now run the build command. This creates a Docker image, which we’re going to name using the `--tag` option. Use `-t` if you want to use the shorter option.

```
docker build --tag=friendlyhello .
```

Where is your built image? It’s in your machine’s local Docker image registry:

```
$ docker image ls

REPOSITORY            TAG                 IMAGE ID
friendlyhello         latest              326387cea398
```

<u>Note how the tag defaulted to `latest`. The full syntax for the tag option would be something like `--tag=friendlyhello:v0.0.1`.</u>

> Troubleshooting for Linux users
>
> *Proxy server settings*
>
> Proxy servers can block connections to your web app once it’s up and running. If you are behind a proxy server, add the following lines to your Dockerfile, using the `ENV` command to specify the host and port for your proxy servers:
>
> ```
> # Set proxy server, replace host:port with values for your servers
> ENV http_proxy host:port
> ENV https_proxy host:port
> ```
>
> *DNS settings*
>
> DNS misconfigurations can generate problems with `pip`. You need to set your own DNS server address to make `pip` work properly. You might want to change the DNS settings of the Docker daemon. You can edit (or create) the configuration file at `/etc/docker/daemon.json` with the `dns` key, as following:
>
> ```
> {
>   "dns": ["your_dns_address", "8.8.8.8"]
> }
> ```
>
> In the example above, the first element of the list is the address of your DNS server. The second item is Google’s DNS which can be used when the first one is not available.
>
> Before proceeding, save `daemon.json` and restart the docker service.
>
> `sudo service docker restart`
>
> Once fixed, retry to run the `build` command.

## Run the app

Run the app, mapping your machine’s port 4000 to the container’s published port 80 using `-p`:

```
docker run -p 4000:80 friendlyhello
```

You should see a message that Python is serving your app at `http://0.0.0.0:80`. But that message is coming from inside the container, which doesn’t know you mapped port 80 of that container to 4000, making the correct URL `http://localhost:4000`.

Go to that URL in a web browser to see the display content served up on a web page.

![Hello World in browser](https://docs.docker.com/get-started/images/app-in-browser.png)

> **Note**: If you are using Docker Toolbox on Windows 7, use the Docker Machine IP instead of `localhost`. For example, http://192.168.99.100:4000/. To find the IP address, use the command `docker-machine ip`.

You can also use the `curl` command in a shell to view the same content.

```
$ curl http://localhost:4000

<h3>Hello World!</h3><b>Hostname:</b> 8fc990912a14<br/><b>Visits:</b> <i>cannot connect to Redis, counter disabled</i>
```

This port remapping of `4000:80` demonstrates the difference between `EXPOSE` within the `Dockerfile` and what the `publish` value is set to when running `docker run -p`. In later steps, map port 4000 on the host to port 80 in the container and use `http://localhost`.

Hit `CTRL+C` in your terminal to quit.

> On Windows, explicitly stop the container
>
> On Windows systems, `CTRL+C` does not stop the container. So, first  type `CTRL+C` to get the prompt back (or open another shell), then type  `docker container ls` to list the running containers, followed by  `docker container stop <Container NAME or ID>` to stop the  container. Otherwise, you get an error response from the daemon  when you try to re-run the container in the next step.

Now let’s run the app in the background, in detached mode:

```
docker run -d -p 4000:80 friendlyhello
```

You get the long container ID for your app and then are kicked back to your terminal. Your container is running in the background. You can also see the abbreviated container ID with `docker container ls` (and both work interchangeably when running commands):

```
$ docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED
1fa4ab2cf395        friendlyhello       "python app.py"     28 seconds ago
```

Notice that `CONTAINER ID` matches what’s on `http://localhost:4000`.

Now use `docker container stop` to end the process, using the `CONTAINER ID`, like so:

```
docker container stop 1fa4ab2cf395
```

## Share your image

To demonstrate the portability of what we just created, let’s upload our built image and run it somewhere else. After all, you need to know how to push to registries when you want to deploy containers to production.

A registry is a collection of repositories, and a repository is a collection of images—sort of like a GitHub repository, except the code is already built. An account on a registry can create many repositories. The `docker` CLI uses Docker’s public registry by default.

为了演示我们刚刚创建的内容的可移植性，让我们上传我们构建的图像并在其他地方运行它。 毕竟，当您想要将容器部署到生产环境时，您需要知道如何推送到注册表。 

注册表是repositories 的集合，repositories 是images 的集合 - 类似于GitHub存储库，除了代码已经构建。注册表上的帐户可以创建许多存储库。 docker CLI默认使用Docker的公共注册表。 



> **Note**: We use Docker’s public registry here just because it’s free and pre-configured, but there are many public ones to choose from, and you can even set up your own private registry using [Docker Trusted Registry](https://docs.docker.com/datacenter/dtr/2.2/guides/).
>
> 使用公共repository

### Log in with your Docker ID

If you don’t have a Docker account, sign up for one at [hub.docker.com](https://hub.docker.com). Make note of your username.

Log in to the Docker public registry on your local machine.

```
$ docker login
```

### Tag the image

The notation for associating a local image with a repository on a registry is `username/repository:tag`. The tag is optional, but recommended, since it is the mechanism that registries use to give Docker images a version. Give the repository and tag meaningful names for the context, such as `get-started:part2`. This puts the image in the `get-started` repository and tag it as `part2`.

将本地映像与注册表上的存储库相关联的表示法是username / repository：tag 。标签是可选的，但建议使用，因为它是注册管理机构用来为Docker镜像提供版本的机制。 为存储库提供存储库和标记有意义的名称，例如get-started：part2。这会将图像放入启动存储库并将其标记为part2。 

Now, put it all together to tag the image. Run `docker tag image` with your username, repository, and tag names so that the image uploads to your desired destination. The syntax of the command is:

现在，把它们放在一起来标记图像。使用您的用户名，存储库和标记名称运行docker标记图像，以便将图像上载到所需的目标位置。该命令的语法是： 

```
docker tag image username/repository:tag
```

For example:

```
docker tag friendlyhello gordon/get-started:part2
```

Run [docker image ls](https://docs.docker.com/engine/reference/commandline/image_ls/) to see your newly tagged image.

```
$ docker image ls

REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
friendlyhello            latest              d9e555c53008        3 minutes ago       195MB
gordon/get-started         part2               d9e555c53008        3 minutes ago       195MB
python                   2.7-slim            1c7128a655f6        5 days ago          183MB
...
```

### Publish the image

Upload your tagged image to the repository:

```
docker push username/repository:tag
```

Once complete, the results of this upload are publicly available. If you log in to [Docker Hub](https://hub.docker.com/), you see the new image there, with its pull command.

无论docker run在哪里执行，它都会提取你的图像，以及Python和requirements.txt中的所有依赖项，并运行你的代码。它们都在一个整洁的小包中一起旅行，你不需要在主机上安装任何东西让Docker运行它。 

### Pull and run the image from the remote repository

From now on, you can use `docker run` and run your app on any machine with this command:

```
docker run -p 4000:80 username/repository:tag
```

If the image isn’t available locally on the machine, Docker pulls it from the repository.

```
$ docker run -p 4000:80 gordon/get-started:part2
Unable to find image 'gordon/get-started:part2' locally
part2: Pulling from gordon/get-started
10a267c67f42: Already exists
f68a39a6a5e4: Already exists
9beaffc0cf19: Already exists
3c1fe835fb6b: Already exists
4c9f1fa8fcb8: Already exists
ee7d8f576a14: Already exists
fbccdcced46e: Already exists
Digest: sha256:0601c866aab2adcc6498200efd0f754037e909e5fd42069adeff72d1e2439068
Status: Downloaded newer image for gordon/get-started:part2
 * Running on http://0.0.0.0:80/ (Press CTRL+C to quit)
```

No matter where `docker run` executes, it pulls your image, along with Python and all the dependencies from `requirements.txt`, and runs your code. It all travels together in a neat little package, and you don’t need to install anything on the host machine for Docker to run it.

## Conclusion of part two

That’s all for this page. In the next section, we learn how to scale our application by running this container in a **service**.

[Continue to Part 3 >>](https://docs.docker.com/get-started/part3/)

Or, learn how to [launch your container on your own machine using Digital Ocean](https://docs.docker.com/machine/examples/ocean/).

## Recap and cheat sheet (optional)

Here’s [a terminal recording of what was covered on this page](https://asciinema.org/a/blkah0l4ds33tbe06y4vkme6g):

Here is a list of the basic Docker commands from this page, and some related ones if you’d like to explore a bit before moving on.

```shell
docker build -t friendlyhello .  # Create image using this directory's Dockerfile
docker run -p 4000:80 friendlyhello  # Run "friendlyhello" mapping port 4000 to 80
docker run -d -p 4000:80 friendlyhello         # Same thing, but in detached mode
docker container ls                                # List all running containers
docker container ls -a             # List all containers, even those not running
docker container stop <hash>           # Gracefully stop the specified container
docker container kill <hash>         # Force shutdown of the specified container
docker container rm <hash>        # Remove specified container from this machine
docker container rm $(docker container ls -a -q)         # Remove all containers
docker image ls -a                             # List all images on this machine
docker image rm <image id>            # Remove specified image from this machine
docker image rm $(docker image ls -a -q)   # Remove all images from this machine
docker login             # Log in this CLI session using your Docker credentials
docker tag <image> username/repository:tag  # Tag <image> for upload to registry
docker push username/repository:tag            # Upload tagged image to registry
docker run username/repository:tag                   # Run image from a registry
```

### 

# Part 3: Services

   Prerequisites

- [Install Docker version 1.13 or higher](https://docs.docker.com/engine/installation/).
- Get [Docker Compose](https://docs.docker.com/compose/overview/). On [Docker Desktop for Mac](https://docs.docker.com/docker-for-mac/) and [Docker Desktop for Windows](https://docs.docker.com/docker-for-windows/) it’s pre-installed, so you’re good-to-go. On Linux systems you need to [install it directly](https://github.com/docker/compose/releases). On pre Windows 10 systems *without Hyper-V*, use [Docker Toolbox](https://docs.docker.com/toolbox/overview/).
- Read the orientation in [Part 1](https://docs.docker.com/get-started/).
- Learn how to create containers in [Part 2](https://docs.docker.com/get-started/part2/).
- Make sure you have published the `friendlyhello` image you created by [pushing it to a registry](https://docs.docker.com/get-started/part2/#share-your-image). We use that shared image here.
- Be sure your image works as a deployed container. Run this command, slotting in your info for `username`, `repo`, and `tag`: `docker run -p 4000:80 username/repo:tag`, then visit `http://localhost:4000/`.

## Introduction

In part 3, we scale our application and enable load-balancing. To do this, we must go one level up in the hierarchy of a distributed application: the **service**.

- Stack
- **Services** (you are here)
- Container (covered in [part 2](https://docs.docker.com/get-started/part2/))

## About services

In a distributed application, different pieces of the app are called “services”. For example, if you imagine a video sharing site, it probably includes a service for storing application data in a database, a service for video transcoding in the background after a user uploads something, a service for the front-end, and so on.

Services are really just “containers in production.” A service only runs one image, but it codifies the way that image runs—what ports it should use, how many replicas of the container should run so the service has the capacity it needs, and so on. Scaling a service changes the number of container instances running that piece of software, assigning more computing resources to the service in the process.

Luckily it’s very easy to define, run, and scale services with the Docker platform -- just write a `docker-compose.yml` file.

​	在分布式应用程序中，应用程序的不同部分称为“服务”。例如，如果您想象一个视频共享站点，它可能包括一个用于在数据库中存储应用程序数据的服务，一个用户在上传内容后在后台进行视频转码的服务，一个用于前端的服务，等等。

​	**服务实际上只是“生产环境的容器”**。服务只运行一个映像，但它编码了映像**运行的方式**  -  它应该使用哪些**端口**，应该运行**多少个**容器副本，以便服务具有所需的**容量**，以及等等。扩展服务会改变运行该软件的容器实例的数量，为流程中的服务分配更多的计算资源。幸运的是，使用Docker平台定义，运行和扩展服务非常容易 - 只需编写一个`docker-compose.yml`文件即可。

## Your first `docker-compose.yml` file

A `docker-compose.yml` file is a **YAML** file that defines how Docker containers should behave in production.

### `docker-compose.yml`

Save this file as `docker-compose.yml` wherever you want. Be sure you have [pushed the image](https://docs.docker.com/get-started/part2/#share-your-image) you created in [Part 2](https://docs.docker.com/get-started/part2/) to a registry, and update this `.yml` by replacing `username/repo:tag` with your image details.

```yml
version: "3"
services:
  web:
    # replace username/repo:tag with your name and image details
    image: username/repo:tag
    deploy:
      replicas: 5
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
      restart_policy:
        condition: on-failure
    ports:
      - "4000:80"
    networks:
      - webnet
networks:
  webnet:
```

This `docker-compose.yml` file tells Docker to do the following:

- Pull [the image we uploaded in step 2](https://docs.docker.com/get-started/part2/) from the registry.
- Run 5 instances of that image as a service called `web`, limiting each one to use, at most, 10% of a single core of  CPU time (this could also be e.g. “1.5” to mean 1 and half core for each),  and 50MB of RAM.
- Immediately restart containers if one fails.
- Map port 4000 on the host to `web`’s port 80.
- Instruct `web`’s containers to share port 80 via a load-balanced network called `webnet`. (Internally, the containers themselves publish to `web`’s port 80 at an ephemeral port.)
- Define the `webnet` network with the default settings (which is a load-balanced overlay network).

## Run your new load-balanced app

Before we can use the `docker stack deploy` command we first run:

负载均衡软件

```
docker swarm init
```

> **Note**: We get into the meaning of that command in [part 4](https://docs.docker.com/get-started/part4/). If you don’t run `docker swarm init` you get an error that “this node is not a swarm manager.”

Now let’s run it. You need to give your app a name. Here, it is set to `getstartedlab`:

```shell
docker stack deploy -c docker-compose.yml getstartedlab
```

Our single service stack is running 5 container instances of our deployed image on one host. Let’s investigate.

Get the service ID for the one service in our application:

现在让我们来运行吧。您需要为您的应用程序命名。在这里，它被设置为getstartedlab：

在我们的应用程序中获取一项服务的服务ID：

```shell
docker service ls
```

Look for output for the `web` service, prepended with your app name. If you named it the same as shown in this example, the name is `getstartedlab_web`. The service ID is listed as well, along with the number of replicas, image name, and exposed ports.

Alternatively, you can run `docker stack services`, followed by the name of your stack. The following example command lets you view all services associated with the `getstartedlab` stack:

查找Web服务的输出，并以您的应用名称为前缀。如果您将其命名为与此示例中显示的相同，则名称为getstartedlab_web。还列出了服务ID，以及副本数，映像名称和公开端口。

或者，您可以运行`docker stack services`，然后运行堆栈的名称。以下示例命令允许您查看与getstartedlab堆栈关联的所有服务：

```
docker stack services getstartedlab
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
bqpve1djnk0x        getstartedlab_web   replicated          5/5                 username/repo:tag   *:4000->80/tcp
```

A single container running in a service is called a **task**. Tasks are given unique IDs that numerically increment, up to the number of `replicas` you defined in `docker-compose.yml`. List the tasks for your service:

在服务中运行的单个`container`称为`task`。任务被赋予以数字递增的唯一ID，最多为您在docker-compose.yml中定义的副本数。列出您的服务任务：

```
docker service ps getstartedlab_web
```

Tasks also show up if you just list all the containers on your system, though that is not filtered by service:

如果您只列出系统上的所有容器，则任务也会显示，但不会被服务过滤：

```
docker container ls -q
```

You can run `curl -4 http://localhost:4000` several times in a row, or go to that URL in your browser and hit refresh a few times.

![Hello World in browser](https://docs.docker.com/get-started/images/app80-in-browser.png)

Either way, the container ID changes, demonstrating the load-balancing; with each request, one of the 5 tasks is chosen, in a round-robin fashion, to respond. The container IDs match your output from the previous command (`docker container ls -q`).

无论哪种方式，容器ID都会发生变化，从而证明负载均衡;对于每个请求，以循环方式选择5个任务中的一个来响应。容器ID与上一个命令（docker 
container ls -q）的输出匹配。要查看堆栈的所有任务，可以运行docker stack ps，然后运行应用程序名称，如以下示例所示：

To view all tasks of a stack, you can run `docker stack ps` followed by your app name, as shown in the following example:

```shell
docker stack ps getstartedlab
ID                  NAME                  IMAGE               NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
uwiaw67sc0eh        getstartedlab_web.1   username/repo:tag   docker-desktop      Running             Running 9 minutes ago                       
sk50xbhmcae7        getstartedlab_web.2   username/repo:tag   docker-desktop      Running             Running 9 minutes ago                       
c4uuw5i6h02j        getstartedlab_web.3   username/repo:tag   docker-desktop      Running             Running 9 minutes ago                       
0dyb70ixu25s        getstartedlab_web.4   username/repo:tag   docker-desktop      Running             Running 9 minutes ago                       
aocrb88ap8b0        getstartedlab_web.5   username/repo:tag   docker-desktop      Running             Running 9 minutes ago
```

> Running Windows 10?
>
> Windows 10 PowerShell should already have `curl` available, but if not you can grab a Linux terminal emulator like [Git BASH](https://git-for-windows.github.io/), or download [wget for Windows](http://gnuwin32.sourceforge.net/packages/wget.htm) which is very similar.

> Slow response times?
>
> Depending on your environment’s networking configuration, it may take up to 30 seconds for the containers to respond to HTTP requests. This is not indicative of Docker or swarm performance, but rather an unmet Redis dependency that we address later in the tutorial. For now, the visitor counter isn’t working for the same reason; we haven’t yet added a service to persist data.
>
> 根据您环境的网络配置，容器最多可能需要30秒才能响应HTTP请求。这并不表示Docker或swarm性能，而是我们在本教程后面讨论的未满足的Redis依赖性。目前，访客柜台因同样的原因不起作用;我们还没有添加服务来保存数据。

## Scale the app

You can scale the app by changing the `replicas` value in `docker-compose.yml`, saving the change, and re-running the `docker stack deploy` command:

```
docker stack deploy -c docker-compose.yml getstartedlab
```

Docker performs an in-place update, no need to tear the stack down first or kill any containers.

Now, re-run `docker container ls -q` to see the deployed instances reconfigured. If you scaled up the replicas, more tasks, and hence, more containers, are started.

### Take down the app and the swarm

- Take the app down with `docker stack rm`:

  ```
  docker stack rm getstartedlab
  ```

- Take down the swarm.

  ```
  docker swarm leave --force
  ```

It’s as easy as that to stand up and scale your app with Docker. You’ve taken a huge step towards learning how to run containers in production. Up next, you learn how to run this app as a bonafide swarm on a cluster of Docker machines.

> **Note**: Compose files like this are used to define applications with Docker, and can be uploaded to cloud providers using [Docker Cloud](https://docs.docker.com/docker-cloud/), or on any hardware or cloud provider you choose with [Docker Enterprise Edition](https://www.docker.com/enterprise-edition).

[On to “Part 4” >>](https://docs.docker.com/get-started/part4/)

## Recap and cheat sheet (optional)

Here’s [a terminal recording of what was covered on this page](https://asciinema.org/a/b5gai4rnflh7r0kie01fx6lip):

To recap, while typing `docker run` is simple enough, the true implementation of a container in production is running it as a service. Services codify a container’s behavior in a Compose file, and this file can be used to scale, limit, and redeploy our app. Changes to the service can be applied in place, as it runs, using the same command that launched the service: `docker stack deploy`.

Some commands to explore at this stage:

回顾一下，在键入`docker run`时很简单，生产中容器的真正实现是将其作为服务运行。服务在Compose文件中编码容器的行为，此文件可用于扩展，限制和重新部署我们的应用程序。在运行时，可以使用启动服务的相同命令在服务中应用对服务的更改：docker stack deploy。在此阶段要探索的一些命令：

```shell
docker stack ls                                            # List stacks or apps
docker stack deploy -c <composefile> <appname>  # Run the specified Compose file
docker service ls                 # List running services associated with an app
docker service ps <service>                  # List tasks associated with an app
docker inspect <task or container>                   # Inspect task or container
docker container ls -q                                      # List container IDs
docker stack rm <appname>                             # Tear down an application
docker swarm leave --force      # Take down a single node swarm from the manager
```

# Part 4: Swarms

## Prerequisites

- [Install Docker version 1.13 or higher](https://docs.docker.com/engine/installation/).
- Get [Docker Compose](https://docs.docker.com/compose/overview/) as described in [Part 3 prerequisites](https://docs.docker.com/get-started/part3/#prerequisites).
- Get [Docker Machine](https://docs.docker.com/machine/overview/), which is pre-installed with [Docker Desktop for Mac](https://docs.docker.com/docker-for-mac/) and [Docker Desktop for Windows](https://docs.docker.com/docker-for-windows/), but on Linux systems you need to [install it directly](https://docs.docker.com/machine/install-machine/#installing-machine-directly). On pre Windows 10 systems *without Hyper-V*, as well as Windows 10 Home, use [Docker Toolbox](https://docs.docker.com/toolbox/overview/).
- Read the orientation in [Part 1](https://docs.docker.com/get-started/).
- Learn how to create containers in [Part 2](https://docs.docker.com/get-started/part2/).
- Make sure you have published the `friendlyhello` image you created by [pushing it to a registry](https://docs.docker.com/get-started/part2/#share-your-image). We use that shared image here.
- Be sure your image works as a deployed container. Run this command, slotting in your info for `username`, `repo`, and `tag`: `docker run -p 80:80 username/repo:tag`, then visit `http://localhost/`.
- Have a copy of your `docker-compose.yml` from [Part 3](https://docs.docker.com/get-started/part3/) handy.

## Introduction

In [part 3](https://docs.docker.com/get-started/part3/), you took an app you wrote in [part 2](https://docs.docker.com/get-started/part2/), and defined how it should run in production by turning it into a service, scaling it up 5x in the process.

Here in part 4, you deploy this application onto a cluster, running it on multiple machines. Multi-container, multi-machine applications are made possible by joining multiple machines into a “Dockerized” cluster called a **swarm**.

在part3中，您使用了您在`part2`中编写的`app`，并通过将其转换为`service`来定义它应如何在生产中`run`，在此过程中将其扩展5倍。

在part4中，您将此应用程序部署到**集群**中，在多台机器上运行它。通过将多台机器连接到称为群集的“Dockerized”集群，可以实现**多容器**，**多机器**应用程序。

## Understanding Swarm clusters理解Swarm

A swarm is a group of machines that are running Docker and joined into a cluster. After that has happened, you continue to run the Docker commands you’re used to, but now they are executed on a cluster by a **swarm manager**. The machines in a swarm can be physical or virtual. After joining a swarm, they are referred to as **nodes**.

Swarm managers can use several strategies to run containers, such as “emptiest node” -- which fills the least utilized machines with containers. Or “global”, which ensures that each machine gets exactly one instance of the specified container. You instruct the swarm manager to use these strategies in the Compose file, just like the one you have already been using.

Swarm managers are the only machines in a swarm that can execute your commands, or authorize other machines to join the swarm as **workers**. Workers are just there to provide capacity and do not have the authority to tell any other machine what it can and cannot do.

Up until now, you have been using Docker in a single-host mode on your local machine. But Docker also can be switched into **swarm mode**, and that’s what enables the use of swarms. Enabling swarm mode instantly makes the current machine a swarm manager. From then on, Docker runs the commands you execute on the swarm you’re managing, rather than just on the current machine.

**swarm** 是一组运行Docker并加入群集的计算机。在此之后，您继续运行您习惯使用的Docker命令，但现在它们由**swarm manager**在群集上执行。群中的机器可以是物理的或虚拟的。加入群组后，它们被称为**nodes**。

**swarm manager**可以使用多种策略来运行容器，例如“emptiest node” - 它使用容器填充利用率最低的机器。或“global”，它确保每台机器只获得指定容器的一个实例。您指示**swarm管理器**在**Compose**文件中使用这些策略，就像您已经使用的那样。

**swarm manager**是群中唯一可以执行命令的机器，或授权其他机器作为**workers**加入群集。**worker**只是在那里提供能力，没有权力告诉任何其他机器它能做什么和不能做什么。

到目前为止，您一直在本地计算机上以 single-host单主机模式使用Docker。但Docker也可以切换到**swarm mode**，这就是使用群集的原因。立即启用群集模式使当前计算机成为群集管理器。从那时起，Docker就会运行您在管理的swarm上执行的命令，而不仅仅是在当前机器上。

## Set up your swarm

​	A swarm is made up of multiple nodes, which can be either physical or virtual machines. The basic concept is simple enough: run `docker swarm init` to enable swarm mode and make your current machine a swarm manager, then run `docker swarm join` on other machines to have them join the swarm as workers. Choose a tab below to see how this plays out in various contexts. We use VMs to quickly create a two-machine cluster and turn it into a swarm.

​	swarm 由多个nodes组成，可以是物理或虚拟机。基本概念很简单：运行`docker swarm init`以启用**swarm模式**并使当前计算机成为一个**swarm管理器**，然后在其他计算机上运行`docker swarm join`以使它们作为**worker**加入**swarm**。选择下面的标签，了解它在各种情况下的效果。我们使用VM快速创建一个双机群集并将其转换为群集。



#### VMs on your local machine (Mac, Linux, Windows 7 and 8)

You need a hypervisor that can create virtual machines (VMs), so [install Oracle VirtualBox](https://www.virtualbox.org/wiki/Downloads) for your machine’s OS.

> **Note**: If you are on a Windows system that has  Hyper-V installed, such as Windows 10, there is no need to install  VirtualBox and you should use Hyper-V instead. View the instructions for  Hyper-V systems by clicking the Hyper-V tab above. If you are using [Docker Toolbox](https://docs.docker.com/toolbox/overview/), you should already have VirtualBox installed as part of it, so you are good to go.

Now, create a couple of VMs using `docker-machine`, using the VirtualBox driver:

```
docker-machine create --driver virtualbox myvm1
docker-machine create --driver virtualbox myvm2
```

------

#### List the VMs and get their IP addresses

You now have two VMs created, named `myvm1` and `myvm2`.

Use this command to list the machines and get their IP addresses.

> **Note**: you need to run the following as administrator or else you don’t get any resonable output (only “UNKNOWN”).

```
docker-machine ls
```

Here is example output from this command.

```
$ docker-machine ls
NAME    ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
myvm1   -        virtualbox   Running   tcp://192.168.99.100:2376           v17.06.2-ce
myvm2   -        virtualbox   Running   tcp://192.168.99.101:2376           v17.06.2-ce
```

#### Initialize the swarm and add nodes

The first machine acts as the manager, which executes management commands and authenticates workers to join the swarm, and the second is a worker.

You can send commands to your VMs using `docker-machine ssh`. Instruct `myvm1` to become a swarm manager with `docker swarm init` and look for output like this:

```
$ docker-machine ssh myvm1 "docker swarm init --advertise-addr <myvm1 ip>"
Swarm initialized: current node <node ID> is now a manager.

To add a worker to this swarm, run the following command:

  docker swarm join \
  --token <token> \
  <myvm ip>:<port>

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

> Ports 2377 and 2376
>
> Always run `docker swarm init` and `docker swarm join` with port 2377 (the swarm management port), or no port at all and let it take the default.
>
> The machine IP addresses returned by `docker-machine ls` include port 2376, which is the Docker daemon port. Do not use this port or [you may experience errors](https://forums.docker.com/t/docker-swarm-join-with-virtualbox-connection-error-13-bad-certificate/31392/2).

> Having trouble using SSH? Try the --native-ssh flag
>
> Docker Machine has [the option to let you use your own system’s SSH](https://docs.docker.com/machine/reference/ssh/#different-types-of-ssh), if for some reason you’re having trouble sending commands to your Swarm manager. Just specify the `--native-ssh` flag when invoking the `ssh` command:
>
> ```
> docker-machine --native-ssh ssh myvm1 ...
> ```

As you can see, the response to `docker swarm init` contains a pre-configured `docker swarm join` command for you to run on any nodes you want to add. Copy this command, and send it to `myvm2` via `docker-machine ssh` to have `myvm2` join your new swarm as a worker:

```
$ docker-machine ssh myvm2 "docker swarm join \
--token <token> \
<ip>:2377"

This node joined a swarm as a worker.
```

Congratulations, you have created your first swarm!

Run `docker node ls` on the manager to view the nodes in this swarm:

```
$ docker-machine ssh myvm1 "docker node ls"
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS
brtu9urxwfd5j0zrmkubhpkbd     myvm2               Ready               Active
rihwohkh3ph38fhillhhb84sk *   myvm1               Ready               Active              Leader
```

> Leaving a swarm
>
> If you want to start over, you can run `docker swarm leave` from each node.

## Deploy your app on the swarm cluster

The hard part is over. Now you just repeat the process you used in [part 3](https://docs.docker.com/get-started/part3/) to deploy on your new swarm. Just remember that only swarm managers like `myvm1` execute Docker commands; workers are just for capacity.

### Configure a `docker-machine` shell to the swarm manager

So far, you’ve been wrapping Docker commands in `docker-machine ssh` to talk to the VMs. Another option is to run `docker-machine env <machine>` to get and run a command that configures your current shell to talk to the Docker daemon on the VM. This method works better for the next step because it allows you to use your local `docker-compose.yml` file to deploy the app “remotely” without having to copy it anywhere.

Type `docker-machine env myvm1`, then copy-paste and run the command provided as the last line of the output to configure your shell to talk to `myvm1`, the swarm manager.

The commands to configure your shell differ depending on whether you are Mac, Linux, or Windows, so examples of each are shown on the tabs below.

- [Mac, Linux](https://docs.docker.com/get-started/part4/#mac-linux-machine)
- [Windows](https://docs.docker.com/get-started/part4/#win-machine)

#### Docker machine shell environment on Mac or Linux

Run `docker-machine env myvm1` to get the command to configure your shell to talk to `myvm1`.

```
$ docker-machine env myvm1
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.99.100:2376"
export DOCKER_CERT_PATH="/Users/sam/.docker/machine/machines/myvm1"
export DOCKER_MACHINE_NAME="myvm1"
# Run this command to configure your shell:
# eval $(docker-machine env myvm1)
```

Run the given command to configure your shell to talk to `myvm1`.

```
eval $(docker-machine env myvm1)
```

Run `docker-machine ls` to verify that `myvm1` is now the active machine, as indicated by the asterisk next to it.

```
$ docker-machine ls
NAME    ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
myvm1   *        virtualbox   Running   tcp://192.168.99.100:2376           v17.06.2-ce
myvm2   -        virtualbox   Running   tcp://192.168.99.101:2376           v17.06.2-ce
```

------

### Deploy the app on the swarm manager

Now that you have `myvm1`, you can use its powers as a swarm manager to deploy your app by using the same `docker stack deploy` command you used in part 3 to `myvm1`, and your local copy of `docker-compose.yml.`. This command may take a few seconds to complete and the deployment takes some time to be available. Use the `docker service ps <service_name>` command on a swarm manager to verify that all services have been redeployed.

You are connected to `myvm1` by means of the `docker-machine` shell configuration, and you still have access to the files on your local host. Make sure you are in the same directory as before, which includes the [`docker-compose.yml` file you created in part 3](https://docs.docker.com/get-started/part3/#docker-composeyml).

Just like before, run the following command to deploy the app on `myvm1`.

```
docker stack deploy -c docker-compose.yml getstartedlab
```

And that’s it, the app is deployed on a swarm cluster!

> **Note**: If your image is stored on a private registry instead of Docker Hub, you need to be logged in using `docker login <your-registry>` and then you need to add the `--with-registry-auth` flag to the above command. For example:
>
> ```
> docker login registry.example.com
> 
> docker stack deploy --with-registry-auth -c docker-compose.yml getstartedlab
> ```
>
> This passes the login token from your local client to the swarm nodes where the service is deployed, using the encrypted WAL logs. With this information, the nodes are able to log into the registry and pull the image.

Now you can use the same [docker commands you used in part 3](https://docs.docker.com/get-started/part3/#run-your-new-load-balanced-app). Only this time notice that the services (and associated containers) have been distributed between both `myvm1` and `myvm2`.

```
$ docker stack ps getstartedlab

ID            NAME                  IMAGE                   NODE   DESIRED STATE
jq2g3qp8nzwx  getstartedlab_web.1   gordon/get-started:part2  myvm1  Running
88wgshobzoxl  getstartedlab_web.2   gordon/get-started:part2  myvm2  Running
vbb1qbkb0o2z  getstartedlab_web.3   gordon/get-started:part2  myvm2  Running
ghii74p9budx  getstartedlab_web.4   gordon/get-started:part2  myvm1  Running
0prmarhavs87  getstartedlab_web.5   gordon/get-started:part2  myvm2  Running
```

> Connecting to VMs with `docker-machine env` and `docker-machine ssh`
>
> - To set your shell to talk to a different machine like `myvm2`, simply re-run `docker-machine env` in the same or a different shell, then run the given command to point to `myvm2`. This is always specific to the current shell. If you change to an unconfigured shell or open a new one, you need to re-run the commands. Use `docker-machine ls` to list machines, see what state they are in, get IP addresses, and find out which one, if any, you are connected to. To learn more, see the [Docker Machine getting started topics](https://docs.docker.com/machine/get-started/#create-a-machine).
> - Alternatively, you can wrap Docker commands in the form of `docker-machine ssh <machine> "<command>"`, which logs directly into the VM but doesn’t give you immediate access to files on your local host.
> - On Mac and Linux, you can use `docker-machine scp <file> <machine>:~` to copy files across machines, but Windows users need a Linux terminal emulator like [Git Bash](https://git-for-windows.github.io/) for this to work.
>
> This tutorial demos both `docker-machine ssh` and `docker-machine env`, since these are available on all platforms via the `docker-machine` CLI.

### Accessing your cluster

You can access your app from the IP address of **either** `myvm1` or `myvm2`.

The network you created is shared between them and load-balancing. Run `docker-machine ls` to get your VMs’ IP addresses and visit either of them on a browser, hitting refresh (or just `curl` them).

![Hello World in browser](https://docs.docker.com/get-started/images/app-in-browser-swarm.png)

There are five possible container IDs all cycling by randomly, demonstrating the load-balancing.

The reason both IP addresses work is that nodes in a swarm participate in an ingress **routing mesh**. This ensures that a service deployed at a certain port within your swarm always has that port reserved to itself, no matter what node is actually running the container. Here’s a diagram of how a routing mesh for a service called `my-web` published at port `8080` on a three-node swarm would look:

![routing mesh diagram](https://docs.docker.com/engine/swarm/images/ingress-routing-mesh.png)

> Having connectivity trouble?
>
> Keep in mind that to use the ingress network in the swarm, you need to have the following ports open between the swarm nodes before you enable swarm mode:
>
> - Port 7946 TCP/UDP for container network discovery.
> - Port 4789 UDP for the container ingress network.
>
> Double check what you have in the ports section under your web service and make sure the ip addresses you enter in your browser or curl reflects that

## Iterating and scaling your app

From here you can do everything you learned about in parts 2 and 3.

Scale the app by changing the `docker-compose.yml` file.

Change the app behavior by editing code, then rebuild, and push the new image. (To do this, follow the same steps you took earlier to [build the app](https://docs.docker.com/get-started/part2/#build-the-app) and [publish the image](https://docs.docker.com/get-started/part2/#publish-the-image)).

In either case, simply run `docker stack deploy` again to deploy these changes.

You can join any machine, physical or virtual, to this swarm, using the same `docker swarm join` command you used on `myvm2`, and capacity is added to your cluster. Just run `docker stack deploy` afterwards, and your app can take advantage of the new resources.

## Cleanup and reboot

### Stacks and swarms

You can tear down the stack with `docker stack rm`. For example:

```
docker stack rm getstartedlab
```

> Keep the swarm or remove it?
>
> At some point later, you can remove this swarm if you want to with `docker-machine ssh myvm2 "docker swarm leave"` on the worker and `docker-machine ssh myvm1 "docker swarm leave --force"` on the manager, but *you need this swarm for part 5, so keep it around for now*.

### Unsetting docker-machine shell variable settings

You can unset the `docker-machine` environment variables in your current shell with the given command.

On **Mac or Linux** the command is:

```
  eval $(docker-machine env -u)
```

On **Windows** the command is:

```
  & "C:\Program Files\Docker\Docker\Resources\bin\docker-machine.exe" env -u | Invoke-Expression
```

This disconnects the shell from `docker-machine` created virtual machines, and allows you to continue working in the same shell, now using native `docker` commands (for example, on Docker Desktop for Mac or Docker Desktop for Windows). To learn more, see the [Machine topic on unsetting environment variables](https://docs.docker.com/machine/get-started/#unset-environment-variables-in-the-current-shell).

### Restarting Docker machines

If you shut down your local host, Docker machines stops running. You can check the status of machines by running `docker-machine ls`.

```
$ docker-machine ls
NAME    ACTIVE   DRIVER       STATE     URL   SWARM   DOCKER    ERRORS
myvm1   -        virtualbox   Stopped                 Unknown
myvm2   -        virtualbox   Stopped                 Unknown
```

To restart a machine that’s stopped, run:

```
docker-machine start <machine-name>
```

For example:

```
$ docker-machine start myvm1
Starting "myvm1"...
(myvm1) Check network to re-create if needed...
(myvm1) Waiting for an IP...
Machine "myvm1" was started.
Waiting for SSH to be available...
Detecting the provisioner...
Started machines may have new IP addresses. You may need to re-run the `docker-machine env` command.

$ docker-machine start myvm2
Starting "myvm2"...
(myvm2) Check network to re-create if needed...
(myvm2) Waiting for an IP...
Machine "myvm2" was started.
Waiting for SSH to be available...
Detecting the provisioner...
Started machines may have new IP addresses. You may need to re-run the `docker-machine env` command.
```

[On to Part 5 >>](https://docs.docker.com/get-started/part5/)

## Recap and cheat sheet (optional)

Here’s [a terminal recording of what was covered on this page](https://asciinema.org/a/113837):

<iframe src="https://asciinema.org/a/113837/embed?" id="asciicast-iframe-113837" name="asciicast-iframe-113837" scrolling="no" allowfullscreen="true" style="overflow: hidden; margin: 0px; border: 0px none; display: inline-block; width: 884px; float: none; visibility: visible; height: 419px;"></iframe>

In part 4 you learned what a swarm is, how nodes in swarms can be managers or workers, created a swarm, and deployed an application on it. You saw that the core Docker commands didn’t change from part 3, they just had to be targeted to run on a swarm master. You also saw the power of Docker’s networking in action, which kept load-balancing requests across containers, even though they were running on different machines. Finally, you learned how to iterate and scale your app on a cluster.

Here are some commands you might like to run to interact with your swarm and your VMs a bit:

```
docker-machine create --driver virtualbox myvm1 # Create a VM (Mac, Win7, Linux)
docker-machine create -d hyperv --hyperv-virtual-switch "myswitch" myvm1 # Win10
docker-machine env myvm1                # View basic information about your node
docker-machine ssh myvm1 "docker node ls"         # List the nodes in your swarm
docker-machine ssh myvm1 "docker node inspect <node ID>"        # Inspect a node
docker-machine ssh myvm1 "docker swarm join-token -q worker"   # View join token
docker-machine ssh myvm1   # Open an SSH session with the VM; type "exit" to end
docker node ls                # View nodes in swarm (while logged on to manager)
docker-machine ssh myvm2 "docker swarm leave"  # Make the worker leave the swarm
docker-machine ssh myvm1 "docker swarm leave -f" # Make master leave, kill swarm
docker-machine ls # list VMs, asterisk shows which VM this shell is talking to
docker-machine start myvm1            # Start a VM that is currently not running
docker-machine env myvm1      # show environment variables and command for myvm1
eval $(docker-machine env myvm1)         # Mac command to connect shell to myvm1
& "C:\Program Files\Docker\Docker\Resources\bin\docker-machine.exe" env myvm1 | Invoke-Expression   # Windows command to connect shell to myvm1
docker stack deploy -c <file> <app>  # Deploy an app; command shell must be set to talk to manager (myvm1), uses local Compose file
docker-machine scp docker-compose.yml myvm1:~ # Copy file to node's home dir (only required if you use ssh to connect to manager and deploy the app)
docker-machine ssh myvm1 "docker stack deploy -c <file> <app>"   # Deploy an app using ssh (you must have first copied the Compose file to myvm1)
eval $(docker-machine env -u)     # Disconnect shell from VMs, use native docker
docker-machine stop $(docker-machine ls -q)               # Stop all running VMs
docker-machine rm $(docker-machine ls -q) # Delete all VMs and their disk images
```



[swarm](https://docs.docker.com/glossary/?term=swarm), [scale](https://docs.docker.com/glossary/?term=scale), [cluster](https://docs.docker.com/glossary/?term=cluster), [machine](https://docs.docker.com/glossary/?term=machine), [vm](https://docs.docker.com/glossary/?term=vm), [manager](https://docs.docker.com/glossary/?term=manager), [worker](https://docs.docker.com/glossary/?term=worker), [deploy](https://docs.docker.com/glossary/?term=deploy), [ssh](https://docs.docker.com/glossary/?term=ssh), [orchestration](https://docs.docker.com/glossary/?term=orchestration)