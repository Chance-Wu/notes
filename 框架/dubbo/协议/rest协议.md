基于标准的Java REST API——JAX-RS 2.0（Java API for RESTful Web Services的简写）实现的REST调用支持。

#### 1. 示例

在dubbo中开发一个REST风格的服务会比较简单，下面以一个注册用户的简单服务为例说明。

这个服务要实现的功能是提供如下URL（注：这个URL不是完全符合REST的风格，但是更简单实用）：

```fallback
http://localhost:8080/users/register
```

而任何客户端都可以将包含用户信息的JSON字符串POST到以上URL来完成用户注册。

首先，开发服务的接口：

```java
public interface UserService {    
   void registerUser(User user);
}
```

然后，开发服务的实现：

```java
@Path("/users")
@DubboService // 声明需要暴露的服务接口
public class UserServiceImpl implements UserService {
       
    @POST
    @Path("/register")
    @Consumes({MediaType.APPLICATION_JSON})
    public void registerUser(User user) {
        // save the user...
    }
}
```

面的实现非常简单，但是由于该 REST 服务是要发布到指定 URL 上，供任意语言的客户端甚至浏览器来访问，所以这里额外添加了几个 JAX-RS 的标准 annotation 来做相关的配置。

`@Path("/users")`：指定访问UserService的URL相对路径是/users，即http://localhost:8080/users

`@Path("/register")`：指定访问registerUser()方法的URL相对路径是/register，再结合上一个@Path为UserService指定的路径，则调用UserService.register()的完整路径为http://localhost:8080/users/register

`@POST`：指定访问registerUser()用HTTP POST方法

`@Consumes({MediaType.APPLICATION_JSON})`：指定registerUser()接收JSON格式的数据。==REST框架会自动将JSON数据反序列化为User对象。==

最后，添加配置，即完成所有服务开发工作：

```yaml
# 用rest协议在8080端口暴露服务
dubbo:
  protocol:
    name: rest
    port: 8080
```

#### 2. REST服务提供端详解

REST服务中虽然建议使用HTTP协议中四种标准方法POST、DELETE、PUT、GET来分别实现常见的“增删改查”，但实际中，我们==一般情况直接用POST来实现“增改”，GET来实现“删查”即可==（DELETE和PUT甚至会被一些防火墙阻挡）。

前面已经简单演示了POST的实现，在此，我们为UserService添加一个获取注册用户资料的功能，来演示GET的实现。

这个功能就是要实现客户端通过访问如下不同URL来获取不同ID的用户资料：

```fallback
http://localhost:8080/users/1001
http://localhost:8080/users/1002
http://localhost:8080/users/1003
```

当然，也可以通过其他形式的URL来访问不同ID的用户资料，例如：

```fallback
http://localhost:8080/users/load?id=1001
```

JAX-RS本身可以支持所有这些形式。但是上面那种在URL路径中包含查询参数的形式（http://localhost:8080/users/1001） 更符合REST的一般习惯，所以更推荐大家来使用。下面我们就为UserService添加一个getUser()方法来实现这种形式的URL访问：

```java
@GET
@Path("/{id : \\d+}")
@Produces({MediaType.APPLICATION_JSON})
public User getUser(@PathParam("id") Long id) {
    // ...
}
```

@GET：指定用HTTP GET方法访问

@Path("/{id : \d+}")：根据上面的功能需求，访问getUser()的URL应当是“http://localhost:8080/users/ + 任意数字"，并且这个数字要被做为参数传入getUser()方法。 这里的annotation配置中，@Path中间的{id: xxx}指定URL相对路径中包含了名为id参数，而它的值也将被自动传递给下面用@PathParam(“id”)修饰的方法参数id。{id:后面紧跟的\d+是一个正则表达式，指定了id参数必须是数字。

@Produces({MediaType.APPLICATION_JSON})：指定getUser()输出JSON格式的数据。框架会自动将User对象序列化为JSON数据。

















































