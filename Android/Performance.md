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