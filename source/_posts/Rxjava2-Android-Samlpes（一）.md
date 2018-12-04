---
title: Rxjava2-Android-Samlpes（一）
date: 2018-11-30 09:20:05
tags:
---
* 最近工作不那么忙，计划重新搭建一个MVP架构！基于`Rxjava2-Rxandroid`首先掌握RxJava2的使用的方式！
* 此文章是根据老外[amitshekhariitbhu](https://github.com/amitshekhariitbhu)的`RxJava2-Android-Samples`Demo 改装而成，是个翻译版本，足够应付对RxJava2的全部姿势！
<!--  more  -->


* GitHub地址： [RxJava2-Android-Samples](https://github.com/Shimingli/RxJava2-Android-Samples)
  * `Map`  - >通过对每个项应用函数来变换Observable发出的项
  * `Zip`  - >通过指定的函数将多个Observable的排放组合在一起，并根据此函数的结果为每个组合发出单个项目
   * `Filter`  - >仅发出通过谓词测试的Observable中的那些项
  * `FlatMap`  - >将Observable发出的项目转换为Observables，然后将这些项目的排放量变为单个Observable
  * `Take`  - >仅发出Observable发出的前n项
  * `Reduce`  - >按顺序将一个函数应用于Observable发出的每个项目，并发出最终值
  * `Skip`  - >抑制Observable发出的前n项
  * `Buffer`  - >定期将Observable发出的项目收集到bundle中并发出这些bundle而不是一次发送一个项目
  * `Concat`  - >从两个或多个Observable发射发射而不交错
  * `Replay`  - >确保所有观察者看到相同的发射物品序列，即使他们在Observable开始发射物品后订阅
  * `Merge`  - >通过合并它们的排放将多个Observable组合成一个
  * `SwitchMap`  - >将Observable发出的项目变换为Observables，并镜像最近转换的Observable发出的项目

#### Operators

*  1、 简单的一个顺序执行的Demo
```
 /*
     * 一个一个地发出两个值的简单例子
     */
    private void doSomeWork() {
        getObservable()
                // 在后台线程上运行
                .subscribeOn(Schedulers.io())
                // 在主线程上被通知
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(getObserver());
    }

    private Observable<String> getObservable() {
        return Observable.just("1", "2","3","4","5","6");
    }
```

* d.dispose();  todo 如果这个方法放开的话，就不会往下面走了
```
 private Observer<String> getObserver() {
        return new Observer<String>() {
            /**
             *
             *为观察者提供取消（处理）的方法。
             *连接（通道）和可观察到的两个
             *同步（从{{Link Lang-OnNeXT（object）}）和异步方式。
             */
            @Override
            public void onSubscribe(Disposable d) {
                Log.d(TAG, " onSubscribe : " + d.isDisposed());
                //处理资源，操作应该是幂等的。
                //d.dispose();  todo 如果这个方法放开的话，就不会往下面走了

            }

            @Override
            public void onNext(String value) {
                Log.d(TAG, " onNext : value : " + value);
            }

            @Override
            public void onError(Throwable e) {
                Log.d(TAG, " onError : " + e.getMessage());
            }

            @Override
            public void onComplete() {
                Log.d(TAG, " onComplete");
            }
        };
    }
```
* 输出结果
```
11-30 10:03:17.883 16586-16586/com.rxjava2.android.samples D/SimpleExampleActivity:  onSubscribe : false
11-30 10:03:17.923 16586-16586/com.rxjava2.android.samples D/SimpleExampleActivity:  onNext : value : 1
11-30 10:03:17.931 16586-16586/com.rxjava2.android.samples D/SimpleExampleActivity:  onNext : value : 2
11-30 10:03:17.938 16586-16586/com.rxjava2.android.samples D/SimpleExampleActivity:  onNext : value : 3
11-30 10:03:17.944 16586-16586/com.rxjava2.android.samples D/SimpleExampleActivity:  onNext : value : 4
11-30 10:03:17.950 16586-16586/com.rxjava2.android.samples D/SimpleExampleActivity:  onNext : value : 5
11-30 10:03:17.955 16586-16586/com.rxjava2.android.samples D/SimpleExampleActivity:  onNext : value : 6
11-30 10:03:17.961 16586-16586/com.rxjava2.android.samples D/SimpleExampleActivity:  onComplete
```



* 2、通过`map`运算符 处理网咯请求的`Demo`,就比如说我去网络上请求个`ApiUser`回来，然后转化成我想要的`User`类

```
  private void doSomeWork() {
        getObservable()
                // Run on a background thread
                .subscribeOn(Schedulers.io())
                // Be notified on the main thread
                .observeOn(AndroidSchedulers.mainThread())
                .map(new Function<List<ApiUser>, List<User>>() {
                     //通过前面的东西，如何获取后面的东西
                    @Override
                    public List<User> apply(List<ApiUser> apiUsers) {
                        return Utils.convertApiUserListToUserList(apiUsers);
                    }
                })
                .subscribe(getObserver());
    }

    private Observable<List<ApiUser>> getObservable() {
        return Observable.create(new ObservableOnSubscribe<List<ApiUser>>() {
            @Override
            public void subscribe(ObservableEmitter<List<ApiUser>> e) {
                if (!e.isDisposed()) {
                    // List<ApiUser> 得到这个 对象
                    e.onNext(Utils.getApiUserList());
                    e.onComplete();
                }
            }
        });
    }

```
* 处理得到的` List<User>`
```

  //处理得到的 List<User>
    private Observer<List<User>> getObserver() {
        return new Observer<List<User>>() {

            @Override
            public void onSubscribe(Disposable d) {
                Log.d(TAG, " onSubscribe : " + d.isDisposed());
            }

            @Override
            public void onNext(List<User> userList) {
                Log.d(TAG, " onNext : " + userList.size());
            }

            @Override
            public void onError(Throwable e) {
                Log.d(TAG, " onError : " + e.getMessage());
            }

            @Override
            public void onComplete() {
                Log.d(TAG, " onComplete");
            }
        };
```

* 输出结果

```
11-30 10:22:26.566 16586-16586/com.rxjava2.android.samples D/MapExampleActivity:  onSubscribe : false
11-30 10:22:26.619 16586-16586/com.rxjava2.android.samples D/MapExampleActivity:  onNext : 3
11-30 10:22:26.624 16586-16586/com.rxjava2.android.samples D/MapExampleActivity:  onComplete

```

* 3、两个复杂的操作放到子线程中去，然后在主线程中去处理  两队球迷的问题 `zip`
* 获取数据源
```
  private Observable<List<User>> getCricketFansObservable() {
        return Observable.create(new ObservableOnSubscribe<List<User>>() {
            @Override
            public void subscribe(ObservableEmitter<List<User>> e) {
                if (!e.isDisposed()) {
                    e.onNext(Utils.getUserListWhoLovesCricket());
                    e.onComplete();
                }
            }
        });
    }

    private Observable<List<User>> getFootballFansObservable() {
        return Observable.create(new ObservableOnSubscribe<List<User>>() {
            @Override
            public void subscribe(ObservableEmitter<List<User>> e) {
                if (!e.isDisposed()) {
                    e.onNext(Utils.getUserListWhoLovesFootball());
                    e.onComplete();
                }
            }
        });
    }
```
* 处理
```
  private void doSomeWork() {
        // 获取喜欢足球名单的人员   获取板球球迷的名单
        Observable.zip(getCricketFansObservable(), getFootballFansObservable(),
                // 有点kotlin的啊 第一个对应的是沙面位置的第一个，最后是期望
                new BiFunction<List<User>, List<User>, List<User>>() {
                    @Override
                    public List<User> apply(List<User> cricketFans, List<User> footballFans) {
                        return Utils.filterUserWhoLovesBoth(cricketFans, footballFans);
                    }
                })
                // Run on a background thread
                .subscribeOn(Schedulers.io())
                // Be notified on the main thread
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(getObserver());
    }
```
* 结果监听
```
  private Observer<List<User>> getObserver() {
        return new Observer<List<User>>() {

            @Override
            public void onSubscribe(Disposable d) {
                Log.d(TAG, " onSubscribe : " + d.isDisposed());
            }

            @Override
            public void onNext(List<User> userList) {
                textView.append(" onNext");
                textView.append(AppConstant.LINE_SEPARATOR);
                for (User user : userList) {
                    textView.append(" firstname : " + user.firstname);
                    textView.append(AppConstant.LINE_SEPARATOR);
                }
                Log.d(TAG, " onNext : " + userList.size());
            }

            @Override
            public void onError(Throwable e) {
                textView.append(" onError : " + e.getMessage());
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " onError : " + e.getMessage());
            }

            @Override
            public void onComplete() {
                textView.append(" onComplete");
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " onComplete");
            }
        };
```
*  输出，过程就是找出两个类的共性，然后放到主线程操作它的结果
```
11-30 10:28:16.326 16586-16586/com.rxjava2.android.samples D/ZipExampleActivity:  onSubscribe : false
11-30 10:28:16.353 16586-16586/com.rxjava2.android.samples D/ZipExampleActivity:  onNext : 1
11-30 10:28:16.358 16586-16586/com.rxjava2.android.samples D/ZipExampleActivity:  onComplete
```
* 4、对一些耗时操作的问题，可以使用使用容器 Disposable ,在活动被破坏后不要发送事件

```
    //一次性容器，可以容纳多个其他一次性物品，并提供O(1)添加和移除复杂性。
    private final CompositeDisposable disposables = new CompositeDisposable();
  @Override
    protected void onDestroy() {
        super.onDestroy();
        //在活动被破坏后不要发送事件
        disposables.clear(); // do not send event after activity has been destroyed
    }
```

```
 disposables.add(sampleObservable()
                // Run on a background thread
                .subscribeOn(Schedulers.io())
                // Be notified on the main thread
                .observeOn(AndroidSchedulers.mainThread())
                .subscribeWith(new DisposableObserver<String>() {
                    @Override
                    public void onComplete() {
                        textView.append(" onComplete");
                        textView.append(AppConstant.LINE_SEPARATOR);
                        Log.d(TAG, " onComplete");
                    }

                    @Override
                    public void onError(Throwable e) {
                        textView.append(" onError : " + e.getMessage());
                        textView.append(AppConstant.LINE_SEPARATOR);
                        Log.d(TAG, " onError : " + e.getMessage());
                    }

                    @Override
                    public void onNext(String value) {
                        textView.append(" onNext : value : " + value);
                        textView.append(AppConstant.LINE_SEPARATOR);
                        Log.d(TAG, " onNext value : " + value);
                    }
                }));

```
* sampleObservable 
```

 static Observable<String> sampleObservable() {
        return Observable.defer(new Callable<ObservableSource<? extends String>>() {
            @Override
            public ObservableSource<? extends String> call() {
                // Do some long running operation
                // 做一些长时间运行的操作
                SystemClock.sleep(2000);
                return Observable.just("one", "two", "three", "four", "five");
            }
        });
    }
```
* 输出结果
```
11-30 10:32:47.735 16586-16586/com.rxjava2.android.samples D/DisposableExampleActivity:  onNext value : one
11-30 10:32:47.748 16586-16586/com.rxjava2.android.samples D/DisposableExampleActivity:  onNext value : two
11-30 10:32:47.755 16586-16586/com.rxjava2.android.samples D/DisposableExampleActivity:  onNext value : three
11-30 10:32:47.762 16586-16586/com.rxjava2.android.samples D/DisposableExampleActivity:  onNext value : four
11-30 10:32:47.770 16586-16586/com.rxjava2.android.samples D/DisposableExampleActivity:  onNext value : five
11-30 10:32:47.775 16586-16586/com.rxjava2.android.samples D/DisposableExampleActivity:  onComplete
```
*  5、使用取算符，它只发出所需数量的值。这里只有5个中的3个
```
    private void doSomeWork() {
        getObservable()
                // Run on a background thread
                .subscribeOn(Schedulers.io())
                // Be notified on the main thread
                .observeOn(AndroidSchedulers.mainThread())
                .take(3)
                .subscribe(getObserver());
    }

    private Observable<Integer> getObservable() {
        return Observable.just(1, 2, 3, 4, 5);
    }
```
*  监听 
```
 private Observer<Integer> getObserver() {
        return new Observer<Integer>() {

            @Override
            public void onSubscribe(Disposable d) {
                Log.d(TAG, " onSubscribe : " + d.isDisposed());
            }

            @Override
            public void onNext(Integer value) {
                textView.append(" onNext : value : " + value);
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " onNext value : " + value);
            }

            @Override
            public void onError(Throwable e) {
                textView.append(" onError : " + e.getMessage());
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " onError : " + e.getMessage());
            }

            @Override
            public void onComplete() {
                textView.append(" onComplete");
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " onComplete");
            }
        };
    }
```
*  输出结果
```
11-30 10:33:43.235 16586-16586/com.rxjava2.android.samples D/TakeExampleActivity:  onSubscribe : false
11-30 10:33:43.254 16586-16586/com.rxjava2.android.samples D/TakeExampleActivity:  onNext value : 1
11-30 10:33:43.259 16586-16586/com.rxjava2.android.samples D/TakeExampleActivity:  onNext value : 2
11-30 10:33:43.265 16586-16586/com.rxjava2.android.samples D/TakeExampleActivity:  onNext value : 3
11-30 10:33:43.271 16586-16586/com.rxjava2.android.samples D/TakeExampleActivity:  onComplete

```
* 6、延迟两秒运行
```
    private void doSomeWork() {
        getObservable()
                // Run on a background thread
                .subscribeOn(Schedulers.io())
                // Be notified on the main thread
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(getObserver());
    }

    private Observable<? extends Long> getObservable() {
        return Observable.timer(2, TimeUnit.SECONDS);
    }
```
* 7、定时器 不断重复的运行  使用RxJava运行  使用间隔2秒的间隔运行任务的简单示例
```
 private final CompositeDisposable disposables = new CompositeDisposable();
 @Override
    protected void onDestroy() {
        super.onDestroy();
        disposables.clear(); // clearing it : do not emit after destroy
    }
```
* 使用间隔2秒的间隔运行任务的简单示例立即开始，`initialDelay`的开始的时间为0
```
   private void doSomeWork() {
        disposables.add(getObservable()
                // Run on a background thread
                .subscribeOn(Schedulers.io())
                // Be notified on the main thread
                .observeOn(AndroidSchedulers.mainThread())
                .subscribeWith(getObserver()));
    }

    private Observable<? extends Long> getObservable() {
        return Observable.interval(0, 2, TimeUnit.SECONDS);
    }
```
* 8、使用单观察者的简单例子
```
 /*
     * simple example using SingleObserver
     *使用单观察者的简单例子
     */
    private void doSomeWork() {
        Single.just("Amit")
                .subscribe(getSingleObserver());
    }
```
* 观察者
```
 private SingleObserver<String> getSingleObserver() {
        return new SingleObserver<String>() {
            @Override
            public void onSubscribe(Disposable d) {
                Log.d(TAG, " onSubscribe : " + d.isDisposed());
            }

            @Override
            public void onSuccess(String value) {
                textView.append(" onNext : value : " + value);
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " onNext value : " + value);
            }

            @Override
            public void onError(Throwable e) {
                textView.append(" onError : " + e.getMessage());
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " onError : " + e.getMessage());
            }
        };
    }
```
* 输出结果 ：注意这里没有onNext的方法了，这个比较特殊
```
11-30 11:11:00.612 16586-16586/com.rxjava2.android.samples D/SingleObserverExampleActivity:  onSubscribe : true
11-30 11:11:00.615 16586-16586/com.rxjava2.android.samples D/SingleObserverExampleActivity:  onNext value : Amit

```
* 9、使用完全观测器的简单示例，延迟1s才去自行
```
  Completable completable = Completable.timer(1000, TimeUnit.MILLISECONDS);

        completable
                .subscribeOn(Schedulers.io())
                // Be notified on the main thread
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(getCompletableObserver());

 private CompletableObserver getCompletableObserver() {
        return new CompletableObserver() {
            @Override
            public void onSubscribe(Disposable d) {
                Log.d(TAG, " onSubscribe : " + d.isDisposed());
            }

            @Override
            public void onComplete() {
                textView.append(" onComplete");
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " onComplete");
            }

            @Override
            public void onError(Throwable e) {
                textView.append(" onError : " + e.getMessage());
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " onError : " + e.getMessage());
            }
        };
    }
```
* 输出结果 ：延迟1s
```
11-30 11:12:41.248 16586-16586/com.rxjava2.android.samples D/CompletableObserverExampleActivity:  onSubscribe : false
11-30 11:12:42.270 16586-16586/com.rxjava2.android.samples D/CompletableObserverExampleActivity:  onComplete
```
* 10、使用流动性的简单示例，说白了 就是累加的过程 1+2+3+4+5  ====  使用Rxjava管理  这个带了一个初始值
```
    Flowable<Integer> observable = Flowable.just(100, 2, 3, 4);

        observable.reduce(50+1, new BiFunction<Integer, Integer, Integer>() {
            @Override
            public Integer apply(Integer t1, Integer t2) {
                Log.d(TAG, "  t1 : " + t1);
                Log.d(TAG, "  t2 : " + t2);
                return t1 + t2;
            }
        }).subscribe(getObserver());

 private SingleObserver<Integer> getObserver() {

        return new SingleObserver<Integer>() {
            @Override
            public void onSubscribe(Disposable d) {
                Log.d(TAG, " onSubscribe : " + d.isDisposed());
            }

            @Override
            public void onSuccess(Integer value) {
                textView.append(" onSuccess : value : " + value);
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " onSuccess : value : " + value);
            }

            @Override
            public void onError(Throwable e) {
                textView.append(" onError : " + e.getMessage());
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " onError : " + e.getMessage());
            }
        };
    }
```
*  输出结果：51+100+2+3+4=160
```
11-30 11:14:40.489 16586-16586/com.rxjava2.android.samples D/FlowableExampleActivity:  onSubscribe : false
11-30 11:14:40.490 16586-16586/com.rxjava2.android.samples D/FlowableExampleActivity:   t1 : 51
      t2 : 100
      t1 : 151
      t2 : 2
      t1 : 153
      t2 : 3
      t1 : 156
11-30 11:14:40.491 16586-16586/com.rxjava2.android.samples D/FlowableExampleActivity:   t2 : 4
11-30 11:14:40.496 16586-16586/com.rxjava2.android.samples D/FlowableExampleActivity:  onSuccess : value : 160
```
* 11、说白了 就是累加的过程 1+2+3+4+5  ====  使用Rxjava管理  这个没有带一个初始值
```
  /*
     * simple example using reduce to add all the number
     *
     * 简单的例子，用以减少所有数字的添加
     */
    private void doSomeWork() {
        getObservable()
                .reduce(new BiFunction<Integer, Integer, Integer>() {
                    @Override
                    public Integer apply(Integer t1, Integer t2) {
                        return t1 + t2;
                    }
                })
                .subscribe(getObserver());
    }

    private Observable<Integer> getObservable() {
        return Observable.just(1, 2, 3, 4);
    }

    private MaybeObserver<Integer> getObserver() {
        return new MaybeObserver<Integer>() {
            @Override
            public void onSubscribe(Disposable d) {
                Log.d(TAG, " onSubscribe : " + d.isDisposed());
            }

            @Override
            public void onSuccess(Integer value) {
                textView.append(" onSuccess : value : " + value);
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " onSuccess : value : " + value);
            }

            @Override
            public void onError(Throwable e) {
                textView.append(" onError : " + e.getMessage());
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " onError : " + e.getMessage());
            }

            @Override
            public void onComplete() {
                textView.append(" onComplete");
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " onComplete");
            }
        };
    }
```
* 输出结果
```
11-30 11:16:17.035 16586-16586/com.rxjava2.android.samples D/ReduceExampleActivity:  onSubscribe : false
11-30 11:16:17.039 16586-16586/com.rxjava2.android.samples D/ReduceExampleActivity:  onSuccess : value : 10
```
* 12 、一个buffer 缓冲取 ，我要从缓冲区中取数据，而且是跳着取数据，就要这样做
```
   /*
     * simple example using buffer operator - bundles all emitted values into a list
     *
     * 使用缓冲运算符的简单示例-将所有发出的值捆绑到列表中
     */
    private void doSomeWork() {

        Observable<List<String>> buffered = getObservable().buffer(3, 1);

        // 3 means,  从开始索引和创建列表中最多需要三个
        // 1 means, 每次跳一步
        // so the it gives the following list
        // 1 - one, two, three
        // 2 - two, three, four
        // 3 - three, four, five
        // 4 - four, five
        // 5 - five

        buffered.subscribe(getObserver());
    }

    private Observable<String> getObservable() {
        return Observable.just("one", "two", "three", "four", "five");
    }

    private Observer<List<String>> getObserver() {
        return new Observer<List<String>>() {

            @Override
            public void onSubscribe(Disposable d) {
                Log.d(TAG, " onSubscribe : " + d.isDisposed());
            }

            @Override
            public void onNext(List<String> stringList) {
                textView.append(" onNext size : " + stringList.size());
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " onNext : size :" + stringList.size());
                for (String value : stringList) {
                    textView.append(" value : " + value);
                    textView.append(AppConstant.LINE_SEPARATOR);
                    Log.d(TAG, " : value :" + value);
                }

            }

            @Override
            public void onError(Throwable e) {
                textView.append(" onError : " + e.getMessage());
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " onError : " + e.getMessage());
            }

            @Override
            public void onComplete() {
                textView.append(" onComplete");
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " onComplete");
            }
        };
    }
```
* 输出结果：以此取出3个数字，同时角标移动1.把这个buffer中的值去完成就ok
```
11-30 11:17:25.002 16586-16586/com.rxjava2.android.samples D/BufferExampleActivity:  onSubscribe : false
11-30 11:17:25.009 16586-16586/com.rxjava2.android.samples D/BufferExampleActivity:  onNext : size :3
11-30 11:17:25.012 16586-16586/com.rxjava2.android.samples D/BufferExampleActivity:  : value :one
11-30 11:17:25.014 16586-16586/com.rxjava2.android.samples D/BufferExampleActivity:  : value :two
11-30 11:17:25.016 16586-16586/com.rxjava2.android.samples D/BufferExampleActivity:  : value :three
11-30 11:17:25.018 16586-16586/com.rxjava2.android.samples D/BufferExampleActivity:  onNext : size :3
11-30 11:17:25.019 16586-16586/com.rxjava2.android.samples D/BufferExampleActivity:  : value :two
11-30 11:17:25.021 16586-16586/com.rxjava2.android.samples D/BufferExampleActivity:  : value :three
11-30 11:17:25.022 16586-16586/com.rxjava2.android.samples D/BufferExampleActivity:  : value :four
11-30 11:17:25.025 16586-16586/com.rxjava2.android.samples D/BufferExampleActivity:  onNext : size :3
11-30 11:17:25.027 16586-16586/com.rxjava2.android.samples D/BufferExampleActivity:  : value :three
11-30 11:17:25.029 16586-16586/com.rxjava2.android.samples D/BufferExampleActivity:  : value :four
11-30 11:17:25.031 16586-16586/com.rxjava2.android.samples D/BufferExampleActivity:  : value :five
11-30 11:17:25.033 16586-16586/com.rxjava2.android.samples D/BufferExampleActivity:  onNext : size :2
11-30 11:17:25.035 16586-16586/com.rxjava2.android.samples D/BufferExampleActivity:  : value :four
11-30 11:17:25.037 16586-16586/com.rxjava2.android.samples D/BufferExampleActivity:  : value :five
11-30 11:17:25.039 16586-16586/com.rxjava2.android.samples D/BufferExampleActivity:  onNext : size :1
11-30 11:17:25.040 16586-16586/com.rxjava2.android.samples D/BufferExampleActivity:  : value :five
11-30 11:17:25.041 16586-16586/com.rxjava2.android.samples D/BufferExampleActivity:  onComplete
```
* 13、 对数据中特定的信息 做处理 使用过滤器操作符只发出偶数值的简单示例
```
  /*
     * simple example by using filter operator to emit only even value
     * 使用过滤器操作符只发出偶数值的简单示例
     */
    private void doSomeWork() {
        Observable.just(1, 2, 3, 4, 5, 6)
                .filter(new Predicate<Integer>() {
                    @Override
                    public boolean test(Integer integer) {
                        return integer % 2 == 0;
                    }
                })
                .subscribe(getObserver());
    }


    private Observer<Integer> getObserver() {
        return new Observer<Integer>() {

            @Override
            public void onSubscribe(Disposable d) {
                Log.d(TAG, " onSubscribe : " + d.isDisposed());
            }

            @Override
            public void onNext(Integer value) {
                textView.append(" onNext : ");
                textView.append(AppConstant.LINE_SEPARATOR);
                textView.append(" value : " + value);
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " onNext ");
                Log.d(TAG, " value : " + value);
            }

            @Override
            public void onError(Throwable e) {
                textView.append(" onError : " + e.getMessage());
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " onError : " + e.getMessage());
            }

            @Override
            public void onComplete() {
                textView.append(" onComplete");
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " onComplete");
            }
        };
    }
```
* 输出结果，简单的意思 就是取偶数
```
11-30 11:20:24.775 16586-16586/com.rxjava2.android.samples D/FilterExampleActivity:  onSubscribe : false
11-30 11:20:24.779 16586-16586/com.rxjava2.android.samples D/FilterExampleActivity:  onNext 
11-30 11:20:24.780 16586-16586/com.rxjava2.android.samples D/FilterExampleActivity:  value : 2
11-30 11:20:24.782 16586-16586/com.rxjava2.android.samples D/FilterExampleActivity:  onNext 
     value : 4
11-30 11:20:24.786 16586-16586/com.rxjava2.android.samples D/FilterExampleActivity:  onNext 
     value : 6
11-30 11:20:24.788 16586-16586/com.rxjava2.android.samples D/FilterExampleActivity:  onComplete
```

* 14、使用跳过操作符，它不会发出前2个值。 对前面两个值不会操作
```
 /* Using skip operator, it will not emit
    * the first 2 values.
    * 使用跳过操作符，它不会发出前2个值。
    */
    private void doSomeWork() {
        getObservable()
                // Run on a background thread
                .subscribeOn(Schedulers.io())
                // Be notified on the main thread
                .observeOn(AndroidSchedulers.mainThread())
                .skip(2)
                .subscribe(getObserver());
    }

    private Observable<Integer> getObservable() {
        return Observable.just(1, 2, 3, 4, 5);
    }

    private Observer<Integer> getObserver() {
        return new Observer<Integer>() {

            @Override
            public void onSubscribe(Disposable d) {
                Log.d(TAG, " onSubscribe : " + d.isDisposed());
            }

            @Override
            public void onNext(Integer value) {
                textView.append(" onNext : value : " + value);
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " onNext value : " + value);
            }

            @Override
            public void onError(Throwable e) {
                textView.append(" onError : " + e.getMessage());
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " onError : " + e.getMessage());
            }

            @Override
            public void onComplete() {
                textView.append(" onComplete");
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " onComplete");
            }
        };
    }
```
* 输出结果：跳出前面的两个值
```
11-30 11:21:22.224 16586-16586/com.rxjava2.android.samples D/SkipExampleActivity:  onSubscribe : false
11-30 11:21:22.235 16586-16586/com.rxjava2.android.samples D/SkipExampleActivity:  onNext value : 3
11-30 11:21:22.236 16586-16586/com.rxjava2.android.samples D/SkipExampleActivity:  onNext value : 4
     onNext value : 5
11-30 11:21:22.237 16586-16586/com.rxjava2.android.samples D/SkipExampleActivity:  onComplete
```
* 15、 使用扫描算子，它也发送先前的结果。 意思就是 我关心每次运算的结果 ，是每次运算的结果 这个有个关键的地方 subscribe 里面的 观察者 onNext的方法是先行执行的
```
  /* Using scan operator, it sends also the previous result
    * 使用扫描算子，它也发送先前的结果。
    * */
    private void doSomeWork() {
        getObservable()
                // Run on a background thread
                .subscribeOn(Schedulers.io())
                // Be notified on the main thread
                .observeOn(AndroidSchedulers.mainThread())
                .scan(new BiFunction<Integer, Integer, Integer>() {
                    @Override
                    public Integer apply(Integer int1, Integer int2) {
                        Log.d(TAG, "  int1 : " + int1);
                        Log.d(TAG, "  int2 : " + int2);
                        return int1 + int2;
                    }
                })
                .subscribe(getObserver());
    }

    private Observable<Integer> getObservable() {
        return Observable.just(1, 2, 3, 4, 5);
    }

    private Observer<Integer> getObserver() {
        return new Observer<Integer>() {

            @Override
            public void onSubscribe(Disposable d) {
                Log.d(TAG, " onSubscribe : " + d.isDisposed());
            }

            @Override
            public void onNext(Integer value) {
                textView.append(" onNext : value : " + value);
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " onNext value : " + value);
            }

            @Override
            public void onError(Throwable e) {
                textView.append(" onError : " + e.getMessage());
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " onError : " + e.getMessage());
            }

            @Override
            public void onComplete() {
                textView.append(" onComplete");
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " onComplete");
            }
        };
    }
```
* 输出结果：三角形数
```
11-30 11:22:24.396 16586-16586/com.rxjava2.android.samples D/ScanExampleActivity:  onSubscribe : false
11-30 11:22:24.409 16586-16586/com.rxjava2.android.samples D/ScanExampleActivity:  onNext value : 1
      int1 : 1
      int2 : 2
11-30 11:22:24.411 16586-16586/com.rxjava2.android.samples D/ScanExampleActivity:  onNext value : 3
      int1 : 3
      int2 : 3
11-30 11:22:24.414 16586-16586/com.rxjava2.android.samples D/ScanExampleActivity:  onNext value : 6
      int1 : 6
      int2 : 4
11-30 11:22:24.417 16586-16586/com.rxjava2.android.samples D/ScanExampleActivity:  onNext value : 10
      int1 : 10
      int2 : 5
11-30 11:22:24.419 16586-16586/com.rxjava2.android.samples D/ScanExampleActivity:  onNext value : 15
11-30 11:22:24.420 16586-16586/com.rxjava2.android.samples D/ScanExampleActivity:  onComplete
```
* 16、PublishSubject 我个人理解的话，就是一堆数据我要发送给别人，但是呢最后几个数字我又要发送给其他人，所以就需要使用到这个，使用重放操作符，重放确保所有观察者看到相同的序列。发射项目，即使它们订阅后，可观测已经开始发射项目。我个人理解的是，发送一个buffer，给第一个观察者，同时我要把这个buffer的尾部长度为4的在发送给第二个观察者。

```
 private void doSomeWork() {
        PublishSubject<Integer> source = PublishSubject.create();
        ConnectableObservable<Integer> connectableObservable = source.replay(4); 
         //连接可连接的可观察的
        connectableObservable.connect();
        connectableObservable.subscribe(getFirstObserver());
        source.onNext(1);
        source.onNext(2);
        source.onNext(3);
        source.onNext(4);
        source.onNext(5);
        source.onNext(6);
        source.onNext(7);
        source.onComplete();
        connectableObservable.subscribe(getSecondObserver());
    }


    private Observer<Integer> getFirstObserver() {
        return new Observer<Integer>() {

            @Override
            public void onSubscribe(Disposable d) {
                Log.d(TAG, " First onSubscribe : " + d.isDisposed());
            }

            @Override
            public void onNext(Integer value) {
                textView.append(" First onNext : value : " + value);
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " First onNext value : " + value);
            }

            @Override
            public void onError(Throwable e) {
                textView.append(" First onError : " + e.getMessage());
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " First onError : " + e.getMessage());
            }

            @Override
            public void onComplete() {
                textView.append(" First onComplete");
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " First onComplete");
            }
        };
    }

    private Observer<Integer> getSecondObserver() {
        return new Observer<Integer>() {

            @Override
            public void onSubscribe(Disposable d) {
                textView.append(" Second onSubscribe : isDisposed :" + d.isDisposed());
                Log.d(TAG, " Second onSubscribe : " + d.isDisposed());
                textView.append(AppConstant.LINE_SEPARATOR);
            }

            @Override
            public void onNext(Integer value) {
                textView.append(" Second onNext : value : " + value);
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " Second onNext value : " + value);
            }

            @Override
            public void onError(Throwable e) {
                textView.append(" Second onError : " + e.getMessage());
                Log.d(TAG, " Second onError : " + e.getMessage());
            }

            @Override
            public void onComplete() {
                textView.append(" Second onComplete");
                Log.d(TAG, " Second onComplete");
            }
        };
    }
```
* 输出结果
```
11-30 11:39:02.882 19690-19690/com.rxjava2.android.samples D/ReplayExampleActivity:  First onSubscribe : false
11-30 11:39:02.903 19690-19690/com.rxjava2.android.samples D/ReplayExampleActivity:  First onNext value : 1
11-30 11:39:02.911 19690-19690/com.rxjava2.android.samples D/ReplayExampleActivity:  First onNext value : 2
11-30 11:39:02.919 19690-19690/com.rxjava2.android.samples D/ReplayExampleActivity:  First onNext value : 3
11-30 11:39:02.926 19690-19690/com.rxjava2.android.samples D/ReplayExampleActivity:  First onNext value : 4
11-30 11:39:02.932 19690-19690/com.rxjava2.android.samples D/ReplayExampleActivity:  First onNext value : 5
11-30 11:39:02.938 19690-19690/com.rxjava2.android.samples D/ReplayExampleActivity:  First onNext value : 6
11-30 11:39:02.944 19690-19690/com.rxjava2.android.samples D/ReplayExampleActivity:  First onNext value : 7
11-30 11:39:02.950 19690-19690/com.rxjava2.android.samples D/ReplayExampleActivity:  First onComplete
11-30 11:39:02.954 19690-19690/com.rxjava2.android.samples D/ReplayExampleActivity:  Second onSubscribe : false
11-30 11:39:02.963 19690-19690/com.rxjava2.android.samples D/ReplayExampleActivity:  Second onNext value : 4
11-30 11:39:02.968 19690-19690/com.rxjava2.android.samples D/ReplayExampleActivity:  Second onNext value : 5
11-30 11:39:02.972 19690-19690/com.rxjava2.android.samples D/ReplayExampleActivity:  Second onNext value : 6
11-30 11:39:02.977 19690-19690/com.rxjava2.android.samples D/ReplayExampleActivity:  Second onNext value : 7
11-30 11:39:02.979 19690-19690/com.rxjava2.android.samples D/ReplayExampleActivity:  Second onComplete
```
*  17 、依次的发送两个数组，而且里面两个数组的是有序的输出的，所以就要使用到这个里面的
```
 /**
     * 使用CONTAT运算符组合可观察性：CONTAT维护
     * 可观察的顺序。
     * 将按顺序发射所有7个值
     * 这里第一个“A1”，“A2”，“A3”，“A4”，然后是“B1”，“B2”，“B3”。
     * 首先从第一个观察到然后
     * 所有从第二可观察到的所有顺序
     */
    private void doSomeWork() {
        final String[] aStrings = {"A1", "A2", "A3", "A4"};
        final String[] bStrings = {"B1", "B2", "B3"};

        final Observable<String> aObservable = Observable.fromArray(aStrings);
        final Observable<String> bObservable = Observable.fromArray(bStrings);

        Observable.concat(aObservable, bObservable)
                .subscribe(getObserver());
    }


    private Observer<String> getObserver() {
        return new Observer<String>() {

            @Override
            public void onSubscribe(Disposable d) {
                Log.d(TAG, " onSubscribe : " + d.isDisposed());
            }

            @Override
            public void onNext(String value) {
                textView.append(" onNext : value : " + value);
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " onNext : value : " + value);
            }

            @Override
            public void onError(Throwable e) {
                textView.append(" onError : " + e.getMessage());
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " onError : " + e.getMessage());
            }

            @Override
            public void onComplete() {
                textView.append(" onComplete");
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " onComplete");
            }
        };
    }
```
* 输出结果
```
11-30 11:42:43.359 19690-19690/com.rxjava2.android.samples D/ConcatExampleActivity:  onSubscribe : false
11-30 11:42:43.369 19690-19690/com.rxjava2.android.samples D/ConcatExampleActivity:  onNext : value : A1
11-30 11:42:43.375 19690-19690/com.rxjava2.android.samples D/ConcatExampleActivity:  onNext : value : A2
11-30 11:42:43.382 19690-19690/com.rxjava2.android.samples D/ConcatExampleActivity:  onNext : value : A3
11-30 11:42:43.388 19690-19690/com.rxjava2.android.samples D/ConcatExampleActivity:  onNext : value : A4
11-30 11:42:43.393 19690-19690/com.rxjava2.android.samples D/ConcatExampleActivity:  onNext : value : B1
11-30 11:42:43.399 19690-19690/com.rxjava2.android.samples D/ConcatExampleActivity:  onNext : value : B2
11-30 11:42:43.404 19690-19690/com.rxjava2.android.samples D/ConcatExampleActivity:  onNext : value : B3
11-30 11:42:43.409 19690-19690/com.rxjava2.android.samples D/ConcatExampleActivity:  onComplete
```
* 18、依次的发送两个数组，而且里面两个数组的不是有序的输出的，但是我始终没有测出来结果，哎哎 难受的很  日了狗！ RxJava 合并组合两个（或多个）Observable数据源

```
  private void doSomeWork() {
        final String[] aStrings = {"A1", "A2", "A3", "A4","1", "2", "3", "4","5", "6", "7", "8","9", "10", "11", "12",};
        final String[] bStrings = {"B1", "B2", "B3","B1", "B2", "B3","B1", "B2", "B3","B1", "B2", "B3","B1", "B2", "B3","B1", "B2", "B3","B1", "B2", "B3"};

        final Observable<String> aObservable = Observable.fromArray(aStrings);
        final Observable<String> bObservable = Observable.fromArray(bStrings);

        Observable.merge(aObservable, bObservable)
                .subscribe(getObserver());
    }


    private Observer<String> getObserver() {
        return new Observer<String>() {

            @Override
            public void onSubscribe(Disposable d) {
                Log.d(TAG, " onSubscribe : " + d.isDisposed());
            }

            @Override
            public void onNext(String value) {
                textView.append(" onNext : value : " + value);
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " onNext : value : " + value);
            }

            @Override
            public void onError(Throwable e) {
                textView.append(" onError : " + e.getMessage());
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " onError : " + e.getMessage());
            }

            @Override
            public void onComplete() {
                textView.append(" onComplete");
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " onComplete");
            }
        };
    }
```
* 输出结果：合并多个Observables的发射物， Merge 可能会让合并的Observables发射的数据交错（有一个类似的操作符 Concat 不会让数 据交错，它会按顺序一个接着一个发射多个Observables的发射物，虽然我的结果没有测试出来，但是呢？真的有可能数据会交叉！！！！
```
11-30 11:46:44.466 19690-19690/com.rxjava2.android.samples D/MergeExampleActivity:  onSubscribe : false
11-30 11:46:44.477 19690-19690/com.rxjava2.android.samples D/MergeExampleActivity:  onNext : value : A1
11-30 11:46:44.484 19690-19690/com.rxjava2.android.samples D/MergeExampleActivity:  onNext : value : A2
11-30 11:46:44.490 19690-19690/com.rxjava2.android.samples D/MergeExampleActivity:  onNext : value : A3
11-30 11:46:44.495 19690-19690/com.rxjava2.android.samples D/MergeExampleActivity:  onNext : value : A4
11-30 11:46:44.499 19690-19690/com.rxjava2.android.samples D/MergeExampleActivity:  onNext : value : 1
11-30 11:46:44.503 19690-19690/com.rxjava2.android.samples D/MergeExampleActivity:  onNext : value : 2
11-30 11:46:44.508 19690-19690/com.rxjava2.android.samples D/MergeExampleActivity:  onNext : value : 3
11-30 11:46:44.512 19690-19690/com.rxjava2.android.samples D/MergeExampleActivity:  onNext : value : 4
11-30 11:46:44.515 19690-19690/com.rxjava2.android.samples D/MergeExampleActivity:  onNext : value : 5
11-30 11:46:44.517 19690-19690/com.rxjava2.android.samples D/MergeExampleActivity:  onNext : value : 6
11-30 11:46:44.519 19690-19690/com.rxjava2.android.samples D/MergeExampleActivity:  onNext : value : 7
11-30 11:46:44.522 19690-19690/com.rxjava2.android.samples D/MergeExampleActivity:  onNext : value : 8
11-30 11:46:44.525 19690-19690/com.rxjava2.android.samples D/MergeExampleActivity:  onNext : value : 9
11-30 11:46:44.528 19690-19690/com.rxjava2.android.samples D/MergeExampleActivity:  onNext : value : 10
11-30 11:46:44.532 19690-19690/com.rxjava2.android.samples D/MergeExampleActivity:  onNext : value : 11
11-30 11:46:44.535 19690-19690/com.rxjava2.android.samples D/MergeExampleActivity:  onNext : value : 12
11-30 11:46:44.537 19690-19690/com.rxjava2.android.samples D/MergeExampleActivity:  onNext : value : B1
11-30 11:46:44.539 19690-19690/com.rxjava2.android.samples D/MergeExampleActivity:  onNext : value : B2
11-30 11:46:44.540 19690-19690/com.rxjava2.android.samples D/MergeExampleActivity:  onNext : value : B3
11-30 11:46:44.542 19690-19690/com.rxjava2.android.samples D/MergeExampleActivity:  onNext : value : B1
11-30 11:46:44.543 19690-19690/com.rxjava2.android.samples D/MergeExampleActivity:  onNext : value : B2
11-30 11:46:44.546 19690-19690/com.rxjava2.android.samples D/MergeExampleActivity:  onNext : value : B3
11-30 11:46:44.548 19690-19690/com.rxjava2.android.samples D/MergeExampleActivity:  onNext : value : B1
11-30 11:46:44.551 19690-19690/com.rxjava2.android.samples D/MergeExampleActivity:  onNext : value : B2
11-30 11:46:44.553 19690-19690/com.rxjava2.android.samples D/MergeExampleActivity:  onNext : value : B3
11-30 11:46:44.556 19690-19690/com.rxjava2.android.samples D/MergeExampleActivity:  onNext : value : B1
11-30 11:46:44.559 19690-19690/com.rxjava2.android.samples D/MergeExampleActivity:  onNext : value : B2
11-30 11:46:44.561 19690-19690/com.rxjava2.android.samples D/MergeExampleActivity:  onNext : value : B3
11-30 11:46:44.564 19690-19690/com.rxjava2.android.samples D/MergeExampleActivity:  onNext : value : B1
11-30 11:46:44.567 19690-19690/com.rxjava2.android.samples D/MergeExampleActivity:  onNext : value : B2
11-30 11:46:44.570 19690-19690/com.rxjava2.android.samples D/MergeExampleActivity:  onNext : value : B3
11-30 11:46:44.573 19690-19690/com.rxjava2.android.samples D/MergeExampleActivity:  onNext : value : B1
11-30 11:46:44.575 19690-19690/com.rxjava2.android.samples D/MergeExampleActivity:  onNext : value : B2
11-30 11:46:44.577 19690-19690/com.rxjava2.android.samples D/MergeExampleActivity:  onNext : value : B3
11-30 11:46:44.578 19690-19690/com.rxjava2.android.samples D/MergeExampleActivity:  onNext : value : B1
11-30 11:46:44.580 19690-19690/com.rxjava2.android.samples D/MergeExampleActivity:  onNext : value : B2
11-30 11:46:44.581 19690-19690/com.rxjava2.android.samples D/MergeExampleActivity:  onNext : value : B3
11-30 11:46:44.582 19690-19690/com.rxjava2.android.samples D/MergeExampleActivity:  onComplete
```


* 19、让属性跟着数据bean传递下去
Car 类
```
public class Car {

    private String brand;

    public void setBrand(String brand) {
        this.brand = brand;
    }

    public Observable<String> brandDeferObservable() {
        return Observable.defer(new Callable<ObservableSource<? extends String>>() {
            @Override
            public ObservableSource<? extends String> call() {
                return Observable.just(brand);
            }
        });
    }
}
```
* 
```
  /*
     * Defer used for Deferring Observable code until subscription in RxJava
     * 推迟在RxJava订阅可观察代码直到订阅
     */
    private void doSomeWork() {

        Car car = new Car();

        Observable<String> brandDeferObservable = car.brandDeferObservable();
        // 即使我们在创建了可观察的品牌之后设置了品牌，我们也会得到宝马的品牌。如果我们不使用延迟器，我们将没有作为品牌。
        car.setBrand("BMW");  // Even if we are setting the brand after creating Observable
        // we will get the brand as BMW.
        // If we had not used defer, we would have got null as the brand.

        brandDeferObservable
                .subscribe(getObserver());
    }

    private Observer<String> getObserver() {
        return new Observer<String>() {

            @Override
            public void onSubscribe(Disposable d) {
                Log.d(TAG, " onSubscribe : " + d.isDisposed());
            }

            @Override
            public void onNext(String value) {
                textView.append(" onNext : value : " + value);
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " onNext : value : " + value);
            }

            @Override
            public void onError(Throwable e) {
                textView.append(" onError : " + e.getMessage());
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " onError : " + e.getMessage());
            }

            @Override
            public void onComplete() {
                textView.append(" onComplete");
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " onComplete");
            }
        };
    }

```
* 输出结果
```
11-30 14:17:07.380 19690-19690/com.rxjava2.android.samples D/DeferExampleActivity:  onSubscribe : false
11-30 14:17:07.388 19690-19690/com.rxjava2.android.samples D/DeferExampleActivity:  onNext : value : BMW
11-30 14:17:07.392 19690-19690/com.rxjava2.android.samples D/DeferExampleActivity:  onComplete

```
* 20、去重，对数据源，进行去重的操作

```
 /*
     * distinct() suppresses duplicate items emitted by the source Observable.
     * 区别（）抑制由可观察到的源发出的重复项。
     */
    private void doSomeWork() {

        getObservable()
                .distinct()
                .subscribe(getObserver());
    }

    private Observable<Integer> getObservable() {
        return Observable.just(1, 2, 1, 1, 2, 3, 4, 6, 4);
    }


    private Observer<Integer> getObserver() {
        return new Observer<Integer>() {

            @Override
            public void onSubscribe(Disposable d) {
                Log.d(TAG, " onSubscribe : " + d.isDisposed());
            }

            @Override
            public void onNext(Integer value) {
                textView.append(" onNext : value : " + value);
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " onNext value : " + value);
            }

            @Override
            public void onError(Throwable e) {
                Log.d(TAG, " onError : " + e.getMessage());
            }

            @Override
            public void onComplete() {
                Log.d(TAG, " onComplete");
            }
        };
    }
```
* 输出的结果
```
11-30 14:20:04.382 19690-19690/com.rxjava2.android.samples D/DistinctExampleActivity:  onSubscribe : false
11-30 14:20:04.388 19690-19690/com.rxjava2.android.samples D/DistinctExampleActivity:  onNext value : 1
11-30 14:20:04.391 19690-19690/com.rxjava2.android.samples D/DistinctExampleActivity:  onNext value : 2
11-30 14:20:04.394 19690-19690/com.rxjava2.android.samples D/DistinctExampleActivity:  onNext value : 3
11-30 14:20:04.397 19690-19690/com.rxjava2.android.samples D/DistinctExampleActivity:  onNext value : 4
11-30 14:20:04.400 19690-19690/com.rxjava2.android.samples D/DistinctExampleActivity:  onNext value : 6
     onComplete

```
* 21、对数据源传递，但是只不过是取最后一个数，把前面的都不需要了，同时给了一个默认的值 `A1`
```
private void doSomeWork() {
        // the default item ("A1") to emit if the source ObservableSource is empty
        getObservable().last("A1") // the default item ("A1") to emit if the source ObservableSource is empty
                .subscribe(getObserver());
    }

    private Observable<String> getObservable() {
        return Observable.just("A1", "A2", "A3", "A4", "A5", "A6");
    }

    private SingleObserver<String> getObserver() {
        return new SingleObserver<String>() {

            @Override
            public void onSubscribe(Disposable d) {
                Log.d(TAG, " onSubscribe : " + d.isDisposed());
            }

            @Override
            public void onSuccess(String value) {
                textView.append(" onNext : value : " + value);
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " onNext value : " + value);
            }


            @Override
            public void onError(Throwable e) {
                textView.append(" onError : " + e.getMessage());
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " onError : " + e.getMessage());
            }
        };
    }
```
* 输出结果
```
11-30 14:20:42.788 19690-19690/com.rxjava2.android.samples D/DistinctExampleActivity:  onSubscribe : false
11-30 14:20:42.800 19690-19690/com.rxjava2.android.samples D/DistinctExampleActivity:  onNext value : A6
```
* 未完待续。。。。。[Rxjava2-Android-Samlpes（二）](https://www.jianshu.com/p/d58cc2ec3539)
























