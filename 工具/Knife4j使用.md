### 一、集成knife4j

---

SpringBoot2+OpenApi3

#### 1.1 引入依赖

```xml
<dependency>
  <groupId>com.github.xiaoymin</groupId>
  <artifactId>knife4j-openapi3-spring-boot-starter</artifactId>
  <version>4.4.0</version>
</dependency>
```

#### 1.2 Springdoc配置

由于使用该版本的 Knife4J 是基于 Springdoc 的，所以可以使用Springdoc的配置来自定义一部分效果。

| 参数名称                                       | 默认值 | 参数描述                                                |
| ---------------------------------------------- | ------ | ------------------------------------------------------- |
| springdoc.api-docs.enabled                     | true   | 禁用springdoc-openapi端点(默认为/v3/api-docs)           |
| springdoc.packages-to-scan                     | *      | 字符串列表。要扫描的包列表(逗号分隔)                    |
| springdoc.packages-to-exclude                  |        | 字符串列表。要排除的包列表(以逗号分隔)                  |
| springdoc.paths-to-match                       | /*     | 字符串列表。要匹配的路径列表(以逗号分隔)                |
| springdoc.paths-to-exclude                     |        | 字符串列表。要排除的路径列表(以逗号分隔)                |
| springdoc.show-actuator                        | false  | 显示actuator端点                                        |
| springdoc.auto-tag-classes                     | true   | 自动从应用中扫描并创建 Swagger 文档中的标签（tags）     |
| springdoc.api-docs.groups.enabled              | true   | 禁用springdoc-openapi组                                 |
| springdoc.group-configs[0].group               |        | 组名                                                    |
| springdoc.group-configs[0].display-name        |        | 组名组的显示名称                                        |
| springdoc.group-configs[0].packages-to-scan    | *      | 要扫描的包列表(以逗号分隔)                              |
| springdoc.group-configs[0].packages-to-exclude |        | 组中要排除的包列表(用逗号分隔)                          |
| springdoc.group-configs[0].paths-to-match      | /*     | 配置特定分组应该包含哪些路径的属性                      |
| springdoc.api-docs.resolve-schema-properties   | false  | 在@Schema (name, title and description)上启用属性解析器 |
| springdoc.pre-loading-enabled                  | false  | 在应用程序启动时加载OpenAPI的预加载设置。               |
| springdoc.writer-with-order-by-keys            | false  | 控制生成的 OpenAPI 文档中 JSON 对象键的排序方式         |
| springdoc.disable-i18n                         | false  | 使用i18n禁用自动翻译                                    |
| springdoc.show-spring-cloud-functions          | true   | 显示spring-cloud-function web端点                       |
| springdoc.swagger-ui.enabled                   | true   | 禁用swagger-ui端点(默认为/swagger-ui.html)              |

```yaml
# springdoc-openapi项目配置
springdoc:
  pre-loading-enabled: false
  writer-with-order-by-keys: true
  swagger-ui:
    path: /swagger-ui.html
    tags-sorter: alpha
    operations-sorter: alpha
  api-docs:
    path: /v3/api-docs
  group-configs:
    - group: 'ALL'
      display-name: '全部'
      paths-to-match: '/**'
      packages-to-scan: com.chance.controller
    - group: 'COURSE'
      display-name: '课程'
      paths-to-match: '/**'
      packages-to-scan: com.chance.controller.course
    - group: 'FILE'
      display-name: '文件'
      paths-to-match: '/**'
      packages-to-scan: com.chance.controller.file
    - group: 'TEST'
      display-name: '测试'
      paths-to-match: '/**'
      packages-to-scan: com.chance.controller.test
```

#### 1.3 knife4j配置















































#### 1.4 访问

访问Knife4j的文档地址：`http://ip:port/doc.html` 即可查看文档。

访问swagger-ui页面：`http://ip:port/swagger-ui/index.html`



### 二、接口添加作者

---

#### 2.1 接口上添加

```java
@ApiOperationSupport(author = "chance")
```

#### 2.2 controller上添加

```java
@ApiSupport(author = "chance")
```



### 三、接口排序

```java
@ApiOperationSupport(order = 1)
```



### 四、OpenApi3注解

---

| Swagger3                                                     | 注解说明                                              |
| ------------------------------------------------------------ | ----------------------------------------------------- |
| @Tag(name = “接口类描述”)                                    | Controller 类                                         |
| @Operation(summary =“接口方法描述”)                          | Controller 方法                                       |
| @Parameters                                                  | Controller 方法                                       |
| @Parameter(description=“参数描述”)                           | Controller 方法上 @Parameters 里Controller 方法的参数 |
| @Parameter(hidden = true) 、@Operation(hidden = true)@Hidden | 排除或隐藏api                                         |
| @Schema                                                      | DTO实体DTO实体属性                                    |

#### 4.1 示例

```java
@RestController
@RequestMapping("body")
@Tag(name = "body参数")
public class BodyController {

  @Operation(summary = "普通body请求")
  @PostMapping("/body")
  public ResponseEntity<FileResp> body(@RequestBody FileResp fileResp){
    return ResponseEntity.ok(fileResp);
  }

  @Operation(summary = "普通body请求+Param+Header+Path")
  @Parameters({
    @Parameter(name = "id",description = "文件id",in = ParameterIn.PATH),
    @Parameter(name = "token",description = "请求token",required = true,in = ParameterIn.HEADER),
    @Parameter(name = "name",description = "文件名称",required = true,in=ParameterIn.QUERY)
  })
  @PostMapping("/bodyParamHeaderPath/{id}")
  public ResponseEntity<FileResp> bodyParamHeaderPath(@PathVariable("id") String id,@RequestHeader("token") String token, @RequestParam("name")String name,@RequestBody FileResp fileResp){
    fileResp.setName(fileResp.getName()+",receiveName:"+name+",token:"+token+",pathID:"+id);
    return ResponseEntity.ok(fileResp);
  }
}
```