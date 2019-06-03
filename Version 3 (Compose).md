# Version 3 (Compose)

## 参考和指导

这些主题描述了撰写文件格式的第3版。这是最新版本。

## **Compose** 和 Docker 兼容性矩阵

有几种版本的 Compose 文件格式 -  1，2，2.x 和3.x. 下表是一个快速查看。有关每个版本包含的内容以及如何升级的完整详细信息，请参阅**关于版本和升级**。

此表显示哪些 Compose 文件版本支持特定的 Docker 版本。

| Compose file format | Docker Engine release |
| :------------------ | :-------------------- |
| 3.3                 | 17.06.0+              |
| 3.2                 | 17.04.0+              |
| 3.1                 | 1.13.1+               |
| 3.0                 | 1.13.0+               |
| 2.3                 | 17.06.0+              |
| 2.2                 | 1.13.0+               |
| 2.1                 | 1.12.0+               |
| 2.0                 | 1.10.0+               |
| 1.0                 | 1.9.1.+               |

除了表格中显示的 Compose 文件格式版本外，Compose 本身也处于发布日程安排中，如 [Compose 发行版中](https://github.com/docker/compose/releases/)所示，但文件格式版本不一定随每个发行版而增加。例如，Compose 文件格式3.0首先在 [Compose 版本1.10.0中](https://github.com/docker/compose/releases/tag/1.10.0)引入，并在随后的版本中逐步版本化。

## **Compose** 文件结构和示例

 示例撰写文件版本3  

```javascript
version: "3"
services:

redis:
  image: redis:alpine
  ports:
    - "6379"
  networks:
    - frontend
  deploy:
    replicas: 2
    update_config:
      parallelism: 2
      delay: 10s
    restart_policy:
      condition: on-failure
db:
  image: postgres:9.4
  volumes:
    - db-data:/var/lib/postgresql/data
  networks:
    - backend
  deploy:
    placement:
      constraints: [node.role == manager]
vote:
  image: dockersamples/examplevotingapp_vote:before
  ports:
    - 5000:80
  networks:
    - frontend
  depends_on:
    - redis
  deploy:
    replicas: 2
    update_config:
      parallelism: 2
    restart_policy:
      condition: on-failure
result:
  image: dockersamples/examplevotingapp_result:before
  ports:
    - 5001:80
  networks:
    - backend
  depends_on:
    - db
  deploy:
    replicas: 1
    update_config:
      parallelism: 2
      delay: 10s
    restart_policy:
      condition: on-failure

worker:
  image: dockersamples/examplevotingapp_worker
  networks:
    - frontend
    - backend
  deploy:
    mode: replicated
    replicas: 1
    labels: [APP=VOTING]
    restart_policy:
      condition: on-failure
      delay: 10s
      max_attempts: 3
      window: 120s
    placement:
      constraints: [node.role == manager]

visualizer:
  image: dockersamples/visualizer:stable
  ports:
    - "8080:8080"
  stop_grace_period: 1m30s
  volumes:
    - "/var/run/docker.sock:/var/run/docker.sock"
  deploy:
    placement:
      constraints: [node.role == manager]

networks:
  frontend:
  backend:

volumes:
  db-data:
```

此参考页面上的主题按顶级键按字母顺序组织，以反映撰写文件本身的结构。定义配置文件中的一部分，如顶级键`build`，`deploy`，`depends_on`，`networks`，等等，都与所有支持他们的子课题的选项中列出。这映射到`<key>: <option>: <value>`撰写文件的缩进结构。

入门教程是一个很好的起点，该教程使用版本3的 Compose 堆栈文件来实现多容器应用程序，服务定义和群集模式。以下是本教程中使用的一些撰写文件。

- 你的第一个 docker-compose.yml 文件
- 添加一项新服务并重新部署

另一个很好的参考是 [Docker for Beginners 实验室中](https://github.com/docker/labs/tree/master/beginner/)使用的投票应用程序示例的 Compose 文件，其中介绍了如何[将应用程序部署到 Swarm](https://github.com/docker/labs/blob/master/beginner/chapters/votingapp/)。这也在本节顶部中显示。

## Service configuration reference 服务配置参考

Compose 文件是一个定义服务，网络和卷的 [YAML ](http://yaml.org/)文件。Compose 文件的默认路径是`./docker-compose.yml`。

>  **提示**：您可以对此文件使用a `.yml`或`.yaml`扩展名。他们都工作。  

服务定义包含将应用于为该服务启动的每个容器的配置，就像传递命令行参数一样`docker run`。同样，网络和卷的定义类似于`docker network create`和`docker volume create`。

正如`docker run`在 Dockerfile 指定选项（例如，`CMD`，`EXPOSE`，`VOLUME`，`ENV`）是默认的尊重-你不需要再次指定它们`docker-compose.yml`。

您可以使用类 Bash `${VARIABLE}`语法在配置值中使用环境变量- 有关完整详细信息，请参阅变量替换。

本节包含版本3中服务定义所支持的所有配置选项的列表。

### build

在构建时应用的配置选项。

`build` 可以指定为包含构建上下文路径的字符串：

```javascript
version: '2'
services:
  webapp:
    build: ./dir
```

或者，作为具有在上下文中指定的路径的对象，并且可以选择  Dockerfile 和 args：

```javascript
version: '2'
services:
  webapp:
    build:
      context: ./dir
      dockerfile: Dockerfile-alternate
      args:
        buildno: 1
```

如果指定`image`以及`build`，然后撰写的名称与内置的镜像`webapp`和可选的`tag`规定`image`：

```javascript
build: ./dir
image: webapp:tag
```

这将产生一个名为`webapp`和标记的图像，由此`tag`构建而成`./dir`。

>  **注意**：使用（版本3） Compose 文件在群集模式下部署堆栈时，将忽略此选项。`docker stack`命令仅接受预先构建的镜像。  

#### context

可以是包含 Dockerfile 的目录的路径，也可以是到 git 存储库的  URL。

当提供的值是相对路径时，它被解释为相对于撰写文件的位置。这个目录也是发送到 Docker 守护进程的构建上下文。

撰写将使用生成的名称进行构建和标记，然后使用该镜像。

```javascript
build:
  context: ./dir
```

#### dockerfile

备用 Dockerfile。

撰写将使用替代文件来构建。还必须指定构建路径。

```javascript
build:
  context: .
  dockerfile: Dockerfile-alternate
```

#### args

添加构建参数，这些参数是仅在构建过程中可访问的环境变量。

首先，在 Dockerfile 中指定参数：

```javascript
ARG buildno
ARG password

RUN echo "Build number: $buildno"
RUN script-requiring-password.sh "$password"
```

然后指定`build`键下的参数。您可以传递映射或列表：

```javascript
build:
  context: .
  args:
    buildno: 1
    password: secret

build:
  context: .
  args:
    - buildno=1
    - password=secret
```

指定构建参数时可以省略该值，在这种情况下，构建时的值是  Compose 运行环境中的值。

```javascript
args:
  - buildno
  - password
```

>  **注**：YAML 布尔值（`true`，`false`，`yes`，`no`，`on`，`off`）必须用引号括起来，这样分析器会将它们解释为字符串。  

#### cache_from

>  **注意：**这个选项在 v3.2中是新的  

引擎将用于缓存解析的图像列表。

```javascript
build:
  context: .
  cache_from:
    - alpine:latest
    - corp/web_app:3.14
```

#### labels

>  **Note:** This option is new in v3.3  

使用 Docker 标签将元数据添加到生成的图像。您可以使用数组或字典。

建议您使用反向 DNS 标记来防止您的标签与其他软件使用的标签冲突。

```javascript
build:
  context: .
  labels:
    com.example.description: "Accounting webapp"
    com.example.department: "Finance"
    com.example.label-with-empty-value: ""


build:
  context: .
  labels:
    - "com.example.description=Accounting webapp"
    - "com.example.department=Finance"
    - "com.example.label-with-empty-value"
```

### cap_add, cap_drop

添加或删除容器功能。请参阅`man 7 capabilities`完整列表。

```javascript
cap_add:
  - ALL

cap_drop:
  - NET_ADMIN
  - SYS_ADMIN
```

>  **注意**：在使用（版本3）Compose 文件的群集模式下部署堆栈时，会忽略这些选项。  

### command

覆盖默认命令。

```javascript
command: bundle exec thin -p 3000
```

该命令也可以是一个列表，方式类似于 dockerfile：

```javascript
command: ["bundle", "exec", "thin", "-p", "3000"]
```

### configs

使用每项服务`configs`配置为每个服务授予对配置的访问权限。支持两种不同的语法变体。

>  **注意**：配置必须已经存在或在`configs`此堆栈文件的顶层配置中定义，否则堆栈部署将失败。  

#### Short syntax

简短的语法变体只能指定配置名称。这允许容器访问配置并将其安装在`/<config_name>`容器内。源名称和目标装入点都设置为配置名称。

以下示例使用简短语法将`redis`服务访问权限授予`my_config`和`my_other_config`configs。值`my_config`被设置为文件的内容`./my_config.txt`，并被`my_other_config`定义为外部资源，这意味着它已经在 Docker 中定义，可以通过运行该`docker config create`命令或通过另一个堆栈部署。如果外部配置不存在，则堆叠部署失败并出现`config not found`错误。

>  **注**：`config`定义仅在3.3版及更高版本的撰写文件格式中受支持。  

```javascript
version: "3.3"
services:
  redis:
    image: redis:latest
    deploy:
      replicas: 1
    configs:
      - my_config
      - my_other_config
configs:
  my_config:
    file: ./my_config.txt
  my_other_config:
    external: true
```

#### Long syntax

长语法提供了在服务的任务容器中如何创建配置的更细粒度。

- `source`：它在 Docker 中存在的配置名称。
- `target`：将被安装在服务的任务容器中的文件的路径和名称。`/<source>`如果未指定，则默认为。
- `uid`以及`gid`：将在服务的任务容器中拥有安装的配置文件的数字 UID 或 GID。`0`如果未指定，则默认为在 Linux 上。Windows 不支持。
- `mode`：将以八进制表示法将要装入服务的任务容器内的文件的权限。例如，`0444`代表世界可读的。默认是`0444`。配置文件无法写入，因为它们安装在临时文件系统中，所以如果设置了可写位，它将被忽略。可执行位可以被设置。如果您不熟悉 UNIX 文件权限模式，则可能会发现此[权限计算器](http://permissions-calculator.org/)很有用。

以下示例在容器中设置`my_config`的名称`redis_config`，将模式设置为`0440`（group-readable）并将用户和组设置为`103`。该`redis`服务无法访问`my_other_config`配置。

```javascript
version: "3.3"
services:
  redis:
    image: redis:latest
    deploy:
      replicas: 1
    configs:
      - source: my_config
        target: /redis_config
        uid: '103'
        gid: '103'
        mode: 0440
configs:
  my_config:
    file: ./my_config.txt
  my_other_config:
    external: true
```

您可以授予多个配置的服务访问权限，您可以混合使用长短语法。定义配置并不意味着授予服务访问权限。

### cgroup_parent

为容器指定一个可选的父 cgroup。

```javascript
cgroup_parent: m-executor-abcd
```

>  **注意**：使用（版本3）Compose 文件在群集模式下部署堆栈时，将忽略此选项。  

### container_name

指定一个自定义容器名称，而不是生成的默认名称。

```javascript
container_name: my-web-container
```

由于 Docker 容器名称必须是唯一的，因此如果您指定了自定义名称，则无法将服务扩展到1个容器之外。试图这样做会导致错误。

>  **注意**：使用（版本3）Compose 文件在群集模式下部署堆栈时，将忽略此选项。  

### credential_spec

>  **注意：**该选项已添加到 v3.3中  

为托管服务帐户配置凭据规范。此选项仅用于使用 Windows 容器的服务。在`credential_spec`必须在格式`file://<filename>`或`registry://<value-name>`。

使用时`file:`，引用的文件必须存在于`CredentialSpecs`docker 数据目录的子目录中，该目录默认为`C:\ProgramData\Docker\`在 Windows 上。以下示例从名为的文件加载凭证规范`C:\ProgramData\Docker\CredentialSpecs\my-credential-spec.json`：

```javascript
credential_spec:
  file: my-credential-spec.json
```

使用时`registry:`，将从守护进程主机上的 Windows 注册表中读取凭据规范。具有给定名称的注册表值必须位于：

```javascript
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers\CredentialSpecs
```

以下示例从`my-credential-spec`注册表中指定的值加载凭证规范：

```javascript
credential_spec:
  registry: my-credential-spec
```

### 部署

> 仅版本3  

指定与部署和运行服务相关的配置。这仅在部署到具有 docker 堆栈部署的群集时生效，并且被 docker-compose up 和 docker-compose run 忽略。

```javascript
version: '3'
services:
  redis:
    image: redis:alpine
    deploy:
      replicas: 6
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure
```

有几个子选项可供选择：

#### endpoint_mode

为连接到群集的外部客户端指定服务的发现方法。

> 仅版本3.3  

endpoint_mode：vip   -  Docker 为服务分配一个虚拟IP（VIP），作为客户端到达网络服务的“前端”。 Docker  在客户端和服务的可用工作节点之间路由请求，而无需客户端知道有多少节点参与服务或其IP地址或端口。 （这是默认设置。）

endpoint_mode：dnsrr   -  DNS 循环（DNSRR）服务发现不使用单个虚拟IP。 Docker 为服务设置DNS条目，以便服务名称的 DNS  查询返回IP地址列表，并且客户端直接连接到其中一个。如果您要使用自己的负载均衡器，或者对于混合 Windows 和 Linux 应用程序，DNS  循环法非常有用。

```javascript
version: "3.3"

services:
  wordpress:
    image: wordpress
    ports:
      - 8080:80
    networks:
      - overlay
    deploy:
      mode: replicated
      replicas: 2
      endpoint_mode: vip

  mysql:
    image: mysql
    volumes:
       - db-data:/var/lib/mysql/data
    networks:
       - overlay
    deploy:
      mode: replicated
      replicas: 2
      endpoint_mode: dnsrr

volumes:
  db-data:

networks:
  overlay:
```

endpoint_mode 的选项也可用作 swarm 模式 CLI 命令 docker service create 上的标志。有关所有与swarm 相关的 docker 命令的快速列表，请参阅 Swarm 模式 CLI 命令。



要了解有关群集模式下的服务发现和网络的更多信息，请参阅在群集模式主题中配置服务发现。

#### 标签

指定服务的标签。这些标签只能在服务上设置，而不能在服务的任何容器上设置。

```javascript
version: "3"
services:
  web:
    image: web
    deploy:
      labels:
        com.example.description: "This label will appear on the web service"
```

要在容器上设置标签，请使用 deploy 之外的 labels 键：

```javascript
version: "3"
services:
  web:
    image: web
    labels:
      com.example.description: "This label will appear on all containers for the web service"
```

#### 模型

global （每个群集节点恰好一个容器）或 replicated（指定数量的容器）。默认值已 replicated 。 （要了解更多信息，请参阅群组主题中的复制和全局服务。）

```javascript
version: '3'
services:
  worker:
    image: dockersamples/examplevotingapp_worker
    deploy:
      mode: global
```

#### 放置

指定放置约束。有关语法和可用约束类型的完整说明，请参阅docker service create documentation



```javascript
version: '3'
services:
  db:
    image: postgres
    deploy:
      placement:
        constraints:
          - node.role == manager
          - engine.labels.operatingsystem == ubuntu 14.04
```

#### 副本

如果 replicated 了服务（这是默认设置），请指定在任何给定时间应运行的容器数。

```javascript
version: '3'
services:
  worker:
    image: dockersamples/examplevotingapp_worker
    networks:
      - frontend
      - backend
    deploy:
      mode: replicated
      replicas: 6
```

资源

配置资源限制。这将替换版本3之前的 Compose 文件中的旧资源约束选项（cpu_shares，cpu_quota，cpuset，mem_limit，memswap_limit，mem_swappiness）。



这些中的每一个都是单个值，类似于其docker service create对应物



```javascript
version: '3'
services:
  redis:
    image: redis:alpine
    deploy:
      resources:
        limits:
          cpus: '0.001'
          memory: 50M
        reservations:
          cpus: '0.0001'
          memory: 20M
```

##### Out Of Memory Exceptions (OOME)

如果您的服务或容器尝试使用的内存超过系统可用的内存，则可能会遇到内存不足异常（OOME），并且容器或 Docker 守护程序可能会被内核 OOM 杀手杀死。要防止这种情况发生，请确保您的应用程序在具有足够内存的主机上运行，并参阅了解内存不足的风险。

#### restart_policy



配置是否以及如何在容器退出时重新启动容器。替换 restart

condition：none，on-failure 或 any（默认值：any）之一。

 delay：重新启动尝试之间等待的时间，指定为持续时间（默认值：0）。 

max_attempts：在放弃之前尝试重启容器的次数（默认值：永不放弃）。 

window：在判断重启是否成功之前等待多长时间，指定为持续时间（默认值：立即决定）。



```javascript
version: "3"
services:
  redis:
    image: redis:alpine
    deploy:
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
        window: 120s
```

#### update_config

配置服务应如何更新。用于配置滚动更新。

parallelism：一次更新的容器数。 

delay：更新一组容器之间的等待时间。 

failure_action：如果更新失败怎么办。continue，rollback 或 pause 之一（默认值：pause）。 

monitor：每次更新任务后的持续时间以监视失败（ns | us | ms | s | m | h）（默认为0）。 

max_failure_ratio：更新期间容忍的失败率。



```javascript
version: '3'
services:
  vote:
    image: dockersamples/examplevotingapp_vote:before
    depends_on:
      - redis
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
        delay: 10s
```

#### 不支持 `docker stack deploy`

Docker 堆栈部署或部署密钥不支持以下子选项（支持 docker compose up 和 docker compose run）。

- build
- cgroup_parent
- container_name
- devices
- dns
- dns_search
- tmpfs
- external_links
- links
- network_mode
- security_opt
- stop_signal
- sysctls
- userns_mode

> 提示：另请参阅有关如何为服务，swarms 和 docker-stack.yml 文件配置卷的部分。支持卷但是为了使用群集和服务，必须将它们正确配置为命名卷，或者与受限于可访问必需卷的节点的服务相关联。  

### 设备

设备映射列表。使用与--device docker client create 选项相同的格式



```javascript
devices:
  - "/dev/ttyUSB0:/dev/ttyUSB0"
```

> 注意：使用（版本3）Compose 文件在群集模式下部署堆栈时，将忽略此选项。

### depends_on

服务之间的快速依赖关系，有两个影响：

docker-compose up 将按依赖顺序启动服务。在以下示例中，db 和 redis 将在 web 之前启动。

docker-compose up SERVICE 将自动包含 SERVICE 的依赖项。在以下示例中，docker-compose up web 还将创建并启动 db 和 redis 。

简单的例子：

```javascript
version: '3'
services:
  web:
    build: .
    depends_on:
      - db
      - redis
  redis:
    image: redis
  db:
    image: postgres
```

使用depends_on时有几点需要注意： 

depends_on 不会等到 db 和 redis 在启动 web 之前“准备好” - 只有在它们启动之前。如果您需要等待服务准备就绪，请参阅控制启动顺序以了解有关此问题的更多信息以及解决此问题的策略。

版本3不再支持 depends_on 的条件形式。

在具有版本3 Compose 文件的 swarm 模式下部署堆栈时，将忽略 depends_on 选项。

### dns

自定义DNS服务器。可以是单个值或列表。

```javascript
dns: 8.8.8.8
dns:
  - 8.8.8.8
  - 9.9.9.9
```

>  注意：使用（版本3）Compose 文件在群集模式下部署堆栈时，将忽略此选项。

### dns_search

自定义 DNS 搜索域。可以是单个值或列表。

```javascript
dns_search: example.com
dns_search:
  - dc1.example.com
  - dc2.example.com
```

注意：使用（版本3）Compose 文件在群集模式下部署堆栈时，将忽略此选项。

### tmpfs

> 版本2文件格式及以上。

在容器内安装临时文件系统。可以是单个值或列表。

```javascript
tmpfs: /run
tmpfs:
  - /run
  - /tmp
```

> 注意：使用（版本3）Compose 文件在群集模式下部署堆栈时，将忽略此选项。

### 入口点

覆盖默认入口点。

```javascript
entrypoint: /code/entrypoint.sh
```

入口点也可以是一个列表，方式类似于dockerfile：

```javascript
entrypoint:
    - php
    - -d
    - zend_extension=/usr/local/lib/php/extensions/no-debug-non-zts-20100525/xdebug.so
    - -d
    - memory_limit=-1
    - vendor/bin/phpunit
```

注意：设置入口点将覆盖使用 ENTRYPOINT Dockerfile 指令在服务图像上设置的任何默认入口点，并清除图像上的任何默认命令 - 这意味着如果 Dockerfile 中有 CMD 指令，它将被忽略。

### env_file

```javascript
从文件添加环境变量。可以是单个值或列表。
```



如果已使用docker-compose -f FILE 指定了 Compose 文件，则 env_file 中的路径相对于该文件所在的目录。



在环境部分中声明的环境变量会覆盖这些值 - 即使这些值为空或未定义也适用。



```javascript
env_file: .env

env_file:
  - ./common.env
  - ./apps/web.env
  - /opt/secrets.env
```

Compose 期望 env 文件中的每一行都是 VAR = VAL 格式。以＃开头的行（即注释）将被忽略，空行也将被忽略。



```javascript
# Set Rails/Rack environment
RACK_ENV=development
```

>  注意：如果您的服务指定了构建选项，则在构建期间，环境文件中定义的变量将不会自动显示。使用build 的 args 子选项来定义构建时环境变量。



VAL 的值按原样使用，根本不进行修改。例如，如果值由引号括起（通常是 shell 变量的情况），则引号将包含在传递给 Compose 的值中。



请记住，列表中文件的顺序对于确定分配给多次显示的变量的值非常重要。列表中的文件从上到下进行处理。对于文件  a.env 中指定的相同变量，并在文件 b.env 中指定了不同的值，如果 b.env 列在下面（后面），则 b.env 中的值表示。例如，在  docker_compose.yml 中给出以下声明：



```javascript
services:
  some-service:
    env_file:
      - a.env
      - b.env
```

如下文件；

```javascript
# a.env
VAR=1
```

和

```javascript
# b.env
VAR=hello
```

$VAR 将会成为 `hello`.

### 环境

添加环境变量。您可以使用数组或字典。任何布尔值; true，false，yes no，需要用引号括起来，以确保YML解析器不会将它们转换为 True 或 False 。

仅具有键的环境变量将解析为计算机正在运行的计算机上的值，这对于特定于机密或特定于主机的值很有用。



```javascript
environment:
  RACK_ENV: development
  SHOW: 'true'
  SESSION_SECRET:

environment:
  - RACK_ENV=development
  - SHOW=true
  - SESSION_SECRET
```

**注意**：如果您的服务指定了构建选项，`environment`则在构建期间将*不会*自动显示定义的变量。使用args子选项`build`定义构建时环境变量。



### expose

暴露端口而不将它们发布到主机 - 它们只能被链接服务访问。只能指定内部端口。

```javascript
expose:
 - "3000"
 - "8000"
```

### 外部链接

链接到此组件之外`docker-compose.yml`或甚至在 Compose 之外的容器，尤其是对于提供共享或公共服务的容器。在指定容器名称和链接别名（）时，`external_links`遵循类似于遗留选项的语义。`linksCONTAINER:ALIAS`

```javascript
external_links:
 - redis_1
 - project_db_1:mysql
 - project_db_1:postgresql
```

>  **注意：**   如果您使用的是版本2或更高版本的文件格式，则外部创建的容器必须至少连接到与链接到它们的服务相同的网络之一。从版本2开始，链接是一种传统选项。我们建议使用网络。在群集模式下使用（版本3）Compose 文件部署堆栈时，将忽略此选项。  

### extra_hosts

添加主机名映射。使用与docker client `--add-host`参数相同的值。

```javascript
extra_hosts:
 - "somehost:162.242.195.82"
 - "otherhost:50.31.209.229"
```

将在`/etc/hosts`此服务的内部容器中创建具有 ip 地址和主机名的条目，例如：

```javascript
162.242.195.82  somehost
50.31.209.229   otherhost
```

### 健康检查

>  版本2.1文件格式及以上。  

配置运行的检查以确定此服务的容器是否“健康”。有关healthchecks如何工作的详细信息，请参阅HEALTHCHECK Dockerfile指令的文档。

```javascript
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost"]
  interval: 1m30s
  timeout: 10s
  retries: 3
```

`interval`并`timeout`指定为持续时间。

`test`必须是字符串或列表。如果是列表，则第一项必须是`NONE`，`CMD`或者`CMD-SHELL`。如果它是一个字符串，则相当于指定`CMD-SHELL`后跟该字符串。

```javascript
# Hit the local web app
test: ["CMD", "curl", "-f", "http://localhost"]

# As above, but wrapped in /bin/sh. Both forms below are equivalent.
test: ["CMD-SHELL", "curl -f http://localhost && echo 'cool, it works'"]
test: curl -f https://localhost && echo 'cool, it works'
```

要禁用图像设置的任何默认运行状况检查，您可以使用`disable: true`。这相当于指定`test: ["NONE"]`。

```javascript
healthcheck:
  disable: true
```

### 图片

指定图像以从中启动容器。可以是存储库/标记或部分图像 ID 。

```javascript
image: redis
image: ubuntu:14.04
image: tutum/influxdb
image: example-registry.com:4000/postgresql
image: a4bc65fd
```

如果图像不存在，Compose 尝试拉取它，除非您还指定了构建，在这种情况下，它使用指定的选项构建它并使用指定的标记对其进行标记。

### 隔离

指定容器的隔离技术。在 Linux 上，唯一支持的值是`default`。在 Windows 中，可接受的值是`default`，`process`和`hyperv`。有关详细信息，请参阅 Docker Engine 文档。

### 标签

使用 Docker 标签向容器添加元数据。您可以使用数组或字典。

建议您使用反向 DNS 表示法来防止标签与其他软件使用的标签冲突。

```javascript
labels:
  com.example.description: "Accounting webapp"
  com.example.department: "Finance"
  com.example.label-with-empty-value: ""

labels:
  - "com.example.description=Accounting webapp"
  - "com.example.department=Finance"
  - "com.example.label-with-empty-value"
```

### 链接

链接到另一个服务中的容器。指定服务名称和链接别名（`SERVICE:ALIAS`），或仅指定服务名称。

```javascript
web:
  links:
   - db
   - db:database
   - redis
```

链接服务的容器可以在与别名相同的主机名上访问，如果未指定别名，则可以访问服务名称。

启用服务进行通信不需要链接 - 默认情况下，任何服务都可以通过该服务的名称访问任何其他服务。（另请参阅撰写网络中的链接主题。）

链接还以与depends_on相同的方式表示服务之间的依赖关系，因此它们确定服务启动的顺序。

> 注意

- 如果同时定义链接和网络，则它们之间具有链接的服务必须共享至少一个共同的网络才能进行通信。
- 在群集模式下使用（版本3）Compose 文件部署堆栈时，将忽略此选项。

### 记录

记录服务的配置。

```javascript
logging:
  driver: syslog
  options:
    syslog-address: "tcp://192.168.0.42:123"
```

该`driver`名称指定服务容器的日志记录驱动程序，与`--log-driver`docker run选项一起使用（此处记录）。

默认值为json-file。

```javascript
driver: "json-file"
driver: "syslog"
driver: "none"
```

>  **注意**：只有`json-file`和`journald`驱动程序可以直接从`docker-compose up`和使用日志`docker-compose logs`。使用任何其他驱动程序将不会打印任何日志。  

使用`options`密钥指定日志记录驱动程序的日志记录选项，与`--log-opt`选项一样`docker run`。

记录选项是键值对。`syslog`选项的一个例子：

```javascript
driver: "syslog"
options:
  syslog-address: "tcp://192.168.0.42:123"
```

默认驱动程序 json-file 具有限制存储日志量的选项。为此，请使用键值对来获得最大存储大小和最大文件数：

```javascript
options:
  max-size: "200k"
  max-file: "10"
```

上面显示的示例将存储日志文件，直到它们达到`max-size`200kB，然后旋转它们。存储的各个日志文件的数量由`max-file`值指定。随着日志超出最大限制，将删除较旧的日志文件以允许存储新日志。

以下是`docker-compose.yml`限制日志记录存储的示例文件：

```javascript
services:
  some-service:
    image: some-service
    logging:
      driver: "json-file"
      options:
        max-size: "200k"
        max-file: "10"
```

> 可用的日志选项取决于您使用的日志驱动程序上面的控制日志文件和大小的示例使用特定于json文件驱动程序的选项。其他日志记录驱动程序不提供这些特定选项。有关支持的日志记录驱动程序及其选项的完整列表，请参阅记录驱动程序  

### 网络模式

网络模式。使用与 docker client `--net`参数相同的值以及特殊表单`service:[service name]`。

```javascript
network_mode: "bridge"
network_mode: "host"
network_mode: "none"
network_mode: "service:[service name]"
network_mode: "container:[container name/id]"
```

> 注意

- 在群集模式下使用（版本3）Compose 文件部署堆栈时，将忽略此选项。
- `network_mode: "host"` 不能与链接混在一起。

### 网络

要加入的网络，引用顶级`networks`密钥下的条目。

```javascript
services:
  some-service:
    networks:
     - some-network
     - other-network
```

#### 别名

网络上此服务的别名（备用主机名）。同一网络上的其他容器可以使用服务名称或此别名连接到其中一个服务的容器。

由于`aliases`是网络范围的，因此相同的服务可以在不同的网络上具有不同的别名。

>  **注意**：网络范围的别名可以由多个容器共享，甚至可以由多个服务共享。如果是，则无法保证名称将解析为哪个容器。  

一般格式如下所示。

```javascript
services:
  some-service:
    networks:
      some-network:
        aliases:
         - alias1
         - alias3
      other-network:
        aliases:
         - alias2
```

在下面的例子中，提供了三种服务（`web`，`worker`，和`db`），其中两个网络（沿`new`和`legacy`）。该`db`服务是在到达的主机名`db`或`database`上`new`网络，并`db`或`mysql`将上`legacy`网络。

```javascript
version: '2'

services:
  web:
    build: ./web
    networks:
      - new

  worker:
    build: ./worker
    networks:
      - legacy

  db:
    image: mysql
    networks:
      new:
        aliases:
          - database
      legacy:
        aliases:
          - mysql

networks:
  new:
  legacy:
```

#### ipv4_address, ipv6_address

在加入网络时为此服务指定容器的静态IP地址。

顶级网络部分中的相应网络配置必须具有`ipam`包含每个静态地址的子网配置的块。如果需要IPv6寻址，则`enable_ipv6`必须设置该选项。

一个例子：

```javascript
version: '2.1'

services:
  app:
    image: busybox
    command: ifconfig
    networks:
      app_net:
        ipv4_address: 172.16.238.10
        ipv6_address: 2001:3984:3989::10

networks:
  app_net:
    driver: bridge
    enable_ipv6: true
    ipam:
      driver: default
      config:
      -
        subnet: 172.16.238.0/24
      -
        subnet: 2001:3984:3989::/64
```

### pid

```javascript
pid: "host"
```

将 PID 模式设置为主机 PID 模式。这打开了容器和主机操作系统之间的 PID 地址空间共享。使用此标志启动的容器将能够访问和操作裸机计算机命名空间中的其他容器，反之亦然。

### ports

Expose ports.

#### 短语法

既指定 ports（`HOST:CONTAINER`），也指定容器端口（将选择随机主机端口）。

>  **注意**：以`HOST:CONTAINER`格式映射端口时，使用低于60的容器端口时可能会遇到错误的结果，因为YAML将以格式`xx:yy`为sexagesimal（基数为60）解析数字。因此，我们建议始终将端口映射明确指定为字符串。  

```javascript
ports:
 - "3000"
 - "3000-3005"
 - "8000:8000"
 - "9090-9091:8080-8081"
 - "49100:22"
 - "127.0.0.1:8001:8001"
 - "127.0.0.1:5000-5010:5000-5010"
 - "6060:6060/udp"
```

#### 长语法

长格式语法允许配置无法以简短形式表示的其他字段。

- `target`: the port inside the container
- `published`: the publicly exposed port
- `protocol`: the port protocol (`tcp` or `udp`)
- `mode`: `host` for publishing a host port on each node, or `ingress` for a swarm mode port which will be load balanced.

```javascript
ports:
  - target: 80
    published: 8080
    protocol: tcp
    mode: host
```

>  **注意：**长语法是v3.2中的新增功能  

### secrets

使用每服务`secrets`配置基于每个服务授予对秘密的访问权限。支持两种不同的语法变体。

>  **注意**：秘密必须已存在或在`secrets`此堆栈文件的顶级配置中定义，否则堆栈部署将失败。  

#### 短语法

短语法变体仅指定机密名称。这允许容器访问秘密并将其安装在`/run/secrets/<secret_name>`容器内。源名称和目标安装点都设置为机密名称。

以下示例使用短语法授予`redis`服务访问权限`my_secret`和`my_other_secret`机密。值的值`my_secret`设置为文件的内容`./my_secret.txt`，并被`my_other_secret`定义为外部资源，这意味着它已经在Docker中定义，可以通过运行`docker secret create`命令或通过其他堆栈部署来定义。如果外部机密不存在，则堆栈部署失败并显示`secret not found`错误。

```javascript
version: "3.1"
services:
  redis:
    image: redis:latest
    deploy:
      replicas: 1
    secrets:
      - my_secret
      - my_other_secret
secrets:
  my_secret:
    file: ./my_secret.txt
  my_other_secret:
    external: true
```

#### 长语法

长语法提供了在服务的任务容器中如何创建秘密的更多粒度。

- `source`：Docker 中存在的秘密名称。
- `target`：将`/run/secrets/`在服务的任务容器中装入的文件的名称。`source`如果未指定，则默认为。
- `uid`和`gid`：将`/run/secrets/`在服务的任务容器中拥有该文件的数字UID或GID 。`0`如果未指定，则默认为默认值。
- `mode`：将以`/run/secrets/`八进制表示法在服务的任务容器中装入的文件的权限。例如，`0444`代表世界可读。Docker 1.13.1中的默认值是`0000`，但将来会出现`0444`。秘密不能写，因为它们安装在临时文件系统中，所以如果设置了可写位，它将被忽略。可以设置可执行位。如果您不熟悉 UNIX 文件权限模式，则可能会发现此[权限计算器](http://permissions-calculator.org/)很有用。

以下示例设置容器中`my_secret`to的名称`redis_secret`，将模式设置为`0440`（group-readable）并将用户和组设置为`103`。该`redis`服务无法访问该`my_other_secret`秘密。

```javascript
version: "3.1"
services:
  redis:
    image: redis:latest
    deploy:
      replicas: 1
    secrets:
      - source: my_secret
        target: redis_secret
        uid: '103'
        gid: '103'
        mode: 0440
secrets:
  my_secret:
    file: ./my_secret.txt
  my_other_secret:
    external: true
```

您可以授予对多个机密的服务访问权限，您可以混合使用长短语法。定义机密并不意味着授予服务访问权限。

### security_opt

覆盖每个容器的默认标签方案。

```javascript
security_opt:
  - label:user:USER
  - label:role:ROLE
```

>  **注意**：使用（版本3）Compose 文件在群集模式下部署堆栈时，将忽略此选项。  

### stop_grace_period

指定`stop_signal`在发送SIGKILL之前，如果它未处理SIGTERM（或指定了任何停止信号），则尝试停止容器时要等待多长时间。指定为持续时间。

```javascript
stop_grace_period: 1s
stop_grace_period: 1m30s
```

默认情况下，`stop`在发送SIGKILL之前等待容器退出10秒。

### stop_signal

设置替代信号以停止容器。默认情况下`stop`使用SIGTERM。使用替代信号设置`stop_signal`将导致`stop`发送该信号。

```javascript
stop_signal: SIGUSR1
```

>  **注意**：使用（版本3）Compose 文件在群集模式下部署堆栈时，将忽略此选项。  

### sysctls

要在容器中设置的内核参数。您可以使用数组或字典。

```javascript
sysctls:
  net.core.somaxconn: 1024
  net.ipv4.tcp_syncookies: 0

sysctls:
  - net.core.somaxconn=1024
  - net.ipv4.tcp_syncookies=0
```

>  **注意**：使用（版本3）Compose 文件在群集模式下部署堆栈时，将忽略此选项。  

### ulimits

覆盖容器的默认 ulimits 。您可以将单个限制指定为整数，也可以将软/硬限制指定为映射。

```javascript
ulimits:
  nproc: 65535
  nofile:
    soft: 20000
    hard: 40000
```

### userns_mode

```javascript
userns_mode: "host"
```

如果 Docker 守护程序配置了用户名称空间，则禁用此服务的用户名称空间。有关更多信息，请参阅 dockerd 。

>  **注意**：使用（版本3）Compose 文件在群集模式下部署堆栈时，将忽略此选项。  

### volumes 卷

​	装入主机路径或命名卷，指定为服务的子选项。装载mount一个主机**路径path**作为**单个服务**定义的一部分，无需在顶级`volumes`键中定义。

​	如果要跨**多个服务**重用**卷volume** ，请在顶级`volumes`键中定义命名卷。将命名卷与服务，群组和堆栈文件一起使用。

>  **注意**：**顶级卷**键定义了命名卷，并从每个服务`volumes`列表中引用它。这取代了`volumes_from`早期版本的Compose文件格式。有关卷的一般信息，请参阅使用卷和卷插件。  

​	下面这个例子，展示了**命名卷**（`mydata`）被`web`服务使用，以及为单个服务（`db`服务下的第一个路径`volumes`）定义的绑定安装。该`db`服务还使用名为`dbdata`（`db`服务中的第二个路径`volumes`）的命名卷，但使用旧的字符串格式定义它以安装命名卷。必须在顶级`volumes`键下列出命名卷，如图所示。

```javascript
version: "3.2"
services:
  web:
    image: nginx:alpine
    volumes:
      - type: volume
        source: mydata
        target: /data
        volume:
          nocopy: true
      - type: bind
        source: ./static
        target: /opt/app/static

  db:
    image: postgres:latest
    volumes:
      - "/var/run/postgres/postgres.sock:/var/run/postgres/postgres.sock"
      - "dbdata:/var/lib/postgresql/data"

volumes:
  mydata:
  dbdata:
```

>  **注意**：有关卷的一般信息，请参阅使用卷和卷插件。  

#### Short syntax 短语法

（可选）指定主机（`HOST:CONTAINER`）上的路径或访问模式（`HOST:CONTAINER:ro`）。

​	可以在主机上**挂载相对路径**，该路径将相对于正在使用的**Compose配置文件**的目录进行扩展。相对路径应始终以`.`或=`..`开头。

```yml
volumes:
  # Just specify a path and let the Engine create a volume
  # 仅仅指定一个path，让机器创建volume
  - /var/lib/mysql
  # 指定一个绝对路径映射 Specify an absolute path mapping 
  - /opt/data:/var/lib/mysql
  # 主机上的相对路径 Path on the host, relative to the Compose file
  - ./cache:/tmp/cache
  # User-relative path
  - ~/configs:/etc/configs/:ro
  # Named volume
  - datavolume:/var/lib/mysql
```

#### Long syntax 长语法

​	长格式语法允许配置简短形式无法扩展的其他字段。

- `type`：安装类型`volume`或`bind` 

- `source`：mount 的源，**主机上**用于绑定装载的路径，或顶级`volumes`键中定义的卷的名称

- `target`：**容器中**将安装卷的路径

- `read_only`：flag将卷设置为只读

- `bind`  ：配置其他绑定选项 

  - `propagation`：用于绑定的传播模式

- `volume`  ：配置其他卷选项 

  - `nocopy`：flag用于在创建卷时禁用从容器复制数据

- `tmpfs`: configure additional tmpfs options
  - `size`: the size for the tmpfs mount in bytes
- `consistency`: the consistency requirements of the mount, one of consistent (host and container have identical view), cached (read cache, host view is authoritative) or delegated (read-write cache, container’s view is authoritative)

```yml
version: "3.7"
services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - type: volume
        source: mydata
        target: /data
        volume:
          nocopy: true
      - type: bind
        source: ./static
        target: /opt/app/static

networks:
  webnet:

volumes:
  mydata:

```

>  **注意：**长语法是v3.2中的新增功能  

#### 服务，群组和堆栈文件的卷

使用服务，群组和`docker-stack.yml`文件时，请记住，支持服务的任务（容器）可以部署在群中的任何节点上，每次更新服务时，这些节点可能是不同的节点。

如果没有指定具有指定源的卷，Docker 会为支持服务的每个任务创建一个匿名卷。删除关联的容器后，匿名卷不会保留。

如果希望数据保持不变，请使用可识别多主机的命名卷和卷驱动程序，以便可以从任何节点访问数据。或者，在服务上设置约束，以便将其任务部署在具有卷的节点上。

例如，[Docker Labs中](https://github.com/docker/labs/blob/master/beginner/chapters/votingapp/)`docker-stack.yml`的[votingapp 示例](https://github.com/docker/labs/blob/master/beginner/chapters/votingapp/)文件定义了一个`db`运行`postgres`数据库的服务。它被配置为命名卷，以便将数据持久保存在 swarm 上，*并且*仅限于在`manager`节点上运行。以下是该文件中的相关剪辑：

```javascript
version: "3"
services:
  db:
    image: postgres:9.4
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend
    deploy:
      placement:
        constraints: [node.role == manager]
```

#### 卷装入的缓存选项（适用于Mac的Docker）

在Docker 17.04 CE Edge 及更高版本（包括17.06 CE Edge和Stable）上，您可以在 Compose 文件中为绑定装载目录配置容器和主机一致性要求，以便在读取/写入卷装入时获得更好的性能。这些选项可解决特定于`osxfs`文件共享的问题，因此仅适用于 Docker for Mac。

标志是：

- `consistent`：完全一致。容器运行时和主机始终保持相同的安装视图。这是默认值。
- `cached`：主机的 mount 视图是权威的。在主机上进行的更新在容器中可见之前可能会有延迟。
- `delegated`：容器运行时的 mount 视图是权威的。在容器中的更新在主机上可见之前可能会有延迟。

以下是将卷配置为的示例`cached`：

```javascript
version: '3'
services:
  php:
    image: php:7.1-fpm
    ports:
      - 9000
    volumes:
      - .:/var/www/project:cached
```

关于这些标志的全部细节，它们解决的问题以及它们的`docker run`对应物是在Docker for Mac 主题中[对卷安装进行性能调整（共享文件系统）](https://docs.docker.com/docker-for-mac/osxfs-caching/)。

### 重新开始

`no`是默认的重启策略，在任何情况下都不会重启容器。当`always`指定时，容器总是重新启动。该`on-failure`如果退出代码指示的故障错误政策重启的容器。

```javascript
restart: "no"
restart: always
restart: on-failure
restart: unless-stopped
```

### domainname, 

### hostname,

###  ipc, 

### mac_address, 

### privileged, 

### read_only, 

### shm_size, 

### stdin_open,

###  tty, 

### user, 

### working_dir

每个都是单个值，类似于 docker run 对应物。

```javascript
user: postgresql
working_dir: /code

domainname: foo.com
hostname: foo
ipc: host
mac_address: 02:42:ac:11:65:43

privileged: true


read_only: true
shm_size: 64M
stdin_open: true
tty: true
```

## 指定持续时间

某些配置选项（例如`interval`和`timeout`子选项`healthcheck`）接受持续时间作为字符串，格式如下所示：

```javascript
2.5s
10s
1m30s
2h32m
5h34m56s
```

支持的单位是`us`，`ms`，`s`，`m`和`h`。

## Volume configuration reference 卷配置参考

​	虽然可以在文件上**作为服务声明**的一部分声明卷，但本部分允许您**创建命名卷**（不依赖于`volumes_from`）可以跨越多个服务中重用。并且可以使用 docker **命令行或API** 轻松**回收和检查**Volume。有关更多信息，请参阅 [docker volume](https://docs.docker.com/engine/reference/commandline/volume_create/) 子命令文档。

​	有关卷的一般信息，请参阅使用卷 [Use volumes](https://docs.docker.com/engine/admin/volumes/volumes/)和卷插件[Volume Plugins](https://docs.docker.com/engine/extend/plugins_volume/) 。

以下是双服务设置的示例，其中数据库的数据目录与另一个服务作为卷共享，以便可以定期备份：

```yml
version: "3"

services:
  db:
    image: db
    volumes:
      - data-volume:/var/lib/db
  backup:
    image: backup-service
    volumes:
      - data-volume:/var/lib/backup/data

volumes:
  data-volume:
```

顶级`volumes`键下的条目可以为空，在这种情况下，它将使用引擎配置的默认驱动程序（在大多数情况下，这是`local`驱动程序）。（可选）您可以使用以下键进行配置：

### driver 驱动

指定应为此卷使用哪个卷驱动程序。默认为 Docker Engine 配置使用的任何驱动程序，在大多数情况下是`local`。如果驱动程序不可用，则在`docker-compose up`尝试创建卷时，Engine 将返回错误。

```javascript
 driver: foobar
```

### driver_opts

将选项列表指定为键值对，以传递给此卷的驱动程序。这些选项取决于驱动程序 - 请参阅驱动程序的文档以获取更多信息。可选的。

```javascript
 driver_opts:
   foo: "bar"
   baz: 1
```

### external 外部

​	如果设置为`true`，则指定已在 Compose 之外创建此卷。`docker-compose up`不会尝试创建它，如果它不存在则会引发错误。

​	对于版本3.3及更低版本的格式，`external`不能与其他卷配置键（`driver`，`driver_opts`）一起使用。版本3.4及更高版本不再存在此限制。

​	在下面的示例中，`[projectname]_data`Compose 不会尝试创建一个被调用的卷，而是寻找一个简单调用的现有卷，`data`并将其挂载到`db`服务的容器中。

```javascript
version: '2'

services:
  db:
    image: postgres
    volumes:
      - data:/var/lib/postgresql/data

volumes:
  data:
    external: true
```

您还可以在 Compose 文件中与用于引用它的名称分别指定卷的名称：

```javascript
volumes:
  data:
    external:
      name: actual-name-of-volume
```

> 始终使用 docker stack deploy 创建外部卷如果使用 docker stack deploy 以群集模式启动应用程序（而不是 docker compose up），*则将创建*不存在的外部卷。在群集模式下，当服务定义卷时，会自动创建卷。由于服务任务是在新节点上调度的，因此[swarmkit](https://github.com/docker/swarmkit/blob/master/README/)会在本地节点上创建卷。要了解更多信息，请参阅[moby / moby＃29976](https://github.com/moby/moby/issues/29976)。  

### labels 标签

使用 Docker 标签向容器添加元数据。您可以使用数组或字典。

建议您使用反向 DNS 表示法来防止标签与其他软件使用的标签冲突。

```javascript
labels:
  com.example.description: "Database volume"
  com.example.department: "IT/Ops"
  com.example.label-with-empty-value: ""

labels:
  - "com.example.description=Database volume"
  - "com.example.department=IT/Ops"
  - "com.example.label-with-empty-value"
```

## 网络配置参考

顶级`networks`键允许您指定要创建的网络。

- 有关 Compose 使用 Docker 网络功能和所有网络驱动程序选项的完整说明，请参阅网络指南。
- 有关网络的[Docker Labs](https://github.com/docker/labs/blob/master/README/)教程，请从[设计可扩展的便携式Docker容器网络开始](https://github.com/docker/labs/blob/master/networking/README/)

### 驱动

指定应该为此网络使用哪个驱动程序。

默认驱动程序取决于您正在使用的 Docker Engine 的配置方式，但在大多数情况下，它将`bridge`位于单个主机和`overlay`Swarm上。

如果驱动程序不可用，Docker Engine 将返回错误。

```javascript
driver: overlay
```

#### bridge

Docker默认使用`bridge`单个主机上的网络。有关如何使用桥接网络的示例，请参阅有关[Bridge网络](https://github.com/docker/labs/blob/master/networking/A2-bridge-networking/)的Docker Labs教程。

#### 覆盖

该`overlay`驱动程序创建一个群跨多个节点命名的网络。

- 有关如何`overlay`在群集模式下使用服务构建和使用网络的工作示例，请参阅有关[覆盖网络和服务发现](https://github.com/docker/labs/blob/master/networking/A3-overlay-networking/)的Docker Labs教程。
- 要深入了解它的工作原理，请参阅[Overlay Driver Network Architecture](https://github.com/docker/labs/blob/master/networking/concepts/06-overlay-networks/)上的网络概念实验。

### driver_opts

将选项列表指定为键值对，以传递给此网络的驱动程序。这些选项取决于驱动程序 - 请参阅驱动程序的文档以获取更多信息。可选的。

```javascript
  driver_opts:
    foo: "bar"
    baz: 1
```

### enable_ipv6

在此网络上启用IPv6网络。

### ipam

指定自定义 IPAM 配置。这是一个具有多个属性的对象，每个属性都是可选的：

- `driver`：自定义IPAM驱动程序，而不是默认值。

- ```
  config
  ```

  ：包含零个或多个配置块的列表，每个配置块包含以下任意键： 

  - `subnet`：CIDR格式的子网，表示网段

一个完整的例子：

```javascript
ipam:
  driver: default
  config:
    - subnet: 172.28.0.0/16
```

>  **注意**：其他IPAM配置（例如）`gateway`仅适用于版本2。  

### 内部

默认情况下，Docker 还将桥接网络连接到它以提供外部连接。如果要创建外部隔离的覆盖网络，可以将此选项设置为`true`。

### 标签

使用 Docker 标签向容器添加元数据。您可以使用数组或字典。

建议您使用反向 DNS 表示法来防止标签与其他软件使用的标签冲突。

```javascript
labels:
  com.example.description: "Financial transaction network"
  com.example.department: "Finance"
  com.example.label-with-empty-value: ""

labels:
  - "com.example.description=Financial transaction network"
  - "com.example.department=Finance"
  - "com.example.label-with-empty-value"
```

### 外部

如果设置为`true`，则指定此网络已在 Compose 之外创建。`docker-compose up`不会尝试创建它，如果它不存在则会引发错误。

`external`可以不与其它网络配置键（结合使用`driver`，`driver_opts`，`ipam`，`internal`）。

在下面的示例中，`proxy`是通往外部世界的门户。而不是尝试创建一个被调用的网络`[projectname]_outside`，Compose 将寻找一个简单调用的现有网络`outside`，并将`proxy`服务的容器连接到它。

```javascript
version: '2'

services:
  proxy:
    build: ./proxy
    networks:
      - outside
      - default
  app:
    build: ./app
    networks:
      - default

networks:
  outside:
    external: true
```

您还可以在 Compose 文件中与用于引用它的名称分开指定网络名称：

```javascript
networks:
  outside:
    external:
      name: actual-name-of-network
```

## 配置参考

顶级`configs`声明定义或引用可以授予此堆栈中的服务的配置。配置的来源是`file`或`external`。

- `file`：使用指定路径上的文件内容创建配置。
- `external`：如果设置为 true，则指定已创建此配置。Docker 不会尝试创建它，如果它不存在，`config not found`则会发生错误。

在此示例中，`my_first_config`将创建（如`<stack_name>_my_first_config)`部署堆栈时，并且`my_second_config`已存在于 Docker 中）。

```javascript
configs:
  my_first_config:
    file: ./config_data
  my_second_config:
    external: true
```

外部配置的另一个变体是 Docker 中的配置名称与服务中存在的名称不同。以下示例修改前一个示例以使用调用的外部配置`redis_config`。

```javascript
configs:
  my_first_config:
    file: ./config_data
  my_second_config:
    external:
      name: redis_config
```

您仍然需要为堆栈中的每个服务授予对配置的访问权限。

## 秘密配置参考

顶级`secrets`声明定义或引用可以授予此堆栈中的服务的机密。秘密的来源是`file`或`external`。

- `file`：使用指定路径上的文件内容创建密码。
- `external`：如果设置为 true，则指定已创建此密钥。Docker 不会尝试创建它，如果它不存在，`secret not found`则会发生错误。

在此示例中，`my_first_secret`将创建（如`<stack_name>_my_first_secret)`部署堆栈时，并且`my_second_secret`已存在于 Docker 中）。

```javascript
secrets:
  my_first_secret:
    file: ./secret_data
  my_second_secret:
    external: true
```

外部机密的另一个变体是 Docker 中的机密名称与服务中存在的名称不同。以下示例修改前一个示例以使用调用的外部机密`redis_secret`。

```javascript
secrets:
  my_first_secret:
    file: ./secret_data
  my_second_secret:
    external:
      name: redis_secret
```

您仍然需要授予对堆栈中每个服务的秘密的访问权限。

## 变量替换

您的配置选项可以包含环境变量。Compose 使用`docker-compose`运行的 shell 环境中的变量值。例如，假设shell包含`POSTGRES_VERSION=9.3`并提供此配置：

```javascript
db:
  image: "postgres:${POSTGRES_VERSION}"
```

`docker-compose up`使用此配置运行时，Compose 会`POSTGRES_VERSION`在 shell 中查找环境变量并将其值替换为。对于此示例，Compose `image`会`postgres:9.3`在运行配置之前解析to 。

如果未设置环境变量，请使用空字符串 Compose 替换。在上面的示例中，如果`POSTGRES_VERSION`未设置，则`image`选项的值为`postgres:`。

您可以使用`.env`文件为环境变量设置默认值，Compose 将自动查找该文件。shell 环境中设置的值将覆盖`.env`文件中设置的值。

支持两者`$VARIABLE`和`${VARIABLE}`语法。此外，使用2.1文件格式时，可以使用典型的 shell 语法提供内联默认值：

- `${VARIABLE:-default}`将评估环境中`default`是否`VARIABLE`未设置或为空。
- `${VARIABLE-default}default`只有`VARIABLE`在环境中未设置时才会评估。

`${VARIABLE/foo/bar}`不支持其他扩展的 shell 样式功能，例如。

`$$`当配置需要文字美元符号时，您可以使用（双美元符号）。这也可以防止 Compose 插值，因此`$$`允许您引用不希望由 Compose 处理的环境变量。

```javascript
web:
  build: .
  command: "$$VAR_NOT_INTERPOLATED_BY_COMPOSE"
```

如果您忘记并使用单个美元符号（`$`），Compose 会将该值解释为环境变量，并会警告您：

未设置VAR_NOT_INTERPOLATED_BY_COMPOSE。替换空字符串。

## 撰写文档

- User guide
- Installing Compose
- Compose file versions and upgrading
- Get started with Docker
- [Samples](https://docs.docker.com/samples/)
- Command line reference

[纠错](javascript:;)

[fig](https://docs.docker.com/glossary/?term=fig), [composition](https://docs.docker.com/glossary/?term=composition), [compose](https://docs.docker.com/glossary/?term=compose), [docker](https://docs.docker.com/glossary/?term=docker)