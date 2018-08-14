---
title: 如何用AndroidStudio查看和解决OOM
date: 2017-04-17 16:43:40
tags: OOM
---
## 1、什么是oom
 一句话：c++ 中内存的泄漏指定的new出来的对象 ，没有delete掉，变成了空指针.java中指的是new出来的对象放在heap上，无法GC。安卓中的四种引用（强引用就是我报oom，也不会让你gc我，软引用是，空间不够的话，gc自己，弱引用是在gc 的时候，不管你的空间是不是不够，都可以gc，虚引用随时都可以gc）
>#### 为什么要性能优化
>Android每一个应用都是运行的独立的Dalivk虚拟机，根据不同的手机分配的可用内存可能只有（32M、64M等），所谓的4GB、6GB运行内存其实对于我们的应用不是可以任意索取
>优秀的算法与效率低下的算法之间的运行效率要远远超过计算机硬件的的发展，虽然手机单核、双核到4核、8核的发展，但性能优化任然不可忽略
>现在一般的用户都不会重启手机，可能一个月都不会重启。像微信这样的APP，每天都在使用。如果一旦发生内存泄漏，那么可能一点一点的累积，程序就会出现OOM。

<!--  more  -->

##2、常见的安卓oom

1、adapter没有使用缓存的convertview（listview）

2、bitmap对象没有被 recycle（）掉在源码中有一句话是 

since the normal GC percess will free up this mermory   意思就是你要主动调用这个方法才会释放内存

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/5363507-08e192da2e1de089.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3、context使用不正确，不要对activity context保持长生命周期的引用，竟可能使用applicationcontext代替  context的地方进行替换。

4、数据库的 cursor必须手动的关闭，流对象的关闭、webview的销毁。   

5、注册了系统的服务，但onDestory未注销

```
//注册了系统的服务
SensorManager sensorManager = getSystemService(SENSOR_SERVICE);
Sensor sensor = sensorManager.getDefaultSensor(Sensor.TYPE_ALL);
sensorManager.registerListener(this,sensor,SensorManager.SENSOR_DELAY_FASTEST);
```

```
//不需要用的时候记得移除监听
    sensorManager.unregisterListener(listener);
```
6、还用一种的情况
```
//add监听，放到集合里面
    tv.getViewTreeObserver().addOnWindowFocusChangeListener(new ViewTreeObserver.OnWindowFocusChangeListener() {
        @Override
        public void onWindowFocusChanged(boolean b) {
            //监听view的加载，view加载出来的时候，计算他的宽高等。
            //  在这里必须要注销，要不然会内存溢出
              //计算完后，一定要移除这个监听
            tv.getViewTreeObserver().removeOnWindowFocusChangeListener(this);
        }
    });
```
##3、**jvm**
静态的

>静态的存储区，内存在程序编译的时候就已经分配好了，这块内存在程序整个运行期间都一直存在，它主要存放静态数据、全局的static数据和一些常量。

栈式的

>在执行方法时，方法一些内部变量的存储都可以放在栈上面创建，方法执行结束的时候这些存储单元就会自动被注释掉。栈 内存包括分配的运算速度很快，因为内在在处理器里面。当然容量有限，并且栈式一块连续的内存区域，大小是由操作系统决定的，他先进后 出，进出完成不会产生碎片，运行效率高且稳定

堆式的

>也叫动态内存 。我们通常使用new 来申请分配一个内存。这里也是我们讨论内存泄漏优化的关键存储区。GC会根据内存的使用情况，对堆内存里的垃圾内存进行回收。堆内存是一块不连续的内存区域，如果频繁地new/remove会造成大量的内存碎片，GC频繁的回收，导致内存抖动，这也会消耗我们应用的性能
我们知道可以调用 System.gc();进行内存回收，但是GC不一定会执行。面对GC的机制，我们是否无能为力？其实我们可以通过声明一些引用标记来让GC更好对内存进行回收。

类型	回收时机	生命周期
>StrongReference （强引用）	任何时候GC是不能回收他的，哪怕内存不足时，系统会直接抛出异常OutOfMemoryError，也不会去回收	进程终止
SoftReference （软引用）	当内存足够时不会回收这种引用类型的对象，只有当内存不够用时才会回收	内存不足，进行GC的时候
WeakReference （弱引用）	GC一运行就会把给回收了	GC后终止
PhantomReference (虚引用)	如果一个对象与虚引用关联，则跟没有引用与之关联一样，在任何时候都可能被垃圾回收器回收	任何时候都有可能
开发时，为了防止内存溢出，处理一些比较占用内存并且生命周期长的对象时，可以尽量使用软引用和弱引用。
***

##4、内存泄漏的定义

当一个对象已经不需要使用了，本该被回收时，而有另外一个正在使用的对象持有它的引用，从而导致了对象不能被GC回收。这种导致了本该被回收的对象不能被回收而停留在堆内存中，就产生了内存泄漏

内存泄漏与内存溢出的区别
***
内存泄漏（Memory Leak）
进程中某些对象已经没有使用的价值了，但是他们却还可以直接或间接地被引用到GC Root导致无法回收。当内存泄漏过多的时候，再加上应用本身占用的内存，日积月累最终就会导致内存溢出OOM

内存溢出（OOM）
当 应用的heap资源超过了Dalvik虚拟机分配的内存就会内存溢出
***
内存泄漏带来的影响

应用卡顿
泄漏的内存影响了GC的内存分配，过多的内存泄漏会影响应用的执行效率

应用异常（OOM）
过多的内存泄漏，最终会导致 Dalvik分配的内存，出现OOM
##5、如何使用安卓是studio的工具查看内存溢出
####第一种情况
```
   //泄漏 activity 最简单的方法就是在 activity 类中定义一个 static 变量，并且将其指向一个运行中的 activity 实例。
    // 如果在 activity 的生命周期结束之前，没有清除这个引用，那它就会泄漏了。这是因为 activity
    // （例如 MainActivity） 的类对象是静态的，一旦加载，就会在 APP 运行时一直常驻内存，
    // 因此如果类对象不卸载，其静态成员就不会被垃圾回收。
    public static MainActivity mMainActivity;
       @Override
    public void onClick(View v) {
                mMainActivity = this;
                Intent intent = new Intent(this, TwoActivity.class);
                startActivity(intent);
                finish();
}
```
原理就是简单的点击事件，同时在这个对象为一个静态的持有static的变量，开启第二个activity的时候，这个activity关闭掉

通过monitors查看


![Paste_Image.png](http://upload-images.jianshu.io/upload_images/5363507-2cc8e8e8f9e5f749.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
然后出现了

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/5363507-c0271c13c309b3d6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/5363507-9abb0e1d3652af49.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
通过点击上面的额package tree view 进入到我们自己创建的类

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/5363507-043e569635edeea3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
会发现一个有趣的现象，mainactivity不是已经销毁了，为什么在java heap中还看的到？

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/5363507-5b70b55f99a4961c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
原来是我们的变量mMainActivity中还持有activity的对象 ，虽然我们的activity的ondestory的方法已经走了，但是实际在内存中是没有销毁的，所以这里就有oom的可能，进入相对应的activity

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/5363507-ab69df31832cfcb2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
真的发现是这样，这就可以通过这种方式发现了oom的可能了



通过Memory Usage去发现我们activity在内存中存在多少
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/5363507-73772fe00a35c1b6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
我们第一步知道，我们开了两个activity，开启第二个的activity的时候，我们的第一个activity已经finish（）；所以内存中存在一个activity
But  看图

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/5363507-0afafcbc27fc7397.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
我擦，是不是我眼睛瞎了，咋还有两个activity呢？对，就是怎么神奇，你即使finish掉了，但是在内存中还是存在的，所以必须处理，要不然以后的应用有崩掉的可能性
####第二种情况:view持有static
```
    //view 持有静态的变量
//    view 被加入到界面中，它就会持有 context 的强引用，也就是我们的 activity。
// 由于我们通过一个静态成员引用了这个 view，
//    所以我们也就引用了 activity，因此 activity 就发生了泄漏。
//    所以一定不要把加载的 view 赋值给静态变量，如果你真的需要，那一定要确保在 activity 销毁之前将其从 view 层级中移除。
    private static View mView;
    mView = findViewById(R.id.btn_goto_two_activity);
```

####第三种情况：单利
当调用getmUtils时，如果传入的context是Activity的context。只要这个单例没有被释放，那么这个Activity也不会被释放一直到进程退出才会释放。
```
public class TestUtils {
    private static  TestUtils mUtils;
    private Context mContext;
    private TestUtils(Context context){
        mContext=context;
    }
    public static TestUtils getmUtils(Context context){
        if (mUtils!=null){
            mUtils=new TestUtils(context);
        }
        return mUtils;
    }
}

```
能使用Application的Context就不要使用Activity的Content，Application的生命周期伴随着整个进程的周期
####第四种情况：handler泄漏
mHandler是Handler的非静态匿名内部类的实例，所以它持有外部类Activity的引用，我们知道消息队列是在一个Looper线程中不断轮询处理消息，那么当这个Activity退出时消息队列中还有未处理的消息或者正在处理消息，而消息队列中的Message持有mHandler实例的引用，mHandler又持有Activity的引用，所以导致该Activity的内存资源无法及时回收，引发内存泄漏。
```
private MyHandler mHandler = new MyHandler(this);
    private TextView mTextView ;
    private static class MyHandler extends Handler {
        private WeakReference<Context> reference;
        public MyHandler(Context context) {
            reference = new WeakReference<>(context);
        }
        @Override
        public void handleMessage(Message msg) {
            MainActivity activity = (MainActivity) reference.get();
            if(activity != null){
                activity.mTextView.setText("");
            }
        }
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mTextView = (TextView)findViewById(R.id.textview);
        loadData();
    }

    private void loadData() {

        Message message = Message.obtain();
        mHandler.sendMessage(message);
    }
```
解决的方法

创建一个静态Handler内部类，然后对Handler持有的对象使用弱引用，这样在回收时也可以回收Handler持有的对象，这样虽然避免了Activity泄漏，不过Looper线程的消息队列中还是可能会有待处理的消息，所以我们在Activity的Destroy时或者Stop时应该移除消息队列中的消息
```
 private MyHandler mHandler = new MyHandler(this);
    private TextView mTextView ;
    private static class MyHandler extends Handler {
        private WeakReference<Context> reference;
        public MyHandler(Context context) {
            reference = new WeakReference<>(context);
        }
        @Override
        public void handleMessage(Message msg) {
            MainActivity activity = (MainActivity) reference.get();
            if(activity != null){
                activity.mTextView.setText("");
            }
        }
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mTextView = (TextView)findViewById(R.id.textview);
        loadData();
    }

    private void loadData() {
        //...request
        Message message = Message.obtain();
        mHandler.sendMessage(message);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        mHandler.removeCallbacksAndMessages(null);
    }
}
```
####第五种情况：线程造成的内存泄漏
异步任务和Runnable都是一个匿名内部类，因此它们对当前Activity都有一个隐式引用。如果Activity在销毁之前，任务还未完成， 那么将导致Activity的内存资源无法回收，造成内存泄漏
```
   new AsyncTask<Void, Void, Void>() {
            @Override
            protected Void doInBackground(Void... params) {
                SystemClock.sleep(10000);
                return null;
            }
        }.execute();


        new Thread(new Runnable() {
            @Override
            public void run() {
                SystemClock.sleep(10000);
            }
        }).start();
```
解决的方法：尽量的使用静态的内部类
使用 静态内部类，避免了Activity的内存资源泄漏，当然在Activity销毁时候也应该取消相应的任务AsyncTask::cancel()，避免任务在后台执行浪费资源
```
 static class MyAsyncTask extends AsyncTask<Void, Void, Void> {
        private WeakReference<Context> weakReference;

        public MyAsyncTask(Context context) {
            weakReference = new WeakReference<>(context);
        }

        @Override
        protected Void doInBackground(Void... params) {
            SystemClock.sleep(10000);
            return null;
        }

        @Override
        protected void onPostExecute(Void aVoid) {
            super.onPostExecute(aVoid);
            MainActivity activity = (MainActivity) weakReference.get();
            if (activity != null) {
                //...
            }
        }
    }
    static class MyRunnable implements Runnable{
        @Override
        public void run() {
            SystemClock.sleep(10000);
        }
    }
//——————
    new Thread(new MyRunnable()).start();
    new MyAsyncTask(this).execute();
```
####第六种的情况:资源未关闭造成的内存泄漏


对于使用了BraodcastReceiver，ContentObserver，File，Cursor，Stream，Bitmap等资源的使用，应该在Activity销毁时及时关闭或者注销，否则这些资源将不会被回收，造成内存泄漏

解决：在Activity销毁时及时关闭或者注销
####第七种情况：使用静态的方法

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/5363507-418e0f47c02ba722.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
比如说我在一个activity开启一个dialog的方法，那么持有这个变量context
```
   private static BaseDialog baseDialog;
    public static void sendVideo(Context context) {
        baseDialog = new BaseDialog(context)
                .setCustomerContent(R.layout.nim_layout_shiming_test_layout)
                .setDialogSize(145.0f, 145.0f)
                .setWindowBackground(R.color.clr_000000)
                .setViewOnClickListener(R.id.shiming_text_send_video, new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        //开启activity录像
                        videoFile = openCaptureVideo(context);
//                        gotoSendVideo(file,listener);
                    }
                }).setViewOnClickListener(R.id.shiming_text_send_in_phone, new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        System.out.println( "shiming phone");
                    }
                });
        baseDialog.setCanCancelOutside(true);
        baseDialog.show();
    }

```
由于我们的dialog的对象为静态的，所以就有了内存泄漏的可能，但是我们的context的对象必须为activity，因为我们的dialog必须持有的activity的context才能开启，要不然会报错，窗体泄漏
todo
##end  
地址：https://github.com/Shimingli/DemoOOM

