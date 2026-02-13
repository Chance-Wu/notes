### 一、什么是面向对象编程

---

>OOP 是一种以对象为中心的变成思想。对象是“**属性**+**方法**”的集合。

JavaScript 的对象：

```javascript
const book = {
    title: 'js',
    author: 'chance',
    read() {
        console.log(`正在阅读 《${this.title}》`);
    }
}

book.read();
```



### 二、构造函数与原型（ES5 写法）

---

在 ES6 之前，JS 使用 **构造函数 + prototype** 模拟类：

```js
```























































































