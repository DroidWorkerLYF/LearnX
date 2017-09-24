# Activity

![Activity继承关系](https://github.com/DroidWorkerLYF/LearnX/blob/master/Android/reading_source_code/res/activity_extends.png?raw=true)

> An activity is a single, focused thing that the user can do. Almost all activities interact with the user, so the Activity class takes care of creating a window for you in which you can place your UI with setContentView(View). While activities are often presented to the user as full-screen windows, they can also be used in other ways: as floating windows (via a theme with windowIsFloating set) or embedded inside of another activity (using ActivityGroup). 

`Activity`负责与用户的交互，所以需要负责创建`Window`来展现我们`setContentView`添加进来的内容，看到这里就很明确了，我们想要解答之前的疑问，就需要了解一下`Activity`这个非常重要的组件。

## startActivity
从哪里下手呢，既然`Activity`要呈现出来，那必然要先由我们启动一个，所以就从`startActivity`入手。


```
@Override
    public void startActivity(Intent intent, @Nullable Bundle options) {
        if (options != null) {
            startActivityForResult(intent, -1, options);
        } else {
            // Note we want to go through this call for compatibility with
            // applications that may have overridden the method.
            startActivityForResult(intent, -1);
        }
    }
```
这是`Activity`中的实现，直接Override父类的方法，`startActivity`会启动一个新的`Activity`，相比`Context.startActivity`方法，这个方法不需要`Intent#FLAG_ACTIVITY_NEW_TASK`，我们肯定都不陌生在非`Activity`中启动`Activity`时，都需要这个参数，因为没有task可以存放新的`Activity`，而在`Activity`中，在没有特别指定的情况下，新的`Activity`会被添加到调用者所在的task。

```
public void startActivityForResult(Intent intent, int requestCode, @Nullable Bundle options) {
        if (mParent == null) {
            Instrumentation.ActivityResult ar =
                mInstrumentation.execStartActivity(
                    this, mMainThread.getApplicationThread(), mToken, this,
                    intent, requestCode, options);
            if (ar != null) {
                mMainThread.sendActivityResult(
                    mToken, mEmbeddedID, requestCode, ar.getResultCode(),
                    ar.getResultData());
            }
            if (requestCode >= 0) {
                // If this start is requesting a result, we can avoid making
                // the activity visible until the result is received.  Setting
                // this code during onCreate(Bundle savedInstanceState) or onResume() will keep the
                // activity hidden during this time, to avoid flickering.
                // This can only be done when a result is requested because
                // that guarantees we will get information back when the
                // activity is finished, no matter what happens to it.
                mStartedActivity = true;
            }

            cancelInputsAndStartExitTransition(options);
            // TODO Consider clearing/flushing other event sources and events for child windows.
        } else {
            if (options != null) {
                mParent.startActivityFromChild(this, intent, requestCode, options);
            } else {
                // Note we want to go through this method for compatibility with
                // existing applications that may have overridden it.
                mParent.startActivityFromChild(this, intent, requestCode);
            }
        }
    }
```
最终还是调用到了`startActivityForResult`方法，在外部调用时，requestCode为负数，就等同于调用上面的`startActivity`方法，即新的`Activity`不会被作为一个sub-activity来启动。需要返回result时，需要配合`Intent`中指定的协议。比如如果你使用`singleTask`作为lunch mode，那么因为新的`Activity`不会运行在当前的task中，你会立即收到一个cancel结果。  
这里有一个特殊情况，如果你在onCreate/onResume中调用`startActivityForResult`方法，那么当前window会在你拿到了返回结果之后才显示出来，以避免闪烁问题，比如我们进入一个界面发现用户没登录直接跳转登录页并且等待这个页面的结果。就是代码中`requestCode >= 0`这个判断的情况。  
那么回到启动流程上，来到了`Instrumentation`的`execStartActivity`方法。

## Instrumentation
```
/**
 * Base class for implementing application instrumentation code.  When running
 * with instrumentation turned on, this class will be instantiated for you
 * before any of the application code, allowing you to monitor all of the
 * interaction the system has with the application.  An Instrumentation
 * implementation is described to the system through an AndroidManifest.xml's
 * &lt;instrumentation&gt; tag.
 */
public class Instrumentation {
}
```
先来看看什么是`Instrumentation`。

```
public ActivityResult execStartActivity(
            Context who, IBinder contextThread, IBinder token, Activity target,
            Intent intent, int requestCode, Bundle options) {
        IApplicationThread whoThread = (IApplicationThread) contextThread;
        Uri referrer = target != null ? target.onProvideReferrer() : null;
        if (referrer != null) {
            intent.putExtra(Intent.EXTRA_REFERRER, referrer);
        }
        if (mActivityMonitors != null) {
            synchronized (mSync) {
                final int N = mActivityMonitors.size();
                for (int i=0; i<N; i++) {
                    final ActivityMonitor am = mActivityMonitors.get(i);
                    if (am.match(who, null, intent)) {
                        am.mHits++;
                        if (am.isBlocking()) {
                            return requestCode >= 0 ? am.getResult() : null;
                        }
                        break;
                    }
                }
            }
        }
        try {
            intent.migrateExtraStreamToClipData();
            intent.prepareToLeaveProcess();
            int result = ActivityManagerNative.getDefault()
                .startActivity(whoThread, who.getBasePackageName(), intent,
                        intent.resolveTypeIfNeeded(who.getContentResolver()),
                        token, target != null ? target.mEmbeddedID : null,
                        requestCode, 0, null, options);
            checkStartActivityResult(result, intent);
        } catch (RemoteException e) {
            throw new RuntimeException("Failure from system", e);
        }
        return null;
    }
```

