## 面向对象编程的基本理念与核心设计思想
#### 解释多态性（polymorphism），封装性（encapsulation），内聚（cohesion）以及耦合（coupling）
继承：[计算机程序运行时，相同的消息可能会送给多个不同的类别之对象，而系统可依据对象所属类别，引发对应类别的方法，而有不同的行为。简单来说，所谓多态意指相同的消息给予不同的对象会引发不同的动作称之。](https://zh.wikipedia.org/wiki/%E5%A4%9A%E5%9E%8B_(%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%A7%91%E5%AD%A6))  
动态多态：通过类(接口)继承机制和虚函数机制生效于运行时。
静态多态：函数重载，运算符重载，参数化多态(编译期)  

封装性：[将抽象性函数接口的实现细节部分包装、隐藏起来的方法,防止外界调用端，去访问对象内部实现细节的手段](https://zh.wikipedia.org/wiki/%E5%B0%81%E8%A3%9D_(%E7%89%A9%E4%BB%B6%E5%B0%8E%E5%90%91%E7%A8%8B%E5%BC%8F%E8%A8%AD%E8%A8%88))  
让代码更容易理解与维护，也加强了代码的安全性。

内聚，耦合：[wiki](https://zh.wikipedia.org/wiki/%E5%85%A7%E8%81%9A%E6%80%A7_(%E8%A8%88%E7%AE%97%E6%A9%9F%E7%A7%91%E5%AD%B8))

#### 继承（Inheritance）与聚合（Aggregation）的区别在哪里
继承(is a)是从上到下，聚合(has a)是部分与整体

#### 你是如何理解干净的代码（Clean Code）与技术负载（Technical Debt）的


#### 描述下常用的重构技巧
1. 重复代码的提炼
2. 分割冗长代码
3. 优化嵌套条件分支(1.卫语句 2.合并条件)
4. 一次性变量
5. 过长参数列表
6. 提取常量
7. 让类提供应该提供的方法
8. 拆分冗长的类
9. 提取继承体系中重复的属性与方法到父类


#### 阐述下 SOLID 原则
1. 单一功能原则(Single responsibility principle)
2. 开闭原则(Open - close principle)
3. 里氏替换原则(Liskov Substitution principle)
4. 接口隔离原则(interface-segregation principle)
5. 依赖反转原则(Dependency inversion principle)

#### 其他的譬如 KISS,DRY,YAGNI 等原则又是什么含义

#### 什么是设计模式（Design Patterns）？你知道哪些设计模式？

#### 你有了解过存在哪些反模式（Anti-Patterns）吗？

#### 你会如何设计登陆舰/数学表达式计算程序/一条龙？

#### 你知道哪些基本的排序算法，它们的计算复杂度如何？在给定数据的情况下你会倾向于使用哪种算法呢？

#### 尝试编写如下代码
1. 计算指定数字的阶乘
2. 开发 Fizz Buzz 小游戏
3. 倒转句子中的单词
4. 回文字符串检测
5. 枚举给定字符串的所有排列组合

## Java 核心概念
#### equals 与 hashCode 的异同点在哪里？Java 的集合中又是如何使用它们的

#### 描述下 Java 中集合（Collections），接口（Interfaces），实现（Implementations）的概念。LinkedList 与 ArrayList 的区别是什么

#### 基础类型（Primitives）与封装类型（Wrappers）的区别在哪里？

#### final 与 static 关键字可以用于哪里？它们的作用是什么？

#### 阐述下 Java 中的访问描述符（Access Modifiers）

#### 描述下 String,StringBuilder 以及 StringBuffer 区别

#### 接口（Interface）与抽象类（Abstract Class）的区别在哪里

#### 覆盖（Overriding）与重载（OverLoading）的区别在哪里

#### 异常分为哪几种类型？以及所谓的handle or declare原则应该如何理解？

#### 简述垃圾回收器的工作原理

#### 你是如何处理内存泄露或者栈溢出问题的？

#### 如何构建不可变的类结构？关键点在哪里？

#### 什么是 JIT 编译？

#### Java 8 / Java 7 为我们提供了什么新功能？即将到来的 Java 9 又带来了怎样的新功能？



### 原文
[https://howtotrainyourjava.com/2016/07/14/java-developer-interview-questions-the-hard-part/](https://howtotrainyourjava.com/2016/07/14/java-developer-interview-questions-the-hard-part/)