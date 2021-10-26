### 一、什么是索引模板

索引模板：就是把已经创建好的某个索引的参数设置（settings）和索引映射（mapping）保存下来作为模板，在创建新索引时，指定要使用的模板名，就可以直接重用已经定义好的模板中的设置和映射。



#### 1.1 索引模板中的内容

---

>**settings**：指定index的配置信息，比如分片数、副本数，tranlog同步条件、refresh策略等信息。

>**mappings**：指定index的内部构建信息，主要有：
>
>1. `_source`：Source Field字段，ES为每个文档都保存了一份数据源，如果不开启，也就是"_source":{"enabled":false}，查询的时候就只会返回文档的ID，其他的文档内容需要通过Fields字段到索引中再次获取，效率很低，但若开启，索引的体积会更大，此时就可以通过Compress进行压缩，并通过includes、excludes等方式在field上进行限制——指定允许哪些字段存储到`_source`中，哪些不存储。
>2. `properties`：最重要的配置，是对索引结构和文档字段的设置。



### 二、创建索引模板

创建一个商品的索引模板示例。

ES6.0之后的版本：

```json
PUT _template/shop_template
{
  "index_patterns": ["shop*", "bar*"],  // 可以通过"shop*"和"bar*"来适配, template字段已过期
  "order": 0,                // 模板的权重, 多个模板的时候优先匹配用, 值越大, 权重越高
  "settings": {
    "number_of_shards": 1  // 分片数量, 可以定义其他配置项
  },
  "aliases": {
    "alias_1": {}          // 索引对应的别名
  },
  "mappings": {
    // ES 6.0开始只支持一种type, 名称为“_doc”
    "_doc": {
      "_source": {            // 是否保存字段的原始值
        "enabled": false
      },
      "properties": {        // 字段的映射
        "@timestamp": {    // 具体的字段映射
          "type": "date",           
          "format": "yyyy-MM-dd HH:mm:ss"
        },
        "@version": {
          "doc_values": true,
          "index": "false",   // 设置为false, 不索引
          "type": "text"      // text类型
        },
        "logLevel": {
          "type": "long"
        }
      }
    }
  }
}
```



### 三、查看索引模板

```
GET _template									// 查看所有模板
GET _template/temp*						// 查看与通配符相匹配的模板
GET _template/camp_sys_call_log_index,temp2			// 查看多个模板
GET _template/shop_template		// 查看指定模板
```

```
HEAD _template/camp_sys_call_log_index
```

- 存在，200 - OK
- 不存在，404 - Not Found



### 四、删除索引模板

```json
DELETE _template/camp_sys_call_log_index
```



### 五、模板的使用建议

#### 5.1 设置_source = false

---

我们只关心查询的评分结果，而不用查看原始文档的内容，就设置为false。这样既能节省磁盘空间并减少磁盘IO上的开销。

>可以把原始的数据存储在MySQL等数据库，从ES中得到文档的ID之后，再到相应的数据库中获取数据。



#### 5.2 设置dynamic = strict

---

如果我们的数据是结构化数据，就设置"dynamic":"strict"

>把动态类型判断设置为严格，也就是不允许ES为插入的数据进行动态类型设置，避免注入脏数据。



#### 5.3 使用keyword类型

---

如果只关心精确匹配，就设置test_field : {"type" : "keyword"}

> keyword要比text类型的性能更高，并且还能节省磁盘的存储空间。



### 六、按月划分索引进行日志存储

---

设置好模板后，按月设置索引后会自动匹配模板创建索引。