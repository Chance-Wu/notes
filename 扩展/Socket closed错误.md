### 一、理解错误含义

---

在socket编程中报错`java.net.SocketException: Socket closed`

>This exception means that you closed the socket, and then continued to try to use it.
>
>你把socket关闭了，却还想继续使用，就会报这个错。



### 二、可能原因

---

#### 2.1 关闭了IO流

在你发送、接收操作做完之前别关IO流，也许就能解决；
**注意，你可能没有关闭IO流，或没有关掉socket的`socket.getOutputStream()`和 `socket.getInputStream()`；
但它可能被其他IO流关闭影响而自动关闭。**

>socket 只要在 io流close的情况下 自动关闭，意思就是你想边发送边接受最正确的方式就是发送和 接受的操作都做完之后 再一起关闭IO流 完美解决。

#### 2.2 没有主动关闭IO流，还是报这个错

常见于在socket中发送了对象的情况，也即使用了`ObjectOutputStream`，
**出错原因：使用完毕的`ObjectOutputStream`关闭时，会导致其包装的`OutputStream`也自动关闭。**

```java
try(OutputStream os = socket.getOutputStream();
    InputStream is = socket.getInputStream())
{
  Resource resource = new Resource();
  sendResourceObj(resource,os);
  os.write("exit".getBytes(StandardCharsets.UTF_8));
} catch (Exception e) {
  e.printStackTrace();
}
```

其中发送对象的函数如下（本意是为了解耦，把功能尽量模块化）：

```java
public static void sendResourceObj(Resource r,OutputStream os){
  try(ObjectOutputStream oos = new ObjectOutputStream(os))
  {
    oos.writeObject(r);
    oos.flush();
  } catch (Exception e) {
    e.printStackTrace();
  }
}
```

可以看到，在sendResourceObj执行结束后，ObjectOutputStreem就关闭了；修改方法：

把ObjectOutputStream和OutputStream一同，直接放到try-with-resource的声明语句中，待使用IO完毕后一起关闭即可。

```java
try(OutputStream os = socket.getOutputStream();
    ObjectOutputStream oos = new ObjectOutputStream(os);
    InputStream is = socket.getInputStream())
{
  Resource resource = new Resource();
  oos.writeObject(resource);
  oos.flush();
  os.write("exit".getBytes(StandardCharsets.UTF_8));
}

} catch (Exception e) {
  e.printStackTrace();
}
```