---
title: Android源码分析ViewStub源码解析
date: 2018-06-07 16:11:17
tags: [Android源码,ViewStub,绘制]
---
## 源码基于安卓8.0分析结果
* `ViewStub`是一种不可见的并且大小为0的试图，它可以延迟到运行时才填充`inflate` 布局资源，当`Viewstub`设为可见或者是`inflate`的时候，就会填充布局资源，这个布局和普通的试图就基本上没有任何区别，比如说，加载网络失败，或者是一个比较消耗性能的功能，需要用户去点击才可以加载!从而这样更加的节约了性能。对安卓布局很友好！
<!--  more  -->
* `ViewStub`用法
```
    <ViewStub
        android:padding="10dp"
        android:background="@color/colorPrimary"
        android:layout_gravity="center"
        android:inflatedId="@+id/view_stub_inflateid"
        android:id="@+id/view_stub"
        android:layout="@layout/view_stub_imageview"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />
```
* 这篇文章[安卓代码、图片、布局、网络和电量优化](https://www.jianshu.com/p/82b76e0cb41e)说如果这个根布局是个View，比如说是个ImagView，那么找出来的id为null，得必须注意这一点  -----2018.6.7修正这个说法，以前我说的是错误的，根本上的原因是ViewStub设置了 inflateid ，这才是更本身的原因
* 在这里记住一点，如果在 `ViewStub`标签中设置了`android:inflatedId="@+id/view_stub_inflateid"`,在`layout`布局中的根布局在设置` android:id="@+id/view_stub_layout"`,这个`id`永远找出来都是为null的，原因会在下面说明
```
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:padding="10dp"
    android:id="@+id/view_stub_layout"
    android:src="@drawable/ic_launcher_background"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <TextView
        android:text="如果这个根布局是个View，比如说是个ImagView，那么找出来的id为null，得必须注意这一点  -----2018.6.7修正这个说法，以前我说的是错误的，根本上的原因是ViewStub设置了 inflateid ，这才是更本身的原因"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />
    <ImageView
        android:layout_marginTop="20dp"
        android:id="@+id/imageview"
        android:padding="10dp"
        android:src="@drawable/ic_launcher_background"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>
</FrameLayout>

```
* 在`activity`或者是`fragment`中的使用,`mViewStub.getParent()==null`就是说明没有被填充，需要填充，如果填充了，那么它的`parent`不会为null，具体的骚操作，后续我介绍`View`的绘制流程的时候在详细说明。
* 第一种使用的方法
```
  mViewStub = findViewById(R.id.view_stub);
  if (null!=mViewStub.getParent()){
  View inflate = mViewStub.inflate();
  ....
 }
```
* 第二种方式:` mViewStub.setVisibility(View.VISIBLE);`和`inflate()`方法一样。
```
  mViewStub = findViewById(R.id.view_stub);
  if (null!=mViewStub.getParent()){
    mViewStub.setVisibility(View.VISIBLE);
  ....
 }
```
* 第三种方式,`my_title_parent_id`是`layout`的根布局的`id`
```
      mViewStub = findViewById(R.id.view_stub);
    // 成员变量commLv2为空则代表未加载 commLv2 的id为ViewStub中的根布局的id
    View commLv2=findViewById(R.id.my_title_parent_id);
   if ( commLv2 == null ) {
      // 加载评论列表布局, 并且获取评论ListView,inflate函数直接返回ListView对象
        commLv2 = (View)mViewStub.inflate();
      } else {
         // ViewStub已经加载
     }
```
* `ViewStub`构造方法,注意获取了几个值`mInflatedId `就是` android:inflatedId="@+id/find_view_stub"`这个值， `mLayoutResource `就是`layout`的resId，ViewStub的  `mID `id。可以看出`ViewStub`是`View`的子类.
```
public final class ViewStub extends View {
 public ViewStub(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes) {
        super(context);
        final TypedArray a = context.obtainStyledAttributes(attrs,
                R.styleable.ViewStub, defStyleAttr, defStyleRes);
        // TODO: 2018/5/23  ViewStub 中设置的标签id 如果设置了 这里就一定有值 mInflatedId！=NO_Id
        mInflatedId = a.getResourceId(R.styleable.ViewStub_inflatedId, NO_ID);
        mLayoutResource = a.getResourceId(R.styleable.ViewStub_layout, 0);
        mID = a.getResourceId(R.styleable.ViewStub_id, NO_ID);
        a.recycle();
        //不可见
        setVisibility(GONE);
        // 设置不绘制
        setWillNotDraw(true);
    }
}
```
* 在构造方法中:同时注意不可见 `setVisibility(GONE);` ,设置不绘制` setWillNotDraw(true);`,同时通过下面的方法看出，ViewStub 是一个大小为0的视图。
```
   @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        // 宽高都为0   onMeasure的时候 宽高都为0
        setMeasuredDimension(0, 0);
    }
    //todo 为啥这个控件 是个大小为0的控件 ，那是因为他妈的这里更不就没有画
    @Override
    public void draw(Canvas canvas) {
    }

    @Override
    protected void dispatchDraw(Canvas canvas) {
    }
```
* 关于`inflate()`方法
```
  public View inflate() {
        // 1、获取ViewStub的parent view，也是目标布局根元素的parent view
        final ViewParent viewParent = getParent();

        if (viewParent != null && viewParent instanceof ViewGroup) {
            if (mLayoutResource != 0) {
                final ViewGroup parent = (ViewGroup) viewParent;
                /// 2、加载目标布局  牛逼的方法
                final View view = inflateViewNoAdd(parent);
                // 3、将ViewStub自身从parent中移除
                replaceSelfWithView(view, parent);

                mInflatedViewRef = new WeakReference<>(view);
                if (mInflateListener != null) {
                    mInflateListener.onInflate(this, view);
                }

                return view;
            } else {
                // TODO: 2018/5/23 必须设置布局的文件
                throw new IllegalArgumentException("ViewStub must have a valid layoutResource");
            }
        } else {
            // TODO: 2018/5/23 iewParent instanceof ViewGroup 不属于的话，就好比在一个TextView创建一个ViewStub直接爆炸
            throw new IllegalStateException("ViewStub must have a non-null ViewGroup viewParent");
        }
    }
```
* 第一点，ViewStup也只能在ViewGroup中使用，不能在View中去使用，要不然会抛出异常`IllegalStateException("ViewStub must have a non-null ViewGroup viewParent");`
* 第二点，也必须设置`layout`属性，要不然也会抛出异常` throw new IllegalArgumentException("ViewStub must have a valid layoutResource");`;


* 关于方法`inflateViewNoAdd(parent);`
```
   private View inflateViewNoAdd(ViewGroup parent) {
        final LayoutInflater factory;
        if (mInflater != null) {
            factory = mInflater;
        } else {
            factory = LayoutInflater.from(mContext);
        }
        final View view = factory.inflate(mLayoutResource, parent, false);
        //和 LayoutInflater一个道理，设置了，ViewStub 引用进来的根布局的id找出来为null  非常有些意思
        if (mInflatedId != NO_ID) {
            view.setId(mInflatedId);
        }
        return view;
    }
```
* 第一点，底层调用的还是`LayoutInflater.from(mContext).inflate(mLayoutResource, parent, false);`,[Android源码分析（LayoutInflater.from(this).inflate(resId,null);源码解析）](https://www.jianshu.com/p/38c6a9842efc),这里不做过多的说明
* 第二点，又看到这个方法，似曾相识，对，这也是为什么`ViewStub`找不到根布局id的原因，因为`mInflatedId != NO_ID`,就会` view.setId(mInflatedId);`
```
   if (mInflatedId != NO_ID) {
         view.setId(mInflatedId);
 }
```
* 将ViewStub自身从parent中移除`replaceSelfWithView(view, parent);`,具体的原因，这里不做分析，因为有点小复杂，这里就大概明白就行，对于理解这个ViewStub不困难，哈哈
```
    private void replaceSelfWithView(View view, ViewGroup parent) {
        final int index = parent.indexOfChild(this);
        // 3、将ViewStub自身从parent中移除
        parent.removeViewInLayout(this);

        final ViewGroup.LayoutParams layoutParams = getLayoutParams();
        if (layoutParams != null) {
            // 4、将目标布局的根元素添加到parent中，有参数
            parent.addView(view, index, layoutParams);
        } else {
           // 5、将目标布局的根元素添加到parent中
            parent.addView(view, index);
        }
    }

```
* 这里使用到了弱引用，只有弱引用指向的对象的生命周期更短，当垃圾回收器扫描到只有具有弱引用的对象的时候，不论当前空间是否不足，都会对弱引用对象进行回收，当然弱引用也可以和一个队列配合着使用，为了更好地释放内存，[安卓代码、图片、布局、网络和电量优化](https://www.jianshu.com/p/82b76e0cb41e)这篇文章有很好的解释，而且这个`mInflatedViewRef `只在这里初始化，如果说没有调用`inflate`的方法的话，这个对象一定为`null`;
```
 //更好的释放内存
  private WeakReference<View> mInflatedViewRef;
  mInflatedViewRef = new WeakReference<>(view);
                if (mInflateListener != null) {
               mInflateListener.onInflate(this, view)
  }
```

* 为啥`setVisibility（View.VISIBLE）`等同于`inflate`,原因是`ViewStub`进行了重写。可以看出代码的逻辑，只要没有调用过，`inflate()`方法，`setVisibility(VISIBLE ) ` 和`setVisibility(INVISIBLE)`这个两个参数走的方法一样，只不过，一个看不到，实际上的位置已经确定了（INVISIBLE）。但是如果调用多次的话`setVisibility()`记得也得判断下`null!=mViewStub.getParent()`
```
@Override
    @android.view.RemotableViewMethod(asyncImpl = "setVisibilityAsync")
    public void setVisibility(int visibility) {
        // TODO: 2018/5/23  弱引用的使用
        //如果已经加载过则只设置Visibility属性
        if (mInflatedViewRef != null) {
            View view = mInflatedViewRef.get();
            if (view != null) {
                view.setVisibility(visibility);
            } else {
                throw new IllegalStateException("setVisibility called on un-referenced view");
            }
        } else {
            // 如果未加载,这加载目标布局
            super.setVisibility(visibility);
            if (visibility == VISIBLE || visibility == INVISIBLE) {
                inflate();// 调用inflate来加载目标布局
            }
        }
    }
```
* 贴出全部的代码，有空的话，可以研究下。
```
@RemoteView
public final class ViewStub extends View {
    private int mInflatedId;
    private int mLayoutResource;
    // TODO: 2018/5/23 弱引用：弱引用是比软引用更弱的一种的引用的类型，
    // 只有弱引用指向的对象的生命周期更短，当垃圾回收器扫描到只有具有弱引用的对象的时候，
    // 不敢当前空间是否不足，都会对弱引用对象进行回收，当然弱引用也可以和一个队列配合着使用

    //更好的释放内存
    private WeakReference<View> mInflatedViewRef;

    private LayoutInflater mInflater;
    private OnInflateListener mInflateListener;

    public ViewStub(Context context) {
        this(context, 0);
    }

    /**
     * Creates a new ViewStub with the specified layout resource.
     *
     * @param context The application's environment.
     * @param layoutResource The reference to a layout resource that will be inflated.
     */
    public ViewStub(Context context, @LayoutRes int layoutResource) {
        this(context, null);

        mLayoutResource = layoutResource;
    }

    public ViewStub(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public ViewStub(Context context, AttributeSet attrs, int defStyleAttr) {
        this(context, attrs, defStyleAttr, 0);
    }

    public ViewStub(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes) {
        super(context);

        final TypedArray a = context.obtainStyledAttributes(attrs,
                R.styleable.ViewStub, defStyleAttr, defStyleRes);
        // TODO: 2018/5/23  ViewStub 中设置的标签id 如果设置了 这里就一定有值 mInflatedId！=NO_Id
        mInflatedId = a.getResourceId(R.styleable.ViewStub_inflatedId, NO_ID);
        mLayoutResource = a.getResourceId(R.styleable.ViewStub_layout, 0);
        mID = a.getResourceId(R.styleable.ViewStub_id, NO_ID);
        a.recycle();
        //不可见
        setVisibility(GONE);
        // 设置不绘制
        setWillNotDraw(true);
    }

    /**
     * Returns the id taken by the inflated view. If the inflated id is
     * {@link View#NO_ID}, the inflated view keeps its original id.
     *
     * @return A positive integer used to identify the inflated view or
     *         {@link #NO_ID} if the inflated view should keep its id.
     *
     * @see #setInflatedId(int)
     * @attr ref android.R.styleable#ViewStub_inflatedId
     */
    @IdRes
    public int getInflatedId() {
        return mInflatedId;
    }

    /**
     * Defines the id taken by the inflated view. If the inflated id is
     * {@link View#NO_ID}, the inflated view keeps its original id.
     *
     * @param inflatedId A positive integer used to identify the inflated view or
     *                   {@link #NO_ID} if the inflated view should keep its id.
     *
     * @see #getInflatedId()
     * @attr ref android.R.styleable#ViewStub_inflatedId
     */
    @android.view.RemotableViewMethod(asyncImpl = "setInflatedIdAsync")
    public void setInflatedId(@IdRes int inflatedId) {
        mInflatedId = inflatedId;
    }

    /** @hide **/
    public Runnable setInflatedIdAsync(@IdRes int inflatedId) {
        mInflatedId = inflatedId;
        return null;
    }

    /**
     * Returns the layout resource that will be used by {@link #setVisibility(int)} or
     * {@link #inflate()} to replace this StubbedView
     * in its parent by another view.
     *
     * @return The layout resource identifier used to inflate the new View.
     *
     * @see #setLayoutResource(int)
     * @see #setVisibility(int)
     * @see #inflate()
     * @attr ref android.R.styleable#ViewStub_layout
     */
    @LayoutRes
    public int getLayoutResource() {
        return mLayoutResource;
    }

    /**
     * Specifies the layout resource to inflate when this StubbedView becomes visible or invisible
     * or when {@link #inflate()} is invoked. The View created by inflating the layout resource is
     * used to replace this StubbedView in its parent.
     *
     * @param layoutResource A valid layout resource identifier (different from 0.)
     *
     * @see #getLayoutResource()
     * @see #setVisibility(int)
     * @see #inflate()
     * @attr ref android.R.styleable#ViewStub_layout
     */
    @android.view.RemotableViewMethod(asyncImpl = "setLayoutResourceAsync")
    public void setLayoutResource(@LayoutRes int layoutResource) {
        mLayoutResource = layoutResource;
    }

    /** @hide **/
    public Runnable setLayoutResourceAsync(@LayoutRes int layoutResource) {
        mLayoutResource = layoutResource;
        return null;
    }

    /**
     * Set {@link LayoutInflater} to use in {@link #inflate()}, or {@code null}
     * to use the default.
     */
    public void setLayoutInflater(LayoutInflater inflater) {
        mInflater = inflater;
    }

    /**
     * Get current {@link LayoutInflater} used in {@link #inflate()}.
     */
    public LayoutInflater getLayoutInflater() {
        return mInflater;
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        // 宽高都为0   onMeasure的时候 宽高都为0
        setMeasuredDimension(0, 0);
    }
    //todo 为啥这个控件 是个大小为0的控件 ，那是因为他妈的这里更不就没有画
    @Override
    public void draw(Canvas canvas) {
    }

    @Override
    protected void dispatchDraw(Canvas canvas) {
    }

    /**
     * When visibility is set to {@link #VISIBLE} or {@link #INVISIBLE},
     * {@link #inflate()} is invoked and this StubbedView is replaced in its parent
     * by the inflated layout resource. After that calls to this function are passed
     * through to the inflated view.
     *
     * @param visibility One of {@link #VISIBLE}, {@link #INVISIBLE}, or {@link #GONE}.
     *
     * @see #inflate()
     */
    @Override
    @android.view.RemotableViewMethod(asyncImpl = "setVisibilityAsync")
    public void setVisibility(int visibility) {
        // TODO: 2018/5/23  弱引用的使用
        //如果已经加载过则只设置Visibility属性
        if (mInflatedViewRef != null) {
            View view = mInflatedViewRef.get();
            if (view != null) {
                view.setVisibility(visibility);
            } else {
                throw new IllegalStateException("setVisibility called on un-referenced view");
            }
        } else {
            // 如果未加载,这加载目标布局
            super.setVisibility(visibility);
            if (visibility == VISIBLE || visibility == INVISIBLE) {
                inflate();// 调用inflate来加载目标布局
            }
        }
    }

    /** @hide **/
    public Runnable setVisibilityAsync(int visibility) {
        if (visibility == VISIBLE || visibility == INVISIBLE) {
            ViewGroup parent = (ViewGroup) getParent();
            return new ViewReplaceRunnable(inflateViewNoAdd(parent));
        } else {
            return null;
        }
    }

    private View inflateViewNoAdd(ViewGroup parent) {
        final LayoutInflater factory;
        if (mInflater != null) {
            factory = mInflater;
        } else {
            factory = LayoutInflater.from(mContext);
        }
        final View view = factory.inflate(mLayoutResource, parent, false);
        //和 LayoutInflater一个道理，设置了，ViewStub 引用进来的根布局的id找出来为null  非常有些意思
        if (mInflatedId != NO_ID) {
            view.setId(mInflatedId);
        }
        return view;
    }

    // TODO: 2018/5/23 关注他
    private void replaceSelfWithView(View view, ViewGroup parent) {
        final int index = parent.indexOfChild(this);
        // 3、将ViewStub自身从parent中移除
        parent.removeViewInLayout(this);

        final ViewGroup.LayoutParams layoutParams = getLayoutParams();
        if (layoutParams != null) {
            // 4、将目标布局的根元素添加到parent中，有参数
            parent.addView(view, index, layoutParams);
        } else {
           // 5、将目标布局的根元素添加到parent中
            parent.addView(view, index);
        }
    }

    /**
     * Inflates the layout resource identified by {@link #getLayoutResource()}
     * and replaces this StubbedView in its parent by the inflated layout resource.
     *
     * @return The inflated layout resource.
     *
     */
    public View inflate() {
        // 1、获取ViewStub的parent view，也是目标布局根元素的parent view
        final ViewParent viewParent = getParent();

        if (viewParent != null && viewParent instanceof ViewGroup) {
            if (mLayoutResource != 0) {
                final ViewGroup parent = (ViewGroup) viewParent;
                /// 2、加载目标布局  牛逼的方法
                final View view = inflateViewNoAdd(parent);
                // 3、将ViewStub自身从parent中移除
                replaceSelfWithView(view, parent);

                mInflatedViewRef = new WeakReference<>(view);
                if (mInflateListener != null) {
                    mInflateListener.onInflate(this, view);
                }

                return view;
            } else {
                // TODO: 2018/5/23 必须设置布局的文件
                throw new IllegalArgumentException("ViewStub must have a valid layoutResource");
            }
        } else {
            // TODO: 2018/5/23 iewParent instanceof ViewGroup 不属于的话，就好比在一个TextView创建一个ViewStub直接爆炸
            throw new IllegalStateException("ViewStub must have a non-null ViewGroup viewParent");
        }
    }

    /**
     * Specifies the inflate listener to be notified after this ViewStub successfully
     * inflated its layout resource.
     *
     * @param inflateListener The OnInflateListener to notify of successful inflation.
     *
     * @see ViewStub.OnInflateListener
     */
    public void setOnInflateListener(OnInflateListener inflateListener) {
        mInflateListener = inflateListener;
    }

    /**
     * Listener used to receive a notification after a ViewStub has successfully
     * inflated its layout resource.
     *
     * @see ViewStub#setOnInflateListener(ViewStub.OnInflateListener)
     */
    public static interface OnInflateListener {
        /**
         * Invoked after a ViewStub successfully inflated its layout resource.
         * This method is invoked after the inflated view was added to the
         * hierarchy but before the layout pass.
         *
         * @param stub The ViewStub that initiated the inflation.
         * @param inflated The inflated View.
         */
        void onInflate(ViewStub stub, View inflated);
    }

    /** @hide **/
    public class ViewReplaceRunnable implements Runnable {
        public final View view;

        ViewReplaceRunnable(View view) {
            this.view = view;
        }

        @Override
        public void run() {
            replaceSelfWithView(view, (ViewGroup) getParent());
        }
    }
}

```
* 最后做了一张图
![ViewStub源码解析.jpg](https://upload-images.jianshu.io/upload_images/5363507-f369f9eecac1b239.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 说明一下
  * `ViewStub`的原理很简单！好吧，这个有点皮


