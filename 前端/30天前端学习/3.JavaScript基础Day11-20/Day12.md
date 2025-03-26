### 一、函数基础

---

#### 1.1 函数声明

```javascript
function greet(name) {
    return `Hello, ${name}!`;
}
console.log(greet("Chance"));
```

特点：

- **存在函数提升（可在定义前调用）**
- 拥有函数名标识符
- 适合定义主要功能函数

#### 1.2 函数表达式

```javascript
const area = function (width, height) {
    return width * height;
};
console.log(area(3, 4));
```

- 匿名函数（可具名，但一般匿名）

  ```javascript
  const sum = function(a, b) { return a + b; };
  // 或具名函数表达式（函数名仅在函数体内有效）
  const sum = function namedSum(a, b) { return a + b; };

- 无提升特性

- 适合作为回调函数

#### 1.3 箭头函数（ES6+）

```javascript
const square = x => x ** 2;
const add = (a, b) => a + b;
const log = () => console.log("execute");
```

- 没有自己的`this`绑定
- 不能作为构造函数
- 适合简短操作和回调



### 二、参数与返回值

---

#### 2.1 参数处理

```javascript
function sum(a, b = 0) { // 默认参数
    console.log(arguments); // 类数组参数集合
    return a + b;
}
console.log(sum(5));

// 剩余参数
function showNames(...names) {
    names.forEach(name => console.log(name));
}
showNames("Chance", "Wcy");
```

#### 2.2 返回值

```javascript
function checkAge(age) {
    if (age < 0) return;
    return age >= 18 ? "成人" : "未成年";
}
console.log(checkAge(25));
```



### 三、作用域与闭包

---

#### 3.1 作用域链

```javascript
```



















































































































