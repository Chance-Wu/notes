### 一、数组核心方法

---

```javascript
// 创建数组
let scores = [85, 92, 78, 95];
let students = new Array(3).fill(null);

// 基础操作
scores.push(88);       // 末尾添加
const last = scores.pop();     // 移除末尾
scores.unshift(90);    // 开头添加
scores.shift();        // 移除开头

// 高阶方法
const highScores = scores.filter(s => s >= 90);
const doubleScores = scores.map(s => s * 2);
const total = scores.reduce((sum, s) => sum + s, 0);
scores.sort((a,b) => b - a);  // 降序排序

// 解构赋值
const [first, second, ...others] = scores;
```



### 二、对象操作技巧

---



























































































































