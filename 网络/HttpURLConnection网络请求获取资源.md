当前，Http协议是Internet上使用最多，也最重要的网络协议，无可厚非，更多的java程序直接采用http请求访问网络资源。

JDK的java.net包中提供了访问HTTP协议的基本功能的类：HttpURLConnection。
HttpURLConnection是Java的标准类，它继承自URLConnection，可用于向指定Url发送GET请求、POST请求。它在URLConnection的基础上提供了更加便捷的方法
同样进行网络请求的还有Apache的HttpClient，==Google建议使用httpURLconnection进行网络访问操作==。
总结：HttpURLConnection基于Http协议，支持get，post，put，delete等各种请求方式。



#### 1. 示例

```java
String fileUrl = logoUrl;
URL url = new URL(fileUrl);
HttpURLConnection urlCon = (HttpURLConnection) url.openConnection();
urlCon.setRequestMethod("GET");
urlCon.setConnectTimeout(6000);
urlCon.setReadTimeout(6000);
int code = urlCon.getResponseCode();
if (code != HttpURLConnection.HTTP_OK) {
  throw new Exception("文件读取失败");
}
//读文件流
InputStream inputStream = urlCon.getInputStream();
```



#### 2. 主流程

- new Url()，设置Url地址
- 创建HttpURLConnection
- 设置链接相关参数：请求方式，连接时间等
- 获取响应资源
- 资源处理

