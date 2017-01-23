## Java 核心概念

#### equals 与 hashCode 的异同点在哪里？Java 的集合中又是如何使用它们的
定义在Object中，默认的，Object类的hashCode()方法返回这个对象存储的内存地址的编号。
重写equals必须重写hashcode，

#### 接口（Interface）及意义
1. 接口是抽象类型，无法实例化
2. 用来指定`class`必须实现的方法，使不同类的对象可以利用相同的interface进行沟通
3. 用来模拟多继承
4. 接口无法实现另一个接口

规范，扩展，回调

#### 抽象类(Abstract class) 及意义
1. 由abstract关键字定义，不一定包含abstract方法
2. 无法实例化，但可以是其他类的子类
3. 定义抽象概念，对共通的方法和属性进行规约

苹果和橘子都是水果，但是水果是个概念，不会有实例。

#### 比较抽象类和接口，何时使用哪一个？
1. 抽象类可以对共通的方法和属性提供一个实现，而接口不能
2. 接口的属性是public static final的，而抽象类没有限制
3. 接口的所有方法都是public的，而抽象类则没有限制
4. 一个类只能实现一个class，但是可以实现任意多接口

使用抽象类：

1. 在密切相关的类间共享代码
2. 子类继承抽象类有许多公共属性和方法或者需要非public修饰符
3. 需要声明非static，非final的属性

使用接口：

1. 不相关的类实现接口
2. 指定特定数据类型的行为，但是不关心实现
3. 利用多继承

#### Java 中集合（Collections）
1. 集合，也可称作容器，是一个将多个元素组成一个单元的object。
2. 用于存储，检索，操作和传达聚合数据

[集合框架](https://docs.oracle.com/javase/tutorial/collections/intro/)：集合框架适用于展示和操作集合的统一框架，都包含有接口，实现，算法。

#### LinkedList 与 ArrayList 的区别是什么?
1. LinkedList是基于链表，ArrayList基于数组
2. get/set，ArrayList更优，add/remove，LinedList更优

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
[wiki](https://zh.wikipedia.org/wiki/SOLID_(%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E8%AE%BE%E8%AE%A1))  

1. 单一功能原则(Single responsibility principle)
2. 开闭原则(Open - close principle)
3. 里氏替换原则(Liskov Substitution principle)
4. 接口隔离原则(interface-segregation principle)
5. 依赖反转原则(Dependency inversion principle)

#### 其他的譬如 KISS,DRY,YAGNI 等原则又是什么含义

#### 什么是设计模式（Design Patterns）？你知道哪些设计模式？
[设计模式](https://zh.wikipedia.org/wiki/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F_(%E8%AE%A1%E7%AE%97%E6%9C%BA))是对软件设计中普遍存在（反复出现）的各种问题，所提出的解决方案。是描述在各种不同情况下，要怎么解决问题的一种方案。  

1. 抽象工厂(Abstract factory pattern)
2. 工厂方法(Factory method pattern)
3. 生成器(Builder Pattern)
4. 原型模式
5. 单例模式
6. 适配器模式
7. 组合模式
8. 修饰模式
9. 外观模式
10. 责任链
11. 迭代器
12. 终结者
13. 备忘录
14. 观察者
15. 策略模式
16. 模板方法
17. 访问者

#### 你有了解过存在哪些反模式（Anti-Patterns）吗？
一个[反面模式](https://zh.wikipedia.org/wiki/%E5%8F%8D%E9%9D%A2%E6%A8%A1%E5%BC%8F)(anti-pattern或antipattern)指的是在实践中明显出现但又低效或是有待优化的设计模式，是用来解决问题的带有共同性的不良方法。它们已经经过研究并分类，以防止日后重蹈覆辙，并能在研发尚未投产的系统时辨认出来。

#### 你会如何设计登陆舰/数学表达式计算程序/一条龙？

#### 你知道哪些基本的排序算法，它们的计算复杂度如何？在给定数据的情况下你会倾向于使用哪种算法呢？
[wiki](https://zh.wikipedia.org/wiki/%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95)  
冒泡排序  
插入排序  
归并排序  
堆排序  
快速排序


#### 尝试编写如下代码
1. 计算指定数字的阶乘
2. 开发 Fizz Buzz 小游戏
3. 倒转句子中的单词
4. 回文字符串检测
5. 枚举给定字符串的所有排列组合


### 原文
[https://howtotrainyourjava.com/2016/07/14/java-developer-interview-questions-the-hard-part/](https://howtotrainyourjava.com/2016/07/14/java-developer-interview-questions-the-hard-part/)  
[https://github.com/JackyAndroid/AndroidInterview-Q-A/blob/master/README-CN.md](https://github.com/JackyAndroid/AndroidInterview-Q-A/blob/master/README-CN.md)