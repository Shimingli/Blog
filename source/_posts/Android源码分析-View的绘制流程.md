---
title: 'Android源码分析:View的绘制流程'
date: 2018-06-08 16:08:28
tags: [Android源码,View,绘制]
---

* 本篇文章已授权微信公众号 guolin_blog （郭霖）独家发布 
## 源码基于安卓8.0分析结果
* `View`是何时开始绘制的?`Activity`走了`onCreate`方法吗？这篇文章就是从程序的入口`ActivityThread`入口程序，去解释`View中的measure()方法`、`View中的layout`、`View中的draw`怎么开始调用的，非常有意思！虽然好多的技术文档，在半个月前已经做好了，这篇文章，对我自己来讲的话，是个很好的复习~~
* 为了更好地阐述着这篇文章，我这里就直接抛出结论了，为啥会这样的，在下篇文章会讲到，这里就记住一点，在`Activity onResume`后，调用了`View onAttachedToWindow` 才会开始`View measure `
<!--  more  -->
![Activity的生命周期和View的生命周期.jpg](https://upload-images.jianshu.io/upload_images/5363507-9c0147257f0e015b.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 为什么会这样子？先看`ActivityThread`类里面有个内部`private class H extends Handler `这就是系统的`Handler`，具体分析请看[Android源码分析（Handler机制）](https://www.jianshu.com/p/a2c53e96cae6),里面有个`case RESUME_ACTIVITY`,获取焦点
```
      case RESUME_ACTIVITY:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "activityResume");
                    SomeArgs args = (SomeArgs) msg.obj;
                    handleResumeActivity((IBinder) args.arg1, true, args.argi1 != 0, true,
                            args.argi3, "RESUME_ACTIVITY");
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
       break;
```
* handleResumeActivity()方法,这里只截取了关键的代码
```
 final void handleResumeActivity(IBinder token, boolean clearHide, boolean isForward, boolean reallyResume, int seq, String reason) {
        ActivityClientRecord r = mActivities.get(token);
        if (!checkAndUpdateLifecycleSeq(seq, r, "resumeActivity")) {
            return;
        }
        mSomeActivitiesChanged = true;
        //在这里执行performResumeActivity的方法中会执行Activity的onResume()方法
        r = performResumeActivity(token, clearHide, reason);
        if (r != null) {
            final Activity a = r.activity;

            if (localLOGV) Slog.v(
                    TAG, "Resume " + r + " started activity: " +
                            a.mStartedActivity + ", hideForNow: " + r.hideForNow
                            + ", finished: " + a.mFinished);
            final int forwardBit = isForward ?
                    WindowManager.LayoutParams.SOFT_INPUT_IS_FORWARD_NAVIGATION : 0;
            boolean willBeVisible = !a.mStartedActivity;
            if (!willBeVisible) {
                try {
                    willBeVisible = ActivityManager.getService().willActivityBeVisible(
                            a.getActivityToken());
                } catch (RemoteException e) {
                    throw e.rethrowFromSystemServer();
                }
            }
            //PhoneWindow在这里获取到
            if (r.window == null && !a.mFinished && willBeVisible) {
                r.window = r.activity.getWindow();
                //DecorView在这里获取到
                View decor = r.window.getDecorView();
                decor.setVisibility(View.INVISIBLE);
                //获取ViewManager对象，在这里getWindowManager()实质上获取的是ViewManager的子类对象WindowManager
                // TODO: 2018/5/24 WindowManager
                ViewManager wm = a.getWindowManager();
                WindowManager.LayoutParams l = r.window.getAttributes();
                a.mDecor = decor;
                l.type = WindowManager.LayoutParams.TYPE_BASE_APPLICATION;
                l.softInputMode |= forwardBit;
                if (r.mPreserveWindow) {
                    a.mWindowAdded = true;
                    r.mPreserveWindow = false;
                    // Normally the ViewRoot sets up callbacks with the Activity
                    // in addView->ViewRootImpl#setView. If we are instead reusing
                    // the decor view we have to notify the view root that the
                    // callbacks may have changed.
                    //获取ViewRootImpl对象
                    ViewRootImpl impl = decor.getViewRootImpl();
                    if (impl != null) {
                        impl.notifyChildRebuilt();
                    }
                }
        }
}
```
* 第一点分析得出`performResumeActivity()`肯定先于`wm.addView(decor, l);`执行的~这也是为啥我们 `Activity`先获取焦点了，才去绘制`View`
* performResumeActivity(),可以得出调用的是`r.activity.performResume();`
![performResumeActivity](https://upload-images.jianshu.io/upload_images/5363507-c61528fcbe49a9b3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 关于`r.activity.performResume();`；这里也可以，看出，在activity 中的fragment获取焦点要晚于activity，虽然这是常识。注意这个方法` mInstrumentation.callActivityOnResume(this);`;然后才会执行`onPostResume`;这也就是为什么，Activity先获取焦点，后执行`onPostResume()`;
```
 final void performResume() {
        performRestart();
        mInstrumentation.callActivityOnResume(this);
        mCalled = false;
       //这里也可以，看出，在activity 中的fragment获取焦点要晚于activity，虽然这是常识
        mFragments.dispatchResume();
        mFragments.execPendingActions();
        onPostResume();
    }
```
* 关注这个方法`mInstrumentation.callActivityOnResume(this)`;果然不出所料，这里执行了`activity.onResume();`;
![callActivityOnResume.png](https://upload-images.jianshu.io/upload_images/5363507-06a0a0bfd5bd0ebf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 既然在上面知道了，activity 获取焦点，会在上面执行，那么View的绘制就会在下面的函数中进行。
   *  1、获取PhoneWindow；  `activity.getWindow()`,`Window`类的唯一子类
    *  2、获取`window.getDecorView()`;DecorView,PhoneWindow的内部类，`private final class DecorView extends FrameLayout` ，安卓的事件分发和它密切相关[Android源码分析（事件传递）](https://www.jianshu.com/p/f7e3a14daf51),也就是从`Activity` 传递到 `ViewGroup`的过程~~
  *  3、获取` ViewManager wm = a.getWindowManager();`，其实也就是`activity.getWindowManager()`,也就是获取的是`ViewManager`的子类对象`WindowManager`,这里的知道`WindowManager`其实也是一个接口.
![image.png](https://upload-images.jianshu.io/upload_images/5363507-14b4c1d369701373.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
   * 4、  `wm.addView(decor, l);`,也就是到这里来了，`WindowManager.addView(decor,l)`.
```
 //PhoneWindow在这里获取到
            if (r.window == null && !a.mFinished && willBeVisible) {
                r.window = r.activity.getWindow();
                //DecorView在这里获取到
                View decor = r.window.getDecorView();
                decor.setVisibility(View.INVISIBLE);
                //获取ViewManager对象，在这里getWindowManager()实质上获取的是ViewManager的子类对象WindowManager
                // TODO: 2018/5/24 WindowManager
                ViewManager wm = a.getWindowManager();
                WindowManager.LayoutParams l = r.window.getAttributes();
                a.mDecor = decor;
                l.type = WindowManager.LayoutParams.TYPE_BASE_APPLICATION;
                l.softInputMode |= forwardBit;
                if (r.mPreserveWindow) {
                    a.mWindowAdded = true;
                    r.mPreserveWindow = false;
                    // Normally the ViewRoot sets up callbacks with the Activity
                    // in addView->ViewRootImpl#setView. If we are instead reusing
                    // the decor view we have to notify the view root that the
                    // callbacks may have changed.
                    //获取ViewRootImpl对象
                    ViewRootImpl impl = decor.getViewRootImpl();
                    if (impl != null) {
                        impl.notifyChildRebuilt();
                    }
                if (!a.mWindowAdded) {
                        a.mWindowAdded = true;
                        //在这里WindowManager将DecorView添加到PhoneWindow中
                        wm.addView(decor, l);
                    }
                }
```
* 分析到这里来了，会通过`WindowManager.addView(decor,l)`.我们需要去找`WindowManager`的实现。`WindowManagerImpl`;
```
public final class WindowManagerImpl implements WindowManager {
    private final WindowManagerGlobal mGlobal = WindowManagerGlobal.getInstance();
   @Override
    public void addView(@NonNull View view, @NonNull ViewGroup.LayoutParams params) {
        applyDefaultToken(params);
        mGlobal.addView(view, params, mContext.getDisplay(), mParentWindow);
    }
}
```
* 去寻找`WindowManagerGlobal`的`addView()`方法。这里有个单利模式,在源码好多地方使用的单利模式都是这样，并没有进行双重判断，在老牌的图片加载框架`ImageLoader`也是这样获取单利对象，如果想了解更多设计模式的姿势，可以看这片文章[二十三种设计模式](https://www.jianshu.com/p/4e01479b6a2c).
```
  public static WindowManagerGlobal getInstance() {
        synchronized (WindowManagerGlobal.class) {
            if (sDefaultWindowManager == null) {
                sDefaultWindowManager = new WindowManagerGlobal();
            }
            return sDefaultWindowManager;
        }
    }
```
* 在这里！就是`WindowManagerGlobal.addView() `的关键的方法，我做了两个注释，一个是` view.setLayoutParams(wparams);`,这个方法非常有意思，最近在研究ViewGroup的源码，发现不论什么情况下，View或者是ViewGroup都会有两次测量，这里是根本的原因，我先给结论。
api26：执行2次onMeasure、1次onLayout、1次onDraw。
api25-24：执行2次onMeasure、2次onLayout、1次onDraw，
api23-21：执行3次onMeasure、2次onLayout、1次onDraw，
api19-16：执行2次onMeasure、2次onLayout、1次onDraw，
API等级24：Android 7.0 Nougat
API等级25：Android 7.1 Nougat
API等级26：Android 8.0 Oreo
API等级27：Android 8.1 Oreo
后续我会做一篇文章详细解释下，为什么会这样，这里不过多的解释了，自提一句，非常有意思的代码！以前还会有两次的layout，说明谷歌也在优化安卓 `framework`。todo
```
  public void addView(View view, ViewGroup.LayoutParams params,Display display, Window parentWindow) {
     ...
      root = new ViewRootImpl(view.getContext(), display);
            //view setLLayoutParams()在这里
      view.setLayoutParams(wparams);
      try {
                // TODO: 2018/6/4  这里呢？就是ViewRootImpl 调用的setView的方法，就在这里
                root.setView(view, wparams, panelParentView);
            } catch (RuntimeException e) {
                // BadTokenException or InvalidDisplayException, clean up.
                if (index >= 0) {
                    removeViewLocked(index, true);
                }
                throw e;
            }
```
* ok，现在继续的关注这个方法`ViewRootImpl.setView(view, wparams, panelParentView)`
```
  public void setView(View view, WindowManager.LayoutParams attrs, View panelParentView) {
        synchronized (this) {   try {
                 // TODO: 2018/6/4 这里传入的attrs 决定了View 或者是ViewGroup是否会onMeasure 两次
                mWindowAttributes.copyFrom(attrs);
                } catch (RemoteException e) {
                   
                    // TODO: 2018/5/24 就会调动这里的来
                    unscheduleTraversals();
                } finally {
                    if (restore) {
                        attrs.restore();
                    } if (res < WindowManagerGlobal.ADD_OKAY) {
                    
                    // TODO: 2018/5/24 就会调动这里的来
                    unscheduleTraversals();}
                }
```
* unscheduleTraversals(),没有Activity获取焦点的时候，这个方法肯定会执行
```
    void unscheduleTraversals() {
        if (mTraversalScheduled) {
            mTraversalScheduled = false;
            mHandler.getLooper().getQueue().removeSyncBarrier(mTraversalBarrier);
            mChoreographer.removeCallbacks(
                    Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);
        }
    }
```
*  关注mTraversalRunnable对象
```
  final class TraversalRunnable implements Runnable {
        @Override
        public void run() {
            doTraversal();
        }
    }
```
* doTraversal()方法,Traversal翻译过来就是遍历的意思~~
```

    // TODO: 2018/5/24  到这里来了 ---->     Traversal 遍历
    void doTraversal() {
        if (mTraversalScheduled) {
            mTraversalScheduled = false;
            mHandler.getLooper().getQueue().removeSyncBarrier(mTraversalBarrier);
            if (mProfile) {
                Debug.startMethodTracing("ViewAncestor");
            }
            performTraversals();
            if (mProfile) {
                Debug.stopMethodTracing();
                mProfile = false;
            }
        }
    }
```
## `performTraversals`这里就是整个View绘制的开始，所有的绘制，都会从这里开始，虽然这个方法代码有点多，但是关键的地方我都做了注释，下面一步一步的分析
```
   /**
     * 最终会调动到这里
     * ViewRootImpl的performTraversals()方法完成具体的视图绘制流程
     */
    private void performTraversals() {
        // cache mView since it is used so much below...
        //mView 就是DecorView根布局
        final View host = mView;

        if (DBG) {
            System.out.println("======================================");
            System.out.println("performTraversals");
            host.debug();
        }
        //如果host=null  或者是mAdded=false 直接就return了
        if (host == null || !mAdded)
            return;
        //是否正在遍历
        mIsInTraversal = true;
        //是否马上需要绘制View
        mWillDrawSoon = true;
        boolean windowSizeMayChange = false;
        boolean newSurface = false;
        boolean surfaceChanged = false;
        /**
         * 这个WindowMananger 这里标记了 是否是需要 onMeasure 两次 哈哈
         */
        // TODO: 2018/6/4
        WindowManager.LayoutParams lp = mWindowAttributes;
        /*
        顶层视图DecorView所需要的窗口的宽度和高度
         */
        int desiredWindowWidth;
        int desiredWindowHeight;

        final int viewVisibility = getHostVisibility();
        final boolean viewVisibilityChanged = !mFirst
                && (mViewVisibility != viewVisibility || mNewSurfaceNeeded);
        final boolean viewUserVisibilityChanged = !mFirst &&
                ((mViewVisibility == View.VISIBLE) != (viewVisibility == View.VISIBLE));

        WindowManager.LayoutParams params = null;
        if (mWindowAttributesChanged) {
            mWindowAttributesChanged = false;
            surfaceChanged = true;
            params = lp;
        }
        CompatibilityInfo compatibilityInfo =
                mDisplay.getDisplayAdjustments().getCompatibilityInfo();
        if (compatibilityInfo.supportsScreen() == mLastInCompatMode) {
            params = lp;
            mFullRedrawNeeded = true;
            mLayoutRequested = true;
            if (mLastInCompatMode) {
                params.privateFlags &= ~WindowManager.LayoutParams.PRIVATE_FLAG_COMPATIBLE_WINDOW;
                mLastInCompatMode = false;
            } else {
                params.privateFlags |= WindowManager.LayoutParams.PRIVATE_FLAG_COMPATIBLE_WINDOW;
                mLastInCompatMode = true;
            }
        }

        mWindowAttributesChangesFlag = 0;

        Rect frame = mWinFrame;
        //在构造方法中mFirst已经设置为true，表示是否是第一次绘制DecorView
        if (mFirst) {
            mFullRedrawNeeded = true;
            mLayoutRequested = true;

            final Configuration config = mContext.getResources().getConfiguration();
            // TODO: 2018/5/25 注意这个方法内部做了什么
            /*
             return lp.type == TYPE_STATUS_BAR_PANEL
                || lp.type == TYPE_INPUT_METHOD
                || lp.type == TYPE_VOLUME_OVERLAY;
             */
            // 如果窗口的类型是有状态栏的，那么顶层视图DecorView所需要的窗口的宽度和高度
            //就是除了状态栏
            if (shouldUseDisplaySize(lp)) {
                // NOTE -- system code, won't try to do compat mode.
                Point size = new Point();
                mDisplay.getRealSize(size);
                desiredWindowWidth = size.x;
                desiredWindowHeight = size.y;
            } else {
                //否者顶层视图DecorView所需要的窗口的宽度和高度就是整个屏幕的宽度
                desiredWindowWidth = dipToPx(config.screenWidthDp);
                desiredWindowHeight = dipToPx(config.screenHeightDp);
            }

            // We used to use the following condition to choose 32 bits drawing caches:
            // PixelFormat.hasAlpha(lp.format) || lp.format == PixelFormat.RGBX_8888
            // However, windows are now always 32 bits by default, so choose 32 bits
            mAttachInfo.mUse32BitDrawingCache = true;
            mAttachInfo.mHasWindowFocus = false;
            mAttachInfo.mWindowVisibility = viewVisibility;
            mAttachInfo.mRecomputeGlobalAttributes = false;
            mLastConfigurationFromResources.setTo(config);
            mLastSystemUiVisibility = mAttachInfo.mSystemUiVisibility;
            // Set the layout direction if it has not been set before (inherit is the default)
            if (mViewLayoutDirectionInitial == View.LAYOUT_DIRECTION_INHERIT) {
                host.setLayoutDirection(config.getLayoutDirection());
            }
            host.dispatchAttachedToWindow(mAttachInfo, 0);
            mAttachInfo.mTreeObserver.dispatchOnWindowAttachedChange(true);
            dispatchApplyInsets(host);
            //Log.i(mTag, "Screen on initialized: " + attachInfo.mKeepScreenOn);

        } else {
            desiredWindowWidth = frame.width();
            desiredWindowHeight = frame.height();
            if (desiredWindowWidth != mWidth || desiredWindowHeight != mHeight) {
                if (DEBUG_ORIENTATION) Log.v(mTag, "View " + host + " resized to: " + frame);
                mFullRedrawNeeded = true;
                mLayoutRequested = true;
                windowSizeMayChange = true;
            }
        }

        if (viewVisibilityChanged) {
            mAttachInfo.mWindowVisibility = viewVisibility;
            host.dispatchWindowVisibilityChanged(viewVisibility);
            if (viewUserVisibilityChanged) {
                host.dispatchVisibilityAggregated(viewVisibility == View.VISIBLE);
            }
            if (viewVisibility != View.VISIBLE || mNewSurfaceNeeded) {
                endDragResizing();
                destroyHardwareResources();
            }
            if (viewVisibility == View.GONE) {
                // After making a window gone, we will count it as being
                // shown for the first time the next time it gets focus.
                mHasHadWindowFocus = false;
            }
        }

        // Non-visible windows can't hold accessibility focus.
        if (mAttachInfo.mWindowVisibility != View.VISIBLE) {
            host.clearAccessibilityFocus();
        }

        // Execute enqueued actions on every traversal in case a detached view enqueued an action
        getRunQueue().executeActions(mAttachInfo.mHandler);

        boolean insetsChanged = false;

        boolean layoutRequested = mLayoutRequested && (!mStopped || mReportNextDraw);
        if (layoutRequested) {

            final Resources res = mView.getContext().getResources();

            if (mFirst) {
                // make sure touch mode code executes by setting cached value
                // to opposite of the added touch mode.
                mAttachInfo.mInTouchMode = !mAddedTouchMode;
                ensureTouchModeLocally(mAddedTouchMode);
            } else {
                if (!mPendingOverscanInsets.equals(mAttachInfo.mOverscanInsets)) {
                    insetsChanged = true;
                }
                if (!mPendingContentInsets.equals(mAttachInfo.mContentInsets)) {
                    insetsChanged = true;
                }
                if (!mPendingStableInsets.equals(mAttachInfo.mStableInsets)) {
                    insetsChanged = true;
                }
                if (!mPendingVisibleInsets.equals(mAttachInfo.mVisibleInsets)) {
                    mAttachInfo.mVisibleInsets.set(mPendingVisibleInsets);
                    if (DEBUG_LAYOUT) Log.v(mTag, "Visible insets changing to: "
                            + mAttachInfo.mVisibleInsets);
                }
                if (!mPendingOutsets.equals(mAttachInfo.mOutsets)) {
                    insetsChanged = true;
                }
                if (mPendingAlwaysConsumeNavBar != mAttachInfo.mAlwaysConsumeNavBar) {
                    insetsChanged = true;
                }
                if (lp.width == ViewGroup.LayoutParams.WRAP_CONTENT
                        || lp.height == ViewGroup.LayoutParams.WRAP_CONTENT) {
                    windowSizeMayChange = true;

                    if (shouldUseDisplaySize(lp)) {
                        // NOTE -- system code, won't try to do compat mode.
                        Point size = new Point();
                        mDisplay.getRealSize(size);
                        desiredWindowWidth = size.x;
                        desiredWindowHeight = size.y;
                    } else {
                        Configuration config = res.getConfiguration();
                        desiredWindowWidth = dipToPx(config.screenWidthDp);
                        desiredWindowHeight = dipToPx(config.screenHeightDp);
                    }
                }
            }

            // Ask host how big it wants to be
            windowSizeMayChange |= measureHierarchy(host, lp, res,
                    desiredWindowWidth, desiredWindowHeight);
        }

        if (collectViewAttributes()) {
            params = lp;
        }
        if (mAttachInfo.mForceReportNewAttributes) {
            mAttachInfo.mForceReportNewAttributes = false;
            params = lp;
        }

        if (mFirst || mAttachInfo.mViewVisibilityChanged) {
            mAttachInfo.mViewVisibilityChanged = false;
            int resizeMode = mSoftInputMode &
                    WindowManager.LayoutParams.SOFT_INPUT_MASK_ADJUST;
            // If we are in auto resize mode, then we need to determine
            // what mode to use now.
            if (resizeMode == WindowManager.LayoutParams.SOFT_INPUT_ADJUST_UNSPECIFIED) {
                final int N = mAttachInfo.mScrollContainers.size();
                for (int i=0; i<N; i++) {
                    if (mAttachInfo.mScrollContainers.get(i).isShown()) {
                        resizeMode = WindowManager.LayoutParams.SOFT_INPUT_ADJUST_RESIZE;
                    }
                }
                if (resizeMode == 0) {
                    resizeMode = WindowManager.LayoutParams.SOFT_INPUT_ADJUST_PAN;
                }
                if ((lp.softInputMode &
                        WindowManager.LayoutParams.SOFT_INPUT_MASK_ADJUST) != resizeMode) {
                    lp.softInputMode = (lp.softInputMode &
                            ~WindowManager.LayoutParams.SOFT_INPUT_MASK_ADJUST) |
                            resizeMode;
                    params = lp;
                }
            }
        }

        if (params != null) {
            if ((host.mPrivateFlags & View.PFLAG_REQUEST_TRANSPARENT_REGIONS) != 0) {
                if (!PixelFormat.formatHasAlpha(params.format)) {
                    params.format = PixelFormat.TRANSLUCENT;
                }
            }
            mAttachInfo.mOverscanRequested = (params.flags
                    & WindowManager.LayoutParams.FLAG_LAYOUT_IN_OVERSCAN) != 0;
        }

        if (mApplyInsetsRequested) {
            mApplyInsetsRequested = false;
            mLastOverscanRequested = mAttachInfo.mOverscanRequested;
            dispatchApplyInsets(host);
            if (mLayoutRequested) {
                // Short-circuit catching a new layout request here, so
                // we don't need to go through two layout passes when things
                // change due to fitting system windows, which can happen a lot.
                windowSizeMayChange |= measureHierarchy(host, lp,
                        mView.getContext().getResources(),
                        desiredWindowWidth, desiredWindowHeight);
            }
        }

        if (layoutRequested) {
            // Clear this now, so that if anything requests a layout in the
            // rest of this function we will catch it and re-run a full
            // layout pass.
            mLayoutRequested = false;
        }

        boolean windowShouldResize = layoutRequested && windowSizeMayChange
                && ((mWidth != host.getMeasuredWidth() || mHeight != host.getMeasuredHeight())
                || (lp.width == ViewGroup.LayoutParams.WRAP_CONTENT &&
                frame.width() < desiredWindowWidth && frame.width() != mWidth)
                || (lp.height == ViewGroup.LayoutParams.WRAP_CONTENT &&
                frame.height() < desiredWindowHeight && frame.height() != mHeight));
        windowShouldResize |= mDragResizing && mResizeMode == RESIZE_MODE_FREEFORM;

        // If the activity was just relaunched, it might have unfrozen the task bounds (while
        // relaunching), so we need to force a call into window manager to pick up the latest
        // bounds.
        windowShouldResize |= mActivityRelaunched;

        // Determine whether to compute insets.
        // If there are no inset listeners remaining then we may still need to compute
        // insets in case the old insets were non-empty and must be reset.
        final boolean computesInternalInsets =
                mAttachInfo.mTreeObserver.hasComputeInternalInsetsListeners()
                        || mAttachInfo.mHasNonEmptyGivenInternalInsets;

        boolean insetsPending = false;
        int relayoutResult = 0;
        boolean updatedConfiguration = false;

        final int surfaceGenerationId = mSurface.getGenerationId();

        final boolean isViewVisible = viewVisibility == View.VISIBLE;
        final boolean windowRelayoutWasForced = mForceNextWindowRelayout;
        if (mFirst || windowShouldResize || insetsChanged ||
                viewVisibilityChanged || params != null || mForceNextWindowRelayout) {
            mForceNextWindowRelayout = false;

            if (isViewVisible) {
                // If this window is giving internal insets to the window
                // manager, and it is being added or changing its visibility,
                // then we want to first give the window manager "fake"
                // insets to cause it to effectively ignore the content of
                // the window during layout.  This avoids it briefly causing
                // other windows to resize/move based on the raw frame of the
                // window, waiting until we can finish laying out this window
                // and get back to the window manager with the ultimately
                // computed insets.
                insetsPending = computesInternalInsets && (mFirst || viewVisibilityChanged);
            }

            if (mSurfaceHolder != null) {
                mSurfaceHolder.mSurfaceLock.lock();
                mDrawingAllowed = true;
            }

            boolean hwInitialized = false;
            boolean contentInsetsChanged = false;
            boolean hadSurface = mSurface.isValid();

            try {
                if (DEBUG_LAYOUT) {
                    Log.i(mTag, "host=w:" + host.getMeasuredWidth() + ", h:" +
                            host.getMeasuredHeight() + ", params=" + params);
                }

                if (mAttachInfo.mThreadedRenderer != null) {
                    // relayoutWindow may decide to destroy mSurface. As that decision
                    // happens in WindowManager service, we need to be defensive here
                    // and stop using the surface in case it gets destroyed.
                    if (mAttachInfo.mThreadedRenderer.pauseSurface(mSurface)) {
                        // Animations were running so we need to push a frame
                        // to resume them
                        mDirty.set(0, 0, mWidth, mHeight);
                    }
                    mChoreographer.mFrameInfo.addFlags(FrameInfo.FLAG_WINDOW_LAYOUT_CHANGED);
                }
                relayoutResult = relayoutWindow(params, viewVisibility, insetsPending);

                if (DEBUG_LAYOUT) Log.v(mTag, "relayout: frame=" + frame.toShortString()
                        + " overscan=" + mPendingOverscanInsets.toShortString()
                        + " content=" + mPendingContentInsets.toShortString()
                        + " visible=" + mPendingVisibleInsets.toShortString()
                        + " visible=" + mPendingStableInsets.toShortString()
                        + " outsets=" + mPendingOutsets.toShortString()
                        + " surface=" + mSurface);

                final Configuration pendingMergedConfig =
                        mPendingMergedConfiguration.getMergedConfiguration();
                if (pendingMergedConfig.seq != 0) {
                    if (DEBUG_CONFIGURATION) Log.v(mTag, "Visible with new config: "
                            + pendingMergedConfig);
                    performConfigurationChange(mPendingMergedConfiguration, !mFirst,
                            INVALID_DISPLAY /* same display */);
                    pendingMergedConfig.seq = 0;
                    updatedConfiguration = true;
                }

                final boolean overscanInsetsChanged = !mPendingOverscanInsets.equals(
                        mAttachInfo.mOverscanInsets);
                contentInsetsChanged = !mPendingContentInsets.equals(
                        mAttachInfo.mContentInsets);
                final boolean visibleInsetsChanged = !mPendingVisibleInsets.equals(
                        mAttachInfo.mVisibleInsets);
                final boolean stableInsetsChanged = !mPendingStableInsets.equals(
                        mAttachInfo.mStableInsets);
                final boolean outsetsChanged = !mPendingOutsets.equals(mAttachInfo.mOutsets);
                final boolean surfaceSizeChanged = (relayoutResult
                        & WindowManagerGlobal.RELAYOUT_RES_SURFACE_RESIZED) != 0;
                final boolean alwaysConsumeNavBarChanged =
                        mPendingAlwaysConsumeNavBar != mAttachInfo.mAlwaysConsumeNavBar;
                if (contentInsetsChanged) {
                    mAttachInfo.mContentInsets.set(mPendingContentInsets);
                    if (DEBUG_LAYOUT) Log.v(mTag, "Content insets changing to: "
                            + mAttachInfo.mContentInsets);
                }
                if (overscanInsetsChanged) {
                    mAttachInfo.mOverscanInsets.set(mPendingOverscanInsets);
                    if (DEBUG_LAYOUT) Log.v(mTag, "Overscan insets changing to: "
                            + mAttachInfo.mOverscanInsets);
                    // Need to relayout with content insets.
                    contentInsetsChanged = true;
                }
                if (stableInsetsChanged) {
                    mAttachInfo.mStableInsets.set(mPendingStableInsets);
                    if (DEBUG_LAYOUT) Log.v(mTag, "Decor insets changing to: "
                            + mAttachInfo.mStableInsets);
                    // Need to relayout with content insets.
                    contentInsetsChanged = true;
                }
                if (alwaysConsumeNavBarChanged) {
                    mAttachInfo.mAlwaysConsumeNavBar = mPendingAlwaysConsumeNavBar;
                    contentInsetsChanged = true;
                }
                if (contentInsetsChanged || mLastSystemUiVisibility !=
                        mAttachInfo.mSystemUiVisibility || mApplyInsetsRequested
                        || mLastOverscanRequested != mAttachInfo.mOverscanRequested
                        || outsetsChanged) {
                    mLastSystemUiVisibility = mAttachInfo.mSystemUiVisibility;
                    mLastOverscanRequested = mAttachInfo.mOverscanRequested;
                    mAttachInfo.mOutsets.set(mPendingOutsets);
                    mApplyInsetsRequested = false;
                    dispatchApplyInsets(host);
                }
                if (visibleInsetsChanged) {
                    mAttachInfo.mVisibleInsets.set(mPendingVisibleInsets);
                    if (DEBUG_LAYOUT) Log.v(mTag, "Visible insets changing to: "
                            + mAttachInfo.mVisibleInsets);
                }

                if (!hadSurface) {
                    if (mSurface.isValid()) {
                        // If we are creating a new surface, then we need to
                        // completely redraw it.  Also, when we get to the
                        // point of drawing it we will hold off and schedule
                        // a new traversal instead.  This is so we can tell the
                        // window manager about all of the windows being displayed
                        // before actually drawing them, so it can display then
                        // all at once.
                        newSurface = true;
                        mFullRedrawNeeded = true;
                        mPreviousTransparentRegion.setEmpty();

                        // Only initialize up-front if transparent regions are not
                        // requested, otherwise defer to see if the entire window
                        // will be transparent
                        if (mAttachInfo.mThreadedRenderer != null) {
                            try {
                                hwInitialized = mAttachInfo.mThreadedRenderer.initialize(
                                        mSurface);
                                if (hwInitialized && (host.mPrivateFlags
                                        & View.PFLAG_REQUEST_TRANSPARENT_REGIONS) == 0) {
                                    // Don't pre-allocate if transparent regions
                                    // are requested as they may not be needed
                                    mSurface.allocateBuffers();
                                }
                            } catch (OutOfResourcesException e) {
                                handleOutOfResourcesException(e);
                                return;
                            }
                        }
                    }
                } else if (!mSurface.isValid()) {
                    // If the surface has been removed, then reset the scroll
                    // positions.
                    if (mLastScrolledFocus != null) {
                        mLastScrolledFocus.clear();
                    }
                    mScrollY = mCurScrollY = 0;
                    if (mView instanceof RootViewSurfaceTaker) {
                        ((RootViewSurfaceTaker) mView).onRootViewScrollYChanged(mCurScrollY);
                    }
                    if (mScroller != null) {
                        mScroller.abortAnimation();
                    }
                    // Our surface is gone
                    if (mAttachInfo.mThreadedRenderer != null &&
                            mAttachInfo.mThreadedRenderer.isEnabled()) {
                        mAttachInfo.mThreadedRenderer.destroy();
                    }
                } else if ((surfaceGenerationId != mSurface.getGenerationId()
                        || surfaceSizeChanged || windowRelayoutWasForced)
                        && mSurfaceHolder == null
                        && mAttachInfo.mThreadedRenderer != null) {
                    mFullRedrawNeeded = true;
                    try {
                        // Need to do updateSurface (which leads to CanvasContext::setSurface and
                        // re-create the EGLSurface) if either the Surface changed (as indicated by
                        // generation id), or WindowManager changed the surface size. The latter is
                        // because on some chips, changing the consumer side's BufferQueue size may
                        // not take effect immediately unless we create a new EGLSurface.
                        // Note that frame size change doesn't always imply surface size change (eg.
                        // drag resizing uses fullscreen surface), need to check surfaceSizeChanged
                        // flag from WindowManager.
                        mAttachInfo.mThreadedRenderer.updateSurface(mSurface);
                    } catch (OutOfResourcesException e) {
                        handleOutOfResourcesException(e);
                        return;
                    }
                }

                final boolean freeformResizing = (relayoutResult
                        & WindowManagerGlobal.RELAYOUT_RES_DRAG_RESIZING_FREEFORM) != 0;
                final boolean dockedResizing = (relayoutResult
                        & WindowManagerGlobal.RELAYOUT_RES_DRAG_RESIZING_DOCKED) != 0;
                final boolean dragResizing = freeformResizing || dockedResizing;
                if (mDragResizing != dragResizing) {
                    if (dragResizing) {
                        mResizeMode = freeformResizing
                                ? RESIZE_MODE_FREEFORM
                                : RESIZE_MODE_DOCKED_DIVIDER;
                        startDragResizing(mPendingBackDropFrame,
                                mWinFrame.equals(mPendingBackDropFrame), mPendingVisibleInsets,
                                mPendingStableInsets, mResizeMode);
                    } else {
                        // We shouldn't come here, but if we come we should end the resize.
                        endDragResizing();
                    }
                }
                if (!USE_MT_RENDERER) {
                    if (dragResizing) {
                        mCanvasOffsetX = mWinFrame.left;
                        mCanvasOffsetY = mWinFrame.top;
                    } else {
                        mCanvasOffsetX = mCanvasOffsetY = 0;
                    }
                }
            } catch (RemoteException e) {
            }

            if (DEBUG_ORIENTATION) Log.v(
                    TAG, "Relayout returned: frame=" + frame + ", surface=" + mSurface);

            mAttachInfo.mWindowLeft = frame.left;
            mAttachInfo.mWindowTop = frame.top;

            // !!FIXME!! This next section handles the case where we did not get the
            // window size we asked for. We should avoid this by getting a maximum size from
            // the window session beforehand.
            if (mWidth != frame.width() || mHeight != frame.height()) {
                mWidth = frame.width();
                mHeight = frame.height();
            }

            if (mSurfaceHolder != null) {
                // The app owns the surface; tell it about what is going on.
                if (mSurface.isValid()) {
                    // XXX .copyFrom() doesn't work!
                    //mSurfaceHolder.mSurface.copyFrom(mSurface);
                    mSurfaceHolder.mSurface = mSurface;
                }
                mSurfaceHolder.setSurfaceFrameSize(mWidth, mHeight);
                mSurfaceHolder.mSurfaceLock.unlock();
                if (mSurface.isValid()) {
                    if (!hadSurface) {
                        mSurfaceHolder.ungetCallbacks();

                        mIsCreating = true;
                        SurfaceHolder.Callback callbacks[] = mSurfaceHolder.getCallbacks();
                        if (callbacks != null) {
                            for (SurfaceHolder.Callback c : callbacks) {
                                c.surfaceCreated(mSurfaceHolder);
                            }
                        }
                        surfaceChanged = true;
                    }
                    if (surfaceChanged || surfaceGenerationId != mSurface.getGenerationId()) {
                        SurfaceHolder.Callback callbacks[] = mSurfaceHolder.getCallbacks();
                        if (callbacks != null) {
                            for (SurfaceHolder.Callback c : callbacks) {
                                c.surfaceChanged(mSurfaceHolder, lp.format,
                                        mWidth, mHeight);
                            }
                        }
                    }
                    mIsCreating = false;
                } else if (hadSurface) {
                    mSurfaceHolder.ungetCallbacks();
                    SurfaceHolder.Callback callbacks[] = mSurfaceHolder.getCallbacks();
                    if (callbacks != null) {
                        for (SurfaceHolder.Callback c : callbacks) {
                            c.surfaceDestroyed(mSurfaceHolder);
                        }
                    }
                    mSurfaceHolder.mSurfaceLock.lock();
                    try {
                        mSurfaceHolder.mSurface = new Surface();
                    } finally {
                        mSurfaceHolder.mSurfaceLock.unlock();
                    }
                }
            }

            final ThreadedRenderer threadedRenderer = mAttachInfo.mThreadedRenderer;
            if (threadedRenderer != null && threadedRenderer.isEnabled()) {
                if (hwInitialized
                        || mWidth != threadedRenderer.getWidth()
                        || mHeight != threadedRenderer.getHeight()
                        || mNeedsRendererSetup) {
                    threadedRenderer.setup(mWidth, mHeight, mAttachInfo,
                            mWindowAttributes.surfaceInsets);
                    mNeedsRendererSetup = false;
                }
            }

            if (!mStopped || mReportNextDraw) {
                boolean focusChangedDueToTouchMode = ensureTouchModeLocally(
                        (relayoutResult&WindowManagerGlobal.RELAYOUT_RES_IN_TOUCH_MODE) != 0);
                if (focusChangedDueToTouchMode || mWidth != host.getMeasuredWidth()
                        || mHeight != host.getMeasuredHeight() || contentInsetsChanged ||
                        updatedConfiguration) {
                    // TODO: 2018/5/25 //获得view宽高的测量规格，
                    // TODO: 2018/5/25 mWidth和mHeight表示窗口的宽高，lp.widthhe和lp.height表示DecorView根布局宽和高
                    int childWidthMeasureSpec = getRootMeasureSpec(mWidth, lp.width);
                    int childHeightMeasureSpec = getRootMeasureSpec(mHeight, lp.height);

                    if (DEBUG_LAYOUT) Log.v(mTag, "Ooops, something changed!  mWidth="
                            + mWidth + " measuredWidth=" + host.getMeasuredWidth()
                            + " mHeight=" + mHeight
                            + " measuredHeight=" + host.getMeasuredHeight()
                            + " coveredInsetsChanged=" + contentInsetsChanged);

                    // Ask host how big it wants to be
                    // TODO: 2018/5/25  这里是第一步的  执行测量的操作
                    performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);

                    // Implementation of weights from WindowManager.LayoutParams
                    // We just grow the dimensions as needed and re-measure if
                    // needs be
                    int width = host.getMeasuredWidth();
                    int height = host.getMeasuredHeight();
                    boolean measureAgain = false;
                    /**
*指示额外空间的多少将被水平分配。
*与这些LayOutPARAMs关联的视图。如果视图指定0
*不应被拉伸。否则额外像素将被优先评估。
*在所有重量大于0的视图中。
*/                   // TODO: 2018/5/31  这里
                    // WindowManager.LayoutParams=lp;
                    if (lp.horizontalWeight > 0.0f) {
                        width += (int) ((mWidth - width) * lp.horizontalWeight);
                        childWidthMeasureSpec = MeasureSpec.makeMeasureSpec(width,
                                MeasureSpec.EXACTLY);
                        measureAgain = true;
                    }
                    if (lp.verticalWeight > 0.0f) {
                        height += (int) ((mHeight - height) * lp.verticalWeight);
                        childHeightMeasureSpec = MeasureSpec.makeMeasureSpec(height,
                                MeasureSpec.EXACTLY);
                        measureAgain = true;
                    }
                    // TODO: 2018/5/31  我知道这里为啥执行两次
                    if (measureAgain) {
                        if (DEBUG_LAYOUT) Log.v(mTag,
                                "And hey let's measure once more: width=" + width
                                        + " height=" + height);
                        performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);
                    }

                    layoutRequested = true;
                }
            }
        } else {
            // Not the first pass and no window/insets/visibility change but the window
            // may have moved and we need check that and if so to update the left and right
            // in the attach info. We translate only the window frame since on window move
            // the window manager tells us only for the new frame but the insets are the
            // same and we do not want to translate them more than once.
            maybeHandleWindowMove(frame);
        }

        final boolean didLayout = layoutRequested && (!mStopped || mReportNextDraw);
        boolean triggerGlobalLayoutListener = didLayout
                || mAttachInfo.mRecomputeGlobalAttributes;
        if (didLayout) {
            // TODO: 2018/5/25  执行布局操作
            performLayout(lp, mWidth, mHeight);

            // By this point all views have been sized and positioned
            // We can compute the transparent area

            if ((host.mPrivateFlags & View.PFLAG_REQUEST_TRANSPARENT_REGIONS) != 0) {
                // start out transparent
                // TODO: AVOID THAT CALL BY CACHING THE RESULT?
                host.getLocationInWindow(mTmpLocation);
                mTransparentRegion.set(mTmpLocation[0], mTmpLocation[1],
                        mTmpLocation[0] + host.mRight - host.mLeft,
                        mTmpLocation[1] + host.mBottom - host.mTop);

                host.gatherTransparentRegion(mTransparentRegion);
                if (mTranslator != null) {
                    mTranslator.translateRegionInWindowToScreen(mTransparentRegion);
                }

                if (!mTransparentRegion.equals(mPreviousTransparentRegion)) {
                    mPreviousTransparentRegion.set(mTransparentRegion);
                    mFullRedrawNeeded = true;
                    // reconfigure window manager
                    try {
                        mWindowSession.setTransparentRegion(mWindow, mTransparentRegion);
                    } catch (RemoteException e) {
                    }
                }
            }

            if (DBG) {
                System.out.println("======================================");
                System.out.println("performTraversals -- after setFrame");
                host.debug();
            }
        }

        if (triggerGlobalLayoutListener) {
            mAttachInfo.mRecomputeGlobalAttributes = false;
            mAttachInfo.mTreeObserver.dispatchOnGlobalLayout();
        }

        if (computesInternalInsets) {
            // Clear the original insets.
            final ViewTreeObserver.InternalInsetsInfo insets = mAttachInfo.mGivenInternalInsets;
            insets.reset();

            // Compute new insets in place.
            mAttachInfo.mTreeObserver.dispatchOnComputeInternalInsets(insets);
            mAttachInfo.mHasNonEmptyGivenInternalInsets = !insets.isEmpty();

            // Tell the window manager.
            if (insetsPending || !mLastGivenInsets.equals(insets)) {
                mLastGivenInsets.set(insets);

                // Translate insets to screen coordinates if needed.
                final Rect contentInsets;
                final Rect visibleInsets;
                final Region touchableRegion;
                if (mTranslator != null) {
                    contentInsets = mTranslator.getTranslatedContentInsets(insets.contentInsets);
                    visibleInsets = mTranslator.getTranslatedVisibleInsets(insets.visibleInsets);
                    touchableRegion = mTranslator.getTranslatedTouchableArea(insets.touchableRegion);
                } else {
                    contentInsets = insets.contentInsets;
                    visibleInsets = insets.visibleInsets;
                    touchableRegion = insets.touchableRegion;
                }

                try {
                    mWindowSession.setInsets(mWindow, insets.mTouchableInsets,
                            contentInsets, visibleInsets, touchableRegion);
                } catch (RemoteException e) {
                }
            }
        }

        if (mFirst && sAlwaysAssignFocus) {
            // handle first focus request
            if (DEBUG_INPUT_RESIZE) Log.v(mTag, "First: mView.hasFocus()="
                    + mView.hasFocus());
            if (mView != null) {
                if (!mView.hasFocus()) {
                    mView.restoreDefaultFocus();
                    if (DEBUG_INPUT_RESIZE) Log.v(mTag, "First: requested focused view="
                            + mView.findFocus());
                } else {
                    if (DEBUG_INPUT_RESIZE) Log.v(mTag, "First: existing focused view="
                            + mView.findFocus());
                }
            }
        }

        final boolean changedVisibility = (viewVisibilityChanged || mFirst) && isViewVisible;
        final boolean hasWindowFocus = mAttachInfo.mHasWindowFocus && isViewVisible;
        final boolean regainedFocus = hasWindowFocus && mLostWindowFocus;
        if (regainedFocus) {
            mLostWindowFocus = false;
        } else if (!hasWindowFocus && mHadWindowFocus) {
            mLostWindowFocus = true;
        }

        if (changedVisibility || regainedFocus) {
            // Toasts are presented as notifications - don't present them as windows as well
            boolean isToast = (mWindowAttributes == null) ? false
                    : (mWindowAttributes.type == WindowManager.LayoutParams.TYPE_TOAST);
            if (!isToast) {
                host.sendAccessibilityEvent(AccessibilityEvent.TYPE_WINDOW_STATE_CHANGED);
            }
        }

        mFirst = false;
        mWillDrawSoon = false;
        mNewSurfaceNeeded = false;
        mActivityRelaunched = false;
        mViewVisibility = viewVisibility;
        mHadWindowFocus = hasWindowFocus;

        if (hasWindowFocus && !isInLocalFocusMode()) {
            final boolean imTarget = WindowManager.LayoutParams
                    .mayUseInputMethod(mWindowAttributes.flags);
            if (imTarget != mLastWasImTarget) {
                mLastWasImTarget = imTarget;
                InputMethodManager imm = InputMethodManager.peekInstance();
                if (imm != null && imTarget) {
                    imm.onPreWindowFocus(mView, hasWindowFocus);
                    imm.onPostWindowFocus(mView, mView.findFocus(),
                            mWindowAttributes.softInputMode,
                            !mHasHadWindowFocus, mWindowAttributes.flags);
                }
            }
        }

        // Remember if we must report the next draw.
        if ((relayoutResult & WindowManagerGlobal.RELAYOUT_RES_FIRST_TIME) != 0) {
            reportNextDraw();
        }

        boolean cancelDraw = mAttachInfo.mTreeObserver.dispatchOnPreDraw() || !isViewVisible;

        if (!cancelDraw && !newSurface) {
            if (mPendingTransitions != null && mPendingTransitions.size() > 0) {
                for (int i = 0; i < mPendingTransitions.size(); ++i) {
                    mPendingTransitions.get(i).startChangingAnimations();
                }
                mPendingTransitions.clear();
            }
            // TODO: 2018/5/25 执行绘制的操作
            performDraw();
        } else {
            if (isViewVisible) {
                // Try again
                scheduleTraversals();
            } else if (mPendingTransitions != null && mPendingTransitions.size() > 0) {
                for (int i = 0; i < mPendingTransitions.size(); ++i) {
                    mPendingTransitions.get(i).endChangingAnimations();
                }
                mPendingTransitions.clear();
            }
        }

        mIsInTraversal = false;
    }
```
* 1、如果窗口的类型是有状态栏的，那么顶层视图DecorView所需要的窗口的宽度和高度
```
 // 如果窗口的类型是有状态栏的，那么顶层视图DecorView所需要的窗口的宽度和高度
            //就是除了状态栏
            if (shouldUseDisplaySize(lp)) {
                // NOTE -- system code, won't try to do compat mode.
                Point size = new Point();
                mDisplay.getRealSize(size);
                desiredWindowWidth = size.x;
                desiredWindowHeight = size.y;
            } else {
                //否者顶层视图DecorView所需要的窗口的宽度和高度就是整个屏幕的宽度
                desiredWindowWidth = dipToPx(config.screenWidthDp);
                desiredWindowHeight = dipToPx(config.screenHeightDp);
            }

```
* 2、/获得view宽高的测量规格
```
                   // TODO: 2018/5/25 //获得view宽高的测量规格，
                    // TODO: 2018/5/25 mWidth和mHeight表示窗口的宽高，lp.widthhe和lp.height表示DecorView根布局宽和高
                    int childWidthMeasureSpec = getRootMeasureSpec(mWidth, lp.width);
                    int childHeightMeasureSpec = getRootMeasureSpec(mHeight, lp.height);
```


* 3、注意这个对象`WindowManager.LayoutParams lp ` ,如果说`lp.horizontalWeight > 0.0f`或者是`lp.verticalWeight > 0.0f`,那么measureAgain =true；`horizontalWeight `这个标记大概是这个意思指示额外空间的多少将被水平分配。如果视图指定0不应被拉伸。否则额外像素将被优先评估。在所有重量大于0的视图中。一般都指示出还有多少的水平的空间将要被分配。
```
   /**
         * 这个WindowMananger 这里标记了 是
         */
        // TODO: 2018/6/4
        WindowManager.LayoutParams lp = mWindowAttributes;
              // TODO: 2018/5/25  这里是第一步的  执行测量的操作
           performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);
          if (lp.horizontalWeight > 0.0f) {
                        width += (int) ((mWidth - width) * lp.horizontalWeight);
                        childWidthMeasureSpec = MeasureSpec.makeMeasureSpec(width,
                                MeasureSpec.EXACTLY);
                        measureAgain = true;
                    }
                    if (lp.verticalWeight > 0.0f) {
                        height += (int) ((mHeight - height) * lp.verticalWeight);
                        childHeightMeasureSpec = MeasureSpec.makeMeasureSpec(height,
                                MeasureSpec.EXACTLY);
                        measureAgain = true;
                    }
```
* 如果这个measureAgain=true的话，就会再次调用`performMeasure()`,通过代码可以发现这就调用了两次`performMeasure`;
#### 其实我这里犯了一个错误，不是这样的子，这个标记不一定是为`true`。
```
 if (measureAgain) {
                        if (DEBUG_LAYOUT) Log.v(mTag,
                                "And hey let's measure once more: width=" + width
                                        + " height=" + height);
                        performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);
   }
```
* 4、关于` performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);`方法,其实就是调用的是`mView.measure(childWidthMeasureSpec, childHeightMeasureSpec);`,也就是View第一步是测量。
```
   private void performMeasure(int childWidthMeasureSpec, int childHeightMeasureSpec) {
        if (mView == null) {
            return;
        }
        Trace.traceBegin(Trace.TRACE_TAG_VIEW, "measure");
        try {
            mView.measure(childWidthMeasureSpec, childHeightMeasureSpec);
        } finally {
            Trace.traceEnd(Trace.TRACE_TAG_VIEW);
        }
    }
```
5、第一次绘制的时候，这个标记一定是`didLayout`一定是true，一定会走到这个方法里面去`performLayout(lp, mWidth, mHeight);`
```
     final boolean didLayout = layoutRequested && (!mStopped || mReportNextDraw);
        boolean triggerGlobalLayoutListener = didLayout
                || mAttachInfo.mRecomputeGlobalAttributes;
        if (didLayout) {
            // TODO: 2018/5/25  执行布局操作
            performLayout(lp, mWidth, mHeight);
        }
}
```
* 关于`performLayout`这个方法，直接会调用` host.layout(0, 0, host.getMeasuredWidth(), host.getMeasuredHeight());`,也就是View的layout的方法。
```
  private void performLayout(WindowManager.LayoutParams lp, int desiredWindowWidth,
                               int desiredWindowHeight) {
final View host = mView;
  try {
            host.layout(0, 0, host.getMeasuredWidth(), host.getMeasuredHeight());
           //测试层级
                    measureHierarchy(host, lp, mView.getContext().getResources(),
                            desiredWindowWidth, desiredWindowHeight);
 } finally {
            Trace.traceEnd(Trace.TRACE_TAG_VIEW);
        }

```
6、这两个标记也是`!cancelDraw && !newSurface`为`true`,那么就会走到`  performDraw();`
```
   if (!cancelDraw && !newSurface) {
            if (mPendingTransitions != null && mPendingTransitions.size() > 0) {
                for (int i = 0; i < mPendingTransitions.size(); ++i) {
                    mPendingTransitions.get(i).startChangingAnimations();
                }
                mPendingTransitions.clear();
            }
            // TODO: 2018/5/25 执行绘制的操作
            performDraw();
        } 
```
* 关于`performDraw();`方法,直接调用的是`draw(fullRedrawNeeded);`
```
    private void performDraw() {
        mIsDrawing = true;
        Trace.traceBegin(Trace.TRACE_TAG_VIEW, "draw");
        try {
            draw(fullRedrawNeeded);
        } finally {
            mIsDrawing = false;
            Trace.traceEnd(Trace.TRACE_TAG_VIEW);
        }
}
```
* 关于`draw(fullRedrawNeeded);`,会调用到这里来`drawSoftware(surface, mAttachInfo, xOffset, yOffset, scalingRequired, dirty)`
```
  private void draw(boolean fullRedrawNeeded) {
        if (!drawSoftware(surface, mAttachInfo, xOffset, yOffset, scalingRequired, dirty)) {
                    return;
                }
    }
```
* 关于这个方法`drawSoftware(surface, mAttachInfo, xOffset, yOffset, scalingRequired, dirty)`。官方的解释为如果绘图成功，如果发生错误，则为false。返回false的，程序就发生了异常，也就是程序GG掉了，绘制失败，这里仅仅贴出关键的代码~~~，这样字，就调用到了` mView.draw(canvas);`
```
  private boolean drawSoftware(Surface surface, AttachInfo attachInfo, int xoff, int yoff,
                                 boolean scalingRequired, Rect dirty) {
    try {
                canvas.translate(-xoff, -yoff);
                if (mTranslator != null) {
                    mTranslator.translateCanvas(canvas);
                }
                canvas.setScreenDensity(scalingRequired ? mNoncompatDensity : 0);
                attachInfo.mSetIgnoreDirtyState = false;
                // TODO: 2018/5/25   调用了View里面的draw方法
                mView.draw(canvas);

                drawAccessibilityFocusedDrawableIfNeeded(canvas);
            } finally {
                if (!attachInfo.mSetIgnoreDirtyState) {
                    // Only clear the flag if it was not set during the mView.draw() call
                    attachInfo.mIgnoreDirtyState = false;
                }
            }
}

```
* 最后做了两张图
 ![View的绘制流程（一）.jpg](https://upload-images.jianshu.io/upload_images/5363507-9f68fbf91f7c7eac.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![View的绘制流程（二）ViewRootImpl.performTraversals.jpg](https://upload-images.jianshu.io/upload_images/5363507-07125ab1e5530813.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 说明几点
   * 如果感兴趣的，一定要去打个断点看一下这个流程
   * 限于作者水平有限，一定会存在有些错误，还望指出，谢谢
    