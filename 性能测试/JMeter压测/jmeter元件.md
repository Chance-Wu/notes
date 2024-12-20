### 一、主要元件

---

1. 测试计划：是使用JMeter进行测试的起点，它是其他JMeter测试元件的容器。
2. **线程组**：代表一定数量的用户，它可以用来模拟用户并发发送请求。实际的请求内容在 Sampler 中定义，它被线程组包含。
3. **配置元件**：维护Sampler需要的配置信息，并根据实际的需要修改请求的内容。
4. 前置处理器：负责在请求之前工作，常用来修改请求的设置。
5. 定时器：负责定义请求之间的延迟间隔。
6. **取样器(Sampler)**：是性能测试中向服务器发送请求，记录响应信息、响应时间的最小单元，如：HTTP Request Sampler、FTP Request Sample、TCP Request Sample、DBC Request Sampler等，每一种不同类型的sampler 可以根据设置的参数向服务器发出不同类型的请求。
7. 后置处理器：负责在请求之后工作，常用获取返回的值。
8. 断言：用来判断请求响应的结果是否如用户所期望的。
9. 监听器：负责收集测试结果，同时确定结果显示的方式。
10. 逻辑控制器：可以自定义JMeter发送请求的行为逻辑，它与Sampler结合使用可以模拟复杂的请求序列。



### 二、元件的作用域和执行顺序

---

#### 2.1 元件作用域

- 配置元件：影响其作用范围内的所有元件。
- 前置处理器：在其作用范围内的每一个sampler元件之前执行。
- 定时器：在其作用范围内的每一个sampler有效
- 后置处理器：在其作用范围内的每一个sampler元件之后执行。
- 断言：在其作用范围内的对每一个sampler元件执行后的结果进行校验。
- 监听器：在其作用范围内对每一个sampler元件的信息收集并呈现。

#### 2.2 元件执行顺序

配置元件->前置处理器->定时器->取样器->后置处理程序->断言->监听器