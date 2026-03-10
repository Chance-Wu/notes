>这个模块实现了一些专门化的容器，提供了对 Python 的通用内建容器 `dict`、`list`、`set` 和 `tuple` 的补充。

|             |                                                          |
| ----------- | -------------------------------------------------------- |
| namedtuple  | 一个工厂函数，用来创建元组的字类，字类的字段是有名称的。 |
| deque       | 类似列表的容器，但 append 和 pop 在其两端的速度都很快。  |
| ChainMap    | 类似字典的类，用于创建包含多个映射的单个视图。           |
| Counter     | 用于计数 hashable 对象的字典字类。                       |
| OrderedDict | 字典的子类，能记住条目被添加进去的顺序。                 |
| defaultdict | 字典的子类，通过调用用户指定的工厂函数，为键提供默认值。 |
| UserDict    | 封装了字典对象，简化了字典字类化。                       |
| UserList    | 封装了列表对象，简化了列表子类化。                       |
| UserString  | 封装了字符串对象，简化了字符串字类化。                   |



## defaultdict

---

### 一、语法

defaultdict来自：

```python
from collections import defaultdict
```

本质是：

>一个“带默认值生成规则”的 dict

普通 dict：

```python
d = {}
d["a"] += 1   # ❌ KeyError
```

defaultdict：

```python
from collections import deafultdict

d =defaultdict(int)
d["a"] += 1   # ✅ 自动变成 1
```

普通 dict 在访问不存在key时：

```python
d["a"]  # KeyError
```

`defaultdict` 在访问不存在 key 时：

1. 调用你提供的函数
2. 生成一个默认值
3. 把这个值存入 dict
4. 再返回它

等价于内部做了：

```python
if key not in d:
    d[key] = default_factory()
return d[key]
```

构造函数：

```python
defaultdict(default_factory)
```

`default_factory` 必须是一个“可调用对象”。

比如：

| 传入 | 默认值 |
| ---- | ------ |
| int  | 0      |
| list | []     |
| set  | set[]  |
| dict | {}     |



### 二、常见三大使用场景

#### 2.1 计数器（统计频率）

普通写法：

```python
def count_frequency_common(word):
    counter = {}
    for c in word:
        if c not in counter:
            counter[c] = 0
        counter[c] += 1
    return counter


print(json.dumps(count_frequency_common("hello")))
```

defaultdict 写法：

```python
def statistical_frequency(word):
    counter = defaultdict(int)
    for c in word:
        counter[c] += 1
    return counter


print(statistical_frequency("hello"))  # 
```

输出：`{"h": 1, "e": 1, "l": 2, "o": 1}`

#### 2.2 分组

```python
users = [{"name": "A", "age": 18}, {"name": "B", "age": 25}, {"name": "C", "age": 18}]


def group_by_age(users):
    groups = defaultdict(list)
    for user in users:
        groups[user["age"]].append(user)
    return groups


print(json.dumps(group_by_age(users)))
```

结果：`{"18": [{"name": "A", "age": 18}, {"name": "C", "age": 18}], "25": [{"name": "B", "age": 25}]}`

#### 2.3 去重分组

```python
users2 = [{"name": "A", "age": 18}, {"name": "B", "age": 25}, {"name": "A", "age": 18}]


def group_by_age_unique(users):
    groups = defaultdict(set)
    for user in users:
        groups[user["age"]].add(user["name"])
    return groups


print(group_by_age_unique(users2))
```

结果：`defaultdict(<class 'set'>, {18: {'A'}, 25: {'B'}})`



### 三、dict 和 dict.setdefault 的区别

普通 dict 也可以这样：

```python
users = [{"name": "A", "age": 18}, {"name": "B", "age": 25}, {"name": "C", "age": 18}]


d = {}

for u in users:
    d.setdefault(u["age"], []).append(u["name"])
print(d)
```

结果：`{18: ['A', 'C'], 25: ['B']}`

区别：

| 方式        | 性能 | 可读性 | 推荐 |
| ----------- | ---- | ------ | ---- |
| setdefault  | 略慢 | 一般   | ⚠️    |
| defaultdict | 更优 | 清晰   | ✅    |

为什么 setdefault 慢，因为：

```python
d.setdefault(key, [])
```

每次都会创建一个新的 `[]`，即使 key 已存在。



### 四、工程细节

会偷偷创建 key。

```python
d = defaultdict(list)
print(d["not_exist"])
```

此时 dict 已经变成：

```python
{'not_exist': []}
```

有副作用。

如果只是想要判断是否存在：

```python
if key in d:
```

不直接访问。

转成普通 dict，很多时候不希望返回 defaultdict：

```python
return dict(d)
```

否则打印时是：`defaultdict(<class 'list'>, {...})`



### 五、自定义默认值

```python
def default_value():
    return "unknown"

default = defaultdict(default_value)
print(default["a"]) # unknown
```



### 六、嵌套defaultdict

```python
default = defaultdict(lambda: defaultdict(int))
default["2024"]["click"] += 1
print(json.dumps(default))
```

输出：`{"2024": {"click": 1}}`

这在日志系统/业务指标统计里非常常见。



### 七、底层原理

`defaultdict` 重写了：

```python
__missing__(self, key)
```

当key不存在时调用，普通 dict 没实现这个方法。



### 八、什么时候不该用

- 需要严格控制 key
- 不能允许隐式创建 key
- 高安全逻辑
  - 比如：权限系统、支付逻辑



### 九、关键工程认知

#### 9.1 统计 `default(int)`

```python
from collections import defaultdict


orders = [
    {"shop": "A", "user": "u1", "amount": 100},
    {"shop": "A", "user": "u2", "amount": 200},
    {"shop": "B", "user": "u1", "amount": 150},
]


def sum_by_shop(orders):
    """
    按店铺统计订单总金额

    Args:
        orders: 订单列表，每个订单是一个字典，包含"shop"(店铺名)和"amount"(金额)字段

    Returns:
        dict: 以店铺名为键，该店铺订单总金额为值的字典
    """
    # 使用defaultdict初始化店铺金额统计，默认值为0
    order_total = defaultdict(int)
    for order in orders:
        # 累加每个订单的金额到对应店铺的总计中
        order_total[order["shop"]] += order["amount"]
    return dict(order_total)


print(sum_by_shop(orders))
```

#### 9.2 分组 `default(list)`

```python
from collections import defaultdict


users = [{"name": "A", "age": 18}, {"name": "B", "age": 25}, {"name": "A", "age": 18}]


def group_by_age_unique(datas):
    group_age_unique = defaultdict(set)
    for data in datas:
        group_age_unique[data["age"]].add(data["name"])
    return dict(group_age_unique)


print(group_by_age_unique(users))
```

#### 9.3 去重分组 `default(set)`

```python
from collections import defaultdict


users = [{"name": "A", "age": 18}, {"name": "B", "age": 25}, {"name": "A", "age": 18}]


def group_by_age_unique(datas):
    group_age_unique = defaultdict(set)
    for data in datas:
        group_age_unique[data["age"]].add(data["name"])
    return dict(group_age_unique)


print(group_by_age_unique(users))
```

#### 9.4 嵌套统计 `default(lambda: defaultdict(int))`

```python
from collections import defaultdict
import json

orders = [
    ("Beijing", "Apple"),
    ("Beijing", "Apple"),
    ("Shanghai", "Banana"),
    ("Beijing", "Banana"),
]


# 嵌套
def nested_defaultdict(orders):
    """
    统计每个城市各种水果的订购数量

    参数:
        orders: 包含(城市, 水果)元组的列表

    返回:
        defaultdict: 嵌套的defaultdict，外层key为城市名，
                     内层key为水果名，value为该城市该水果的订购次数
    """
    stats = defaultdict(lambda: defaultdict(int))
    # 遍历所有订单，统计每个城市每种水果的数量
    for city, fruit in orders:
        stats[city][fruit] += 1
    return stats


print(json.dumps(nested_defaultdict(orders)))
```

#### 9.5 复杂 defaultdict 推荐写法

```python
def new_shop():
    return {
        "total_amount": 0,
        "users": set()
    }

stats = defaultdict(new_shop)
```

>比 lambda **更容易被 IDE 识别**。



### 十、无限嵌套

> **无限嵌套 defaultdict（自动生成 JSON 树）**。
>
> 很多日志系统 / BI 系统 / 数据统计都会用。

假设有日志数据：

```python
logs = [
    {"country": "CN", "city": "Beijing", "user": "u1"},
    {"country": "CN", "city": "Beijing", "user": "u2"},
    {"country": "CN", "city": "Shanghai", "user": "u3"},
    {"country": "US", "city": "NY", "user": "u4"},
]
```

希望统计成：

```python
{
    "CN": {
        "Beijing": ["u1", "u2"],
        "Shanghai": ["u3"]
    },
    "US": {
        "NY": ["u4"]
    }
}
```

```python
def tree():
    return defaultdict(tree)


def tree_aggregate(logs):
    data = tree()

    for log in logs:
        country = log["country"]
        city = log["city"]
        user = log["user"]

        data[country][city].setdefault("users", []).append(user)
    return dict(data)


print(tree_aggregate(logs))
```

