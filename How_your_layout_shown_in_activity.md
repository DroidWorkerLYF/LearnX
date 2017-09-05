# How your layout shown in activity

通常我们调用`setContentView(R.layout.yout_layout)`页面内容就被添加到了`Activity`并显示出来，那么这个过程到底发生了什么？

## setContentView
```
protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
}
```
一段很常见的代码。让我们跟进去一点点看看。

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
```

发现其实调用的是`Window`中的`setContentView(int layoutResID)`方法。我们已经知道`Window`的实现类是`PhoneWindow`，那么继续深入。

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

显然这里就是判断当没有Decor view的时候，我们需要先install一个。

### installDecor
```
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
	// 根据TypedArray中的值setFlags，requestFeature.
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
到了这里就可以知道mContentParent其实就是我们熟悉的那个android布局中的content。

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