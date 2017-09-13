# How your layout shown in activity

通常我们调用`setContentView(R.layout.yout_layout)`页面内容就被添加到了`Activity`并显示出来，那么这个过程到底发生了什么？

## setContentView
```
protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
}
```
一段很常见的代码。让我们跟进去看一看。

`setContentView`有三个重载的方法。

```
public void setContentView(@LayoutRes int layoutResID)
public void setContentView(View view)
public void setContentView(View view, ViewGroup.LayoutParams params)
```

以第一个为例

```
public void setContentView(@LayoutRes int layoutResID) {
        getWindow().setContentView(layoutResID);
        initWindowDecorActionBar();
    }
    
   /**
     * Retrieve the current {@link android.view.Window} for the activity.
     * This can be used to directly access parts of the Window API that
     * are not available through Activity/Screen.
     *
     * @return Window The current window, or null if the activity is not
     *         visual.
     */
    public Window getWindow() {
        return mWindow;
}
```

发现其实调用的是`Window`中的`setContentView(int layoutResID)`方法。查看`Activity`中代码可知，在`attach`方法中`mWindow = new PhoneWindow(this)`，那么继续看`PhoneWindow`的`setContentView`方法。

```
public void setContentView(int layoutResID) {
        // Note: FEATURE_CONTENT_TRANSITIONS may be set in the process of installing the window
        // decor, when theme attributes and the like are crystalized. Do not check the feature
        // before this happens.
        if (mContentParent == null) {
            installDecor();
        } else if (!hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
            mContentParent.removeAllViews();
        }

        if (hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
            final Scene newScene = Scene.getSceneForLayout(mContentParent, layoutResID,
                    getContext());
            transitionTo(newScene);
        } else {
            mLayoutInflater.inflate(layoutResID, mContentParent);
        }
        mContentParent.requestApplyInsets();
        final Callback cb = getCallback();
        if (cb != null && !isDestroyed()) {
            cb.onContentChanged();
        }
        mContentParentExplicitlySet = true;
    }
```
那么`mContentParent`是什么？
  
```
// This is the view in which the window contents are placed. 
// It is either mDecor itself, or a child of mDecor where the contents go.
private ViewGroup mContentParent;
```

根据源码的注释，我们得知，我们的内容就是添加到这个View中，他可能是`Decor`或者`Decor`的child。

显然这里就是先install一个Decor。

### installDecor
```
// This is the top-level view of the window, containing the window decor.
private DecorView mDecor;

private void installDecor(){
	if (mDecor == null) {
		mDecor = generateDecor();
		......
	}
	if (mContentParent == null) {
		mContentParent = generateLayout(mDecor);
		......
	}
}
```
首先会创建`Decor`，实际上就是`new DecorView(getContext(), -1)`。接下来，我们就要创建上文提到的`mContentParent`了。

#### generateLayout(DecorView decor)
```
protected ViewGroup generateLayout(DecorView decor) {
	TypedArray a = getWindowStyle();
	// 根据TypedArray中的值setFlags，requestFeature.这也是我们自己在代码中requestFeature时，为什么要放在
	// setContentView前面。
	// The rest are only done if this window is not embedded; otherwise,
    // the values are inherited from our container.
    if (getContainer() == null) {
    	......
    }
    int layoutResource;
    int features = getLocalFeatures();
    
    if ((features & (1 << FEATURE_SWIPE_TO_DISMISS)) != 0) {
        layoutResource = R.layout.screen_swipe_dismiss;
    } else if ......
    // 一堆if-else来给layoutResources赋值。
    // 这里会设置一个mChanging变量为true，在Decor中的方法drawableChanged()中会依赖这个值
    mDecor.startChanging();
    
    // 最终inflate确定的layout
    View in = mLayoutInflater.inflate(layoutResource, null);
    decor.addView(in, new ViewGroup.LayoutParams(MATCH_PARENT, MATCH_PARENT));
    // view in被添加到decor中并且赋值给mContentRoot
    mContentRoot = (ViewGroup) in;

	// ID_ANDROID_CONTENT:The ID that the main layout in the XML layout file should have.
   ViewGroup contentParent = (ViewGroup)findViewById(ID_ANDROID_CONTENT);
   if (contentParent == null) {
       throw new RuntimeException("Window couldn't find content container view");
   }

   if ((features & (1 << FEATURE_INDETERMINATE_PROGRESS)) != 0) {
       ProgressBar progress = getCircularProgressBar(false);
       if (progress != null) {
           progress.setIndeterminate(true);
       }
   }

   if ((features & (1 << FEATURE_SWIPE_TO_DISMISS)) != 0) {
       registerSwipeCallbacks();
   }

   // Remaining setup -- of background and title -- that only applies to top-level windows.
   // getContainer()定义在Window中，如果是top-level则返回null，显然我们这里就是top-level
   if (getContainer() == null) {
       final Drawable background;
       if (mBackgroundResource != 0) {
           background = getContext().getDrawable(mBackgroundResource);
       } else {
           background = mBackgroundDrawable;
       }
       mDecor.setWindowBackground(background);

       final Drawable frame;
       if (mFrameResource != 0) {
           frame = getContext().getDrawable(mFrameResource);
       } else {
           frame = null;
       }
       // 这里其实是设置foreground，因为前面调用了Decor的startChanging()，所以不会出发drawableChanged()
       mDecor.setWindowFrame(frame);

       mDecor.setElevation(mElevation);
       mDecor.setClipToOutline(mClipToOutline);

       if (mTitle != null) {
           setTitle(mTitle);
       }

       if (mTitleColor == 0) {
           mTitleColor = mTextColor;
       }
       setTitleColor(mTitleColor);
   }
	// 将Decor中的mChanging变量置为false，并且出发drawableChanged();
   mDecor.finishChanging();

   return contentParent;
}
```
到了这里就可以知道mContentParent其实就是我们熟悉的那个android布局中的android:id/content。当没有title bar时，那么其实就呼应了前文注释中说的`mContentParent`是Decor本身，有title bar时则是child。

#### drawableChanged()
前面返回contentParent之前，最终是执行了`PhoneWindow`中的`drawableChanged()`。

### PhoneWindow#setContentView
```
public void setContentView(int layoutResID) {
       // install decor

        if (hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
            final Scene newScene = Scene.getSceneForLayout(mContentParent, layoutResID,
                    getContext());
            transitionTo(newScene);
        } else {
            mLayoutInflater.inflate(layoutResID, mContentParent);
        }
        mContentParent.requestApplyInsets();
        final Callback cb = getCallback();
        if (cb != null && !isDestroyed()) {
            cb.onContentChanged();
        }
        mContentParentExplicitlySet = true;
    }
```
再回到前面的`setContentView`方法，decor install之后，判断如果没有设定转场动画，则直接将我们传入的布局id，inflate到`mContentParent`，这也是为什么叫做`setCotentView`。

### Activity#initWindowDecorActionBar
前面已经把layout加到了布局中，接下来注释也说的比较清楚了。

```
	/**
     * Creates a new ActionBar, locates the inflated ActionBarView,
     * initializes the ActionBar with the view, and sets mActionBar.
     */
    private void initWindowDecorActionBar() {
        Window window = getWindow();

        // Initializing the window decor can change window feature flags.
        // Make sure that we have the correct set before performing the test below.
        window.getDecorView();

        if (isChild() || !window.hasFeature(Window.FEATURE_ACTION_BAR) || mActionBar != null) {
            return;
        }

        mActionBar = new WindowDecorActionBar(this);
        mActionBar.setDefaultDisplayHomeAsUpEnabled(mEnableDefaultActionBarUp);

        mWindow.setDefaultIcon(mActivityInfo.getIconResource());
        mWindow.setDefaultLogo(mActivityInfo.getLogoResource());
    }
```

## 总结
那么问题来了，我们只是分析了自己的布局是如何被添加到Decor中的，那么Decor又是如何被添加到Window中的呢，有了整个的布局，又是如何展现出来的。

##### 参考文章
[【Android View源码分析（一）】setContentView加载视图机制深度分析](http://blog.csdn.net/qq_23191031/article/details/77172090?locationNum=4&fps=1)

[Android View源码解读：浅谈DecorView与ViewRootImpl](http://www.jianshu.com/p/687010ccad66)