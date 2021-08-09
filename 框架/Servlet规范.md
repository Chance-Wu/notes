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

2. 设置响应头中【content-type】属性值，从而控制浏览器使用对应编译器将响应体二进制数据编译为【文字、图片、视频、命令】

   ```java
   //设置响应头content-type
   response.setContentType("text/html;charset=utf-8");
   ```

3. 设置响应头中【location】属性，将一个请求地址赋值给location。从而控制浏览器向指定服务器发送请求。

   ```java
   String result ="http://www.baidu.com?userName=mike";
   
   //通过响应对象，将地址赋值给响应头中location属性
   response.sendRedirect(result);//[响应头  location="http://www.baidu.com"]
   }
   /*
    *  浏览器在接收到响应包之后，如果
    *  发现响应头中存在location属性
    *  自动通过地址栏向location指定网站发送请求
    *
    *  sendRedirect方法远程控制浏览器请求行为【请求地址，请求方式，请求参数】
    */
   ```



#### 5. HttpServletRequest接口

---

1. HttpServletRequest接口来自于Servlet规范中，在Tomcat中存在servlet-api.jar；

2. HttpServletRequest接口实现类由Http服务器负责提供；

3. HttpServletRequest接口负责在doGet/doPost方法运行时读取Http请求协议包中信息。

   ```java
   //post通知请求对象，使用utf-8字符集对请求体二进制内容进行一次重写解码
   request.setCharacterEncoding("utf-8");
   ```

   ```java
   //1.通过请求对象获得【请求头】中【所有请求参数名】
   Enumeration paramNames =request.getParameterNames(); //将所有请求参数名称保存到一个枚举对象进行返回
   while(paramNames.hasMoreElements()){
     String paramName = (String)paramNames.nextElement();
     //2.通过请求对象读取指定的请求参数的值
     String value = request.getParameter(paramName);
     System.out.println("请求参数名 "+paramName+" 请求参数值 "+value);
   }
   ```



作用：

1. 可以读取Http请求协议包中【请求行】信息；

2. 可以读取保存在Http请求协议包中【请求头】或者【请求体】中请求参数信息；

3. 可以代替浏览器向Http服务器申请资源文件调用。

   ```java
   //1.通过请求对象，读取【请求行】中【url】信息
   String url = request.getRequestURL().toString();
   //2.通过请求对象，读取【请求行】中【method】信息
   String method = request.getMethod();
   
   //3.通过请求对象，读取【请求行】中uri信息
   /*
    * URI：资源文件精准定位地址，在请求行并没有URI这个属性。
    *      实际上URL中截取一个字符串，这个字符串格式"/网站名/资源文件名"
    *      URI用于让Http服务器对被访问的资源文件进行定位
    */
   String uri =  request.getRequestURI();// substring
   
   System.out.println("URL "+url);
   System.out.println("method "+method);
   System.out.println("URI "+uri);
   ```



#### 6. 请求对象和响应对象生命周期

---

1. 在Http服务器接收到浏览器发送的【Http请求协议包】之后，自动为当前的【Http请求协议包】生成一个【请求对象】和一个【响应对象】；

2. 在Http服务器调用doGet/doPost方法时，负责将【请求对象】和【响应对象】作为实参传递到方法，确保doGet/doPost正确执行；

3. 在Http服务器准备推送Http响应协议包之前，负责将本次请求关联的【请求对象】和【响应对象】销毁。

   【请求对象】和【响应对象】生命周期贯穿一次请求的处理过程中

   【请求对象】和【响应对象】相当于用户在服务端的代言人



#### 7. 欢迎资源文件

---

1. 前提：用户可以记住网站名，但是不会记住网站资源文件名；

2. 默认欢迎资源文件：用户发送了一个针对某个网站的【默认请求】时，此时由Http服务器自动从当前网站返回的资源文件

   正常请求： http://localhost:8080/myWeb/index.html

   默认请求： http://localhost:8080/myWeb/

3. Tomcat对于默认欢迎资源文件定位规则

   - 规则位置：Tomcat安装位置/conf/web.xml

   - 规则命令：

     ```xml
     <welcome-file-list>
       <welcome-file>index.html</welcome-file>
       <welcome-file>index.htm</welcome-file>
       <welcome-file>index.jsp</welcome-file>
     </welcome-file-list>
     ```

4. 设置当前网站的默认欢迎资源文件规则

   - 规则位置：网站/web/WEB-INFO/web.xml

   - 规则命令：

     ```xml
     <welcome-file-list>
       <welcome-file>login.html</welcome-file>
     </welcome-file-list>
     ```

   - ==网站设置自定义默认文件定位规则，此时Tomcat自带定位规则将失效==。



#### 8. 状态码

---

> 1XX：
>
> 通知浏览器本次返回的资源文件，并不是一个独立的资源文件，需要浏览器在接收
> 响应包之后，继续向Http服务器索要依赖的其他资源文件。

>2XX：
>
>通知浏览器本次返回的资源文件是一个，完整独立资源文件，浏览器在接收到之后不需要索要
>其他关联文件。

>3XX：
>
>通知浏览器本次返回的不是一个资源文件内容而是一个资源文件地址，需要浏览器根据这个地址自动发起请求来索要这个资源文件。
>
>response.sendRedirect("资源文件地址")写入到响应头中location，而这个行为导致Tomcat将302状态码写入到状态行。

>4XX：
>
>404: 通知浏览器，由于在服务端没有定位到被访问的资源文件
>因此无法提供帮助
>
>405：通知浏览器，在服务端已经定位到被访问的资源文件（Servlet）
>但是这个Servlet对于浏览器采用的请求方式不能处理

>5XX：
>
>500：通知浏览器，在服务端已经定位到被访问的资源文件（Servlet）
>这个Servlet可以接收浏览器采用请求方式，但是Servlet在处理
>请求期间，由于Java异常导致处理失败。



#### 9. 多个Servlet之间调用规则

---

1. 前提条件：
   某些来自于浏览器发送请求，往往需要服务端中多个Servlet协同处理。
   但是浏览器一次只能访问一个Servlet。

2. 提高用户使用感受规则：

　　无论本次请求涉及到多少个Servlet，用户只需要【手动】通知浏览器发起一次请求即可。

3. 多个Servlet之间调用规则：
   - 重定向解决方案
   - 请求转发解决方案

>**重定向**
>
>用户第一次通过【手动方式】通知浏览器访问OneServlet。
>OneServlet工作完毕后，将TwoServlet地址写入到响应头`location属性`中，导致Tomcat将302状态码写入到状态行。
>
>在浏览器接收到响应包之后，会读取到302状态。此时浏览器自动根据响应头中location属性地址发起第二次请求，访问TwoServlet去完成请求中剩余任务。
>
>```java
>System.out.println("OneServlet 负责 洗韭菜");
>//重定向解决方案:
>//response.sendRedirect("/myWeb/two");// [地址格式: /网站名/资源文件名]
>response.sendRedirect("http://www.baidu.com");
>```
>
>特点：
>
>1. 请求地址
>既可以把当前网站内部的资源文件地址发送给浏览器 （/网站名/资源文件名）
>也可以把其他网站资源文件地址发送给浏览器(http://ip地址:端口号/网站名/资源文件名)
>
>2. 请求次数
>
>   浏览器至少发送两次请求，但是只有第一次请求是用户手动发送。后续请求都是浏览器自动发送的。
>
>3) 请求方式
>重定向解决方案中，通过地址栏通知浏览器发起下一次请求，因此
>通过重定向解决方案调用的资源文件接收的请求方式一定是`【GET】`。
>
>4. 缺点
>重定向解决方案需要在浏览器与服务器之间进行多次往返，大量时间
>消耗在往返次数上，增加用户等待服务时间。

>**请求转发**
>
>用户第一次通过手动方式要求浏览器访问OneServlet。OneServlet工作完毕后，==通过当前的请求对象代替浏览器向Tomcat发送请求==，申请调用TwoServlet。Tomcat在接收到这个请求之后，自动调用TwoServlet来完成剩余任务。
>
>```java
>System.out.println("OneServlet 实施麻醉。。。。。");
>//请求转发方案：
>//1.通过当前请求对象生成资源文件申请报告对象
>RequestDispatcher report = request.getRequestDispatcher("/two");
>//2.将报告对象发送给Tomcat
>report.forward(request, response);
>```
>
>特点：
>
>1. 请求次数
>
>   在请求转发过程中，浏览器只发送一次请求
>
>2. 请求地址
>   只能向Tomcat服务器申请调用当前网站下资源文件地址
>   request.getRequestDispathcer("/资源文件名") ****不要写网站名****
>
>3. 请求方式
>
>   在请求转发过程中，浏览器只发送一个了个Http请求协议包。参与本次请求的所有Servlet共享同一个请求协议包，因此这些Servlet接收的请求方式与浏览器发送的请求方式保持一致.



#### 10. 多个Servlet之间数据实现方案

---

1. 数据共享：OneServlet工作完毕后，将产生数据交给TwoServlet来使用
2. Servlet规范中提供四种数据共享方案
   1. ==ServletContext接口==
   2. ==Cookie类==
   3. ==HttpSession接口==
   4. ==HttpServletRequest接口==

>**ServletContext接口**
>
>1. 来自于Servlet规范中一个接口。在Tomcat中存在servlet-api.jar
>   在Tomcat中负责提供这个接口实现类。
>2. 如果两个Servlet来自于同一个网站。彼此之间通过网站的ServletContext
>   实例对象实现数据共享。
>3. ServletContext对象称为==【全局作用域对象】==。
>
>原理：
>
>每一个网站都存在一个全局作用域对象。这个全局作用域对象相当于一个Map。在这个网站中OneServlet可以将一个数据存入到全局作用域对象，当前网站中其他Servlet此时都可以从全局作用域对象得到这个数据进行使用。
>
>ServeltContext对象生命周期：
>
>1. 在Http服务器启动过程中，自动为当前网站在内存中创建一个全局作用域对象；
>2. 在Http服务器运行期间时，一个网站只有一个全局作用域对象；
>3. 在Http服务器运行期间，全局作用域对象一直处于存活状态；
>4. 在Http服务器准备关闭时，负责将当前网站中全局作用域对象进行销毁处理。
>
>```java
>// OneServlet将数据共享给TwoServlet
>//1.通过请求对象向Tomcat索要当前网站全局作用域对象
>ServletContext application = request.getServletContext();
>//2.将数据添加到全局作用域对象，作为共享数据
>application.setAttribute("key1", 100);
>```
>
>```java
>//1.通过请求对象向Tomcat索要当前网站全局作用域对象
>ServletContext application = request.getServletContext();
>//2.从全局作用域对象得到指定关键字对应的值
>Integer money=(Integer)application.getAttribute("key1");
>```

>**Cookie**
>
>1. Cookie来自于Servlet规范中一个工具类，存在于Tomcat提供servlet-api.jar中；
>
>2. 如果两个Servlet来自于同一个网站，并且为同一个浏览器/用户提供服务，此时
>借助于Cookie对象进行数据共享；
>
>3. ==Cookie存放当前用户的私人数据，在共享数据过程中提高服务质量==；
>
>4. 在现实生活场景中，Cookie相当于用户在服务端得到【会员卡】。
>
>用户通过浏览器第一次向MyWeb网站发送请求申请OneServlet。==OneServlet在运行期间创建一个Cookie存储与当前用户相关数据OneServlet工作完毕后，【将Cookie写入到响应头】交还给当前浏览器。==
>浏览器收到响应响应包之后，==将cookie存储在浏览器的缓存一段时间之后，用户通过【同一个浏览器】再次向【myWeb网站】发送请求申请TwoServlet时。【浏览器需要无条件的将myWeb网站之前推送过来的Cookie，写入到请求头】发送过去此时TwoServlet在运行时==，就可以通过读取请求头中cookie中信息，得到OneServlet提供的共享数据。
>
>```java
>String userName,money;
>//1.调用请求对象读取【请求头】参数信息
>userName = request.getParameter("userName");
>money = request.getParameter("money");
>//2.开卡
>Cookie card1 = new Cookie("userName", userName);
>Cookie card2 = new Cookie("money",money);
>//3.发卡，将Cookie写入到响应头交给浏览器
>response.addCookie(card1);
>response.addCookie(card2);
>//4.通知Tomcat将【点餐页面】内容写入到响应体交给浏览器(请求转发)
>request.getRequestDispatcher("/index_2.html").forward(request, response);
>```
>
>```java
>int jiaozi_money = 30;
>int gaifan_money = 15;
>int miantiao_money = 20;
>int money = 0, xiaofei = 0, balance = 0;
>String food, userName = null;
>Cookie cookieArray[] = null;
>response.setContentType("text/html;charset=utf-8");
>PrintWriter out = response.getWriter();
>Cookie newCard = null;
>//1.读取请求头参数信息，得到用户点餐食物类型
>food = request.getParameter("food");
>//2.读取请求中Cookie
>cookieArray = request.getCookies();
>//3.刷卡消费
>for (Cookie card : cookieArray) {
>  String key = card.getName();
>  String value = card.getValue();
>  if ("userName".equals(key)) {
>    userName = value;
>  } else if ("money".equals(key)) {
>    money = Integer.valueOf(value);
>    if ("饺子".equals(food)) {
>      if (jiaozi_money > money) {
>        out.print("用户 " + userName + " 余额不足，请充值");
>      } else {
>        newCard = new Cookie("money", (money - jiaozi_money) + "");
>        xiaofei = jiaozi_money;
>        balance = money - jiaozi_money;
>      }
>    } else if ("面条".equals(food)) {
>      if (miantiao_money > money) {
>        out.print("用户 " + userName + " 余额不足，请充值");
>      } else {
>        newCard = new Cookie("money", (money - miantiao_money) + "");
>        xiaofei = miantiao_money;
>        balance = money - miantiao_money;
>      }
>    } else if ("盖饭".equals(food)) {
>      if (gaifan_money > money) {
>        out.print("用户 " + userName + " 余额不足，请充值");
>      } else {
>        newCard = new Cookie("money", (money - gaifan_money) + "");// 10+"abc"="10abc"
>        xiaofei = gaifan_money;
>        balance = money - gaifan_money;
>      }
>    }
>
>  }
>
>
>}
>//4.将用户会员卡返还给用户
>response.addCookie(newCard);
>//5.将消费记录写入到响应
>out.print("用户 " + userName + "本次消费 " + xiaofei + " 余额 :" + balance);
>```
>
>Cookie销毁时机：
>
>1. 在默认情况下，Cookie对象存放在浏览器的缓存中。因此只要浏览器关闭，Cookie对象就被销毁掉。
>
>2. 在手动设置情况下，可以要求浏览器将接收的Cookie存放在客户端计算机上硬盘上，同时需要指定Cookie在硬盘上存活时间。在存活时间范围内，关闭浏览器关闭客户端计算机，关闭服务器，都不会导致Cookie被销毁。在存活时间到达时，Cookie自动从硬盘上被
>  删除。
>
>  ```java
>  cookie.setMaxAge(60); //cookie在硬盘上存活1分钟
>  ```

>**HttpSession接口**
>
>1. 自于Servlet规范下一个接口。存在于Tomcat中servlet-api.jar。其实现类由Http服务器提供。Tomcat提供实现类存在于servlet-api.jar。
>2. 如果两个Servlet来自于同一个网站，并且为同一个浏览器/用户提供服务，此时借助于HttpSession对象进行数据共享。
>3. HttpSession接口修饰对象称为==【会话作用域对象】==。
>
>```java
>String goodsName;
>//1.调用请求对象，读取请求头参数，得到用户选择商品名
>goodsName = request.getParameter("goodsName");
>//2.调用请求对象，向Tomcat索要当前用户在服务端的私人储物柜
>HttpSession session = request.getSession();
>//session.setMaxInactiveInterval(5);
>//3.将用户选购商品添加到当前用户私人储物柜
>Integer  goodsNum = (Integer)session.getAttribute(goodsName);
>if(goodsNum == null){
>  session.setAttribute(goodsName, 1);
>}else{
>  session.setAttribute(goodsName, goodsNum+1);
>}
>```
>
>```java
>//1.调用请求对象，向Tomcat索要当前用户在服务端私人储物柜
>HttpSession session = request.getSession();
>
>//2.将session中所有的key读取出来，存放一个枚举对象
>Enumeration goodsNames =session.getAttributeNames();
>while(goodsNames.hasMoreElements()){
>  String goodsName =(String) goodsNames.nextElement();
>  int goodsNum = (int)session.getAttribute(goodsName);
>  System.out.println("商品名称 "+goodsName+" 商品数量 "+goodsNum);
>}
>```
>
>HttpSession销毁时机：
>
>1. 用户与HttpSession关联时使用的Cookie只能存放在浏览器缓存中。
>
>2. 在浏览器关闭时，意味着用户与他的HttpSession关系被切断。
>
>3. 由于Tomcat无法检测浏览器何时关闭，因此在浏览器关闭时并不会导致Tomcat将浏览器关联的HttpSession进行销毁。
>
>4. 为了解决这个问题，Tomcat为每一个HttpSession对象设置【空闲时间】这个空闲时间默认30分钟，如果当前HttpSession对象空闲时间达到30分钟此时Tomcat认为用户已经放弃了自己的HttpSession，此时Tomcat就会销毁掉这个HttpSession。
>
>  ```xml
>  <!--在当前网站/web/WEB-INF/web.xml-->
>  <session-config>
>    <session-timeout>5</session-timeout> <!--手动设置当前网站中每一个session最大空闲时间5分钟-->
>  </session-config>
>  ```

>**HttpServletRequest接口**
>
>1. 在同一个网站中，如果两个Servlet之间通过【请求转发】方式进行调用，彼此之间共享同一个请求协议包。而一个请求协议包只对应一个请求对象因此servlet之间共享同一个请求对象，此时可以利用这个请求对象在两个Servlet之间实现数据共享。
>
>2. 在请求对象实现Servlet之间数据共享功能时，将请求对象称为【请求作用域对象】。
>
>  ```java
>  //1.将数据添加到请求请求作用域中,作为共享数据
>  request.setAttribute("key1","hello world");
>  //2.代替浏览器,向Tomcat索要TwoServlet
>  request.getRequestDispatcher("/two").forward(request,response);
>  ```
>
>  ```java
>  //从同一个请求作用域对象得到OneServlet写入到共享数据
>  String value =(String)request.getAttribute("key1");
>  System.out.println("TwoServlet得到的共享数据为"+value);
>  ```



#### 11. Servlet规范扩展——Listener接口

---

1. 一组来自于Servlet规范下接口，共有8个接口。在Tomcat存在servlet-api.jar包；
2. ==监听器接口需要由开发人员亲自实现==，Http服务器提供jar包并没有对应的实现类；
3. ==监听器接口用于监控【作用域对象生命周期变化时刻】以及【作用域对象共享数据变化时刻】==。

>作用域对象：
>
>1. 在Servlet规范中，在服务端内存中可以在某些条件下为两个Servlet之间提供数据共享方案的对象，被称为【作用域对象】。
>2. Servlet规范下作用域对象:
>   ServletContext：全局作用域对象
>   HttpSession：会话作用域对象
>   HttpServletRequest：请求作用域对象

>监听器接口实现类开发规范：
>
>1. 根据监听的实际情况，选择对应监听器接口进行实现；
>2. 重写监听器接口声明【监听事件处理方法】；
>3. ==在web.xml文件将监听器接口实现类注册到Http服务器==。

>**ServletContextListener接口**
>
>作用：通过这个接口合法的检测全局作用域对象被初始化时刻以及被销毁时刻。
>
>监听事件处理方法：
>
>`public void contextInitlized()`：在全局作用域对象被Http服务器初始化被调用
>
>`public void contextDestory()`：在全局作用域对象被Http服务器销毁时候触发调用。

>**ServletContextAttributeListener接口**
>
>作用：通过这个接口合法的检测全局作用域对象共享数据变化时刻。
>
>监听事件处理方法：
>
>`public void contextAdd()`：在全局作用域对象添加共享数据
>`public void contextReplaced()`：在全局作用域对象更新共享数据
>`public void contextRemove()`：在全局作用域对象删除共享数据



#### 12. Servlet规范扩展——Filter接口

---

1. 来自于Servlet规范下接口，在Tomcat中存在于servlet-api.jar包；
2. Filter接口实现类由开发人员负责提供，Http服务器不负责提供；
3. Filter接口在Http服务器调用资源文件之前，对Http服务器进行拦截。

>作用：
>
>1. 拦截Http服务器，帮助Http服务器检测当前请求合法性；
>2. 拦截Http服务器，对当前请求进行增强操作。

>Filter接口实现类开发步骤：
>
>1. 创建一个Java类实现Filter接口；
>2. 重写Filter接口中doFilter方法；
>3. web.xml将过滤器接口实现类注册到Http服务器。
>
>```java
>@Component
>@WebFilter(urlPatterns = "/*", filterName = "oneFilter")
>public class OneFilter implements Filter {
>
>  @Override
>  public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
>    //1.通过拦截请求对象得到请求包参数信息,从而得到来访用户的真实年龄
>    String age=servletRequest.getParameter("age");
>    ///2.根据年龄,帮助Http服务器判断本次请求的合法性
>    if (Integer.valueOf(age)<70){
>      //合法请求
>      filterChain.doFilter(servletRequest,servletResponse);
>    }else{
>      //过滤器代替服务器拒绝本次请求
>      servletResponse.setContentType("text/html; charset=utf-8");
>      PrintWriter out=servletResponse.getWriter();
>      out.print("<center><font style='color:red;font-size:40px'>大爷,请珍爱生命!!</font></center>");
>    }
>  }
>}
>```
>
>要求Tomcat在调用网站中任意文件时，来调用OneFilter拦截。

- 在servlet的规范当中，servlet容器或者叫web容器，如tomcat，中运行的每个应用都由一个ServletContext表示，在web容器中可以包含多个ServletContext，即可以有多个web应用在web容器中运行。如在tomcat的webapp目录下，每个war包都对应一个web应用，tomcat启动时会解压war包，并启动相关的应用。
- 在web容器启动的时候，会初始化web应用，即创建ServletContext对象，加载解析web.xml文件，获取该应用的Filters，Listener，Servlet等组件的配置并创建对象实例，作为ServletContext的属性，保存在ServletContext当中。之后web容器接收到客户端请求时，则会根据请求信息，匹配到处理这个请求的Servlet，同时在交给servlet处理之前，会先使用应用配置的Filters对这个请求先进行过滤，最后才交给servlet处理。
- 了解web容器启动，之后接受客户端请求这些知识有啥用处呢？这里我们需要回过头来看我们的spring项目。我们在日常开发中，直接接触的是spring相关的组件，然后打成war包，放到web容器中，如拷贝到tomcat的webapp目录，并不会直接和web容器打交道。经过以上的分析，其实一个spring项目就是对应web容器里的一个ServletContext，所以在ServletContext对象的创建和初始化的时候，就需要一种机制来触发spring相关组件的创建和初始化，如包含@Controller和@RequestMapping注解的类和方法，这样才能处理请求。
