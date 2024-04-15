### export default

---

在 JavaScript 中，`export default` 是用于导出一个模块的主要功能或者是一个具体的实体。这个关键字用于模块化编程，通常与 `import` 语句一起使用。

在一个模块中使用 `export default` 时，你可以选择导出模块的一个默认值。这个默认值可以是一个函数、一个类、一个对象或者任何其他的 JavaScript 实体。

具体用法如下：

```js
// 导出一个默认函数
export default function add(a, b) {
  return a + b;
}

// 或者导出一个默认对象
export default {
  name: 'John Doe',
  age: 30,
  greet() {
    console.log(`Hello, my name is ${this.name}`);
  }
}

// 导出一个默认类
export default class Person {
  constructor(name) {
    this.name = name;
  }
}
```

当从另一个模块导入这些默认导出时，可以自由地命名导入的变量：

```js
// 导入默认导出并赋予任意名称
import sum from './add'; // sum 将会是上面定义的 add 函数
sum(2, 3); // 输出 5

import user from './user'; // user 将会是上面定义的对象
user.greet(); // 输出 "Hello, my name is John Doe"

import PersonClass from './Person';
const person = new PersonClass('Alice');
```

每个模块只能有一个 `export default`，但可以有多个 `export`（称为命名导出），并且命名导出在导入时必须按照导出时的名字来引用。而导入默认导出时，则无需关注其原始导出名称。



































