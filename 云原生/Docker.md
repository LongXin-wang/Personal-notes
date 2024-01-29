- [Docker 核心概念](#docker-核心概念)
  - [Docker重要组件](#docker重要组件)
- [镜像使用](#镜像使用)
  - [Dockerfile](#dockerfile)
    - [多阶段构建](#多阶段构建)
- [镜像实现原理](#镜像实现原理)
  - [镜像结构](#镜像结构)
    - [镜像的基础架构](#镜像的基础架构)
- [容器操作](#容器操作)
  - [容器监控](#容器监控)
- [资源](#资源)
  - [资源隔离](#资源隔离)
    - [NameSpace](#namespace)
  - [资源限制](#资源限制)
    - [cgroups](#cgroups)

 # Docker 核心概念

镜像是容器的基石，容器是由镜像创建的。一个镜像可以创建多个容器，容器是镜像运行的实体。仓库就非常好理解了，就是用来存放和分发镜像的。
![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202401241025701.png)

## Docker重要组件

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202401241026255.png)


事实上，dockerd 启动的时候， containerd 就随之启动了，dockerd 与 containerd 一直存在。当执行 docker run 命令（通过 busybox 镜像创建并启动容器）时，containerd 会创建 containerd-shim 充当 “垫片” 进程，然后启动容器的真正进程 sleep 3600 。这个过程和架构图是完全一致的。  
```Bash
$ docker run -d busybox sleep 3600

$ sudo ps aux |grep dockerd
root      4147  0.3  0.2 1447892 83236 ?       Ssl  Jul09 245:59 /usr/bin/dockerd

$ sudo pstree -l -a -A 4147
dockerd
  |-containerd --config /var/run/docker/containerd/containerd.toml --log-level info
  |   |-containerd-shim -namespace moby -workdir /var/lib/docker/containerd/daemon/io.containerd.runtime.v1.linux/moby/d14d20507073e5743e607efd616571c834f1a914f903db6279b8de4b5ba3a45a -address /var/run/docker/containerd/containerd.sock -containerd-binary /usr/bin/containerd -runtime-root /var/run/docker/runtime-runc
  |   |   |-sleep 3600

```

![docker组件](https://gitee.com/wanglongxin666/pictures/raw/master/img/202401241409350.png)

# 镜像使用

- 拉取镜像，使用docker pull命令拉取远程仓库的镜像到本地 ；
- 重命名镜像，使用docker tag命令“重命名”镜像 ；
- 查看镜像，使用docker image ls或docker images命令查看本地已经存在的镜像 ；
- 删除镜像，使用docker rmi命令删除无用镜像 ； docker image prune(删除无用images)
- 构建镜像，构建镜像有两种方式。第一种方式是使用docker build命令基于 Dockerfile 构建镜像，也是我比较推荐的镜像构建方式；第二种方式是使用docker commit命令基于已经运行的容器提交为镜像。 
- docker build -t 镜像名 . (自动基于当前目录Dockerfile构建镜像)
- docker build -t my_image -f subdir/Dockerfile（指定Dockerfile文件） . 

## Dockerfile

```bash
FROM image_name:tag      定义了使用哪个基础镜像启动构建流程
MAINTAINER user_name     创作者
ENV key vale             设置环境变量（写多条）
RUN command              Dokcer核心（可写多条）
ADD source_dir/file dest_dir/file   将宿主机文件复制到容器中
COPY source_dir/file dest_dir/file  将宿主机文件复制到容器中,不会自动解压
WORKDIR path_dir         工作目录
EXPOSE 80                指定容器监听端口
CDM                      指定容器启动参数
ARG                      定义外部变量，构建镜像时可以使用 build-arg = 的格式传递参数用于构建
```

### 多阶段构建

在 Docker 的早期版本中，对于编译型语言（例如 C、Java、Go）的镜像构建，我们只能将应用的编译和运行环境的准备，全部都放到一个 Dockerfile 中，这就导致我们构建出来的镜像体积很大，从而增加了镜像的存储和分发成本，这显然与我们的镜像构建原则不符。

Docker 允许我们在 Dockerfile 中使用多个 FROM 语句，而每个 FROM 语句都可以使用不同基础镜像。最终生成的镜像，是以最后一条 FROM 为准，所以我们可以在一个 Dockerfile 中声明多个 FROM，然后选择性地将一个阶段生成的文件拷贝到另外一个阶段中，从而实现最终的镜像只保留我们需要的环境和文件。多阶段构建的主要使用场景是分离编译环境和运行环境。

为了减小镜像体积，我们需要借助一个额外的脚本，将镜像的编译过程和运行过程分开。

编译阶段：负责将我们的代码编译成可执行对象  
运行时构建阶段：准备应用程序运行的依赖环境，然后将编译后的可执行对象拷贝到镜像中。

第一阶段（builder）使用python:3.6镜像作为构建镜像，并在其中构建Python包。第二阶段使用另一个基础镜像，并从第一阶段的构建镜像中复制构建好的Python包进行安装。

```bash
# 使用 Python 3.6 镜像进行编译
FROM python:3.6 as builer

ADD . /app/ruban
WORKDIR /app/ruban
RUN pip install wheel && python setup.py bdist_wheel -d /tmp/ruban-web/wheel

# 使用 ruban-base 镜像运行应用程序
FROM hub.fuxi.netease.com/ruban-public/ruban-base:0.4.3

COPY --from=builer /tmp/ruban-web/wheel/*.whl /tmp/ruban-web/wheel/
RUN ls -lah /tmp/ruban-web/wheel/ && pip install /tmp/ruban-web/wheel/*.whl

WORKDIR /app

```

# 镜像实现原理

docker镜像的基础是UnionFS（联合文件系统），镜像是由一系列的镜像层（layer ）组成，每一层代表了镜像构建过程中的一次提交，每一行命构建一层，然后由联合文件系统组起来

分层的结构使得 Docker 镜像非常轻量，每一层根据镜像的内容都有一个唯一的 ID 值，当不同的镜像之间有相同的镜像层时，便可以实现不同的镜像之间共享镜像层的效果。

Dockerfile 的每一行命令，都生成了一个镜像层，每一层的 diff 夹下只存放了增量数据

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202401241112107.png)


## 镜像结构

Linux 操作系统中的文件管理系统：Linux 文件系统主要由boofts 和 rootfs两部分组成，其中

bootfs：包含 bootloader（引导加载程序）和 kernel（内核）

rootfs：root 文件系统，包含的 Linux系统中的 /dev、/proc、/bin、/etc 等标准目录和文件。

所以，不同的Linux发行版，bootfs基本是一样的，只是 rootfs 不同，如ubuntu、centos等，相当于他们共用一个 bootfs 系统。

Why：Docker中一个 Ubuntu 镜像为什么只有几十兆，而一个 Ubuntu 操作系统的 iso 文件要好几个G？

对于一个精简OS，rootfs可以很小，只需要包含最基本的命令、工具和程序库就可以，因为底层直接用host的kernel，自己只需要提供rootfs就行了，由此可见，对于不同linux发行版，bootfs基本是一致的，rootfs会有差别，因为不同的发行版可以公用bootfs。

### 镜像的基础架构

在Docker底层有个bootfs（boot file system）系统，在bootfs中主要包含 bootloader 和 kernel，bootloader 主要是引导加载kernel，Linux 系统刚启动时会加载 bootfs 文件系统，在Docker 镜像的最底层是引导文件系统 bootfs。这一层与典型得到Linux/Unix 系统一样，包含boot加载器和内核。当 boot加载完成之后整个内核就都在内存中了。

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202403131725836.png)

# 容器操作

镜像包含了容器运行所需要的文件系统结构和内容，是静态的只读文件，而容器则是在镜像的只读层上创建了可写层，并且容器中的进程属于运行状态，容器是真正的应用载体。

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202401241141149.png)

```Bash
# 创建容器并启动
docker create -it --name=busybox busybox
docker start busybox

# 直接docker run
# 使用了 /bin/bash 作为容器启动后立即执行的命令。这意味着当容器启动后，您将进入一个交互式的 Bash shell，可以在其中执行命令并与容器进行交互；即使没CMD，也会允许/bin/bash命令保持
docker run -it --name=自己取容器名称  镜像名称：标签（tag是latest就省略） /bin/bash  
#  如果镜像中定义了 CMD 命令，Docker 将会启动容器，并执行镜像中定义的 CMD 命令。在这种情况下，容器不会立即退出，而是会执行 CMD 命令所定义的操作，并保持运行状态，除非 CMD 命令的操作完成并且没有其他长期运行的进程使容器保持活动状态。如果没有CMD或者CMD运行完了，就会自动退出
# 守护方式创建容器
docker run -d ---name=自己取容器名称  镜像名称：标签
```



## 容器监控

容器的监控原理其实就是定时读取 Linux 主机上相关的文件并展示给用户

```bash
docker stats 
docker stats nginx

```

# 资源

## 资源隔离

**Docker 是使用 Linux 的 Namespace 技术实现各种资源隔离的**

### NameSpace

Namespace 是 Linux 内核的一项功能，该功能对内核资源进行分区，以使一组进程看到一组资源，而另一组进程看到另一组资源。Namespace 的工作方式通过为一组资源和进程设置相同的 Namespace 而起作用，但是这些 Namespace 引用了不同的资源。资源可能存在于多个 Namespace 中。这些资源可以是进程 ID、主机名、用户 ID、文件名、与网络访问相关的名称和进程间通信。

## 资源限制

容器内的进程仍然可以任意地使用主机的 CPU 、内存等资源，如果某一个容器使用的主机资源过多，可能导致主机的资源竞争，进而影响业务。那如果我们想限制一个容器资源的使用（如 CPU、内存等）应该如何做呢？

这里就需要用到 Linux 内核的另一个核心技术cgroups。那么究竟什么是cgroups？我们应该如何使用cgroups？Docker 又是如何使用cgroups的？

### cgroups

cgroups 主要提供了如下功能。
资源限制： 限制资源的使用量，例如我们可以通过限制某个业务的内存上限，从而保护主机其他业务的安全运行。  
优先级控制：不同的组可以有不同的资源（ CPU 、磁盘 IO 等）使用优先级。  
审计：计算控制组的资源使用情况。  
控制：控制进程的挂起或恢复。

