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

接下来`mAutoMeasure`，true则由RV处理，false则是`LayoutManager`自己来处理measure。framework为我们提供的LayoutManagers都是mAutoMeasure=true，在RV的`onMeasure`调用时，如果大小是确定的，那么RV会在调用`LayoutManager`的`onMeasure`方法(默认就是defaultOnMeasure)后，直接return。  

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
                // 此方法会处理adapter的update，决定应该使用哪一种动画，保存当前views的信息，
                // 如果需要进行predictive layout，则保存响应信息。
                dispatchLayoutStep1();
            }
            // set dimensions in 2nd step. Pre-layout should happen with old dimensions for
            // consistency
            // 将width和height设置给layout manager但如果mode是MeasureSpec.UNSPECIFIED，并且不被允许，则会设置为0
            // 此时的宽高还是旧数值
            mLayout.setMeasureSpecs(widthSpec, heightSpec);
            mState.mIsMeasuring = true;
            // 此方法中会首先处理好所有的update，然后执行mLayout.onLayoutChildren(mRecycler, mState)
            dispatchLayoutStep2();

            // now we can get the width and height from the children.
            // layout完成，根据child的计算结果和之前的widthSpec，heightSpec来setMeasuredDimension
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

然后就是`mAutoMeasure`为false的情况了。

```
            if (mHasFixedSize) {
                mLayout.onMeasure(mRecycler, mState, widthSpec, heightSpec);
                return;
            }
            // custom onMeasure
            // measure过程中，adapter update了
            if (mAdapterUpdateDuringMeasure) {
                eatRequestLayout();
                processAdapterUpdatesAndSetAnimationFlags();

                if (mState.mRunPredictiveAnimations) {
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
            eatRequestLayout();
            mLayout.onMeasure(mRecycler, mState, widthSpec, heightSpec);
            resumeRequestLayout(false);
            mState.mInPreLayout = false; // clear
```
如果RV的大小是固定的，那么measure后直接return，在知道RV的大小不会受item影响的情况下，RV是可以做一些优化的。不过这种情况下RV的大小仍然可以改变，只要影响因素不依赖于item。其余情况就是调用layout manager自己实现的`onMeasure`了。


## onLayout
`onLayout`方法实际上只是调用了`dispatchLayout`方法。`dispatchLayout`就是对`layoutChildren`方法的包装，来处理layout导致的变化动画。源码中假设动画类型有五种 

* PERSISTENT: item在layout前后都是可见的
* REMOVED: 移除，item在layout前可见
* ADDED: 增加，item在layout前不存在
* DISAPPEARING: 消失，item仍然存在于数据集中，由于其他change的发生，在layout过程中，移除了屏幕，不可见了
* APPEARING: 出现，item存在于数据集中，由于其他change的发生，在layout过程中，出现在了屏幕内，可见了

这个方法会指出哪些item在layout前/后存在并推断出每个item属于上面五种状态的哪一种，然后设置动画。

```
    void dispatchLayout() {
        if (mAdapter == null) {
            Log.e(TAG, "No adapter attached; skipping layout");
            // leave the state in START
            return;
        }
        if (mLayout == null) {
            Log.e(TAG, "No layout manager attached; skipping layout");
            // leave the state in START
            return;
        }
        mState.mIsMeasuring = false;
        if (mState.mLayoutStep == State.STEP_START) {
            dispatchLayoutStep1();
            mLayout.setExactMeasureSpecsFrom(this);
            dispatchLayoutStep2();
        } else if (mAdapterHelper.hasUpdates() || mLayout.getWidth() != getWidth() ||
                mLayout.getHeight() != getHeight()) {
            // First 2 steps are done in onMeasure but looks like we have to run again due to
            // changed size.
            mLayout.setExactMeasureSpecsFrom(this);
            dispatchLayoutStep2();
        } else {
            // always make sure we sync them (to ensure mode is exact)
            mLayout.setExactMeasureSpecsFrom(this);
        }
        dispatchLayoutStep3();
    }
```
如果layout step还是`STEP_START`那么就按顺序去调用step1，step2。如果之前onMeasure已经执行过layout，但是adapter更新了，或者RV的宽高有变化，那么在此执行step2。最后一定会执行step3。step3会负责保存views的动画信息，并且触发动画，并且做一些必要的清理。最后会有一个onLayoutCompleted的回调。

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
 
### Recycle
在layout的step2中会调用`mLayout.onLayoutChildren(mRecycler, mState);`，显然这个`mRecycler`是和复用视图有关的了。

### 参考
[掌握RecyclerView动画不得不看的文章](http://www.jianshu.com/p/3be9a519fe79)  
[RecyclerView Animations Part 1 - How Animations Work](http://www.birbit.com/recyclerview-animations-part-1-how-animations-work/)  
[](https://sanjay-f.github.io/2015/12/23/源码探索系列10---替代Listview的RecycleView/)  
[](http://www.jianshu.com/p/115c7bba2d1e)  
[](http://www.jianshu.com/p/9ddfdffee5d3)
[](http://blog.csdn.net/qq_36523667/article/details/78750805)
[](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2016/0630/4400.html)