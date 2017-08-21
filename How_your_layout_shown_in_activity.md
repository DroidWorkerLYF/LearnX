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

已第一个为例

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

## installDecor
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

### generateLayout(DecorView decor)
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
}
```
