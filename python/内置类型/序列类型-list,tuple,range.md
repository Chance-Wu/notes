## range 对象

---

`range` 类型表示**不可变的数字序列**，通常用于在 for 循环中循环指定的次数，

- *class* `range`(*stop*)
- *class* `range`(*start*, *stop*[, *step*])

```python
>>> list(range(10))
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
>>> list(range(1, 11))
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
>>> list(range(0, 30, 5))
[0, 5, 10, 15, 20, 25]
>>> list(range(0, 10, 3))
[0, 3, 6, 9]
>>> list(range(0, -10, -1))
[0, -1, -2, -3, -4, -5, -6, -7, -8, -9]
>>> list(range(0))
[]
>>> list(range(1, 0))
[]
```

`range` 类型相比常规 `list` 或 `tuple` 的优势在于一个 `range` 对象总是占用固定数量的（较小）内存，不论其所表示的范围有多大（因为它只保存了 `start`, `stop` 和 `step` 值，并会根据需要计算具体单项或子范围的值）。

range 对象实现了 `collections.abc.Sequence` ABC，提供如包含检测、元素索引查找、切片等特性，并支持负索引。

### 检测

```python
r = range(0, 20, 2)

if 11 in r:
    print(False)
if 10 in r:
    print(True)
```

输出：`True`

### 元素索引查找

```python
print(r.index(10))
```

输出：`5`

### 切片

```python
print(r[:5])
```

输出：`range(0, 10, 2)`

### 负索引

```python
print(r[-1])
```

输出：`18`

### 比较

使用 `==` 和 `!=` 检测 range 对象是否相等是将其作为序列来比较。 也就是说，如果两个 range 对象表示相同的值序列就认为它们是相等的。 （请注意比较结果相等的两个 range 对象可能会具有不同的 `start`, `stop` 和 `step` 属性，例如 `range(0) == range(2, 1, 3)` 而 `range(0, 3, 2) == range(0, 4, 2)`。）

```python
r1 = range(0, 3, 2)
r2 = range(0, 4, 2)

if r1 == r2:
    print(True)
```

输出：`True`





















































































































