### 服务器相关

---

- info [section]：返回redis相关信息
- config get dir/* 实时传递接收的请求
- showlog：显示慢查询
- select n：切换到数据库n，redis默认有16个数据库（DB 0~DB 15），默认使用的第0个
- dbsize：查看当前数据库大小



### key相关命令

-  keys * ：查看当前数据库中所有的key
- dbsize： 键总数
- exists key： 检查键是否存在
- del key [key …]： 删除键
- expire key seconds： 键过期
- ttl key： 获取键的有效时长
- persist key： 移除键的过期时间
- type key： 键的数据结构类型