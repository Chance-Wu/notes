继承可以实现类的复用。但是，遇到想要复用的场景就直接使用继承，这样做是不对的。长期大量的使用继承会给代码带来很高的维护成本。

还有一种可以帮助我们复用的新的概念——组合。

面向对象的复用技术的三种具体的表现形式：

- 继承
- 组合
- 代理

#### 1. 继承

---

继承是类与类或者接口与接口之间最常见的一种关系；继承是一种`is-a`关系。

>is-a：表示"是一个"的关系，如狗是一个动物。
>
>![](https://tva1.sinaimg.cn/large/e6c9d24egy1h0hvc3qxyyj208505omx1.jpg)



#### 2. 组合

---

组合(Composition)体现的是整体与部分、拥有的关系，即`has-a`的关系。

> is-a：表示"有一个"的关系，如狗有一个尾巴。
>
> ![](https://tva1.sinaimg.cn/large/e6c9d24egy1h0hvdxz07kj20by02ya9v.jpg)



#### 3. 组合与继承的区别

---

在**继承**结构中，==父类的内部细节对于子类是可见的==。通常也可以说通过继承的代码复用是一种白盒式代码复用。（如果基类的实现发生改变，那么派生类的实现也将随之改变。这样就导致了子类行为的不可预知性；）

**组合**是通过**对现有的对象进行拼装（组合）产生新的、更复杂的功能**。因为在对象之间，各自的内部细节是不可见的，所以我们也说这种方式的代码复用是黑盒式代码复用。（因为组合中一般都定义一个类型，所以在编译期根本不知道具体会调用哪个实现类的方法）

继承，在写代码的时候就要指名具体继承哪个类，所以，在**编译期**就确定了关系。（从基类继承来的实现是无法在运行期动态改变的，因此降低了应用的灵活性。）

组合，在写代码的时候可以采用面向接口编程。所以，类的组合关系一般在运行期确定。

| 组 合 关 系                                                  | 继 承 关 系                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 优点：不破坏封装，整体类与局部类之间松耦合，彼此相对独立     | 缺点：破坏封装，子类与父类之间紧密耦合，子类依赖于父类的实现，子类缺乏独立性 |
| 优点：具有较好的可扩展性                                     | 缺点：支持扩展，但是往往以增加系统结构的复杂度为代价         |
| 优点：支持动态组合。在运行时，整体对象可以选择不同类型的局部对象 | 缺点：不支持动态继承。在运行时，子类无法选择不同的父类       |
| 优点：整体类可以对局部类进行包装，封装局部类的接口，提供新的接口 | 缺点：子类不能改变父类的接口                                 |
| 缺点：整体类不能自动获得和局部类同样的接口                   | 优点：子类能自动继承父类的接口                               |
| 缺点：创建整体类的对象时，需要创建所有局部类的对象           | 优点：创建子类的对象时，无须创建父类的对象                   |



#### 4. 如何选择

---

>建议在同样可行的情况下，优先使用组合而不是继承。
>
>因为组合更安全，更简单，更灵活，更高效。

>继承要慎用，一个判断方法是，问一问自己是否需要从新类向基类进行向上转型。如果是必须的，则继承是必要的。反之则应该好好考虑是否需要继承。《[Java编程思想](http://s.click.taobao.com/t?e=m%3D2%26s%3DHzJzud6zOdocQipKwQzePOeEDrYVVa64K7Vc7tFgwiHjf2vlNIV67vo5P8BMUBgoEC56fBbgyn5pS4hLH%2FP02ckKYNRBWOBBey11vvWwHXSniyi5vWXIZhtlrJbLMDAQihpQCXu2JnPFYKQlNeOGCsYMXU3NNCg%2F&pvid=10_125.119.86.125_222_1458652212179)》
>
>只有当子类真正是超类的子类型时，才适合用继承。换句话说，对于两个类A和B，只有当两者之间确实存在 is-a 关系的时候，类B才应该继承类A。《[Effective Java](http://s.click.taobao.com/t?e=m%3D2%26s%3DwIPn8%2BNPqLwcQipKwQzePOeEDrYVVa64K7Vc7tFgwiHjf2vlNIV67vo5P8BMUBgoUOZr0mLjusdpS4hLH%2FP02ckKYNRBWOBBey11vvWwHXSniyi5vWXIZvgXwmdyquYbNLnO%2BjzYQLqKnzbV%2FMLqnMYMXU3NNCg%2F&pvid=10_125.119.86.125_345_1458652241780)》