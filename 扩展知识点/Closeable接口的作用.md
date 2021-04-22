Closeable是可以关闭的数据源或目标。调用close方法可释放对象保存的资源（如打开文件）。

```java
public interface Closeable extends AutoCloseable {

  /**
   * 关闭此流并释放与此流关联的所有系统资源。如果已经关闭该流，则调用此方法无效
   */
  public void close() throws IOException;
}

```