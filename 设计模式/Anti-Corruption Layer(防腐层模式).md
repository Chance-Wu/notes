### 一、意图

---

ACL是Java开发中一个至关重要的设计模式，尤其适用于系统集成和维护数据得完整性。在不共享相同语义的子系统之间实现外观层或适配器层。它可以在不同的数据格式和系统之间进行转换，确保系统之间的集成不会导致业务逻辑或数据完整性的损坏。



### 二、详解

---

#### 2.1 核心目标

隔离系统边界，保护内部系统不受外部系统变化的影响。

#### 2.2 关注点

系统边界、领域模型转换、异构系统集成。



### 三、实例

---

#### 3.1 场景描述

1. 系统需要向一个第三方支付服务发起支付请求，并接收支付结果。
2. 第三方支付服务的接口数据格式与领域模型不一致。
3. 使用防腐层模式保护领域模型，隔离外部系统的复杂性。

#### 3.2 领域模型

**支付请求模型：**

```java
@Data
public class PaymentRequest {

    private String orderId;

    private double amount;

    private String currency;
}
```

**支付结果模型：**

```java
@NoArgsConstructor
@AllArgsConstructor
@Data
public class PaymentResult {
    
    private String transactionId;
    
    private boolean success;
    
    private String message;
}
```

#### 3.3 第三方支付系统的接口和模型

**外部支付请求：**

```java
@Data
public class ExternalPaymentRequest {

    private String externalOrderId;
    
    private double totalAmount;
    
    private String currencyCode;
}
```

**外部支付响应：**

```java
@AllArgsConstructor
@NoArgsConstructor
@Data
public class ExternalPaymentResponse {

    private String transactionId;

    private boolean status;

    private String errorMessage;
}
```

#### 3.4 防腐层接口与实现

```java
/**
 * 防腐层接口
 * 防腐层的设计目的是在系统的不同部分之间提供一个解耦层，
 * 以减少系统组件之间的直接依赖，提高系统的可维护性和扩展性。本接口主要用于处理支付请求，
 * 通过封装支付处理逻辑，对外提供统一的支付处理接口，同时隐藏内部实现细节，增加系统的灵活性和容错能力。
 *
 * @author chance
 * @date 2024/12/10 15:58
 * @since 1.0
 */
public interface AntiCorruptionLayerService {

    /**
     * 处理支付请求
     * 本方法接收一个支付请求对象，对支付请求进行处理，并返回支付结果。通过本方法的实现，
     * 可以将支付处理逻辑与系统的其他部分隔离开来，减少支付系统与其他系统组件之间的直接依赖，
     * 提高系统的可维护性和扩展性。
     *
     * @param paymentRequest 支付请求对象，包含支付所需的信息
     * @return PaymentResult 支付结果对象，表示支付的处理结果
     */
    PaymentResult processPayment(PaymentRequest paymentRequest);
}
```

**防腐层实现：**

```java
/**
 * 防腐层实现
 * 该类的主要作用是作为领域模型与外部系统（如支付API）之间的适配器，实现两者的解耦
 * 通过本类，将领域模型转换为外部系统所需的格式，发送请求，并将外部系统的响应转换回领域模型
 *
 * @author chance
 * @date 2024/12/10 16:00
 * @since 1.0
 */
public class AntiCorruptionLayerServiceImpl implements AntiCorruptionLayerService {

    /**
     * 外部支付API接口
     */
    private final ExternalPaymentApi externalPaymentApi;

    /**
     * 构造函数，注入外部支付API
     *
     * @param externalPaymentApi 外部支付API实例
     */
    public AntiCorruptionLayerServiceImpl(ExternalPaymentApi externalPaymentApi) {
        this.externalPaymentApi = externalPaymentApi;
    }

    /**
     * 处理支付请求
     * 该方法首先将领域模型的支付请求转换为外部系统可接受的格式，然后调用外部支付API处理支付
     * 如果外部API调用失败，将捕获异常并返回失败结果
     *
     * @param paymentRequest 支付请求领域模型
     * @return 支付结果领域模型
     */
    @Override
    public PaymentResult processPayment(PaymentRequest paymentRequest) {
        // 转换领域模型到外部系统模型
        ExternalPaymentRequest externalRequest = mapToExternalRequest(paymentRequest);

        // 调用外部支付接口
        ExternalPaymentResponse response;
        try {
            response = externalPaymentApi.processPayment(externalRequest);
        } catch (Exception e) {
            // 处理外部系统调用异常
            return new PaymentResult(null, false, "Payment failed: " + e.getMessage());
        }

        // 转换外部系统响应到领域模型
        return mapToDomainResult(response);
    }

    /**
     * 数据转换：领域模型 -> 外部模型
     * 将支付请求的领域模型转换为外部支付系统所需的请求格式
     *
     * @param request 支付请求领域模型
     * @return 外部支付请求模型
     */
    private ExternalPaymentRequest mapToExternalRequest(PaymentRequest request) {
        ExternalPaymentRequest externalRequest = new ExternalPaymentRequest();
        externalRequest.setExternalOrderId(request.getOrderId());
        externalRequest.setTotalAmount(request.getAmount());
        externalRequest.setCurrencyCode(request.getCurrency());
        return externalRequest;
    }

    /**
     * 数据转换：外部模型 -> 领域模型
     * 将外部支付系统的响应转换为领域模型的支付结果
     *
     * @param response 外部支付响应模型
     * @return 支付结果领域模型
     */
    private PaymentResult mapToDomainResult(ExternalPaymentResponse response) {
        if (response.isStatus()) {
            return new PaymentResult(response.getTransactionId(), true, "Payment successful");
        } else {
            return new PaymentResult(null, false, response.getErrorMessage());
        }
    }
}
```

```java
/**
 * 外部系统API模拟
 *
 * @author chance
 * @date 2024/12/10 16:01
 * @since 1.0
 */
public class ExternalPaymentApi {

    public ExternalPaymentResponse processPayment(ExternalPaymentRequest request) {
        // 模拟外部系统的支付处理
        ExternalPaymentResponse response = new ExternalPaymentResponse();
        response.setTransactionId("TX123456789");
        response.setStatus(true); // 模拟支付成功
        response.setErrorMessage(null);
        return response;
    }
}
```

#### 3.5 应用服务使用防腐层

```java
public class OrderService {

    private final AntiCorruptionLayerService antiCorruptionLayerService;

    public OrderService(AntiCorruptionLayerService antiCorruptionLayerService) {
        this.antiCorruptionLayerService = antiCorruptionLayerService;
    }

    public void processOrder(String orderId, double amount, String currency) {
        PaymentRequest paymentRequest = new PaymentRequest();
        paymentRequest.setOrderId(orderId);
        paymentRequest.setAmount(amount);
        paymentRequest.setCurrency(currency);

        // 调用防腐层
        PaymentResult result = antiCorruptionLayerService.processPayment(paymentRequest);

        if (result.isSuccess()) {
            System.out.println("Payment processed successfully: " + result.getTransactionId());
        } else {
            System.out.println("Payment failed: " + result.getMessage());
        }
    }
}
```

#### 3.6 测试

```java
// 创建外部系统API和防腐层实例
ExternalPaymentApi externalPaymentApi = new ExternalPaymentApi();
AntiCorruptionLayerService antiCorruptionLayerService = new AntiCorruptionLayerServiceImpl(externalPaymentApi);

// 创建订单服务
OrderService orderService = new OrderService(antiCorruptionLayerService);

// 模拟订单处理
orderService.processOrder("ORDER123", 100.00, "USD");

// 输出：
// Payment processed successfully: TX123456789
```



### 四、应用场景

---

- 遗留系统集成：将新系统与旧的、技术栈不同的系统集成。
- 第三方服务调用：调用外部的REST API、SOAP服务等。
- 异构系统交互：在不同的微服务之间进行数据交换。



### 五、优缺点

---

#### 5.1 优点

- **提高系统可维护性：** 隔离外部系统的变化，降低系统维护成本。
- **增强系统可扩展性：** 灵活应对外部系统的变化。
- **保护核心业务逻辑：** 防止外部系统的干扰。

#### 5.2 缺点

- **增加系统复杂性：** 引入了一层额外的抽象。
- **性能开销：** 数据转换和适配会带来一定的性能损耗。



### 六、与适配器和外观模式的区别

---

| 特点     | 防腐层       | 适配器模式   | 外观模式       |
| -------- | ------------ | ------------ | -------------- |
| 目标     | 隔离系统边界 | 接口转换     | 接口简化       |
| 关注点   | 领域模型转换 | 接口兼容性   | 子系统封装     |
| 应用场景 | 异构系统集成 | 类接口不兼容 | 子系统接口复杂 |