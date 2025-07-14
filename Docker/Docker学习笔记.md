# Docker学习笔记

---

## docker安装（Ubuntu）

1. 更新软件源，以便安装docker所需的依赖包

2. 安装依赖包
   
   ```bash
   sudo apt-get install \
       apt-transport-https \
       ca-certificates \
       curl \
       gnupg \
       lsb-release
   ```

3. 添加docker官方的软件源（有代理的话不会太慢）
   
   ```bash
   echo \
     "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
     $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   # 实在慢可以用中科大的源：https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu
   ```

4. 添加docker源的gpg密钥
   
   ```bash
   # 新方法：把密钥添加到/etc/apt/trusted.gpg.d/目录的某个文件中
   sudo curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/docker.gpg
   # 旧方法：把密钥添加到/etc/apt/trusted.gpg中（我尝试的时候这样会报错）
   sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
   ```

5. 再次更新软件源，以获取docker包的下载信息

6. 下载docker
   
   ```bash
   # 需要下载3个组件（最新版本的docker-ce中不再包含后两个组件）
   sudo apt-get install docker-ce docker-ce-cli containerd.io
   
   ### 可以直接下载docker.io
   sudo apt-get install -y docker.io
   ```

`docker-ce`, `docker-ce-cli`, 和 `containerd.io` 这三个包分别包含以下内容：

1. `docker-ce`：这是Docker社区版（Community Edition）的核心组件，包括Docker的守护进程、API、以及其他核心功能。这个守护进程负责管理Docker容器，包括创建、启动、停止容器等。

2. `docker-ce-cli`：这是Docker的命令行接口，提供了一系列的命令，使用户可以通过命令行来操作Docker，例如创建、启动、停止容器，拉取和推送镜像等。

3. `containerd.io`：这是Docker使用的**容器运行时**。容器运行时是一种软件，它负责在操作系统上运行和管理容器。`containerd.io` 是一个开源的容器运行时，它可以独立于Docker使用，也可以作为Docker的一个组件使用。

4. **docker.io**：
   - **包含内容**：Docker 引擎、CLI 和其他必要的工具，类似于 `docker-ce`、`docker-ce-cli` 和 `containerd.io`。
   - **版本**：通常不是最新的版本，而是软件库中认为稳定和可靠的版本。
   - **来源**：来自 Debian 或 Ubuntu 的官方软件库。


## docker镜像的创建

### 1. commit方法

这种方法是将正在运行的容器保存为一个image

一般不推荐这种方式：

- 手工创建容易出错，且效率低重复性弱

- 隐藏了镜像创建过程，有安全隐患。

```bash
# 在另一个终端中执行以下命令
docker commit <running-image-tag> <new-image-tag>
```

### 2. Dockerfile方法

把创建镜像的过程一步步写到文件`Dockerfile`中，再通过`docker build`命令创建镜像

#### build context（创建上下文）

        `Dockerfile`文件所在的目录，在build的时候，CLI会把这个目录打包发送给daemon，daemon解包后找到`Dockerfile`，再据此创建镜像

        所有需要加入到镜像中的文件 or 创建镜像时需要用到的文件都需要放到这个build context目录下（ADD, COPY等命令），因为daemon创建镜像时可见的文件就只有这里面的文件

#### build命令

所有的镜像都没有名字，只有标签，标签用于唯一标识一个镜像（也可以当作名字来看）

```bash
docker build -t <image-tag-name> <build-context-path>
```

#### 镜像的分层结构 & 镜像层的缓存特性

build命令创建镜像的过程如下：

1. 从base镜像运行一个容器

2. 执行一条指令对容器做修改

3. 在底层执行类似docker commit的操作，生成一个新的镜像层

4. docker再基于刚形成的这层镜像运行一个新容器

5. 重复2~4步，直到dockerfile中的所有指令都执行完毕

由此可见，镜像的创建过程中每执行一条指令就会形成一个镜像层，最后层层叠加得到最终镜像。这个过程中的每一个镜像层都基于它下面的所有镜像层。

**分层结构带来的缓存特性：**

        docker会缓存创建镜像过程中的镜像层，当在创建新镜像时，如果某镜像层已经存在，就无需创建，可以直接使用。例如下面例子：

先创建一个镜像：

```dockerfile
FROM ubuntu
RUN apt-get update && apt-get install -y vim
RUN apt-get install -y net-tools
```

在通过另一个dockerfile创建新的镜像：

```dockerfile
FROM ubuntu
RUN apt-get update && apt-get install -y vim
RUN apt-get install -y net-tools
ENTRYPOINT echo "welcome, here is jchan's docker"
```

构建镜像时直接使用缓存的镜像层（**CACHED**）：

```
root@MagicBook-14-JC:/home/jchan/Edge-Computing/docker-learning/docker2# docker build -t ubuntu-welcome .
[+] Building 2.2s (7/7) FINISHED                                                                 docker:default
 => [internal] load build definition from Dockerfile                                                       0.0s
 => => transferring dockerfile: 179B                                                                       0.0s
 => [internal] load metadata for docker.io/library/ubuntu:latest                                           2.0s
 => [internal] load .dockerignore                                                                          0.0s
 => => transferring context: 2B                                                                            0.0s
 => [1/3] FROM docker.io/library/ubuntu:latest@sha256:77906da86b60585ce12215807090eb327e7386c8fafb5402369  0.0s
 => CACHED [2/3] RUN apt-get update && apt-get install -y vim                                              0.0s
 => CACHED [3/3] RUN apt-get install -y net-tools                                                          0.0s
 => exporting to image                                                                                     0.0s
 => => exporting layers                                                                                    0.0s
 => => writing image sha256:43a8f2b8caf4e1415164715fc755e31fadf2f90b7cba5e57fde9236bf035b5ae               0.0s
 => => naming to docker.io/library/ubuntu-welcome
```

当然，这种情况完全可以直接在前一个image的基础上编写dockerfile（FROM）

### 3. dockerfile中的常用指令

1. `FROM`：指定基础镜像。例如，`FROM ubuntu:18.04`。

2. `RUN`：在镜像构建过程中执行命令，主要用于安装软件包、设置环境变量等。

3. `CMD`：指定容器启动后默认执行的命令和参数。

4. `ENTRYPOINT`：指定容器启动后执行的命令，但是它指定的命令不会被覆盖，除非在运行容器时使用了 `--entrypoint` 选项。

5. `COPY`：将文件或目录从构建上下文复制到正在构建的镜像的文件系统中。

6. `ADD`：与`COPY`类似，但是如果源文件是一个URL或者tar文件，`ADD`会自动下载或解压。

7. `EXPOSE`：声明容器运行时提供的网络端口。

8. `WORKDIR`：设置后续指令的工作目录。

9. `ENV`：设置环境变量。

10. `ARG`：定义构建参数。

11. `VOLUME`：声明持久化存储卷。

12. `USER`：设置后续指令的执行用户。

**WORKDIR：更改工作目录**

- 如果目录不存在，docker会自动创建

- 后续RUN, ADD, COPY等命令的当前目录就是WORKDIR指定的工作目录

- 容器启动时，进入的目录也是这个工作目录

**RUN, CMD, ENTRYPOINT区别**

- 作用
  
  1. `RUN`：RUN 指令用于在镜像构建过程中执行命令，主要用于安装软件包、设置环境变量等。每个 RUN 指令都会创建一个新的镜像层，RUN 指令执行的结果会被包含在这个层中。例如，`RUN apt-get update && apt-get install -y curl` 会在镜像中安装 curl。
  
  2. `CMD`：CMD 指令用于指定容器启动后默认执行的命令和参数。如果在运行容器时指定了命令，那么 CMD 指令指定的命令会被覆盖。一个 Dockerfile 中只能有一个 CMD 指令，如果有多个，那么只有最后一个会生效。例如，`CMD ["echo", "Hello, World!"]` 会在容器启动后输出 "Hello, World!"。
  
  3. `ENTRYPOINT`：ENTRYPOINT 指令也用于指定容器启动后执行的命令，但是它指定的命令不会被覆盖，除非在运行容器时使用了 `--entrypoint` 选项。如果 Dockerfile 中同时指定了 ENTRYPOINT 和 CMD，那么 CMD 指令的内容会作为 ENTRYPOINT 指令的参数。例如，`ENTRYPOINT ["echo"]` 和 `CMD ["Hello, World!"]` 会在容器启动后输出 "Hello, World!"。

- 使用场景
  
  1. `RUN`：一层一层构建镜像时使用
  
  2. `CMD`：设置容器启动时的默认行为（默认参数 or 默认命令），一般不单独使用
  
  3. `ENTRYPOINT`：设置容器启动时的行为，通常是启动某个应用或服务（使得启动容器的行为看起来像启动服务）
     
     - exec格式：可以从CMD获取默认**额外**参数 or 从docker run获取动态**额外**参数
     
     - shell格式：忽略CMD和docker run提供的任何参数

## docker镜像的使用

### 镜像的tag标签

![](..\images\2024-04-12-10-37-13-image.png)

        每个镜像由`REPOSITORY:TAG`唯一标识，通常REPOSITORY是应用/软件的名字（也是docekr build -t 所指定的名字），TAG是版本。在docker build时可以通过`-t REPOSITORY:TAG`直接指定仓库名和标签，如果没有指定标签，则会**默认`latest`标签**。

        之所以使用`REPOSITORY`这个词来表示软件名，是因为通常一个软件会有多个版本，而这众多版本的集合就是一个仓库了。当版本有较大变动时，也可能会取一个新的仓库名。

#### docker tag命令

```bash
# 给SOURCE镜像创建指定标签
docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
```

tag标签本质上是对特定镜像的引用，而docker tag命令执行过后通过docker images会发现多了一个镜像，但实际上并没有创建新的镜像，只是创建了一个旧的镜像的引用。如下所示，镜像只有一个副本，但是有两个引用。

![](..\images\2024-04-12-10-53-12-image.png)

#### 最佳实践

**方案一**

```bash
# 应用的第一个版本
docker build -t my_app .
# 创建标签1.0标识版本 -> latest和1.0都指向第一个版本的镜像
docker tag my_app my_app:1.0

# 版本更新中...

# 构建新版本 -> latest指向的镜像从旧版本迁移到新版本
docker build -t my_app .
# 创建新版本的标签1.1 -> latest和1.1指向新版本，1.0指向旧版本
docker tag my_app my_app:1.1
```

**方案二**

- my_app:latest -> 软件的最新版本

- my_app:1 -> 软件1.x系列版本中的最新版本

- my_app:1.3 -> 指定的1.3版本

当版本更新时，新版本的镜像名字是什么我不关心，只需要在构建镜像之后，更新my_app相关的一系列标签。使得可以统一地在这里通过对my_app进行版本命名管理。

```bash
# 构建应用的1.0版本
docker build -t my_app_v1.0 .
# 创建版本标识  my_app:latest -> my_app_v1.0 
docker tag my_app_v1.0 my_app:latest
# 创建版本标识  my_app:1 -> my_app_v1.0 
docker tag my_app_v1.0 my_app:1
# 创建版本标识  my_app:1.0 -> my_app_v1.0 
docker tag my_app_v1.0 my_app:1.0

# 版本更新中......

# 构建应用的1.1版本
docker build -t my_app_v1.1 .
# 更改my_app:latest的指向 -> my_app_v1.1 
docker tag my_app_v1.1 my_app:latest
# 更改my_app:1的指向 -> my_app_v1.1 
docker tag my_app_v1.1 my_app:1
# 创建版本标识  my_app:1.1 -> my_app_v1.1 
docker tag my_app_v1.1 my_app:1.1
```

## 容器

### 容器的运行

#### run启动命令

容器启动后有三种方式可以标识一个容器：

1. 长ID，如下图所示

2. 短ID，长ID的前12位，通过docker ps可以看到

3. 名字，可以通过`docker run --name <name>`显式指定

![](..\images\2024-04-12-14-22-09-image.png)

**三种启动的方式：**

1. `docker run my-image`：前台运行容器，占用**当前终端用于打印容器的输出**，且容器只有标准输出映射到了外面的终端上，也就是并不能在当前终端进行输入/不能交互。（也就是说，就算想ctrl+c终止容器中运行的程序都不行，因为当前终端根本就没有到容器的标准输入，只能另开一个终端使用run stop来终止容器）

2. `docker run -d my-image`：后台运行容器

3. `docker run -it my-image`：打开容器标准输入的映射，使得可以在当前终端向容器进行输入，同时把容器中运行命令的终端映射到当前终端上，使得可以在当前终端与容器上运行的程序进行交互。通常在容器启动时执行的命令是某交互式程序时（例如echoclient或者shell），使用该方式。

#### 容器启动时的执行命令

有三种方式指定容器启动时执行的命令：

<img src="file:///C:/Users/honor/AppData/Roaming/marktext/images/2024-04-12-14-10-40-image.png" title="" alt="" width="260">

但是要注意的是，当指定的命令执行完时，容器就会退出。例如下图所示，我只通过ENTRYPOINT指定在容器启动时输出欢迎语句，所以run这个容器是一瞬间就执行完。

![](..\images\2024-04-12-14-13-06-image.png)

所以如果想让容器一直运行，就需要容器启动时执行一个长久运行的程序，例如一个web server，加上-d选项可以让其在后台运行。

#### 二次进入容器的方式

**attach进入后台运行的容器**

`docker attach <container>`：把当前终端attach到容器执行启动命令的终端

**exec在容器开一个新进程**

docker exec类似于docker run，只不过run是启动一个镜像并执行命令，而exec是在容器中启动一个进程并执行命令。

`docker exec -it <container> bash`：在容器中开一个新进程，并执行bash命令，提供一个交互式终端。通常通过该方式检查正在运行的容器的状态。

`docker exec -d <container> ./proxyserv`：在容器中开一个新进程，并后台启动proxyserv服务。可以用该方式在容器中开启额外的服务。

### 容器的终止、暂停、重启和删除

- stop终止：exit容器中正在运行的程序，容器状态变为Exited，容器的相关资源依旧保留，可以通过`docker start`重新启动

- pause/unpause：暂停容器的运行

- restart：重启容器，相当于`docker stop` +  `docker start`。如果希望容器在意外出错而停止运行时自动重启，可以在启动容器时指定`docker run -d --restart=always <image>`

- rm：回收exited的容器资源，彻底销毁容器。删除所有已经退出的容器`docker rm $(docker ps -a -q -f status=exited)`

#### 容器的状态转换

从中可以看出，容器可以先创建后启动，也就是把docker run的动作拆分成两步

![](..\images\2024-04-12-17-27-54-image.png)



## 容器网络

### none网络

容器中除了lo没有其他网卡，也就是说容器不能与其他机器通信。适用于需要网络隔离、安全性要求高的地方。

### host网络

容器共享docker host的网络栈，容器的网络配置和host一模一样。好处就是性能高使用方便。坏处是要考虑端口冲突的问题。

### bridge网络

**Bridge网络**：Bridge在Docker环境中可以被视为一个类似于交换机的虚拟网络设备，它连接了两个网络接口：一个是主机上的虚拟网络接口（通常是docker0），另一个是容器内部的虚拟网络接口。

1. 提供网络接口：bridge在主机上创建一个虚拟网络接口，通常名为docker0。这个接口在主机上表现为一个网络设备，就像一个真实的网卡一样。

2. 分配IP地址：当一个容器被创建并连接到bridge时，bridge会从其IP地址池中分配一个IP地址给这个容器。这个IP地址被分配给容器内的网络接口，使容器能够通过这个接口访问网络。IP地址的分配通常由Docker的内置DHCP（动态主机配置协议）服务器处理。当一个新的容器启动并连接到bridge网络时，Docker的DHCP服务器会从预定义的IP地址池中选择一个未被使用的IP地址，并分配给这个新的容器。
   
   默认情况下，Docker的bridge网络使用172.17.0.0/16的私有IP地址范围，其中172.17.0.1通常被分配给bridge网络的网关（即主机），而172.17.0.2开始的IP地址则被分配给连接到bridge网络的容器。

3. 路由和转发：bridge负责在主机和容器之间路由和转发网络流量。当主机需要发送数据包给容器时，数据包会被发送到bridge，然后bridge会根据数据包的目标IP地址将数据包转发给相应的容器。反过来，当容器需要发送数据包给主机或其他容器时，数据包也会被发送到bridge，然后由bridge进行转发。

4. 端口映射（NAT）：bridge还支持端口映射功能，也就是将容器内的端口映射到主机的端口。这使得外部网络能够通过主机的端口访问到容器的服务。

<img src="file:///C:/Users/honor/AppData/Roaming/marktext/images/2024-04-14-12-01-31-image.png" title="" alt="" width="291">

**容器与外部网络的通信：** 通过bridge网络，容器和主机构成了一个小的私有网络，主机就是这个私有网络的网关，这个私有网络中的所有机器想要和外部通信都需要通过主机这个网关，这时候就发生了NAT（网络地址转换）。

**多容器环境 & Bridge网络拓扑**

 一般来说当谈论容器的bridge网络时，这个bridge网络一般指的是docker在主机上创建的用于桥接容器网络的虚拟网卡（docker0）。如果想要让多个容器连接到一个bridge网络中，只需要将每个容器桥接到同一个虚拟网卡，所以通常用虚拟网卡来代指bridge网络，这也就是前面提到的：“通过bridge网络，容器和主机构成了一个小的私有网络”。同一个bridge网络中的容器之间可以相互通信，而不同bridge网络之间的容器不能通信，因为所处不同的私有网络。

![](..\images\2024-04-14-13-44-23-image.png)

### user-defined网络

前三种网络是docker定义好的网络，用户可以根据需要创建自定义的网络。docker提供三种user-defined网络驱动：bridge、overlay和macvlan。后两种用于创建跨主机的网络。

**自定义bridge网络**

用户定义的bridge网络提供了比默认的bridge网络更多的功能，例如网络名称和服务发现。在用户定义的bridge网络中，你可以使用容器的名字作为主机名来进行网络通信，Docker会自动解析这个名字到容器的IP地址。

**1. 创建用户定义的bridge网络**

你可以使用`docker network create`命令来创建一个新的bridge网络。例如，以下命令会创建一个名为`my_bridge_network`的bridge网络：

<BASH>

`docker network create --driver bridge my_bridge_network`

这个命令使用`--driver bridge`选项来指定网络的类型为bridge。如果你不提供`--driver`选项，Docker会默认创建一个bridge网络。Docker会自动为这个网络分配一个默认的IP地址范围，类似于docker0的 172.17.0.0/16。

**2. 启动容器并连接到bridge网络**

然后，你可以在运行容器时使用`--network`选项来连接到你刚才创建的网络。例如，以下命令会启动一个新的容器，并将它连接到`my_bridge_network`网络：

<BASH>

`docker run --network=my_bridge_network -d my-image`

**3. 断开和删除网络**

如果你不再需要这个网络，你可以使用`docker network rm`命令来删除它：

<BASH>

`docker network rm my_bridge_network`

请注意，你需要先停止并断开所有连接到这个网络的容器，才能删除这个网络。

**4. 使用指定的ip网段和ip**

使用`--subnet`和`--gateway`选项来自定义这个网络的IP地址范围和网关

<BASH>

```bash
docker network create --driver bridge --subnet=192.168.0.0/16 --gateway=192.168.0.1 my_bridge_network
docker run --network=my_bridge_network --ip=192.168.0.5 -d my-image
```

在这个网络中，Docker会自动为每个连接到这个网络的容器分配一个IP地址。这个IP地址会在`192.168.0.0/16`范围内，但不会是`192.168.0.1`，因为这个地址已经被用作网关。

你也可以在运行容器时使用`--ip`选项来为容器指定一个静态IP地址。例如，以下命令会启动一个新的容器，并将它连接到`my_bridge_network`网络。

**NOTE：** 只有使用了`--subnet`指定网段的网络，才可以用`--ip`指定容器的静态ip。



## 容器存储

**Bind Mounts**：当你使用bind mounts时，你是在将宿主机上已经存在的文件或目录挂载到容器中。这意味着容器可以访问和修改这些文件或目录，同时这些修改也会反映到宿主机上。这可以很方便得将host上的数据共享到容器中。

**Docker Managed Volumes**：当你使用Docker managed volumes（容器启动-v）时，Docker会在宿主机上创建一个新的目录/volume用于存储数据。如果容器的挂载点目录不为空，Docker会将容器中挂载点的内容复制到这个新的volume中。如果想要将host上的数据共享到容器中，就需要手动将数据复制到host的volume目录中。



因此，如果你需要在容器中访问宿主机上已经存在的文件或目录，你应该使用bind mounts。如果你需要一个由Docker管理的持久化存储（保存容器运行中产生的数据），你应该使用Docker managed volumes。


