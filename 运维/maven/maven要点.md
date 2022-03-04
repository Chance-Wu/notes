#### 1. 简介

---

Maven 是一个**项目管理工具**，它包含了一个项目对象模型(Project Object Model)，反映在配置中，就是一个 pom.xml 文件。是一组标准集合，一个项目的**生命周期**、一个**依赖管理系统**，另外还包括定义 在项目生命周期阶段的**插件(plugin)**以及**目标(goal)**。

通过一个自定义的**项目对象模型**，pom.xml 来详细描述我们自己的项目。

Maven 中的有两大核心：

- 依赖管理：对 jar 的统一管理（Maven 提供了一个 Maven 的中央仓库，https://mvnrepository.com/，当我们在项目中添加完依赖之后，Maven 会自动去中央仓库下载相关的依赖，并且解决依赖的依赖问题）
- 项目构建：对项目进行<u>编译</u>、<u>测试</u>、<u>打包</u>、<u>部署</u>、<u>上传到私服</u>等



#### 2. maven安装配置

---

`mvn -v`

##### 2.1 仓库类型

| 仓库类型 | 说明                                                   |
| -------- | ------------------------------------------------------ |
| 本地仓库 | 个人电脑上的仓库，默认位置在当 前用户名\.m2\repository |
| 私服仓库 | 公司内部搭建的Maven私服，出于局域网中，访问速度较快    |
| 中央仓库 | 有Apache团队来维护，包含了大部分的jar                  |

现在存在3个仓库，那么jar包如何查找呢？

![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzxwut1iooj210008at91.jpg)

>注：
>
>1. 在找寻的过程中，如果发现该仓库有镜像设置，则用镜像的地址代替，例如现在进行到要在respository A仓库中查找某个依赖，但仓库配置了mirror，则会转到从A的mirror中查找该依赖，不会在从A中查找。
>2. settings.xml中配置的profile（激活的）下的respository优先级高于项目中pom文件配置的respository。
>3. ==如果仓库的id设置成“central”，则该仓库会覆盖maven默认的中央仓库配置==。

##### 2.2 本地仓库配置

本地仓库默认位置在 当前用戶名\.m2\repository ，这个位置可以自定义，但是不建议大家自定义这个地址。

##### 2.3 远程镜像配置

由于默认的中央仓库下载较慢，因此，可以将远程仓库地址改为阿里巴巴的仓库地址：

```xml
<mirror>
  <id>nexus-aliyun</id>
  <mirrorOf>central</mirrorOf>
  <name>Nexus aliyun</name>
  <url>http://maven.aliyun.com/nexus/content/groups/public</url>
</mirror>
```



#### 3. 常用命令

---

| 命令        | 含义 | 说明                                     |
| ----------- | ---- | ---------------------------------------- |
| mvn clean   | 清理 | 清理已经编译好的文件                     |
| mvn compile | 编译 | 将Java代码编译成Class文件                |
| mvn test    | 测试 | 项目测试                                 |
| mvn package | 打包 | 根据用户配置，将项目打包成jar包或者war包 |
| mvn install | 安装 | 手动向本地仓库安装一个jar                |
| mvn deploy  | 上传 | 将jar上传到私服                          |

>注：这些命令不是独立运行的，有一个顺序。
>
>如：将 jar 上传到私服，那么就要构建 jar，就需要执行 `mvn package` 命令，要打包，当然也需要测试，那就要走 `mvn test` 命令，要测试就要先编译 `mvn compile`.....，因此，最终所有的命令都会执行一遍。不过，开发者也可以手动配置跳过某一个命令。一般来是，除了测试，其他步骤都不建议跳过。
>
>在执行命令的后面，添加参数`-Dmaven.test.skip=true`或者`-DskipTests=true`

##### 3.1 对项目进行打包

通过 `mvn package` 命令可以将项目打成一个 jar 包。

在打包之前，需要配置 JDK 的版本至少为 7 以上，因此，我们还需要手动修改一下 pom.xml 文件，即添加如下配置：

```xml
<properties>
  <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  <maven.compiler.encoding>UTF-8</maven.compiler.encoding>
  <java.version>1.8</java.version>
  <maven.compiler.source>1.8</maven.compiler.source>
  <maven.compiler.target>1.8</maven.compiler.target>
</properties>
```

执行命令时，命令行要定位到 pom.xml 文件所在的目录。

##### 3.2 将项目安装到本地仓库

如果需要将项目安装到本地仓库，可以直接执行 mvn install 命令，注意，mvn install 命令会包含 mvn package 过程。



#### 4. maven生命周期

---

三种生命周期
1）Clean生命周期
Clean
2）Default生命周期
Compile   test  package  install  deploy
3）Site生命周期
Site

命令和生命周期的阶段的关系：不同的生命周期的命令可以同时执行（mvn clean package）



#### 5. Maven依赖管理

---

Maven 项目，如果需要使用第三方的控件，都是通过依赖管理来完成的。这里用到的一个东西就是 pom.xml 文件，概念叫做**项目对象模型(POM，Project Object Model)**，我们在 pom.xml 中定义了 Maven 项目的形式，pom.xml 相当于是 Maven 项目的一个地图。涉及如下内容：

##### 5.1 Maven坐标

```xml
<dependencies>
  <dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13.1</version>
    <scope>test</scope>
  </dependency>
</dependencies>
```

- dependencies

  在 dependencies 标签中，添加项目需要的 jar 所对应的 maven 坐标。

- dependency

  一个 dependency 标签表示一个坐标

- groupId

  团体、公司、组织机构等等的唯一标识。团体标识的约定是它以创建这个项目的组织名称的逆向域名(例如 org.javaboy)开头。一个 Maven 坐标必须要包含 groupId。一些典型的 groupId 如 apache 的 groupId 是 org.apache.

- artifactId

  artifactId 相当于在一个组织中项目的唯一标识符。

- version

  一个项目的版本。一个项目的话，可能会有多个版本。如果是正在开发的项目，我们可以给版本号加上一个 SNAPSHOT，表示这是一个快照版(新建项目的默认版本号就是快照版)

- scope

  表示依赖范围。

![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzxxp86sesj20zq0b6aay.jpg)

最典型的有两个：

- **数据库驱动**，在使用的过程中，我们自己写代码，写的是 JDBC 代码，==只有在项目运行时，才需要执行 MySQL 驱动中的代码==。所以，MySQL 驱动这个依赖在添加到项目中之后，可以设置它的 scope 为 `runtime`，编译的时候不生效。
- **单元测试**，只在测试的时候生效，所以可以设置它的 scope 为 test，这样，当项目打包发布时，单元测试的依赖就不会跟着发布。

##### 5.2 依赖冲突

依赖冲突产生原因：

<img src="https://tva1.sinaimg.cn/large/e6c9d24egy1gzxxs93r0rj20jm08mjrw.jpg" style="zoom: 75%;" />

在图中，a.jar 依赖 b.jar，同时 a.jar 依赖 d.jar，这个时候，a 和 b、d 的关系是直接依赖的关系，a 和 c 的关系是间接依赖的关系。

>依赖冲突解决办法：
>
>1. 默认行为
>
>   1. 先定义先使用
>
>   2. 路径最近原则（直接声明使用）
>
>      以 spring-context 为例，下图中 x 表示失效的依赖(优级低的依赖，即路径近的依赖优先使用)
>
>      ![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzxxx8wec2j20p80l6tam.jpg)
>
>2. 手动排除依赖
>
>   ```xml
>   <dependency> 
>     <groupId>org.springframework</groupId>
>     <artifactId>spring-context</artifactId>
>     <version>5.1.9.RELEASE</version>
>     <exclusions>
>       <exclusion>
>         <groupId>org.springframework</groupId>
>         <artifactId>spring-core</artifactId>
>       </exclusion>
>     </exclusions>
>   </dependency>
>   ```
>
>   表示从 spring-context 中排除 spring-core 依赖。

4. 版本锁定

   ```xml
   <dependencyManagement>
     <dependencies>
       <dependency>
         <groupId>org.springframework</groupId>
         <artifactId>spring-beans</artifactId>
         <version>4.2.4.RELEASE</version>
       </dependency>
     </dependencies>
   </dependencyManagement>
   ```



#### 7. 分模块开发

---

1. 创建maven父模块

   ```xml
   <groupId>com.chance</groupId>
   <artifactId>demo-parent</artifactId>
   <packaging>pom</packaging>
   <version>1.0-SNAPSHOT</version>
   <modules>
   	<module>demo-domain</module>
   </modules>
   ```

   打包方式为pom，将demo-parent发布到本地仓库——`mvn install`

2. 创建maven子模块

   ```xml
   <parent>
     <groupId>com.chance</groupId>
     <artifactId>demo-parent</artifactId>
     <version>1.0-SNAPSHOT</version>
   </parent>
   
   <packaging>jar</packaging>
   
   <artifactId>demo-domain</artifactId>
   ```

   打包方式为jar。

