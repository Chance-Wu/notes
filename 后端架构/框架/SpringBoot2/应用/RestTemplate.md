### 一、简介

---

RestTemplate 是执行HTTP请求的**同步阻塞式**的客户端，它在HTTP客户端库（例如`JDK HttpURLConnection`，`Apache HttpComponents`，`okHttp`等）基础封装了更加简单易用的模板方法API。也就是说RestTemplate是一个封装，底层的实现还是java应用开发中常用的一些HTTP客户端。

RestTemplate 作为spring-web项目的一部分，在Spring 3.0版本开始被引入。

>根据Spring官方文档及源码中的介绍，RestTemplate在将来的版本中它可能会被弃用，因为他们已在Spring 5中引入了WebClient作为非阻塞式Reactive HTTP客户端。



### 二、将RestTemplate配置初始化为一个Bean

---

添加依赖

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

创建RestTemplateConfig类，设置连接池大小、超时时间、重试机制等。配置如下：

```java
@Configuration
public class RestTemplateConfig {

  @ConditionalOnMissingBean(RestTemplate.class)
  @Bean
  public RestTemplate restTemplate(){
    RestTemplate restTemplate = new RestTemplate();
    return restTemplate;
  }
}
```

这种初始化方法，是使用了JDK 自带的HttpURLConnection作为底层HTTP客户端实现。我们还可以把底层实现切换为Apache HttpComponents，okHttp等。

在需要使用RestTemplate 的位置，注入并使用即可：

```java
@Resource
private RestTemplate restTemplate;
```

#### 2.1 发送GET请求

- getForObject()返回值是HTTP协议的响应体。
- getForEntity()返回的是ResponseEntity，ResponseEntity是对HTTP响应的封装，除了包含响应体，还包含HTTP状态码、contentType、contentLength、Header等信息。

```java
// 1-getForObject()
String response = restTemplate.getForObject(url , String.class);
System.out.println(response);

// 2-getForEntity().
ResponseEntity<String> responseEntity1 = restTemplate.getForEntity(url, String.class);
HttpStatus statusCode = responseEntity1.getStatusCode();
HttpHeaders header = responseEntity1.getHeaders();
String response2 = responseEntity1.getBody();
System.out.println(response2);

// 3-exchange()
RequestEntity requestEntity = RequestEntity.get(new URI(url)).build();
ResponseEntity<String> responseEntity2 = restTemplate.exchange(requestEntity, String.class);
String response3 = responseEntity2.getBody();
System.out.println(response3);
```

#### 2.2 发送POST请求

```java
// 1-postForObject()
User user1 = this.restTemplate.postForObject(uri, user, User.class);

// 2-postForEntity()
ResponseEntity<User> responseEntity1 = this.restTemplate.postForEntity(uri, user, User.class);

// 3-exchange()
RequestEntity<User> requestEntity = RequestEntity.post(new URI(uri)).body(user);
ResponseEntity<User> responseEntity2 = this.restTemplate.exchange(requestEntity, User.class);
```

#### 2.3 设置HTTP Header

```java
// 1-Content-Type
RequestEntity<User> requestEntity = RequestEntity
  .post(new URI(uri))
  .contentType(MediaType.APPLICATION_JSON)
  .body(user);

// 2-Accept
RequestEntity<User> requestEntity = RequestEntity
  .post(new URI(uri))
  .accept(MediaType.APPLICATION_JSON)
  .body(user);

// 3-Other
RequestEntity<User> requestEntity = RequestEntity
  .post(new URI(uri))
  .header("Authorization", "Basic " + base64Credentials)
  .body(user);
```

#### 2.4 发送文件

```java
@RequestMapping(value="/sendFile")
public String sendFile() throws URISyntaxException, IOException {
  MultiValueMap<String, Object> multiPartBody = new LinkedMultiValueMap<>();
  multiPartBody.add("file", new ClassPathResource("tmp/user.txt"));
  RequestEntity<MultiValueMap<String, Object>> requestEntity = RequestEntity
    .post(new URI("http://127.0.0.1:8080/upload1"))
    .contentType(MediaType.MULTIPART_FORM_DATA)
    .body(multiPartBody);
  ResponseEntity<String> response = restTemplate.exchange(requestEntity, String.class);
  return "sueecess";
}
```

#### 2.5 下载文件

```java
/**
* 下载文件
* @param response
* @throws URISyntaxException
* @throws IOException
*/
@RequestMapping(value="/receiveFile")
public void receiveFile(HttpServletResponse response) throws URISyntaxException, IOException {
  // 小文件
  RequestEntity requestEntity = RequestEntity.get(
    new URI("http://127.0.0.1:8080/downLoad.html")).build();
  ResponseEntity<byte[]> responseEntity = restTemplate.exchange(requestEntity, byte[].class);
  byte[] downloadContent = responseEntity.getBody();
  response.reset();
  response.setHeader("Content-Disposition", "attachment; filename=\"myframe.html\"");
  response.addHeader("Content-Length", "" + downloadContent.length);
  response.setContentType("application/octet-stream; charset=UTF-8");
  IOUtils.write(downloadContent, response.getOutputStream());
}

/**
* 大文件
* @param response
* @return
* @throws URISyntaxException
*/
@RequestMapping(value="/bigFile")
public String bigFile(HttpServletResponse response) throws URISyntaxException {
  //大文件
  ResponseExtractor<ResponseEntity<File>> responseExtractor =
    new ResponseExtractor<ResponseEntity<File>>() {
    @Override
    public ResponseEntity<File> extractData(ClientHttpResponse response)
      throws IOException {
      File rcvFile = File.createTempFile("rcvFile", "zip");
      FileCopyUtils.copy(response.getBody(), new FileOutputStream(rcvFile));
      return ResponseEntity.status(response.getStatusCode()).
        headers(response.getHeaders()).body(rcvFile);
    }
  };
  RequestCallback requestCallback = new RequestCallback() {
    @Override
    public void doWithRequest(ClientHttpRequest clientHttpRequest) throws IOException {

    }
  };
  ResponseEntity<File> fileBody = this.restTemplate.execute(
    new URI("http://127.0.0.1:8080/downLoad.html"),
    HttpMethod.GET,  requestCallback, responseExtractor);
  File file = fileBody.getBody();
  file.renameTo(new File("D:/Users/big.hmtl"));
  return "success";
}
```



```java
@Test
public void testGetWithHeaders(){
  HttpHeaders headers = new HttpHeaders();
  headers.setContentType(MediaType.APPLICATION_JSON);
  Map<String,Integer> map = new HashMap<String,Integer>();
  map.put("pageNum",1);
  map.put("pageSize",15);
  HttpEntity<MultiValueMap> httpEntity = new HttpEntity<>(null, headers);
  //get请求
  String newUrl = "https://xxxxxxx.com:8101/operate/operate_pay/getOperatesPay?pageNum="+1+"&pageSize="+15;
  ResponseEntity<String> responseEntity = restTemplate.exchange(newUrl, HttpMethod.GET, httpEntity, String.class);
  String body = responseEntity.getBody();
  System.err.println(body);
}
```



### 三、源码分析

---

`org.springframework.http.client.support.HttpAccessor`用于HTTP接触访问的基础类。

![RestTemplate-HttpAccessor源码分析](img/1815316-20200803072220884-189092191.png)

> 标准JDK HTTP库不支持HTTP PATCH方法。配置Apache HttpComponents或OkHttp请求工厂以启用PATCH。

RestTemplate 支持至少三种HTTP客户端库。

- `SimpleClientHttpRequestFactory`。对应的HTTP库是java JDK自带的 HttpURLConnection。
- `HttpComponentsAsyncClientHttpRequestFactory`。对应的HTTP库是 Apache HttpComponents。
- `OkHttp3ClientHttpRequestFactory`。对应的HTTP库是 OkHttpClient。

可以通过设置setRequestFactory方法，来切换RestTemplate的底层HTTP客户端实现类库。



### 四、底层实现切换方法

---

OkHttp 优于 Apache HttpComponents、Apache HttpComponents优于HttpURLConnection。

#### 4.1 切换为okHTTP

```xml
<dependency>
  <groupId>com.squareup.okhttp3</groupId>
  <artifactId>okhttp</artifactId>
  <version>4.9.1</version>
</dependency>
```

如果是spring 环境下通过如下方式使用OkHttp3ClientHttpRequestFactory初始化RestTemplate bean对象。

```java
@Configuration
public class ContextConfig {
  @Bean("OKHttp3")
  public RestTemplate OKHttp3RestTemplate(){
    RestTemplate restTemplate = new RestTemplate(new OkHttp3ClientHttpRequestFactory());
    return restTemplate;
  }
}
```

#### 4.2 切换为Apache HttpComponents

```xml
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.5.12</version>
</dependency>
```

使用HttpComponentsClientHttpRequestFactory初始化RestTemplate bean对象。

```java
@Bean("httpClient")
public RestTemplate httpClientRestTemplate(){
  RestTemplate restTemplate = new RestTemplate(new HttpComponentsClientHttpRequestFactory());
  return restTemplate;
}
```



### 五、占位符号传参的几种方式

---

以下的几个请求都是在访问"http://jsonplaceholder.typicode.com/posts/1"，只是使用了占位符语法，这样在业务使用上更加灵活。

- 使用占位符的形式传递参数：

```java
String url = "http://jsonplaceholder.typicode.com/{1}/{2}";
PostDTO postDTO = restTemplate.getForObject(url, PostDTO.class, "posts", 1);
```

- 另一种使用占位符的形式：

```python
String url = "http://jsonplaceholder.typicode.com/{type}/{id}";
String type = "posts";
int id = 1;
PostDTO postDTO = restTemplate.getForObject(url, PostDTO.class, type, id);
```

- 也可以使用 map 装载参数：

```vhdl
String url = "http://jsonplaceholder.typicode.com/{type}/{id}";
Map<String,Object> map = new HashMap<>();
map.put("type", "posts");
map.put("id", 1);
PostDTO  postDTO = restTemplate.getForObject(url, PostDTO.class, map);
```



























































































