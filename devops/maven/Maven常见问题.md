#### Maven继承父工程时的relativePath标签解析

---

>父项目的pom.xml文件的相对路径。默认值为../pom.xml。maven首先从当前构建项目开始查找父项目的pom文件，然后从本地仓库，最有从远程仓库。RelativePath允许你选择一个不同的位置。
>
>
>如果默认../pom.xml 没找到父元素的pom ，不配置 relativePath 指向父项目的pom则会报错。

