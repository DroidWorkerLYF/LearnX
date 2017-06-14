# Performance
## Android官方
[Best Practices for Performance](https://developer.android.com/training/best-performance.html)

### Performance Tips
写出高效的代码，有两个基本法则

* Don't do work that you don't need to do
* Don't allocate memory if you can avoid it

### Avoid Creating Unnecessary Objects
避免创建不需要的对象。从而降低垃圾回收的频率，用户体验上的改善非常直观。

* 避免创建大量临时对象，比如String。
* 当从输入流提取String时，考虑使用subString，而不是copy一份。
* 多维数组变为一维，当然设计API，对外时是个例外。

### Prefer Static Over Virtual
如果你的方法不需要访问对象的实例，那么使用`static`，调用会快15%-20%。同时也可以通过方法签名来直观的体现，调用这个方法不会改变对象的状态。

### Use Static Final For Constants
常量要记得使用`static final`修饰。

### Avoid Internal Getters/Setters
> Virtual method calls are expensive, much more so than instance field lookups

方法调用的代价比直接使用实例的变量更高，对外的接口设计，仍然应该使用getts/setters，但是在class内部直接使用变量就好。

没有JIT的情况，直接访问属性比用getter方法快3倍，有JIT，则快7倍。

### Use Enhanced For Loop Syntax
for-each loop，看Effective Java, item 46。

### Consider Package Instead of Private Access with Private Inner Classes
```
public class Foo {
    private class Inner {
        void stuff() {
            Foo.this.doStuff(Foo.this.mValue);
        }
    }

    private int mValue;

    public void run() {
        Inner in = new Inner();
        mValue = 27;
        in.stuff();
    }

    private void doStuff(int value) {
        System.out.println("Value is " + value);
    }
}
```
这种写法虽然能够得到正确的输出，但是`Inner`内部调用了outer class的私有变量和方法，虚拟机认为这种直接访问时非法的，因为`Foo`和`Foo$Inner`是两个不同的Class。但是Java语言允许这么做，所以编译器需要创建相应的方法，来保证可以实现。

```
/*package*/ static int Foo.access$100(Foo foo) {
    return foo.mValue;
}
/*package*/ static void Foo.access$200(Foo foo, int value) {
    foo.doStuff(value);
}
```
前面已经说过使用accessor方法比直接访问属性要慢。所以可以考虑使用package修饰，但问题就是同一个package下的其他class也可以访问到了，所以在对外的API中不要这么做。

### Avoid Using Floating-Point
根据经验，浮点在android设备上比integer慢2倍。  
在速度方面，`float`和`double`没有区别。在空间上，double是2倍。与台式机一样，假设空间不是问题，那么应该优先使用double。  

### Know and Use the Libraries
使用系统library。
Effective Java, item 47。

### Use Native Methods Carefully
>Native code is primarily useful when you have an existing native codebase that you want to port to Android, not for "speeding up" parts of your Android app written with the Java language.  

Effective Java, item 54。

### Performance Myths

### Always Measurexw