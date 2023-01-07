```java
@Service
public class RedisService {

  @Resource
  private StringRedisTemplate stringRedisTemplate;

  public R generateRkNumber() {
    LocalDate now = LocalDate.now();
    DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyyMMdd");
    String currentDate = formatter.format(now);
    String key = SequenceUtil.REDIS_RUKU_PREFIX + currentDate;
    return stringRedisTemplate.opsForValue().increment(key, 1);
  }
}
```

