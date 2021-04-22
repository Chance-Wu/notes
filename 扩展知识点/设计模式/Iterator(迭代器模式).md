>==迭代器模式==（又称游标模式）。这种模式提供一种方法访问一个容器对象中各个元素，而又不需要暴露该对象的内部细节。
>
>```java
>Iterator<String> it = collection.iterator();
>
>while (it.hasNext()){
>    //using "it.next();" do some business logic
>}
>```

#### 1. 举例

>一个书架上放着好几本书，现在我想知道书架上都有哪些书，并且都把书名打印出来。那么书架就可以具有迭代的功能，能把它存放的所有书籍都迭代出来。

>1）一个迭代器接口：
>
>```java
>public interface Iterator<E> {
>    /**
>     * 检测是否还有下一个元素
>     */
>    boolean hasNext();
>
>    /**
>     * 获得下一个元素
>     */
>    E next();
>
>    default void remove() {
>        throw new UnsupportedOperationException("remove");
>    }
>
>    default void forEachRemaining(Consumer<? super E> action) {
>        Objects.requireNonNull(action);
>        while (hasNext())
>            action.accept(next());
>    }
>}
>```
>
>2）定义含有迭代器对象的接口
>
>```java
>/**
> * 只有实现此接口才可以获得迭代器对象
> */
>public interface Aggregate<E> {
>
>    /**
>     * 获得迭代器
>     */
>    Iterator<E> iterator();
>}
>
>```
>
>3）书籍类
>
>```java
>/**
> * 书籍类
> */
>public class Book {
>
>    /**
>     * 书籍名称
>     */
>    private String name;
>
>    public Book(String name) {
>        this.name = name;
>    }
>
>    /**
>     * 获得书籍名称
>     */
>    public String getName() {
>        return name;
>    }
>}
>```
>
>4）书架迭代器
>
>```java
>/**
> * 书架迭代器
> */
>public class BookShelfIterator<E> implements Iterator<E> {
>
>    private BookShelf bookShelf;
>    private int index;
>
>    public BookShelfIterator(BookShelf bookShelf) {
>        this.bookShelf = bookShelf;
>        this.index = 0;
>    }
>
>    /**
>     * 检测是否还有下一本书
>     */
>    @Override
>    public boolean hasNext() {
>        if (index < bookShelf.getLength()) {
>            return true;
>        } else {
>            return false;
>        }
>    }
>
>    /**
>     * 返回下一本书
>     */
>    @Override
>    public E next() {
>        Book book = bookShelf.getBookAt(index);
>        index++;
>        return (E) book;
>    }
>}
>```
>
>5）书架类
>
>```java
>/**
> * 书架类
> */
>public class BookShelf implements Aggregate<Book> {
>
>    private Book[] books;
>
>    private int last = 0;
>
>    public BookShelf(int maxSize) {
>        this.books = new Book[maxSize];
>    }
>
>    /**
>     * 获得书籍
>     */
>    public Book getBookAt(int index) {
>        return books[index];
>    }
>
>    /**
>     * 添加书籍
>     */
>    public void appendBook(Book book) {
>        this.books[last] = book;
>        last++;
>    }
>
>    /**
>     * 获得书架上的书籍数量
>     */
>    public int getLength() {
>        return books.length;
>    }
>
>    /**
>     * 获得书架迭代器对象
>     */
>    @Override
>    public Iterator iterator() {
>        return new BookShelfIterator<Book>(this);
>    }
>}
>```

>测试：
>
>```java
>public static void main(String[] args) {
>    //创建一个书架
>    BookShelf bookShelf = new BookShelf(5);
>    //向书架中添加书籍
>    bookShelf.appendBook(new Book("深入理解Java虚拟机"));
>    bookShelf.appendBook(new Book("Java编程思想"));
>    bookShelf.appendBook(new Book("高性能MySQL"));
>    bookShelf.appendBook(new Book("Effective Java 中文版"));
>    bookShelf.appendBook(new Book("数据结构与算法分析Java语言描述"));
>    //获得书架迭代器
>    Iterator iterator = bookShelf.iterator();
>    //迭代
>    while (iterator.hasNext()){
>        Book book = (Book) iterator.next();
>        System.out.println(book.getName());
>    }
>}
>```

#### 2. 迭代器的结构

><img src="https://tva1.sinaimg.cn/large/0081Kckwgy1gmaz2o1r7pj31980nm74x.jpg" style="zoom:60%">
>
>迭代器模式主要由以下角色组成：
>
>- ==抽象迭代器（Iterator）==：抽象迭代器定义访问和遍历元素的接口。
>- ==具体迭代器（Concreate Iterator）==：具体迭代器要实现迭代器接口，并要记录遍历中的当前位置。上面例子中BookShelfIterator类就是代表的这个角色。
>- ==容器（Aggregate）==：容器负责提供创建具体迭代器的接口。上面例子中的Aggregate接口代表的就是这个角色。
>- ==具体容器（Concreate Aggregate）==：具体容器角色实现具体迭代器角色的接口，这个具体迭代器角色与该容器的结构相关。上面例子中书架类BookShelf代表的就是这个角色。

#### 3. 总结

>迭代器模式是一种使用频率非常高的设计模式，==通过引入迭代器可以将数据的遍历功能从聚合对象中分离出来==，聚合对象只负责存储数据，而遍历数据由迭代器实现完成。Java语言类库中已经实现了迭代器模式，在实际开发中我们直接使用已经定义好的迭代器就可以了，像List、Set等集合都可以直接使用。

>优点：
>
>1. 它支持以不同的方式遍历一个聚合对象，在同一个聚合对象上可以定义多种遍历方式。替换迭代器就可以切换遍历方法。
>2. 迭代器简化了聚合类。聚合对象可以不用自己再提供遍历方法。
>3. 在迭代器模式中由于引入了抽象层，增加新的聚合类和迭代器类都很方便，无需修改原有代码，满足“开闭原则”的要求。

>缺点：
>
>1. 由于迭代器模式将存储数据和遍历数据的职责分离，增加新的聚合类需要对应增加新的迭代器来，类的个数成对增加，这在一定程度上增加了系统的复杂性。
>2. 抽象迭代器设计难度相对较大，需要充分考虑到系统将来的扩展，，例如JDK内置迭代器Iterator就无法实现逆向遍历，==如果需要实现逆向遍历，只能通过其子类ListIterator等来实现，而ListIterator迭代器无法用于操作Set类型的聚合对象==。

#### 4. 适用场景

>1. 访问一个聚合对象的内容而无需暴露它的内部表示。将聚合对象的访问与内部数据的存储分离，使得访问聚合对象时无须了解其内部实现细节。
>2. 需要为一个聚合对象提供多种遍历方式。
>3. 为遍历不同聚合结构提供统一的接口，该接口的实现类中为不同的聚合结构提供不同的遍历方式，而客户端可以一致性的操作该接口。



