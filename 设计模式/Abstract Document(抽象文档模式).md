### 一、目的

---

使用**动态属性**，并在保持**类型安全**的同时实现非类型化语言的灵活性。



### 二、解释

---

此模式使用特征的概念来实现类型安全，并将不同类的属性分离为一组接口。

#### 2.1 真实世界示例

>考虑由多个部分组成的汽车。我们不知道特定汽车是否真的拥有所有零件，或者仅仅是零件中的一部分。我们的汽车是动态而且非常灵活的。

抽象文档模式允许在对象不知道的情况下将属性附加到对象。

#### 2.2 程序示例

定义基类：Document和AbstractDocument

```java
/**
 * Document接口定义了抽象文档的基本操作方法。
 *
 * <p>抽象文档是一种键值对集合，可以包含嵌套的子文档。
 * 该接口的主要用途是提供一个标准的方式来操作这些文档中的数据。
 *
 * @author chance
 * @date 2024/11/28 14:03
 * @since 1.0
 */
public interface Document {

    /**
     * 向文档中插入一个键值对。
     * 如果指定键已存在，该方法应替换旧的值。
     *
     * @param key   文档中的键，用于标识值。
     * @param value 与键关联的值。
     */
    void put(String key, Object value);

    /**
     * 根据键获取文档中的值。
     *
     * @param key 要检索的键。
     * @return 与键关联的值，如果键不存在，则返回null。
     */
    Object get(String key);

    /**
     * 获取指定键下所有子文档的流。
     * 该方法允许用户通过提供一个构造函数来转换每个子文档，
     * 这使得用户可以以一致和类型安全的方式处理子文档。
     *
     * @param key         子文档的键。
     * @param constructor 一个函数，用于将子文档（表示为键值对映射）转换为指定类型T的实例。
     * @param <T>         转换后子文档的类型。
     * @return 子文档的流，每个子文档都已转换为指定类型T的实例。
     */
    <T> Stream<T> children(String key, Function<Map<String, Object>, T> constructor);
}
```

```java
public class AbstractDocument implements Document {

    private final Map<String, Object> properties;

    public AbstractDocument(Map<String, Object> properties) {
        Objects.requireNonNull(properties, "Properties must be provided");
        this.properties = properties;
    }

    @Override
    public void put(String key, Object value) {
        properties.put(key, value);
    }

    @Override
    public Object get(String key) {
        return properties.get(key);
    }

    @Override
    public <T> Stream<T> children(String key, Function<Map<String, Object>, T> constructor) {
        // 获取指定键下的所有值
        Object value = get(key);
        if (value instanceof List) {
            // 如果值是列表，则尝试将每个元素转换为Map，并应用构造函数
            List<?> list = (List<?>) value;
            return list.stream()
                    .filter(Map.class::isInstance)
                    .map(el -> (Map<String, Object>) el)
                    .map(constructor);
        } else {
            // 如果不是列表或没有找到键，则返回空流
            return Stream.empty();
        }
    }

    @Override
    public String toString() {
        StringBuilder builder = new StringBuilder();
        builder.append(getClass().getName()).append("[");
        properties.forEach((key, value) ->
                builder.append("[").append(key).append(" : ")
                        .append(value).append("]")
        );
        builder.append("]");
        return builder.toString();
    }
}
```

接着定义一个枚举“属性”和一组类型，价格，模型和零件的接口。这使我们能够为Car类创建静态外观的界面。

```java
/**
 * 汽车枚举属性
 *
 * @author chance
 * @date 2024/11/28 14:50
 * @since 1.0
 */
public enum CarProperty {
    /**
     * 部件
     */
    PARTS,
    /**
     * 类型
     */
    TYPE,
    /**
     * 价格
     */
    PRICE,
    /**
     * 模型
     */
    MODEL
}

```

```java
/**
 * 定义了一个具有类型的文档接口
 *
 * <p>该接口继承自Document，并提供了一种获取类型信息的通用方法
 *
 * @author chance
 * @date 2024/11/28 14:53
 * @since 1.0
 */
public interface HasType extends Document {

    default Optional<String> getType() {
        return Optional.ofNullable((String) get(CarProperty.TYPE.toString()));
    }
}

/**
 * 定义了一个具有价格的文档接口
 *
 * <p>该接口继承自Document，并提供了一种获取价格信息的通用方法
 *
 * @author chance
 * @date 2024/11/28 14:53
 * @since 1.0
 */
public interface HasPrice extends Document {

    default Optional<String> getPrice() {
        return Optional.ofNullable((String) get(CarProperty.PRICE.toString()));
    }
}

/**
 * 定义了一个具有模型的文档接口
 *
 * <p>该接口继承自Document，并提供了一种获取模型信息的通用方法
 *
 * @author chance
 * @date 2024/11/28 14:53
 * @since 1.0
 */
public interface HasModel extends Document {

    default Optional<String> getModel() {
        return Optional.ofNullable((String) get(CarProperty.MODEL.toString()));
    }
}



/**
 * 定义了一个具有部件的文档接口
 *
 * <p>该接口继承自Document，并提供了一种获取部件信息的通用方法
 *
 * @author chance
 * @date 2024/11/28 14:53
 * @since 1.0
 */
public interface HasParts extends Document {

    default Stream<Part> getParts() {
        return children(CarProperty.PARTS.toString(), Part::new);
    }
}
```

接着创建Car类

```java
/**
 * Car 类表示一个特定的抽象文档实现，专注于汽车领域。
 * <p>它继承自 AbstractDocument，并实现了与汽车特性相关的接口，如型号、部件、价格和类型。
 * 这种设计允许 Car 类封装与汽车相关的数据和行为，同时提供灵活的属性管理结构。
 *
 * @author chance
 * @date 2024/11/28 14:59
 * @since 1.0
 */
public class Car extends AbstractDocument implements HasModel, HasParts, HasPrice, HasType {

    /**
     * 构造一个 Car 实例，使用属性集合进行初始化。
     *
     * @param properties 包含汽车属性的键值对映射，例如型号、部件、价格等。
     *                   该构造函数将属性直接传递给父类 AbstractDocument 进行初始化。
     */
    public Car(Map<String, Object> properties) {
        super(properties);
    }
}
```

最后是Car构造和使用方法：

```java
Map<String, Object> wheelProperties = new HashMap<>();
wheelProperties.put(CarProperty.TYPE.toString(), "wheel");
wheelProperties.put(CarProperty.MODEL.toString(), "15C");
wheelProperties.put(CarProperty.PRICE.toString(), 100L);

Map<String, Object> doorProperties = new HashMap<>();
doorProperties.put(CarProperty.TYPE.toString(), "door");
doorProperties.put(CarProperty.MODEL.toString(), "Lambo");
doorProperties.put(CarProperty.PRICE.toString(), 300L);

Map<String, Object> carProperties = new HashMap<>();
carProperties.put(CarProperty.MODEL.toString(), "300SL");
carProperties.put(CarProperty.PRICE.toString(), 10000L);
carProperties.put(CarProperty.PARTS.toString(), Arrays.asList(wheelProperties, doorProperties));

Car car = new Car(carProperties);
System.out.println("Here is our car:");
System.out.println("-> model: " + car.getModel().orElseThrow(() -> new IllegalStateException("Model not found")));
System.out.println("-> price: " + car.getPrice().orElseThrow(() -> new IllegalStateException("Price not found")));
System.out.println("-> parts: ");

car.getParts().forEach(part ->
        System.out.println("\t"
                + "/" + part.getType().orElse(null)
                + "/" + part.getModel().orElse(null)
                + "/" + part.getPrice().orElse(null))
);
```

```mermaid
classDiagram
    class Document {
        <<Java Interface>>
        put(String, Object): Void
        get(String): Object
        children(String, Function<Map<String, Object>, T>): Stream<T>
    }

    class AbstractDocument {
        <<Java Class>>
        properties: Map<String, Object>
        AbstractDocument(Map<String, Object>)
        put(String, Object): Void
        get(String): Object
        children(String, Function<Map<String, Object>, T>): Stream<T>
        toString(): String
    }

    class Car {
        <<Java Class>>
        Car(Map<String, Object>)
    }

    class Part {
        <<Java Class>>
        Part(Map<String, Object>)
    }

    class HasParts {
        <<Java Interface>>
        getParts(): Stream<Part>
    }

    class HasModel {
        <<Java Interface>>
        getModel(): Optional<String>
    }

    class HasPrice {
        <<Java Interface>>
        getPrice(): Optional<Number>
    }

    class HasType {
        <<Java Interface>>
        getType(): Optional<String>
    }

    Document <|-- AbstractDocument
    AbstractDocument <|-- Car
    AbstractDocument <|-- Part
    AbstractDocument <|-- HasParts
    AbstractDocument <|-- HasModel
    AbstractDocument <|-- HasPrice
    AbstractDocument <|-- HasType
```



### 三、适用性

---

- 需要即时添加新属性
- 你想要一种灵活的方式来以树状结构组织域
- 你想要更宽松的耦合系统



### 四、内容管理系统CMS

---

开发一个CMS（Content Management System）中的多格式内容处理。其中用户可以创建包含文本、图片、视频等多种类型的内容的文章。可以使用抽象文档模式来设计这样的系统。

#### 4.1 定义一个通用的Content接口

```java
/**
 * 通用的 Content 接口
 * <p>定义了内容对象的基本操作，旨在为不同类型的内容提供一个统一的处理方式
 * 主要用途是获取内容的类型信息，并渲染为字符串形式
 *
 * @author chance
 * @date 2024/11/29 09:49
 * @since 1.0
 */
public interface Content {

    /**
     * 获取内容的类型
     *
     * @return 内容的类型，如文本、图片、视频等
     */
    String getType();

    /**
     * 渲染内容
     * 将内容渲染为字符串形式，具体表现取决于内容的类型和实现
     *
     * @return 渲染后的内容字符串
     */
    String render();
}
```

#### 4.2 文本内容的具体实现

```java
/**
 * 文本内容的具体实现
 * <p>该类实现了{@link Content}接口，用于定义和管理文本类型的内容
 * 主要功能包括返回内容的类型标识和渲染内容的HTML格式
 *
 * @author chance
 * @date 2024/11/29 09:50
 * @since 1.0
 */
public class TextContent implements Content {

    /**
     * 存储文本内容的字符串变量
     */
    private final String text;

    /**
     * 构造函数，用于创建TextContent实例
     *
     * @param text 文本内容
     */
    public TextContent(String text) {
        this.text = text;
    }

    /**
     * 获取内容的类型
     *
     * @return 返回内容类型，此处固定为"text"
     */
    @Override
    public String getType() {
        return "text";
    }

    /**
     * 渲染文本内容为HTML格式
     * <p>
     * 此方法将文本内容封装在HTML的 <p> 标签中，以便在Web页面上显示
     *
     * @return 返回封装好的HTML格式文本内容
     */
    @Override
    public String render() {
        return "<p>" + text + "</p>";
    }
}
```

#### 4.3 图片内容的具体实现

```java
/**
 * 图片内容的具体实现
 * <p>该类实现了{@link Content}接口，用于处理和渲染图片内容
 *
 * @author chance
 * @date 2024/11/29 09:51
 * @since 1.0
 */
public class ImageContent implements Content {

    /**
     * 图片的URL地址
     */
    private final String imageUrl;

    /**
     * 构造函数，用于创建ImageContent对象
     *
     * @param imageUrl 图片的URL地址
     */
    public ImageContent(String imageUrl) {
        this.imageUrl = imageUrl;
    }

    /**
     * 获取内容类型
     * 重写Content接口的getType方法，返回图片类型
     *
     * @return 内容类型，此处固定为"image"
     */
    @Override
    public String getType() {
        return "image";
    }

    /**
     * 渲染内容
     * 重写Content接口的render方法，将图片内容渲染为HTML格式
     *
     * @return 渲染后的HTML字符串，包含img标签和图片URL
     */
    @Override
    public String render() {
        return "<img src=\"" + imageUrl + "\" alt=\"Image\">";
    }
}
```

#### 4.4 视频内容的具体实现

```java
/**
 * 视频内容的具体实现
 *
 * @author chance
 * @date 2024/11/29 09:55
 * @since 1.0
 */
public class VideoContent implements Content {

    /**
     * 视频的URL地址
     */
    private final String videoUrl;

    /**
     * 构造一个新的视频内容实例
     *
     * @param videoUrl 视频的URL地址
     */
    public VideoContent(String videoUrl) {
        this.videoUrl = videoUrl;
    }

    /**
     * 获取内容的类型
     *
     * @return 返回内容类型，此处固定为"video"
     */
    @Override
    public String getType() {
        return "video";
    }

    /**
     * 渲染内容为HTML格式
     * 对于视频内容，这将生成一个HTML视频标签
     *
     * @return 视频内容的HTML表示
     */
    @Override
    public String render() {
        return "<video controls><source src=\"" + videoUrl + "\" type=\"video/mp4\"></video>";
    }
}
```

#### 4.5 文章类（存储和渲染多种内容）

```java
/**
 * 文章，用来存储和渲染多种内容
 *
 * @author chance
 * @date 2024/11/29 09:57
 * @since 1.0
 */
public class Article {

    private List<Content> contents;

    public Article(List<Content> contents) {
        this.contents = contents;
    }

    public String render() {
        StringBuilder sb = new StringBuilder();
        for (Content content : contents) {
            sb.append(content.render());
        }
        return sb.toString();
    }
}
```

#### 4.6 测试

```java
log.info("构造文章");
Article article = new Article(Arrays.asList(
        new TextContent("Hello World"),
        new ImageContent("https://www.baidu.com"),
        new VideoContent("https://www.baidu.com")
));
log.info("文章渲染结果：{}", article.render());
```



### 五、电子表格软件

---

考虑一个简单的电子表格软件，其中每个单元格可以是数字、文本或公式。可以用抽象文档模式来设计单元格。

#### 5.1 单元格接口

```java
/**
 * 单元格接口
 * <p>定义了电子表格中单元格的基本操作和属性
 * 它提供了获取单元格值和显示单元格内容的方法
 *
 * @author chance
 * @date 2024/11/29 14:11
 * @since 1.0
 */
public interface Cell {

    /**
     * 获取单元格的值
     *
     * @return 单元格的值，类型为Object，以便可以存储各种类型的值
     */
    Object getValue();

    /**
     * 显示单元格的内容
     *
     * @return 以字符串形式返回单元格的内容，便于显示或进一步处理
     */
    String display();
}
```

#### 5.2 数字单元格

```java
/**
 * 数字单元格
 * <p>实现了{@link Cell}接口，用于表示Excel表格中的数字类型单元格
 *
 * @author chance
 * @date 2024/11/29 14:17
 * @since 1.0
 */
public class NumberCell implements Cell {

    private final double value;

    public NumberCell(double value) {
        this.value = value;
    }

    @Override
    public Object getValue() {
        return value;
    }

    @Override
    public String display() {
        return String.valueOf(value);
    }
}
```

#### 5.3 文本单元格

```java
/**
 * 文本单元格
 * <p>实现了{@link Cell}接口用于表示包含文本数据的单元格对象
 * @author chance
 * @date 2024/11/29 14:20
 * @since 1.0
 */
public class TextCell implements Cell {

    private final String text;

    public TextCell(String text) {
        this.text = text;
    }

    @Override
    public Object getValue() {
        return text;
    }

    @Override
    public String display() {
        return text;
    }
}
```

#### 5.4 公式单元格

```java
/**
 * 公式单元格
 * <p>实现了{@link Cell}接口，用于表示Excel表格中的公式类型单元格
 * 公式单元格包含一个公式字符串和一个计算结果
 *
 * @author chance
 * @date 2024/11/29 14:27
 * @since 1.0
 */
public class FormulaCell implements Cell {

    /**
     * 公式字符串，用于表示单元格中的计算公式
     */
    private final String formula;

    /**
     * 计算结果，公式计算后的数值结果
     */
    private final double result;


    /**
     * 构造函数，用于创建一个公式单元格对象
     *
     * @param formula 公式字符串
     * @param result  计算结果
     */
    public FormulaCell(String formula, double result) {
        this.formula = formula;
        this.result = result;
    }

    /**
     * 获取单元格的值
     * 对于公式单元格，返回的是计算结果
     *
     * @return 单元格的值（计算结果）
     */
    @Override
    public Object getValue() {
        return result;
    }

    /**
     * 显示单元格的内容
     * 对于公式单元格，以"=公式 (计算结果)"的格式显示
     *
     * @return 单元格的显示内容
     */
    @Override
    public String display() {
        return "=" + formula + " (" + result + ")";
    }
}
```

#### 5.5 表格类

```java
/**
 * 电子表格
 * <p>用于管理单元格对象
 * 它提供了设置单元格内容和获取单元格显示内容的方法
 *
 * @author chance
 * @date 2024/11/29 14:41
 * @since 1.0
 */
public class Spreadsheet {

    /**
     * 二维数组存储单元格对象
     */
    private Cell[][] cells;

    /**
     * 构造方法，初始化电子表格的行数和列数
     *
     * @param rows 行数
     * @param cols 列数
     */
    public Spreadsheet(int rows, int cols) {
        cells = new Cell[rows][cols];
    }

    /**
     * 设置指定位置的单元格内容
     *
     * @param row  行号
     * @param col  列号
     * @param cell 单元格对象
     */
    public void setCell(int row, int col, Cell cell) {
        cells[row][col] = cell;
    }

    /**
     * 获取指定位置的单元格显示内容
     * 如果该位置没有单元格对象，则返回空字符串
     *
     * @param row 行号
     * @param col 列号
     * @return 单元格的显示内容或空字符串
     */
    public String getDisplay(int row, int col) {
        if (cells[row][col] != null) {
            return cells[row][col].display();
        } else {
            return "";
        }
    }

}
```

#### 5.6 测试

```java
Spreadsheet spreadsheet = new Spreadsheet(1, 3);
spreadsheet.setCell(0, 0, new TextCell("Hello"));
spreadsheet.setCell(0, 1, new NumberCell(3));
spreadsheet.setCell(0, 2, new FormulaCell("1+2", 3));
log.info("表格渲染结果：{}", spreadsheet.getDisplay(0, 0) + spreadsheet.getDisplay(0, 1) + spreadsheet.getDisplay(0, 2));
// 输出：
// 表格渲染结果：Hello3.0=1+2 (3.0)
```

### 六、电子商务系统

---

#### 6.1 实现思路

对商品的属性特征拆分成**价格**、**重量**、**品牌**、**类型**几个特征接口，**不同类型的商品可以根据需要装配对应的属性特征**，本文用的是数码产品，只需要实现特征接口，就可以拥有操作该属性的能力，而且多个不同类型的商品都可以共用一个映射集合，使用key区分开，后期可以动态装配自己的商品属性，可以不断动态添加同一类型的商品(Mp3、耳机、耳塞、音箱等)。

- 数码产品
  - 商品分类
    - 电视
    - 平板
    - 手机
    - xxx

#### 6.2 商品类别

















































































































