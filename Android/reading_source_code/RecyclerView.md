# RecyclerView

![](https://github.com/DroidWorkerLYF/LearnX/blob/master/Android/reading_source_code/res/recyclerview_extends.png?raw=true)

> A flexible view for providing a limited window into a large data set.

一个在有限窗口内展示大量数据集合的灵活的视图。

在看的过程中，其实对于preLayout，runPredictiveAnimation什么的都是一脸懵逼。看了一些文章，才有所了解。这里要注意view和item的区别。对于一个`ViewGroup`一个view add了，那就可以执行一个fade in动画，表示新增，但是对于一个list，一个item可能只是从不可见到可见了，虽然view是新add进来的，但是还是执行fade in动画就会很奇怪，也就说item和view是有区别的。那么为了对view实施正确的动画，RV的两次layout过程，一次preLayout，一次postLayout，避免LayoutManager过于复杂的同时，还可以处理好所有的change。在preLayout时，是执行动画前的状态，而postLayout则是动画执行后的状态，这样就可以确定对view该执行什么动画了。

## onMeasure
```
    protected void onMeasure(int widthSpec, int heightSpec) {
        if (mLayout == null) {
            defaultOnMeasure(widthSpec, heightSpec);
            return;
        }
        if (mLayout.mAutoMeasure) {
            ......
        } else {
            ......
        }
    }
```
在设置`LayoutManager`之前使用`defaultOnMeasure`。

```
    void defaultOnMeasure(int widthSpec, int heightSpec) {
        // calling LayoutManager here is not pretty but that API is already public and it is better
        // than creating another method since this is internal.
        final int width = LayoutManager.chooseSize(widthSpec,
                getPaddingLeft() + getPaddingRight(),
                ViewCompat.getMinimumWidth(this));
        final int height = LayoutManager.chooseSize(heightSpec,
                getPaddingTop() + getPaddingBottom(),
                ViewCompat.getMinimumHeight(this));

        setMeasuredDimension(width, height);
    }
```
这里就是简单的根据指定的参数返回一个大小。

接下来`mAutoMeasure`，true则由RV处理，false则是`LayoutManager`自己来处理measure。framework为我们提供的LayoutManagers都是mAutoMeasure=true，在RV的`onMeasure`调用时，如果大小是确定的，那么RV会在调用`LayoutManager`的`onMeasure`方法后，直接return。  

```
        if (mLayout.mAutoMeasure) {
            final int widthMode = MeasureSpec.getMode(widthSpec);
            final int heightMode = MeasureSpec.getMode(heightSpec);
            final boolean skipMeasure = widthMode == MeasureSpec.EXACTLY
                    && heightMode == MeasureSpec.EXACTLY;
            mLayout.onMeasure(mRecycler, mState, widthSpec, heightSpec);
            if (skipMeasure || mAdapter == null) {
                return;
            }
            ......
        }
```
但是只要宽高有一个不是EXACTLY，RV就会在`onMeasure`中开始layout，RV会处理adapter的update并决定是否执行predictive layout。如果需要执行，那么，会先进行pre layout(State.isPreLayout=true)，这时候的`getWidth`，`getHeight`都会返回上一次layout完的结果。然后再进行post layout(State.isPreLayout=false && State.isMeasuring=true)，两次layout结束后，RV会set measured dmension。之后的on measure调用，都只会是post layout。measure过程结束，`onLayout`方法被调用，RV会检查自身是否在measure过程中已经完成了layout计算，并复用之前的结果。当然如果在measure和layout之间，adapter update了或者measure的spec和最终的dimension不一致，还是会调用onLayoutChildren。

```
        if (mLayout.mAutoMeasure) {
        	  .......
            if (mState.mLayoutStep == State.STEP_START) {
                dispatchLayoutStep1();
            }
            // set dimensions in 2nd step. Pre-layout should happen with old dimensions for
            // consistency
            mLayout.setMeasureSpecs(widthSpec, heightSpec);
            mState.mIsMeasuring = true;
            dispatchLayoutStep2();

            // now we can get the width and height from the children.
            mLayout.setMeasuredDimensionFromChildren(widthSpec, heightSpec);

            // if RecyclerView has non-exact width and height and if there is at least one child
            // which also has non-exact width & height, we have to re-measure.
            if (mLayout.shouldMeasureTwice()) {
                mLayout.setMeasureSpecs(
                        MeasureSpec.makeMeasureSpec(getMeasuredWidth(), MeasureSpec.EXACTLY),
                        MeasureSpec.makeMeasureSpec(getMeasuredHeight(), MeasureSpec.EXACTLY));
                mState.mIsMeasuring = true;
                dispatchLayoutStep2();
                // now we can get the width and height from the children.
                mLayout.setMeasuredDimensionFromChildren(widthSpec, heightSpec);
            }
        }
```

`mLayoutStep`的初始值就是State.STEP_START，最开始会先执行`dispatchLayoutStep1`。

### dispatchLayoutStep1
```
    private void processAdapterUpdatesAndSetAnimationFlags() {
        // 当notifyDataSetChanged调用时，最终会在RecyclerViewDataObserver中的onChanged方法被设置为true
        if (mDataSetHasChangedAfterLayout) {
            // Processing these items have no value since data set changed unexpectedly.
            // Instead, we just reset it.
            mAdapterHelper.reset();
            markKnownViewsInvalid();
            mLayout.onItemsChanged(this);
        }
        // simple animations are a subset of advanced animations (which will cause a
        // pre-layout step)
        // If layout supports predictive animations, pre-process to decide if we want to run them
        // 如果mItemAnimator不为null，但是supportsPredictiveItemAnimations返回false，则使用simple item animations
        // 也就是简单的faded in/out，反之supportsPredictiveItemAnimations为true，就会触发两次onLayoutChildren来确定
        // 动画从哪里开始出现/消失
        if (predictiveItemAnimationsEnabled()) {
            mAdapterHelper.preProcess();
        } else {
            mAdapterHelper.consumeUpdatesInOnePass();
        }
        boolean animationTypeSupported = mItemsAddedOrRemoved || mItemsChanged;
        mState.mRunSimpleAnimations = mFirstLayoutComplete && mItemAnimator != null &&
                (mDataSetHasChangedAfterLayout || animationTypeSupported ||
                        mLayout.mRequestedSimpleAnimations) &&
                (!mDataSetHasChangedAfterLayout || mAdapter.hasStableIds());
        mState.mRunPredictiveAnimations = mState.mRunSimpleAnimations &&
                animationTypeSupported && !mDataSetHasChangedAfterLayout &&
                predictiveItemAnimationsEnabled();
    }
```









回到`onMeasure`，接下来是从adapter中获取item的count并赋值给mState的mItemCount。mState是`State`(RecyclerView的静态内部类)的实例。  

### State
`State`中包含了RV的很多有用信息，可以在各个组件中传递`State`对象，根据需要获取相应的信息。而且可以通过resource id作为key，存储任意数据。

再往下如果`mLayout`为`null`，那么会调用默认的measure方法否则调用`mLayout.onMeasure`，`mLayout`是一个`LayoutManager`对象。

### LayoutManager
`LayoutManager`负责测量和定位item views，并且决定何时回收不再对用户可见的item views。使用不同的`LayoutManager`，RV就可以实现不用的展示效果。源码中也默认为我们提供了一些实现。

#### LinearLayoutManager
我们来看一下`LinearLayoutManager`是如何实现`onMeasure`的。 
 
```
public void onMeasure(Recycler recycler, State state, int widthSpec, int heightSpec) {
       mRecyclerView.defaultOnMeasure(widthSpec, heightSpec);
}
```
这里是使用宿主RV的实现。Exactly和At most当做一样处理，Unspecified也会使用最小尺寸。

```
	private void defaultOnMeasure(int widthSpec, int heightSpec) {
        final int widthMode = MeasureSpec.getMode(widthSpec);
        final int heightMode = MeasureSpec.getMode(heightSpec);
        final int widthSize = MeasureSpec.getSize(widthSpec);
        final int heightSize = MeasureSpec.getSize(heightSpec);

        int width = 0;
        int height = 0;

        switch (widthMode) {
            case MeasureSpec.EXACTLY:
            case MeasureSpec.AT_MOST:
                width = widthSize;
                break;
            case MeasureSpec.UNSPECIFIED:
            default:
                width = ViewCompat.getMinimumWidth(this);
                break;
        }

        switch (heightMode) {
            case MeasureSpec.EXACTLY:
            case MeasureSpec.AT_MOST:
                height = heightSize;
                break;
            case MeasureSpec.UNSPECIFIED:
            default:
                height = ViewCompat.getMinimumHeight(this);
                break;
        }

        setMeasuredDimension(width, height);
    }
```

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

`dispatchLayout`是个比较长的方法，分为几步。

```
		 if (mAdapter == null) {
            Log.e(TAG, "No adapter attached; skipping layout");
            return;
        }
        if (mLayout == null) {
            Log.e(TAG, "No layout manager attached; skipping layout");
            return;
        }
        mState.mDisappearingViewsInLayoutPass.clear();
        eatRequestLayout();
        onEnterLayoutOrScroll();

        processAdapterUpdatesAndSetAnimationFlags();

        mState.mOldChangedHolders = mState.mRunSimpleAnimations && mItemsChanged
                && supportsChangeAnimations() ? new ArrayMap<Long, ViewHolder>() : null;
        mItemsAddedOrRemoved = mItemsChanged = false;
        ArrayMap<View, Rect> appearingViewInitialBounds = null;
        mState.mInPreLayout = mState.mRunPredictiveAnimations;
        mState.mItemCount = mAdapter.getItemCount();
        findMinMaxChildLayoutPositions(mMinMaxLayoutPositions);
```
首先会吃掉request layout的请求，这里有个`mEatRequestLayout`和`mLayoutFrozen`。

```
	void eatRequestLayout() {
        if (!mEatRequestLayout) {
            mEatRequestLayout = true;
            if (!mLayoutFrozen) {
                mLayoutRequestEaten = false;
            }
        }
    }
    
    @Override
    public void requestLayout() {
        if (!mEatRequestLayout && !mLayoutFrozen) {
            super.requestLayout();
        } else {
            mLayoutRequestEaten = true;
        }
    }
```
`mEatRequestLayout`这个变量为true时就会屏蔽掉任何requestLayout的调用，而`mLayoutFrozen`则可以控制layout和scroll。

```
   public void setLayoutFrozen(boolean frozen) {
        if (frozen != mLayoutFrozen) {
            assertNotInLayoutOrScroll("Do not setLayoutFrozen in layout or scroll");
            if (!frozen) {
                mLayoutFrozen = frozen;
                if (mLayoutRequestEaten && mLayout != null && mAdapter != null) {
                    requestLayout();
                }
                mLayoutRequestEaten = false;
            } else {
                final long now = SystemClock.uptimeMillis();
                MotionEvent cancelEvent = MotionEvent.obtain(now, now,
                        MotionEvent.ACTION_CANCEL, 0.0f, 0.0f, 0);
                onTouchEvent(cancelEvent);
                mLayoutFrozen = frozen;
                mIgnoreMotionEventTillDown = true;
                stopScroll();
            }
        }
    }
```
当`mLayoutFrozen`为true时，request layout也会被屏蔽；所有RV的child不会更新，`smoothScrollBy`，`scrollBy`，`scrollToPosition`，`smoothScrollToPosition`这些滚动方法也不会响应；touch事件不会响应；源码中默认在`setAdapter`和`swapAdapter`时会`setLayoutFrozen(false)`，那么具体是如何stop scroll？

#### stop scroll
```
	public void stopScroll() {
        setScrollState(SCROLL_STATE_IDLE);
        stopScrollersInternal();
    }
```
这里会先将`mScrollState`设置为`SCROLL_STATE_IDLE`，在`setScrollState`方法中，只要状态不是`SCROLL_STATE_SETTLING`，就会调用`stopScrollersInternal`。

```
	private void stopScrollersInternal() {
        mViewFlinger.stop();
        if (mLayout != null) {
            mLayout.stopSmoothScroller();
        }
    }
```
注释中说这个方法和`stopScroll`的区别就是它不设置状态。`mViewFlinger`是个`ViewFlinger`(RV的内部类)对象，实现了`Runnable`接口，内部封装了Scroller，所有滚动相关的操作都是交由它处理的，不断post自身。然后再调用`LayoutManager`的`stopSmoothScroller`。

```
void stopSmoothScroller() {
     if (mSmoothScroller != null) {
         mSmoothScroller.stop();
     }
}
```
`SmoothScroller`也是RV的一个内部类。负责平滑滚动的基类，保存目标视图的位置，提供触发滚动的方法。最终还是由`ViewFlinger`去执行。

再回到`dispatchLayout`方法。`onEnterLayoutOrScroll`会使得mLayoutOrScrollCounter ++，这个变量大于0，`isComputingLayout`就会返回true，也就意味着此时你不能更新adapter，你应该用`Handler`或者其他类似的机制来update。

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


### 参考
[掌握RecyclerView动画不得不看的文章](http://www.jianshu.com/p/3be9a519fe79)  
[RecyclerView Animations Part 1 - How Animations Work](http://www.birbit.com/recyclerview-animations-part-1-how-animations-work/)  
[](https://sanjay-f.github.io/2015/12/23/源码探索系列10---替代Listview的RecycleView/)  
[](http://www.jianshu.com/p/115c7bba2d1e)  
[](http://www.jianshu.com/p/9ddfdffee5d3)