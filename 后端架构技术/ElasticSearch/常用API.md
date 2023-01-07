#### 删除索引下所有数据

---

```json
POST ${索引名称}/_delete_by_query
{“query”:{“match_all”:{}}}
```

