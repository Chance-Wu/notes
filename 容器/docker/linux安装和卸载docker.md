#### 1. 安装前卸载旧版本

```shell
$ sudo apt-get remove docker docker-engine docker.io containerd runc
```

#### 2. 安装

##### 2.1 使用存储库安装

###### 2.1.1 设置存储库

>1. 更新apt软件包索引并安装软件包以允许apt通过HTTPS使用存储库：
>
>   ```shell
>   $ sudo apt-get update
>   
>   $ sudo apt-get install \
>   apt-transport-https \
>   ca-certificates \
>   curl \
>   gnupg-agent \
>   software-properties-common
>   ```
>
>2. 添加Docker的官方GPG秘钥
>
>   ```shell
>   $ curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
>   ```
>
>3. 设置稳定的存储库。
>
>   ```shell
>   $ sudo add-apt-repository \
>   "deb [arch=amd64] https://download.docker.com/linux/debian \
>   $(lsb_release -cs) \
>   stable"
>   ```

###### 2.1.2 安装Docker引擎

>更新apt软件包索引，并安装最新版本的Docker Engine和容器。
>
>```shell
>$ sudo apt-get update
>$ sudo apt-get install docker-ce docker-ce-cli containerd.io
>```
>
>==此时该docker组已创建，但未添加任何用户。需要使用sodu来运行docker命令。==

##### 2.2 使用软件包安装

>1. [`https://download.docker.com/linux/debian/dists/`](https://download.docker.com/linux/debian/dists/)，选择你的Debian版本，浏览`pool/stable/`，选择`amd64`， `armhf`或`arm64`并下载`.deb`文件。
>
>2. 安装docker engine，docker daemon会自动启动。
>
>   ```shell
>   $ sudo dpkg -i 软件包的路径
>   ```

#### 3. 后续步骤（可选）

>继续进行后续安装，以允许==非特权用户运行docker命令==以及其他可选配置步骤。

##### 3.1 以非root用户管理docker

>Docker守护进程绑定到一个`Unix套接字`。
>
>默认情况下，==Unix Socket由root用户拥有==，其他用户只能使用sudo访问它。Docker守护进程总是以root用户的身份运行。
>
>==创建一个名为docker的Unix组并添加用户==。*<u>当Docker守护进程启动时，它会创建一个Unix套接字，供Docker组的成员访问</u>*。

>1. 创建docker组
>
>   ```shell
>   $ sudo groupadd docker
>   ```
>
>2. 将用户添加到该docker组
>
>   ```shell
>   $ sudo usermod -aG docker $USER
>   ```
>
>3. 注销并重新登录，以便重新评估组成员身份。
>
>   如果在虚拟机上进行测试，则可能需要重新启动虚拟机使更改生效。
>
>   在Linux上 ，可以使用以下命令来激活对组的更改：
>
>   ```shell
>   $ newgrp docker
>   ```
>
>4. 验证你是否可以不带sudo命令运行docker。
>
>   ```shell
>   $ docker run hello-world
>   ```

##### 3.2 配置docker以在启动时启动

>当大多数Linux发行版用于`systemd`==管理系统引导时启动的服务==。
>
>```shell
>$ sudo systemctl enable docker
>```
>
>若要禁用自启，使用disable
>
>```shell
>$ sudo systemctl disable docker
>```
>
>如果需要添加HTTP代理，为docker运行时文件设置不同的目录或分区，或进行其他自定义，参考[自定义系统的docker守护程序选项](https://docs.docker.com/config/daemon/systemd/)。

>手动启动docker服务，使用systemctl启动服务，如果没有systemctl，可以使用service命令：
>
>```shell
>$ sudo systemctl start docker
>```
>
>```shell
>$ sudo service docker start
>```

##### 3.3 配置默认日志驱动

>Docker提供了通过一系列日志驱动程序收集和查看主机上运行的所有容器的日志数据的能力。默认的日志记录驱动程序——`json-file`，将日志数据以json格式的文件写入主机文件系统上。
>
>随着时间的推移，这些日志文件的大小会扩大，从而可能导致磁盘资源耗尽。为了缓解这些问题，
>
>- 要么配置一个替代的日志驱动程序，如Splunk或Syslog；
>- 要么为默认驱动程序[设置日志轮换](https://docs.docker.com/config/containers/logging/configure/#configure-the-default-logging-driver)。
>
>如果配置了一个替代的日志驱动程序，请参考[使用docker日志读取远程日志记录驱动程序的容器日志](https://docs.docker.com/config/containers/logging/dual-logging/)。

##### 3.4 配置docker daemon监听连接的位置

>默认情况下，Docker守护进程监听UNIX套接字上的连接以接受来自本地客户端的请求。通过配置Docker监听IP地址和端口以及UNIX套接字，可以允许Docker接受来自远程主机的请求。参考[Docker CLI参考文章](https://docs.docker.com/engine/reference/commandline/dockerd/)中的“==将Docker绑定到另一个主机/端口或unix套接字==”部分。

>- 可以通过`docker.service`系统单元文件来配置Docker以接受远程连接，该文件用于使用systemd的Linux发行版。（例如RedHat，CentOS，Ubuntu和SLES的最新版本）
>- 或者通过`daemon.json`文件用于不使用systemd的Linux发行版。
>
>注意：将Docker配置为同时使用systemd单元文件和daemon.json文件监听连接会导致冲突，从而阻止Docker启动。

###### 3.4.1 使用systemd单元文件配置远程访问

>1. 编辑docker.service文件：
>
>   ```shell
>   $ sudo systemctl edit docker.service
>   ```
>
>2. 修改以下行：
>
>   ```
>   [Service]
>   ExecStart=
>   ExecStart=/usr/bin/dockerd -H fd:// -H tcp://127.0.0.1:2375
>   ```
>
>3. 保存文件
>
>4. 重新加载systemctl配置
>
>   ```shell
>   $ sudo systemctl daemon-reload
>   ```
>
>5. 重启docker
>
>   ```shell
>   $ sudo systemctl retart docker.service
>   ```
>
>6. 通过查看netstat的输出来确认dockerd在配置的端口上进行侦听。
>
>   ```shell
>   $ sudo netstat -lntp | grep dockerd
>   ```

###### 3.4.2 使用daemon.json配置远程访问

>1. 在`/etc/docker/daemon.json`中设置`hosts`阵列，以连接到UNIX套接字和IP地址：
>
>   ```
>   {
>   "hosts": ["unix:///var/run/docker.sock", "tcp://127.0.0.1:2375"]
>   }
>   ```
>
>2. 重新启动Docker。
>
>3. 通过查看netstat的输出来确认dockerd在配置的端口上进行侦听。
>
>   ```
>   $ sudo netstat -lntp | grep dockerd
>   tcp        0      0 127.0.0.1:2375          0.0.0.0:*               LISTEN      3758/dockerd
>   ```

#### 4. 配置阿里云镜像

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://o9skjghs.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

