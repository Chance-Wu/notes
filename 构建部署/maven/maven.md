#### 1. 简介

---

Maven是apache下的一个开源项目，只是用来==管理java项目==的。

分析：maven项目为什么这么小？没有jar。 需要jar吗？肯定需要。没有存在于maven项目里面，jar存在于哪？

> 优点：
>
> 1. 依赖管理 
>    对jar包的统一管理  可以节省空间
> 2. 项目一键构建
>    一个tomcat:run就能把项目运行起来
>    Maven能做的事：编译、测试(junit)、运行、打包、部署
> 3. 跨平台
> 4. 应用于大型项目，提高开发效率



#### 2. maven安装配置

---

mvn -v

> maven仓库（三种仓库）
>
> 1. 本地仓库【自己维护】
>    修改settings.xml文件就可以
>    `<localRepository>E:\repository</localRepository>`
> 2. 远程仓库（私服）【公司维护】
> 3. 中央仓库【maven团队维护】



#### 3. 常用命令

---

```
mvn clean		清理编译的文件
mvn compile		编译了主目录的文件
mvn test		编译并运行了test目录的代码
mvn package		打包
mvn install		就是把项目发布到本地仓库
mvn tomcat:run	一键启动
```



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



#### 5. 四个依赖范围

---

compile		struts2框架
provided		jsp-api.jar
runtime		数据库驱动包
test		junit.jar



#### 6. 版本冲突解决

---

1. 路径近者优先原则
  自己添加一个依赖

2. 第一声明者优先原则
  哪个依赖在前用哪个

3. 排除原则

  ```xml
  <!-- 排除 -->
  <exclusions>
    <exclusion>
      <groupId>org.springframework</groupId>
      <artifactId>spring-beans</artifactId>
    </exclusion>
  </exclusions>
  ```

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