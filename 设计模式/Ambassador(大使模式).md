### 一、意图

---

通常用于在分布式系统中处理远程调用或与外部服务交互。通过引入一个大使对象，可以在实际调用服务之前进行一些预处理，比如日志记录、验证、安全检查或重试机制，从而增强系统的稳定性和可维护性。



### 二、详解

---

通过代理对象封装与远程服务的交互，隐藏复杂性并在本地实现对远程调用的控制。它类似于代理模式，但更强调增强和远程调用的细节。

通过该模式，可以实现来自客户端的较低频率轮询以及延迟检查和日志记录。



### 三、实例

---

#### 3.1 场景说明

需要查询某个商品的库存情况，但该库服务是一个远程服务，调用可能失败、延迟或返回错误。通过大使模式，可以实现**重试机制**、**日志记录**和**性能监控**。

#### 3.2 定义远程库存服务接口

```java
/**
 * 库存服务接口
 *
 * @author chance
 * @date 2024/12/10 13:46
 * @since 1.0
 */
public interface InventoryService {

    /**
     * 获取库存
     * 根据产品ID查询当前的库存数量
     *
     * @param productId 产品ID，用于标识特定的产品
     * @return 当前产品的库存数量
     */
    int getStock(String productId);
}
```

#### 3.3 实现远程库存服务

```java
public class InventoryServiceImpl implements InventoryService {

    @Override
    public int getStock(String productId) {
        System.out.println("Fetching stock for product: " + productId);
        // 模拟服务响应时间
        try {
            // 模拟网络延迟
            Thread.sleep(100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        // 模拟库存数据
        if ("P123".equals(productId)) {
            return 50; // 商品库存
        } else {
            throw new RuntimeException("Product not found!");
        }

    }
}
```

#### 3.4 创建大使类

封装对远程服务的调用，提供日志记录、重试和性能监控。

```java
/**
 * InventoryServiceAmbassador 类作为 InventoryService 的大使模式实现，旨在解耦客户端与库存服务的直接交互。
 * 它不仅处理库存查询逻辑，还负责输入验证、错误处理和重试逻辑，从而保护服务免受无效请求的影响，并增强系统的健壮性。
 *
 * @author chance
 * @date 2024/12/10 14:33
 * @since 1.0
 */
public class InventoryServiceAmbassador {

    private final InventoryService inventoryService;

    public InventoryServiceAmbassador(InventoryService inventoryService) {
        this.inventoryService = inventoryService;
    }

    /**
     * 根据产品ID获取库存数量。
     * 此方法首先验证产品ID的有效性，然后尝试调用 inventoryService 的 getStock 方法获取库存。
     * 如果因任何原因查询失败，将重试最多两次。如果所有尝试均失败，将返回 -1 表示错误。
     *
     * @param productId 产品的唯一标识符，用于查询库存。
     * @return 产品的库存数量，如果查询失败或产品ID无效，返回 -1。
     */
    public int getStock(String productId) {
        // 验证输入参数的有效性
        System.out.println("Ambassador: Validating input...");
        if (productId == null || productId.isEmpty()) {
            System.out.println("Ambassador: Invalid product ID.");
            // 表示错误
            return -1;
        }

        // 记录开始时间，用于计算整个过程的耗时
        long startTime = System.currentTimeMillis();
        // 初始化尝试次数
        int attempts = 0;
        // 定义最大重试次数
        int maxRetries = 2;

        // 尝试查询库存，最多重试两次
        while (attempts <= maxRetries) {
            attempts++;
            try {
                // 尝试调用库存服务
                System.out.println("Ambassador: Forwarding request to inventory service (attempt " + attempts + ")...");
                int stock = inventoryService.getStock(productId);
                System.out.println("Ambassador: Stock fetched successfully.");
                return stock;
            } catch (Exception e) {
                // 处理查询过程中出现的异常
                System.out.println("Ambassador: Error occurred - " + e.getMessage());
                if (attempts > maxRetries) {
                    System.out.println("Ambassador: Max retries reached. Returning error.");
                    // 表示错误
                    return -1;
                }
                System.out.println("Ambassador: Retrying...");
            }
        }

        // 计算并打印整个过程的耗时
        long elapsedTime = System.currentTimeMillis() - startTime;
        System.out.println("Ambassador: Process took " + elapsedTime + " ms.");
        // 如果未成功，返回错误码
        return -1;
    }
}
```

#### 3.5 测试

```java
// 创建远程服务实例
InventoryService remoteService = new InventoryServiceImpl();

// 创建大使类实例
InventoryServiceAmbassador ambassador = new InventoryServiceAmbassador(remoteService);

// 模拟业务调用
System.out.println("Client: Checking stock for product P123...");
int stock = ambassador.getStock("P123");
if (stock >= 0) {
    System.out.println("Client: Product stock: " + stock);
} else {
    System.out.println("Client: Failed to fetch product stock.");
}

System.out.println("\nClient: Checking stock for an invalid product...");
stock = ambassador.getStock("INVALID");
if (stock >= 0) {
    System.out.println("Client: Product stock: " + stock);
} else {
    System.out.println("Client: Failed to fetch product stock.");
}

// 输出：
// Client: Checking stock for product P123...
// Ambassador: Validating input...
// Ambassador: Forwarding request to inventory service (attempt 1)...
// Fetching stock for product: P123
// Ambassador: Stock fetched successfully.
// Client: Product stock: 50
// 
// Client: Checking stock for an invalid product...
// Ambassador: Validating input...
// Ambassador: Forwarding request to inventory service (attempt 1)...
// Fetching stock for product: INVALID
// Ambassador: Error occurred - Product not found!
// Ambassador: Retrying...
// Ambassador: Forwarding request to inventory service (attempt 2)...
// Fetching stock for product: INVALID
// Ambassador: Error occurred - Product not found!
// Ambassador: Retrying...
// Ambassador: Forwarding request to inventory service (attempt 3)...
// Fetching stock for product: INVALID
// Ambassador: Error occurred - Product not found!
// Ambassador: Max retries reached. Returning error.
// Client: Failed to fetch product stock.
```



### 四、适用业务场景

---

>- Ambassador 模式对于 Java 中的云原生和微服务架构特别有益。它有助于监控、记录和保护服务间通信，使其成为分布式系统的理想选择。
>- 遗留系统集成：通过处理必要但非核心的功能促进与新服务的通信。
>- 性能增强：可用于缓存结果或者压缩数据以提高通信效率。

- **支付网关集成**：封装调用支付服务的复杂逻辑。
- **短信服务发送**：处理与外部短信服务商的交互。
- **分布式服务调用**：在微服务架构中调用其他微服务时的失败处理。
- **远程API封装**：对接第三方 API 时实现统一入口和增强。
- 控制对另一个对象的访问
- 实施日志记录
- **实施熔断**
- 卸载远程服务任务
- 促进网络连接

- 服务网格实现：在 Istio 或 Linkerd 等服务网格架构中，Ambassador 模式通常用作处理服务间通信的 Sidecar 代理。这包括服务发现、路由、负载平衡、遥测（指标和跟踪）和安全（身份验证和授权）等任务。
- API 网关：API 网关可以使用 Ambassador 模式来封装常见功能，例如速率限制、缓存、请求整形和身份验证。这样一来，后端服务就可以专注于其核心业务逻辑，而不必担心这些跨切关注点。
- 日志记录和监控：Ambassador 可以汇总来自各种服务的日志和指标，并将其转发到 Prometheus 或 ELK Stack（Elasticsearch、Logstash、Kibana）等集中式监控工具。这简化了每个服务的日志记录和监控设置，并提供了系统运行状况的统一视图。
- 安全性：SSL/TLS 终止、身份验证和加密等安全相关功能可由 Ambassador 管理。这可确保跨服务的安全实践一致，并降低因配置错误而导致安全漏洞的可能性。
- 弹性：Ambassador 可以实现断路器、重试和超时等弹性模式。例如，Netflix 的 Hystrix 库可以在 Ambassador 中使用，以防止微服务生态系统中出现级联故障。
- 数据库代理：大使可以充当数据库连接的代理，提供连接池、副本的读/写拆分和查询缓存等功能。这大大减轻了应用服务的复杂性。
- 遗留系统集成：在现代微服务需要与遗留系统通信的情况下，大使可以充当中介，翻译协议，处理必要的转换并实施现代安全实践，从而简化集成过程。
- 网络优化：对于部署在不同地理位置或云区域的服务，大使可以通过压缩数据、批处理请求甚至实施智能路由来优化通信，以减少延迟和成本。



### 五、优缺点

---

#### 5.1 优点

- 关注点分离：从服务逻辑中卸载横切关注点，从而产生更清晰、更易于维护的代码。
- 可重复使用的基础设施逻辑：大使模式允许在多个服务中重复使用相同的逻辑（例如，日志记录、监控）。
- 提高的安全性：集中 SSL 终止或身份验证等安全功能，降低配置错误的风险。
- 灵活性：无需修改服务代码即可更轻松地更新或替换基础设施问题。

#### 5.2 缺点

- 关注点分离：从服务逻辑中卸载横切关注点，从而产生更清晰、更易于维护的代码。
- 可重复使用的基础设施逻辑：大使模式允许在多个服务中重复使用相同的逻辑（例如，日志记录、监控）。
- 提高的安全性：集中 SSL 终止或身份验证等安全功能，降低配置错误的风险。
- 灵活性：无需修改服务代码即可更轻松地更新或替换基础设施问题。