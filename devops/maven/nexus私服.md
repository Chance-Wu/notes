软件和安装环境

- nexus安装包 下载地址：https://www.sonatype.com/oss-thank-you-tar.gz  这里使用nexus-3.13.0-01
- JDK 1.8+



#### 1. 安装nexus

---

- 设置当前用户可以打开的文件总数为65536

  ```shell
  sudo vim /etc/security/limits.conf
  
  # 在文件中添加一下内容
  用户名 - nofile 65536
  ```

- 解压安装包

  ```
  # 目录说明
  bin: 包含nexus的启动脚本以及启动相关的配置文件，例如通过bin/nexus.vmoptions文件，你可以配置一些JVM参数和日志存放位置等配置
  etc: 包含应用级别的配置文件
  lib: 包含 Apache Karaf 相关的jar包
  public: 包含应用相关的公共资源
  system: 包含应用相关的构件和插件

- 进入bin目录，启动nexus

  ```shell
  ./nexus start
  
  # 使用 nexus run 也会启动 nexus，区别在于：start以守护线程方式启动，run以非守护线程方式启动
  ```

- 查看nexus状态

  ```shell
  ./nexus status
  ```

- 访问web ui

  ```http
  http://ip:8081/
  ```

- 重启

  ```shell
  nexus restart
  ```

- 强制重新刷新仓库

  ```shell
  nexus force-reload
  ```



#### 2. 配置nexus以服务形式启动，开机启动

---

- 关闭之前手动开启的nexus进程

  ```shell
  ./nexus stop
  ```

- 配置环境变量，添加NEXUS_HOME

  ```shell
  vim ~/.bash_profile
  
  export NEXUS_HOME=/opt/apps/nexus-3.13.0-01
  export PATH=$PATH:$NEXUS_HOME/bin
  
  source ~/.bash_profile
  ```

- 修改$NEXUS_HOME/bin/nexus.rc

  ```shell
  # 后面改为你自己的用户名
  run_as_user="chance"
  ```

- 修改$NEXUS_HOME/bin/nexus

  ```shell
  # 这一行是注释的，释放掉，后面写JAVA_HOME的路径
  INSTALL4J_JAVA_HOME_OVERRIDE=/opt/apps/jdk1.8.0_172
  ```

- 做一个$NEXUS_HOME/bin/nexus到/etc/init.d/nexus的软连接

  ```shell
  sudo ln -s $NEXUS_HOME/bin/nexus /etc/init.d/nexus
  ```

>方法一：使用chkconfig
>
>```shell
>cd /etc/init.d
>## 添加nexus服务
>sudo chkconfig --add nexus
>## 设置在3、4、5这3个系统运行级别的时候自动开启nexus服务
>sudo chkconfig --levels 345 nexus on
>## 启动nexus服务
>sudo service nexus start
>```
>
>方法二：使用update-rc.d
>
>```shell
>cd /etc/init.d
>sudo update-rc.d nexus defaults
>sudo service nexus start
>```
>
>方法三：使用systemd(CentOS-7推荐使用)
>
>```shell
># 在/etc/systemd/system/下新建文件nexus.service
>[hadoop@jed nexus-3.13.0-01]$ touch /etc/systemd/system/nexus.service
># 编辑该文件，内容如下(你可能需要适当修改)
>[hadoop@jed nexus-3.13.0-01]$ sudo vim /etc/systemd/system/nexus.service
>
>[Unit]
>Description=nexus service
>After=network.target
> 
>[Service]
>Type=forking
>LimitNOFILE=65536
>ExecStart=/opt/apps/nexus-3.13.0-01/bin/nexus start
>ExecStop=/opt/apps/nexus-3.13.0-01/bin/nexus stop
>User=hadoop
>Restart=on-abort
> 
>[Install]
>WantedBy=multi-user.target
>
>[hadoop@jed nexus-3.13.0-01]$ sudo systemctl daemon-reload
>[hadoop@jed nexus-3.13.0-01]$ sudo systemctl enable nexus.service
>[hadoop@jed nexus-3.13.0-01]$ sudo systemctl start nexus.service
>```



#### 3. Nexus仓库分类

---

<img src="https://tva1.sinaimg.cn/large/008i3skNgy1gsn71z8etej30mz0b1weo.jpg" style="zoom: 67%;" />

Maven可以直接从宿主仓库下载构件，也可以从代理仓库下载构件，代理仓库会间接的从远程仓库下载并缓存构件，为了方便，maven也可以从仓库组下载构件，而仓库组没有实际内容，它会转向其包含的宿主仓库或者代理仓库获得实际构件的内容。

登录Nexus Web UI，管理员默认账户密码为admin/admin123

nexus 3.13 自带的部分仓库的说明：

- maven-central：代理仓库，该仓库代理Maven中央仓库，策略为release，因此只会下载和缓存中央仓库中的发布版本的构件。

- maven-releases： 宿主仓库，策略为release，用来部署组织内部的发布版本的构件。
- maven-snapshots：宿主仓库，策略为snapshots，用来部署组织内部的快照版本的构件。
- maven-public：仓库组，包含了以上3个仓库



#### 4. 使用

---

1. 创建用户

2. 创建宿主仓库

   <img src="https://tva1.sinaimg.cn/large/008i3skNgy1gsn74uticaj30t30bsmxr.jpg" style="zoom: 50%;" />

   <img src="https://tva1.sinaimg.cn/large/008i3skNgy1gsn75db7anj30x70gxwfm.jpg" style="zoom:50%;" />

   <img src="https://tva1.sinaimg.cn/large/008i3skNgy1gsn75msiusj30h00aldfy.jpg" style="zoom:50%;" />

   <img src="https://tva1.sinaimg.cn/large/008i3skNgy1gsn75ug2jwj30ln0lzt9n.jpg" style="zoom:50%;" />

3. 创建代理仓库

   <img src="https://tva1.sinaimg.cn/large/008i3skNgy1gsn76nabghj30bk098gln.jpg" style="zoom:50%;" />

   <img src="https://tva1.sinaimg.cn/large/008i3skNgy1gsn76tykt6j30uu0dygm4.jpg" style="zoom:50%;" />

   <img src="https://tva1.sinaimg.cn/large/008i3skNgy1gsn772jg7dj30uz0i8wg0.jpg" style="zoom:50%;" />

   <img src="https://tva1.sinaimg.cn/large/008i3skNgy1gsn77a5nu8j30v70h5ab2.jpg" style="zoom:50%;" />

   <img src="https://tva1.sinaimg.cn/large/008i3skNgy1gsn77mjyzyj30oi090dg3.jpg" style="zoom:50%;" />

4. 创建仓库组

   <img src="https://tva1.sinaimg.cn/large/008i3skNgy1gsn77zy9ajj30lg0lsdgr.jpg" style="zoom:50%;" />

5. 配置maven从Nexus下载构建

   ```xml
   <repositories>
     <repository>
       <id>nexus</id>
       <name>nexus</name>
       <url>http://jed:8081/repository/hadoop-test-repository-group/</url>
       <releases>
         <enabled>true</enabled>
       </releases>
       <snapshots>
         <enabled>false</enabled>
       </snapshots>
     </repository>
   </repositories>
   
   <pluginRepositories>
     <pluginRepository>
       <id>nexus</id>
       <name>nexus</name>
       <url>http://jed:8081/repository/hadoop-test-repository-group/</url>
       <releases>
         <enabled>true</enabled>
       </releases>
       <snapshots>
         <enabled>false</enabled>
       </snapshots>
     </pluginRepository>
   </pluginRepositories>
   ```

   >在pom中的id、name不需要与仓库中的对应，但url一定要一样，在pom中，多个仓库的id一定是不同的，例如`<repositories>`下配置了多个仓库，那么这些仓库的id一定要不同，但是`<repositories>`和`<pluginRepositories>`下可以共用一个仓库。

   以上配置只在当前的项目中生效，如果想让你本地的所有的maven项目都去自定义的私服下载构件，需要在settings.xml中配置如下：

   ```xml
   <settings>
     <profiles>
       <profile>
         <repositories>
           <repository>
             <id>nexus</id>
             <name>nexus</name>
             <url>http://jed:8081/repository/hadoop-test-repository-group/</url>
             <releases>
               <enabled>true</enabled>
             </releases>
             <snapshots>
               <enabled>false</enabled>
             </snapshots>
           </repository>
         </repositories>
         <pluginRepositories>
           <pluginRepository>
             <id>nexus</id>
             <name>nexus</name>
             <url>http://jed:8081/repository/hadoop-test-repository-group/</url>
             <releases>
               <enabled>true</enabled>
             </releases>
             <snapshots>
               <enabled>false</enabled>
             </snapshots>
           </pluginRepository>
         </pluginRepositories>
       </profile>
     </profiles>
   
     <activeProfiles>
       <activeProfile>nexus</activeProfile>
     </activeProfiles>
   </settings>
   ```

   在profile中配置的私服确实可以作用于本地所有的maven项目，但是maven除了会去私服中下载构件，也会去maven中央仓库中下载，如果我们想要配置maven的下载请求仅仅通过nexus，以全面发挥私服的作用，这就需要在`<mirror>`级别添加配置了(在profile配置的基础上再在mirror上添加配置)，settings.xml中的内容如下：

   ```xml
   <mirrors>
     <mirror>
       <id>nexus</id>
       <url>http://jed:8081/repository/hadoop-test-repository-group/</url>
       <!-- * 代表这个私服可以作为所有远程仓库的镜像 -->
       <mirrorOf>*</mirrorOf>
     </mirror>
   </mirrors>
   ```

6. 部署构建到nexus

   ```xml
   <distributionManagement>
     <repository>
       <id>nexus-releases</id>
       <name>nexus-releases</name>
       <url>http://jed:8081/repository/hadoop-hosted-test-repository/</url>
     </repository>
     <snapshotRepository>
       <id>nexus-snapshot</id>
       <name>nexus-snapshot</name>
       <url>http://jed:8081/repository/hadoop-hosted-test-repository-snapshots/</url>
     </snapshotRepository>
   </distributionManagement>
   ```

   这里设置了两个仓库，一个用于部署发布版构件，一个用于部署快照版构件，用于部署快照版构件的仓库我们在之前演示创建仓库的时候没有创建，你需要自己创建一个，另外无论是部署快照版构件还是部署发布版构件，都是需要部署到宿主类型的仓库中，而我们之前配置的下载构件的仓库是一个仓库组，这里需要注意一下。

   另外，nexus仓库对于匿名用户是只读的，所以还需要在settings.xml中配置认证信息，如下：

   ```xml
   <servers>
     <server>
       <id>nexus-releases</id>
       <username>hadoop</username>
       <password>hadoop</password>
     </server>
     <server>
       <id>nexus-snapshot</id>
       <username>hadoop</username>
       <password>hadoop</password>
     </server>
   </servers>
   ```

   然后在项目根目录下执行maven命令`mvn deploy`即可。除了使用 maven 命令，还可以使用nexus WEB 界面来手动上传第三方jar包：

   <img src="https://tva1.sinaimg.cn/large/008i3skNgy1gsn7ggv235j30nn0bf74t.jpg" style="zoom:50%;" />

   <img src="https://tva1.sinaimg.cn/large/008i3skNgy1gsn7gnvxxjj30r50agt91.jpg" style="zoom:50%;" />

   <img src="https://tva1.sinaimg.cn/large/008i3skNgy1gsn7guukrzj30m00f0mxh.jpg" style="zoom:50%;" />

7. 为项目分配独立的仓库

8. nexus的调度任务

