# Java 核心概念
## 基础数据类型
8个：byte，short，int，long，float，double，char，boolean  
byte：8位，1字节，有符号，-128 ~ 127  
short：16位，有符号，-32768 ~ 32767  
int：32位，4字节，有符号  
long：64位，有符号  
float：32位，单精度，不能用来表示精确的值，如货币  
double：64位，双精度，不能表示精确的值，如货币  
char：16位Unicode字符

自动类型转换：  
byte，short，char—> int —> long—> float —> double

### 基础类型（Primitives）与封装类型（Wrappers）的区别在哪里？
基础数据类型是基本类型，但是封装类型是类

#### 拆箱/装箱问题
输出结果是什么？

```
public class Main {
	public static void main(String[] args) {
		Integer i1 = 100;
		Integer i2 = 100;
		Integer i3 = newInteger(100);
		Integer i4 = newInteger(100);
		
		System.out.println(i1==i2);
		System.out.println(i3==i4);
	}
}
```

true  
false

```
public static Integer valueOf(int i) {
        return  i >= 128 || i < -128 ? new Integer(i) : SMALL_VALUES[i + 128];
}
```

#### 其他
`3*0.1 == 0.3`返回值  
false，因为浮点数不能完全精确的表示出来

`a=a+b`与`a+=b`有什么区别吗?  
+=操作符会进行隐式自动类型转换

## == equals and hashcode
|               | ==              | equals        |
|:------------- |:---------------:|:-------------:|
| 基本数据类型    | 值              | 不可用         |
| 包装类         | 内存地址         | 内容           |
| 字符串         | 内存首地址        | 字符串内容      |
| 非字符串       | 内存首地址        | 内存首地址      |

* == 是一个运算符。 equals则是对象的方法
* java中 值类型 是存储在内存中的栈中。 而引用类型在栈中仅仅是存储引用类型变量的地址，而其本身则存储在堆中。所以字符串的内容相同，引用地址不一定相同，有可能创建了多个对象。
* ==操作比较的是两个变量的值是否相等，对于引用型变量表示的是两个变量在堆中存储的地址是否相同，即栈中的内容是否相同。
* ==比较的是2个对象的地址（栈中），而equals比较的是2个对象的内容（堆中）。所以当equals为true时，==不一定为true。

### String的equals方法
```
public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }
        if (anObject instanceof String) {
            String anotherString = (String) anObject;
            int n = count;
            if (n == anotherString.count) {
                int i = 0;
                while (n-- != 0) {
                    if (charAt(i) != anotherString.charAt(i))
                            return false;
                    i++;
                }
                return true;
            }
        }
        return false;
    }
```

### equals 与 hashCode。Java 的集合中又是如何使用它们的
* 都定义在Object中，如果两个对象`equal()`方法比较相等，那么两者的`hashCode()`方法必须产生相同的哈希值。反之不然。  
* 重写equals必须重写hashcode。
* hashcode常用于`Hashtable`,`HashMap`,`LinkedHashMap`等基于Hash的集合类


```
HashMap.java

	public V put(K key, V value) {
        if (table == EMPTY_TABLE) {
            inflateTable(threshold);
        }
        if (key == null)
            return putForNullKey(value);
        int hash = sun.misc.Hashing.singleWordWangJenkinsHash(key);
        int i = indexFor(hash, table.length);
        for (HashMapEntry<K,V> e = table[i]; e != null; e = e.next) {
            Object k;
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }

        modCount++;
        addEntry(hash, key, value, i);
        return null;
    }
```
看`HashMap`的实现，最终会算出hash值，然后得到对应的位置i，如果在此位置，没有value，直接`addEntry`，如果有，则判断hash和key都相同的情况下，替换原来的值，如果不都相同，最后也会执行到`addEntry`方法。

```
void addEntry(int hash, K key, V value, int bucketIndex) {
        if ((size >= threshold) && (null != table[bucketIndex])) {
            resize(2 * table.length);
            hash = (null != key) ? sun.misc.Hashing.singleWordWangJenkinsHash(key) : 0;
            bucketIndex = indexFor(hash, table.length);
        }

        createEntry(hash, key, value, bucketIndex);
    }
```
方法中判断需要增大空间，则申请原来2倍大的内存然后重新计算要插入的参数。

```
void createEntry(int hash, K key, V value, int bucketIndex) {
        HashMapEntry<K,V> e = table[bucketIndex];
        table[bucketIndex] = new HashMapEntry<>(hash, key, value, e);
        size++;
    }
```
`HashMapEntry`中有个next节点，这是把所有值用链表串了起来。

所以当get value的时候，对应位置上的取值是要遍历找到那个hash和key都相等的value。

```
final Entry<K,V> getEntry(Object key) {
        if (size == 0) {
            return null;
        }

        int hash = (key == null) ? 0 : sun.misc.Hashing.singleWordWangJenkinsHash(key);
        for (HashMapEntry<K,V> e = table[indexFor(hash, table.length)];
             e != null;
             e = e.next) {
            Object k;
            if (e.hash == hash &&
                ((k = e.key) == key || (key != null && key.equals(k))))
                return e;
        }
        return null;
    }
```

## String，StringBuilder，StringBuffer
* String 字符串常量  
* StringBuilder 字符串变量(非线程安全)
* StringBuffer 字符串变量(线程安全)

String是不可变的，所以每次改变都是创建新的对象，如果经常改变字符串内容会导致产生大量无用对象，触发GC。 StringBuilder比StringBuffer效率更高。单线程时，优先使用StringBuilder。

```
	1.String S1 = "This is only a" + " simple" + " test";
	2.StringBuffer Sb = new StringBuilder(“This is only a”)
						 .append(“ simple”).append(“ test”)
```  
对于JVM 1其实就是`String S1 = "This is only a simple test"`，所以比StringBuffer还要快。  

```
	String s = new String("xyz")创建了几个String对象
```
2个，"xyz"创建于字符串常量池，new String()创建在堆中。

### 为什么String要设计成不可变的
* 字符串常量池的需要，可以节约heap空间
* 线程安全。同一个字符串实例可以被多个线程共享。这样便不用因为线程安全问题而使用同步
* 类加载器用到字符串，不可变性提供了安全性，以便正确的类被加载。譬如你想加载java.sql.Connection类，而这个值被改成了myhacked.Connection，那么会对你的数据库造成不可知的破坏。
* 支持hash映射和缓存。 因为字符串是不可变的，所以在它创建的时候hashcode就被缓存了，不需要重新计算。这就使得字符串很适合作为Map中的键，字符串的处理速度要快过其它的键对象。这就是HashMap中的键往往都使用字符串。

### Switch能都用String做参数
jdk1.7之后支持，1.7之前支持`byte`，`short`，`char`，`int`，并且会自动转换为int类型，精度大于int的不会。

## 集合
### Java 中集合（Collections）
1. 集合，也可称作容器，是一个将多个元素组成一个单元的object。
2. 用于存储，检索，操作和传达聚合数据

[集合框架](https://docs.oracle.com/javase/tutorial/collections/intro/)：集合框架适用于展示和操作集合的统一框架，都包含有接口，实现，算法。C++的STL。  

Java集合框架的好处：  

1. 减少编程工作量，专注于核心业务
2. 增加程序运行速度和质量
3. 允许无关API之间相互调用
4. 减少设计新API的工作量
5. 促进重用

[集合框架的层次结构和使用规则梳理](https://github.com/helen-x/AndroidInterview/blob/master/java/%5BJava%5D%20集合框架的层次结构和使用规则梳理.md)

![](https://github.com/DroidWorkerLYF/LearnX/blob/master/Basis/resource/CollectionFramework.png?raw=true)

### ArrayList继承AbstractList并且实现了List接口，AbstractList已经实现了List接口，这是为什么
1. 便于查看代码，而不用遍历整个结构
2. AbstractList这样的类只是用于减少实现List接口的class的重复代码
3. 如果AbstractList以后实现的接口改变了，那么会导致之前的代码编译失败

### ArrayList，LinkedList与Vector
三者都实现了`List`接口  

#### ArrayList

1. `ArrayList`是`List`接口的可变大小数组实现，支持所有list的操作，并且接受所有元素参数，包括`Null`
2. `ArrayList`提供方法来操作内部存储数据的数组的大小
3. `ArrayList`几乎等同于`Vector`，不过不是同步的，fail-fast
4. `size`,`isEmpty`,`get`,`set`,`iterator`,`listIterator`操作时间复杂度都是常量，添加n个元素需要O(n)的时间，恒定分摊时间，其他操作都是线性时间

#### LinkedList
1. `LinkedList`是`List`和`Deque(双端队列)`的双链表实现，支持所有list的操作，并且接受所有元素参数，包括`Null`
2. 索引操作将从list靠近指定索引的一端(头部或尾部)开始遍历
3. 不是同步的，fail-fast

#### Vector
1. 实现可增长的对象数组，大小可以根据需要增加或减小
2. `Vector`会优化存储，使用capacity和capacityIncrement
3. 同步的，fail-fast

`ArrayList`是一个可变大小数组，可以动态增长大小，随机访问快，删除非头尾元素慢，新增元素时，空间不够，则操作慢且费时，非线程安全。  
`LinkedList`是一个双链表，添加删除元素快，但是访问元素慢，不耗费多余资源，非线程安全。  
`Vector`类似`ArrayList`，不过是同步的，与`ArrayList`一样，添加更多元素时请求更大空间，`Vector`请求其大小的双倍空间，`ArrayList`通常是每次增大50%。  

最终发现需要增长大小时  
`ArrayList`

```
private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```

`Vector`

```
private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                         capacityIncrement : oldCapacity);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```

### HashMap 和 Hashtable 的区别？
1. Hashtable是同步的(线程安全)，HashMap是异步的。
2. Hashtable不接受null作为key或者value，HashMap可以接受一个key为null和多个value为null。

#### HashMap
[HashMap分析](https://github.com/helen-x/AndroidInterview/blob/master/java/%5BJava%5D%20HashMap源码分析.md)

1. 实现了`Map`接口，并提供所有可选操作，接受null作为key和value。
2. 非同步版本的`Hashtable`
3. `get`和`put`操作都是常量时间，遍历操作受到`capacity`和key-value对数量的影响
4. 初始容量(capacity)和load factor影响性能

#### Hashtable
1. 接受任何null以外的对象作为key和value
2. 作为key的对象必须实现hashcode和equals方法


### HashMap 和 ArrayMap 的区别？
ArrayMap内部使用一个integer数组维护每个item的hash code，一个Object数组存储key/value对，这样避免了每次put都创建额外的对象，而且增长大小的时候，不需要重建整个Hash map。所以ArrayMap被用来更好的平衡内存使用，但是包含大量item时，效率不及传统的HashMap。

## 泛型

## 进程，线程
[Processes and Threads](https://docs.oracle.com/javase/tutorial/essential/concurrency/procthread.html)
#### 进程
进程包含完整的执行环境。一个进程通常有一套完整的私有的基本运行时资源。每个进程有自己的内存空间。  
为了帮助进程之间通信，大部分操作系统支持IPC，例如管道和套接字。

#### 线程
线程可称为轻量级的进程，进程和线程都提供执行环境，但是创建进程比线程需要更少的资源。  
线程存在于进程中，每个进程有至少一个线程，线程间共享进程的资源包括内存和打开的文件

### sleep()和wait()
1. sleep是Thread类方法，调用此方法的线程sleep，比如a线程中调用b.sleep()，实际是a sleep。
2. wait是Object的方法
3. sleep是自动唤醒，wait需要其他线程唤醒
3. sleep不会释放同步锁，wait会释放同步锁
4. sleep可以用在任意方法中，wait，notify，notifyAll只能用在同步方法或同步块中

### 线程池
[ThreadPool用法与示例](https://github.com/helen-x/AndroidInterview/blob/master/java/%5BJava%5D%20ThreadPool用法与示例.md)

### ThreadLocal
[参考1](https://github.com/helen-x/AndroidInterview/blob/master/java/%5BJava%5D%20ThreadLocal的使用规则和源码分析.md)
[参考2](http://blog.csdn.net/lufeng20/article/details/24314381)

### 线程安全
多个线程访问时，不管运行时环境采用何种调度方式，在代码中不需要任何额外的同步，这个类都能表现出正确的行为，那么这个类就是线程安全的。

#### volatile
保持单个简单volatile变量的读/写操作的具有原子性

#### 方发锁，对象锁，类锁
[](https://github.com/helen-x/AndroidInterview/blob/master/java/%5BJava%5D%20方法锁、对象锁和类锁的意义和区别.md)

#### 线程同步
[](https://github.com/helen-x/AndroidInterview/blob/master/java/%5BJava%5D%20线程同步的方法：sychronized、lock、reentrantLock分析.md)

### 生产者消费者

## IO

## 异常
异常是程序执行期间打乱了正常指令流的事件。

### 运行时异常
继承于RuntimeException，运行时才会抛出

### 编译时异常
继承于Exception又不属于RuntimeException，必须在编译时捕获，否则无法编译通过。

### try catch finally执行顺序
先执行`try`，代码发生异常执行`catch`，最后一定会执行finally。
try{}里面有一个return语句，紧跟在try后的finally{}里的code会在return前执行。

[Excption与Error包结构,OOM和SOF](https://github.com/helen-x/AndroidInterview/blob/master/java/%5BJava%5D%20Excption与Error包结构%2COOM和SOF.md)

## 面向对象
### 封装，继承，多态
[wiki](https://zh.wikipedia.org/wiki/%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E7%A8%8B%E5%BA%8F%E8%AE%BE%E8%AE%A1)

#### 封装
将抽象性函数接口的实现细节部分包装、隐藏起来的方法,防止外界调用端，去访问对象内部实现细节的手段。  
让代码更容易理解与维护，也加强了代码的安全性。[Wiki](https://zh.wikipedia.org/wiki/%E5%B0%81%E8%A3%9D_(%E7%89%A9%E4%BB%B6%E5%B0%8E%E5%90%91%E7%A8%8B%E5%BC%8F%E8%A8%AD%E8%A8%88))  

#### 继承
计算机程序运行时，相同的消息可能会送给多个不同的类别之对象，而系统可依据对象所属类别，引发对应类别的方法，而有不同的行为。简单来说，所谓多态意指相同的消息给予不同的对象会引发不同的动作称之。[Wiki](https://zh.wikipedia.org/wiki/%E5%A4%9A%E5%9E%8B_(%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%A7%91%E5%AD%A6))

#### 多态
动态多态：通过类(接口)继承机制和虚函数机制生效于运行时。  
静态多态：函数重载，运算符重载，参数化多态(编译期) 

#### 继承（Inheritance）与聚合（Aggregation）的区别在哪里
继承(is a)是从上到下，聚合(has a)是部分与整体

内聚，耦合：[wiki](https://zh.wikipedia.org/wiki/%E5%85%A7%E8%81%9A%E6%80%A7_(%E8%A8%88%E7%AE%97%E6%A9%9F%E7%A7%91%E5%AD%B8))

#### 父类的静态方法能否被子类重写
不能

1. 静态方法是编译时就决定的。
2. Override依赖于类的实例

#### final 与 static 关键字可以用于哪里？它们的作用是什么？
[final：](https://en.wikipedia.org/wiki/Final_(Java))可以修饰class，method，variable。final不能修饰构造函数，父类的private成员方法不能被子类覆盖，所以是final的。

1. final class无法被继承，可以提供安全性和效率
2. final method无法被子类重写或隐藏，可以避免，子类带来不期望的行为
3. final variable只能被初始化一次，不一定要在声明时初始化。如果声明的是一个引用(`reference`)，意味着无法再引用其他对象，但是被引用的对象如果是mutable，那么依然是mutable
4. final 参数

##### final variable
没有在声明时初始化的final variable 称为 "blank final" variable

1. 在每个构造函数中初始化
2. 如果还有static修饰，则在static initializer中初始化

##### anonymous inner class
匿名内部类中访问outer class的变量要求是final的。

static：

1. 修饰变量，方法，static代码块
2. static修饰的会在类加载时存放到方法区
3. 不依赖于实例，也就不能使用this和super关键字
4. 静态代码块只在类加载到内存时执行一次，如果有多个，则按照在类中的出现顺序执行
5. 只能在内部类中定义静态类，如果内部类不是静态内部类，则变量和方法也不能用static修饰

#### 静态代码块，构造代码块，构造函数以及JAVA类初始化顺序
[链接](http://www.cnblogs.com/Qian123/p/5713440.html)
>静态代码块：用staitc声明，jvm加载类时执行，仅执行一次  
构造代码块：类中直接用{}定义，每一次创建对象时执行。  
执行顺序优先级：静态块,main(),构造块,构造方法。

```
class Parent {
        /* 静态变量 */
    public static String p_StaticField = "父类--静态变量";
         /* 变量 */
    public String    p_Field = "父类--变量";
    protected int    i    = 9;
    protected int    j    = 0;
        /* 静态初始化块 */
    static {
        System.out.println( p_StaticField );
        System.out.println( "父类--静态初始化块" );
    }
        /* 初始化块 */
    {
        System.out.println( p_Field );
        System.out.println( "父类--初始化块" );
    }
        /* 构造器 */
    public Parent()
    {
        System.out.println( "父类--构造器" );
        System.out.println( "i=" + i + ", j=" + j );
        j = 20;
    }
}

public class SubClass extends Parent {
         /* 静态变量 */
    public static String s_StaticField = "子类--静态变量";
         /* 变量 */
    public String s_Field = "子类--变量";
        /* 静态初始化块 */
    static {
        System.out.println( s_StaticField );
        System.out.println( "子类--静态初始化块" );
    }
       /* 初始化块 */
    {
        System.out.println( s_Field );
        System.out.println( "子类--初始化块" );
    }
       /* 构造器 */
    public SubClass()
    {
        System.out.println( "子类--构造器" );
        System.out.println( "i=" + i + ",j=" + j );
    }


        /* 程序入口 */
    public static void main( String[] args )
    {
        System.out.println( "子类main方法" );
        new SubClass();
    }
}
```
结果：  
父类--静态变量  
父类--静态初始化块  
子类--静态变量  
子类--静态初始化块  
子类main方法  
父类--变量  
父类--初始化块  
父类--构造器  
i=9, j=0  
子类--变量  
子类--初始化块  
子类--构造器  
i=9,j=20  

## 抽象类&接口
### 抽象类
1. 由`abstract`关键字定义，不一定包含`abstract`方法
2. 无法实例化，但可以是其他类的子类
3. 可以有构造函数
4. 定义抽象概念，对共通的方法和属性进行规约

苹果和橘子都是水果，但是水果是个概念，不会有实例。

### 接口
1. 接口是抽象类型，由interface修饰，无法实例化
2. 用来指定`class`必须实现的方法，使不同类的对象可以利用相同的interface进行沟通
3. 用来模拟多继承
4. 接口无法实现另一个接口

接口的意义：规范，扩展，回调

#### 比较抽象类和接口，何时使用哪一个？
1. 抽象类可以对共通的方法和属性提供一个实现，而接口不能
2. 接口的属性是`public static final`的，而抽象类没有限制
3. 接口的所有方法都是`public`的，而抽象类则没有限制
4. 一个类只能实现一个`class`，但是可以实现任意多接口

使用抽象类：

1. 在密切相关的类间共享代码
2. 子类继承抽象类有许多公共属性和方法或者需要非`public`修饰符
3. 需要声明非`static`，非`final`的属性

使用接口：

1. 不相关的类实现接口
2. 指定特定数据类型的行为，但是不关心实现
3. 利用多继承

## 嵌套类
[Doc](https://docs.oracle.com/javase/tutorial/java/javaOO/nested.html)

```
class OuterClass {
    ...
    //静态嵌套类
    static class StaticNestedClass {
        ...
    }
    //内部类
    class InnerClass {
        ...
    }
}
```
嵌套类分为静态嵌套类(Nested classes that are declared static are called static nested classes)和内部类(Non-static nested classes are called inner classes)。嵌套类是外部类的成员。内部类可以访问外部类的成员，即使是private修饰的，静态嵌套类则不能。嵌套类作为类的成员，可以被`public`，`protectd`，`private`，`package private`修饰，而外部类则只能是`public`或者`package private`。

### 何时使用嵌套类
1. 是一种组合只在一个地方使用的`class`的逻辑方式
2. 增加了封装性
3. 增加代码的可读性和可维护性

内部类因为与外部类实例相关联，所以不能定义`static`成员。

1. 内部类可以用多个实例，每个实例都有自己的状态信息，并且与其他外围对象的信息相互独立
2. 在单个外部类中，可以让多个内部类以不同的方式实现同一个接口，或者继承同一个类
3. 创建内部类对象的时刻并不依赖于外部类对象的创建
4. 内部类并没有“is-a”关系，他就是一个独立的实体

## 访问描述符
top-level：`public`，`package-private`  
member-level：`public`，`protected`，`private`，`package-private`

## 重写，重载
都是Java多态性的表现

`Override`

1. 子类对父类方法的实现进行重新编写，返回值，参数都不变，修改实现。
2. 不能抛出新的检查异常或者比被重写方法申明更加宽泛的异常
3. 访问权限不能比父类中被重写的方法的访问权限更低
4. 子类和父类在同一个包中，那么子类可以重写父类所有方法，除了声明为private和final的方法
5. 子类和父类不在同一个包中，那么子类只能够重写父类的声明为public和protected的非final方法

`Overload`  
同一个类中，相同方法名但是参数不同。返回类型可以相同也可以不同，比如构造方法


# JVM
## 四种引用类型
1. 强引用-StrongReference
2. 软引用-SoftReference
3. 弱引用-WeakReference
4. 虚引用-PhantomReference

## 垃圾回收
## 类加载


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
* Keep it simple, stupid
* Don't repeat yourself
* You ain't gonna need it

#### 什么是设计模式（Design Patterns）？你知道哪些设计模式？
[设计模式](https://zh.wikipedia.org/wiki/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F_(%E8%AE%A1%E7%AE%97%E6%9C%BA))是对软件设计中普遍存在（反复出现）的各种问题，所提出的解决方案。是描述在各种不同情况下，要怎么解决问题的一种方案。  

1. 单例模式
2. Builder模式
3. 原型模式
4. 工厂方法模式
5. 抽象工厂模式
6. 策略模式
7. 状态模式
8. 责任链模式
9. 解释器模式
10. 命令模式
11. 观察者模式
12. 备忘录模式
13. 迭代器模式
14. 模板方法模式
15. 访问者模式
16. 中介者模式
17. 代理模式
18. 组合模式
19. 适配器模式
20. 装饰模式
21. 享元模式
22. 外观模式
23. 桥接模式

#### 你有了解过存在哪些反模式（Anti-Patterns）吗？
一个[反面模式](https://zh.wikipedia.org/wiki/%E5%8F%8D%E9%9D%A2%E6%A8%A1%E5%BC%8F)(anti-pattern或antipattern)指的是在实践中明显出现但又低效或是有待优化的设计模式，是用来解决问题的带有共同性的不良方法。它们已经经过研究并分类，以防止日后重蹈覆辙，并能在研发尚未投产的系统时辨认出来。

#### 尝试编写如下代码
1. 计算指定数字的阶乘
2. 开发 Fizz Buzz 小游戏
3. 倒转句子中的单词
4. 回文字符串检测
5. 枚举给定字符串的所有排列组合

## 算法
#### 你知道哪些基本的排序算法，它们的计算复杂度如何？在给定数据的情况下你会倾向于使用哪种算法呢？
[wiki](https://zh.wikipedia.org/wiki/%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95)  
冒泡排序  
插入排序  
归并排序  
堆排序  
快速排序


### 参考
[https://howtotrainyourjava.com/2016/07/14/java-developer-interview-questions-the-hard-part/](https://howtotrainyourjava.com/2016/07/14/java-developer-interview-questions-the-hard-part/)  
[https://github.com/JackyAndroid/AndroidInterview-Q-A/blob/master/README-CN.md](https://github.com/JackyAndroid/AndroidInterview-Q-A/blob/master/README-CN.md)  
[https://ydmmocoo.github.io/2016/06/22/Android面试题整理/](https://ydmmocoo.github.io/2016/06/22/Android面试题整理/)  
[Java面试题集](http://blog.csdn.net/dd864140130/article/details/55833087)  
[Android面试题集](http://blog.csdn.net/dd864140130/article/details/57408502)  
[Android 名企面试题及涉及知识点整理](https://github.com/helen-x/AndroidInterview?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)