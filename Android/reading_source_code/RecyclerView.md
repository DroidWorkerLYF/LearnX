# RecyclerView
既然是个`View`，那就免不了Measure，Layout，Draw。

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

在RV的内部类`RecyclerViewDataObserver`中

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
`State`中包含了RV的很多有用信息，可以在各个组件中传递`State`对象，根据需要获取相应的信息。而且可以通过resource id作为key，存储任意数据。

再往下如果`mLayout`为`null`，那么会调用默认的measure方法否则调用`mLayout.onMeasure`，`mLayout`是一个`LayoutManager`对象。
### LayoutManager
`LayoutManager`负责测量和定位item views，并且决定何时回收不再对用户可见的item views。使用不同的`LayoutManager`，RV就可以实现不用的展示效果。源码中也默认为我们提供了一些实现。

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
显然`dispatchLayout`是重点了。`dispatchLayout`就是对`layoutChildren`方法的包装，来处理layout导致的变化动画。源码中假设动画类型有五种：

* PERSISTENT: item在layout前后都是可见的
* REMOVED: 移除，item在layout前可见
* ADDED: 增加，item在layout前不存在
* DISAPPEARING: 消失，item仍然存在于数据集中，由于其他change的发生，在layout过程中，移除了屏幕，不可见了
* APPEARING: 出现，item存在于数据集中，由于其他change的发生，在layout过程中，出现在了屏幕内，可见了

这个方法会指出哪些item在layout前/后存在并推断出每个item属于上面五种状态的哪一种，然后设置动画。

## draw

### onDraw
```
	public void onDraw(Canvas c) {
        super.onDraw(c);

        final int count = mItemDecorations.size();
        for (int i = 0; i < count; i++) {
            mItemDecorations.get(i).onDraw(c, this, mState);
        }
    }
```

#### ItemDecoration
这里又出现了一个新的概念，`ItemDecoration`。使用`ItemDecoration`我们可以在绘制和布局时为指定的item视图增加额外的偏移量，所以这可以很方便的绘制分割线，高亮某一项，或者对items进行分组等等。我们添加的`ItemDecoration`会按照添加的顺序依次绘制。在item被绘制之前会调用`ItemDecoration`的`onDraw`方法，在item绘制之后会调用`onDrawOver`方法。所以上面的源码中，RV的`onDraw`方法调用了`ItemDecoration`的`onDraw`。`onDraw`方法绘制的内容位于item视图的下面，`onDrawOver`则位于上面。

### draw
让我们回到RV的`draw`方法。

```
	public void draw(Canvas c) {
        super.draw(c);

        final int count = mItemDecorations.size();
        for (int i = 0; i < count; i++) {
            mItemDecorations.get(i).onDrawOver(c, this, mState);
        }
        // TODO If padding is not 0 and chilChildrenToPadding is false, to draw glows properly, we
        // need find children closest to edges. Not sure if it is worth the effort.
        boolean needsInvalidate = false;
        if (mLeftGlow != null && !mLeftGlow.isFinished()) {
            final int restore = c.save();
            final int padding = mClipToPadding ? getPaddingBottom() : 0;
            c.rotate(270);
            c.translate(-getHeight() + padding, 0);
            needsInvalidate = mLeftGlow != null && mLeftGlow.draw(c);
            c.restoreToCount(restore);
        }
        // 省略Top，Right，Bottom绘制代码

        // If some views are animating, ItemDecorators are likely to move/change with them.
        // Invalidate RecyclerView to re-draw decorators. This is still efficient because children's
        // display lists are not invalidated.
        if (!needsInvalidate && mItemAnimator != null && mItemDecorations.size() > 0 &&
                mItemAnimator.isRunning()) {
            needsInvalidate = true;
        }

        if (needsInvalidate) {
            ViewCompat.postInvalidateOnAnimation(this);
        }
    }
```

正如前面提到的，这里一上来就调用了`ItemDecoration`的`onDrawOver`方法。接下来就是绘制Edge effect(旋转移动canvas
然后绘制)。在接下来，则是处理一下看是否需要invalidate。
