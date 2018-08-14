---
title: MVP网络框架Retorfit+Rxjava+Rxandroid
date: 2017-08-08 16:44:43
tags: MVP网络框架Retorfit+Rxjava+Rxandroid
---

* 时隔三年再次过来回看这个Demo，修复聚合接口不能访问的问题，完善下载的Demo
![image.png](https://upload-images.jianshu.io/upload_images/5363507-4d657789d66e123f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<!--  more  -->

目前我个人觉得是一套比较好的network的网络请求框架，大体时间用了8个小时，由于其中的网络请求不会来数据，就解决了一会bug，中间拖了5天由于在工作，所以今天晚上来完成这篇博客
1、先了解mvp的模式

![](http://upload-images.jianshu.io/upload_images/5363507-eadc63a77316195d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)包的结构，在外层可以套个ui的包，

逻辑处理交互到persenter去处理，在view层就关心数据和view的绑定

![](http://upload-images.jianshu.io/upload_images/5363507-2857080996687c46.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)在view中只需要new persenter（）

![](http://upload-images.jianshu.io/upload_images/5363507-d97be7aef14051de.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)点击事件去请求网络，或者在baseActivity中去封装一个方法只执行一次的网络请求

[图片上传中。。。（4）]数据回来再回调中去处理

![](http://upload-images.jianshu.io/upload_images/5363507-f54fb365f90b2bde.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)在p中去做接口会掉的操作

![](http://upload-images.jianshu.io/upload_images/5363507-72e015f3efad0088.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)omodel层拿到我偶们的service

![](http://upload-images.jianshu.io/upload_images/5363507-d62825c5843371dc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)service中去包装我们的接口

一套完美的代码结构，逻辑清晰，适合多个开发一个项目，从而不会显得代码网络请求紊乱，mvp的模式
2、怎么去封装网络请求

![](http://upload-images.jianshu.io/upload_images/5363507-041cef98ce3ce8ea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)创建依赖不要全部放在app去管理，一个好的工程结构，看起来给人的感觉很美

1、加入依赖
```
compile'com.squareup.retrofit2:retrofit:2.1.0'
compile'com.squareup.retrofit2:converter-gson:2.1.0'
compile'com.squareup.retrofit2:adapter-rxjava:2.1.0'
compile'com.squareup.okhttp3:logging-interceptor:3.4.1'// 日记拦截，方便调试
compile'io.reactivex:rxandroid:1.1.0'//经典的rx
```
2、创建我们自己的retrofit

![](http://upload-images.jianshu.io/upload_images/5363507-0d4484cc4862f94e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)retrofit的关键的代码

okhttpclient的使用，build出来，和我们的retorfit相关联在一起

![](http://upload-images.jianshu.io/upload_images/5363507-b2b98430027354b5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)通过这里去调用返回一个service

![](http://upload-images.jianshu.io/upload_images/5363507-9a0800a9dedcab3b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)这里必须要使用单利的模式去管理

建立baseSubscriber去继承rx安卓的订阅者
```
public  class    BaseSubscriber    extends    Subscriber
创建我们的接口
public abstract class SubscriberListener<T> {
    public abstract void onSuccess(T response);
    //why  后台定义来改动
    public abstract void onFail(String errorCode,String errorMsg);
    public abstract void onError(Throwable e);
    public void onCompleted(){  }
    public void onBegin(){}
    //如果app需要检查你的账号是否在别的设备上登录，根据返回吗在这里做登录的操作
    //每当用户在app中请求了一次网络的请求的话，就会走到这个方法里，然后在这个方法中做统一的操作
    //比如退出app，重新登录。。。
    public abstract void checkLogin(String errorCode,String erroMsg);
}
```
这个的接口是根据我们根据后台的服务器去自定义，onbegin的方法在哪里去调用
```
package network.shiming.com.network.retrofit;
/**
 * Created by shiming on 2017/4/3.
 * 订阅者
 */
public class BaseSubscriber<T extends SMResponse> extends Subscriber<T>{
    private static final String TAG="BaseSubscriber";
    private SubscriberListener mBaseSubscriber;
    public BaseSubscriber(SubscriberListener baseSubscriber){
        this.mBaseSubscriber=baseSubscriber;
    }
    public void onBegin(){
        Log.i(TAG,"onbegin");
        if (mBaseSubscriber!=null){
            mBaseSubscriber.onBegin();
        }
    }
    /**
     * 为了防止可能的内存泄露，在你的 Activity 或 Fragment 的 onDestroy 里
     * 用 Subscription.isUnsubscribed() 检查你的 Subscription 是否是
     * unsubscribed。如果调用了 Subscription.unsubscribe()
     * Unsubscribing将会对 items 停止通知给你的 Subscriber，
     * 并允许垃圾回收机制释放对象，防止任何 RxJava 造成内存泄露
     * 最终要走到我们的完成中去  要不然内存有可能泄露
     */
    @Override
    public void onCompleted() {
        Log.i(TAG,"onCompleted");
        if (mBaseSubscriber!=null){
            mBaseSubscriber.onCompleted();
        }
        if (!this.isUnsubscribed()){//如果用户已取消订阅返回true 加上！false
            //请求完成了调用这个 手动的给他取消掉订阅  防止内存泄露
            this.unsubscribe();
        }
    }
    @Override
    public void onError(Throwable e) {
        Log.i(TAG,"onError");
        if (mBaseSubscriber!=null){
            mBaseSubscriber.onError(e);//gothere  nopass down
        }
        onCompleted();
    }
    @Override
    public void onNext(T response) {
        Log.i(TAG,"onNext");
        if (mBaseSubscriber!=null){
            System.out.println(response.error_code==0);
            if (response.error_code==0){//结果码0  聚合数据的返回码看文档 o
                mBaseSubscriber.onSuccess(response);
            }
            // 这些返回码 都是需要和服务器 那边相对应的
            if (response.error_code==1){
                mBaseSubscriber.onFail(response.error_code+"",response.reason);
            }
            if (response.error_code==2){
                mBaseSubscriber.checkLogin(response.error_code+"",response.reason);
            }
        }
    }
}
```
这个方法我们封装到我们的base中，在HttpMethod 中订阅开始call方法的开始，绑定call

```
package network.shiming.com.network.retrofit;


import rx.Observable;
import rx.android.schedulers.AndroidSchedulers;
import rx.functions.Action0;
import rx.schedulers.Schedulers;

/**
 * Created by shiming on 2017/4/8.
 * RxJava最核心的两个东西是Observables（被观察者，事件源）
 * 和Subscribers（订阅者）。Observables发出一系列事件，
 * Subscribers处理这些事件
 */
public class HttpMethod {
    // 这个可以处理一个app中大多的网络请求了
    public static <T extends SMResponse> void toSubscribe(Observable<T> observable, final BaseSubscriber<T> baseSubscriber) {
        observable.subscribeOn(Schedulers.io())
                .retryWhen(new RetryWhenHandler(1, 5))//chongxin lianjie
                .doOnSubscribe(new Action0() {
                    @Override
                    public void call() {
                        baseSubscriber.onBegin();
                    }
                })
                .subscribeOn(AndroidSchedulers.mainThread())
                .unsubscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(baseSubscriber);//绑定订阅者
    }
    // 多个数据流，整合多个数据流的方法 调用  在第一版本 我先不讲这个这么用  如果有合适的接口调用
//    我会把这个讲讲怎么用
//    public static <T> Observable.Transformer<T, T> defaultHandler() {
//        return new Observable.Transformer<T, T>() {
//            @Override
//            public Observable<T> call(Observable<T> observable) {
//                return observable
//                        .subscribeOn(Schedulers.io())
//                        .retryWhen(new RetryWhenHandler(1, 5))
//                        .unsubscribeOn(Schedulers.io())
//                        .observeOn(AndroidSchedulers.mainThread());
//            }
//        };
//    }
}
```
SMRetrofit 类是我们关键类的开始，通过这个SMRetrofit 去调用
```
package network.shiming.com.network.retrofit;
/**
 * Created by shiming on 2017/4/3.
 */
public class SMRetrofit {
    private static SMRetrofit mRetrofit;
    private  Context mContext;
    //是否https的链接
    private static  boolean mIsHttps;
    private String mServerAddressFormal;
    private static final int DEFAULT_TIMEOUT = 15;
    private OkHttpClient mClient;
    private Retrofit mBuild;
    private SMRetrofit(Context context,boolean isHttps){
        //为什么要取这个context的对象，因为这个生命周期比较短，百度
        mContext = context.getApplicationContext();
        mIsHttps = isHttps;
        initRetrofit();
    }
    private void initRetrofit() {
        //no https
        mServerAddressFormal = BuildConfig.SERVER_ADDRESS_FORMAL;
        String publishEnvironment = BuildConfig.PUBLISH_ENVIRONMENT;
        if (mIsHttps){
            //is https
            mServerAddressFormal=BuildConfig.SERVER_ADDRESS_FORMAL_S;
        }
        System.out.println( " host shiming "+ mServerAddressFormal);
        initRetrofit(mServerAddressFormal)；
    }
    private void initRetrofit(String serverAddressFormal) {
        Log.i(this.getClass().getName(),"serverAddress="+serverAddressFormal);
        HttpLoggingInterceptor httpLoggingInterceptor = new HttpLoggingInterceptor();
        Log.i("shiming ",BuildConfig.DEBUG+"");
        if (!BuildConfig.DEBUG){
            System.out.println( "shiming debug log");
            httpLoggingInterceptor.setLevel(HttpLoggingInterceptor.Level.BODY);
        }else {
            //上线  不给日记输出
            System.out.println( "shiming "+" no log");
            httpLoggingInterceptor.setLevel(HttpLoggingInterceptor.Level.NONE);
        }
        File cache = new File(mContext.getCacheDir(), "cache");
        int cacheSize=10*1024*1024;
        Cache cache1 = new Cache(cache, cacheSize);
        //这里我还得说明一下 这个拦截器的原因  你比如说一个ip地址 ，用app端去访问和 pc端访问回来的数据不一样
        //这就是拦截器的作用  比如在app的版本不同 对一个地址访问回来的数据会不一样
        mClient = new OkHttpClient.Builder().cache(cache1)
//              .addInterceptor(new Interceptor(mContext))//拦截器 如果项目需要的话 如果大公司的话 该死的项目经理会要求拿到用户的手机设备的信息发布渠道的东西 ，需要这里填入
                .connectTimeout(DEFAULT_TIMEOUT, TimeUnit.SECONDS)
                .readTimeout(DEFAULT_TIMEOUT, TimeUnit.SECONDS)
                .writeTimeout(DEFAULT_TIMEOUT, TimeUnit.SECONDS)
                .addInterceptor(httpLoggingInterceptor)
                .cookieJar(new PersistentCookieJar(new SetCookieCache(), new SharedPrefsCookiePersistor(mContext)))
                .build();
        mBuild = new Retrofit.Builder()
                .baseUrl(serverAddressFormal)
                .addConverterFactory(GsonConverterFactory.create())
                .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
                .client(mClient)// 关联
                .build();
    }
    /**
     *
     * @param context
     * @return
     * 双重锁判断 单利
     * 由于我自己用的聚合的数据接口 我这里就默认了我这里不是https的接口
     *
     */
    public static SMRetrofit getInstance(Context context){
        if (mRetrofit==null||mIsHttps){
            synchronized (SMRetrofit.class){
                if (mRetrofit==null||mIsHttps){
                    mRetrofit=new SMRetrofit(context,false);
                }
            }
        }
        return mRetrofit;
    }
    // 如果项目中有用到的https的地方  使用这个初始化 并且在build文件去配置的你地址
    public static SMRetrofit getInstance(Context context,boolean isHttps){
        if (mRetrofit==null||isHttps){
            synchronized (SMRetrofit.class){
                if (mRetrofit==null||isHttps){
                    mRetrofit=new SMRetrofit(context,isHttps);
                }
            }
        }
        return mRetrofit;
    }
    /**
     *
     * @return 真正的retorfit在这里
     */
    public Retrofit getBuild(){
        return mBuild;
    }
    /**
     *
     * @param context
     * @param servcie 真正的service在这里
     * @param <T>
     * @return
     */
    public static <T> T getService(Context context,Class<T> servcie){
        return SMRetrofit.getInstance(context).getBuild().create(servcie);
    }
`
```
调用的方法model中去调用 
```
/**
 * Created by shiming on 2017/4/3.
 *
 * import rx.Observable; 注意这里的导入的是哪个包下的  不要导入了其他的安卓报下的observable了  就行了
 */
public class TestModel {
    private  TestService mService;
    public TestModel(Context context){
        mService = SMRetrofit.getService(context, TestService.class);
    }
    public void getTaday(String cai,BaseSubscriber infos){
        Observable<SMResponse<ArrayList<TadayBean>>> data = mService.getTaday("b15674dbd34ec00ded57b369dfdabd90", "1.0", 4, 4);
        HttpMethod.toSubscribe(data,infos);
    }
}
```
关键的数据基类
```
/**
 * Created by shiming on 2017/4/3.
 * 保证一个类的唯一     子类自需要重写一个 uniquekey的方法 ，就可以了
 * 为什么呢 ？ 有时候  我们传入是对象  ，我们只需要一个对象的值
 * 但是要保证我们的对象是同一个的对象  so  你懂得
 */
public abstract class BaseOneBean implements Serializable{
    @Override
    public int hashCode() {
        String[] uniqueKey = uniqueKey();
        if (uniqueKey != null && uniqueKey.length > 0) {
            int result = 0;
            for (int i = 0; i < uniqueKey.length; i++) {
                String unique = uniqueKey[i];
                if (!TextUtils.isEmpty(unique)) {
                    result = 31 * result + unique.hashCode();
                }
            }
            if (result == 0) {
                return super.hashCode();
            } else {
                return result;
            }
        } else {
            return super.hashCode();
        }
    }
    public abstract String[] uniqueKey() ;
    @Override
    public boolean equals(Object o) {
        String[] uniqueKey = uniqueKey();
        if (uniqueKey != null && uniqueKey.length > 0) {
            BaseOneBean user = (BaseOneBean) o;
            String[] modelKey = user.uniqueKey();
            for (int i = 0; i < uniqueKey.length; i++) {
                String unique = uniqueKey[i];
                if (TextUtils.isEmpty(unique) || !unique.equals(modelKey[i])) {
                    return super.equals(o);
                }
            }
            return true;
        } else {
            return super.equals(o);
        }
    }
}
```
这个类的作用，仅仅是让我们的数据bean保持唯一，在做业务比如去刷新我们的adapter的时候，我们传入的数据bean要是唯一，就必须去继承它，但是我们项目中最好去做个数据bean的管理
第二个重要的类
```

public class SMResponse<T> extends BaseOneBean {
    public T result; // TODO: 2017/4/8   这里哈哈  这里妈的最坑  这里返回的状态码 写错了，我们就拿不到数据
    public String reason;
    public int error_code;
    @Override
    public String[] uniqueKey() {
        return new String[]{""};
    }
}
```
这个类的返回的状态的数据一定要和服务器的返回的bean正确，要不然你拿到额数据一定会为空，我一时手快把result写成了reuslt，导致我找了半天问题出现在哪里，巨难受，想哭!
##最后项目已经在git开源，欢迎star 哈哈
https://github.com/Shimingli/Demo#demo