#Touch事件分发
	public boolean dispatchTouchEvent(MotionEvent ev)
	public boolean onInterceptTouchEvent(MotionEvent ev)
	public boolean onTouchEvent(MotionEvent ev)
	
	//伪代码的一个关系描述
	public boolean dispathTouchEvent(MotionEvent ev){
		boolean consume = false;
		if(onInterceptTouchEvent(ev)){
			consume = onTouchEvent(ev);
		} else {
			consume = child.dispatchTouchEvent(ev);
		}
		return consume;
	}  
activity -> windows -> view

##Activity分发
	public boolean dispatchTouchEvent(MotionEvent ev) {
        if (ev.getAction() == MotionEvent.ACTION_DOWN) {
            onUserInteraction();
        }
        if (getWindow().superDispatchTouchEvent(ev)) {
            return true;
        }
        return onTouchEvent(ev);
    }
事件交由window分发，如果没有view处理，则调用activity的onTouchEvent  

##Windows分发
	public boolean superDispatchTouchEvent(MotionEvent ev){
		return mDecor.superDispatchTouchEvent(ev);
	}
	
##ViewGroup分发
	// Handle an initial down.
    if (actionMasked == MotionEvent.ACTION_DOWN) {
        // Throw away all previous state when starting a new touch gesture.
        // The framework may have dropped the up or cancel event for the previous gesture
        // due to an app switch, ANR, or some other state change.
       cancelAndClearTouchTargets(ev);
       resetTouchState();
    }
    // Check for interception.
    final boolean intercepted;
    if (actionMasked == MotionEvent.ACTION_DOWN
                    || mFirstTouchTarget != null) {
          final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
          if (!disallowIntercept) {
                intercepted = onInterceptTouchEvent(ev);
                ev.setAction(action); // restore action in case it was changed
          } else {
               intercepted = false;
          }
    } else {
        // There are no touch targets and this action is not an initial down
        // so this view group continues to intercept touches.
        intercepted = true;
        }
        
FLAG\_DISALLOW\_INTERCEPT由子view的requestDisallowInterceptTouchEvent设置，导致ViewGroup无法拦截ACTION\_DOWN以外的事件，唯有ACTION\_DOWN时，会resetTouchState(),将会还原FLAG\_DISALLOW\_INTERCEPT，  
onInterceptTouchEvent也将因此无法保证每次都调用，只有dispatchTouchEvent会保证每次调用。  

	final View[] children = mChildren;
        for (int i = childrenCount - 1; i >= 0; i--) {
            final int childIndex = customOrder
                    ? getChildDrawingOrder(childrenCount, i) : i;
            final View child = (preorderedList == null)
                    ? children[childIndex] : preorderedList.get(childIndex);

            // If there is a view that has accessibility focus we want it
            // to get the event first and if not handled we will perform a
            // normal dispatch. We may do a double iteration but this is
            // safer given the timeframe.
            if (childWithAccessibilityFocus != null) {
                if (childWithAccessibilityFocus != child) {
                    continue;
                }
                childWithAccessibilityFocus = null;
                i = childrenCount - 1;
            }

            if (!canViewReceivePointerEvents(child)
                    || !isTransformedTouchPointInView(x, y, child, null)) {
                ev.setTargetAccessibilityFocus(false);
                continue;
            }

            newTouchTarget = getTouchTarget(child);
            if (newTouchTarget != null) {
                // Child is already receiving touch within its bounds.
                // Give it the new pointer in addition to the ones it is handling.
                newTouchTarget.pointerIdBits |= idBitsToAssign;
                break;
            }

            resetCancelNextUpFlag(child);
            if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
                // Child wants to receive touch within its bounds.
                mLastTouchDownTime = ev.getDownTime();
                if (preorderedList != null) {
                    // childIndex points into presorted list, find original index
                    for (int j = 0; j < childrenCount; j++) {
                        if (children[childIndex] == mChildren[j]) {
                            mLastTouchDownIndex = j;
                            break;
                        }
                    }
                } else {
                    mLastTouchDownIndex = childIndex;
                }
                mLastTouchDownX = ev.getX();
                mLastTouchDownY = ev.getY();
                newTouchTarget = addTouchTarget(child, idBitsToAssign);
                alreadyDispatchedToNewTouchTarget = true;
                break;
            }

            // The accessibility focus didn't handle the event, so clear
            // the flag and do a normal dispatch to all children.
            ev.setTargetAccessibilityFocus(false);
        }
 
dispatchTransformedTouchEvent中，child == null，就会将此viewgroup当成普通view，调用super.dispatchTouchEvent，否则调用child的dispatchTouchEvent。  

##View分发
			if (li != null && li.mOnTouchListener != null
                    && (mViewFlags & ENABLED_MASK) == ENABLED
                    && li.mOnTouchListener.onTouch(this, event)) {
                result = true;
            }

            if (!result && onTouchEvent(event)) {
                result = true;
            }
            
TouchListener的优先级高于onTouchEvent，  

#滑动冲突
1. 内外部方向不一致
2. 内外部方向一致
3. 1和2综合

##解决方法
1. 在父容器中拦截，重写onInterceptTouchEvent，ACTION\_DOWN返回false，ACTION\_MOVE根据需求返回值，ACTION\_UP返回false，ACTION\_UP返回true会导致子view的onClick无法触发，而只要父容器拦截了事件，那么一些列事件都会传递给父容器，包括ACTION\_UP，所以ACTION\_UP可以返回false
2. 在子view中，使用requestDisallowInterceptTouchEvent，并且父容器的onInterceptTouchEvent中ACTION\_DOWN要返回false。