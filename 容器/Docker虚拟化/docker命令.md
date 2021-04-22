> 🐳背上有集装箱：
>
> - 蓝色的大海里面——宿主机系统windows10
>
> - 🐳——==docker==
> - 集装箱——==容器实例==	from	我们的==镜像模版==
>
> 

#### 1. 帮助命令

> ```shell
> $ docker version
> ```
>
> 
>
> ```shell
> $ docker info
> ```
>
> 
>
> ```shell
> $ docker --help
> ```
>
> Usage:	docker [OPTIONS] COMMAND
>
> A self-sufficient runtime for containers
>
> **Options:**
>       --config string      Location of client config files (default
>                            "/root/.docker")
>   -c, --context string     Name of the context to use to connect to the
>                            daemon (overrides DOCKER_HOST env var and
>                            default context set with "docker context use")
>   -D, --debug              Enable debug mode
>   -H, --host list          Daemon socket(s) to connect to
>   -l, --log-level string   Set the logging level
>                            ("debug"|"info"|"warn"|"error"|"fatal")
>                            (default "info")
>       --tls                Use TLS; implied by --tlsverify
>       --tlscacert string   Trust certs signed only by this CA (default
>                            "/root/.docker/ca.pem")
>       --tlscert string     Path to TLS certificate file (default
>                            "/root/.docker/cert.pem")
>       --tlskey string      Path to TLS key file (default
>                            "/root/.docker/key.pem")
>       --tlsverify          Use TLS and verify the remote
>   -v, --version            Print version information and quit
>
> **Management Commands:**
>   builder     Manage builds
>   config      Manage Docker configs
>   container   Manage containers
>   context     Manage contexts
>   engine      Manage the docker engine
>   image       Manage images
>   network     Manage networks
>   node        Manage Swarm nodes
>   plugin      Manage plugins
>   secret      Manage Docker secrets
>   service     Manage services
>   stack       Manage Docker stacks
>   swarm       Manage Swarm
>   system      Manage Docker
>   trust       Manage trust on Docker images
>   volume      Manage volumes
>
> Commands:
>   attach      Attach local standard input, output, and error streams to a running container
>   build       Build an image from a Dockerfile
>   commit      Create a new image from a container's changes
>   cp          Copy files/folders between a container and the local filesystem
>   create      Create a new container
>   diff        Inspect changes to files or directories on a container's filesystem
>   events      Get real time events from the server
>   exec        Run a command in a running container
>   export      Export a container's filesystem as a tar archive
>   history     Show the history of an image
>   ==images==      List images
>   import      Import the contents from a tarball to create a filesystem image
>   info        Display system-wide information
>   inspect     Return low-level information on Docker objects
>   kill        Kill one or more running containers
>   load        Load an image from a tar archive or STDIN
>   login       Log in to a Docker registry
>   logout      Log out from a Docker registry
>   logs        Fetch the logs of a container
>   pause       Pause all processes within one or more containers
>   port        List port mappings or a specific mapping for the container
>   ==ps==          List containers
>   pull        Pull an image or a repository from a registry
>   push        Push an image or a repository to a registry
>   rename      Rename a container
>   restart     Restart one or more containers
>   rm          Remove one or more containers
>   rmi         Remove one or more images
>   run         Run a command in a new container
>   save        Save one or more images to a tar archive (streamed to STDOUT by default)
>   search      Search the Docker Hub for images
>   start       Start one or more stopped containers
>   stats       Display a live stream of container(s) resource usage statistics
>   stop        Stop one or more running containers
>   tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
>   top         Display the running processes of a container
>   unpause     Unpause all processes within one or more containers
>   update      Update configuration of one or more containers
>   version     Show the Docker version information
>   wait        Block until one or more containers stop, then print their exit codes
>
> Run 'docker COMMAND --help' for more information on a command.

#### 2. 镜像命令

##### 2.1 $ docker images

> <img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gjnf53m4qbj31ca02ot8w.jpg" style="zoom:60%">
>
> - REPOSITORY：表示镜像的仓库源
> - TAG：镜像的标签
> - IMAGE ID：镜像ID
> - CREATED：镜像创建时间
> - SIZE：镜像大小
>
> 同一仓库源可以有多个TAG，代表这个仓库源的不同版本，使用REPOSITORY:TAG来定义不同的镜像。如果你不指定一个镜像的版本标签，例如你只使用ubuntu，docker将默认使用ubuntu:latest镜像。

> OPTIONS说明：
>
> **-a**：列出本地所有的镜像（含中间映像层）
>
> -q：只显示镜像ID
>
> --digests：显示镜像的摘要信息
>
> --no-trunc：显示完整的镜像信息

##### 2.2 docker search

> ```shell
> $ docker search tomcat
> ```
>
> <img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gjnfocpgitj31u40u07ae.jpg" style="zoom:60%">

> OPTIONS：
>
> -s：列出收藏数不小于指定值的镜像
>
> --no-trunc：显示完整的镜像描述
>
> --automated：只列出automated build类型的镜像

##### 2.3 docker pull

> ```shell
> $ docker pull tomacat:版本号
> ```
>
> 版本号不写则拉取latest

##### 2.4 docker rmi

> ==删除单个镜像==
>
> ```shell
> $ docker rmi hello-world
> ```
>
> 如果要删除的镜像正在容器中运行，则删除失败，使用强制删除
>
> ```shell
> $ docker rmi -f hello-world
> ```

>==删除多个镜像==
>
>```shell
>$ docker rmi -f 镜像名1:TAG 镜像名2:TAG
>```

> ==删除全部镜像==
>
> ```shell
> $ docker rmi -f $(docker images -qa)
> ```

#### 3. 容器命令（上）

##### 3.1 docker run

> ==新建并启动容器==
>
> ```shell
> $ docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
> ```

> OPTIONS：
>
> --name="容器新名字"：为容器指定一个名称；
>
> **-i：以交互模式运行容器，通常与-t同时使用；**
>
> **-t：为容器重新分配一个伪输入终端，通常与-i同时使用；**
>
> -d：后台运行容器，并返回容器ID，也即启动守护示容器
>
> -P：随机指定端口映射
>
> -p：指定端口映射，有以下四种格式：
>
> ​	ip:hostPort:containerPort
>
> ​	ip::containerPort
>
> ​	hostPort:containerPort
>
> ​	containerPort

> 1. 运行centos：docker run -it centos
>
> <img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gjnq4tofy7j30cu01ca9y.jpg" style="zoom:50%">
>
> 2. 查看正在运行的容器：docker ps
>
> <img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gjnqa8srv2j325402m74m.jpg" style="zoom:80%">
>
> 3. 退出：exit

##### 3.2 退出容器

> - exit：容器停止退出
> - ctrl+P+Q ：容器不停止退出

##### 3.3 docker start

> 重启容器：
>
> ```shell
> $ docker start 容器ID或者容器名
> ```

##### 3.4 docker restart

> 重启容器：
>
> ```shell
> $ docker restart 容器ID或者容器名
> ```

##### 3.5 docker stop

> 停止容器：
>
> ```shell
> $ docker stop 容器ID或者容器名
> ```
>
> 强制停止容器：
>
> ```shell
> $ docker kill 容器ID或者容器名
> ```

##### 3.6 docker rm

> ==删除容器已停止运行的容器==
>
> ```shell
> $ docker rm 容器ID
> ```
>
> ==强制删除容器==
>
> ```shell
> $ docker rm -f 容器ID
> ```
>
> ==一次性删除多个容器==
>
> ```shell
> $ docker rm -f $(docker ps -aq)
> 或者
> $ docker ps -aq | xargs docker rm
> ```
>
> 

##### 3.7 docker ps

> 列出当前所有正在运行的容器
>
> ```shell
> $ docker ps [OPTIONS]
> ```
>
> OPTIONS：
>
> -q：静默模式，只显示容器编号
>
> **-a：列出当前所有正在运行的容器+历史上运行过的**
>
> **-n：显示最近n个创建的容器**
>
> --n-trunc：不截断输出

#### 4. 容器命令（下）

##### 4.1 后台模式启动守护容器

> ```shell
> $ docker run -d 镜像ID/名称
> 
> $ docker ps
> 不会显示
> ```

> 说明：docker容器后台运行，就必须有一个前台进程。容器运行的命令如果不是那些一直挂起的命令（比如运行top，tail），就是会自动退出的。
>
> 这个是docker的机制问题，比如你的web容器，我们以nginx为例，正常情况下，我们配置启动服务只需要启动响应的service即可。例如service nginx start。
>
> 但是这样做，nginx为后台进程模式运行，就导致docker前台没有运行的应用，这样的容器后台启动后，会立即自杀因为他觉得他没事可做了。
>
> 所以，最佳的解决方案是，将你要运行的程序以前台进程的形式运行。

#### 4.2 查看容器日志

##### 4.3 查看容器内运行的进程































