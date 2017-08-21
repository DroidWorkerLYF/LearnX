# How your layout shown in activity

通常我们调用`setContentView(R.layout.yout_layout)`页面内容就被添加到了`Activity`并显示出来，那么这个过程到底发生了什么？

## setContentView
```
protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
}
```
一段很常见的代码，每个`Activity`都需要`setContentView`来设置显示的布局。让我们跟进去一点点看看。

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

发现其实调用的是`Window`中的方法。我们已经知道	`Window`的实现类是`PhoneWindow`，那么继续深入。

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
实际最终调用到了这里，