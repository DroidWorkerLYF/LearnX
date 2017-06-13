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

```
static final int intVal = 42;
static final String strVal = "Hello, world!";
```

### Avoid Internal Getters/Setters