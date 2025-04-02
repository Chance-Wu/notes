### 一、数组核心方法

---

```javascript
// 创建数组
let scores = [85, 92, 78, 95];
console.log(">>>>>>>>创建数组scores: {}", scores);
let students = new Array(3).fill(null);
console.log(">>>>>>>>创建数组students: {}", students);

// 基础操作
scores.push(88);       // 末尾添加
console.log(">>>>>>>>scores末尾添加: {}", scores);
const last = scores.pop();     // 移除末尾
console.log(">>>>>>>>scores移除末尾: {}", scores);
scores.unshift(90);    // 开头添加
console.log(">>>>>>>>scores开头添加: {}", scores);
scores.shift();        // 移除开头
console.log(">>>>>>>>scores移除开头: {}", scores);

// 高阶方法
const highScores = scores.filter(s => s >= 90);
console.log(">>>>>>>筛选scores中大于等于90的元素: {}", highScores);
const doubleScores = scores.map(s => s * 2);
console.log(">>>>>>>scores中所有元素乘以2: {}", doubleScores);
const total = scores.reduce((sum, s) => sum + s, 0);
console.log(">>>>>>>scores中所有元素总和: {}", total);
scores.sort((a, b) => b - a);  // 降序排序
console.log(">>>>>>>scores降序排序: {}", scores);

// 解构赋值
const [first, second, ...others] = scores;
console.log(">>>>>>>scores解构赋值: {}", [first]);

```



### 二、对象操作技巧

---

```javascript
// 对象创建
const student = {
    id: 1001,
    name: "张三",
    courses: ["数学", "语文"],
    address: {
        city: "北京"
    }
};

// 属性操作
student.age = 20;                     // 添加属性
delete student.age;                   // 删除属性
const keys = Object.keys(student);    // 获取键数组
console.log(">>>>>>>>属性操作，获取键数组keys: {}", keys);
const values = Object.values(student);// 获取值数组
console.log(">>>>>>>>属性操作，获取值数组values: {}", values);

// 对象合并
const record = Object.assign({}, student, { score: 95 });
console.log(">>>>>>>>对象合并record: {}", record);
const newStudent = { ...student, grade: "三年级" };
console.log(">>>>>>>>newStudent: {}", newStudent);

// 可选链操作符
const city = student.address?.city ?? "未知"; // ES2020+
console.log(">>>>>>>>city: {}", city);
```



### 三、JSON数据处理

---

```javascript
// 对象创建
const student = {
    id: 1001,
    name: "张三",
    courses: ["数学", "语文"],
    address: {
        city: "北京"
    }
};

// 对象转JSON
const jsonStr = JSON.stringify(student, null, 2);
console.log(">>>>>>>>对象转JSON: ", jsonStr);

// JSON解析
const parsed = JSON.parse('{"id":1001,"name":"李四"}');
console.log(">>>>>>>>JSON解析: ", parsed);

// 深拷贝技巧
const deepCopy = JSON.parse(JSON.stringify(student));
console.log(">>>>>>>>深拷贝: ", deepCopy);
```



### 四、实战：学生成绩管理系统

---

```html
<!DOCTYPE html>
<html>
<head>
    <title>成绩管理系统</title>
    <style>
        table { width: 100%; border-collapse: collapse; }
        th, td { padding: 10px; border: 1px solid #ddd; }
        .form-group { margin: 10px 0; }
    </style>
</head>
<body>
    <div class="container">
        <h2>学生成绩管理</h2>
        
        <!-- 添加表单 -->
        <form id="addForm">
            <div class="form-group">
                <input type="text" id="name" placeholder="姓名" required>
            </div>
            <div class="form-group">
                <input type="number" id="score" placeholder="成绩" min="0" required>
            </div>
            <button type="submit">添加记录</button>
        </form>

        <!-- 筛选功能 -->
        <div class="filter">
            <input type="number" id="filterScore" placeholder="筛选分数">
            <button onclick="filterRecords()">筛选</button>
        </div>

        <!-- 数据展示 -->
        <table id="scoreTable">
            <thead>
                <tr>
                    <th>姓名</th>
                    <th>成绩</th>
                    <th>操作</th>
                </tr>
            </thead>
            <tbody></tbody>
        </table>
    </div>

    <script>
        // 数据存储模块
        const store = (() => {
            let records = JSON.parse(localStorage.getItem('scores')) || [];
            
            return {
                add: (name, score) => {
                    records.push({ id: Date.now(), name, score });
                    store.save();
                },
                delete: id => {
                    records = records.filter(r => r.id !== id);
                    store.save();
                },
                filter: minScore => {
                    return records.filter(r => r.score >= minScore);
                },
                save: () => {
                    localStorage.setItem('scores', JSON.stringify(records));
                    renderTable(records);
                },
                getAll: () => [...records]
            };
        })();

        // 表格渲染
        function renderTable(data) {
            const tbody = document.querySelector('#scoreTable tbody');
            tbody.innerHTML = data.map(r => `
                <tr>
                    <td>${r.name}</td>
                    <td>${r.score}</td>
                    <td>
                        <button onclick="deleteRecord(${r.id})">删除</button>
                    </td>
                </tr>
            `).join('');
        }

        // 表单提交
        document.getElementById('addForm').addEventListener('submit', e => {
            e.preventDefault();
            const name = document.getElementById('name').value.trim();
            const score = +document.getElementById('score').value;
            
            if(name && score >= 0) {
                store.add(name, score);
                e.target.reset();
            }
        });

        // 筛选功能
        function filterRecords() {
            const minScore = +document.getElementById('filterScore').value || 0;
            const filtered = store.filter(minScore);
            renderTable(filtered);
        }

        // 删除记录
        function deleteRecord(id) {
            if(confirm('确定删除？')) {
                store.delete(id);
            }
        }

        // 初始化
        renderTable(store.getAll());
    </script>
</body>
</html>
```

1. **数据持久化**：使用localStorage存储数据
2. **模块化设计**：通过IIFE封装数据操作
3. **响应式更新**：数据变更自动触发界面刷新
4. **数据验证**：表单输入有效性检查
5. **交互功能**：包含增删查基本操作



### 五、扩展

---

1. 添加成绩修改功能
2. 实现数据分页显示
3. 添加课程分类字段
4. 开发数据统计图表
5. 增加数据导入/导出功能
6. 实现姓名模糊搜索