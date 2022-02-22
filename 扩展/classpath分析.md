#### 1. 什么是classpath

---

classpath相当于Java执行环境，它指定了一些常用的包或jar的位置，方便我们对项目文件的使用，而不必重复多次写所需要文件的位置。

在classpath中可能需要使用所有相同名字的资源文件，如果用`classpath:`只会加载第一个，而使用`classpath*:前缀`则能够==加载所有符合类型的文件==。使用classpath*:需要遍历所有的classpath，加载速度很慢，因此您应该尽量避免使用classpath*。

