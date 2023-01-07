### 一、注解类

---

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface AuditLog {
  /**菜单名称*/
  String menuName() default "";
}
```



### 二、切面类

---

```java
/**
 * @Description: AuditLogAspect
 * @Author: wuchenyang
 * @Date: 2022-12-28 15:12
 * @Version 1.0
 */
@Order(-10)
@Aspect
@Component
public class AuditLogAspect {

  public static final Logger logger = LoggerFactory.getLogger(AuditLogAspect.class);

  public static final int GENERAL_DOC = 3;

  @Resource
  private RedisService redisService;

  @Pointcut("execution(* com.ibm.scrm.operation.web..*Controller.*(..))) and @annotation(com.ibm.scrm.operation.annotation.AuditLog)")
  public void auditLogPointCut() {
    // do nothing
  }

  @Around(value = "auditLogPointCut()")
  public Object aroundMethod(ProceedingJoinPoint pjp) throws Throwable {
    Date searchDateTime = new Date();
    ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
    HttpServletRequest request = attributes.getRequest();
    MethodSignature signature = (MethodSignature) pjp.getSignature();

    AuditLog auditLogAnnotation = signature.getMethod().getAnnotation(AuditLog.class);
    String params = "";
    if (auditLogAnnotation != null) {
      Object[] agrs = pjp.getArgs();
      // POST参数通过jp.getArgs()获取
      if (("POST").equals(request.getMethod())) {
        params = JSON.toJSONString(JSON.toJSONString(agrs[0]));
      } else {
        //GET参数使用getParameterMap获取参数
        Map<String, String[]> parameterMap = request.getParameterMap();
        StringBuilder getParam = new StringBuilder();
        for (String key : parameterMap.keySet()) {
          String[] value = parameterMap.get(key);
          getParam.append(key + ":" + value[0] + ";");
        }
        params = getParam.toString();
      }
    }
    Object resultObj = pjp.proceed();

    if (auditLogAnnotation != null) {
      String brandCode = "";
      String operationUser = "";
      //获取当前登录用户信息
      User user = SecurityContext.getUserPrincipal();
      if (null != user) {
        brandCode = user.getOrgCode();
        operationUser = user.getName();
      }
      Integer searchResultNum = getResultNum(resultObj);
      String operationUserClientIp = IPUtils.getIpAddr(request);
      DocumentLogVo documentLogVo = DocumentLogVo.builder()
        .id(redisService.generateEsSequence())
        .type(GENERAL_DOC)
        .brandCode(brandCode)
        .menuName(auditLogAnnotation.menuName())
        .searchConditions(params)
        .searchDateTime(searchDateTime)
        .operationUser(operationUser)
        .searchResultNum(searchResultNum)
        .operationUserClientIp(operationUserClientIp)
        .createTime(new Date())
        .build();
      EsUtils.addAuditLog(documentLogVo);
    }
    return resultObj;
  }

  /**
   * 获取结果数量
   *
   * @param resultObj
   * @return
   */
  private Integer getResultNum(Object resultObj) {
    ResponseBean<T> responseBean = JSON.parseObject(JSON.toJSONString(resultObj), ResponseBean.class);
    String jsonString = JSON.toJSONString(responseBean.getResponseBody());
    Map<String, Object> map = JSON.parseObject(jsonString, Map.class);
    return (Integer) map.get("total");
  }

}
```
