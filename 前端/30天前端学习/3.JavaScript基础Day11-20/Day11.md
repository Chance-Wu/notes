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

#### 2.3 逻辑运算符

```javascript
let age = 20;
console.log(age > 18 && age < 30); // true
let isStudent = true;
console.log(!isStudent);           // 取反
```

#### 2.4 三元运算符

```javascript
let score = 80;
let qualified = score >= 60 ? "合格" : "不合格";
```



### 三、流程控制

---

#### 3.1 if条件判断

```javascript
let hour = new Date().getHours();
if (hour < 12) {
    console.log("上午好");
} else if (hour < 18) {
    console.log("下午好");
} else {
    console.log("晚上好");
}
```

#### 3.2 switch语句

```javascript
let day = new Date().getDay();
switch (day) {
    case 6, 7:
        console.log("周末");
        break;
    default:
        console.log("工作日");
}
```

#### 3.3 for循环

```javascript
for (let i = 0; i < 5; i++) {
    console.log(i);
}
```

#### 3.4 while循环

```javascript
let n = 3;
while (n > 0) {
    console.log(n--);
}
```



### 四、实战：计算器

---

```html
<!DOCTYPE html>
<html>
<head>
    <title>简易计算器</title>
    <style>
        .calculator {
            width: 300px;
            margin: 20px auto;
            padding: 20px;
            border: 1px solid #ccc;
            border-radius: 8px;
            background: #f5f5f5;
        }

        #display {
            width: 100%;
            height: 40px;
            margin-bottom: 10px;
            text-align: right;
            padding: 5px;
            font-size: 1.5rem;
        }

        .buttons {
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            gap: 5px;
        }

        button {
            padding: 15px;
            font-size: 1.2rem;
            border: 1px solid #ddd;
            background: white;
            cursor: pointer;
        }

        button:hover {
            background: #eee;
        }

        .operator {
            background: #f0ad4e;
            color: white;
        }

        .equals {
            background: #5cb85c;
            color: white;
        }
    </style>
</head>
<body>
    <div class="calculator">
        <input type="text" id="display" readonly>
        <div class="buttons">
            <button onclick="clearDisplay()">C</button>
            <button onclick="appendToDisplay('/')">/</button>
            <button onclick="appendToDisplay('*')">×</button>
            <button onclick="deleteLast()">⌫</button>
            <button onclick="appendToDisplay('7')">7</button>
            <button onclick="appendToDisplay('8')">8</button>
            <button onclick="appendToDisplay('9')">9</button>
            <button class="operator" onclick="appendToDisplay('-')">-</button>
            <button onclick="appendToDisplay('4')">4</button>
            <button onclick="appendToDisplay('5')">5</button>
            <button onclick="appendToDisplay('6')">6</button>
            <button class="operator" onclick="appendToDisplay('+')">+</button>
            <button onclick="appendToDisplay('1')">1</button>
            <button onclick="appendToDisplay('2')">2</button>
            <button onclick="appendToDisplay('3')">3</button>
            <button class="equals" onclick="calculate()" style="grid-row: span 2">=</button>
            <button onclick="appendToDisplay('0')" style="grid-column: span 2">0</button>
            <button onclick="appendToDisplay('.')">.</button>
        </div>
    </div>

    <script>
        let display = document.getElementById('display');

        // 添加输入
        function appendToDisplay(value) {
            if (display.value === '0' && value !== '.') {
                display.value = value;
            } else {
                display.value += value;
            }
        }

        // 清空显示
        function clearDisplay() {
            display.value = '0';
        }

        // 删除末位
        function deleteLast() {
            display.value = display.value.slice(0, -1) || '0';
        }

        // 执行计算
        function calculate() {
            try {
                let result = eval(display.value);
                display.value = Number.isFinite(result) ? result : '错误';
            } catch {
                display.value = '错误';
            }
        }

        // 键盘支持
        document.addEventListener('keydown', (e) => {
            if (e.key >= '0' && e.key <= '9' || e.key === '.') {
                appendToDisplay(e.key);
            } else if ('+-*/'.includes(e.key)) {
                appendToDisplay(e.key);
            } else if (e.key === 'Enter') {
                calculate();
            } else if (e.key === 'Backspace') {
                deleteLast();
            } else if (e.key === 'Escape') {
                clearDisplay();
            }
        });
    </script>
</body>
</html>
```

#### 3.1 核心功能实现

1. **输入处理**：`appendToDisplay`管理输入逻辑
2. **计算引擎**：使用`eval`快速实现（生产环境建议使用表达式解析器）
3. **错误处理**：通过try-catch捕获计算错误
4. **键盘支持**：监听keydown事件增强用户体验
