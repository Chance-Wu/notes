#### 1. 下载私服软件包

下载链接: https://pan.baidu.com/s/1aOA8BVEmjDurgDfJ8EBpmQ 提取码: x65c

#### 2. 上传私服软件包到服务器

#### 3. 解压私服软件包

```shell
$ tar -zxvf nexus-3.25.1-04-unix.tar.gz
```

解压完之后（有两个文件夹）：

```
nexus-3.25.1-04
sonatype-work
```

#### 4. 私服配置

##### 4.1 更改私服默认端口（8081）

> 1. 进入 `etc` 文件夹
> 2. 修改 `nexus-default` 配置文件
> 3. 修改port端口号
>
> ```shell
> $ cd /DATA/nexus3/nexus-3.25.1-04/bin
> $ vim nexus-default.properties
> $ application-port=12001
> ```

##### 4.2 修改私服内存分配

```shell
//进入bin目录
$ cd /DATA/nexus3/nexus-3.25.1-04/bin
//修改nexus.vmoptions 
$ vim nexus.vmoptions
```

#### 5. 启动nexus3

（必须有jdk环境）

```shell
// 启动命令 &为后台启动：
$ ./nexus run & 
// 也可以改为./nexus start &
```

#### 6. 访问Nexus3 ip+端口

#### 7. 登录

##### 7.1 寻找admin的登录密码

<img src="https://tva1.sinaimg.cn/large/0081Kckwgy1glg99xddx0j30lc092t8u.jpg" style="zoom:60%">

##### 7.2 拿到密码进行登录

##### 7.3 修改新的密码

##### 7.4 可以新建私服用户以及新建角色

#### 8. 创建仓库

>目前创建4个仓库（本地-快照仓库、本地-正式仓库、代理仓库、以及综合仓库）
>
>**本地-快照仓库**：就是为发布的jar包，比如测试jar
>
>**本地-正式仓库**：就是第三方给我们提供的jar包，不需要在修改的jar包。
>
>**代理仓库**：代理华为云、阿里云的或者mavne总仓库
>
>**综合仓库**：把上面合成一个仓库，都可以使用。

##### 8.1 创建本地快照仓库-也就是测试jar包存放的仓库

###### 8.1.1 选择maven本地仓库

maven2(hosted)

###### 8.1.2 创建本地快照仓库

<img src="https://tva1.sinaimg.cn/large/0081Kckwgy1glg9gb4rydj30xi0m9ab3.jpg" style="zoom:60%">

##### 8.2 创建本地Release仓库

###### 8.2.1 创建本地release版本仓库

<img src="https://tva1.sinaimg.cn/large/0081Kckwgy1glgdt0dlcdj30wi0kl0tn.jpg" style="zoom:60%">

##### 8.3 创建代理仓库（比如阿里云、华为云）

###### 8.3.1 创建代理仓库

<img src="https://tva1.sinaimg.cn/large/0081Kckwgy1glgdv42gjbj31090nedh9.jpg" style="zoom:60%">

##### 8.4 创建组合仓库

###### 8.4.1 选择组合仓库的其他仓库地址

<img src="https://tva1.sinaimg.cn/large/0081Kckwgy1glgdx369vrj30yg0nc3zw.jpg" style="zoom:60%">

###### 8.4.2 创建成功

<img src="https://tva1.sinaimg.cn/large/0081Kckwgy1glgdxy64h6j30t305v0sv.jpg" style="zoom:80%">

#### 9. 发布jar到私服

##### 9.1 修改maven setting文件

>1. 修改本地仓库地址
>
>```xml
><!--自定义maven本地仓库地址-->
><localRepository>D:\hcr\dev\apps\test_nexus3\apache-maven-3.6.3-bin\apache-maven-3.6.3\resp</localRepository>
>```
>
>2. 新增servers的配置，指定发布版本账号密码
>
>```xml
><servers>
>    <server>  
>        <id>kingyifan-releases</id>  <!-- id一定唯一 -->
>        <username>admin</username>  
>        <password>admin123</password>  
>    </server>
>    <server>  
>        <id>kingyifan-snapshots</id>  
>        <username>admin</username>  
>        <password>admin123</password>  
>    </server>   
></servers>
>```

##### 9.2 获取release仓库和快照仓库的地址

<img src="https://tva1.sinaimg.cn/large/0081Kckwgy1glge2jjdadj315v0m9dhb.jpg" style="zoom:60%">

##### 9.3 项目pom文件新增

>```xml
><!--打包上传maven私服-->
><distributionManagement>
>    
>    <repository>
>        <!--id的名字可以任意取，但是在setting文件中的属性<server>的ID与这里一致-->
>        <id>kingyifan-releases</id>
>        <!--指向仓库类型为host(宿主仓库）的储存类型为Release的仓库-->
>        <url>http://192.168.189.129:12001/repository/kingyifan-hosted-release/</url>
>    </repository>
>    
>    <snapshotRepository>
>        <id>kingyifan-snapshots</id>
>        <url>http://192.168.189.129:12001/repository/kingyifan-hosted-snapshot/</url>
>    </snapshotRepository>
>    
></distributionManagement>
>```

##### 9.4 发布到私服

```shell
$ mvn deploy
```

>自动就打到快照版本了。
>
>Q：为什么自动到snapshot版本而不是release版本呢？
>
>A：因为创建项目的时候指定的版本号就是快照版本。
>
><img src="https://tva1.sinaimg.cn/large/0081Kckwgy1glgel4zxf2j30w002oq37.jpg" style="zoom:60%">

##### 9.5 如果只有jar包怎么上传私服

>1. 选择上传的仓库地址（只能选择发布版本）
>
>Upload —> maven-release
>
>2. 上传jar包并且命名
>
><img src="https://tva1.sinaimg.cn/large/0081Kckwgy1glgeqhyv39j30xr0jyt9u.jpg" style="zoom:60%">

#### 10. 本地使用私服环境并且从私服下载jar包

##### 10.1 配置maven的setting文件

>1. 增加综合仓库的服务器配置
>
>```xml
><server>
>    <id>nexus-kingyifan</id>
>    <username>admin</username>
>    <password>admin123</password>
></server>
>```
>
>2. 增加私服的综合仓库地址
>
>```xml
><mirror>
>    <id>nexus</id>
>    <name>internal nexus repository</name>
>    <!--镜像采用配置好的组的地址-->
>    <url>http://192.168.189.129:8001/nexus/repository/maven-public/</url>
>    <mirrorOf>!internal.repo,*</mirrorOf>
></mirror>
>```
>
>3. 配置仓库列表
>
>```xml
><profiles>
>    <profile>
>        <id>jdk-1.8</id>
>        <activation>
>            <activeByDefault>true</activeByDefault>
>            <jdk>1.8</jdk>
>        </activation>
>        <properties>
>            <maven.compiler.source>1.8</maven.compiler.source>
>            <maven.compiler.target>1.8</maven.compiler.target>
>            <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
>        </properties>
>    </profile>
>    <profile>
>        <id>nexus-kingyifan</id>
>        <!-- 远程仓库列表 -->
>        <repositories>
>            <repository>
>                <id>nexus-kingyifan</id>
>                <name>Nexus Central</name>
>                <!-- 虚拟的URL形式,指向镜像的URL-->
>                <url>http://192.168.189.129:12001/repository/kingyifan-group/</url>
>                <layout>default</layout>
>                <!-- 表示可以从这个仓库下载releases版本的构件--> 
>                <releases>
>                    <enabled>true</enabled>
>                </releases>
>                <!-- 表示可以从这个仓库下载snapshot版本的构件 --> 
>                <snapshots>
>                    <enabled>true</enabled>
>                </snapshots>
>            </repository>
>        </repositories>
>        <!-- 插件仓库列表 -->
>        <pluginRepositories>
>            <pluginRepository>
>                <id>nexus-kingyifan</id>
>                <name>Nexus Central</name>
>                <url>http://192.168.189.129:12001/repository/kingyifan-group/</url>
>                <layout>default</layout>
>                <snapshots>
>                    <enabled>true</enabled>
>                </snapshots>
>                <releases>
>                    <enabled>true</enabled>
>                </releases>
>            </pluginRepository>
>        </pluginRepositories>
>    </profile>
></profiles>
><activeProfiles>
>    <!--需要激活 <profile>中的ID才生效--> 
>    <activeProfile>nexus-kingyifan</activeProfile>
>    <activeProfile>jdk-1.8</activeProfile>
></activeProfiles>
>```

#### 11. 私服下载jar包路径

>本地仓库 =》私服发布版本 =》私服正式版本 =》私服代理仓库 =》直到寻找结束

