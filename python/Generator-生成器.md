- 流式处理
- 超大数据处理
- 协程处理
- 很多框架的底层原理



## 问题

---

假设有一个 1000 万行的文件。

普通写法：

```python
def read_file():
    """
    读取文件，一次性把所有数据加载到内存
    """
    with open("") as f:
        return f.readlines()
```



## 生成器核心思想

---

Generator = **按需生产数据**，而不是：一次性生成所有数据。



## 最简单的生成器

---

### 一、普通函数

```python
def numbers():
    return [1, 2, 3]
```

### 二、生成器

```python
def numbers_generator():
    yield 1
    yield 2
    yield 3
    
for n in numbera_generator():
    print(n)
```

输出：

```python
1
2
3
```



## 生成器本质

---

当函数包含 `yield`:

```python
def numbers_generator():
    print("start")
    yield 1
    print("middle")
    yield 2
    print("end")
    yield 3
```

调用：

```python
g = numbers_generator()
print(g)
```

不会执行函数。而是返回：

```python
<generator object numbers_generator at 0x1075ef840>
```

真正执行发生在：`next(g)`

输出：`start`

再执行：`next(g)`

输出：`end`

再执行：`StopIteration`



## 读取大文件

---

读取大文件：

```python
def read_large_file(file_path):
    with open(file_path) as f:
        for line in f:
            yield line
```

使用：

```python
for line in read_large_file("./data/large_file.txt"):
    print(line)
```

特点：

```python
内存占用 ≈ 1 行
```



## 生成器表达式

---

### 一、列表

```python
nums = [x*x for x in range(10)]
print(type(nums))
```

### 二、生成器

```python
generator_nums = (x for x in range(10))
print(type(generator_nums))
```

### 三、区别

```python
<class 'list'>
<class 'generator'>
```



## 内存对比

---

### 一、列表

```python
nums = [x*x for x in range(10000000)]
```

占用：`～80MB+`

### 二、生成器

```python
nums = (x for x in range(10000000))
```

占用：几十字节

因为数据**没有真正创建**。



## Pipeline

---

生成器可以像 **流水线** 一样连接。

```python
def read_file(file):
    """
    读取文件并逐行生成内容
    
    Args:
        file: 文件路径字符串，指定要读取的文件
        
    Yields:
        str: 文件中的每一行内容
    """
    with open(file) as f:
        for line in f:
            yield line


def strip_lines(lines):
    """
    去除输入行序列中每一行的首尾空白字符
    
    Args:
        lines: 可迭代对象，包含待处理的字符串行
        
    Yields:
        str: 去除首尾空白后的每一行内容
    """
    for line in lines:
        yield line.strip()


def filter_empty(lines):
    """
    过滤输入行序列中的空行，只生成非空行
    
    Args:
        lines: 可迭代对象，包含待过滤的字符串行
        
    Yields:
        str: 非空的每一行内容
    """
    for line in lines:
        if line:
            yield line
```

使用：

```python
lines = read_file("./data/large_file.txt")
lines = strip_lines(lines)
lines = filter_empty(lines)

for line in lines:
    print(line)
```

特点：

- 逐行处理
- 完全流式
- 几乎零内存



## python 内置生成器

---

```python
```



























































































