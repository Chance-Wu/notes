【参考】：

https://tech.meituan.com/2017/04/21/mt-leaf.html

https://github.com/Meituan-Dianping/Leaf/blob/master/README_CN.md



>在复杂分布式系统中，往往需要对大量的数据和消息进行**唯一标识**。
>
>同时，数据日渐增长，对数据分库分表后需要有一个唯一ID来标识一条数据或消息，数据库的自增ID显然不能满足需求；比如订单、优惠券都需要有唯一ID做标识。此时一个能够生成全局唯一ID的系统是非常必要的。
>
>业务系统对ID号的要求如下：
>
>1. 全局唯一性：不能出现重复的ID号。
>2. 趋势递增：在MySQL InnoDB引擎中使用的是聚集索引，由于多数RDBMS



































































