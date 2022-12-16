### 一、注解类

---

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface AuditLogAspect {
	/**操作内容*/
  String operation();
  
  /**操作类型*/
  String oerationType();
  
  /**日志类型*/
  String logType();
  
  /**应用类型*/
  String appType();
}
```



### 二、切面类

---

```java
@Aspect
@Component
public class VinceAspectj {

  private Logger logger = LoggerFactory.getLogger(VinceAspectj.class);
  @Autowired private AuditLogService auditLogService;

  //  @Pointcut("execution(public * com.my.provider.*.*.service.*.*(..))")
  //使用自定义注解类的，进入切面  @Pointcut("@annotation(com.my.provider.system.annotation.AuditLogAspect)")
  public void auditLogPointCut() {}

  @AfterReturning(returning = "tag", pointcut = "auditLogPointCut()")
  public void auditLog(JoinPoint joinPoint, Object tag) {
    MethodSignature signature = (MethodSignature) joinPoint.getSignature();
    Method method = signature.getMethod();
    Object[] agrs = joinPoint.getArgs();
    String name = null;
    // 如果传参包含类且包含name属性 取出name的内容
    for (Object object : agrs) {
      if (object == null) {
        continue;
      }
      Integer nameIndex = object.toString().indexOf("name=");
      if (nameIndex > 0) {
        name = object.toString().substring(nameIndex + "name=".length()).split(",")[0];
      }
    }
    AuditLogAspect auditLogAspect = signature.getMethod().getAnnotation(AuditLogAspect.class);
    // 没有添加注释的不写入日志
    if (auditLogAspect == null) {
      return;
    } else {
      String userId = ApiUtils.currentUserId();//当前登录的机构ID。使用自己项目的
      String operation = auditLogAspect.operation();
      if (name != null) {
        operation = operation + ",名称为:" + name;
      }
      String logType = auditLogAspect.logType();
      String appType = auditLogAspect.appType();
      String operationType = auditLogAspect.operationType();
      //保存日志内容，自行替换自己的保存业务
      auditLogService.saveOperation(userId, operation, operationType, logType, appType);
    }
  }
}`

```



### 三、service业务类

---

```java
@Service
public class UserService extends BaseService<UserMapper, User> {
  @AuditLogAspect(operation = "添加用户", operationType = "处理", logType = "权限信息", appType = "4")
  public boolean insertUser(UserInsertReq userInsertReq) {
    //添加用户代码
  }
}
```



### 四、controller控制器

---

```java
@Api(tags = {"用户服务"})
@RestController
@RequestMapping(value = "/user", produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
@Validated
@Transactional
public class UserRestController extends SuperController {
  @Autowired private UserService userService;

  @ApiOperation("添加用户")
  @PostMapping("/insertUser")
  public ApiResponses<Boolean> insertUser(
    @RequestBody @Validated(UserInsertReq.Create.class) UserInsertReq userInsertReq) {

    return success(userService.insertUser(userInsertReq));
  }
}
```