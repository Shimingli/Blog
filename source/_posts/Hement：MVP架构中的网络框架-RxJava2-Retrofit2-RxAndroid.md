---
title: Hement：MVP架构中的网络框架(RxJava2+Retrofit2+RxAndroid)
date: 2018-12-06 10:11:46
tags: [RxJava2,Retrofit2,RxAndroid]
---

* 如何设计好网络框架，我心中也没答案，看过很多别人的轮子，有些太简单，有限过于复杂，有些为了满足去请求网络，褒贬不一
* 本文的网络框架基于我三年前的网络框架的升级版,具体请看这里 [MVP网络框架(Retorfit+Rxjava+Rxandroid)](https://www.jianshu.com/p/141ee58eb143)

#### **加入依赖** 
```
    api "com.squareup.retrofit2:retrofit:2.5.0"
    api "com.squareup.retrofit2:converter-gson:2.5.0"
    api "com.squareup.retrofit2:adapter-rxjava2:2.5.0"
    api 'io.reactivex.rxjava2:rxandroid:2.1.0'
    api 'io.reactivex.rxjava2:rxjava:2.2.2'
    api "com.squareup.okhttp3:logging-interceptor:3.12.0"
```
<!--  more  --> 
* `rxjava`:是反应式扩展的`Java VM`实现：一个用于使用可观察序列组成异步和基于事件的程序的库。
* `rxandroid`:这个模块向`RxJava`添加了最少的类，使得在`Android`应用程序中编写反应性组件变得简单且无麻烦。更具体地说，它提供了在主线程或在任何`Looper`上调度的调度器。
* `converter-gson`:通过`GsonConverterFactory`为`Retrofit`添加`Gson`支持
* `logging-interceptor`:可以很直观的观察到网络请求的日记
      ![image.png](https://upload-images.jianshu.io/upload_images/5363507-46540b7a41e10c5a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
   ![image.png](https://upload-images.jianshu.io/upload_images/5363507-eeeae085f77ab263.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* `adapter-rxjava2`:一个用于适应`RXJava 2 `类型的适配器。

####  **搭建网络框架** 
 *  IMvpView
  ```
  public interface IMvpView {

    /**
     * 数据失败的同一处理
     * @param errorCode
     * @param errorMsg
     */
    void getDataFail(String errorCode, String errorMsg);

    /**
     * 发成异常的同一处理
     * @param e
     */
    void onError(Throwable e);
  }
  ```
* IPresenter  处理和View的生命周期
```

public interface IPresenter<V extends IMvpView> {

    /**
     * 和view结合
     * @param mvpView
     */
    void attachView(V mvpView);

    /**
     * 和view 断开
     */
    void detachView();
}
```
* `BasePresenter ` 所有`Presenter`的基类,同时绑定所有`Activity`的生命周期
```
public class BasePresenter<V extends IMvpView> implements IPresenter<V> {


    private V mMvpView;

    @Override
    public void attachView(V mvpView) {
        mMvpView = mvpView;
    }

    @Override
    public void detachView() {
        mMvpView = null;
    }

    /**
     * 判断是否还在连接在一起的
     * @return
     */
    public boolean isViewAttached() {
        return mMvpView != null;
    }

    public V getMvpView() {
        return mMvpView;
    }

    /**
     * 检查View是否附着
     */
    public void checkViewAttached() {
        if (!isViewAttached()) throw new MvpViewNotAttachedException();
    }

    public static class MvpViewNotAttachedException extends RuntimeException {
        public MvpViewNotAttachedException() {
            super("在绑定数据之前一定要绑定视图");
        }
    }

    /**
     * 这是Observer 中的 onServer ,当我们调用这个方法，直接就不会走到 onNext中去
     * @param disposable
     */
    public  void dispose(Disposable disposable) {
        if (disposable != null && !disposable.isDisposed()) {
            disposable.dispose();
        }
    }
}
```
* 总体结构图
![总体结构图](https://upload-images.jianshu.io/upload_images/5363507-6f1a621fa6cf605b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
####  **网络框架DataManager**
* 根据上篇文章的设计思路 [Hement：MVP架构设计（一）](https://www.jianshu.com/p/157f781c164c)
![Android中的MVP模式.jpg](https://upload-images.jianshu.io/upload_images/5363507-cc2c20e2e4937c87.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 可以知道`DataManager`不仅仅是处理网络请求那么简单，同时还需要处理本地存储`SharedPreferences`和`SQLite`
* 所以就转变成了，DataManager向外界提供数据对象，数据的处理已经帮您处理好了，使用的时候只需要获取你想要的对象实例，就可以了，具体封装如下
```
@Singleton
public class DataManager {


    private final IRemoteServer mIRemoteServer;
    private final PreferencesHelper mPreferencesHelper;
    private final DatabaseHelper mDatabaseHelper;

    @Inject
    public DataManager(IRemoteServer server, PreferencesHelper preferencesHelper, DatabaseHelper databaseHelper) {
        mIRemoteServer = server;
        mPreferencesHelper = preferencesHelper;
        mDatabaseHelper = databaseHelper;
    }

    public void loadData(String key, String day, BaseObserver observer) {
        Observable<SMResponse<ArrayList<TodayBean>>> today = mIRemoteServer.getToday(key, day);
        today.observeOn(AndroidSchedulers.mainThread())
                .subscribeOn(Schedulers.io())
                .subscribe(observer);
    }

    public PreferencesHelper getPreferencesHelper() {
        return mPreferencesHelper;
    }

    /**
     * 从网络获取数据，缓存到数据库，然后从数据库中取数据
     * @return
     */
    public Observable<TodayBean> syncDBBean() {
        return mIRemoteServer.getToday("b15674dbd34ec00ded57b369dfdabd90", "1/1").concatMap(new Function<SMResponse<ArrayList<TodayBean>>, ObservableSource<? extends TodayBean>>() {
            @Override
            public ObservableSource<? extends TodayBean> apply(@NonNull SMResponse<ArrayList<TodayBean>> response)
                    throws Exception {
                return mDatabaseHelper.setDBData(response.result);
            }
        });
    }

    /**
     * 从数据库中取数据
     * @return
     */
    public Observable<List<TodayBean>> getDBBean() {
        return mDatabaseHelper.getDBData().distinct();
    }
}

```

 

####  **网络框架如何使用** 
* 这里使用的是聚合数据的接口 ,在这里感谢无私的贡献，三年了这个接口还能用，万分感谢
```
http://v.juhe.cn/todayOnhistory/queryEvent.php?key=b15674dbd34ec00ded57b369dfdabd90&date=1/1
```
* 1、定义远程Server:`IRemoteServer`,在这里我把以前的单独生成的`Retrofit`的`Server`抽取到`IRemoteServer`,因为项目使用了`Dagger2`，后面介绍 
```
public interface IRemoteServer {
    /**
     * http://v.juhe.cn/todayOnhistory/queryEvent.php?key=b15674dbd34ec00ded57b369dfdabd90&date=1/1
     * @param key 申请的key
     * @param date 日期
     * @return 返回历史上的今天
     */
    @GET("queryEvent.php")
    Observable<SMResponse<ArrayList<TodayBean>>> getToday(@Query("key") String key, @Query("date") String date);

    class Creator {
        public static IRemoteServer newHementService() {
          return  SMRetrofit.getService(HementApplication.getContext(), IRemoteServer.class);
        }
    }
}
```
* 2、`DataManager`中的线程调度,结果回调到`AndroidSchedulers.mainThread()`线程，就是安卓的`UI`线程。
```
   private final IRemoteServer mIRemoteServer;
    private final PreferencesHelper mPreferencesHelper;
    private final DatabaseHelper mDatabaseHelper;

    @Inject
    public DataManager(IRemoteServer server, PreferencesHelper preferencesHelper, DatabaseHelper databaseHelper) {
        mIRemoteServer = server;
        mPreferencesHelper = preferencesHelper;
        mDatabaseHelper = databaseHelper;
    }

    public void loadData(String key, String day, BaseObserver observer) {
        Observable<SMResponse<ArrayList<TodayBean>>> today = mIRemoteServer.getToday(key, day);
        today.observeOn(AndroidSchedulers.mainThread())
                .subscribeOn(Schedulers.io())
                .subscribe(observer);
    }
```
* 3、`NetWorkPresenter`定义
```
@ConfigPersistent
public class NetWorkPresenter  extends BasePresenter<NetWorkView> {

    private final DataManager mDataManager;

    private Disposable mDisposable;

    @Inject
    public NetWorkPresenter(DataManager dataManager) {
        mDataManager = dataManager;
    }

    @Override
    public void attachView(NetWorkView mvpView) {
        super.attachView(mvpView);
    }

    @Override
    public void detachView() {
        super.detachView();
        if (mDisposable != null) mDisposable.dispose();
    }
    public void loadData(String key,String day){
        //检查View是否附着在上面，不在直接抛出异常
        checkViewAttached();
        //检查是否往下运行
        dispose(mDisposable);
        mDataManager.loadData(key,day,new BaseObserver<SMResponse<ArrayList<TodayBean>>>(new SubscriberListener<SMResponse<ArrayList<TodayBean>>>() {

            @Override
            public void onSubscribe(Disposable disposable) {
                super.onSubscribe(disposable);
                mDisposable = disposable;
            }

            @Override
            public void onSuccess(SMResponse<ArrayList<TodayBean>> response) {
                getMvpView().getDataSuccess(response.result);
            }

            @Override
            public void onFail(String errorCode, String errorMsg) {
                getMvpView().getDataFail(errorCode,errorMsg);
            }

            @Override
            public void onError(Throwable e) {
                getMvpView().onError(e);
            }
        }));
    }
}

```
* 4、在`Activity`中的使用,通过Dagger2依赖注入`NetWorkPresenter`,同时在 `onDestroy()`中，销毁`RxJava2`正在调度的任务,个人理解的就是，关闭Activity，网络请求也有可能是在执行中，所以需要销毁。

```
public class NetWorkActivity extends BaseActivity implements NetWorkView, View.OnClickListener {

    String key="b15674dbd34ec00ded57b369dfdabd90";
    @Inject
    NetWorkPresenter mMainPresenter;

    private Button mBtn;
    private EditText mDay;
    private EditText mMonth;
    private RecyclerView mRecyclerView;
    private SMAdapter mSmAdapter;
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        activityComponent().inject(this);
        setContentView(R.layout.activity_net_work);
        Timber.tag(getClassName()).i("mMainPresenter   =%s",mMainPresenter);
        mMainPresenter.attachView(this);
        initView();
        initListener();
    }
    private void initView() {
        mBtn = (Button) findViewById(R.id.btn);
        mMonth = (EditText) findViewById(R.id.et_month);
        mDay = (EditText) findViewById(R.id.et_day);
        mRecyclerView = (RecyclerView) findViewById(R.id.recyclerview);
    }
    private void initListener() {
        mBtn.setOnClickListener(this);
        mSmAdapter = new SMAdapter(this, null);
        mRecyclerView.setAdapter(mSmAdapter);
        mRecyclerView.setLayoutManager(new LinearLayoutManager(this));
    }

    @Override
    public void getDataFail(String errorCode, String errorMsg) {
        Toast.makeText(this,errorMsg+errorCode,Toast.LENGTH_LONG).show();
    }

    @Override
    public void onError(Throwable e) {
        Toast.makeText(this,e.toString(),Toast.LENGTH_LONG).show();
    }

    @Override
    public void getDataSuccess(ArrayList<TodayBean> result) {
        String s = new Gson().toJson(result);
        Timber.tag(getClassName()).i(s);
        Thread thread = Thread.currentThread();
        Timber.tag(getClassName()).i(thread.toString());
        mSmAdapter.addData(result);
    }
    @Override
    protected void onDestroy() {
        super.onDestroy();
        mMainPresenter.detachView();
    }

    @Override
    public void onClick(View v) {
        if (TextUtils.isEmpty(mMonth.getText())||TextUtils.isEmpty(mDay.getText())){
            Toast.makeText(NetWorkActivity.this,"不能为空",Toast.LENGTH_SHORT).show();
        }else {
            mMainPresenter.loadData(key,mMonth.getText()+"/"+mDay.getText());
        }
    }
}
```

* 未完待续 [下一篇文章]()

 *  **GitHub地址**：[Hement](https://github.com/Shimingli/Hement):持续更新中



