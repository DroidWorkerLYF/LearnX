# RecyclerView
既然是个`View`，那就免不了`onMeasure`，`onLayout`，`onDraw`。

## onMeasure
```
	protected void onMeasure(int widthSpec, int heightSpec) {
        if (mAdapterUpdateDuringMeasure) {
            eatRequestLayout();
            processAdapterUpdatesAndSetAnimationFlags();

            if (mState.mRunPredictiveAnimations) {
                // TODO: try to provide a better approach.
                // When RV decides to run predictive animations, we need to measure in pre-layout
                // state so that pre-layout pass results in correct layout.
                // On the other hand, this will prevent the layout manager from resizing properly.
                mState.mInPreLayout = true;
            } else {
                // consume remaining updates to provide a consistent state with the layout pass.
                mAdapterHelper.consumeUpdatesInOnePass();
                mState.mInPreLayout = false;
            }
            mAdapterUpdateDuringMeasure = false;
            resumeRequestLayout(false);
        }

        if (mAdapter != null) {
            mState.mItemCount = mAdapter.getItemCount();
        } else {
            mState.mItemCount = 0;
        }
        if (mLayout == null) {
            defaultOnMeasure(widthSpec, heightSpec);
        } else {
            mLayout.onMeasure(mRecycler, mState, widthSpec, heightSpec);
        }

        mState.mInPreLayout = false; // clear
    }
```
`mAdapterUpdateDuringMeasure`显然是和adpater更新时有关的。

在`RecyclerView`的内部类`RecyclerViewDataObserver`中

```
void triggerUpdateProcessor() {
            if (mPostUpdatesOnAnimation && mHasFixedSize && mIsAttached) {
                ViewCompat.postOnAnimation(RecyclerView.this, mUpdateChildViewsRunnable);
            } else {
                mAdapterUpdateDuringMeasure = true;
                requestLayout();
            }
        }
```
我们调用的各种onItemInsert/onItemRemove最终会触发此方法，这里会将`mAdapterUpdateDuringMeasure`设置为true。  
接下来是从adapter中获取item的count并赋值给mState的mItemCount。mState是`State`(RecyclerView的静态内部类)的实例。  

### State
`State`中包含了`RecyclerView`的很多有用信息，可以在各个组件中传递`State`对象，根据需要获取相应的信息。而且可以通过resource id作为key，存储任意数据。

再往下如果`mLayout`为`null`，那么会调用默认的measure方法否则调用`mLayout.onMeasure`，`mLayout`是一个`LayoutManager`对象。
### LayoutManager
`LayoutManager`负责测量和定位item views，并且决定何时回收不再对用户可见的item views。使用不同的`LayoutManager`，`RecyclerView`就可以实现不用的展示效果。源码中也默认为我们提供了一些实现。

## onLayout
```
	protected void onLayout(boolean changed, int l, int t, int r, int b) {
        eatRequestLayout();
        TraceCompat.beginSection(TRACE_ON_LAYOUT_TAG);
        dispatchLayout();
        TraceCompat.endSection();
        resumeRequestLayout(false);
        mFirstLayoutComplete = true;
    }
```

### dispatchLayout
`dispatchLayout`就是对`layoutChildren`方法的包装，来处理layout导致的变化动画。源码中假设动画类型有五种：

* PERSISTENT: item在layout前后都是可见的
* REMOVED: 移除，item在layout前可见
* ADDED: 增加，item在layout前不存在
* DISAPPEARING: 消失，item仍然存在于数据集中，由于其他change的发生，在layout过程中，移除了屏幕，不可见了
* APPEARING: 出现，item存在于数据集中，由于其他change的发生，在layout过程中，出现在了屏幕内，可见了

这个方法会指出哪些item在layout前/后存在并推断出每个item属于上面五种状态的哪一种，然后设置动画。
