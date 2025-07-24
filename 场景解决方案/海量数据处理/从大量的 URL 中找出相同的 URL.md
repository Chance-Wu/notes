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

#### 7.1 Utils.java

```java
public class Utils {

    /**
     * 计算给定URL的哈希值并将其映射到指定数量的分区中。
     * 此方法使用SHA-256算法生成URL的哈希，然后通过取模运算确定分区索引。
     *
     * @param url           需要计算哈希值的字符串，必须不为空。
     * @param numPartitions 分区的数量，必须是正整数。
     * @return 返回一个非负整数，表示URL对应的分区索引。
     * @throws IllegalArgumentException 如果url为null或numPartitions小于等于0。
     * @throws NoSuchAlgorithmException 如果SHA-256算法不可用。
     */
    public static int hashUrl(String url, int numPartitions) throws NoSuchAlgorithmException {
        if (url == null || numPartitions <= 0) {
            throw new IllegalArgumentException("url must not be null and numPartitions must be positive");
        }

        try {
            MessageDigest md = MessageDigest.getInstance("SHA-256"); // 更安全的哈希算法
            byte[] digest = md.digest(url.getBytes(StandardCharsets.UTF_8)); // 明确指定字符集
            long hash = ((long) (digest[0] & 0xFF) << 24)
                    | ((long) (digest[1] & 0xFF) << 16)
                    | ((long) (digest[2] & 0xFF) << 8)
                    | ((long) (digest[3] & 0xFF));
            return (int) ((hash & Long.MAX_VALUE) % numPartitions); // 避免负数问题
        } catch (NoSuchAlgorithmException e) {
            // 记录异常日志
            throw new RuntimeException("Failed to compute hash for URL: " + url, e);
        }
    }

    /**
     * 如果指定的目录不存在，则创建它。如果目录已存在，则不会执行任何操作。
     * 尝试以特定权限创建目录，若因文件系统不支持权限设置而失败，则回退至默认行为。
     *
     * @param dirPath 要检查和创建的目录路径。
     * @throws IOException 如果创建目录时发生I/O错误。
     */
    public static void createDirectoryIfNotExists(String dirPath) throws IOException {
        Path path = Paths.get(dirPath);
        if (!Files.exists(path)) {
            try {
                // 显式设置目录权限为 rwx------
                Files.createDirectories(path, PosixFilePermissions.asFileAttribute(PosixFilePermissions.fromString("rwx------")));
            } catch (FileAlreadyExistsException ignored) {
                // 并发情况下目录可能已被其他线程创建，忽略此异常
            } catch (UnsupportedOperationException e) {
                // 当前文件系统不支持设置权限，回退到默认行为
                Files.createDirectories(path);
            }
        }
    }
}
```

#### 7.2 Splitter.java

```java
public class Splitter {

    /**
     * 输出目录名称，所有拆分后的文件将保存在此目录下
     */
    public static final String OUTPUT_DIR = "split";

    /**
     * 将指定文件按行拆分为多个分区文件。
     *
     * <p>该方法首先创建输出目录（如果不存在），然后根据文件名创建相应数量的输出文件。
     * 接着读取输入文件的每一行，对其进行哈希计算以确定应写入哪个分区文件。</p>
     *
     * @param filename      输入文件的路径
     * @param numPartitions 分区的数量，决定拆分后的文件个数
     * @throws IOException 如果在文件读取或写入过程中发生 I/O 错误
     */
    public static void splitFile(String filename, int numPartitions) throws Exception {
        Utils.createDirectoryIfNotExists(OUTPUT_DIR);

        BufferedWriter[] writers = new BufferedWriter[numPartitions];
        for (int i = 0; i < numPartitions; i++) {
            Path outputPath = Paths.get(OUTPUT_DIR, filename + "_" + i); // 跨平台路径拼接
            writers[i] = new BufferedWriter(new FileWriter(outputPath.toFile()));
        }

        /*
          使用 try-with-resources 确保 BufferedReader 正确关闭。
          逐行读取输入文件，并对每行进行 trim 操作。
          使用 hashUrl 方法计算该行应写入的分区索引。
          写入时确保每条记录都立即刷新至磁盘，防止数据丢失。
         */
        try (BufferedReader reader = new BufferedReader(new FileReader(filename))) {
            String line;
            while ((line = reader.readLine()) != null) {
                String trimmedLine = line.trim();
                int idx = Utils.hashUrl(trimmedLine, numPartitions);
                if (idx < 0 || idx >= numPartitions) {
                    throw new IllegalStateException("Invalid partition index: " + idx);
                }
                writers[idx].write(line);
                writers[idx].newLine();
                writers[idx].flush(); // 确保每次写入都刷新
            }
        }

        for (BufferedWriter writer : writers) {
            writer.flush(); // 再次确认刷新
            writer.close(); // close 可以省略，因为 flush 已经在 try-with-resources 中保证
        }
    }
}
```

#### 7.3 Comparator.java

```java
public class Comparator {

    /**
     * 查找两个文件中共同的URL，并将结果追加写入到common_urls.txt文件中。
     * 该方法会根据给定的分区ID加载对应的两个文件，
     * 使用HashSet来存储第一个文件的所有行，然后遍历第二个文件查找共有行。
     *
     * @param partitionId 分区ID，用于确定要比较的文件名。
     * @throws IOException 如果在读取或写入文件时发生I/O错误。
     */
    public static void findCommon(int partitionId) throws IOException {
        Set<String> setA = new HashSet<>();
        String fileA = Splitter.OUTPUT_DIR + "/a_" + partitionId;
        String fileB = Splitter.OUTPUT_DIR + "/b_" + partitionId;

        // 将fileA中的所有非空行加入HashSet
        BufferedReader readerA = new BufferedReader(new FileReader(fileA));
        String line;
        while ((line = readerA.readLine()) != null) {
            setA.add(line.trim());
        }
        readerA.close();

        // 读取fileB并查找与setA中的公共行，写入common_urls.txt
        BufferedReader readerB = new BufferedReader(new FileReader(fileB));
        BufferedWriter writer = new BufferedWriter(new FileWriter("common_urls.txt", true));

        while ((line = readerB.readLine()) != null) {
            if (setA.contains(line.trim())) {
                writer.write(line);
                writer.newLine();
            }
        }

        readerB.close();
        writer.close();
    }
}
```

#### 7.4 FindCommonUrls.java

```java
public class FindCommonUrls {

    private static final int NUM_PARTITIONS = 1000;

    public static void main(String[] args) throws Exception {
        // Step 1: 分割文件
        System.out.println("开始分割文件...");
        Splitter.splitFile("a", NUM_PARTITIONS);
        Splitter.splitFile("b", NUM_PARTITIONS);
        System.out.println("文件分割完成");

        // Step 2: 比较每个子文件对
        System.out.println("开始查找共同 URL...");
        for (int i = 0; i < NUM_PARTITIONS; i++) {
            Comparator.findCommon(i);
            System.out.print(".");
        }

        System.out.println("\n查找完成，结果已写入 common_urls.txt");
    }

}
```

