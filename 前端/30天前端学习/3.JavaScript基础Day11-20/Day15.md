### 一、ES6特性

---

#### 1.1 let

`let` 是一种新的变量声明方式，用于声明块级作用域的变量。与 `var` 不同，`let` 提供了更好的作用域控制，避免了一些 `var` 的常见问题。

1. 块级作用域

   let声明的变量只在它所在的代码块{}中有效，而var声明的变量是函数级作用域。

   ```javascript
   {
     let x = 1;
     var y = 2;
   }
   console.log(x); // ReferenceError: x is not defined
   console.log(y); // 2
   ```

2. 暂时性死区（Temporal Dead Zone, TDZ）

   在 `let` 声明的变量之前访问该变量会抛出错误，这被称为“暂时性死区”。

   ```javascript
   console.log(x); // ReferenceError: x is not defined
   let x = 1;
   ```

3. 不允许重复声明

   let不允许在同一个作用域内重复声明同一个变量。

   ```javascript
   let x = 1;
   let x = 2; // SyntaxError: Identifier 'x' has already been declared
   ```

4. 与var的区别

   - **作用域**：`let` 是块级作用域，`var` 是函数级作用域。
   - **重复声明**：`let` 不允许重复声明，`var` 允许。
   - **暂时性死区**：`let` 有暂时性死区，`var` 没有。
   - **变量提升**：`let` 不会变量提升，`var` 会变量提升。

5. 使用场景

   `let` 通常用于需要更严格作用域控制的场景，比如**循环、条件语句**等。

####  1.2 const

`const` 是一种用于声明常量的关键字，它与 `let` 类似，但有一些重要的区别。`const` 声明的变量是块级作用域的，并且**一旦赋值后就不能更改**。

1. 块级作用域

   ```javascript
   {
     const x = 1;
   }
   console.log(x); // ReferenceError: x is not defined
   ```

2. 不能重复声明

   ```javascript
   const x = 1;
   const x = 2; // SyntaxError: Identifier 'x' has already been declared
   ```

3. 不能更改值

   ```javascript
   const x = 1;
   x = 2; // TypeError: Assignment to constant variable.
   ```

4. 必须初始化

   ```java
   const x; // SyntaxError: Missing initializer in const declaration
   ```

5. 暂时性死区

   ````javascript
   console.log(x); // ReferenceError: x is not defined
   const x = 1;
   ````

6. 与let的区别

   - **可变性**：`let` 声明的变量可以重新赋值，而 `const` 声明的变量不能重新赋值。
   - **初始化**：`let` 可以声明后赋值，而 `const` 必须在声明时初始化。

7. 使用场景

   `const` 适用于声明不需要重新赋值的变量，比如函数、对象、数组等。

   ```javascript
   const person = {
     name: 'John',
     age: 30
   };
   
   // 修改对象的属性是可以的
   person.age = 31; // 允许
   
   // 重新赋值是不允许的
   person = { name: 'Jane', age: 25 }; // TypeError: Assignment to constant variable.
   ```

#### 1.3 箭头函数

箭头函数提供了更简洁的语法，并且**在某些情况下可以自动绑定 `this` 的上下文**。

```javascript
// 传统函数表达式
const multiply = function(x, y) {
  return x * y;
};

// 箭头函数
const multiply = (x, y) => {
  return x * y;
};

// 更简洁的箭头函数（单行返回）
const multiply = (x, y) => x * y;
```

- 简洁语法：减少了 `function` 关键字和 `return` 关键字的使用。
- 没有自己的 `this`：箭头函数不会创建自己的 `this` 上下文，而是继承自父作用域的 `this`。
- 没有 `arguments` 对象：箭头函数内部不能使用 `arguments` 对象。
- 不能用作构造函数：箭头函数不能用 `new` 关键字调用。
- 没有 `super`：箭头函数内部不能使用 `super` 关键字。

适用场景：

- 简单的回调函数

  ```javascript
  // 传统写法
  setTimeout(function() {
    console.log('Hello after 1 second');
  }, 1000);
  
  // 箭头函数写法
  setTimeout(() => {
    console.log('Hello after 1 second');
  }, 1000);
  ```

- 对象方法

  ```javascript
  const obj = {
    name: 'John',
    greet: () => {
      console.log(`Hello, I'm ${this.name}`); // this 指向 window 或 undefined（严格模式）
    }
  };
  
  obj.greet(); // Hello, I'm undefined
  ```

  箭头函数没有自己的 `this`，它会继承父作用域的 `this`。在对象方法中，箭头函数的 `this` 不会指向对象。

- 类的方法

  ```javascript
  class Counter {
    constructor() {
      this.count = 0;
      setInterval(() => {
        console.log(this.count); // this 指向 Counter 实例
        this.count++;
      }, 1000);
    }
  }
  ```

- 返回对象

  如果箭头函数返回一个对象，需要用括号包裹对象字面量：

  ```javascript
  const getUser = () => ({ name: 'John', age: 30 });
  console.log(getUser()); // { name: 'John', age: 30 }
  ```

>注意事项：
>
>1. **`this` 的上下文**：箭头函数不会创建自己的 `this`，它会继承父作用域的 `this`。因此，在对象方法中使用箭头函数时，`this` 不会指向对象。
>
>2. **`arguments` 对象**：箭头函数内部没有 `arguments` 对象，可以使用 `rest` 参数代替：
>
>   ```javascript
>   const sum = (...args) => args.reduce((a, b) => a + b, 0);
>   console.log(sum(1, 2, 3)); // 6
>   ```
>
>3. **不能用作构造函数**：箭头函数不能用 `new` 关键字调用，否则会抛出错误。



### 二、模板字符串

---

模板字符串使用反引号（`）来定义，并且支持多行字符串和表达式嵌入。

#### 2.1 基本语法

```javascript
const name = "John";
const age = 30;

// 传统字符串拼接
console.log("Hello, my name is " + name + " and I am " + age + " years old.");

// 模板字符串
console.log(`Hello, my name is ${name} and I am ${age} years old.`);
```

#### 2.2 多行字符串

模板字符串支持多行字符串，无需使用 `\n` 或连接符。

```javascript
// 传统多行字符串
console.log("Hello,\nThis is a multi-line string.");

// 模板字符串
console.log(`Hello,
This is a multi-line string.`);
```

#### 2.3 表达式嵌入

模板字符串允许在字符串中嵌入表达式，表达式需要用 `${}` 包裹。

```javascript
const a = 10;
const b = 20;

// 嵌入表达式
console.log(`The sum of ${a} and ${b} is ${a + b}.`);
console.log(`The result of ${a} * ${b} is ${a * b}.`);
```

#### 2.4 条件语句

可以在模板字符串中嵌入条件语句。

```javascript
const isStudent = true;

console.log(`Is this person a student? ${isStudent ? 'Yes' : 'No'}`);
```

#### 2.5 函数调用

可以在模板字符串中嵌入函数调用。

```javascript
function greet(name) {
  return `Hello, ${name}!`;
}

console.log(greet("John")); // Hello, John!
```

#### 2.6 标签模板

模板字符串可以与函数结合使用，形成标签模板。标签模板允许在模板字符串前加上一个函数，函数会处理模板字符串。

```javascript
function highlight(strings, ...values) {
  let result = '';
  for (let i = 0; i < values.length; i++) {
    result += strings[i] + `<strong>${values[i]}</strong>`;
  }
  return result + strings[strings.length - 1];
}

const name = "John";
const age = 30;

console.log(highlight`Name: ${name}, Age: ${age}`);
// 输出：Name: <strong>John</strong>, Age: <strong>30</strong>
```

#### 2.7 转义字符

在模板字符串中，反引号（`）需要用反斜杠（\）转义。

```javascript
console.log(`This is a backtick: \``);
```



### 三、实战：天气卡片生成器

---

#### 3.1 准备

1. 前往[OpenWeatherMap](https://openweathermap.org/)注册账号并获取API Key
2. 准备天气相关图标（或使用图标字体）

#### 3.2 实现

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>天气卡片生成器</title>
    <style>
        :root {
            --sunny: linear-gradient(135deg, #f6d365 0%, #fda085 100%);
            --rainy: linear-gradient(135deg, #6B8DD6 0%, #8E37D7 100%);
            --cloudy: linear-gradient(135deg, #BDC3C7 0%, #2C3E50 100%);
        }

        body {
            font-family: 'Segoe UI', Arial, sans-serif;
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
            background: #f0f2f5;
        }

        .search-box {
            display: flex;
            gap: 10px;
            margin-bottom: 30px;
        }

        #cityInput {
            flex: 1;
            padding: 12px;
            border: 2px solid #ddd;
            border-radius: 8px;
            font-size: 16px;
        }

        #searchBtn {
            padding: 12px 30px;
            background: #2196F3;
            color: white;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            transition: transform 0.2s;
        }

        #searchBtn:hover {
            transform: scale(1.05);
        }

        .weather-cards {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
            gap: 20px;
        }

        .weather-card {
            background: white;
            border-radius: 12px;
            padding: 20px;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
            animation: fadeIn 0.5s ease-in;
            position: relative;
            overflow: hidden;
            transition: transform 0.2s, box-shadow 0.2s;
            /* 添加过渡效果 */
            cursor: pointer;
            /* 添加鼠标指针样式 */
        }

        .weather-card:hover {
            transform: scale(1.05);
            /* 悬停时放大卡片 */
            box-shadow: 0 8px 12px rgba(0, 0, 0, 0.2);
            /* 悬停时加深阴影 */
        }

        .weather-card:active {
            transform: scale(0.98);
            /* 点击时缩小卡片 */
            box-shadow: 0 12px 16px rgba(0, 0, 0, 0.3);
            /* 点击时加深阴影 */
        }

        .weather-card.clicked {
            transform: scale(0.98);
            box-shadow: 0 12px 16px rgba(0, 0, 0, 0.3);
            transition: transform 0.2s, box-shadow 0.2s;
        }

        .weather-card::before {
            content: "";
            position: absolute;
            top: -50px;
            right: -50px;
            width: 100px;
            height: 100px;
            background: rgba(255, 255, 255, 0.1);
            border-radius: 50%;
        }

        @keyframes fadeIn {
            from {
                opacity: 0;
                transform: translateY(20px);
            }

            to {
                opacity: 1;
                transform: translateY(0);
            }
        }

        .card-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 20px;
        }

        .city-name {
            font-size: 1.4rem;
            font-weight: bold;
            color: #333;
        }

        .weather-icon {
            width: 60px;
            height: 60px;
        }

        .temperature {
            font-size: 2.5rem;
            font-weight: bold;
            color: #2196F3;
            margin-bottom: 10px;
        }

        .weather-info {
            display: grid;
            grid-template-columns: repeat(2, 1fr);
            gap: 15px;
            color: #949292;
        }

        .info-item {
            display: flex;
            align-items: center;
            gap: 8px;
        }

        .info-item img {
            width: 24px;
            height: 24px;
        }

        .loading {
            text-align: center;
            padding: 20px;
            color: #666;
            display: none;
        }

        .error {
            color: #f44336;
            text-align: center;
            margin-top: 20px;
        }
    </style>
</head>

<body>
    <div class="search-box">
        <input type="text" id="cityInput" placeholder="输入城市名称，例如：北京" autocomplete="off">
        <button id="searchBtn">查询天气</button>
    </div>

    <div class="loading" id="loading">加载中...</div>
    <div class="error" id="error"></div>
    <div class="weather-cards" id="weatherCards"></div>

    <script src="./weather-card-generator.js"></script>
</body>

</html>
```

```javascript
const API_KEY = '5ce5a9ba3d778ce6604b6325d28a8d2d'; // 替换为你的OpenWeatherMap API Key
const weatherCards = document.getElementById('weatherCards');
const cityInput = document.getElementById('cityInput');
const searchBtn = document.getElementById('searchBtn');
const loading = document.getElementById('loading');
const error = document.getElementById('error');

// 天气图标映射
const ICON_BASE_URL = 'https://openweathermap.org/img/wn/';
const weatherThemes = {
    'Clear': { icon: '01', bg: 'var(--sunny)' },
    'Clouds': { icon: '02', bg: 'var(--cloudy)' },
    'Rain': { icon: '09', bg: 'var(--rainy)' },
    'Thunderstorm': { icon: '11', bg: 'var(--rainy)' },
    'Snow': { icon: '13', bg: 'var(--cloudy)' }
};

// 事件监听
searchBtn.addEventListener('click', getWeather);
cityInput.addEventListener('keypress', e => {
    if (e.key === 'Enter') getWeather();
});

async function getWeather() {
    const city = cityInput.value.trim();
    if (!city) return showError('请输入城市名称');

    try {
        showLoading(true);
        const data = await fetchWeatherData(city);
        data.list.forEach(element => {
            addWeatherCard(element);
        });
        cityInput.value = '';
        error.textContent = '';
    } catch (err) {
        showError(err.message);
    } finally {
        showLoading(false);
    }
}

async function fetchWeatherData(city) {
    const response = await fetch(
        `http://api.openweathermap.org/data/2.5/forecast?q=${city}&appid=${API_KEY}&units=metric&lang=zh_cn`
    );

    if (!response.ok) {
        const errData = await response.json();
        throw new Error(errData.message || '获取天气数据失败');
    }

    return response.json();
}

function addWeatherCard(element) {
    let cityName = cityInput.value.trim();
    const {
        main: { temp, humidity },
        weather: [weatherInfo],
        wind: { speed: windSpeed }
    } = element;

    const card = document.createElement('div');
    card.className = 'weather-card';
    card.innerHTML = `
                <div class="card-header">
                    <div class="city-name">${cityName}</div>
                    <img class="weather-icon" 
                         src="${ICON_BASE_URL}${weatherInfo.icon}@2x.png" 
                         alt="${weatherInfo.description}">
                </div>
                <div class="temperature">${Math.round(temp)}°C</div>
                <div class="weather-info">
                    <div class="info-item">
                        <img src="humidity-icon.png" alt="湿度">
                        <span>湿度 ${humidity}%</span>
                    </div>
                    <div class="info-item">
                        <img src="wind-icon.png" alt="风速">
                        <span>风速 ${windSpeed}m/s</span>
                    </div>
                    <div class="info-item">
                        <img src="weather-icon.png" alt="天气">
                        <span>${weatherInfo.description}</span>
                    </div>
                </div>
            `;

    // 根据天气类型设置背景
    const theme = weatherThemes[weatherInfo.main] || weatherThemes.Clouds;
    card.style.background = theme.bg;

    // 根据温度设置字体颜色
    const temperatureElement = card.querySelector('.temperature');
    if (temp > 30) {
        temperatureElement.style.color = 'red';
    } else {
        temperatureElement.style.color = '#2196F3'; // 默认颜色
    }

    // 添加新卡片到最前面
    weatherCards.insertAdjacentElement('afterbegin', card);
}

function showLoading(show) {
    loading.style.display = show ? 'block' : 'none';
}

function showError(message) {
    error.textContent = message;
    setTimeout(() => error.textContent = '', 3000);
}
```

