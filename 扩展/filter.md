#### 1. filter 简介

---

filter是Servlet规范里的一个高级特性，只==用于对request、response进行修改==。

filter提出了 `FilterChain` 的概念，==客户端请求request在抵达servlet之前都会经过filterChain里的所有fiter==，如图所示：

<img src="img/2279594-166a5c28d392f743.png" alt="filterchain工作原理" style="zoom: 67%;" />



#### 2. filter 生命周期

---

在web.xml中配置filter，当启动服务器时会实例化，并且会初始化，当有网络请求时会进行过滤操作，当服务器关闭时，会进行销毁，全过程如下图所示：

![filter生命周期](img/2279594-0bfdef229be804e4.png)

#### 3. 编写filter

---

filter类需实现javax.servlet.Filter接口，需复写里面的三个方法，其中init(),在初始化时调用；doFiler()方法每次都会调用，在这个方法中一定要执行chain.doFilter(),否则request不会交给后面的filter或者servler;ondestroy()在关闭服务器时调用。

```java
@Component
@WebFilter(urlPatterns = "/*", filterName = "firstFilter")
public class FirstFilter implements Filter {

  @Override
  public void init(FilterConfig filterConfig) throws ServletException {
    System.out.println("init---FirstFilter");
  }

  @Override
  public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
    System.out.println("start----doFilter--FirstFilter");
    chain.doFilter(request, response);
    System.out.println("end------doFilter--FirstFilter");
  }

  @Override
  public void destroy() {
    System.out.println("destroy---FirstFilter");
  }
}
```

urlPattern可以配置多个，也可以用通配符，当访问满足路径匹配，并且符合dispatcher时，request会被filter拦截进行处理，处理完后的response再次被filter拦截，可以进行处理。

dispatcherTypes 默认`REQUEST`。

- REQUEST：请求时有效
- FORWARD：当某servlet通过forward到该servlet才有效
- INCLUDE：jsp通过< jsp: incluser/> 请求servlet有效
- ERROR：< %@page errorPage=”” % >有效



#### 4. 防盗链

---

filter的特性使它可以处理特殊的工作，例如防盗链，字符编码的处理，日志记录，数据加密，过滤一些黑词等等。

例如： 防盗链图片，当其他网站请求本网站图片资源时显示错误的图片，只有本应用先生的图片才显示正确的图片，代码如下：

```java
@Component
@WebFilter(urlPatterns = "/images/*", filterName = "imageFilter")
public class ImageFilter implements Filter{
  public void init(FilterConfig config) throws Exception(){

  }
  public void doFilter(ServletRequest req,ServletResponse res,FilterChain chain)throws Exception{
    HttpServletRequest request=(HttpServletRequest )req;
    HttpServletResponse  response=(HttpServletResponse)res;
    String referer=request.getHeader("referer");
    if(referer==null||!referer.contains(request.getServerName())){
      request.getRequestDispatcher("/error.png").forwar(request,response);
    }else{
      chain.doFilter(request,response);
    }
  }
  public void destroy(){}

}
```

当访问images下的所有图片会经过该filter，根据访问头信息，如果说本站点的访问则显示正确图片，否则先生错误图片。



#### 5. 字符编码

---

```javascript
@WebFilter(urlPatterns = "/*", filterName = "CharsetFilter",
           initParams = {@WebInitParam(name = "characterEncoding", value = "UTF-8"),
@WebInitParam(name = "enabled", value = "true")})
public class CharsetFilter implements Filter{
  private String characterEncoding;
  private String enabled;
  public void init(FilterConfig config) throws Exception(){
    characterEncoding=config.getInitParameter("characterEncoding");
    enabled=config.getInitParameter("enabled").equals("true");
  }
  public void doFilter(ServletRequest req,ServletResponse res,FilterChain chain)throws Exception{
    if(enabled|| characterEncoding!=null){
      req.setCharacterEncoding(characterEncoding);
      res.setCharacterEncoding(characterEncoding);
    }
    chain.doFilter(req,res);
  }

  public void destroy(){
    characterEncoding=null;
  }
}
```



>另外，还有比较常见的日志记录filter、异常捕捉filter、权限校验、内容替换filter等等。
>
>filter有很大的弹性机制，功能强大，而且跟servlet、jsp没耦合.filter是现在面向切面编程aop的一种思想体现，它能够胜任很多工作。
>
>2.5的fiter需要在web.xml中配置，执行顺序按照配置顺序，另外3.0可以用注解的方式配置filter，此时没有配置的顺序。