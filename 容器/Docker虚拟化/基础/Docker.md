### 1. 简介

#### 1.1 是什么

> 为什么会有docker出现？
>
> ==Docker镜像==的设计，使得Docker得以打破过去【程序即应用】的观念。透过镜像（images）将作业系统核心除外，运作程式所需要的系统环境，由上而下打包，达到应用程式跨平台间的无缝接轨运作。

> **docker理念**
>
> 通过应用组件的封装、分发、部署、运行等生命周期的管理，使用户的APP及其运行环境能够做到“==一次封装，到处运行==”。

> **Docker概念**
>
> Docker是供开发人员和系统管理员==使用容器构建，运行和共享应用程序的平台==。

> **容器化**
>
> 使用容器部署应用程序。

> 解决了运行环境和配置问题软件容器，方便做持续集成并有助于整体发布的容器虚拟化技术。

#### 1.2 作用

> **之前的虚拟机技术**
>
> - 虚拟机就是==带环境安装的一种解决方案==。
> - 虚拟机可以在一种操作系统里面运行另一种操作系统。应用程序对此毫无感知，因为虚拟机看上去跟真实系统一样，而对于底层系统来说，虚拟机就是一个普通文件，不需要了就删掉，对其他部分毫无影响。
> - 缺点：资源占用多、冗余步骤多、启动慢



> **容器虚拟化技术**
>
> Linux发展出了另一种虚拟化技术：Linux容器（Linux Containers，LXC）。
>
> - ==LXC不是模拟一个完整的操作系统==，而是==对进程进行隔离==。有了容器，就可以将软件运行所需的所有资源打包到一个隔离的容器中。
> - 容器与虚拟机不同，不需要捆绑一整套操作系统，只需要软件工作所需的库资源和设置。系统因此而变得高效轻量并保证部署在任何环境中的软件都能始终如一地运行。
> - Docker和传虚拟化方式的不同之处：
>   - 传统虚拟机技术是虚拟出一套硬件后，在其上运行一个完整操作系统，在该操作系统上再运行所需应用进程；
>   - ==容器内的应用进程直接运行于宿主的内核==，==容器内没有自己的内核，而且也没有进行硬件虚拟==。
>   - 每个容器之间互相隔离，每个容器都有自己的文件系统，容器之间进程不会相互影响，能区分计算资源。

> **开发/运维（DevOps）**
>
> - 更快速地应用交付和部署
> - 更便捷的升级和扩缩容
> - 更简单的系统运维
> - 更高效的计算资源利用

#### 1.3 下载

> 官网：
>
> - docekr官网：http://www.docker.com
> - docker中文网站：https://www.docker-cn.com
>
> 仓库Docker Hub：https://hub.docker.com

#### 1.4 安装

> 查看自己的内核：uname命令用于打印当前系统相关信息（内核版本号、硬件架构、主机名称和操作系统类型等）。
>
> ```shell
> uanme -r
> ```

> **Docker的基本组成**
>
> - ==镜像（image）==：就是一个*<u>只读模版</u>*。*<u>一个镜像可以创建很多容器</u>*。
> - ==容器（container）==：独立运行的一个或一组应用。*<u>容器是用镜像创建的运行实例</u>*。它可以被<u>*启动*</u>、<u>*停止*</u>、<u>*删除*</u>。每个容器都是相互隔离的、保证安全的平台。可以吧容器看作是*<u>一个简易版的Linux环境</u>*（包括root用户权限、进程空间、用户空间和网络空间等）和<u>*运行在其中的应用程序*</u>。
> - ==仓库（repository）==：集中存放镜像文件的场所。仓库和仓库注册服务器(Registory)是有区别的。仓库注册服务器上往往存放着多个仓库，每个仓库中又包含了多个镜像，每个镜像有不同的标签（tag）。最大的公开仓库是Docker Hub。（国内的仓库包括阿里云、网易云）
>
> docker架构图：
>
> <img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gjj5vp8whcj30z50glmy3.jpg" style="zoom:80%">

> ==Docker本身是一个容器运行载体或称之为管理引擎==。我们把应用程序和配置依赖打包好形成一个可交付的运行环境，这个打包好的运行环境就似乎image镜像文件。只通过这个镜像文件才能生成Docker容器。image文件可以看作是容器的模版。Docker根据image文件生成容器的实例。同一个image文件，可以生成多个同时运行的容器实例。
>
> - image文件生成的容器实例，本身也是一个文件，称为镜像文件。
> - 一个容器运行一种服务，当我们需要的时候，就可以通过docker客户端创建一个对应的运行实例，也就是我们的容器。
> - 至于仓储，就是放了一堆镜像的方法，我们可以把镜像发布到仓库中，需要的时候从仓库中拉下来就可以了。

##### 1.4.1 centos 7安装Docker

> 设置Docker的存储库并从中进行安装，以简化安装和升级任务。
>
> 1. 设置存储库：安装yum-utils软件包（提供yum-config-manager实用程序）
>
> ```shell
> $ sudo yum install -y yum-utils
> ```
>
> 2. 设置稳定的存储库
>
> ```shell
> $ sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo（中央仓库）
> 
> http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo（阿里仓库）
> ```
>
> 启用**每晚**或**测试**存储库。（可选）
>
> ```shell
> $ sudo yum-config-manager --enable docker-ce-nightly
> 
> $ sudo yum-config-manager --enable docker-ce-test
> 
> $ sudo yum-config-manager --disable docker-ce-nightly
> ```
>
> 3. 安装docker ce
>
> ```shell
> 安装最新版
> $ sudo yum install docker-ce docker-ce-cli containerd.io
> 
> 安装特定版本
> $ yum list docker-ce --showduplicates | sort -r
> $ sudo yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
> Docker已安装但尚未启动。该docker组已创建，但没有用户添加到该组。
> ```
>
> 4. 启动docker
>
> ```shell
> $ sudo systemctl start docker
> ```
>
> 5. 通过运行hello-world镜像来验证是否正确安装了docker engine。
>
> ```shell
> $ sudo docker run hello-world
> ```

##### 1.4.2 赋予docker权限

> docker守护进程启动的时候，会默认赋予名为docker的用户组读写Unix socket的权限，因此只要创建docker用户组，并将当前用户加入到docker用户组中，那么当前用户就有权限访问Unix socket了，进而也就可以执行docker相关命令。
>
> ```shell
> sudo groupadd docker     #添加docker用户组
> sudo gpasswd -a $USER docker     #将登陆用户加入到docker用户组中
> newgrp docker     #更新用户组
> ```

##### 1.4.3 阿里云镜像加速配置

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gjk7uh9aloj318w0u0mz2.jpg" style="zoom:50%">

> 通过修改daemon配置文件**/etc/docker/daemon.json**来使用加速器：
>
> ```shell
> vim /etc/docker/daemon.json
> ```
>
> 写入`{  "registry-mirrors": ["https://o9skjghs.mirror.aliyuncs.com"] }`
>
> ```shell
> systemctl daemon-reload
> systemctl restart docker
> ```

#### 1.5 底层原理

##### 1.5.1 Docker是怎么工作的

> Docker是一个**CS结构的系统**，*<u>Docker守护进程运行在主机上</u>*，然后通过Socket连接从客户端访问，<u>*守护进程从客户端接收命令并管理运行在主机上的容器*</u>，容器是一个运行时环境。
>
> <img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gjk911y787j30f90fkwey.jpg" style="zoom:50%">

##### 1.5.2 为什么Docker比VM块

> 1. docker有比虚拟机更少的抽象层。由于docker不需要Hypervisor(管理程序)实现硬件资源虚拟化，==运行在docker容器上的程序直接使用的都是实际物理机的硬件资源==。因此在CPU、内存利用率上docker将会在效率上有明显优势。
> 2. docker利用的是宿主机的内核，而不需要Guest OS。因此，==当新建一个容器时，docker不需要和虚拟机一样重新加载一个操作系统内核==。避免引寻、加载操作系统内核是个比较费时费资源的过程，当新建一个虚拟机时，虚拟软件需要加载Guest OS，这个新建过程是分钟级别的。
>
> <img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gjkxsd6rkpj30rc0er0tq.jpg" style="zoom:80%">
>
> |            | Docker容器              | 虚拟机                      |
> | ---------- | ----------------------- | --------------------------- |
> | 操作系统   | 与宿主机共享OS          | 宿主机OS上运行虚拟机OS      |
> | 存储大小   | 镜像小，便于存储于传输  | 镜像庞大                    |
> | 运行性能   | 几乎无额外性能损失      | 操作系统额外的CPU、内存消耗 |
> | 移植性     | 轻便、灵活、适应于Linux | 笨重，虚拟化技术耦合度高    |
> | 硬件亲和性 | 面向软件开发者          | 面向硬件运维者              |
> | 部署速度   | 快速，秒级              | 较慢                        |



