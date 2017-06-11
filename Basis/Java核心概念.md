# Java核心概念
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

#### volaitle
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