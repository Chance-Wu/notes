`_type`是es早期版本的设计缺陷。

- 在5.x以前的版本里边，一个index下面是支持多个type的；
- 在6.x的版本里改为一个index只支持一个type，type可以自定义；
- 7.x的版本所有的type默认为_doc（自定义type也能用，但是会提示不推荐）



### 一、为什么要移除type

---

我们经常会用 MySQL 的知识来理解 es 的一些概念，比如将 es 的 index 类比成 database，将 type 类比成 table，但这个比喻实际上是不准确的。在 MySQL 中，table 之间是相互独立的，每个表有自己的 schema，每个表都可以有相同的列名，同时支持不同的类型，比如表 A 的 age 列是 tinyint，而表 B 的 age 列是 varchar(10)，但**在 es 中，相同名字的字段的 mapping 定义必须是一致的**，因为在底层 Lucene 只会存一份。

在这个基础上，如果在一个 index 内存了多个 type，且这些 type 之间只有极少共用的字段，会**使得数据过于离散**，从而**影响 Lucene 的压缩性能**。



### 二、es为了移除type做的工作

---

#### 2.1 elasticsearch 5.6.0

通过对 index 设置参数 `index.mapping.single_type: true` 就能够启用单 index 单 type 限制（一个 index 只能支持一个 type），同样该限制从 6.0 版本开始该限制会强制启用。

#### 2.2 elasticsearch 6.x

一个 index 只能支持一个 type，推荐的 type 名字为 _doc（这样可以在 API 方面向后兼容 7.x 。

在 Elasticsearch 6.8 中，引入了一个参数控制 type 开关：`include_type_name`，默认值为 true，表示仍使用 type，手动设置为 false 后，请求 es 的 API 将不再包含 type，而是使用类 `PUT /{index}/_doc/{id}` 的格式。

#### 2.3 elasticsearch 7.x

`include_type_name` 被默认置为 false，新的 index API 格式为 `PUT /{index}/_doc/{id}` 和 `POST {index}/_doc` 。需要注意的是，**_doc 并不是一个 type ，而仅仅是 API 请求路径中永久的一部分。**

#### 2.4 elasticsearch 8.x

在 Elasticsearch 8.x 中，`include_type_name` 已被删除，同时也表示 es 不再支持任何自定义 type 。
