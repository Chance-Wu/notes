### 一、题目描述

---

给定a、b 两个文件，各存放 50 亿个 URL，每个 URL 各占 64B，内存限制是 4G。请找出两个文件共同的 URL。



### 二、问题分析

---

- 每个 URL 占 64 字节。
- 每个文件大小：50 亿 × 64B = 320GB。
- 两个文件加起来是 640GB。
- 内存限制为 4G，无法将所有数据一次性加载到内存。



### 三、解决方案：哈希分割（Hash Partitioning）

---

- 步骤一：将文件 a 和 b 分别按哈希值分割成多个小文件

  - 对文件 a 的每个 URL 使用哈希函数（如 MD5、CRC32 等），将其映射到 N 个子文件中（例如 `a0, a1, ..., aN-1`）。
  - 同样地，将文件 b 的每个 URL 哈希后映射到对应的 `b0, b1, ..., bN-1`。
  - 保证相同的 URL 会落入相同的子文件对中（如 a0 和 b0）。

  > **注意**：N 的选择应使得每个子文件对的大小可以加载进内存中进行比较。例如，如果 N=1000，则每个子文件平均约为 320MB（320GB ÷ 1000），适合内存处理。

- 步骤二：逐个处理每对子文件（如 a0 和 b0）

  - 将 a0 加载到一个哈希集合（`HashSet`）中。
  - 遍历 b0 的每个 URL，检查是否存在于 a0 的哈希集合中。
  - 如果存在，说明是公共 URL，输出或记录下来。

时间复杂度分析：

- 每个文件只需读取一次。
- 哈希分割是 O(n)。
- 每个子文件在内存中比较是 O(m)，其中 m 是子文件大小。
- 总体复杂度为线性时间：O(n)



### 四、优化建议

---

#### 4.1 使用更高效的哈希算法

- CRC32 足够用于分割，速度快。
- 如果担心哈希冲突，可以在最后对候选 URL 做精确比较。

#### 4.2 多线程或分布式处理（可选）

- 可以并行处理不同的子文件对（如 a0 和 b0、a1 和 b1 等）。
- 若使用分布式系统（如 Hadoop、Spark），可进一步扩展处理能力。



### 五、伪代码示例

---

```python
import hashlib

def hash_url(url, num_partitions):
    return int(hashlib.md5(url.encode()).hexdigest(), 16) % num_partitions

def split_file(filename, num_partitions):
    writers = [open(f"{filename}_{i}", "w") for i in range(num_partitions)]
    with open(filename, "r") as f:
        for line in f:
            idx = hash_url(line.strip(), num_partitions)
            writers[idx].write(line)
    for w in writers:
        w.close()

def find_common_urls(partition_id):
    set_a = set()
    with open(f"a_{partition_id}") as f:
        for line in f:
            set_a.add(line.strip())
    with open(f"b_{partition_id}") as f:
        for line in f:
            if line.strip() in set_a:
                print(line.strip())  # 或写入结果文件

# 主流程
num_partitions = 1000
split_file("a", num_partitions)
split_file("b", num_partitions)

for i in range(num_partitions):
    find_common_urls(i)
```



### 六、总结

---

| 方法                        | 是否可行 | 优点               | 缺点                   |
| --------------------------- | -------- | ------------------ | ---------------------- |
| 哈希分割 + 哈希集合（推荐） | ✅        | 内存可控、效率高   | 需要磁盘 I/O           |
| 排序归并法                  | ✅        | 无需哈希，避免冲突 | 耗时较长，排序慢       |
| Bloom Filter                | ⚠️        | 节省内存           | 有误判风险，需二次验证 |
| MapReduce（分布式）         | ✅        | 可扩展性强         | 部署复杂，资源要求高   |



### 七、Java 实现

---















































































































