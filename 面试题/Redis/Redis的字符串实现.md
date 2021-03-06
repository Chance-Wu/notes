#### 1. Redis字符串的实现

---

> Redis自己实现了一套字符串。构建了一个==简单动态字符串==（Simple Dynamic String）`SDS`。

>**SDS代码结构**
>
>```c
>struct sdshdr{
>  //  记录已使用长度
>  int len;
>  // 记录空闲未使用的长度
>  int free;
>  // 字符数组
>  char[] buf;
>};
>```
>
><img src="https://tva1.sinaimg.cn/large/008i3skNgy1gqfq74jcdsj30fo06ht8o.jpg" style="zoom:40%">
>
>==最后一个字符为空字符。然而这个空字符不会被计算在len里头==。
>
>如果要追加字符，操作步骤：
>
>1. 计算出大小是否足够
>2. 开辟空间至满足所需大小
>3. 开辟与已使用大小len相同长度的空闲free空间（len < 1M）；开辟1M长度的free空间（len >= 1M）。



#### 2. Redis字符串的性能优势

---

>**快速获取字符串长度**
>
>由于SDS里存了已使用字符长度len，获取字符串长度时直接返回len即可，时间复杂度为O(1)。

>**避免缓冲区溢出**
>
>每次追加字符串时都会检查空间是否够用，所以不会存在缓冲区溢出问题。
>
>1. 计算出大小是否足够；
>2. 开辟空间至满足所需大小。

>**降低空间分配次数提升内存使用效率**
>
>字符串的追加操作会涉及到内存分配问题，所以采取如下优化：
>
>- ==空间预分配==：对于<u>*追加操作*</u>来说，Redis不仅会开辟空间至够用而且还会预分配未使用的空间(free)来用于下一次操作。至于==未使用的空间(free)的大小则由修改后的字符串长度决定==。有了预分配策略之后会==减少内存分配次数==，因为分配之前会检查已有的free空间是否够，如果够则不开辟了。
>- ==惰性空间回收==：惰性空间回收适用于字符串<u>*缩减操作*</u>。比如有个字符串s1="hello world"，对s1进行sdstrim(s1," world")操作，执行完该操作之后Redis不会立即回收减少的部分，而是会分配给下一个需要内存的程序。当然，Redis也提供了回收内存的api，可以自己手动调用来回收缩减部分的内存。