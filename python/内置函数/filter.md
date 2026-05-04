> `filter(*function*, *iterable*)`

用 `iterable` 中函数 `function` 返回真的那些元素，构建一个新的迭代器。`iterable` 可以是一个序列，一个支持迭代的容器，或一个迭代器。如果 `function` 是 `None` ，则会假设它是一个身份函数，即 `iterable` 中所有返回假的元素会被移除。请注意， `filter(function, iterable)` 相当于一个生成器表达式，当 function 不是 `None` 的时候为 `(item for item in iterable if function(item))`；function 是 `None` 的时候为 `(item for item in iterable if item)`。



## 基本用法(过滤偶数)

---

```python
numbers = [1, 2, 3, 4, 5, 6]

evens = filter(lambda x: x % 2 == 0, numbers)

print(evens)
print(list(evens))
```

输出：

```python
<filter object at 0x10a45a170>
[2, 4, 6]
```



## 使用自定义函数

---

```python
def is_positiven(n):
    return n > 0


nums = [-2, -1, 0, 1, 2]
positives = filter(is_positiven, nums)
print(list(positives))
```

输出：

```python
[1, 2]
```



## function=None 过滤假值

---

```python
data = [0, 1, None, True, False, 2, "", " ", "hello", [], [1, 2]]
truthy = filter(None, data)
print(list(truthy))
```

>相当于：`truthy = filter(lambda x: x, data)`



## 过滤字典列表

---

```python
users = [
    {"name": "Alice", "age": 30},
    {"name": "Bob", "age": 17},
    {"name": "Charlie", "age": 25},
]
adults = filter(lambda u: u["age"] >= 18, users)
print(list(adults))
```

输出：

```python
[{'name': 'Alice', 'age': 30}, {'name': 'Charlie', 'age': 25}]
```



## 注意事项

---

**返回的是迭代器（惰性求值）**，如果需要多次使用，立即转为 list。