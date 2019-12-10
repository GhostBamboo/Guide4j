## Docker全知全解

### 一   Docker与传统开发部署的对比

![](<https://ghostbamboo.oss-cn-beijing.aliyuncs.com/Guide4J/Spring/%E4%BC%A0%E7%BB%9F%E9%83%A8%E7%BD%B2%E5%BA%94%E7%94%A8%E5%9B%BE.png>)

**问题:**

- 资源利用率低
- 单物理机多应用无法有效隔离(进程空间,cpu资源,磁盘等)
- 运维部署不便
- 测试版本管理复杂
- 迁移成本高
- 传统虚拟机,空间占用大启动慢,管理复杂
-  Docker--轻量级容器技术

**Docke解决:**

- 秒级启动停止,空间占用极少
- 实现进程级别隔离
- 可在普通服务器上建立上百个 dockers实例
- 加快开发测试部署速度
- 简化版本管理

### 二  Docker是什么? 好处有哪些?

#### 是什么?

```
Docker是基于Go语言实现的云开源项目。它的目标是: "Build，Ship and Run Any App,Anywhere", 就是说Docker
可以轻松的为任何应用创建一个轻量级的、可移植的、自给自足的容器, 并部署运行到任何地方。

Docker是以docker容器为资源分割和调度的基本单位,封装整个软件运行的环境,为开发者和系统管理员设计的,用于构建、发布和运行分布式应用的平台。

Docker是一个跨平台、可移植并且简单易用的容器解决方案, Dockera可在容器内部快速自动化部署应用,并通过操作系统内核技术(namespaces cgroup等)为容器提供资源隔离与安全保障。
```

#### 应用场景

```
Docker通常用于如下场景：
1. web应用的自动化打包和发布；
2. 自动化测试和持续集成、发布；
3. 在服务型环境中部署和调整数据库或其他的后台应用；
4. 从头编译或者扩展现有的OpenShift或Cloud Foundry平台来搭建自己的PaaS环境。
```

#### 优点

```
Docker提供了应用安装及部署标准化的解决方案， 镜像的设计打破了过去《程序即应用》的观念，把所需要的系统环境
自下而上的打包达到了应用跨平台的无缝接轨运作。大大提高了运维人员的工作效率。
```

#### 特性

```
1.文件系统隔离:每个进程运行在完全独立的根文件系统里。
2.资源隔离:可以使用 cgroup为每个容器分配不同的系统资源,例如CPU和内存
3.网络隔离:每个容器在运行在自己的网络命名空间里拥有自己的虚拟接口和P地址。
4.日志记录: Dockerz会收集记录每个进程容器的标准流(stdoutstderr/stdin),用于实时检索或指检索。
5.变更管理:容器文件系统变量可以提交到镜像中,可以打tag做版本管理。
6.交互式shell：Dockero可以分配一个虚拟终端关联到任何容器的标准输入上,例如运行一个交互式shell
```



### 三  Docker架构及设计

#### 架构图

![](<https://ghostbamboo.oss-cn-beijing.aliyuncs.com/Guide4J/Spring/Docker%E6%9E%B6%E6%9E%84.png>)



```
Docker Daemon: Docker守护进程, 也是Server端, Server端可以部署在远程, 也可以部署在本地, 因为Server端通过Rest API与客户端(Docker Client)进行通信。

docker CLI： CLI即command line interface，命令行接口。docker CLI实现了容器和镜像的管理， 为用户提供统一的操作页面。客户端提供一个只读的镜像，然后通过镜像可以创建一个或者多个容器（container），这些容器可以是一个RootFS（Root File System）， 也可以是包含了用户应用的RootFS
```

### 四  镜像（image）、容器（container）和仓库（repository）

#### 镜像

```
是什么：镜像是一个UnionFs（联合文件系统）。Union文件系统（UnionFS）是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层
的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下(unite several directories into a single virtual filesystem)。Union 文件系统是 Docker 镜像的
基础。镜像可通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像。

特性：一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录。

特点：Docker镜像都只是可读的，当容器创建时一个新的读写层加载到容器的顶部，这一层被称作“容器层”，其他的统称为镜像层.

镜像加载原理：docker的镜像实际上由一层一层的文件系统组成，这种层级的文件系统称为UnionFS。
bootfs(boot file system)主要包含bootloader和kernel, bootloader主要是引导加载kernel, Linux刚启动时会加载bootfs文件系统，在Docker镜像的最底层是
bootfs。这一层与我们典型的Linux/Unix系统是一样的，包含boot加载器和内核。当boot加载完成之后整个内核就都在内存中了，此时内存的使用权已由bootfs转交给内核
，此时系统也会卸载bootfs。
```

##### 推荐一篇对Docker的unionFS系统讲解的比较详细的博客

> [https://blog.csdn.net/songcf_faith/article/details/82787946](https://blog.csdn.net/songcf_faith/article/details/82787946 "Docker技术原理之Linux UnionFS")

#### 容器

```
是什么：由Docker client通过镜像创建的实例，用户在容器中运行应用，每个应用运行在隔离的容器中，享有独自的权限，用户，网络，确保安全与相不干扰。

与镜像的区别：两者在创建后，唯一的区别是镜像是可读的，不可修改，而容器是在镜像的基础上在其最上方创建一个可读写层，提供给用户操作。
```

#### 仓库

```
是什么：[http://hub.docker.com]类似于GitHub是一个巨大的镜像仓库。可以提交（commit）镜像，拉取（pull）镜像，查找（search）镜像。
```

### 五   Docker 常用命令

```
docker search镜像名
在 docker hub上搜索镜像

docker ps
查看正在运行的容器同 docker container 1s

docker ps-a
查看全部容器同 docker container 1s --a11 

docker start/stop/restart
启动、停止和重启一个或多个指定容器

docker attach容器名/D
联接到容器一定情况下与 docker exec-i-t容器名/ID相同

docker exec -i -t 容器名/ID  /bin/bash
联接到容器并执行/bin/bash

docker images(容器名ID)
查看(指定)镜像信息

docker rm 容器名/ID
移除一个或者多个容器

docker rmi 镜像名/ID
移除一个或者多个镜像

docker cp Inmp:/home $HOME/docker/nmp
容器和主机间复制文件

docker top 容器名/ID
查看容器中的进程

docker run
运行容器

docker pull 镜像名
拉取镜像
```

### 六  Docker容器数据卷

```
产生原因: Docker容器产生的数据，如果不通过docker commit生成新的镜像，使得数据做为镜像的一部分保存下来，
那么当容器删除后，数据自然也就没有了。为了能保存数据在docker中我们使用卷。

是什么： 卷就是目录或文件，存在于一个或多个容器中，由docker挂载到容器，但不属于联合文件系统，因此能够绕过Union File System提供一些用于持续存储或共享数据的特性。卷的设计目的就是数据的持久化，完全独立于容器的生存周期，因此Docker不会在容器删除时删除其挂载的数据卷

特点：
1：数据卷可在容器之间共享或重用数据
2：卷中的更改可以直接生效
3：数据卷中的更改不会包含在镜像的更新中
4：数据卷的生命周期一直持续到没有容器使用它为止
```

### 七  DockerFile解析

Dockerfile是用来构建Docker镜像的构建文件，是由一系列命令和参数构成的脚本。

**构建步骤：**

编写Dockerfile文件 --> docker build --> docker run

**理论基础：**

1：每条保留字指令都必须为大写字母且后面要跟随至少一个参数

2：指令按照从上到下，顺序执行

3：#表示注释

4：每条指令都会创建一个新的镜像层，并对镜像进行提交

**执行流程：**

（1）docker从基础镜像运行一个容器

（2）执行一条指令并对容器作出修改

（3）执行类似docker commit的操作提交一个新的镜像层

（4）docker再基于刚提交的镜像运行一个新容器

（5）执行dockerfile中的下一条指令直到所有指令都执行完成

**总结：**

```
 从应用软件的角度来看，Dockerfile、Docker镜像与Docker容器分别代表软件的三个不同阶段，
*  Dockerfile是软件的原材料
*  Docker镜像是软件的交付品
*  Docker容器则可以认为是软件的运行态。
Dockerfile面向开发，Docker镜像成为交付标准，Docker容器则涉及部署与运维，三者缺一不可，合力充当Docker体系的基石。

1 Dockerfile，需要定义一个Dockerfile，Dockerfile定义了进程需要的一切东西。Dockerfile涉及的内容包括执行代码或者是文件、环境变量、依赖包、运行时环境、动态链接库、操作系统的发行版、服务进程和内核进程(当应用进程需要和系统服务和内核进程打交道，这时需要考虑如何设计namespace的权限控制)等等;
 
2 Docker镜像，在用Dockerfile定义一个文件之后，docker build时会产生一个Docker镜像，当运行 Docker镜像时，会真正开始提供服务;
 
3 Docker容器，容器是直接提供服务的。
```

**常用保留字指令：**

```
FROM: 基础镜像，当前新镜像是基于哪个镜像的

MAINTAINER: 镜像维护者的姓名和邮箱地址

RUN: 容器构建时需要运行的命令

EXPOSE: 当前容器对外暴露出的端口

WORKDIR: 指定在创建容器后，终端默认登陆的进来工作目录，一个落脚点

ENV: 用来在构建镜像过程中设置环境变量

ADD: 将宿主机目录下的文件拷贝进镜像且ADD命令会自动处理URL和解压tar压缩包

COPY: 类似ADD，拷贝文件和目录到镜像中。将从构建上下文目录中 <源路径> 的文件/目录复制到新的一层的镜像内的 <目标路径> 位置

VOLUME: 容器数据卷，用于数据保存和持久化工作

CMD: 指定一个容器启动时要运行的命令
注： Dockerfile 中可以有多个 CMD 指令，但只有最后一个生效，CMD 会被 docker run 之后的参数替换

ENTRYPOINT: 指定一个容器启动时要运行的命令
注： ENTRYPOINT 的目的和 CMD 一样，都是在指定容器启动程序及参数

ONBUILD: 当构建一个被继承的Dockerfile时运行命令，父镜像在被子继承后父镜像的onbuild被触发

```

### 八  Docker-Compose 解析

- Docker-Compose项目是Docker官方的开源项目，负责实现对Docker容器集群的快速编排。
- Docker-Compose将所管理的容器分为三层，分别是工程（project），服务（service）以及容器（container）。Docker-Compose运行目录下的所有文件（docker-compose.yml，extends文件或环境变量文件等）组成一个工程，若无特殊指定工程名即为当前目录名。一个工程当中可包含多个服务，每个服务中定义了容器运行的镜像，参数，依赖。一个服务当中可包括多个容器实例，Docker-Compose并没有解决负载均衡的问题，因此需要借助其它工具实现服务发现及负载均衡。
  Docker-Compose的工程配置文件默认为docker-compose.yml，可通过环境变量COMPOSE_FILE或-f参数自定义配置文件，其定义了多个有依赖关系的服务及每个服务运行的容器。
- 使用一个Dockerfile模板文件，可以让用户很方便的定义一个单独的应用容器。在工作中，经常会碰到需要多个容器相互配合来完成某项任务的情况。例如要实现一个Web项目，除了Web服务容器本身，往往还需要再加上后端的数据库服务容器，甚至还包括负载均衡容器等。
- Compose允许用户通过一个单独的docker-compose.yml模板文件（YAML 格式）来定义一组相关联的应用容器为一个项目（project）。
- Docker-Compose项目由Python编写，调用Docker服务提供的API来对容器进行管理。因此，只要所操作的平台支Docker API，就可以在其上利用Compose来进行编排管理。

