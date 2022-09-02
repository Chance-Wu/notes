#### 1. 定义

>==动态给一个对象添加一些额外的职责==，就像在墙上刷油漆。使用Decorator模式相比用生成子类方式达到功能的扩充显得更为灵活。

#### 2. 为什么使用装饰?

>通常*<u>可以使用继承来实现功能的拓展</u>*，如果这些需要拓展的功能的种类很繁多，那么势必生成很多子类，增加系统的复杂性，同时，使用继承实现功能拓展，我们必须可预见这些拓展功能，这些功能是编译时就确定了，是静态的。
>
>使用装饰模式的理由是：==这些功能需要由用户动态决定加入的方式和时机==。装饰提供了"即插即用"的方法，在运行期间决定何时增加何种功能。

#### 3. 使用

>首先建立一个接口：
>
>```java
>public interface Worker {
>
>    void insert();
>}
>```
>
>Work接口有一个具体实现：插入方形桩或圆形桩，以插入方形桩为例：
>
>```java
>public class SquarePeg implements Worker {
>
>    @Override
>    public void insert() {
>        System.out.println("方形桩插入");
>    }
>}
>```
>
>现在有一个应用：需要在桩打入前，挖坑，在打入后，在桩上钉木板，这些额外的功能是动态，可能随意增加调整修改，比如，可能又需要在打桩之后钉架子(只是比喻)。
>
>那么我们使用Decorator模式,这里方形桩SquarePeg是decoratee(被刷油漆者)，我们需要在decoratee上刷些"油漆"，这些油漆就是那些额外的功能。
>
>```java
>public class Decorator implements Work {
>
>    private Work work;
>
>    /**
>     * 额外增加的功能被打包在这个List中
>     */
>    private ArrayList others = new ArrayList();
>
>    /**
>     * 在构造器中使用组合new方式,引入Work对象
>     */
>    public Decorator(Work work) {
>        this.work = work;
>        others.add("挖坑");
>        others.add("钉木板");
>    }
>
>    @Override
>    public void insert() {
>        newMethod();
>    }
>
>    /**
>     * 在新方法中,我们在insert之前增加其他方法,这里次序先后是用户灵活指定的
>     */
>    public void newMethod() {
>        otherMethod();
>        work.insert();
>    }
>
>    public void otherMethod() {
>        ListIterator listIterator = others.listIterator();
>        while (listIterator.hasNext()) {
>            System.out.println(listIterator.next() + " 正在进行");
>        }
>    }
>}
>```
>
>使用：
>
>```java
>Work work = new SquarePeg();
>Work decorator = new Decorator(squarePeg);
>decorator.insert();
>```

>整个 JAVA IO 的核心就是采用了Decorator（装饰）模式。

>使用JAVA IO的时候经常会看到以下写法：
>
>```java
>InputStream in=new BufferedInputStream(new FileInputStream("filename"));
>```
>
>实际上Java 的I/O API就是使用[Decorator实现的，](http://www-900.ibm.com/developerWorks/java/l-jdkdp/part3/index.shtml#2)I/O变种很多，如果都采取继承方法，将会产生很多子类，显然相当繁琐。

