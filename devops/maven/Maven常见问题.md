### Maven继承父工程时的relativePath标签解析

---

父项目的pom.xml文件的相对路径。默认值为`../pom.xml`。maven首先从当前构建项目开始查找父项目的pom文件，然后从本地仓库，最后从远程仓库。RelativePath允许你选择一个不同的位置。

**如果默认../pom.xml 没找到父元素的pom ，不配置 relativePath 指向父项目的pom则会报错**。

#### 原因分析

因为在项目文件夹的外层包含着另一个项目，此时项目文件无法确定该文件的pom依赖是引用哪一个parent依赖导致的。推荐使用方法二解决。



#### 解决方法一

在pom文件中parent依赖中添加 `<relativePath/>`属性。

```xml
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-parent</artifactId>
  <version>2.3.12.RELEASE</version>
  <relativePath/>
</parent>
```

#### 解决方法二

1. 内部检查：检查是否在文件夹的内部是否存在直接引用spring-boot-starter-parent依赖的项目 并将其修改为依赖父项目文件。
2. 外部检查：文件夹的外部存在另一个包含项目文件的项目，若存在删除其POM文件即可`。`











