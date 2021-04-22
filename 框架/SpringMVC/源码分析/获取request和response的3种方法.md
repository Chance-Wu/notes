#### 1. 通过RequestContextHolder静态方法获取

>```java
>ServletRequestAttributes servletRequestAttributes = (ServletRequestAttributes)RequestContextHolder.getRequestAttributes();
>HttpServletRequest request = servletRequestAttributes.getRequest();
>HttpServletResponse response = servletRequestAttributes.getResponse();
>```

#### 2. 通过参数直接获取

>只要在方法上加上参数，SpringMVC会帮你绑定。如果有多个参数，把两个参数加到末尾即可。
>
>```java
>@GetMapping(value = "")
>public String getMember(HttpServletRequest request,HttpServletResponse response) {
>    //...
>}
>```

#### 3. 注入到类

>```java
>@Autowired
>private HttpServletRequest request;
>
>@Autowired
>private HttpServletResponse response;
>
>@GetMapping(value = "")
>public String center() {
>    //...
>}
>```