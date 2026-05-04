> `map`(*function*, *iterable*, *...*)

返回一个将 `function` 应用于 `iterable` 中每一项并输出其结果的迭代器。 如果传入了额外的 `iterable` 参数，`function` 必须接受相同个数的实参并被应用于从所有可迭代对象中并行获取的项。 当有多个可迭代对象时，最短的可迭代对象耗尽则整个迭代就将结束。 对于函数的输入已经是参数元组的情况，请参阅 `itertools.starmap()`。



## 基本用法(单个 iterable)

将列表中每个数字平方：

```python
n = [1, 2, 3]
squared = map(lambda x: x * x, n)
print(squared)
print(list(squared))
```

输出：

```python
<map object at 0x103cafd60>
[1, 4, 9]
```

>`map()` 返回的是**迭代器**，只能遍历一次！如果需要多次使用，转为 `list`。



## 使用自定义函数

```python
# 自定义函数
def to_upper(s):
    return s.upper()


words = ["hello", "world"]
upper_words = map(to_upper, words)
print(list(upper_words))
```

输出：`['HELLO', 'WORLD']`



## 多个iterable 并行处理

---

```python
nums = [1, 2, 3]
nums2 = [4, 5, 6, 7]
sums = map(lambda x, y: x + y, nums, nums2)
print(list(sums))
```

输出：`[5, 7, 9]`

>多个 iterable 时，map 会在最短的 iterable 耗尽时停止。（类似zip）



## 与内置函数结合

---

```python
# 转换字符串为整数
str_nums = ["1", "2", "3"]
int_nums = map(int, str_nums)
print(list(int_nums))

# 获取每个字符串长度
names = ["Alice", "Bob", "Charlie"]
lengths = map(len, names)
print(list(lengths))
```

输出：

```python
[1, 2, 3]
[5, 3, 7]
```