### 一、仅判断是否存在时，select count 比 select 具体的列更好

---

如判断某个用户userId是否是会员。

#### 1.1 反例

先查从用户信息表查出用户记录，然后再去判断是否是会员。

```xml
<select id="selectUserByUserId" resultMap="BaseResultMap">
  selct user_id , vip_flag from  user_info where user_id =#{userId};
</select>
```

```java
boolean isVip (String userId){
  UserInfo userInfo = userInfoDAp.selectUserByUserId(userId);
  return UserInfo!=null && "Y".equals(userInfo.getVipFlag())
}
```

#### 1.2 正例

针对这种业务场景，更好的视线是直接select count一下，或者select limit 1。

```xml
<select id="countVipUserByUserId" resultType="java.lang.Integer">
  selct count(1) from  user_info where user_id =#{userId} and vip_flag ='Y';
</select>
```

```java
boolean isVip (String userId){
  int vipNum = userInfoDAp.countVipUserByUserId(userId);
  return vipNum>0
}
```



### 二、写sql时，只查需要用到的字段，还有通用的字段，拒绝select *

---

#### 2.1 反例

```sql
select * from user_info where user_id =#{userId};
```

#### 2.2 正例

```sql
select user_id , vip_flag from  user_info where user_id =#{userId};
```

- 节省资源、减少网络开销。
- 可能用到覆盖索引，减少回表，提高查询效率。









































































