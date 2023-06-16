>策略模式是指对一系列的算法定义，并将每一个算法封装起来，而且使它们还可以==相互替换==。策略模式让算法独立于使用它的客户而独立变化。
>
>- 使用策略模式有时候可以让我们的编码从繁琐难维护的if-else中解放出来。
>- 决定request的media types时也用到了策略模式。其中的ContentNegotiationManager是最核心的一个类。
>
>策略接口如下：
>
>```java
>public interface ContentNegotiationStrategy {
>
>    List<MediaType> resolveMediaTypes(NativeWebRequest webRequest)
>        throws HttpMediaTypeNotAcceptableException;
>}
>```
>
>很多实现类，实现具体策略：
>
><img src="https://tva1.sinaimg.cn/large/0081Kckwgy1gm04vd3q23j31ds0ae75g.jpg" style="zoom:50%">
>
>```java
>
>```
>
>

