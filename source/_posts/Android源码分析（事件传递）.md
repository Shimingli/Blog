---
title: Android源码分析（事件传递）
date: 2018-05-14 16:21:57
tags: Android事件传递
---
## 源码基于安卓8.0分析结果
* 首先如何看安卓SDK源码，作者尝试过几种的方法，感觉这种比较方便
把在本地找到的Android.jar 放到工程中的libs的目录下，直接编译，就可以看到PhoneWindow 和DecorView的源码了
![image.png](https://upload-images.jianshu.io/upload_images/5363507-58562a1784cc86ca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<!--  more  -->
* 结论：Android事件分发流程 = Activity -> ViewGroup -> View 
![事件传递.png](https://upload-images.jianshu.io/upload_images/5363507-1a5d392774b949c7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### Activity事件分发机制
* Activity事件分发机制（Activity源码分析）
     *  当一个点击事件发生时，事件最先传到`Activity`的`dispatchTouchEvent()`进行事件分发
![image.png](https://upload-images.jianshu.io/upload_images/5363507-8e8c3d0127a39ff4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
  public boolean dispatchTouchEvent(MotionEvent ev) {
        //第一点
        if (ev.getAction() == MotionEvent.ACTION_DOWN) {
            onUserInteraction();
        }
       //第二点
        if (getWindow().superDispatchTouchEvent(ev)) {
            return true;
        }
      //第三点
        return onTouchEvent(ev);
    }
```
    
 * 1、触发方法  onUserInteraction();这个事件触发的机制是DOWN事件
![image.png](https://upload-images.jianshu.io/upload_images/5363507-5d6a95e44f2b2233.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
/**
  * onUserInteraction()
  * 作用：实现屏保功能
  * 注：
  *    a. 该方法为空方法
  *    b. 当此activity在栈顶时，触屏点击按home，back，menu键等都会触发此方法
  */
      public void onUserInteraction() { 
      }
```

 * 2、触发方法 getWindow().superDispatchTouchEvent(ev)
      *  getWindow()：：：mWindow = new PhoneWindow(this);

![image.png](https://upload-images.jianshu.io/upload_images/5363507-bb53a94f2832e1a7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
  * 定义：属于顶层View（DecorView）
     * 说明：
     *   a. DecorView类是PhoneWindow类的一个内部类
     * b. DecorView继承自FrameLayout，是所有界面的父类
     * c. FrameLayout是ViewGroup的子类，故DecorView的间接父类 = ViewGroup

```
     mDecor = (DecorView) preservedWindow.getDecorView();
```
* 到此可以证明Activity的根布局是FrameLayout
![image.png](https://upload-images.jianshu.io/upload_images/5363507-0ca11b1769d84822.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 调用父类的方法 = ViewGroup的dispatchTouchEvent()
 *  即 将事件传递到ViewGroup去处理，详细请看ViewGroup的事件分发机制
```

 private final class DecorView extends FrameLayout{                            
     public boolean superDispatchTouchEvent(MotionEvent event) {
      // 调用父类的方法 = ViewGroup的dispatchTouchEvent()
        // 即 将事件传递到ViewGroup去处理，详细请看ViewGroup的事件分发机制
        return super.dispatchTouchEvent(event);
    }
 }
```
* 3、Activity.onTouchEvent(ev)分析：Activity中的onTouchEvent，个人理解在返回键的时候哦，退出App（或者是一个Activity的结束）
```
    public boolean onTouchEvent(MotionEvent event) {
        if (mWindow.shouldCloseOnTouch(this, event)) {
            finish();
            return true;
        }
       //只有在点击事件在Window边界外才会返回true，一般情况都返回false
        return false;
    }
```

![image.png](https://upload-images.jianshu.io/upload_images/5363507-a9f0ef459d463f6d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 主要是对于处理边界外点击事件的判断：是否是DOWN事件，event的坐标是否在边界内等
```
   /** @hide */
    public boolean shouldCloseOnTouch(Context context, MotionEvent event) {
        if (mCloseOnTouchOutside && event.getAction() == MotionEvent.ACTION_DOWN
                && isOutOfBounds(context, event) && peekDecorView() != null) {
            return true;
        }      
        return false;
    }

```
![Android事件分发流程.jpg](https://upload-images.jianshu.io/upload_images/5363507-483ad66a339cf763.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### ViewGroup件分发机制
* 上面Activity事件分发机制可知，ViewGroup事件分发机制从dispatchTouchEvent()开始

* 分析1：ViewGroup每次事件分发时，都需调用onInterceptTouchEvent()询问是否拦截事件
![image.png](https://upload-images.jianshu.io/upload_images/5363507-c1728b31ab7b667e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
*  a. 若在onInterceptTouchEvent()中返回false（即不拦截事件），就会让第二个值为true，从而进入到条件判断的内部
 *  b. 若在onInterceptTouchEvent()中返回true（即拦截事件），就会让第二个值为false，从而跳出了这个条件判断
#### 调用dispatchTransformedTouchEvent方法的时机
有个遍历ViewGroup所有的孩子
![image.png](https://upload-images.jianshu.io/upload_images/5363507-a1274ffacf550148.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


* 分析2：dispatchTransformedTouchEvent方法分析
     * 将运动事件转换为特定子视图的坐标空间，
     * 过滤不相关的指针ID，并在必要时重写其操作。
     * 如果孩子是无效的，假设位移事件将被发送到这个ViewGroup相反。
![image.png](https://upload-images.jianshu.io/upload_images/5363507-a7c20bb588a68a87.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


  * 调用ViewGroup父类的dispatchTouchEvent()，即View.dispatchTouchEvent()
  * 因此会执行ViewGroup的onTouch() ->> onTouchEvent() ->> performClick（） ->> onClick()，
  * 即自己处理该事件，事件不会往下传递（具体请参考View事件的分发机制中的机制)
```
   private boolean dispatchTransformedTouchEvent(MotionEvent event, boolean cancel,
                                                  View child, int desiredPointerIdBits) {
        final boolean handled;

        //取消运动是一种特殊情况。我们不需要执行任何转换
        //或过滤。重要的是行动，而不是内容。
        final int oldAction = event.getAction();
        if (cancel || oldAction == MotionEvent.ACTION_CANCEL) {
            event.setAction(MotionEvent.ACTION_CANCEL);
            // 调用ViewGroup父类的dispatchTouchEvent()，即View.dispatchTouchEvent()
            // 因此会执行ViewGroup的onTouch() ->> onTouchEvent() ->> performClick（） ->> onClick()，
            // 即自己处理该事件，事件不会往下传递（具体请参考View事件的分发机制中的View.dispatchTouchEvent（））
            // 此处需与上面区别：子View的dispatchTouchEvent（）

            // 若点击的是空白处（即无任何View接收事件） / 拦截事件（手动复写onInterceptTouchEvent（），从而让其返回true）
            if (child == null) {
                handled = super.dispatchTouchEvent(event);
            } else {
                //调用View onTouchEvent() ->> performClick（） ->> onClick()，
                handled = child.dispatchTouchEvent(event);
            }
            event.setAction(oldAction);
            /**
             调用子View的dispatchTouchEvent后是有返回值的
             若该控件可点击，那么点击时，dispatchTouchEvent的返回值必定是true，因此会导致条件判断成立
             于是给ViewGroup的dispatchTouchEvent（）直接返回了true，即直接跳出
             即把ViewGroup的点击事件拦截掉
             */
            return handled;
        }
    }
```


```
    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        //手势输入的验证者
        if (mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onTouchEvent(ev, 1);
        }
        //如果事件以可访问性为焦点的视图为目标，这就是，开始
        //正常事件分派。也许一个后代会处理点击。
        if (ev.isTargetAccessibilityFocus() && isAccessibilityFocusedViewOrHost()) {
            ev.setTargetAccessibilityFocus(false);
        }

        boolean handled = false;
        if (onFilterTouchEventForSecurity(ev)) {
            final int action = ev.getAction();
            final int actionMasked = action & MotionEvent.ACTION_MASK;

            // 处理初始值。第一个点击的ACTION_DOWN 事件
            if (actionMasked == MotionEvent.ACTION_DOWN) {
                //在开始一个新的触摸手势时丢弃所有以前的状态。
                //框架可能会取消上一个手势的上或取消事件。
                //由于程序开关，ANR，或一些其他的状态变化。
                cancelAndClearTouchTargets(ev);
                resetTouchState();
            }

            //检查拦截。
            final boolean intercepted;
            if (actionMasked == MotionEvent.ACTION_DOWN
                    || mFirstTouchTarget != null) {
                //  // 判断值1：disallowIntercept = 是否禁用事件拦截的功能(默认是false)，
                // 可通过调用requestDisallowInterceptTouchEvent()修改
                final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
                if (!disallowIntercept) {
                    // // 重点分析1：ViewGroup每次事件分发时，都需调用onInterceptTouchEvent()询问是否拦截事件

                    // 判断值2：intercepted= onInterceptTouchEvent(ev)返回值
                    /**
                     a. 若在onInterceptTouchEvent()中返回false（即不拦截事件），就会让第二个值为true，从而进入到条件判断的内部
                     b. 若在onInterceptTouchEvent()中返回true（即拦截事件），就会让第二个值为false，从而跳出了这个条件判断
                     intercepted=false  不拦截
                     */
                    intercepted = onInterceptTouchEvent(ev);
                    //恢复操作，以防更改
                    ev.setAction(action); // restore action in case it was changed
                } else {
                    intercepted = false;
                }
            } else {
                //没有触摸目标，这个动作不是初始的。
                //因此，此视图组继续拦截触摸。
                intercepted = true;
            }

            ///如果被拦截，启动正常事件分派。如果已经有了
            // /正在处理手势的视图，执行正常事件调度。
            /**
             a. 若在onInterceptTouchEvent()中返回false（即不拦截事件），就会让第二个值为true，从而进入到条件判断的内部
             b. 若在onInterceptTouchEvent()中返回true（即拦截事件），就会让第二个值为false，从而跳出了这个条件判断
             */
            if (intercepted || mFirstTouchTarget != null) {
                ev.setTargetAccessibilityFocus(false);
            }

            // Check for cancelation. 检查取消。
            final boolean canceled = resetCancelNextUpFlag(this)
                    || actionMasked == MotionEvent.ACTION_CANCEL;
            //如果需要，指针更新的触摸目标更新列表。
            // Update list of touch targets for pointer down, if needed.
            final boolean split = (mGroupFlags & FLAG_SPLIT_MOTION_EVENTS) != 0;
            TouchTarget newTouchTarget = null;
            boolean alreadyDispatchedToNewTouchTarget = false;
            if (!canceled && !intercepted) {

                //  /如果事件是针对accessiiblity焦点我们给它的
                //   具有可访问性焦点的视图，如果它不处理它
                //   我们清理标记，像往常一样把事件发给所有的孩子。
                // /我们正在查找可访问性集中的主机以避免保留。
                // /由于这些事件非常罕见。
                View childWithAccessibilityFocus = ev.isTargetAccessibilityFocus()
                        ? findChildWithAccessibilityFocus() : null;

                if (actionMasked == MotionEvent.ACTION_DOWN
                        || (split && actionMasked == MotionEvent.ACTION_POINTER_DOWN)
                        || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
                    //按下永远为0
                    final int actionIndex = ev.getActionIndex(); // always 0 for down
                    final int idBitsToAssign = split ? 1 << ev.getPointerId(actionIndex)
                            : TouchTarget.ALL_POINTER_IDS;

                    ///清除这个指针id的早期触摸目标，以防它们
                    //变得不同步。
                    removePointersFromTouchTargets(idBitsToAssign);

                    final int childrenCount = mChildrenCount;
                    if (newTouchTarget == null && childrenCount != 0) {
                        final float x = ev.getX(actionIndex);
                        final float y = ev.getY(actionIndex);

                        ///找到一个可以接收事件的孩子。
                        //对孩子进行前后扫描。
                        final ArrayList<View> preorderedList = buildOrderedChildList();
                        final boolean customOrder = preorderedList == null
                                && isChildrenDrawingOrderEnabled();
                        final View[] children = mChildren;
                        //，遍历了当前ViewGroup下的所有子View
                        for (int i = childrenCount - 1; i >= 0; i--) {
                            final int childIndex = customOrder
                                    ? getChildDrawingOrder(childrenCount, i) : i;
                            final View child = (preorderedList == null)
                                    ? children[childIndex] : preorderedList.get(childIndex);

                            ///如果有可访问性焦点的视图，我们希望它
                            //  获取事件，如果不处理，我们将执行一个
                            //  正常调度。我们可以做双重迭代，但这是
                            //  更安全鉴于时间表。
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
                                ///孩子在其范围内已经收到了触摸。
                                //给它一个新指针，除了它正在处理的指针。
                                newTouchTarget.pointerIdBits |= idBitsToAssign;
                                break;
                            }

                            resetCancelNextUpFlag(child);
                            if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
                                //孩子想在自己的范围内接受边界。
                                mLastTouchDownTime = ev.getDownTime();
                                if (preorderedList != null) {
                                    // 子索引点预设列表，找到原始指标
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

                            ///可访问性焦点没有处理事件，所以很清楚。
                            //[并]向所有的孩子正常派遣。
                            ev.setTargetAccessibilityFocus(false);
                        }
                        if (preorderedList != null) preorderedList.clear();
                    }

                    if (newTouchTarget == null && mFirstTouchTarget != null) {
                        //没有找到一个孩子来接这件事。
                        ///将指针分配给最近添加的目标。
                        newTouchTarget = mFirstTouchTarget;
                        while (newTouchTarget.next != null) {
                            newTouchTarget = newTouchTarget.next;
                        }
                        newTouchTarget.pointerIdBits |= idBitsToAssign;
                    }
                }
            }

            // Dispatch to touch targets./发送到触摸目标。
            if (mFirstTouchTarget == null) {
                // No touch targets so treat this as an ordinary view.
                //没有触及目标，所以把它当作普通的观点看待。
                handled = dispatchTransformedTouchEvent(ev, canceled, null,
                        TouchTarget.ALL_POINTER_IDS);
            } else {
                // //发送到触摸目标，不包括新的触摸目标，如果我们已经
                //     被派往。如有必要取消触摸目标。
                TouchTarget predecessor = null;
                TouchTarget target = mFirstTouchTarget;
                while (target != null) {
                    final TouchTarget next = target.next;
                    if (alreadyDispatchedToNewTouchTarget && target == newTouchTarget) {
                        handled = true;
                    } else {
                        final boolean cancelChild = resetCancelNextUpFlag(target.child)
                                || intercepted;
                        //  条件判断的内部调用了该View的dispatchTouchEvent()
                        // 即 实现了点击事件从ViewGroup到子View的传递（具体请看下面的View事件分发机制）
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

            //更新需要更新或取消的触摸目标列表。
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

![Activity事件分发流程（ViewGroup）.png](https://upload-images.jianshu.io/upload_images/5363507-e3eb13e3297155f4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### Activity事件分发的流程（View.dispatchTouchEvent(event）

![image.png](https://upload-images.jianshu.io/upload_images/5363507-c48d951731d69983.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
关键代码的分析：
* 1. mOnTouchListener != null （设置setOnTouchListener（this））
*  2. (mViewFlags & ENABLED_MASK) == ENABLED （这个控件能否点击，）
* 3. mOnTouchListener.onTouch(this, event) （返回了true事件被拦截）
只有以下3个条件都为真，dispatchTouchEvent()才返回true
```
     // 说明：只有以下3个条件都为真，dispatchTouchEvent()才返回true；
            // 否则执行onTouchEvent()
            //     1. mOnTouchListener != null
            //     2. (mViewFlags & ENABLED_MASK) == ENABLED
            //     3. mOnTouchListener.onTouch(this, event)

            /**
             * 条件2：(mViewFlags & ENABLED_MASK) == ENABLED
             * 说明：
             *     a. 该条件是判断当前点击的控件是否enable
             *     b. 由于很多View默认enable，故该条件恒定为true
             */
            //当li恒等于null的时候，后面的判断会执行
            //但是li在设置了setOnTouchListener就不为空，如果onTouch返回为true的话，
            // onclick就不会走了
            if (li != null && li.mOnTouchListener != null
                    && (mViewFlags & ENABLED_MASK) == ENABLED
                    && li.mOnTouchListener.onTouch(this, event)) {
                result = true;
            }

```

当上面的result返回为false的时候才会走
```
            //没被拦截走到onTouchEvent（event）
            if (!result && onTouchEvent(event)) {
                result = true;
            }
```

关于onTouchEvent方法：记住这点就行ACTION_UP---》手机抬起来了，才会执行performClick（）
#### 目前大多数的app都是只设置了setOnClickListener（），可以自己手动点击一个按钮，会发现只有抬起手，这个事件才会执行，如果让用户体验的比较好一点，对那些有强迫症的用户，应该也重写setOnTouchListener（this）,但是记住不能拦击了，我的影像中，鲁大师App，就是重写了这个方法，所以在手指没有抬起来，下面tab的颜色已经发生了改变。
```
        // 若在onTouch（）返回true，就会让上述三个条件全部成立，
        // 从而使得View.dispatchTouchEvent（）直接返回true，事件分发结束

        // 若在onTouch（）返回false，就会使得上述三个条件不全部成立，
        // 从而使得View.dispatchTouchEvent（）中跳出If，执行onTouchEvent(event)
      button.setOnTouchListener(new View.OnTouchListener() {
            @Override
          public boolean onTouch(View v, MotionEvent event) {
              return false;
            }
        });
```

![image.png](https://upload-images.jianshu.io/upload_images/5363507-0502e8f837fa0621.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


 只要我们通过setOnClickListener（）为控件View注册1个点击事件 ，那么就会给mOnClickListener变量赋值（即不为空）， 则会往下回调onClick（） & performClick（）返回true，performClick 就是模拟了一个请求，点击事件的请求，之类的东西

```
    public boolean performClick() {
        final boolean result;
        final ListenerInfo li = mListenerInfo;
        if (li != null && li.mOnClickListener != null) {
            playSoundEffect(SoundEffectConstants.CLICK);
//          ListenerInfo mOnClickListener=getListenerInfo();
//            getListenerInfo().mOnClickListener.onClick(this);
            li.mOnClickListener.onClick(this);
            result = true;
        } else {
            result = false;
        }
        sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_CLICKED);
        notifyEnterOrExitForAutoFillIfNeeded(true);
        return result;
    }
```
* 通过对performClick（） 源码解读，你可能可以发现有点怪怪的东西，就是一个方法中的全局变量。以前觉得有点怪怪的，但是目前看来，需要去判断下是否为null，不用去调用方法初始化值。

```
   ListenerInfo getListenerInfo() {
        if (mListenerInfo != null) {
            return mListenerInfo;
        }
        mListenerInfo = new ListenerInfo();
        return mListenerInfo;
    }
```

* 涉及到的所有的代码
```
  @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    public boolean dispatchTouchEvent(MotionEvent event) {
        // 如果事件应该通过可访问性焦点来处理。
        if (event.isTargetAccessibilityFocus()) {
            // 我们没有焦点，也没有虚拟后代，没有处理事件。
            if (!isAccessibilityFocusedViewOrHost()) {
                return false;
            }
            // 我们有焦点，得到事件，然后使用正常事件调度。
            event.setTargetAccessibilityFocus(false);
        }

        boolean result = false;

        if (mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onTouchEvent(event, 0);
        }

        final int actionMasked = event.getActionMasked();
        if (actionMasked == MotionEvent.ACTION_DOWN) {
            //防御新手势
            stopNestedScroll();
        }

        if (onFilterTouchEventForSecurity(event)) {
            if ((mViewFlags & ENABLED_MASK) == ENABLED && handleScrollBarDragging(event)) {
                result = true;
            }
            //noinspection SimplifiableIfStatement
            ListenerInfo li = mListenerInfo;
            // 说明：只有以下3个条件都为真，dispatchTouchEvent()才返回true；
            // 否则执行onTouchEvent()
            //     1. mOnTouchListener != null
            //     2. (mViewFlags & ENABLED_MASK) == ENABLED
            //     3. mOnTouchListener.onTouch(this, event)

            /**
             * 条件2：(mViewFlags & ENABLED_MASK) == ENABLED
             * 说明：
             *     a. 该条件是判断当前点击的控件是否enable
             *     b. 由于很多View默认enable，故该条件恒定为true
             */
            //当li恒等于null的时候，后面的判断会执行
            //但是li在设置了setOnTouchListener就不为空，如果onTouch返回为true的话，
            // onclick就不会走了
            if (li != null && li.mOnTouchListener != null
                    && (mViewFlags & ENABLED_MASK) == ENABLED
                    && li.mOnTouchListener.onTouch(this, event)) {
                result = true;
            }
            //没被拦截走到onTouchEvent（event）
            if (!result && onTouchEvent(event)) {
                result = true;
            }
        }

        if (!result && mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onUnhandledEvent(event, 0);
        }

        //在嵌套滚动之后清理，如果这是手势的结束；
        //也取消了如果我们试着action_down但是我们不想休息
        //手势。
        if (actionMasked == MotionEvent.ACTION_UP ||
                actionMasked == MotionEvent.ACTION_CANCEL ||
                (actionMasked == MotionEvent.ACTION_DOWN && !result)) {
            stopNestedScroll();
        }

        return result;
    }
    public boolean onTouchEvent(MotionEvent event) {
        final float x = event.getX();
        final float y = event.getY();
        final int viewFlags = mViewFlags;
        final int action = event.getAction();
        // 若该控件可点击，则进入switch判断中
        final boolean clickable = ((viewFlags & CLICKABLE) == CLICKABLE
                || (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)
                || (viewFlags & CONTEXT_CLICKABLE) == CONTEXT_CLICKABLE;

        if ((viewFlags & ENABLED_MASK) == DISABLED) {
            if (action == MotionEvent.ACTION_UP && (mPrivateFlags & PFLAG_PRESSED) != 0) {
                setPressed(false);
            }
            mPrivateFlags3 &= ~PFLAG3_FINGER_DOWN;
            // A disabled view that is clickable still consumes the touch
            // events, it just doesn't respond to them.
            return clickable;
        }
        if (mTouchDelegate != null) {
            if (mTouchDelegate.onTouchEvent(event)) {
                return true;
            }
        }
        // 若该控件可点击，则进入switch判断中
        if (clickable || (viewFlags & TOOLTIP) == TOOLTIP) {
            switch (action) {
                // a. 若当前的事件 = 抬起View（主要分析）
                case MotionEvent.ACTION_UP:
                    mPrivateFlags3 &= ~PFLAG3_FINGER_DOWN;
                    if ((viewFlags & TOOLTIP) == TOOLTIP) {
                        handleTooltipUp();
                    }
                    if (!clickable) {
                        removeTapCallback();
                        removeLongPressCallback();
                        mInContextButtonPress = false;
                        mHasPerformedLongPress = false;
                        mIgnoreNextUpEvent = false;
                        break;
                    }
                    boolean prepressed = (mPrivateFlags & PFLAG_PREPRESSED) != 0;
                    if ((mPrivateFlags & PFLAG_PRESSED) != 0 || prepressed) {
                        // take focus if we don't have it already and we should in
                        // touch mode.
                        boolean focusTaken = false;
                        if (isFocusable() && isFocusableInTouchMode() && !isFocused()) {
                            focusTaken = requestFocus();
                        }

                        if (prepressed) {
                            // The button is being released before we actually
                            // showed it as pressed.  Make it show the pressed
                            // state now (before scheduling the click) to ensure
                            // the user sees it.
                            setPressed(true, x, y);
                        }

                        if (!mHasPerformedLongPress && !mIgnoreNextUpEvent) {
                            // This is a tap, so remove the longpress check
                            removeLongPressCallback();

                            // Only perform take click actions if we were in the pressed state
                            if (!focusTaken) {
                                // Use a Runnable and post this rather than calling
                                // performClick directly. This lets other visual state
                                // of the view update before click actions start.
                                if (mPerformClick == null) {
                                    mPerformClick = new PerformClick();
                                }
                                if (!post(mPerformClick)) {
                                    // 执行performClick() ->>分析1
                                    performClick();
                                }
                            }
                        }

                        if (mUnsetPressedState == null) {
                            mUnsetPressedState = new UnsetPressedState();
                        }

                        if (prepressed) {
                            postDelayed(mUnsetPressedState,
                                    ViewConfiguration.getPressedStateDuration());
                        } else if (!post(mUnsetPressedState)) {
                            // If the post failed, unpress right now
                            mUnsetPressedState.run();
                        }

                        removeTapCallback();
                    }
                    mIgnoreNextUpEvent = false;
                    break;
                // b. 若当前的事件 = 按下View
                case MotionEvent.ACTION_DOWN:
                    if (event.getSource() == InputDevice.SOURCE_TOUCHSCREEN) {
                        mPrivateFlags3 |= PFLAG3_FINGER_DOWN;
                    }
                    mHasPerformedLongPress = false;

                    if (!clickable) {
                        checkForLongClick(0, x, y);
                        break;
                    }

                    if (performButtonActionOnTouchDown(event)) {
                        break;
                    }

                    // Walk up the hierarchy to determine if we're inside a scrolling container.
                    boolean isInScrollingContainer = isInScrollingContainer();

                    // For views inside a scrolling container, delay the pressed feedback for
                    // a short period in case this is a scroll.
                    if (isInScrollingContainer) {
                        mPrivateFlags |= PFLAG_PREPRESSED;
                        if (mPendingCheckForTap == null) {
                            mPendingCheckForTap = new CheckForTap();
                        }
                        mPendingCheckForTap.x = event.getX();
                        mPendingCheckForTap.y = event.getY();
                        postDelayed(mPendingCheckForTap, ViewConfiguration.getTapTimeout());
                    } else {
                        // Not inside a scrolling container, so show the feedback right away
                        setPressed(true, x, y);
                        checkForLongClick(0, x, y);
                    }
                    break;
                // c. 若当前的事件 = 结束事件（非人为原因）
                case MotionEvent.ACTION_CANCEL:
                    if (clickable) {
                        setPressed(false);
                    }
                    removeTapCallback();
                    removeLongPressCallback();
                    mInContextButtonPress = false;
                    mHasPerformedLongPress = false;
                    mIgnoreNextUpEvent = false;
                    mPrivateFlags3 &= ~PFLAG3_FINGER_DOWN;
                    break;
                // d. 若当前的事件 = 滑动View
                case MotionEvent.ACTION_MOVE:
                    if (clickable) {
                        drawableHotspotChanged(x, y);
                    }

                    // Be lenient about moving outside of buttons
                    if (!pointInView(x, y, mTouchSlop)) {
                        // Outside button
                        // Remove any future long press/tap checks
                        removeTapCallback();
                        removeLongPressCallback();
                        if ((mPrivateFlags & PFLAG_PRESSED) != 0) {
                            setPressed(false);
                        }
                        mPrivateFlags3 &= ~PFLAG3_FINGER_DOWN;
                    }
                    break;
            }
            // 若该控件可点击，就一定返回true
            return true;
        }
        // 若该控件不可点击，就一定返回false
        return false;
    }
    /**
     * Call this view's OnClickListener, if it is defined.  Performs all normal
     * actions associated with clicking: reporting accessibility event, playing
     * a sound, etc.
     * *把这种观点的OnClickListener，如果它的定义。执行所有正常
     *与点击相关的操作：报告可访问性事件，播放
     *声音等。
     * @return True there was an assigned OnClickListener that was called, false
     *         otherwise is returned.
     */
    // 只要我们通过setOnClickListener（）为控件View注册1个点击事件
    // 那么就会给mOnClickListener变量赋值（即不为空）
    // 则会往下回调onClick（） & performClick（）返回true
    //performClick 就是模拟了一个请求，点击事件的请求，子类的东西
    public boolean performClick() {
        final boolean result;
        final ListenerInfo li = mListenerInfo;
        if (li != null && li.mOnClickListener != null) {
            playSoundEffect(SoundEffectConstants.CLICK);
            // TODO: 2018/4/18 这里看源码  有点意思
//          ListenerInfo mOnClickListener=getListenerInfo();
//            getListenerInfo().mOnClickListener.onClick(this);
            li.mOnClickListener.onClick(this);
            result = true;
        } else {
            result = false;
        }

        sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_CLICKED);

        notifyEnterOrExitForAutoFillIfNeeded(true);

        return result;
    }
```
![Activity事件分发的流程（View.dispatchTouchEvent(event）.png](https://upload-images.jianshu.io/upload_images/5363507-6dd9ca20e864ca72.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 最后做了一张图
![事件分发.png](https://upload-images.jianshu.io/upload_images/5363507-4c1e1edc8cf6a571.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

