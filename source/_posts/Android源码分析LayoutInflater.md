---
title: Android源码分析LayoutInflater
date: 2018-06-07 16:12:57
tags: [Android源码,LayoutInflater]
---
### 源码基于安卓8.0分析结果
* 在这篇文章[Android源码分析(Activity.setContentView源码解析)](https://www.jianshu.com/p/d9d919608842)，分析得出，底层走的就是`LayoutInflater.from(this),inflate(),`如果`inflate()`传入的view的话，就调用两次`LayoutInflater.from(this),inflate(),`，如果是`inflate()`传入的resId（布局的id）的话，就调用三次`LayoutInflater.from(this),inflate(),`,具体请看上篇文章。
<!--  more  -->
* 怎么使用,一共有三种的方式，主要看下这种方式`LayoutInflater inflater1 = getLayoutInflater();`,其实调用的就是Activity的getLayoutInflater()方法。
```
  LayoutInflater inflater1 = getLayoutInflater();//调用Activity的getLayoutInflater()
 LayoutInflater inflater2 = LayoutInflater.from(this);
 LayoutInflater inflater3 = (LayoutInflater)this.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
```
* Activity的getLayoutInflater()方法。
```
  /**
     * Convenience for calling
     * {@link android.view.Window#getLayoutInflater}.
     */
    @NonNull
    public LayoutInflater getLayoutInflater() {
        return getWindow().getLayoutInflater();
    }
```
* 通过[Android源码分析（事件传递）](https://www.jianshu.com/p/f7e3a14daf51)这边文章，我们知道，`Window类`只有唯一的子类`PhoneWindow`。所以主要的是去看`PhoneWindow`类
```
 public PhoneWindow(Context context) {
     super(context);
      mLayoutInflater = LayoutInflater.from(context);
   }
```
` LayoutInflater inflater2 = LayoutInflater.from(this);`的实现的方式
```
 /**
     * Obtains the LayoutInflater from the given context.
     */
    public static LayoutInflater from(Context context) {
        LayoutInflater LayoutInflater =
                (LayoutInflater) context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
        if (LayoutInflater == null) {
            throw new AssertionError("LayoutInflater not found.");
        }
        return LayoutInflater;
    }
```
* 以上三种方法，最终还是调用了`context.getSystemService(Context.LAYOUT_INFLATER_SERVICE)`获取LayoutInflater实例对象。
* inflate的方法
```
   public View inflate(XmlPullParser parser, @Nullable ViewGroup root) {
        return inflate(parser, root, root != null);
    }
```
* 到这里来` inflate(@LayoutRes int resource, @Nullable ViewGroup root, boolean attachToRoot) `,个人的对这个方法的理解是：把布局文件填充成View，如果传入的`root`不为`null`，并且`attachToRoot`为`true`,就把view填充到root上，相反就不添加上来！
```
   public View inflate(@LayoutRes int resource, @Nullable ViewGroup root, boolean attachToRoot) {
        final Resources res = getContext().getResources();
        if (DEBUG) {
            Log.d(TAG, "INFLATING from resource: \"" + res.getResourceName(resource) + "\" ("
                    + Integer.toHexString(resource) + ")");
        }
        //这里通过底层的方法，得到一个XmlResourceParser对象
        final XmlResourceParser parser = res.getLayout(resource);
        try {
            return inflate(parser, root, attachToRoot);
        } finally {
            parser.close();
        }
    }
```
* `inflate(parser, root, attachToRoot)`，特别关心这个方法
```
 public View inflate(XmlPullParser parser, @Nullable ViewGroup root, boolean attachToRoot) {
        synchronized (mConstructorArgs) {
            //底层的方法，不知道原理
            Trace.traceBegin(Trace.TRACE_TAG_VIEW, "inflate");
            // TODO: 2018/6/5  我没有搞明白 为啥整个值在这里做什么的， 在这个方法里面都没有使用
            // from传入的Context
            final Context inflaterContext = mContext;
            // 判断parser是否是AttributeSet，如果不是则用XmlPullAttributes去包装一下。
            final AttributeSet attrs = Xml.asAttributeSet(parser);
            // 保存之前的Context
            Context lastContext = (Context) mConstructorArgs[0];
            // 赋值为传入的Context
            mConstructorArgs[0] = inflaterContext;
            // 默认返回的是传入的Parent
            View result = root;
            try {
                // Look for the root node.
                int type;// 迭代xml中的所有元素，挨个解析
                while ((type = parser.next()) != XmlPullParser.START_TAG &&
                        type != XmlPullParser.END_DOCUMENT) {
                    // Empty
                }
               // //如果没找到有效的开始标签则抛出InflateException
                if (type != XmlPullParser.START_TAG) {
                    throw new InflateException(parser.getPositionDescription()
                            + ": No start tag found!");
                }

                final String name = parser.getName();

                if (DEBUG) {
                    System.out.println("**************************");
                    System.out.println("Creating root view: "
                            + name);
                    System.out.println("**************************");
                }
                //// 如果根节点是“merge”标签
                if (TAG_MERGE.equals(name)) {
                    // 根节点为空或者不添加到根节点上，则抛出异常。
                    // 因为“merge”标签必须是要被添加到父节点上的，不能独立存在。
                    if (root == null || !attachToRoot) {
                        throw new InflateException("<merge /> can be used only with a valid "
                                + "ViewGroup root and attachToRoot=true");
                    }
                    // 递归实例化root（也就是传入Parent）下所有的View
                    // 如果xml中的节点是merge节点，则调用rInflate()--方法  // 递归实例化根节点的子View
                    rInflate(parser, root, inflaterContext, attrs, false);
                } else {
                    // Temp is the root view that was found in the xml
                    //通过View的父View，View的名称、attrs属性实例化View（内部调用onCreateView()和createView()）。
                    // TODO: 2018/6/6  // 实例化根节点的View 
                    final View temp = createViewFromTag(root, name, inflaterContext, attrs);

                    ViewGroup.LayoutParams params = null;

                    if (root != null) {
                        if (DEBUG) {
                            System.out.println("Creating params from root: " +
                                    root);
                        }
                        // Create layout params that match root, if supplied
                        params = root.generateLayoutParams(attrs);
                        if (!attachToRoot) {
                            // Set the layout params for temp if we are not
                            // attaching. (If we are, we use addView, below)
                            temp.setLayoutParams(params);
                        }
                    }

                    if (DEBUG) {
                        System.out.println("-----> start inflating children");
                    }

                    // Inflate all children under temp against its context.
                    // 递归实例化跟节点的子View
                    rInflateChildren(parser, temp, attrs, true);

                    if (DEBUG) {
                        System.out.println("-----> done inflating children");
                    }

                    // We are supposed to attach all the views we found (int temp)
                    // to root. Do that now.\

                    if (root != null && attachToRoot) {
                        root.addView(temp, params); // TODO: 2018/6/6      返回父View
                    }
               
                    // Decide whether to return the root that was passed in or the
                    // top view found in xml.
                    // TODO: 2018/6/6  父View是空或者不把填充的View添加到父View)
                    if (root == null || !attachToRoot) {
                        result = temp;  // TODO: 2018/6/6 返回根节点View 
                    }
                }

            } catch (XmlPullParserException e) {
                final InflateException ie = new InflateException(e.getMessage(), e);
                ie.setStackTrace(EMPTY_STACK_TRACE);
                throw ie;
            } catch (Exception e) {
                final InflateException ie = new InflateException(parser.getPositionDescription()
                        + ": " + e.getMessage(), e);
                ie.setStackTrace(EMPTY_STACK_TRACE);
                throw ie;
            } finally {
                // Don't retain static reference on context.
                //不要在上下文中保留静态引用。
                // 把这之前保存的Context从新放回全局变量中。
                mConstructorArgs[0] = lastContext;
                mConstructorArgs[1] = null;

                Trace.traceEnd(Trace.TRACE_TAG_VIEW);
            }

            return result;
        }
    }
```
* 注意其中有个判断`TAG_MERGE.equals(name))` ,这里就是merge标签的开始，以下的问题，都带着这个疑问往下面看。
```
   if (TAG_MERGE.equals(name)) {
                    // 根节点为空或者不添加到根节点上，则抛出异常。
                    // 因为“merge”标签必须是要被添加到父节点上的，不能独立存在。
                    if (root == null || !attachToRoot) {
                        throw new InflateException("<merge /> can be used only with a valid "
                                + "ViewGroup root and attachToRoot=true");
                    }
                    // 递归实例化root（也就是传入Parent）下所有的View
                    // 如果xml中的节点是merge节点，则调用rInflate()--方法  // 递归实例化根节点的子View
                    rInflate(parser, root, inflaterContext, attrs, false);
  }
```
* 解析到`merge`标签的话，就会进入到这个方法`rInflate(parser, root, inflaterContext, attrs, false);`:递归实例化根节点的子View,在这里说明一点的是这个方法传入的`root`会不会为`null`.这篇文章说过[Android源码分析（Activity.setContentView源码解析）](https://www.jianshu.com/p/d9d919608842),肯定不会为null，一般是我们自己去使用的时候，有可能为null，具体的差别可以参考这篇文章[Android LayoutInflater深度解析 给你带来全新的认识](https://blog.csdn.net/lmj623565791/article/details/38171465)

* 关于`rInflate(parser, root, inflaterContext, attrs, false);`方法：递归方法用于降序XML层次结构并实例化视图，实例化它们的子节点，然后调用`onFinishInflate(),`, 完成填充
```
 void rInflate(XmlPullParser parser, View parent, Context context,
            AttributeSet attrs, boolean finishInflate) throws XmlPullParserException, IOException {

        final int depth = parser.getDepth();
        int type;
        boolean pendingRequestFocus = false;
        // 迭代xml中的所有元素，挨个解析
        while (((type = parser.next()) != XmlPullParser.END_TAG ||
                parser.getDepth() > depth) && type != XmlPullParser.END_DOCUMENT) {

            if (type != XmlPullParser.START_TAG) {
                continue;
            }

            final String name = parser.getName();
            //1、解析请求焦点 ，这个控件一直有焦点 requestFocus
            if (TAG_REQUEST_FOCUS.equals(name)) {
                pendingRequestFocus = true;
                consumeChildElements(parser);
            } else if (TAG_TAG.equals(name)) {
                //在包含的视图上设置键标记。
                parseViewTag(parser, parent, attrs);
            } else if (TAG_INCLUDE.equals(name)) {// 如果xml中的节点是include节点，则调用parseInclude方法
                if (parser.getDepth() == 0) {
                    throw new InflateException("<include /> cannot be the root element");
                }
                //解析include标签
                parseInclude(parser, context, parent, attrs);
            } else if (TAG_MERGE.equals(name)) {
                // TODO: 2018/5/23 merge  作为布局里面的元素 会报错的哦 ，注意哦
                //想象下 两个merge 标签重合在一起
                throw new InflateException("<merge /> must be the root element");
            } else {
                // TODO: 2018/5/23 其实就是如果是merge标签，
                // TODO: 2018/5/23  那么直接将其中的子元素添加到merge标签parent中，这样就保证了不会引入额外的层级。
                // 1、我们的例子会进入这里  // 通过View的名称实例化View
                final View view = createViewFromTag(parent, name, context, attrs);
                // 2、获取merge标签的parent
                final ViewGroup viewGroup = (ViewGroup) parent;
                // 3、获取布局参数
                final ViewGroup.LayoutParams params = viewGroup.generateLayoutParams(attrs);
                // 4、递归解析每个子元素
                rInflateChildren(parser, view, attrs, true);
                // 5、将子元素直接添加到merge标签的parent view中
                viewGroup.addView(view, params);
            }
        }

        if (pendingRequestFocus) {
            parent.restoreDefaultFocus();
        }
        /**
         * onFinishInflate() View 中的一个空实现，标记完全填充完成了
         * The method onFinishInflate() will be called after all children have been added.
         * 以后是所有的孩子调用完成了，之后就调用这个方法
         */
        if (finishInflate) {
            parent.onFinishInflate();
        }
    }

```
* 通过对上面的方法分析，还打了断点，打断点注意一点，用模拟器，基本上手机厂商都改了`frameWork`层的代码，打出来是乱的。代码会走到这里来,看代码的分析，就把新的View，通过`ViewGroup.addView()`进去了，这也是merge便签的原理：`merge标签可以自动消除当一个布局插入到另一个布局时产生的多余的View Group，也可用于替换FrameLayout`
```
               // TODO: 2018/5/23 其实就是如果是merge标签，
                // TODO: 2018/5/23  那么直接将其中的子元素添加到merge标签parent中，这样就保证了不会引入额外的层级。
                // 1、我们的例子会进入这里  // 通过View的名称实例化View
                final View view = createViewFromTag(parent, name, context, attrs);
                // 2、获取merge标签的parent
                final ViewGroup viewGroup = (ViewGroup) parent;
                // 3、获取布局参数
                final ViewGroup.LayoutParams params = viewGroup.generateLayoutParams(attrs);
                // 4、递归解析每个子元素
                rInflateChildren(parser, view, attrs, true);
                // 5、将子元素直接添加到merge标签的parent view中
                viewGroup.addView(view, params);
```
* `merge`的原理我们清楚了，分析另外一条线，没有`merge`标签,就会走到else的方法来。
```
                // 如果根节点是“merge”标签
                if (TAG_MERGE.equals(name)) {
                   
                } else {
                    final View temp = createViewFromTag(root, name, inflaterContext, attrs);
                    ViewGroup.LayoutParams params = null;
                    if (root != null) {
                             params = root.generateLayoutParams(attrs);
                        if (!attachToRoot) {
                            temp.setLayoutParams(params);
                        }
                    }
                    // 递归实例化跟节点的子View start inflating children
                    rInflateChildren(parser, temp, attrs, true);

                    if (DEBUG) {
                        System.out.println("-----> done inflating children");
                    }
                    if (root != null && attachToRoot) {
                        root.addView(temp, params); // TODO: 2018/6/6      返回父View
                    }
           
                    // Decide whether to return the root that was passed in or the
                    // top view found in xml.
                    // TODO: 2018/6/6  父View是空或者不把填充的View添加到父View)
                    if (root == null || !attachToRoot) {
                        result = temp;  // TODO: 2018/6/6 返回根节点View 
                    }
```
* 1、这个方法`createViewFromTag(View parent, String name, Context context, AttributeSet attrs)`
```
 private View createViewFromTag(View parent, String name, Context context, AttributeSet attrs) {
        return createViewFromTag(parent, name, context, attrs, false);
    }
```
* 最终调用的是这个方法`createViewFromTag(parent, name, context, attrs, false);`这个方法
* ，什么时候会走到下面来，`name.equals("view")`成立的情况，我记得以前我为了在布局中加了一根线就是这样写的，写快了，写成小写的了`view`，应该写成大写`View`.
```
 <view
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>
```
小写的view：我自己打的断点发现，如果根标签是view的，注意是小写的，view，这个name会为null ，接着下面就会出现空指针的异常---> 我很聪明 很牛逼
```
 View createViewFromTag(View parent, String name, Context context, AttributeSet attrs,
            boolean ignoreThemeAttr) {
        if (name.equals("view")) {
          //根据我的断点的话，只要进来的话，这个那么就会为null
            name = attrs.getAttributeValue(null, "class");
        }
```

```
        // Apply a theme wrapper, if allowed and one is specified.
        //如果允许的话，应用一个主题包装器，并指定一个。
        if (!ignoreThemeAttr) {
            final TypedArray ta = context.obtainStyledAttributes(attrs, ATTRS_THEME);
            final int themeResId = ta.getResourceId(0, 0);
            if (themeResId != 0) {
                context = new ContextThemeWrapper(context, themeResId);
            }
            ta.recycle();
        }
```
name ==null，就会抛出`ava.lang.NullPointerException: Attempt to invoke virtual method 'boolean java.lang.String.equals(java.lang.Object)' on a null object reference`,也就是这个原因，所以写代码还是要仔细，仔细！！

在这里有个很有意思的布局，开发中很少使用到，而且也不清楚，作者为啥写，姑且这样理解`Let's party like it's 1995! 写这个代码的哥们，在1995年经历了一件很开心的事情`
```
        //如果有这个标签的话，就出现这个布局，闪动的布局
        if (name.equals(TAG_1995)) {
            // Let's party like it's 1995! 写这个代码的哥们，在1995年经历了一件很开心的事情
            return new BlinkLayout(context, attrs);
        }
```
`BlinkLayout`使用的方式，blink 翻译过来会一闪一闪的
```
     <blink
            android:layout_width="wrap_content"
            android:layout_height="wrap_content" >
            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:background="@android:color/holo_blue_bright"
                android:text="blink单词也是这个意思.闪烁频率500毫秒" />
        </blink>

```
`BlinkLayout`代码如下,  `BLINK_DELAY ` 这个是闪动的时间间隔，如果开发中需要，改动这个时间就行了，感觉这个布局也没啥用！
```
public  class BlinkLayout extends FrameLayout {
    private static final int MESSAGE_BLINK = 0x42;
    private static final int BLINK_DELAY = 500;
    private boolean mBlink;
    private boolean mBlinkState;
    private final Handler mHandler;

    public BlinkLayout(Context context, AttributeSet attrs) {
        super(context, attrs);
        // TODO: 2018/5/22 对handler  源码比较熟悉的话，这里就是Handler另外的一种使用的方法
        mHandler = new Handler(new Handler.Callback() {
            @Override
            public boolean handleMessage(Message msg) {
                if (msg.what == MESSAGE_BLINK) {
                    if (mBlink) {
                        mBlinkState = !mBlinkState;
                        makeBlink();
                    }
                    //invalidate()是用来刷新View的，必须是在UI线程中进行工作。
                    // 比如在修改某个view的显示时，调用invalidate()才能看到重新绘制的界面
                    invalidate();
                    return true;
                }
                return false;
            }
        });
    }

    private void makeBlink() {
        Message message = mHandler.obtainMessage(MESSAGE_BLINK);
        mHandler.sendMessageDelayed(message, BLINK_DELAY);
    }

    @Override
    protected void onAttachedToWindow() {
        super.onAttachedToWindow();

        mBlink = true;
        mBlinkState = true;

        makeBlink();
    }

    @Override
    protected void onDetachedFromWindow() {
        super.onDetachedFromWindow();

        mBlink = false;
        mBlinkState = true;

        mHandler.removeMessages(MESSAGE_BLINK);
    }

    @Override
    protected void dispatchDraw(Canvas canvas) {
        //根据这个状态是否去dispatchDraw
        if (mBlinkState) {
            super.dispatchDraw(canvas);
        }
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        return super.onTouchEvent(event);
    }

}
```

下面是后续的代码，通过工厂创建了view。去定位过，在`public class Activity extends ContextThemeWrapper implements LayoutInflater.Factory2`,很多的地方，如果需要研究的，还需要更深层的源码理解，这里就先mark下，只需要知道view是通过工厂来得到的
```
        try {
            View view;
            if (mFactory2 != null) {
                //各个工厂先onCreateView()
                view = mFactory2.onCreateView(parent, name, context, attrs);
            } else if (mFactory != null) {
                view = mFactory.onCreateView(name, context, attrs);
            } else {
                view = null;
            }

            if (view == null && mPrivateFactory != null) {
                view = mPrivateFactory.onCreateView(parent, name, context, attrs);
            }

            if (view == null) {
                final Object lastContext = mConstructorArgs[0];
                mConstructorArgs[0] = context;
                try {
                    if (-1 == name.indexOf('.')) {
                        view = onCreateView(parent, name, attrs);
                    } else {
                        view = createView(name, null, attrs);
                    }
                } finally {
                    mConstructorArgs[0] = lastContext;
                }
            }

            return view;
        } catch (InflateException e) {
            throw e;

        } catch (ClassNotFoundException e) {
            final InflateException ie = new InflateException(attrs.getPositionDescription()
                    + ": Error inflating class " + name, e);
            ie.setStackTrace(EMPTY_STACK_TRACE);
            throw ie;

        } catch (Exception e) {
            final InflateException ie = new InflateException(attrs.getPositionDescription()
                    + ": Error inflating class " + name, e);
            ie.setStackTrace(EMPTY_STACK_TRACE);
            throw ie;
        }
    }
```
* 2、关于这个方法`rInflateChildren(parser, temp, attrs, true);`:递归实例化跟节点的子View.其实调用的还是这个方法`rInflate(parser, root, inflaterContext, attrs, false);`,文章的前部分贴出了这部分的代码
```
    final void rInflateChildren(XmlPullParser parser, View parent, AttributeSet attrs,
            boolean finishInflate) throws XmlPullParserException, IOException {
        rInflate(parser, parent, parent.getContext(), attrs, finishInflate);
    }
```
主要是为了分析几个Tag
```
private static final String TAG_MERGE = "merge";
    private static final String TAG_INCLUDE = "include";
    private static final String TAG_1995 = "blink";
    private static final String TAG_REQUEST_FOCUS = "requestFocus";
    private static final String TAG_TAG = "tag";
    private static final String ATTR_LAYOUT = "layout";

```
requestFocus:解析请求焦点 ，这个控件一直有焦点 `requestFocus`,一般作用于 `EditText`,需要长期获取焦点的控件，但是真正的实现的方法，我们都是代码去实现了，很少这样使用，这里就说明一下，有这样的使用的方法！
```
 if (TAG_REQUEST_FOCUS.equals(name)) {
                pendingRequestFocus = true;
                consumeChildElements(parser);
   } 
```
```
       <EditText
            android:id="@+id/text"
            android:layout_width="match_parent"
            android:layout_height="wrap_content" >
            <!-- 当前控件处于焦点状态 -->
            <requestFocus />
        </EditText>
```
tag：在包含的视图上设置键标记,其实就是给view设置上了一个tag标签
```
if (TAG_TAG.equals(name)) {
                //在包含的视图上设置键标记。
                parseViewTag(parser, parent, attrs);
    } 
 final int key = ta.getResourceId(R.styleable.ViewTag_id, 0);
   final CharSequence value = ta.getText(R.styleable.ViewTag_value);
   view.setTag(key, value);
```
merge:在这里如果解析到了`merge`标签的话，这里就直接抛出异常了，必须是根元素！两种情况下，一种是套用了两个merge标签，或者是一个ViewGruop标签嵌套了一个merge，都会抛出异常
```
 if (TAG_MERGE.equals(name)) {
                // TODO: 2018/5/23 merge  作为布局里面的元素 会报错的哦 ，注意哦
                //想象下 两个merge 标签重合在一起
                throw new InflateException("<merge /> must be the root element");
  }
```
include:标签，在这里如果`include`作为了一个根元素，也会抛出异常，当然，如果在代码中见到别人那么用的话，建议自己去面壁思过会！
```
if (TAG_INCLUDE.equals(name)) {// 如果xml中的节点是include节点，则调用parseInclude方法
                if (parser.getDepth() == 0) {
                    throw new InflateException("<include /> cannot be the root element");
                }
                //解析include标签
                parseInclude(parser, context, parent, attrs);
   }
```
 `parseInclude(parser, context, parent, attrs);`,这个方法也就是`include`必须走的方法，搞明白的话，就可以搞明白原理，而且在特殊的情况下，`include`标签会和 `merge` 标签一起使用。这个方法也可以解释`include`标签的原理！
* 可以从这里可以看到，`include`只能在`ViewGroup`中使用，在 'View'中使用的话，会报错
```
   private void parseInclude(XmlPullParser parser, Context context, View parent,
            AttributeSet attrs) throws XmlPullParserException, IOException {
        int type;
   if (parent instanceof ViewGroup) {
            // Apply a theme wrapper, if requested. This is sort of a weird
            // edge case, since developers think the <include> overwrites
            // values in the AttributeSet of the included View. So, if the
            // included View has a theme attribute, we'll need to ignore it.
            //如果VIEW视图有一个主题属性，我们需要忽略它。
            final TypedArray ta = context.obtainStyledAttributes(attrs, ATTRS_THEME);
            final int themeResId = ta.getResourceId(0, 0);
            final boolean hasThemeOverride = themeResId != 0;
            if (hasThemeOverride) {
                context = new ContextThemeWrapper(context, themeResId);
            }
            ta.recycle();

            //如果布局指向主题属性，我们必须
            //按摩该值以从中获取资源标识符。
            int layout = attrs.getAttributeResourceValue(null, ATTR_LAYOUT, 0);
            if (layout == 0) {
                final String value = attrs.getAttributeValue(null, ATTR_LAYOUT);
                // TODO: 2018/6/6 include标签中没有设置layout属性，会抛出异常
                if (value == null || value.length() <= 0) {//
                    throw new InflateException("You must specify a layout in the"
                            + " include tag: <include layout=\"@layout/layoutID\" />");
                }

                // Attempt to resolve the "?attr/name" string to an attribute
                // within the default (e.g. application) package.
                layout = context.getResources().getIdentifier(
                        value.substring(1), "attr", context.getPackageName());

            }

            // The layout might be referencing a theme attribute.
            if (mTempValue == null) {
                mTempValue = new TypedValue();
            }
            if (layout != 0 && context.getTheme().resolveAttribute(layout, mTempValue, true)) {
                layout = mTempValue.resourceId;
            }

            if (layout == 0) {
                final String value = attrs.getAttributeValue(null, ATTR_LAYOUT);
                throw new InflateException("You must specify a valid layout "
                        + "reference. The layout ID " + value + " is not valid.");
            } else {
                final XmlResourceParser childParser = context.getResources().getLayout(layout);

                try {// 获取属性集，即在include标签中设置的属性
                    final AttributeSet childAttrs = Xml.asAttributeSet(childParser);

                    while ((type = childParser.next()) != XmlPullParser.START_TAG &&
                            type != XmlPullParser.END_DOCUMENT) {
                        // Empty.
                    }

                    if (type != XmlPullParser.START_TAG) {
                        throw new InflateException(childParser.getPositionDescription() +
                                ": No start tag found!");
                    }
                   // 1、解析include中的第一个元素
                    final String childName = childParser.getName();
                   // 如果第一个元素是merge标签，那么调用rInflate函数解析
                    if (TAG_MERGE.equals(childName)) {
                        // The <merge> tag doesn't support android:theme, so
                        // nothing special to do here.
                        rInflate(childParser, parent, context, childAttrs, false);
                    } else {
                        //2、我们例子中的情况会走到这一步,首先根据include的属性集创建被include进来的xml布局的根view
                        // 这里的根view对应为my_title_layout.xml中的RelativeLayout
                        final View view = createViewFromTag(parent, childName,
                                context, childAttrs, hasThemeOverride);
                        final ViewGroup group = (ViewGroup) parent;

                        final TypedArray a = context.obtainStyledAttributes(
                                attrs, R.styleable.Include);
                        //这里就是设置的<include>布局设置的id ,如果include设置了这个id ，那么这个id就不等于 View.No_ID=-1
                        final int id = a.getResourceId(R.styleable.Include_id, View.NO_ID);
                        final int visibility = a.getInt(R.styleable.Include_visibility, -1);
                        a.recycle();

                        // We try to load the layout params set in the <include /> tag.
                        // If the parent can't generate layout params (ex. missing width
                        // or height for the framework ViewGroups, though this is not
                        // necessarily true of all ViewGroups) then we expect it to throw
                        // a runtime exception.
                        // We catch this exception and set localParams accordingly: true
                        // means we successfully loaded layout params from the <include>
                        // tag, false means we need to rely on the included layout params.
                        ViewGroup.LayoutParams params = null;
                        try {
                            // 获3、取布局属性
                            params = group.generateLayoutParams(attrs);
                        } catch (RuntimeException e) {
                            // Ignore, just fail over to child attrs.
                        }
                        // 被inlcude进来的根view设置布局参数
                        if (params == null) {
                            params = group.generateLayoutParams(childAttrs);
                        }
                        view.setLayoutParams(params);
                         // 4、Inflate all children. 解析所有子控件
                        // Inflate all children.
                        rInflateChildren(childParser, view, childAttrs, true);

                        if (id != View.NO_ID) {
                         // 5、将include中设置的id设置给根view,因此实际上my_title_layout.xml中
                          // 的RelativeLayout的id会变成include标签中的id，include不设置id，那么也可以通过relative的找到.
                            view.setId(id);
                        }

                        switch (visibility) {
                            case 0:
                                view.setVisibility(View.VISIBLE);
                                break;
                            case 1:
                                view.setVisibility(View.INVISIBLE);
                                break;
                            case 2:
                                view.setVisibility(View.GONE);
                                break;
                        }
                        // 6、将根view添加到父控件中
                        group.addView(view);
                    }
                } finally {
                    childParser.close();
                }
            }
        } else {
            throw new InflateException("<include /> can only be used inside of a ViewGroup");
        }
        //默认的可见性使得LayoutInflater_Delegate代表可以调用它。
        LayoutInflater.consumeChildElements(parser);
```
* 分段分析上面的那段代码，
1、如果VIEW视图有一个主题属性，我们需要忽略它。
```       
            //如果VIEW视图有一个主题属性，我们需要忽略它。
            final TypedArray ta = context.obtainStyledAttributes(attrs, ATTRS_THEME);
            final int themeResId = ta.getResourceId(0, 0);
            final boolean hasThemeOverride = themeResId != 0;
            if (hasThemeOverride) {
                context = new ContextThemeWrapper(context, themeResId);
            }
            ta.recycle();
```
2、如果布局指向主题属性，我们必须从该值以从中获取资源标识符！
```
            //如果布局指向主题属性，我们必须
            //按摩该值以从中获取资源标识符。
            int layout = attrs.getAttributeResourceValue(null, ATTR_LAYOUT, 0);
```
3、在这里出现`value == null || value.length() <= 0`的话，就会抛出异常，也就是说`include`标签中没有设置`layout`属性，会抛出异常。
```
            if (layout == 0) {
                final String value = attrs.getAttributeValue(null, ATTR_LAYOUT);
                // TODO: 2018/6/6 include标签中没有设置layout属性，会抛出异常
                if (value == null || value.length() <= 0) {//
                    throw new InflateException("You must specify a layout in the"
                            + " include tag: <include layout=\"@layout/layoutID\" />");
                }
                // Attempt to resolve the "?attr/name" string to an attribute
                // within the default (e.g. application) package.
                layout = context.getResources().getIdentifier(
                        value.substring(1), "attr", context.getPackageName());

            }
```
3、解析`include`中的第一个元素,如果这里第一个`childeName`是`merge`,那么就是 `include`和`merge` 一起使用的情况，调用的方法是`  rInflate(childParser, parent, context, childAttrs, false);`和上面一样，不会生成多余的层级
```
                   // 1、解析include中的第一个元素
                    final String childName = childParser.getName();
                   // 如果第一个元素是merge标签，那么调用rInflate函数解析
                    if (TAG_MERGE.equals(childName)) {
                        // The <merge> tag doesn't support android:theme, so
                        // nothing special to do here.
                        rInflate(childParser, parent, context, childAttrs, false);
                    } 
```
4、首先根据include的属性集创建被include进来的xml布局的根view,这个`view`其实就`include` 进来的根`ViewGroup`,因为在前面已经把`merge`的情况过滤掉了，
```
                     else {
                        //2、我们例子中的情况会走到这一步,首先根据include的属性集创建被include进来的xml布局的根view
                        final View view = createViewFromTag(parent, childName,
                                context, childAttrs, hasThemeOverride);
```
5、特别关心这个`final int id = a.getResourceId(R.styleable.Include_id, View.NO_ID);`函数，其实就是给`include`标签设是否设置了`id`,就好像下面的布局一样，如果设置了这个` android:id="@+id/my_title_ly"`
```
    <include
        android:id="@+id/my_title_ly"
        android:layout_width="match_parent"
        android:layout_height="200dp"
        layout="@layout/my_include_layout" />
```
这个是上面得`include`的布局`my_include_layout`
```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:id="@+id/my_title_parent_id"
    android:layout_height="wrap_content" >
   ....
</RelativeLayout>
```
下面的这段代码就会执行，这也就解释了：安卓`include`标签的找不到id的问题,因为你在`include`设置了这个`id`属性，那么你找这个`my_title_parent_id`这个id，`findViewById(R.id.my_title_parent_id) `肯定也会找出来为null！
```
                        final int id = a.getResourceId(R.styleable.Include_id, View.NO_ID);
                        if (id != View.NO_ID) {
                         // 5、将include中设置的id设置给根view,因此实际上my_title_layout.xml中
                          // 的RelativeLayout的id会变成include标签中的id，include不设置id，那么也可以通过relative的找到.
                            view.setId(id);
                        }

}
```
 为什么`findViewById(R.id.my_title_parent_id) `会找出来为null,简单叙述下就是在`Activity`中调用方法其实是`Window`的`findViewById`,`getDecorView().findViewById(id)`,也就是`PhoneWindow`里面的`DecorView`的`findViewById`,也就是`View`的`findViewById`;可以看这边文章[Android源码分析（事件传递）](https://www.jianshu.com/p/f7e3a14daf51)涉及到这几个类了。如下面的方法，这个View 有id，找id的话就会走`findViewTraversal`这个方法，找的id和`setId`不一样的话，就会返回null，这就是找不到 `id`的原因。这个原因和`ViewStub`标签中设置了`inflatedId`一样的效果，下篇文章会讲到，非常有意思！已经写好[Android源码分析(ViewStub源码解析)](https://www.jianshu.com/p/63a066e7a5a9)
```
    public final <T extends View> T findViewById(@IdRes int id) {
       if (id == NO_ID) {
            return null;
      }
       return findViewTraversal(id);
    }
   protected <T extends View> T findViewTraversal(@IdRes int id) {
      if (id == mID) {
           return (T) this;
      }
      return null;
    }
```
View中设置`id`的方法
```
    public void setId(@IdRes int id) {
      mID = id;
       if (mID == View.NO_ID && mLabelForId != View.NO_ID) {
            mID = generateViewId();
       }
   }
```
6、关于`final int visibility = a.getInt(R.styleable.Include_visibility, -1);`这行代码，也就是说我们设置了`include`标签'visibility'其实也就是设置`layout`的`visibility`的属性！

```
  final int visibility = a.getInt(R.styleable.Include_visibility, -1);
    switch (visibility) {
                            case 0:
                                view.setVisibility(View.VISIBLE);
                                break;
                            case 1:
                                view.setVisibility(View.INVISIBLE);
                                break;
                            case 2:
                                view.setVisibility(View.GONE);
                                break;
                        }
```
7、将根view添加到父控件中,最后都会走到这里来，只不过使用了`merge`是不会产生多余的层级的，但是使用的时候有场景的要求！
```
        group.addView(view);
```
 
* 最后做了一张图，谢谢
![LayoutInflater.from(this).inflate(resId,null);代码解析.png](https://upload-images.jianshu.io/upload_images/5363507-3328730701d08022.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 说明几点
  *   布局转化为View，关键方法就是`LayoutInflater.from(this),inflate()`
  * `    PhoneWindow`这个类很关键
  *   如果需要打断点，建议使用模拟器，手机的厂商基本上改动过`framework`的代码，打出来不准确
  * `merge`原理就是通过`addView(view)`,从而减少了层级
   * `BlinkLayout`这个是`LayoutInflater`的内部类，感觉是谷歌工程师皮了一下
 * `include`只能在ViewGroup中使用，在 'View'结点中使用的话，会报错
 