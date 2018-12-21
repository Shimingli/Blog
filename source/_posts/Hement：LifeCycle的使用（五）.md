---
title: Hement：LifeCycle的使用（五）
date: 2018-12-21 10:10:24
tags: LifeCycle
---
 *  **GitHub地址**：[Hement](https://github.com/Shimingli/Hement):持续更新中

* **apk下载地址**
![image.png](https://upload-images.jianshu.io/upload_images/5363507-e8d792a95259eb48.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<!--  more  --> 
* **app详情**
![app详情.png](https://upload-images.jianshu.io/upload_images/5363507-ce522a1167b68de7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* **欢迎关注我的公众号**
![公众号](https://upload-images.jianshu.io/upload_images/5363507-0a0cf2e5fd8f843d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* **LifeCycle**
* 在17年的谷歌IO大会上，Google官方向我们推出了 [Android Architecture Components](https://developer.android.com/topic/libraries/architecture/index.html),其中谈到Android组件处理生命周期的问题，向我们介绍了 [Handling Lifecycles](https://developer.android.com/topic/libraries/architecture/lifecycle.html)。
* 由于和小组成员讨论过，感觉使用时机还没有到，而且那个时候我反编译微信的apk，发现也没有使用的到，所以这个框架的学习计划就不断地搁浅，前几天又反编译了一下微信的安卓apk，发现它使用了来减少代码的臃肿！
推荐一个**[反编译工具](https://github.com/Shimingli/jadx)**

![微信反编译的结果图.png](https://upload-images.jianshu.io/upload_images/5363507-6ba74ad7ebbcbde6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 所以就有必要了解它到底是个什么东西

* **为什么要使用Lifecycle**
* 我们在处理Activity或者Fragment组件的生命周期相关时，不可避免会遇到这样的问题：
我们在Activity的onCreate()中初始化某些成员（比如MVP架构中的Presenter，或者AudioManager、MediaPlayer等），然后在onStop中对这些成员进行对应处理，在onDestroy中释放这些资源，这样导致我们的代码也许会像这样：

```

 class MyPresenter{
    public MyPresenter() {
    }

    void create() {
        //do something
    }

    void destroy() {
        //do something
    }
}

class MyActivity extends AppCompatActivity {
    private MyPresenter presenter;

    public void onCreate(...) {
        presenter= new MyPresenter ();
        presenter.create();
    }

    public void onDestroy() {
        super.onDestroy();
        presenter.destory();
    }
}
 ```

* 代码没有问题，关键问题是，实际生产环境中 ，这样的代码会非常复杂，你最终会有太多的类似调用并且会导致 onCreate() 和 onDestroy() 方法变的非常臃肿。


* **如何使用Lifecycle**
在前面我写过一篇文章[Hement：MVP架构中的网络框架(RxJava2+Retrofit2+RxAndroid)（二）](https://www.jianshu.com/p/e46cace343a4)这是没有使用的Lifecycle的网络请求的Presenter必须在onDestroy时候去销毁view！
```
    @Override
    protected void onDestroy() {
        super.onDestroy();
        mMainPresenter.detachView();
    }
```
* 能不能集成到`BasePresenter`的中去呢？其实是可以滴,只需要把`BasePresenter`实现`LifecycleObserver`即可
```
public class NewBasePresenter<V extends IMvpView> implements LifecycleObserver {


    public static final String TAG = "NewBasePresenter";
    private V mMvpView;


    public void attachView(V mvpView) {
        mMvpView = mvpView;
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_DESTROY)
    public void onDestroy() {
        Timber.tag(TAG).d("onDestroy lifecycle event.start ");
        detachView();
    }

    /**
     * 由于在每次这个方法都会去执行，所以可以这样进行
     * @param owner
     * @param event
     */
    @OnLifecycleEvent(Lifecycle.Event.ON_ANY)
    public void onLifecycleChanged(@NotNull LifecycleOwner owner, @NotNull Lifecycle.Event event) {

    }
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

    /**
     * 获取View
     * @return
     */
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
* 具体的Preseter的实现
```
public class NewNetWorkPresenter extends NewBasePresenter<NetWorkView> {


    private Disposable mDisposable;
    private final IRemoteServer mIRemoteServer;

    public NewNetWorkPresenter() {
        mIRemoteServer = IRemoteServer.Creator.newHementService();
    }

    @Override
    public void attachView(NetWorkView mvpView) {
        super.attachView(mvpView);
    }

    @Override
    public void detachView() {
        super.detachView();
        if (mDisposable != null) mDisposable.dispose();
        Timber.tag(TAG).d("onDestroy lifecycle event. end ");
    }
    public void loadData(String key,String day){
        //检查View是否附着在上面，不在直接抛出异常
        checkViewAttached();
        //检查是否往下运行
        dispose(mDisposable);
        loadData(key,day,new BaseObserver<SMResponse<ArrayList<TodayBean>>>(new SubscriberListener<SMResponse<ArrayList<TodayBean>>>() {

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
    public void loadData(String key, String day, BaseObserver observer) {
        Observable<SMResponse<ArrayList<TodayBean>>> today = mIRemoteServer.getToday(key, day);
        today.observeOn(AndroidSchedulers.mainThread())
                .subscribeOn(Schedulers.io())
                .subscribe(observer);
    }
}
```

* 在Activity中的使用,只需要使用这样使用，我们就能够检测到生命周期的使用，有兴趣的可以下载Apk查看日志！

```
    private NewNetWorkPresenter mNewNetWorkPresenter;
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_new_net_work);
        mNewNetWorkPresenter = new NewNetWorkPresenter();
        getLifecycle().addObserver(mNewNetWorkPresenter);
   
    }
```

* **Lifecycle的升级使用**
*  假如我们有个这样的需求，某个页面需要去查询数据库展示数据，同时使用到`RxJava2`的依赖,查询数据的过程在子线程处理数据回调到UI线程，当突然有一天产品经理说这个页面的数据我们要从网络获取，那么你也不确定那到底是把代码删除了还是备份，这些你都不知道！所以这时候你就需要独立绑定` getLifecycle().addObserver(mSyncLifecycleObserver);`Activity只关心结果即可！代码如下
*  在这里需要使用这个依赖
```
  api  "com.jakewharton.rxrelay2:rxrelay:2.0.0"
```
![image.png](https://upload-images.jianshu.io/upload_images/5363507-c8cdd3c913162aa1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* SyncResponse 
```
public class SyncResponse {
    public final SyncResponseEventType eventType;
    public final Events comment;

    public SyncResponse(SyncResponseEventType eventType, Events comment) {
        this.eventType = eventType;
        this.comment = comment;
    }
}
```

 * SyncLifecycleObserver 
```

package com.shiming.hement.lifecycle;

import android.arch.lifecycle.Lifecycle;
import android.arch.lifecycle.LifecycleObserver;
import android.arch.lifecycle.OnLifecycleEvent;
import android.widget.Toast;

import com.shiming.hement.utils.Events;

import org.w3c.dom.Comment;

import io.reactivex.Completable;
import io.reactivex.android.schedulers.AndroidSchedulers;
import io.reactivex.disposables.CompositeDisposable;
import io.reactivex.functions.Action;
import io.reactivex.functions.Consumer;
import io.reactivex.schedulers.Schedulers;
import timber.log.Timber;

/**
 * <p>
 *
 * </p>
 *
 * @author shiming
 * @version v1.0
 * @since 2018/12/18 10:44
 */

public class SyncLifecycleObserver implements LifecycleObserver {

    private static final String TAG = "SyncLifecycleObserver";

    private final CompositeDisposable disposables = new CompositeDisposable();


    @OnLifecycleEvent(Lifecycle.Event.ON_CREATE)
    public void onCreate() {
        Timber.tag(TAG).d("onCreate lifecycle event.");
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_START)
    public void onStart() {
        Timber.tag(TAG).d("onStart lifecycle event.");
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
    public void onResume() {
        Timber.tag(TAG).d("onResume lifecycle event.");
        disposables.add(SyncRxBus.getInstance().toObservable()
                .subscribe(new Consumer<SyncResponse>() {
                    @Override
                    public void accept(SyncResponse syncResponse) throws Exception {
                        handleSyncResponse(syncResponse);
                    }
                }, new Consumer<Throwable>() {
                    @Override
                    public void accept(Throwable throwable) throws Exception {
                        Timber.tag(TAG).d("onResume lifecycle event.  Throwable=  %s",throwable);
                    }
                }));
    }

    private void handleSyncResponse(SyncResponse response) {
        if (response.eventType == SyncResponseEventType.SUCCESS) {
            onSyncSuccess(response.comment);
        } else {
            onSyncFailed(response.comment);
        }
    }

    private void onSyncSuccess(Events comment) {
        Timber.tag(TAG).d("received sync comment success event for comment %s", comment);
        disposables.add(Completable.fromAction(new Action() {
            @Override
            public void run() throws Exception {
                doWhat();
                Thread thread = Thread.currentThread();
                Timber.tag(TAG).i(thread.toString());
            }
        })
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Action() {
                               @Override
                               public void run() throws Exception {
                                   Timber.tag(TAG).d("  success");
                               }
                           },
                        new Consumer<Throwable>() {
                            @Override
                            public void accept(Throwable throwable) throws Exception {
                                Timber.tag(TAG).e(throwable, "  error");
                            }
                        }));
    }

    /**
     * 可以做些子线程的数据操作
     */
    private void doWhat() {

    }
    private void onSyncFailed(Events comment) {
        Timber.tag(TAG).d("received sync comment failed event for comment %s", comment);
        disposables.add(Completable.fromAction(new Action() {
            @Override
            public void run() throws Exception {
                doWhat();
            }
        })
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Action() {
                               @Override
                               public void run() throws Exception {
                                   Timber.tag(TAG).d("  success");
                               }
                           },
                        new Consumer<Throwable>() {
                            @Override
                            public void accept(Throwable throwable) throws Exception {
                                Timber.tag(TAG).e(throwable, "  error");
                            }
                        }));
    }

    /**
     * 12-19 15:35:52.317 29721-29721/com.shiming.hement D/SyncLifecycleObserver: received sync comment success event for comment Events=我是NewRxBusDemoActivity中发出成功的事件
     * 12-19 15:35:52.318 29721-29834/com.shiming.hement I/SyncLifecycleObserver: Thread[RxCachedThreadScheduler-2,5,main]
     * 12-19 15:35:52.318 29721-29721/com.shiming.hement D/SyncLifecycleObserver: received sync comment success event for comment Events=我是NewRxBusDemoActivity中发出成功的事件
     * 12-19 15:35:52.319 29721-29835/com.shiming.hement I/SyncLifecycleObserver: Thread[RxCachedThreadScheduler-3,5,main]
     * 12-19 15:35:52.320 29721-29721/com.shiming.hement D/SyncLifecycleObserver: received sync comment success event for comment Events=我是NewRxBusDemoActivity中发出成功的事件
     * 12-19 15:35:52.321 29721-29834/com.shiming.hement I/SyncLifecycleObserver: Thread[RxCachedThreadScheduler-2,5,main]
     * 12-19 15:35:52.323 29721-29721/com.shiming.hement D/SyncLifecycleObserver:   success
     * 12-19 15:35:52.330 29721-29721/com.shiming.hement D/SyncLifecycleObserver:   success
     */
    @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
    public void onPause() {
        Timber.tag(TAG).d("onPause lifecycle event.");
        // 如果不在这里
        disposables.clear();
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_STOP)
    public void onStop() {
        Timber.tag(TAG).d("onStop lifecycle event.");
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_DESTROY)
    public void onDestroy() {
        Timber.tag(TAG).d("onDestroy lifecycle event.");
    }

    /**
     * 所有的事件都会输出
     */
    @OnLifecycleEvent(Lifecycle.Event.ON_ANY)
    public void onAny() {
        Timber.tag(TAG).d("onAny 所有的事件都会输出");
    }
}

```
* SyncResponseEventType 
```

public enum SyncResponseEventType {
    SUCCESS,
    FAILED
}

```

* SyncRxBus 
```

public class SyncRxBus {

    private static SyncRxBus instance;
    private final PublishRelay<SyncResponse> relay;

    public static synchronized SyncRxBus getInstance() {
        if (instance == null) {
            instance = new SyncRxBus();
        }
        return instance;
    }

    private SyncRxBus() {
        relay = PublishRelay.create();
    }

    public void post(SyncResponseEventType eventType, Events comment) {
        relay.accept(new SyncResponse(eventType, comment));
    }

    public Observable<SyncResponse> toObservable() {
        return relay;
    }
}

```
* 在Activity中的使用
```
   // 如果使用的多的话，可以放在BaseActivity中
    private SyncLifecycleObserver mSyncLifecycleObserver = new SyncLifecycleObserver();


    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_rx_bus_layout);
        getLifecycle().addObserver(mSyncLifecycleObserver);

        findViewById(R.id.btn_mock_send_s_onclick).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Events events = new Events("我是NewRxBusDemoActivity中发出成功的事件");
                SyncRxBus.getInstance().post(SyncResponseEventType.SUCCESS, events);
            }
        });
        findViewById(R.id.btn_mock_send_f_onclick).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Events events = new Events("我是NewRxBusDemoActivity中发出失败的事件");
                SyncRxBus.getInstance().post(SyncResponseEventType.FAILED, events);
            }
        });
```
* 如果后续我们不需要使用的话，直接就把Activity中两行代码注释掉就行~~~ 

 *  **GitHub地址**：[Hement](https://github.com/Shimingli/Hement):持续更新中

* **感谢以下博客对我的帮助**
  * [Android官方架构组件:Lifecycle详解&原理分析](https://blog.csdn.net/mq2553299/article/details/79029657)
     * [Android官方架构组件介绍之LifeCycle](https://www.jianshu.com/p/acf8c55533f1)
