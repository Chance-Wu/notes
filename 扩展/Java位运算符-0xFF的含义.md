#### 1. 区分A~F的意思

---

|      | 十进制 | 二进制 |
| ---- | ------ | ------ |
| A    | 10     | 1010   |
| B    | 11     | 1011   |
| C    | 12     | 1100   |
| D    | 13     | 1101   |
| E    | 14     | 1110   |
| F    | 15     | 1111   |



#### 2. 区分 & | ^ 运算规则

---

>==&== 按位与
>
>两个操作数中 位 都为1，结果就等于1。`1010` & `1110` = `1010`

>==|== 按位或
>
>两个操作数中 位 只要有一个为1，结果就等于1。`1010` | `1111` = `1111`

>==^== 按位异或
>
>两个操作数中 位 如果相同的数值，结果为0。不同则为1。`1010` ^ `1110` = `0100`

>==~== 按位取反
>
>参加运算的一个数据，按二进制位进行“取反”运算。



#### 3. 0xff

---

看到0x ——>16进制数

0xFF：先转换为十进制 15 * 16^^0^ + 15 * 16^^1^ = 15 + 240 = 255

再将十进制转换为二进制：`1111 1111`

