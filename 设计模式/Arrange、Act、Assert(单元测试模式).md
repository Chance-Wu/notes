### 一、意图

---

Arrange/Act/Assert 模式在 Java 单元测试中必不可少。此测试方法将单元测试分为三个不同的部分，从而清晰地构建单元测试：**设置（Arrange）**、**执行（Act）**和**验证（Assert）**。



### 二、详解

---

Arrange/Act/Assert (AAA) 是一种常用的单元测试设计模式，旨在提高测试代码的可读性和可维护性。它将测试代码分为三个清晰的阶段：

- **Arrange:** 准备测试环境，包括创建测试对象、设置输入数据等。
- **Act:** 执行被测试的代码。
- **Assert:** 验证实际结果与预期结果是否一致。



### 三、实例

---

```java
public class CalculatorTest {

    @Test
    public void testAdd() {
        // Arrange
        Calculator calculator = new Calculator();
        int a = 5;
        int b = 3;

        // Act
        int result = calculator.add(a, b);

        // Assert
        assertEquals(8, result);
    }
}
```



### 四、应用

---

- **JUnit:** JUnit 是 Java 中最流行的单元测试框架，它提供了丰富的断言方法和注解来支持 AAA 模式。
- **TestNG:** TestNG 是另一个功能强大的测试框架，它也支持 AAA 模式，并提供了一些额外的特性。
