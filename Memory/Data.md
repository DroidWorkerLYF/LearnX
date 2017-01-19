# 资料截取
## Java 内存分配策略
Java 程序运行时的内存分配策略有三种,分别是静态分配,栈式分配,和堆式分配，对应的，三种存储策略使用的内存空间主要分别是静态存储区（也称方法区）、栈区和堆区。

1. 静态存储区（方法区）：主要存放静态数据、全局 static 数据和常量。这块内存在程序编译时就已经分配好，并且在程序整个运行期间都存在。
2. 栈区 ：当方法被执行时，方法体内的局部变量都在栈上创建，并在方法执行结束时这些局部变量所持有的内存将会自动被释放。因为栈内存分配运算内置于处理器的指令集中，效率很高，但是分配的内存容量有限。
3. 堆区 ： 又称动态内存分配，通常就是指在程序运行时直接 new 出来的内存。这部分内存在不使用时将会由 Java 垃圾回收器来负责回收。

在方法体内定义的（局部变量）一些基本类型的变量和对象的引用变量都是在方法的栈内存中分配的。当在一段方法块中定义一个变量时，Java 就会在栈中为该变量分配内存空间，当超过该变量的作用域后，该变量也就无效了，分配给它的内存空间也将被释放掉，该内存空间可以被重新使用。

堆内存用来存放所有由 new 创建的对象（包括该对象其中的所有成员变量）和数组。在堆中分配的内存，将由 Java 垃圾回收器来自动管理。在堆中产生了一个数组或者对象后，还可以在栈中定义一个特殊的变量，这个变量的取值等于数组或者对象在堆内存中的首地址，这个特殊的变量就是我们上面说的引用变量。我们可以通过这个引用变量来访问堆中的对象或者数组。

### 堆和栈的优缺点
堆的优势是可以动态分配内存大小，生存期也不必事先告诉编译器，因为它是在运行时动态分配内存的。
缺点就是要在运行时动态分配内存，存取速度较慢； 栈的优势是，存取速度比堆要快，仅次于直接位于 CPU 中的寄存器。另外，栈数据可以共享。但缺点是，存在栈中的数据大小与生存期必须是确定的，缺乏灵活性。

### Java 中数据在内存中是如何存储的  
#### 8种基本数据类型
byte，short，int，long，double，float，boolean，char。  

如 int a = 3；这里的 a 是一个指向 int 类型的引用，指向3这个字面值。这些字面值的数据，由于大小可知，生存期可知(这些字面值定义在某个程序块里面，程序块退出后，字段值就消失了)，出于追求速度的原因，就存在于栈中。

另外，栈有一个很重要的特殊性，就是存在栈中的数据可以共享

```
	int a = 3;
	int b = 3;
```

编译器先处理 int a = 3；首先它会在栈中创建一个变量为 a 的引用，然后查找有没有字面值为3的地址，没找到，就开辟一个存放3这个字面值的地址，然后将 a 指向3的地址。接着处理 int b = 3；在创建完 b 这个引用变量后，由于在栈中已经有3这个字面值，便将 b 直接指向3的地址。这样，就出现了 a 与 b 同时均指向3的情况。 定义完 a 与 b 的值后，再令 a = 4；那么，b 不会等于4，还是等于3。在编译器内部，遇到时，它就会重新搜索栈中是否有4的字面值，如果没有，重新开辟地址存放4的值；如果已经有了，则直接将 a 指向这个地址。因此 a 值的改变不会影响到 b 的值。

基本型别都有对应的包装类：如 int 对应 Integer 类，double 对应 Double 类等，基本类型的定义都是直接在栈中，如果用包装类来创建对象，就和普通对象一样了。例如：int i=0；i 直接存储在栈中。 Integer i（i 此时是对象） = new Integer(5)；这样，i 对象数据存储在堆中，i 的引用存储在栈中，通过栈中的引用来操作对象。

#### 对象
在 Java 中，创建一个对象包括对象的声明和实例化两步

#### String
String 是一个特殊的包装类数据。可以用用以下两种方式创建:
```
	String str = new String("abc");
	String str = "abc";
```
第一种创建方式，和普通对象的的创建过程一样； 第二种创建方式，Java 内部将此语句转化为以下几个步骤： (1) 先定义一个名为 str 的对 String 类的对象引用变量：String str； (2) 在栈中查找有没有存放值为"abc"的地址，如果没有，则开辟一个存放字面值为"abc" 地址，接着创建一个新的 String 类的对象 o，并将 o 的字符串值指向这个地址，而且在栈 这个地址旁边记下这个引用的对象 o。如果已经有了值为"abc"的地址，则查找对象 o，并 回 o 的地址。 (3) 将 str 指向对象 o 的地址。 值得注意的是，一般 String 类中字符串值都是直接存值的。但像 String str = "abc"；这种合下，其字符串值却是保存了一个指向存在栈中数据的引用。

```
	String str1="abc"；
	String str2="abc"；
	System.out.println(s1==s2)；//true
```

注意，这里并不用 str1.equals(str2)；的方式，因为这将比较两个字符串的值是否相等。==号，根据 JDK 的说明，只有在两个引用都指向了同一个对象时才返回真值。而我们在这里要看的是，str1 与 str2 是否都指向了同一个对象。 我们再接着看以下的代码。

```
	String str1= new String("abc")；
	String str2="abc"；
	System.out.println(str1==str2)；//false
```

隔离引用：这种情况中，被回收的对象仍具有引用，这种情况称作隔离岛。若存在这两个实例，他们互相引用，并且这两个对象的所有其他引用都删除，其他任何线程无法访问这两个对象中的任意一个。也可以符合垃圾回收条件。

finalize()方法：java 提供了一种机制，使你能够在对象刚要被垃圾回收之前运行一些代码。这段代码位于名为 finalize()的方法内，所有类从 Object 类继承这个方法。由于不能保证垃圾回收器会删除某个对象。因此放在 finalize()中的代码无法保证运行。因此建议不要重写 finalize();

final 只对引用的"值"(也即它所指向的那个对象的内存地址)有效，它迫使引用只能指向初始指向的那个对象，改变它的指向会导致编译期错误。至于它所指向的对象的变化，final 是不负责的。这很类似==操作符：==操作符只负责引用的"值"相等，至于这个地址所指向的对象内容是否相等，==操作符是不管的

如何把程序写得更健壮： 1、尽早释放无用对象的引用。 好的办法是使用临时变量的时候，让引用变量在退出活动域后，自动设置为 null，暗示垃圾收集器来收集该对象，防止发生内存泄露。对于仍然有指针指向的实例，jvm 就不会回收该资源,因为垃圾回收会将值为 null 的对象作为垃圾，提高 GC 回收机制效率； 2、定义字符串应该尽量使用 String str="hello"; 的形式 ，避免使用 String str = new String("hello"); 的形式。因为要使用内容相同的字符串，不必每次都 new 一个 String

我们的程序里不可避免大量使用字符串处理，避免使用 String，应大量使用 StringBuffer ，因为 String 被设计成不可变(immutable)类，所以它的所有对象都是不可变对象

尽量避免在类的构造函数里创建、初始化大量的对象 ，防止在调用其自身类的构造器时造成不必要的内存资源浪费，尤其是大对象，JVM 会突然需要大量内存，这时必然会触发 GC 优化系统内存环境；显示的声明数组空间，而且申请数量还极大

6、尽量在合适的场景下使用对象池技术 以提高系统性能，缩减缩减开销，但是要注意对象池的尺寸不宜过大，及时清除无效对象释放内存资源，综合考虑应用运行环境的内存资源限制，避免过高估计运行环境所提供内存资源的数量。 7、大集合对象拥有大数据量的业务对象的时候，可以考虑分块进行处理 ，然后解决一块释放一块的策略。 8、不要在经常调用的方法中创建对象 ，尤其是忌讳在循环中创建对象。可以适当的使用 hashtable，vector 创建一组对象容器，然后从容器中去取那些对象，而不用每次 new 之后又丢弃。 9、一般都是发生在开启大型文件或跟数据库一次拿了太多的数据，造成 Out Of Memory Error 的状况，这时就大概要计算一下数据量的最大值是多少，并且设定所需最小及最大的内存空间值。 10、尽量少用 finalize 函数 ，因为 finalize()会加大 GC 的工作量，而 GC 相当于耗费系统的计算能力。 11、不要过滥使用哈希表 ，有一定开发经验的开发人员经常会使用 hash 表（hash 表在 JDK 中的一个实现就是 HashMap）来缓存一些数据，从而提高系统的运行速度。比如使用 HashMap 缓存一些物料信息、人员信息等基础资料，这在提高系统速度的同时也加大了系统的内存占用，特别是当缓存的资料比较多的时候。其实我们可以使用操作系统中的缓存的概念来解决这个问题，也就是给被缓存的分配一个一定大小的缓存容器，按照一定的算法淘汰不需要继续缓存的对象，这样一方面会因为进行了对象缓存而提高了系统的运行效率，同时由于缓存容器不是无限制扩大，从而也减少了系统的内存占用。现在有很多开源的缓存实现项目，比如 ehcache、oscache 等，这些项目都实现了 FIFO、MRU 等常见的缓存算法

### JVM内存的构
Java虚拟机会将内存分为几个不同的管理区，这些区域各自有各自的用途，根据不同的特点，承担不同的任务以及在垃圾回收时运用不同的算法。总体分为下面几个部分：
程序计数器（Program Counter Register）、JVM虚拟机栈（JVM Stacks）、本地方法栈（Native Method Stacks）、堆（Heap）、方法区（Method Area）

1. 强引用(Strong Reference).就是为刚被new出来的对象所加的引用，它的特点就是，永远不会被回收。
2. 软引用(Soft Reference).声明为软引用的类，是可被回收的对象，如果JVM内存并不紧张，这类对象可以不被回收，如果内存紧张，则会被回收。此处有一个问题，既然被引用为软引用的对象可以回收，为什么不去回收呢？其实我们知道，Java中是存在缓存机制的，就拿字面量缓存来说，有些时候，缓存的对象就是当前可有可无的，只是留在内存中如果还有需要，则不需要重新分配内存即可使用，因此，这些对象即可被引用为软引用，方便使用，提高程序性能。
3. 弱引用(Weak Reference).弱引用的对象就是一定需要进行垃圾回收的，不管内存是否紧张，当进行GC时，标记为弱引用的对象一定会被清理回收。
4. 虚引用(Phantom Reference).虚引用弱的可以忽略不计，JVM完全不会在乎虚引用，其唯一作用就是做一些跟踪记录，辅助finalize函数的使用

### MAT
[http://blog.csdn.net/ljd2038/article/details/53560829?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io](http://blog.csdn.net/ljd2038/article/details/53560829?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)

### 泄漏的定位
1. 静态代码分析工具 —— Lint
2. 严苛模式 —— StrictMode
3. LeakCanary
4. MAT

[http://www.jianshu.com/p/96c55ea3446e?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io](http://www.jianshu.com/p/96c55ea3446e?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)

### GC
引用计数

每个对象有对应的引用计数器
当一个对象被引用（被复制给变量，传入方法中）,引用计数器加1
当一个对象不被引用（离开变量作用域），引用计数器就会减1
基于这种算法的垃圾回收器效率较高
循环引用的问题引用计数算法的垃圾回收器无法解决。
主流的JVM很少使用基于这种算法的垃圾回收器实现。

GC根节点遍历

识别对象为垃圾从被称为GC 根节点出发
每一个被遍历的强引用可到达对象，都会被标记为存活
在遍历结束后，没有被标记为存活的对象都被视为垃圾，需要后续进行回收处理
主流的JVM一般都采用这种算法的垃圾回收器实现

在Java中，可以作为GC 根节点的有

类，由系统类加载器加载的类。这些类从不会被卸载，它们可以通过静态属性的方式持有对象的引用。注意，一般情况下由自定义的类加载器加载的类不能成为GC Roots
线程，存活的线程
Java方法栈中的局部变量或者参数
JNI方法栈中的局部变量或者参数
JNI全局引用
用做同步监控的对象
被JVM持有的对象，这些对象由于特殊的目的不被GC回收。这些对象可能是系统的类加载器，一些重要的异常处理类，一些为处理异常预留的对象，以及一些正在执行类加载的自定义的类加载器。但是具体有哪些前面提到的对象依赖于具体的JVM实现。
提到强引用，有必要系统说一下Java中的引用类型。Java中的引用类型可以分为一下四种：

强引用： 默认的引用类型，例如StringBuffer buffer = new StringBuffer();就是buffer变量持有的为StringBuilder的强引用类型。
软引用：即SoftReference，其指向的对象只有在内存不足的时候进行回收。
弱引用：即WeakReference,其指向的对象在GC执行时会被回收。
虚引用：即PhantomReference,与ReferenceQueue结合，用作记录该引用指向的对象已被销毁。

http://qiangbo.space/2016-12-08/AndroidAnatomy_Process_Recycle/?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io

http://www.jianshu.com/p/079828251f10?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io

http://weishu.me/2016/12/23/dive-into-android-optimize-vm-heap/

http://blog.csdn.net/waitforfree/article/details/38781927

### 参考的文章
[极客学院 Java 内存管理](http://wiki.jikexueyuan.com/project/java-special-topic/platorm-memory.html)  
[Java之美[从菜鸟到高手演变]之JVM内存管理及垃圾回收](http://blog.csdn.net/zhangerqing/article/details/8214365)