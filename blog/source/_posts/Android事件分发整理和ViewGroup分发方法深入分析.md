---
title: Android事件分发整理和ViewGroup分发方法深入分析
date: 2020-04-04 12:04:19
tags:
- 干货
categories:
- Android
---

## 事件传递流程

#### 事件向下依次传递顺序

1. `Activity`
2. `PhoneWindow`（这层基本啥也没干）
3. `ViewGroup`
4. `View`

#### 主要方法

* `dispatchTouchEvent`

  - 返回true表示下层或者自己消耗了事件
  - 返回false表示下层或者自己没有消耗事件

* `onInterceptTouchEvent`（`View`没有这个方法，只有`ViewGroup`有）

  * 返回true表示拦截事件

  * 返回false表示放行事件
  * 如果当前`ViewGroup`拦截了一个事件序列中的任意一个事件后，那么在同一个事件序列当中，此方法不会被再次调用，并且，后续事件不会再传递给传递下去，返回结果表示是否拦截当前事件。

* `onTouchEvent`

  * 点击事件处理的主要方法

  * 由于View没有拦截方法，所以一旦分发到它会直接调用此方法，View会默认消耗掉事件

## `ViewGroup`主要Tag说明

### `mFirstTouchTarget`

1. `mFirstTouchTarget` 为null, 那么`ViewGroup` 就默认拦截接下来同一序列中所有的点击事件
2. `mFirstTouchTarget` 不为null, 就说明找到子**View**来处理这个事件，并且不会再询问`onInterceptTouchEvent`是否要拦截事件，将后续事件交于这个**View**去处理

**源码证明`android.view.ViewGroup#dispatchTouchEvent`**

```java
if (actionMasked == MotionEvent.ACTION_DOWN
        || mFirstTouchTarget != null) {//观点二中的不会再调用onInterceptTouchEvent的证据
    final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
    if (!disallowIntercept) {
        intercepted = onInterceptTouchEvent(ev);
        ev.setAction(action); // restore action in case it was changed
    } else {
        intercepted = false;
    }
} else {
    //观点一证据
    //当一个事件不是ACTION_DOWN而且mFirstTouchTarget为空
    //说明找不到一个子View来处理这个事件，那就默认拦截
    // There are no touch targets and this action is not an initial down
    // so this view group continues to intercept touches.
    intercepted = true;
}
```

#### 哪些情况`mFirstTouchTarget`会为空？

* `ViewGroup` 没有子元素
* 子元素处理了点击点击事件，但是在`dispatchTouchEvent` 中返回了false, 这一般是因为子元素在`onTouchEvent` 中返回了false

### FLAG_DISALLOW_INTERCEPT

该tag会对`ViewGroup`的拦截逻辑进行影响

调用`requestDisallowInterceptTouchEvent`方法之后，会改变该tag的值

## 整个`ViewGroup的dispatchTouchEvent`详细分析

这是最复杂的部分也是解决滑动冲突的关键部分，代码很长，一步步看，

其中以$$$开头的注释与事件分发无关

```java
public boolean dispatchTouchEvent(MotionEvent ev) {
    if (mInputEventConsistencyVerifier != null) {
        mInputEventConsistencyVerifier.onTouchEvent(ev, 1);
    }

    if (ev.isTargetAccessibilityFocus() && isAccessibilityFocusedViewOrHost()) {
        ev.setTargetAccessibilityFocus(false);
    }

    boolean handled = false;//这个变量就是要返回的boolean类型的值
    //$$$对点击事件进行过滤，这里是方便应用对应的安全策略
    if (onFilterTouchEventForSecurity(ev)) {
        final int action = ev.getAction();//获取当前点击事件
        //$$$对当前点击事件进行 与 操作，这里是对多点触控的处理ACTION_MASK是0xff
        final int actionMasked = action & MotionEvent.ACTION_MASK;
        //判断当前点击事件是不是Down，是的话就会重置一些状态，开启新的一轮点击事件
        //其中包括将FLAG_DISALLOW_INTERCEPT和mFirstTouchTarget重置
        if (actionMasked == MotionEvent.ACTION_DOWN) {
            cancelAndClearTouchTargets(ev);
            resetTouchState();
        }

        //这个变量会根据onInterceptTouchEvent的调用状态或者返回值确定
        //也就是 是否拦截
        final boolean intercepted;
        //如果当前事件是Down，说明开启了新的一轮的事件序列，那么进入条件判断
        if (actionMasked == MotionEvent.ACTION_DOWN
            //如果不是Down，那么如果mFirstTouchTarget不为空的话也满足要求
                || mFirstTouchTarget != null) {
            //查看子元素是否要求父元素不要拦截事件，也就是在子元素中调用了父元素的
            //requestDisallowInterceptTouchEvent方法之后，会在这里根据结果生效
            final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
            //如果子元素要求父元素不要拦截
            //那么父元素就不会执行onInterceptTouchEvent询问自己是否要拦截
            if (!disallowIntercept) {
                //如果子元素没有要求父元素不要拦截，这里则会调用
                intercepted = onInterceptTouchEvent(ev);
                //$$$恢复操作，以防它被更改
                ev.setAction(action); 
            } else {
                //被要求不拦截，默认赋值false，表示不拦截
                intercepted = false;
            }
        } else {
            //没有找到子元素处理（即mFirstTouchTarget == null），也并非是Down事件，则默认拦截
            intercepted = true;
        }

        //如果被拦截，启动正常的事件分发。
        //或者如果已经有一个处理手势的视图，则执行正常的事件分发。
        if (intercepted || mFirstTouchTarget != null) {
            ev.setTargetAccessibilityFocus(false);
        }

        final boolean canceled = resetCancelNextUpFlag(this)
                || actionMasked == MotionEvent.ACTION_CANCEL;

        final boolean split = (mGroupFlags & FLAG_SPLIT_MOTION_EVENTS) != 0;
        
       //新建一个TouchTarget
        TouchTarget newTouchTarget = null;
        //有没有获取到最新的TouchTarget，这里也是一个标志
        boolean alreadyDispatchedToNewTouchTarget = false;
        //没有取消，并且没有被拦截的话就找子元素处理
        if (!canceled && !intercepted) {

            View childWithAccessibilityFocus = ev.isTargetAccessibilityFocus()
                    ? findChildWithAccessibilityFocus() : null;

            if (actionMasked == MotionEvent.ACTION_DOWN
                    || (split && actionMasked == MotionEvent.ACTION_POINTER_DOWN)
                    || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
                final int actionIndex = ev.getActionIndex(); // always 0 for down
                final int idBitsToAssign = split ? 1 << ev.getPointerId(actionIndex)
                        : TouchTarget.ALL_POINTER_IDS;

                removePointersFromTouchTargets(idBitsToAssign);

                final int childrenCount = mChildrenCount;
                //如果当前ViewGroup有子控件，则开始寻找
                if (newTouchTarget == null && childrenCount != 0) {
                    //获取位置
                    final float x = ev.getX(actionIndex);
                    final float y = ev.getY(actionIndex);
                    final ArrayList<View> preorderedList = buildTouchDispatchChildList();
                    final boolean customOrder = preorderedList == null
                            && isChildrenDrawingOrderEnabled();
                    final View[] children = mChildren;
                    //倒叙开始寻找
                    for (int i = childrenCount - 1; i >= 0; i--) {
                        final int childIndex = getAndVerifyPreorderedIndex(
                                childrenCount, i, customOrder);
                        final View child = getAndVerifyPreorderedView(
                                preorderedList, children, childIndex);

                        if (childWithAccessibilityFocus != null) {
                            if (childWithAccessibilityFocus != child) {
                                continue;
                            }
                            childWithAccessibilityFocus = null;
                            i = childrenCount - 1;
                        }

                        //这里判断当前遍历到的这个子元素能不能接收点击事件，
                        //如果不能进行下一次
                        if (!child.canReceivePointerEvents()
                                || !isTransformedTouchPointInView(x, y, child, null)) {
                            ev.setTargetAccessibilityFocus(false);
                            continue;
                        }
                        //从mFirstTouchTarget序列中（其实这里是一个单链表）
                        //找与这个child对应的TouchTarget,没有
                        newTouchTarget = getTouchTarget(child);
                        if (newTouchTarget != null) {
                            //如果能够找到对应的，说明该view已经处理过当前点击事件序列前面一点的事件
                            //已经在其范围内接受触摸,除开它正在处理的再给它新的位置指针
                            newTouchTarget.pointerIdBits |= idBitsToAssign;
                            break;
                        }

                        resetCancelNextUpFlag(child);
                        //如果能够运行到这里说明该子元素满足要求，可以接收点击事件
                        //dispatchTransformedTouchEvent中会执行子元素的dispatchTouchEvent
                        //如果该子元素将事件消耗，则进入判断语句
                        if (dispatchTransformedTouchEvent(ev, false, child, 
                                                          idBitsToAssign)) {
                            mLastTouchDownTime = ev.getDownTime();
                            if (preorderedList != null) {
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
                            //如果这个事件消耗了，则生成一个对应的TouchTarget
                            //并赋值给newTouchTarget
                            //并且在addTouchTarget方法里也将这个赋值给了mFirstTouchTarget
                            newTouchTarget = addTouchTarget(child, idBitsToAssign);
                            alreadyDispatchedToNewTouchTarget = true;
                            //停止在子元素中寻找子元素去处理
                            break;
                        }
                        ev.setTargetAccessibilityFocus(false);
                    }
                    if (preorderedList != null) preorderedList.clear();
                }

                if (newTouchTarget == null && mFirstTouchTarget != null) {
                    newTouchTarget = mFirstTouchTarget;
                    while (newTouchTarget.next != null) {
                        newTouchTarget = newTouchTarget.next;
                    }
                    newTouchTarget.pointerIdBits |= idBitsToAssign;
                }
            }
        }

        //mFirstTouchTarget == null如果这里为空说明没有找到子元素能够处理或者消耗了
        if (mFirstTouchTarget == null) {
            //直接传null进去，那么这个方法内部就会直接调用该ViewGroup的
            //super.dispatchTouchEvent(event)
            handled = dispatchTransformedTouchEvent(ev, canceled, null,
                    TouchTarget.ALL_POINTER_IDS);
        } else {
            TouchTarget predecessor = null;
            TouchTarget target = mFirstTouchTarget;
            while (target != null) {
                final TouchTarget next = target.next;
                if (alreadyDispatchedToNewTouchTarget && target == newTouchTarget) {
                    handled = true;
                } else {
                    //mFirstTouchTarget不为空，说明有子元素想要处理这个事件，
            		//子元素的dispatch方法在Down来临的时候,返回了true
            		//但是此时由于当前这个ViewGroup，发现后续的事件里面，有它想拦截的
            		//那么此时就会在前面的拦截方法里返回true拦截这个事件
                    //那么此时intercepted就为true，下面这个cancelChild就会为true
                    //这个变量表示是否要取消对确认要接收的子元素接收后续事件的资格
                    final boolean cancelChild = resetCancelNextUpFlag(target.child)
                            || intercepted;
                    //调用dispatchTransformedTouchEvent并传参数cancelChild
                    //这样子View就可以收到ACTION_CANCEL的消息了，收到这个消息之后
                    //这个子元素就可以和当前这个事件序列说再见了
                    if (dispatchTransformedTouchEvent(ev, cancelChild,
                            target.child, target.pointerIdBits)) {
                        handled = true;
                    }
                    
                    if (cancelChild) {
                        if (predecessor == null) {
                            //这里赋值为next，但是这个情况下next为null
                            //所以就相当于把mFirstTouchTarget重置了
                            //那么当后续的事件来的时候就会默认拦截
                            //这也是为什么当onInterceptTouchEvent拦截
                            //任意事件一次后，后面的事件统统都不会再往下传的原因
                            //而且onInterceptTouchEvent不会再一次被调用了
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

    if (!handled && mInputEventConsistencyVerifier != null) {
        mInputEventConsistencyVerifier.onUnhandledEvent(ev, 1);
    }
    return handled;
}
```

呼~~好累，这么一分析啥原理不就都来了，什么解决滑动冲突的方法还不是信手拈来，什么外部拦截或者内部拦截的原理不就一下子清晰了。累了，去泡杯茶再来。



**Two thousand years later..........................**



# 解决滑动冲突

有了上面缜密的一步一步的分析，接下来就可以很明白的理解Android开发艺术探索上面说的两种解决滑动冲突的方法

### 外部拦截？

很简单，重写`onInterceptTouchEvent`不拦截Down事件，保证子元素能够接收到，一旦你判断你需要这个事件以及后面的事件的时候，返回true拦截，夺回所有权就好，但是如果你如果你在UP事件的时候再决定夺回所有权时，没啥意义呀根本，而且还会导致子元素不能成功收到UP事件，但是单击监听失效，所以最好别这么蛮横在这时候来夺回所有权。

### 内部拦截？

所有的事件`ViewGroup`都不拦截，在子元素处理拦截逻辑，并调用父布局的`requestDisallowInterceptTouchEvent`方法来影响父布局的FLAG_DISALLOW_INTERCEPT这个标志位，以此来影响父布局的，让父布局们听自己的号召。

> 注意：调用父布局`requestDisallowInterceptTouchEvent`这个方法之后，父布局也会通知它的父布局，还是看看源码，很简单，不摆了，困
>
> ```java
> public void requestDisallowInterceptTouchEvent(boolean disallowIntercept) {
> 
>     if (disallowIntercept == ((mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0)) {
>         // We're already in this state, assume our ancestors are too
>         return;
>     }
> 
>     if (disallowIntercept) {
>         mGroupFlags |= FLAG_DISALLOW_INTERCEPT;
>     } else {
>         mGroupFlags &= ~FLAG_DISALLOW_INTERCEPT;
>     }
> 
>     // Pass it up to our parent
>     if (mParent != null) {
>         mParent.requestDisallowInterceptTouchEvent(disallowIntercept);
>     }
> }
> ```