日常开发中遇到最多的就是NPE问题，NPE问题即是小问题又是破坏健壮性的大问题，我们应该更好的防范下。

1. 入参是对象的时候应该对入参进行判空操作；
2. 调用的其他人写的方法返回值应该进行判空操作；
3. 调用数据库或者远程调用的返回值应该进行判空操作；
4. 保证调用者为已知对象。`str1.equals(str2)` 方法，str1应该是已知对象，str2可以为空。`String.valueOf()` 和 `Integer/Long/Double.toString()` 返回值相同使用前者；
5. 自己写的方法尽量不要返回空值。