### 一、意图

---

业务委托模式是一种结构设计模式，它在表示层和业务层之间添加了一个抽象层。可以实现层之间的松散耦合，并封装有关如何定位、连接和与构成应用程序的业务对象交互的知识。



### 二、详解

---

业务委托充当从表示层调用业务对象的适配器。

#### 2.1 参与者

- **Client（客户端/表示层）：** 发起请求，需要访问业务服务的对象。
- **Business Delegate（业务委托类）：** 负责与业务层交互的中间类，封装了业务逻辑和查找服务等操作。
- **Business Service（业务服务）：** 实际提供业务逻辑的组件，例如 EJB、POJO 等。
- **Lookup Service（查找服务）：** 负责查找和定位业务服务的组件，例如 JNDI。

#### 2.2 工作原理

1. 客户端通过业务委托类发起请求。
2. 业务委托类使用查找服务查找并获取相应的业务服务。
3. 业务委托类调用业务服务的方法，执行业务逻辑。
4. 业务服务将结果返回给业务委托类。
5. 业务委托类将结果返回给客户端。



### 三、代码实现

---

#### 3.1 业务服务接口

```java
public interface VideoStreamingService {

    /**
     * 执行视频流处理
     * 该方法负责处理视频流数据，可以包括但不限于视频数据的接收、处理和转发
     * 具体处理逻辑取决于服务的实现，可能涉及视频格式的转换、视频内容的分析等操作
     */
    void doProcessing();
}
```

#### 3.2 业务服务实现

```java
@Slf4j
public class NetflixService implements VideoStreamingService {

    @Override
    public void doProcessing() {
        log.info("NetflixService is now processing");
    }
}
```

```java
@Slf4j
public class YouTubeService implements VideoStreamingService {

    @Override
    public void doProcessing() {
        log.info("YouTubeService is now processing");
    }
}
```

#### 3.3 查找服务

```java
public class BusinessLookup {

    /**
     * Netflix 服务实例，用于访问 Netflix 的功能
     */
    private NetflixService netflixService;

    /**
     * YouTube 服务实例，用于访问 YouTube 的功能
     */
    private YouTubeService youTubeService;

    public void setNetflixService(NetflixService netflixService) {
        this.netflixService = netflixService;
    }

    public void setYouTubeService(YouTubeService youTubeService) {
        this.youTubeService = youTubeService;
    }

    /**
     * 根据电影名称选择视频流服务。
     *
     * @param movie 电影名称
     * @return 返回选定的视频流服务实例
     */
    public VideoStreamingService getBusinessService(String movie) {
        // 根据电影名称是否包含 "die hard" 来选择视频流服务
        if (movie.toLowerCase().contains("die hard")) {
            return netflixService;
        } else {
            return youTubeService;
        }
    }
}
```

#### 3.4 业务委托类

用于封装对业务服务的请求，它通过一个业务查找对象来定位并激活相应的业务服务。

```java
public class BusinessDelegate {

    /**
     * 业务查找对象，用于获取具体的业务服务
     */
    private BusinessLookup businessLookup;

    public void setBusinessLookup(BusinessLookup businessLookup) {
        this.businessLookup = businessLookup;
    }

    /**
     * 播放电影的方法
     * 根据电影名称查找并激活相应的视频流服务进行电影播放
     *
     * @param movie 电影名称，用以确定使用哪个视频流服务
     */
    public void playbackMovie(String movie) {
        // 根据电影名称获取对应的视频流服务实例
        VideoStreamingService videoStreamingService = businessLookup.getBusinessService(movie);
        // 调用视频流服务的处理方法，开始播放电影
        videoStreamingService.doProcessing();
    }
}
```

#### 3.5 客户端

```java
public class MobileClient {

    /**
     * 业务委托对象，用于处理具体的业务逻辑操作。
     */
    private final BusinessDelegate businessDelegate;

    /**
     * 构造函数，初始化 MobileClient 实例。
     *
     * @param businessDelegate 业务委托对象，用于处理具体的业务逻辑操作。
     */
    public MobileClient(BusinessDelegate businessDelegate) {
        this.businessDelegate = businessDelegate;
    }

    /**
     * 发起电影播放请求。
     * 该方法将播放电影的任务委托给业务委托对象，使得客户端可以在不了解具体实现细节的情况下发起电影播放。
     *
     * @param movie 要播放的电影名称。
     */
    public void playbackMovie(String movie) {
        businessDelegate.playbackMovie(movie);
    }
}
```













































































































