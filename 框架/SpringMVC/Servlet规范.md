![](https://tva1.sinaimg.cn/large/008i3skNgy1gsxlpy4en3j31160l5whs.jpg)



![](https://tva1.sinaimg.cn/large/008i3skNgy1gsxltixrgmj31xm0u0n1e.jpg)



#### 1. Servlet规范介绍

---

在Servlet规范中

1. 指定【动态资源文件】开发步骤；
2. 指定http服务器调用动态资源文件规则；
3. 指定http服务器管理动态资源文件实例对象规则。



#### 2. Servlet接口实现类

---

1. Servlet接口来自于Servlet规范下一个接口，这个接口存在http服务器提供的jar包；
2. Tomcat服务器下lib文件有一个==servlet-api.jar==存放Servlet接口（javax.servlet.Servlet接口）；
3. Servlet规范中任务，==HTTP服务器能调用的【动态资源文件】必须是一个Servlet接口实现类==。

>例如：
>
>```java
>class Student{
>  //不是动态资源文件，Tomcat无权调用
>}
>
>class Teacher implements Servlet{
>  //合法动态资源文件，Tomcat有权利调用
>
>  Servlet obj = new Teacher();
>  obj.doGet()
>}
>```



#### 3. Servlet接口实现类开发步骤

---

1. 创建一个Java类继承于HttpServlet父类，使之成为一个Servlet接口实现类；

2. 重写HttpServlet父类两个方法。doGet或者doPost；

   - 浏览器 ---------> oneServlet.doGet()
   - 浏览器 ---------> oneServlet.doPost()

3. ==将Servlet接口实现类信息【注册】到Tomcat服务器==

   【网站】-------->【web】-------->【WEB-INF】-------> web.xml

   ==将Servlet接口实现类类路径地址交给Tomcat==：

   ```xml
   <servlet>
     <servlet-name>mm</servlet-name>   <!--声明一个变量存储servlet接口实现类类路径-->
     <servlet-class>com.bjpowernode.controller.OneServlet</servlet-class> <!--声明servlet接口实现类类路径-->
   </servlet>
   
   Tomcat String mm = "com.bjpowernode.controller.OneServlet"
   
   <!--为了降低用户访问Servlet接口实现类难度，需要设置简短请求别名-->
   <servlet-mapping>
     <servlet-name>mm</servlet-name>
     <url-pattern>/one</url-pattern> <!--设置简短请求别名,别名在书写时必须以"/"为开头-->
   </servlet-mapping>
   ```

   如果现在浏览器向Tomcat索要OneServlet时地址：http://localhost:8080/myWeb/one

   ```xml
   <servlet>
     <servlet-name>TwoServlet</servlet-name>
     <servlet-class>com.rzpt.controller.TwoServlet</servlet-class>
   </servlet>
   <servlet-mapping>
     <servlet-name>TwoServlet</servlet-name>
     <url-pattern>two</url-pattern>
   </servlet-mapping>
   ```



#### 4. HttpServletResponse接口

---

1. HttpServletResponse接口来自于Servlet规范中，在Tomcat中存在servlet-api.jar；

2. HttpServletResponse接口实现类由Http服务器负责提供；

3. HttpServletResponse接口负责将doGet/doPost方法执行结果写入到【响应体】交给浏览器；

4. 开发人员习惯于将HttpServletResponse接口修饰的对象称为【响应对象】

   ```java
   String result ="Hello World"; //执行结果
   
   //--------响应对象将结果写入到响应体--------------start
   
   //1.通过响应对象，向Tomcat索要输出流
   PrintWriter out = response.getWriter();
   //2.通过输出流，将执行结果以二进制形式写入到响应体
   out.write(result);
   
   //--------响应对象将结果写入到响应体--------------start
   }//doGet执行完毕
   //Tomcat将响应包推送给浏览器
   ```

   ```java
   /*
   *  问题描述： 浏览器接收到数据是2 ，不是50
   *
   *  问题原因:
   *            out.writer方法可以将【字符】，【字符串】，【ASCII码】写入到响应体
   *
   *            【ASCII码】：  a -------------- 97
   *
   *                          2--------------- 50
   *
   *  问题解决方案:  实际开发过程中，都是通过out.print()将真实数据写入到响应体
   *
   */
   protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
   
     int money = 50; //执行结果
   
     PrintWriter out = response.getWriter();
     //out.write(97);
     out.print(money);
   
   }



主要功能：

1. 将执行结果以二进制形式写入到【响应体】；
2. 设置响应头中[content-type]



























































































- 在servlet的规范当中，servlet容器或者叫web容器，如tomcat，中运行的每个应用都由一个ServletContext表示，在web容器中可以包含多个ServletContext，即可以有多个web应用在web容器中运行。如在tomcat的webapp目录下，每个war包都对应一个web应用，tomcat启动时会解压war包，并启动相关的应用。
- 在web容器启动的时候，会初始化web应用，即创建ServletContext对象，加载解析web.xml文件，获取该应用的Filters，Listener，Servlet等组件的配置并创建对象实例，作为ServletContext的属性，保存在ServletContext当中。之后web容器接收到客户端请求时，则会根据请求信息，匹配到处理这个请求的Servlet，同时在交给servlet处理之前，会先使用应用配置的Filters对这个请求先进行过滤，最后才交给servlet处理。
- 了解web容器启动，之后接受客户端请求这些知识有啥用处呢？这里我们需要回过头来看我们的spring项目。我们在日常开发中，直接接触的是spring相关的组件，然后打成war包，放到web容器中，如拷贝到tomcat的webapp目录，并不会直接和web容器打交道。经过以上的分析，其实一个spring项目就是对应web容器里的一个ServletContext，所以在ServletContext对象的创建和初始化的时候，就需要一种机制来触发spring相关组件的创建和初始化，如包含@Controller和@RequestMapping注解的类和方法，这样才能处理请求。



会员生日事件、生日发券发生异常、会员升降级事件营销、会员升降级发券、会员首次完善资料事件营销、注册发券





































