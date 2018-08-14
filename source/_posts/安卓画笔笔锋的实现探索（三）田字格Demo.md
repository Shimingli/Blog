---
title: 安卓画笔笔锋的实现探索（三）田字格Demo
date: 2018-02-24 16:30:33
tags: 田字格Demo
---
### Demo的下载地址
![下载地址](http://upload-images.jianshu.io/upload_images/5363507-fd626ece6fbccfb5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<!--  more  -->
### 效果图：如果对的效果不太明确的地方，请移步上两篇文章
![效果图](http://upload-images.jianshu.io/upload_images/5363507-408927f77c209be3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
####Demo中做了哪些事情
1、提供画板，以及收起画板的动作
2、插入空格
3、换行
4、删除或者长按删除
5、切换笔的样式
6、根据手指抬起来自动插入已绘制的图形到EditText中
7、待续
####1、提供画板，以及收起画板的动作
使用DrawViewLayout 继承FrameLayout 
```
public class DrawViewLayout extends FrameLayout implements View.OnClickListener, View.OnLongClickListener {

    private RelativeLayout mShowKeyboard;
    private RelativeLayout mGotoPreviousStep;
    private RelativeLayout mClearCanvas;
    private NewDrawPenView mDrawView;
    private RelativeLayout mSaveBitmap;
    private ViewStub mViewStub;
    private View mChild;
    private Context mContext;
    private ImageView mUpOrDownIcon;
    private LayoutInflater mInflater;
    private int mPenConfig;
    private boolean mIsShowKeyB;

    public DrawViewLayout(@NonNull Context context) {
        this(context, null);
    }


    public DrawViewLayout(@NonNull Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
        mContext = context;
        initView();

    }
    private void initView() {
        mInflater = LayoutInflater.from(getContext());
        mChild = mInflater.inflate(R.layout.brush_weight_layout, this, false);
        addView(mChild);
        mShowKeyboard = (RelativeLayout) findViewById(R.id.rll_show_keyb_container);
        mGotoPreviousStep = (RelativeLayout) findViewById(R.id.rll_show_space_container);//空格
        mClearCanvas = (RelativeLayout) findViewById(R.id.rll_show_newline_container);
        mSaveBitmap = (RelativeLayout) findViewById(R.id.rll_show_delete_container);
        mViewStub = (ViewStub) findViewById(R.id.draw_view);
        //需要关心的selector的id
        mUpOrDownIcon = (ImageView) findViewById(R.id.rll_show_keyb_container_icon);
        setOnClickListenerT();
    }
```
布局文件使用的ViewStub轻量级的控件:使用Viewstub的在不需要弹出键盘的时候，渲染不占内存
```
    <ViewStub
        android:id="@+id/draw_view"
        android:layout_width="match_parent"
        android:layout_height="@dimen/dp_280"
        android:layout_gravity="center"
        android:layout_marginLeft="@dimen/dp_40"
        android:layout_marginRight="@dimen/dp_40"
        android:layout_marginTop="@dimen/dp_10"
        android:layout_marginBottom="@dimen/dp_10"
        android:layout="@layout/draw_view_layout"
        android:visibility="visible"/>
```

判断是否填充过，没有的话就不填充，第一版设计的时候，产品经理说不用进入页面就弹出软件盘，那使用ViewStub，只要没有弹出来，渲染不占内存，后续产品经理说要弹出来，所以这里进入的时候，我都设计出弹出键盘
```
  if (mViewStub.getParent() != null) {
            mViewStub.inflate();
       }
//隐藏软键盘
  if (mDrawView.getVisibility() == GONE) {
            mIsShowKeyB=true;
            mViewStub.setVisibility(VISIBLE);
            mUpOrDownIcon.setSelected(true);
            Toast.makeText(mContext,"显示键盘",Toast.LENGTH_SHORT).show();
            mDrawView.setVisibility(VISIBLE);
        } else if (mDrawView.getVisibility() == VISIBLE) {
            mIsShowKeyB=false;
            Toast.makeText(mContext,"隐藏键盘",Toast.LENGTH_SHORT).show();
            mDrawView.setVisibility(GONE);
            mViewStub.setVisibility(GONE);
            mUpOrDownIcon.setSelected(false);
        }
```
####2、插入空格
在这里说个开发中的趣事，当说要插入一个空格的时候，无非就是往EditText中插入一个空格吧，就用代码插入，但是后面ui说，你咋插入的空格那么小，我说本来就只有插入这样小啊，我能咋办，我也很绝望，后续使用的是插入一张空白的图
```
   @Override
    public void needSpace() {
        NewDrawPenView view = mDrawViewLayout.getSaveBitmap();
        if (view != null) {
            if (view.getHasDraw()) {
                mBitmap = view.getBitmap();
                mHandler.post(runnableUi);
                //保持一个联系
                mHandler.postDelayed(new Runnable() {
                    @Override
                    public void run() {
                        mIsCreateBitmap = true;
                        if (mCreatBimap == null) {
                            mCreatBimap = creatBimap();
                        }
                        mHandler.post(runnableUi);
                    }
                }, 100);
            } else {
                mIsCreateBitmap = true;
                if (mCreatBimap == null) {
                    mCreatBimap = creatBimap();
                }
                mHandler.post(runnableUi);
            }
        }
        mHandler.removeCallbacks(runnable);
    }
```
创建一个空白的bitmap  这里其实不用创建一个和手机设备一样大的空白的bitmap，只创建一个和文本生成的字体一样大的就行了，总感觉浪费性能，哈哈
```
//创建一个bitmap 
    private Bitmap creatBimap() {
        ColorDrawable drawable = new ColorDrawable(Color.TRANSPARENT);
        DisplayMetrics dm = new DisplayMetrics();
        getWindowManager().getDefaultDisplay().getMetrics(dm);
        Bitmap bitmap = Bitmap.createBitmap(dm.widthPixels, dm.heightPixels, Bitmap.Config.ARGB_8888);
        Canvas canvas = new Canvas(bitmap);
        drawable.draw(canvas);
        return bitmap;
    }
```
把bitmap插入到edittext的线程
```
  Runnable runnableUi = new Runnable() {
        @Override
        public void run() {
            if (mIsCreateBitmap) {
                //110
                mBitmapResize = BitmapDrawUtils.resizeImage(mCreatBimap, mAllHandDrawSize, mAllHandDrawSize);
                mIsCreateBitmap = false;
            } else {
                mBitmapResize = BitmapDrawUtils.resizeImage(mBitmap, mAllHandDrawSize, mAllHandDrawSize);
            }
            if (mBitmapResize != null) {
                //根据Bitmap对象创建ImageSpan对象
                ImageSpan imageSpan = new ImageSpan(FieldCharacterShapeActivity.this, mBitmapResize);
                //创建一个SpannableString对象，以便插入用ImageSpan对象封装的图像
                full_name = LAST_NAME + System.currentTimeMillis();
                String s =FONT_NAME_HEAD + full_name + FONT_NAME_TAIL;
                SpannableString spannableString = new SpannableString(s);
                //  用ImageSpan对象替换face
                spannableString.setSpan(imageSpan, 0, s.length(), Spannable.SPAN_EXCLUSIVE_EXCLUSIVE);
                //将选择的图片追加到EditText中光标所在位置
                //                EditText ed = mSvContent.getFocusEditText();
                EditText ed = mRetContent.getLastFocusEdit();
                int index = ed.getSelectionStart(); //获取光标所在位置
                Editable edit_text = ed.getEditableText();
                if (index < 0 || index >= edit_text.length()) {
                    edit_text.append(spannableString);
                } else {
                    edit_text.insert(index, spannableString);
                }
                testStorage();
            }
            mDrawViewLayout.clearScreen();
        }
    };
```

```
   /**
     * 图片缩放
     * @param originalBitmap 原始的Bitmap
     * @param newWidth 自定义宽度
     * @return 缩放后的Bitmap
     */
    public static Bitmap resizeImage(Bitmap originalBitmap, int newWidth, int newHeight){
        if (originalBitmap==null||originalBitmap.getWidth()==0||originalBitmap.getHeight()==0){
            return null;
        }
        int width = originalBitmap.getWidth();
        int height = originalBitmap.getHeight();
        //定义欲转换成的宽、高
//            int newWidth = 200;
//            int newHeight = 200;
        //计算宽、高缩放率
        float scanleWidth = (float)newWidth/width;
        float scanleHeight = (float)newHeight/height;
        //创建操作图片用的matrix对象 Matrix
        Matrix matrix = new Matrix();
        matrix.postScale(scanleWidth,scanleHeight);
        // 创建新的图片Bitmap
        Bitmap resizedBitmap = Bitmap.createBitmap(originalBitmap,0,0,width,height,matrix,true);
        return resizedBitmap;
    }
```
####3、换行
获取ScrollView最后的一个EditText，对了，这里没说，由于，需要滑动，所以自定义了一个ScrollView，可以插入很多控件，图片，文字和语音等等，好多的控件，由于这里不需要那么多，所以只保留了一个EditText
```
    /**
     * 这里是换行的需要
     */
    @Override
    public void creatNewLine() {
        EditText ed = mRetContent.getLastFocusEdit();
        int index = ed.getSelectionStart();
        Editable editable = ed.getText();
        editable.insert(index, "\n");
    }
```
####4、删除或者长按删除

代码如下
```
    @Override
    public void deleteOnClick() {
        if (mRetContent.getLastFocusEdit().getSelectionStart() == 0) {
            mRetContent.onBackspacePress(mRetContent.getLastFocusEdit());
            if (mHandler!=null) {
                mHandler.removeCallbacks(runnable);
            }
        } else {
            SystemUtils.sendKeyCode(67);
        }
    }
```
```
 /**
     * <pre>
     * 使用Instrumentation接口：对于非自行编译的安卓系统，无法获取系统签名，只能在前台模拟按键，不能后台模拟
     * 注意:调用Instrumentation的sendKeyDownUpSync方法必须另起一个线程，否则无效
     * @param keyCode
     *            按键事件(KeyEvent)的按键值
     * </pre>
     */
    public static void sendKeyCode(final int keyCode) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    // 创建一个Instrumentation对象
                    Instrumentation inst = new Instrumentation();
                    // 调用inst对象的按键模拟方法
                    inst.sendKeyDownUpSync(keyCode);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }
```
长按删除的事件呢，其实无非就是让这个方法不断的执行SystemUtils.sendKeyCode(67);在这里我使用枚举的单利模式，其实这样做不太好，因为枚举的单利比较消耗内存

```
package com.shiming.pen.field_character;

import android.annotation.SuppressLint;
import android.os.Handler;
import android.os.Message;

import com.shiming.pen.R;

import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;


/**
 * @author shiming
 * @version v1.0 create at 2017/9/13
 * @des 使用模式为enum的单利模式
 */
public enum Executor {
    INSTANCE;
    private ScheduledExecutorService mScheduledExecutorService;
    private DrawViewLayout.IActionCallback mCallback;

    public void setCallback(DrawViewLayout.IActionCallback callback) {
        mCallback = callback;
    }

    @SuppressLint("HandlerLeak")
    private Handler handler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            int viewId = msg.what;
            switch (viewId) {
                case R.id.rll_show_delete_container:
                    if (mCallback == null)
                        return;
                    mCallback.deleteOnLongClick();
                    break;
            }
        }
    };

    public void upData(int id) {
        final int vid = id;
        //只有一个线程，用来调度执行将来的任务
        mScheduledExecutorService = Executors.newSingleThreadScheduledExecutor();
        //多少时间执行一次
        mScheduledExecutorService.scheduleWithFixedDelay(new Runnable() {
            @Override
            public void run() {
                Message msg = new Message();
                msg.what = vid;
                handler.sendMessage(msg);
            }
        }, 0, 100, TimeUnit.MILLISECONDS);    //每间隔100ms发送Message
    }

    public void stop() {
        if (mScheduledExecutorService != null) {
            mScheduledExecutorService.shutdownNow();
            mScheduledExecutorService = null;
        }
    }
}

```
执行的方法这里
```
  @Override
    public void deleteOnLongClick() {
        if (mRetContent.getLastFocusEdit().getSelectionStart() == 0) {
            mRetContent.onBackspacePress(mRetContent.getLastFocusEdit());
            if (mHandler!=null) {
                mHandler.removeCallbacks(runnable);
            }
        } else {
            SystemUtils.sendKeyCode(67);
        }
    }
```

####5、切换笔的样式
这里笔的样式能有三种的情况，钢笔，水彩笔，和橡皮擦，由于我这里每次都自动的保存了图片，所以最后一次都是橡皮擦的模式，所以我只要判断不是Pen的模式，就给它赋予这种的模式，同时还需更具点击事件的变化而变化
```
  @Override
    public void onClick(View v) {
        int penConfig = mDrawViewLayout.getPenConfig();
        switch (v.getId()){
            case R.id.btn_change_pen:
                if (penConfig== IPenConfig.STROKE_TYPE_PEN){
                    penConfig=IPenConfig.STROKE_TYPE_BRUSH;
                }else {
                    penConfig=IPenConfig.STROKE_TYPE_PEN;
                }
                mDrawViewLayout.setPenConfig(penConfig);
                break;
        }
    }
```

```
   public void setPenConfig(int penConfig) {
       mDrawView.setCanvasCode(penConfig);
        mPenConfig=penConfig;
    }

```
最底层的方法 在这里 ，关键的地方需要invalidate一次，这样自定义view才知道需要变换
```
    public void setCanvasCode(int canvasCode) {
        mCanvasCode = canvasCode;
        switch (mCanvasCode) {
            case IPenConfig.STROKE_TYPE_PEN:
                mStokeBrushPen = new SteelPen(mContext);
                break;
            case IPenConfig.STROKE_TYPE_BRUSH:
                mStokeBrushPen = new BrushPen(mContext);
                break;

        }
        //设置
        if (mStokeBrushPen.isNull()){
            mStokeBrushPen.setPaint(mPaint);
        }
        invalidate();
    }
```
####6、根据手指抬起来自动插入已绘制的图形到EditText中
在onTouchEvent事件中，记录手指抬起来的时间，同时记录下必须需要停止的时间
```
  @Override
    public boolean onTouchEvent(MotionEvent event) {
        mIsCanvasDraw = true;
        //测试过程中，当使用到event的时候，产生了没有收到事件的问题，所以在这里需要obtian的一下
        MotionEvent event2 = MotionEvent.obtain(event);
        switch (event2.getActionMasked()) {
            case MotionEvent.ACTION_DOWN:
                //每次都是这个笔，因为项目里面就只有这个笔，如果多了，这里需要改动
                setCanvasCode(CANVAS_NORMAL);
                mVisualStrokePen.onDown(mVisualStrokePen.createMotionElement(event2));
                mGetTimeListner.stopTime();
                break;
            case MotionEvent.ACTION_MOVE:
                mVisualStrokePen.onMove(mVisualStrokePen.createMotionElement(event2));
                mGetTimeListner.stopTime();
                break;
            case MotionEvent.ACTION_UP:
                long time = System.currentTimeMillis();
                mGetTimeListner.getTime(time);
                mVisualStrokePen.onUp(mVisualStrokePen.createMotionElement(event2),mCanvas);
                break;
            default:
                break;
        }
        invalidate();
        return true;
    }
```
接口回调
```
        mDrawView.setGetTimeListener(new NewDrawPenView.TimeListener() {
            @Override
            public void getTime(long l) {
                mIActionCallback.getUptime(l);
            }

            @Override
            public void stopTime() {
                mIActionCallback.stopTime();
            }
        });
```

需要做什么，其实就是一个任务
```
    @Override
    public void getUptime(long l) {
        mOldTime = l;
        mHandler.postDelayed(runnable, 100);
    }

    @Override
    public void stopTime() {
        mHandler.removeCallbacks(runnable);
    }
```
当大于时间了，就使用handler发送一个消息
```
   final Runnable runnable = new Runnable() {
        @Override
        public void run() {
            long l1 = System.currentTimeMillis();
            if ((l1 - mOldTime) > HADN_DRAW_TIME) {
                mHandler.removeCallbacks(runnable);
                Message msg = mHandler.obtainMessage();
                msg.obj = true;
                msg.what = 0x123;
                mHandler.sendMessage(msg);
            } else {
                mHandler.postDelayed(this, 100);
            }

        }
    };
```
处理handler消息，关键就是这个  mBitmap = view.clearBlank(100);
```
 @SuppressLint("HandlerLeak")//麻痹
    private Handler mHandler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
            int what = msg.what;
            switch (what) {
                case 0x123:
                    try {
                        boolean obj = (boolean) msg.obj;
                        if (obj) {
                            NewDrawPenView view = mDrawViewLayout.getSaveBitmap();
                            if (view != null) {
                                //边距强行扫描
                                mBitmap = view.clearBlank(100);
                                mHandler.post(runnableUi);
                            }
                        }
                    } catch (Exception e) {

                    } finally {
                        mHandler.removeCallbacks(runnable);
                    }
                    break;
                case 0x124:
                    mRetContent.setVisibilityEdit(View.VISIBLE);
                    mRetContent.setVisibilityClose(View.VISIBLE);
                    mRetContent.getLastFocusEdit().setCursorVisible(true);
                    mRetContent.getLastFocusEdit().requestFocus();
                    break;
                case 0x125:
                    break;
            }
        }
    };
```
关于这个方法clearBlank（int  ），这个方法不是我写的，我又一次研究别人的一个Demo，测试下了这个效果非常牛逼，本来这个方法打算废弃掉，但是最后觉得太可惜了，写这个田字格的Demo又提出来了。其实还有种方法可以实现，就是我们知道画布的大小，同时也知道，需要生成的小图的大小，我们可以动态的算，这就是另外一种方法，在不同设备上有点小许差异，但是也还可以
```
 /**
     * 逐行扫描 清楚边界空白。功能是生成一张bitmap位于正中间，不是位于顶部，此关键的是我们画布需要
     * 成透明色才能生效
     * @param blank 边距留多少个像素
     * @return tks github E-signature
     */
    public Bitmap clearBlank(int blank) {
        if (mBitmap != null) {
            int HEIGHT = mBitmap.getHeight();//1794
            int WIDTH = mBitmap.getWidth();//1080
            int top = 0, left = 0, right = 0, bottom = 0;
            int[] pixs = new int[WIDTH];
            boolean isStop;
            for (int y = 0; y < HEIGHT; y++) {
                mBitmap.getPixels(pixs, 0, WIDTH, 0, y, WIDTH, 1);
                isStop = false;
                for (int pix : pixs) {
                    if (pix != mBackColor) {

                        top = y;
                        isStop = true;
                        break;
                    }
                }
                if (isStop) {
                    break;
                }
            }
            for (int y = HEIGHT - 1; y >= 0; y--) {
                mBitmap.getPixels(pixs, 0, WIDTH, 0, y, WIDTH, 1);
                isStop = false;
                for (int pix : pixs) {
                    if (pix != mBackColor) {
                        bottom = y;
                        isStop = true;
                        break;
                    }
                }
                if (isStop) {
                    break;
                }
            }
            pixs = new int[HEIGHT];
            for (int x = 0; x < WIDTH; x++) {
                mBitmap.getPixels(pixs, 0, 1, x, 0, 1, HEIGHT);
                isStop = false;
                for (int pix : pixs) {
                    if (pix != mBackColor) {
                        left = x;
                        isStop = true;
                        break;
                    }
                }
                if (isStop) {
                    break;
                }
            }
            for (int x = WIDTH - 1; x > 0; x--) {
                mBitmap.getPixels(pixs, 0, 1, x, 0, 1, HEIGHT);
                isStop = false;
                for (int pix : pixs) {
                    if (pix != mBackColor) {
                        right = x;
                        isStop = true;
                        break;
                    }
                }
                if (isStop) {
                    break;
                }
            }
            if (blank < 0) {
                blank = 0;
            }
            left = left - blank > 0 ? left - blank : 0;
            top = top - blank > 0 ? top - blank : 0;
            right = right + blank > WIDTH - 1 ? WIDTH - 1 : right + blank;
            bottom = bottom + blank > HEIGHT - 1 ? HEIGHT - 1 : bottom + blank;
            return Bitmap.createBitmap(mBitmap, left, top, right - left, bottom - top);
        } else {
            return null;
        }
    }
```

##最后，谢谢提出问题的大佬
##GitHub：https://github.com/Shimingli/WritingPen


