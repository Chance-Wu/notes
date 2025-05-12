通过Feign返回`Response`对象获取原始输入流，并手动关闭资源：

```java
@FeignClient(name = "file-service")
public interface DownloadClient {
    @GetMapping("/download")
    feign.Response download;
}

// 调用示例
public void downloadFile(HttpServletResponse response) {
    feign.Response feignResponse = downloadClient.download;
    try (InputStream inputStream = feignResponse.body.asInputStream) {
        byte[] buffer = new byte;
        int len;
        while ((len = inputStream.read(buffer)) != -1) {
            response.getOutputStream.write(buffer, 0, len);
        }
    }
}
```



### 一、下载

---

feign.Response 是 Feign 库中的一个类，用于表示 HTTP 响应。它提供了一些方法来获取响应的状态码、头部信息和响应体等内容。

在使用 Feign 进行服务间通信时，调用远程接口的方法可能会返回一个 feign.Response 对象，您可以通过该对象来获取远程服务的响应信息。

#### 1.1 服务端

```java
@RestController
public class DemoController {
    @GetMapping(value = "/download")
    public void download(HttpServletResponse response) throws Exception {
        InputStream inputStream = ... //读取字节流
        byte[] b = new byte[1024];
        int len;
        while ((len = inputStream.read(b, 0, 1024)) != -1) {
            response.getOutputStream().write(b, 0, len);
        }
    }
}
```

#### 1.2 客户端

- 声明接口

  ```java
  @FeignClient
  public interface IFeignRestService {
      @GetMapping(value = "/download")
      feign.Response download() throws Exception;
  }
  ```

- 下载文件

  ```java
      public void download(HttpServletResponse response) throws Exception {
          feign.Response feignResponse = null;
          InputStream inputStream = null;
          try {
              feignResponse = feignRestService.download(processInstanceId);
              inputStream = feignResponse.body().asInputStream();
              // 其他操作...
              inputStream.close();
          } catch (Exception e) {
              throw e;
          } finally {
              if (inputStream!=null){
                  inputStream.close();
              }
          }
      }   
  ```



### 二、常见问题

---



















































































