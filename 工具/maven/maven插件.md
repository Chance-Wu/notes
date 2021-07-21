maven本质是一个插件框架，核心并不执行任何具体的构建任务，所有这些任务都交给插件完成。



#### 1. maven概念模型

---

maven包含一个==项目对象模型==，一组标准集合，一个==项目生命周期==，一个==依赖管理系统==和用来运行定义在生命周期阶段中插件目标逻辑。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gsm8fu33g0j309h091aa7.jpg)



#### 2. maven生命周期

---

三个标准的生命周期：

- clean：项目清理的处理
- default（或build）：项目部署的处理
- site：项目站点文档创建的处理

每个生命周期中都包含着一系列的阶段。这些阶段相当于maven提供的统一接口，然后这些阶段的实现由maven的插件完成。比如命令`mvn clean`中，clean对应的就是Clean生命周期中的clean阶段。但是clean阶段的具体操作是由`maven-clean-plugin`来实现的。

>maven实际上是一个依赖插件执行的框架，其插件通常被用来：
>
>- 创建jar文件
>- 创建war文件
>- 编译代码文件
>- 代码单元测试
>- 创建工程文档
>- 创建工程报告



#### 3. 插件

插件通常提供一个目标的集合，可以使用下面的语法执行：

`mvn plugin:goal-name`

如编译命令：==mvn compiler:compile==

**maven提供2种类型的插件：**

| 类型              | 描述                                            |
| ----------------- | ----------------------------------------------- |
| build plugins     | 在构件时执行，并在pom.xml的元素中配置。         |
| reporting plugins | 在网站生成过程中执行，并在pom.xml的元素中配置。 |

**常用插件列表：**

| 插件                           | 描述                               |
| ------------------------------ | ---------------------------------- |
| maven-clean-plugin             | 构建之后清理目标文件，删除目标文件 |
| maven-compiler-plugin          | 编译Java源文件                     |
| maven-surefile-plugin          | 运行junit测试，创建测试报告        |
| maven-jar-plugin               | 从当前工程中构建jar文件            |
| maven-war-plugin               | 从当前工程中构建war文件            |
| maven-deploy-plugin            |                                    |
| maven-install-plugin           |                                    |
| maven-resource-plugin          | 实现资源文件过滤                   |
| mybatis-generator-maven-plugin | 自动生成pojo mapper xml文件        |
|                                |                                    |