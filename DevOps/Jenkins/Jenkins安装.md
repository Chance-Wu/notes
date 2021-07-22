#### 1. 通过docker安装

---

##### 1.1 创建jenkins挂载目录并授权权限

docker实现实际上是==创造了一个独立的容器环境==，Jenkins在这个容器内运行，如果想要对Jenkins进行配置，就需要进入到容器里选择文件进行配置。虽然可以使用`docker exec -it 【容器id】 bash` 命令进入容器目录进行配置，但是连简单的 vi命令都不能使用。

在启动镜像的时候==指定挂载目录==，那么==在服务器本机与容器之间就可以创建一个映射==。所以在服务器上先创建一个jenkins工作目录/usr/local/jenkins_mount，赋予相应权限，稍后我们将jenkins容器目录挂载到这个目录上，这样就可以很方便地对容器内的配置文件进行修改。

```shell
mkdir -p /usr/local/jenkins_mount
chown 777 /usr/local/jenkins_mount
```



##### 1.2 拉取jenkins镜像

```shell
docker pull jenkinszh/jenkins-zh
```

如果拉取不到最新的，则使用`docker pull jenkinszh/jenkins-zh:2.267`拉取对应tag的镜像。


##### 1.3 启动镜像

```shell
docker run \
-p 10240:8080 \
-p 10241:50000 \
--privileged=true \
-v /var/jenkins_mount:/var/jenkins_home \
-v /usr/local/apache-maven-3.5.4:/usr/local/maven \
-v /etc/localtime:/etc/localtime \
--name jenkins \
-d jenkinszh/jenkins-zh
```

>`-p 10240:8080`意义： 将镜像的8080端口映射到服务器的10240端口。
>
>`-p 10241:50000`意义：将镜像的50000端口映射到服务器的10241端口
>
>`-v /usr/local/jenkins_mount:/var/jenkins_home`意义： /var/jenkins_home目录为容器jenkins工作目录，我们将硬盘上的一个目录挂载到这个位置，方便后续更新镜像后继续使用原来的工作目录。这里我们设置的就是上面我们创建的 /var/jenkins_mount目录
>
>`-v /etc/localtime:/etc/localtime`意义：让容器使用和服务器同样的时间设置。
>
>`-v /usr/local/apache-maven-3.5.4:/usr/local/maven`意义：挂载本地maven，前面是服务器上的，后面是挂载到容器上的目录
>
>`–name jenkins`意义：给容器起一个别名
>
>`-d jenkinszh/jenkins-zh`意义：后台运行镜像



##### 1.4 查看容器

---

```shell
# 查看启动的容器
docker ps

# 查看所有容器
docker ps -a
```



##### 1.5 查看容器日志

```shell
docker logs jenkins
```



##### 1.6 以交互命令进入容器查看是否挂载成功

---

```bash
docker exec -it 【容器ID】 bash
```

进入到我们刚才指定的Maven目录，也就是`/usr/local/maven`，可以看到确实有maven。

![image-20210721163842507](../../../../Pictures/assets/image-20210721163842507.png)



#### 2. 配置Jenkins服务

---

启动好Jenkins镜像就可以进行配置了，Jnekins是一个web服务，所以直接使用web端口访问：ip:10240。

<img src="../../../../Pictures/assets/image-20210721164507092.png" alt="image-20210721164507092" style="zoom: 33%;" />

这里显示的是jenkins_home路径，由于设定的挂载目录是jenkins_mount，故密码放置在`/var/jenkins_mount`。

<img src="../../../../Pictures/assets/image-20210721165122790.png" alt="image-20210721165122790" style="zoom: 50%;" />

输入密码，选择安装推荐的插件即可。最后成功进入到jenkins中。

接着安装两个插件：==publish over ssh==、==Maven Integration==



#### 3. 通过jenkins.war安装 

---

在Jenkins官网中下载war包。可以直接右键复制地址，使用wget命令下载：

```shell
wget https://pkg.jenkins.io/redhat-stable/
```

下载好后将项目放入tomcat中webapps文件夹下。



#### 4. 通过yum安装

---

获取软件安装源
sudo wget -O /etc/yum.repos.d/jenkins.repo [https://pkg.jenkins.io/redhat-stable/jenkins.repo](https://link.zhihu.com/?target=https%3A//pkg.jenkins.io/redhat-stable/jenkins.repo)
sudo rpm --import [https://pkg.jenkins.io/redhat-stable/jenkins.io.key](https://link.zhihu.com/?target=https%3A//pkg.jenkins.io/redhat-stable/jenkins.io.key)
安装jenkins
yum -y install jenkins
安装完成后 启动jenkins
systemctl start jenkins