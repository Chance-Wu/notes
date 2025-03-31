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
let globalVar = "全局";

function outer() {
  let outerVar = "外层";
  
  function inner() {
    let innerVar = "内层";
    console.log(globalVar + outerVar + innerVar);
  }
  inner();
}
outer(); // "全局外层内层"
```

#### 3.2 闭包应用

```javascript
function createCounter() {
  let count = 0;
  return function() {
    return ++count;
  };
}

const counter = createCounter();
console.log(counter()); // 1
console.log(counter()); // 2
```



### 四、立即调用函数（IIFE）

---

```javascript
(function() {
  const secret = "临时值";
  console.log("立即执行");
})();

// 模块化示例
const calculator = (function() {
  let memory = 0;
  
  return {
    add: x => memory += x,
    get: () => memory
  };
})();

calculator.add(5);
console.log(calculator.get()); // 5
```



### 五、实战案例：待办事项管理

---

```html
<!DOCTYPE html>
<html>
<head>
    <title>待办事项</title>
    <style>
        .todo-item { margin: 5px; padding: 8px; border: 1px solid #ddd; }
        .completed { text-decoration: line-through; opacity: 0.6; }
    </style>
</head>
<body>
    <input id="todoInput" placeholder="输入任务">
    <button onclick="addTodo()">添加</button>
    <div id="todoList"></div>

    <script>
        // 状态管理模块
        const todoManager = (() => {
            let todos = JSON.parse(localStorage.getItem('todos')) || [];
            
            function save() {
                localStorage.setItem('todos', JSON.stringify(todos));
                render();
            }
            
            function render() {
                const list = document.getElementById('todoList');
                list.innerHTML = todos.map((todo, index) => `
                    <div class="todo-item ${todo.done ? 'completed' : ''}">
                        <input type="checkbox" 
                               ${todo.done ? 'checked' : ''} 
                               onchange="toggleTodo(${index})">
                        ${todo.text}
                        <button onclick="deleteTodo(${index})">×</button>
                    </div>
                `).join('');
            }
            
            return {
                add: text => {
                    todos.push({ text, done: false });
                    save();
                },
                toggle: index => {
                    todos[index].done = !todos[index].done;
                    save();
                },
                delete: index => {
                    todos.splice(index, 1);
                    save();
                },
                init: render
            };
        })();
        
        // 用户操作接口
        function addTodo() {
            const input = document.getElementById('todoInput');
            if(input.value.trim()) {
                todoManager.add(input.value.trim());
                input.value = '';
            }
        }
        
        function toggleTodo(index) {
            todoManager.toggle(index);
        }
        
        function deleteTodo(index) {
            todoManager.delete(index);
        }
        
        // 初始化
        todoManager.init();
    </script>
</body>
</html>
```
