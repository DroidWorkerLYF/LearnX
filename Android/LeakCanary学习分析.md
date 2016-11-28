## LeakCanary

A memory leak detection library for Android and Java.

## 配置使用
在`build.gradle`中配置

```
 dependencies {
   debugCompile 'com.squareup.leakcanary:leakcanary-android:1.5'
   releaseCompile 'com.squareup.leakcanary:leakcanary-android-no-op:1.5'
   testCompile 'com.squareup.leakcanary:leakcanary-android-no-op:1.5'
 }
```

在`Application`中

```
public class ExampleApplication extends Application {

  @Override public void onCreate() {
    super.onCreate();
    if (LeakCanary.isInAnalyzerProcess(this)) {
      // This process is dedicated to LeakCanary for heap analysis.
      // You should not init your app in this process.
      return;
    }
    LeakCanary.install(this);
    // Normal app init code...
  }
}
```

至此，`Activity`的内存泄露就会被检测

## 工作流程

`LeakCanary.install(this).watch(this, "application");`   
`LeakCancary`的`install`方法会返回一个`RefWatcher`对象，该对象对外提供了

1. `public void watch(Object watchedReference)`
2. `public void watch(Object watchedReference, String referenceName)`

