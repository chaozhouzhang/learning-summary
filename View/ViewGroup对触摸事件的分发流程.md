# ViewGroup对触摸事件的分发流程

接着上篇《Activity对触摸事件的分发流程》，这篇我们讲《ViewGroup对触摸事件的分发流程》。


## ViewGroup.dispatchTouchEvent()

首先是Activity中dispatchTouchEvent()方法return super之后将会调用ViewGroup的dispatchTouchEvent()方法：

1、如果onInterceptTouchEvent()方法返回false，将通过buildTouchDispatchChildList()方法获取所有能接收该事件的子视图；而后如果dispatchTransformedTouchEvent()方法返回true，将通过addTouchTarget()生成mFirstTouchTarget对象。

2、如果mFirstTouchTarget为空，说明此时子视图为View，将通dispatchTransformedTouchEvent()继续分发事件；如果mFirstTouchTarget不为空，说明此时子视图为ViewGroup，将遍历ViewGroup的子视图通过dispatchTransformedTouchEvent()继续分发事件。


3、dispatchTransformedTouchEvent()方法将通过View的dispatchTouchEvent()方法继续分发事件。

`ViewGroup.java`

```java
    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        boolean handled = false;
        //为了安全性过滤触摸事件
        if (onFilterTouchEventForSecurity(ev)) {
            final int action = ev.getAction();
            // 0xff 动作代码中动作本身部分的位掩码。属于多点触碰。
            final int actionMasked = action & MotionEvent.ACTION_MASK;

            // 处理最初的DOWN事件。
            if (actionMasked == MotionEvent.ACTION_DOWN) {
                // 当开始一个新的触摸手势时扔掉所有以前的状态。
                // 由于应用程序切换、ANR或其他一些状态变化，框架可能已经放弃了前一个手势的up或cancel事件。
                cancelAndClearTouchTargets(ev);
                resetTouchState();
            }

            // 检查拦截。
            final boolean intercepted;
            if (actionMasked == MotionEvent.ACTION_DOWN
                    || mFirstTouchTarget != null) {
                final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
                if (!disallowIntercept) {
					               //TODO 1、当返回true，表示该事件被当前视图拦截；当返回false，继续执行事件分发。
                    intercepted = onInterceptTouchEvent(ev);
                    ev.setAction(action); //恢复操作，以防它被更改
                } else {
                    intercepted = false;
                }
            } else {
                // 没有触摸目标，这个动作不是初始向下，所以这个ViewGroup继续拦截触摸。
                intercepted = true;
            }
            // 如果被拦截，启动正常的事件分发。另外，如果已经有一个处理手势的视图，则执行正常的事件分派。
            if (intercepted || mFirstTouchTarget != null) {
                ev.setTargetAccessibilityFocus(false);
            }

            // 检查取消。
            final boolean canceled = resetCancelNextUpFlag(this)
                    || actionMasked == MotionEvent.ACTION_CANCEL;
            // 如果需要，更新指针指向的触摸目标列表。
            final boolean split = (mGroupFlags & FLAG_SPLIT_MOTION_EVENTS) != 0;
            TouchTarget newTouchTarget = null;
            boolean alreadyDispatchedToNewTouchTarget = false;
            if (!canceled && !intercepted) {
                // 如果事件的目标是可访问性焦点，那么我们将它交给具有可访问性焦点的视图，如果它不处理它，我们将清除标志并像往常一样将事件分派给所有的孩子。
                // 我们正在查找可访问性集中的主机，以避免保持状态，因为这些事件非常罕见。
                View childWithAccessibilityFocus = ev.isTargetAccessibilityFocus()
                        ? findChildWithAccessibilityFocus() : null;

                if (actionMasked == MotionEvent.ACTION_DOWN
                        || (split && actionMasked == MotionEvent.ACTION_POINTER_DOWN)
                        || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
                    final int actionIndex = ev.getActionIndex(); // always 0 for down
                    final int idBitsToAssign = split ? 1 << ev.getPointerId(actionIndex)
                            : TouchTarget.ALL_POINTER_IDS;
                    //清理这个指针id的早期触摸目标，以防它们不同步。
                    removePointersFromTouchTargets(idBitsToAssign);

                    final int childrenCount = mChildrenCount;
                    if (newTouchTarget == null && childrenCount != 0) {
                        final float x = ev.getX(actionIndex);
                        final float y = ev.getY(actionIndex);
                        //TODO 2、从视图最上层到下层，获取所有能接收该事件的子视图
                        final ArrayList<View> preorderedList = buildTouchDispatchChildList();
                        final boolean customOrder = preorderedList == null
                                && isChildrenDrawingOrderEnabled();
                        final View[] children = mChildren;
                        for (int i = childrenCount - 1; i >= 0; i--) {
                            final int childIndex = getAndVerifyPreorderedIndex(
                                    childrenCount, i, customOrder);
                            final View child = getAndVerifyPreorderedView(
                                    preorderedList, children, childIndex);
                            // 如果有一个具有可访问性焦点的视图，我们希望它首先获得事件，如果不处理，我们将执行正常的分派。
                            // 我们可以进行两次迭代，但是考虑到时间范围，这样做更安全。
                            if (childWithAccessibilityFocus != null) {
                                if (childWithAccessibilityFocus != child) {
                                    continue;
                                }
                                childWithAccessibilityFocus = null;
                                i = childrenCount - 1;
                            }

                            if (!child.canReceivePointerEvents()
                                    || !isTransformedTouchPointInView(x, y, child, null)) {
                                ev.setTargetAccessibilityFocus(false);
                                continue;
                            }

                            newTouchTarget = getTouchTarget(child);
                            if (newTouchTarget != null) {
                                // 孩子已经在其范围内接受触摸。
                                // 除了它正在处理的指针之外，再给它一个新的指针。
                                newTouchTarget.pointerIdBits |= idBitsToAssign;
                                break;
                            }

                            resetCancelNextUpFlag(child);
                            // TODO 3、分发触摸事件
                            if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
                                // 孩子想在自己的范围内接受触摸。
                                mLastTouchDownTime = ev.getDownTime();
                                if (preorderedList != null) {
                                    // childIndex点成预分类列表，查找原始索引
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
                                // TODO 4、添加触摸目标
                                newTouchTarget = addTouchTarget(child, idBitsToAssign);
                                alreadyDispatchedToNewTouchTarget = true;
                                break;
                            }
                            // 可访问性焦点没有处理事件，因此清除标记并对所有孩子进行正常分派。
                            ev.setTargetAccessibilityFocus(false);
                        }
                        if (preorderedList != null) preorderedList.clear();
                    }

                    if (newTouchTarget == null && mFirstTouchTarget != null) {
                        // 没有找到一个孩子来接收这个事件。
                        // 将指针分配给最近添加的最少的目标。
                        newTouchTarget = mFirstTouchTarget;
                        while (newTouchTarget.next != null) {
                            newTouchTarget = newTouchTarget.next;
                        }
                        newTouchTarget.pointerIdBits |= idBitsToAssign;
                    }
                }
            }

            // 分发给触摸目标。
            if (mFirstTouchTarget == null) {
                // TODO 3、分发触摸事件。没有接触目标，所以把这当作一个普通的View。 
                handled = dispatchTransformedTouchEvent(ev, canceled, null,
                        TouchTarget.ALL_POINTER_IDS);
            } else {
                // 发送给触摸目标，不包括新的触摸目标，如果我们已经发送给它。必要时取消接触目标。
                TouchTarget predecessor = null;
                TouchTarget target = mFirstTouchTarget;
                while (target != null) {
                    final TouchTarget next = target.next;
                    if (alreadyDispatchedToNewTouchTarget && target == newTouchTarget) {
                        handled = true;
                    } else {
                        final boolean cancelChild = resetCancelNextUpFlag(target.child)
                                || intercepted;
                        //TODO 3、分发触摸事件
                        if (dispatchTransformedTouchEvent(ev, cancelChild,
                                target.child, target.pointerIdBits)) {
                            handled = true;
                        }
                        if (cancelChild) {
                            if (predecessor == null) {
                                mFirstTouchTarget = next;
                            } else {
                                predecessor.next = next;
                            }
                            target.recycle();
                            target = next;
                            continue;
                        }
                    }
                    predecessor = target;
                    target = next;
                }
            }
            
            // 更新列表的触摸目标指针或取消，如果需要。
            if (canceled
                    || actionMasked == MotionEvent.ACTION_UP
                    || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
                resetTouchState();
            } else if (split && actionMasked == MotionEvent.ACTION_POINTER_UP) {
                final int actionIndex = ev.getActionIndex();
                final int idBitsToRemove = 1 << ev.getPointerId(actionIndex);
                removePointersFromTouchTargets(idBitsToRemove);
            }
        }
        return handled;
    }
```
## ViewGroup.onInterceptTouchEvent()
`ViewGroup.java`
```java
/**
 * 实现此方法来拦截所有触摸屏动作事件。这允许您在事件发送给孩子时查看它们，并在任何时候获得当前动作的所有权。
 * 使用这个方法需要谨慎，因为它与View.onTouchEvent(MotionEvent)的交互相当复杂，使用它需要以正确的方式实现这两个方法。事件将按以下顺序接收:
 * 1、您将在这里接收down事件。
 * 2、down事件将由这个视图组的一个子视图组处理，或者交给您自己的onTouchEvent()方法来处理;这意味着您应该实现onTouchEvent()来返回true，这样您将继续看到手势的其余部分(而不是寻找父视图来处理它)。另外，通过从onTouchEvent()返回true，您将不会在onInterceptTouchEvent()中收到任何后续事件，并且所有的触摸处理都必须在onTouchEvent()中正常发生。
 * 3、只要从这个函数返回false，接下来的每个事件(直到并包括最终的up)都将首先在这里传递，然后传递到目标的onTouchEvent()。
 * 4、如果您从这里返回true，您将不会接收到任何以下事件:目标视图将接收到相同的事件，但是带有action MotionEvent#ACTION_CANCEL，并且所有进一步的事件将被传递到您的onTouchEvent()方法，并且不再出现在这里。
 *
 * @param ev 要沿层次结构向下分派的触摸事件。
 * @return 返回true从子元素中窃取动作事件，并通过onTouchEvent()将它们分配给这个ViewGroup。当前目标将接收一个ACTION_CANCEL事件，这里不再传递任何消息。
 */
public boolean onInterceptTouchEvent(MotionEvent ev) {
    if (ev.isFromSource(InputDevice.SOURCE_MOUSE)
            && ev.getAction() == MotionEvent.ACTION_DOWN
            && ev.isButtonPressed(MotionEvent.BUTTON_PRIMARY)
            && isOnScrollbarThumb(ev.getX(), ev.getY())) {
        return true;
    }
    return false;
}
```

## ViewGroup.buildTouchDispatchChildList()

`ViewGroup.java`
```java
/**
 * 提供视图的自定义顺序，在其中分配触摸。这是在紧密循环中调用的，因此不允许分配对象，包括返回数组。相反，您应该返回一个预先分配的列表，该列表将在分派完成后被清除。
 * @hide
 */
public ArrayList<View> buildTouchDispatchChildList() {
    return buildOrderedChildList();
}
```

`ViewGroup.java`
```java
/**
 * 填充(并返回)mPreSortedChildren一个视图的子元素的预定列表，首先按Z排序，然后按子元素绘制顺序排序(如果适用的话)。此列表在使用后必须清除，以避免泄漏子视图。使用一个稳定的插入排序，通常是O(n)的viewgroup很少上升的孩子。
 */
ArrayList<View> buildOrderedChildList() {
    final int childrenCount = mChildrenCount;
    if (childrenCount <= 1 || !hasChildWithZ()) return null;

    if (mPreSortedChildren == null) {
        mPreSortedChildren = new ArrayList<>(childrenCount);
    } else {
        // callers should clear, so clear shouldn't be necessary, but for safety...
        mPreSortedChildren.clear();
        mPreSortedChildren.ensureCapacity(childrenCount);
    }

    final boolean customOrder = isChildrenDrawingOrderEnabled();
    for (int i = 0; i < childrenCount; i++) {
        // 将下一个子元素(按子元素顺序)添加到列表的末尾
        final int childIndex = getAndVerifyPreorderedIndex(childrenCount, i, customOrder);
        final View nextChild = mChildren[childIndex];
        final float currentZ = nextChild.getZ();

        // 在所有Z值较大的视图之前插入
        int insertIndex = i;
        while (insertIndex > 0 && mPreSortedChildren.get(insertIndex - 1).getZ() > currentZ) {
            insertIndex--;
        }
        mPreSortedChildren.add(insertIndex, nextChild);
    }
    return mPreSortedChildren;
}
```
## ViewGroup.dispatchTransformedTouchEvent()

`ViewGroup.java`
```java
    /**
     * 将触摸事件转换为特定子视图的坐标空间，筛选不相关的指针id，并在必要时覆盖其操作。如果child为空，则假设MotionEvent将被发送到这个ViewGroup。
     */
    private boolean dispatchTransformedTouchEvent(MotionEvent event, boolean cancel,
            View child, int desiredPointerIdBits) {
        final boolean handled;

        // 取消触摸是一种特殊情况。我们不需要执行任何转换或过滤。重要的是行动，而不是内容。
        final int oldAction = event.getAction();
        if (cancel || oldAction == MotionEvent.ACTION_CANCEL) {
            event.setAction(MotionEvent.ACTION_CANCEL);
            if (child == null) {
                handled = super.dispatchTouchEvent(event);
            } else {
                handled = child.dispatchTouchEvent(event);
            }
            event.setAction(oldAction);
            return handled;
        }

        // 计算要传递的指针的数量。
        final int oldPointerIdBits = event.getPointerIdBits();
        final int newPointerIdBits = oldPointerIdBits & desiredPointerIdBits;

        // 如果由于某种原因，我们最终处于不一致的状态，看起来我们可能会产生一个没有指针的触摸事件，那么就删除该事件。
        if (newPointerIdBits == 0) {
            return false;
        }

        // 如果指针的数量是相同的，并且我们不需要执行任何复杂的不可逆转换，那么我们可以为这个分派重用motion事件，只要我们小心地还原我们所做的任何更改。否则我们需要复印一份。
        final MotionEvent transformedEvent;
        if (newPointerIdBits == oldPointerIdBits) {
            if (child == null || child.hasIdentityMatrix()) {
                if (child == null) {
                		//TODO * 分发触摸事件
                    handled = super.dispatchTouchEvent(event);
                } else {
                    final float offsetX = mScrollX - child.mLeft;
                    final float offsetY = mScrollY - child.mTop;
                    event.offsetLocation(offsetX, offsetY);
							//TODO * 分发触摸事件
                    handled = child.dispatchTouchEvent(event);

                    event.offsetLocation(-offsetX, -offsetY);
                }
                return handled;
            }
            transformedEvent = MotionEvent.obtain(event);
        } else {
            transformedEvent = event.split(newPointerIdBits);
        }

        // 执行任何必要的转换和分派。
        if (child == null) {
        		//TODO * 分发触摸事件
            handled = super.dispatchTouchEvent(transformedEvent);
        } else {
            final float offsetX = mScrollX - child.mLeft;
            final float offsetY = mScrollY - child.mTop;
            transformedEvent.offsetLocation(offsetX, offsetY);
            if (! child.hasIdentityMatrix()) {
                transformedEvent.transform(child.getInverseMatrix());
            }
				 //TODO * 分发触摸事件
            handled = child.dispatchTouchEvent(transformedEvent);
        }

        // 结束
        transformedEvent.recycle();
        return handled;
    }
```
## ViewGroup.addTouchTarget()

`ViewGroup.java`
```java
    /**
     * 将指定子对象的触摸目标添加到列表的开头。假设目标子节点不存在。
     */
    private TouchTarget addTouchTarget(@NonNull View child, int pointerIdBits) {
        final TouchTarget target = TouchTarget.obtain(child, pointerIdBits);
        target.next = mFirstTouchTarget;
        mFirstTouchTarget = target;
        return target;
    }
```



## View.dispatchTouchEvent()
`View.java`
```java
/**
 * 将触摸屏触摸事件向下传递到目标视图，如果是目标视图，则传递到该视图。
 * @param event 要分发的触摸事件。
 * @return 如果事件是由视图处理的，则为真，否则为假。
 */
public boolean dispatchTouchEvent(MotionEvent event) {
    boolean result = false;
    ...
    return result;
}
```

## 总结



