### mapper

---

```java
`@Select("SELECT u.* , ur.role_id , r.role_name from sys_user_role ur , sys_role r , sys_user u ,sys_user_depart ud " + "where ur.role_id = r.id and ur.user_id = u.id and u.id = ud.user_id " + "and ud.create_user_id in (${createUserId})")
public Page<SysRoleDeptVO> getUserByCreateUserIds(Page page, @Param("createUserId") String createUserId);
```



### service

---

```java
@Override
public Page<SysRoleDeptVO> getByUserIds(Page<SysRoleDeptVO> page, List<String> userIds) {
  /** 如果当前部门下没用户的话,就传个 ""  过去   由于 mybatis 解析后空字符串
   *  后是什么都没有, 这里用  in ()  如果这样,就会异常 ,所以当用户为空的话,
   *  传 "''"  这样解析后 变成了 in ('')
   **/
  String userids = "''" ;
  if(userIds != null && userIds.size() != 0){
    StringBuilder stringBuilder = new StringBuilder("");
    for (int i = 0; i < userIds.size(); i++) {
      stringBuilder.append("'");
      stringBuilder.append(userIds.get(i));
      stringBuilder.append("'");
      stringBuilder.append(",");
    }
    userids = stringBuilder.substring(0, stringBuilder.length() - 1);
  }
  return  userMapper.getByUserIds(page,userids);
```

mapper防止转义字符

>`<![CDATA[]]>`



### mybatis在@Select写IN SQL

---

```java
@Select("<script>" +
        "select * from article where id in " +
        "<foreach item='item' index='index' collection='articleIds' open='(' separator=', ' close=')'>" +
        "#{item}" +
        "</foreach>" +
        "</script>")
List<Article> getArticlesByIds(@Param("articleIds") List<Long> articleIds);
```



### 解决方法

---

注解不支持 `<foreach>` 或其他动态 SQL 标签。所以，如果你需要在 `@Select` 注解中使用 `IN` 语句并且传入一个集合，你可能需要写一个自定义的方法来生成 SQL 字符串，然后使用 `@Select` 注解。

然而，在 Java 8 及以上版本中，你可以利用 Stream API 和 `String.join()` 方法来生成逗号分隔的 ID 列表，然后将其作为参数传递给 `@Select` 注解：

```java
public List<Item> selectItemsByIds(List<Integer> ids) {
  String idList = ids.stream()
    .map(String::valueOf)
    .collect(Collectors.joining(","));
  return sqlSession.selectList("com.example.ItemMapper.selectItemsByIds", idList);
}

@Select("SELECT * FROM items WHERE id IN (${idList})")
List<Item> selectItemsByIds(@Param("idList") String idList);
```

请注意，这种方式同样存在 SQL 注入的风险，因为它使用了 `${}` 来插入参数。在实际应用中，推荐使用 XML 配置文件中的 `<foreach>` 标签来安全地处理集合。