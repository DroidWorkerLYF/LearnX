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
	
##View分发
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
        
FLAG\_DISALLOW\_INTERCEPT由子view的requestDisallowInterceptTouchEvent设置，导致ViewGroup无法拦截ACTION_DOWN以外的事件，唯有ACTION_DOWN时，会resetTouchState(),将会还原FLAG\_DISALLOW\_INTERCEPT，  
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