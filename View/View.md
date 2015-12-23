### ViewRootImpl

The top of a view hierarchy, implementing the needed protocol between View and the WindowManager.  This is for the most part an internal implementation detail of {@link WindowManagerGlobal}.  

view的绘制流程从ViewRoot的performTraversals方法开始:
![ViewRootImpl_performTraversals](https://github.com/DroidWorkerLYF/LearnX/blob/master/View/ViewRootImpl_performTraversals.jpg?raw=true)

Measure完成后,getMeasuredWidth,getMeasuredHeight获取测量后的宽高,基本就是最终的宽高
Layout完成后,可以get到顶点位置,并且getWidth和getHeight可以拿到了  

###MeasureSpec

MeasureSpec:32为int值,高2位代表SpecMode,低30位代表SpecSize  
`UNSPECIFIED`:父容器无限制,要多就多大  
`EXACTLY`:父容器测量出View所需的精确大小,对应于`math_parent`  
`AT_MOST`:父容器指定了可用大小,View不能超过这个值,对应`wrap_content`  

`DecorView`由窗口尺寸和自身`LayoutParams`决定MeasureSpec,`View`由父容器和自身`LayoutParams`决定  

#DecorView确定MeasureSpec#
	private boolean measureHierarchy(final View host, final WindowManager.LayoutParams lp,
            final Resources res, final int desiredWindowWidth, final int desiredWindowHeight) {
		boolean goodMeasure = false;
        if (lp.width == ViewGroup.LayoutParams.WRAP_CONTENT) {
        	//math_parent不执行这里,dialog会走这里
        }

      	if (!goodMeasure) {
            childWidthMeasureSpec = getRootMeasureSpec(desiredWindowWidth, lp.width);
            childHeightMeasureSpec = getRootMeasureSpec(desiredWindowHeight, lp.height);
            performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);
            if (mWidth != host.getMeasuredWidth() || mHeight != host.getMeasuredHeight()) {
                windowSizeMayChange = true;
            }
        }
	}

	/**
     * Figures out the measure spec for the root view in a window based on it's
     * layout params.
     *
     * @param windowSize
     *            The available width or height of the window
     *
     * @param rootDimension
     *            The layout params for one dimension (width or height) of the
     *            window.
     *
     * @return The measure spec to use to measure the root view.
     */
    private static int getRootMeasureSpec(int windowSize, int rootDimension) {
        int measureSpec;
        switch (rootDimension) {

        case ViewGroup.LayoutParams.MATCH_PARENT:
            // Window can't resize. Force root view to be windowSize.
            measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.EXACTLY);
            break;
        case ViewGroup.LayoutParams.WRAP_CONTENT:
            // Window can resize. Set max size for root view.
            measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.AT_MOST);
            break;
        default:
            // Window wants to be an exact size. Force root view to be that size.
            measureSpec = MeasureSpec.makeMeasureSpec(rootDimension, MeasureSpec.EXACTLY);
            break;
        }
        return measureSpec;
    }

#普通View确定MeasureSpec#
