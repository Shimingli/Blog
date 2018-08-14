---
title: 安卓invalidate、postInvalidate、requestLayout源码分析
date: 2018-08-07 15:45:48
tags: [源码解析,invalidate,postInvalidate ,requestLayout]  
---

*  最近在撸`Golang`有点上火了，来整理下安卓源码资料☺☺☺
*  分析结果基于`Audroid API 26`
<!--  more  -->
#### `requestLayout()`源码分析
* 假如在一个页面上有个按钮，点击按钮就对一个 `view.requestLayout()`,这个 `view` 执行的方法如下：
```
InvalidateTextView------onMeasure
InvalidateTextView------onMeasure
InvalidateTextView-------layout
InvalidateTextView--------onLayout
InvalidateTextView----------draw
InvalidateTextView------------onDraw
```
* `view.requestLayout()`  方法的详情
```
  @CallSuper
    public void requestLayout() {
        // 清除绘制的缓存
        if (mMeasureCache != null) mMeasureCache.clear();

        if (mAttachInfo != null && mAttachInfo.mViewRequestingLayout == null) {
            //只有在布局逻辑中触发请求，如果这是请求它的视图，而不是其父层次结构中的视图
            ViewRootImpl viewRoot = getViewRootImpl();
            //如果连续请求两次，其中一次自动返回！
            if (viewRoot != null && viewRoot.isInLayout()) {
                if (!viewRoot.requestLayoutDuringLayout(this)) {
                    return;
                }
            }
            mAttachInfo.mViewRequestingLayout = this;
        }
       //todo   为当前view设置标记位 PFLAG_FORCE_LAYOUT
        mPrivateFlags |= PFLAG_FORCE_LAYOUT;
        mPrivateFlags |= PFLAG_INVALIDATED;

        if (mParent != null && !mParent.isLayoutRequested()) {
           //   todo  向父容器请求布局 这里是向父容器请求布局，即调用父容器的requestLayout方法，为父容器添加PFLAG_FORCE_LAYOUT标记位，而父容器又会调用它的父容器的requestLayout方法，即requestLayout事件层层向上传递，直到DecorView，即根View，而根View又会传递给ViewRootImpl，也即是说子View的requestLayout事件，最终会被ViewRootImpl接收并得到处理
            mParent.requestLayout();
        }
        if (mAttachInfo != null && mAttachInfo.mViewRequestingLayout == this) {
            mAttachInfo.mViewRequestingLayout = null;
        }
    }
```
*  1、如果缓存不为`null`,清除绘制的缓存
```
        if (mMeasureCache != null) mMeasureCache.clear();
```

*  2、这里判断了是否在` layout`，如果是，就返回，也就可以理解为： 如果连续请求两次，并且其中的一次正在`layout`中，其中一次返回！这样做是节约性能
```
   if (mAttachInfo != null && mAttachInfo.mViewRequestingLayout == null) {
            //只有在布局逻辑中触发请求，如果这是请求它的视图，而不是其父层次结构中的视图
            ViewRootImpl viewRoot = getViewRootImpl();
            //如果连续请求两次，其中一次自动返回！
            if (viewRoot != null && viewRoot.isInLayout()) {
                if (!viewRoot.requestLayoutDuringLayout(this)) {
                    return;
                }
            }
            mAttachInfo.mViewRequestingLayout = this;
        }
```
*  3、  为当前`view`设置标记位` PFLAG_FORCE_LAYOUT`,关于 `|=`符号：`a|=b`的意思就是把a和b按位或然后赋值给a 按位或的意思就是先把a和b都换成2进制，然后用或操作，相当于`a=a|b`
```
   mPrivateFlags |= PFLAG_FORCE_LAYOUT;
   mPrivateFlags |= PFLAG_INVALIDATED;
```
*  4、向父容器请求布局，即调用`ViewGroup`父容器的`requestLayout()`方法，为父容器添加`PFLAG_FORCE_LAYOUT`标记位，而父容器又会调用它的父容器的`requestLayout()`方法，即`requestLayout()`事件层层向上传递，直到`DecorView`，即根`View`，而根`View`又会传递给`ViewRootImpl，`也即是说子View的`requestLayout()f`事件，最终会被`ViewRootImpl.requestLayout()`接收并得到处理
```
      if (mParent != null && !mParent.isLayoutRequested()) {
            mParent.requestLayout();
        }
```

* 5、`ViewRootImpl.requestLayout()`方法详情

   ```
  @Override
    public void requestLayout() {
        if (!mHandlingLayoutInLayoutRequest) {
            // 检查是否在主线程，不在的话，抛出异常
            checkThread();
            mLayoutRequested = true;
            scheduleTraversals();
        }
    }

   ```
     *  1、检查是否在主线程，不在的话，抛出异常` checkThread();`    
     ```
    void checkThread() {
        if (mThread != Thread.currentThread()) {
            throw new CalledFromWrongThreadException(
                    "Only the original thread that created a view hierarchy can touch its views.");
        }
    }
    ```
    *  2 、最终走到这个方法来`ViewRootImpl.scheduleTraversals()`,在其中可以看到一行非常有意思的代码
`  mChoreographer.postCallback( Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);` ,其中有个对象`mTraversalRunnable`,这样下去就会重新的测量、布局和绘制；具体的流程可以看这篇文章[Android源码分析(View的绘制流程)](https://www.jianshu.com/p/b63c6afa1844)
    ```
       // requestLayout()  会调用这个方法 
    void scheduleTraversals() {
        if (!mTraversalScheduled) {
            mTraversalScheduled = true;
            mTraversalBarrier = mHandler.getLooper().getQueue().postSyncBarrier();
            // 最终调用的是这个方法
            mChoreographer.postCallback(
                    Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);
            if (!mUnbufferedInputDispatch) {
                scheduleConsumeBatchedInput();
            }
            notifyRendererOfFramePending();
            pokeDrawLockIfNeeded();
        }
    }
   ```

*  有个问题，我先抛出结论，`requessLayout() 、invalidate()、postInvalidate()`最终的底层调用的是 `ViewRootImpl.scheduleTraversals()`的方法,为什么仅仅`requessLayout()`才会执行`onMeasure()  onLayout()  onDraw()`这几个方法？
*  关于`view.measure()`方法:在前面我们知道 `  mPrivateFlags |= PFLAG_FORCE_LAYOUT` 所以 `forceLayout = true`,也就是会执行`  onMeasure(widthMeasureSpec, heightMeasureSpec);`,同时执行完了以后呢，最后为标记位设置为`mPrivateFlags |=PFLAG_LAYOUT_REQUIRED`
```
 public final void measure(int widthMeasureSpec, int heightMeasureSpec) {
        ...
        // requestLayout的方法改变的  mPrivateFlags |= PFLAG_FORCE_LAYOUT; 所以 forceLayout = true
        final boolean forceLayout = (mPrivateFlags & PFLAG_FORCE_LAYOUT) == PFLAG_FORCE_LAYOUT;
        ...
        if (forceLayout || needsLayout) {
        ...
            if (cacheIndex < 0 || sIgnoreMeasureCache) {
                //最终会走到这方法来
                onMeasure(widthMeasureSpec, heightMeasureSpec);
                mPrivateFlags3 &= ~PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT;
            } 
            // 接着最后为标记位设置为PFLAG_LAYOUT_REQUIRED
            mPrivateFlags |= PFLAG_LAYOUT_REQUIRED;
        }
       ...
    }
```
* 关于`view.layout()`方法：判断标记位是否为`PFLAG_LAYOUT_REQUIRED`，如果有，则对该`View`进行布局,也就是走到`    onLayout(changed, l, t, r, b);`,最后清除标记` mPrivateFlags &= ~PFLAG_FORCE_LAYOUT;`
```
 @SuppressWarnings({"unchecked"})
    public void layout(int l, int t, int r, int b) {
        if ((mPrivateFlags3 & PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT) != 0) {
            //第二次调用这个方法，，，
            onMeasure(mOldWidthMeasureSpec, mOldHeightMeasureSpec);
            mPrivateFlags3 &= ~PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT;
        }

        int oldL = mLeft;
        int oldT = mTop;
        int oldB = mBottom;
        int oldR = mRight;

        boolean changed = isLayoutModeOptical(mParent) ?
                setOpticalFrame(l, t, r, b) : setFrame(l, t, r, b);
           //判断标记位是否为PFLAG_LAYOUT_REQUIRED，如果有，则对该View进行布局
        if (changed || (mPrivateFlags & PFLAG_LAYOUT_REQUIRED) == PFLAG_LAYOUT_REQUIRED) {
            onLayout(changed, l, t, r, b);

            if (shouldDrawRoundScrollbar()) {
                if(mRoundScrollbarRenderer == null) {
                    mRoundScrollbarRenderer = new RoundScrollbarRenderer(this);
                }
            } else {
                mRoundScrollbarRenderer = null;
            }
            //  onLayout方法完成后，清除PFLAG_LAYOUT_REQUIRED标记位
            mPrivateFlags &= ~PFLAG_LAYOUT_REQUIRED;
        }
    //  //最后清除PFLAG_FORCE_LAYOUT标记位
        mPrivateFlags &= ~PFLAG_FORCE_LAYOUT;
        mPrivateFlags3 |= PFLAG3_IS_LAID_OUT;
       ...
    }
```
* 以上就是 `requestLayout()`的分析的结果:`view`调用了这个方法，其实会从`view`树重新进行一次测量、布局、绘制这三个流程。
* 做了一张图
![requestLayout()的原理.jpg](https://upload-images.jianshu.io/upload_images/5363507-727eb9d2da018f43.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### `invalidate()源码分析`
* `view.invalidate()`;继承一个Textview，然后重写方法，设置一个but，同时请求方法，打印日志：请求一次的输出的结果
```
InvalidateTextView----------draw
InvalidateTextView------------onDraw
```
* 方法详情 : `view.invalidate()`
```
  public void invalidate() {
        invalidate(true);
    }
```
* 该视图的绘图缓存是否也应无效。对于完全无效,设置为true，但是如果视图的内容或维度没有改变，则可以设置为false。
```
   public void invalidate(boolean invalidateCache) {
        invalidateInternal(0, 0, mRight - mLeft, mBottom - mTop, invalidateCache, true);
    }
```
* `invalidateInternal()`方法详情：其实关键的方法就是`invalidateChild()`

   ```
  void invalidateInternal(int l, int t, int r, int b, boolean invalidateCache,
            boolean fullInvalidate) {
        if (mGhostView != null) {
            mGhostView.invalidate(true);
            return;
        }
        // 判断是否可见，是否在动画中，是否不是ViewGroup，三项满足一项，直接返回
        if (skipInvalidate()) {
            return;
        }
  //根据View的标记位来判断该子View是否需要重绘，假如View没有任何变化，那么就不需要重绘
        if ((mPrivateFlags & (PFLAG_DRAWN | PFLAG_HAS_BOUNDS)) == (PFLAG_DRAWN | PFLAG_HAS_BOUNDS)
                || (invalidateCache && (mPrivateFlags & PFLAG_DRAWING_CACHE_VALID) == PFLAG_DRAWING_CACHE_VALID)
                || (mPrivateFlags & PFLAG_INVALIDATED) != PFLAG_INVALIDATED
                || (fullInvalidate && isOpaque() != mLastIsOpaque)) {
            if (fullInvalidate) {
                mLastIsOpaque = isOpaque();
                mPrivateFlags &= ~PFLAG_DRAWN;
            }
            //设置PFLAG_DIRTY标记位
            mPrivateFlags |= PFLAG_DIRTY;

            if (invalidateCache) {
                mPrivateFlags |= PFLAG_INVALIDATED;
                mPrivateFlags &= ~PFLAG_DRAWING_CACHE_VALID;
            }
            //把需要重绘的区域传递给父容器
            // Propagate the damage rectangle to the parent view.
            final AttachInfo ai = mAttachInfo;
            final ViewParent p = mParent;
            if (p != null && ai != null && l < r && t < b) {
                final Rect damage = ai.mTmpInvalRect;
                damage.set(l, t, r, b);
                //调用父容器的方法，向上传递事件
                p.invalidateChild(this, damage);
            }

            // Damage the entire projection receiver, if necessary.
            // 损坏整个投影接收机，如果不需要。
            if (mBackground != null && mBackground.isProjected()) {
                final View receiver = getProjectionReceiver();
                if (receiver != null) {
                    receiver.damageInParent();
                }
            }
        }
    }
     ```

    *  1、判断是否可见，是否在动画中，是否不是`ViewGroup`，三项满足一项，直接返回，这个方法也可以知道，`invalidate()`针对的是`View`，而不是`ViewGroup`
    ```
  private boolean skipInvalidate() {
        return (mViewFlags & VISIBILITY_MASK) != VISIBLE && mCurrentAnimation == null &&
                (!(mParent instanceof ViewGroup) ||
                        !((ViewGroup) mParent).isViewTransitioning(this));
    }
   ``` 
     *  2、通过View的标记位来判断孩子View是否需要重新绘制，如果没有变化的话，那么就不需要重新绘制

     ```
   mPrivateFlags & (PFLAG_DRAWN | PFLAG_HAS_BOUNDS)) == (PFLAG_DRAWN | 
   PFLAG_HAS_BOUNDS)
                 || (invalidateCache && (mPrivateFlags & PFLAG_DRAWING_CACHE_VALID) == 
   PFLAG_DRAWING_CACHE_VALID)
                     || (mPrivateFlags & PFLAG_INVALIDATED) != PFLAG_INVALIDATED
                || (fullInvalidate && isOpaque() != mLastIsOpaque)
    ```
     *  3、需要重新绘制的区域传递给父容器，向上传递事件，记住在这里 `damage`这个变量肯定不为`null`,要不然在这个方法里面就会直接抛出空指针异常。
     ```
     p.invalidateChild(this, damage);
     ```  
     *  4、损坏整个投影接收机，如果不需要。` mBackground.isProjected()`： 这张画是否需要投影。
     ```
      if (mBackground != null && mBackground.isProjected()) {
                final View receiver = getProjectionReceiver();
                if (receiver != null) {
                    receiver.damageInParent();
                }
            }
     ```


* 关键的方法：`ViewRootImpl.invalidateChild(this, damage);`
```
  @Override
    public void invalidateChild(View child, Rect dirty) {
        invalidateChildInParent(null, dirty);
    }
```
*    invalidateChildInParent(null, dirty);进行了offset和union对坐标的调整，然后把dirty区域的信息保存在mDirty中，最后调用了`scheduleTraversals()`方法，触发View的工作流程，由于没有添加measure和layout的标记位，因此measure、layout流程不会执行，而是直接从`draw`流程开始.
       ``` 
      @Override
       public ViewParent invalidateChildInParent(int[] location, Rect dirty) {
        // 检查线程，不是ui线程，直接抛出异常
        checkThread();
         if (DEBUG_DRAW) Log.v(mTag, "Invalidate child: " + dirty);
 
        if (dirty == null) {
             invalidate();
            return null;
           } else if (dirty.isEmpty() && !mIsAnimating) {
              return null;
          }
   
          if (mCurScrollY != 0 || mTranslator != null) {
            mTempRect.set(dirty);
             dirty = mTempRect;
            if (mCurScrollY != 0) {
                // 将dirty中的坐标转化为父容器中的坐标，考虑mScrollX和mScrollY的影响
                dirty.offset(0, -mCurScrollY);
            }
            if (mTranslator != null) {
                mTranslator.translateRectInAppWindowToScreen(dirty);
            }
            if (mAttachInfo.mScalingRequired) {
                dirty.inset(-1, -1);
            }
         }
         //进行了offset和union对坐标的调整
         invalidateRectOnScreen(dirty);

         return null;
      }
      ```
      *  1、检查线程，不是ui线程，直接抛出异常.和`requestLayout()`一样的
        ```
           checkThread();
        ```
       *  2、如果是从 invalidate() 方法过来的话，那么dirty 肯定不为null  因为要是为null的话，前面调用方法的地方就抛出了空指针的异常
       ```
        if (dirty == null) {
            invalidate();
            return null;
        }
       ```
       * 3、通过将dx=0添加到其左、右坐标，并将 mCurScrollY 添加到其顶部和底部坐标来抵消矩形。

       ```
        dirty.offset(0, -mCurScrollY);
       ```
        *   4、进行了offset和union对坐标的调整
        ```
        invalidateRectOnScreen(dirty);
       ```


* 关于`invalidateRectOnScreen(dirty)`方法:最终的关键的方法： scheduleTraversals();
```
    private void invalidateRectOnScreen(Rect dirty) {
        final Rect localDirty = mDirty;
        if (!localDirty.isEmpty() && !localDirty.contains(dirty)) {
            mAttachInfo.mSetIgnoreDirtyState = true;
            mAttachInfo.mIgnoreDirtyState = true;
        }
        // Add the new dirty rect to the current one
        // 添加一个新的 dirty rect 给当前的Rect
        localDirty.union(dirty.left, dirty.top, dirty.right, dirty.bottom);
        // Intersect with the bounds of the window to skip
        // updates that lie outside of the visible region
        final float appScale = mAttachInfo.mApplicationScale;
        final boolean intersected = localDirty.intersect(0, 0,
                (int) (mWidth * appScale + 0.5f), (int) (mHeight * appScale + 0.5f));
        if (!intersected) {
            localDirty.setEmpty();
        }
        if (!mWillDrawSoon && (intersected || mIsAnimating)) {
            scheduleTraversals();
        }
    }
```
* 最终的关键的方法：` ViewRootImpl.scheduleTraversals();`，也就是会调用到这个对象`mTraversalRunnable`;也就是和`requessLaout()`最终调用的底层的方法一样，只不过对于`invalidate()`没有添加`measure()`和`layout()`的标记位，后面的流程也就不会执行！具体的流程可以看这篇文章[Android源码分析(View的绘制流程)](https://www.jianshu.com/p/b63c6afa1844)
* 该方法的调用会引起`View`树的重绘，常用于内部调用(比如 setVisiblity())或者需要刷新界面的时候,需要在主线程(即UI线程)中调用该方法，` invalidate`有多个重载方法，但最终都会调用`invalidateInternal`方法，在这个方法内部，进行了一系列的判断，判断`View`是否需要重绘，接着为该`View`设置标记位，然后把需要重绘的区域传递给父容器，即调用父容器的`invalidateChild`方法。
* 做了一张图
  ![invalidate()的原理.jpg](https://upload-images.jianshu.io/upload_images/5363507-f43b39cd42720e77.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### postInvalidate()的源码解析

* `view.postInvalidate()`继承一个Textview，然后重写方法，设置一个but，同时请求方法，打印日志：请求一次的输出的结果
```
InvalidateTextView----------draw
InvalidateTextView------------onDraw
```
* `view.postInvalidate()`详情,由于方法是public，也可以调用一个时间，延迟多久开始执行，这里是delayMilliseconds，毫秒
```
   public void postInvalidate() {
        postInvalidateDelayed(0);
    }
```
*  `view.postInvalidateDelayed（） ` 只有`attachInfo`不为null的时候才会继续执行，即只有确保视图被添加到窗口的时候才会通知view树重绘，因为这是一个异步方法，如果在视图还未被添加到窗口就通知重绘的话会出现错误，所以这样要做一下判断!
```
   public void postInvalidateDelayed(long delayMilliseconds) {
        final AttachInfo attachInfo = mAttachInfo;
        if (attachInfo != null) {
            attachInfo.mViewRootImpl.dispatchInvalidateDelayed(this, delayMilliseconds);
        }
    }
```

*  `ViewRootImpl.dispatchInvalidateDelayed()` 用了Handler，发送了一个异步消息到主线程，显然这里发送的是MSG_INVALIDATE，即通知主线程刷新视图
```
  /**
     * 用了Handler，发送了一个异步消息到主线程，显然这里发送的是`MSG_INVALIDATE`，即通知主线程刷新视图
      * @param view  只有 postInvalidate() 使用了handler 来发送消息
     * @param delayMilliseconds
     */
    public void dispatchInvalidateDelayed(View view, long delayMilliseconds) {
        Message msg = mHandler.obtainMessage(MSG_INVALIDATE, view);
        mHandler.sendMessageDelayed(msg, delayMilliseconds);
    }
```
* 通知对象去` invalidate()` ，底层也是调用的是` invalidate()`，只不过使用了`mHandler`发送消息,在这里就发送到主线程了，去调用`invalidate()`方法
```
      case MSG_INVALIDATE:
                    //通知对象去 invalidate ，底层也是调用的是 invalidate，只不过使用了handler发送消息
                    ((View) msg.obj).invalidate();
                    break;0
```
*   方法的解释 ：`postInvalidate`是在非UI线程中调用,但是底层使用的是 `invalidate()`,通过` ViewRootImpl的内部handler： ViewRootHandler`发送的消息，但是也可以在 主线程中使用，如果在强制在主线程中使用，内部有个 `handler` 在工作，是不是显得有点浪费 ，对吧！
* `postInvalidate()`这个方法也可以主线程中使用
* 做了一张图
![postInvalidate()的原理.jpg](https://upload-images.jianshu.io/upload_images/5363507-a267e92df57adf3e.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 最后说明几点
   *   `invalidate()、postInvalidate()、requestLayout()`,最底层处调用的是`viewRootImpl.scheduleTraversals()` 这个方法，`requestLayout`由于设置了`measure`和`layout`的标记位，所以`requestLayout`可以重新走一次绘制的流程
  *  `postInvalidate()` 底层通过`Handler`把非UI线程的工作，调用的是`invalidate()`.
  *  `invalidate()、requestLayout()`,方法都检查了是否在`UI`线程,不在的话，直接抛出异常，所以他们只能在UI线程中使用，`postInvalidate()`可以在UI线程和非UI线程中使用。
  * `view`自身不在适合某个区域，比如说``LayoutParams`发生了改变，需要对其重新的测量、布局和绘制的三个流程，那么使用这个方法最合适`requestLayout()`。
  *  如果说是在刷新当前的view，是当前的view进行重新的绘制，不会进行测量、布局流程。仅仅需要某个`view`重新绘制而不需要测量，使用`invalidate()方法往往比requestLayout()方法更高效`
  


