### 一、变量与数据类型

---

#### 1.1 变量声明

```javascript
let message = "Hello"; // 块级作用域
var count = 0;         // 函数作用域（逐渐淘汰）
const PI = 3.14;       // 常量
```

#### 1.2 基本数据类型

```javascript
let str = "文本";       // String
let num = 100;         // Number
let flag = true;       // Boolean
let empty = null;      // Null
let undef;             // Undefined
```

#### 1.3 类型检测

```javascript
console.log(typeof str); // "string"
console.log(num instanceof Number); // false（基本类型非对象）
```

#### 1.4 类型转换

```javascript
let price = "199";
console.log(Number(price)); // 199
console.log(+price);        // 199（快捷转换）
```



### 二、运算法与表达式

---

#### 2.1 算术运算符

```javascript
console.log(10 % 3); // 1（取模）
console.log(2 ** 3); // 8（幂运算）
```

#### 2.2 比较运算符

```javascript
console.log(5 == "5");  // true（值相等）
console.log(5 === "5"); // false（严格相等）
```

#### 2.3 



























































































