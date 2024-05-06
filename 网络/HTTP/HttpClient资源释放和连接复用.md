### 一、资源释放

---

```java
CloseableHttpClient httpclient = HttpClients.createDefault();
HttpGet httpget = new HttpGet("http://localhost/");
CloseableHttpResponse response = httpclient.execute(httpget);
try {
  HttpEntity entity = response.getEntity();
  if (entity != null) {
    InputStream instream = entity.getContent();
    try {
      // do something useful
    } finally {
      instream.close();
    }
  }
} finally {
  response.close();
}
// httpclient.close();

```

首先了解默认配置createDefault和适应自定义连接池两种情况的区别。通过源码可以看到前者也创建了连接池，最大连接20个，单个 host最大2个，但是区别在于每次创建的 httpclient 都自己维护了自己的连接池，而 custom 连接池时所有 httpclient 共用同一个连接池，这是在 api 使用方面需要注意的地方，要避免每次请求新建连接池、关闭连接池，造成性能问题。

































































































