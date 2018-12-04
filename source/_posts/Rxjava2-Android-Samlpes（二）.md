---
title: Rxjava2-Android-Samlpes（二）
date: 2018-11-30 09:20:22
tags:
---


* 22、就是同一个消息队列，直接两个观察者共享这个消息

<!--  more  --> 
```
 /* ReplaySubject emits to any observer all of the items that were emitted
     * by the source Observable, regardless of when the observer subscribes.
     *
     * RePress主题向所有观察者发出所有被发射的项目。
     *源可观察，不管观察者何时订阅。
     */
    private void doSomeWork() {

        ReplaySubject<Integer> source = ReplaySubject.create();

        source.subscribe(getFirstObserver()); // it will get 1, 2, 3, 4

        source.onNext(1);
        source.onNext(2);
        source.onNext(3);
        source.onNext(4);
        source.onComplete();

        /*
         * it will emit 1, 2, 3, 4 for second observer also as we have used replay
         */
        source.subscribe(getSecondObserver());

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
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " Second onError : " + e.getMessage());
            }

            @Override
            public void onComplete() {
                textView.append(" Second onComplete");
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " Second onComplete");
            }
        };
    }

```
* 输出结果

```
11-30 14:21:52.062 19690-19690/com.rxjava2.android.samples D/ReplaySubjectExampleActivity:  First onSubscribe : false
11-30 14:21:52.070 19690-19690/com.rxjava2.android.samples D/ReplaySubjectExampleActivity:  First onNext value : 1
11-30 14:21:52.074 19690-19690/com.rxjava2.android.samples D/ReplaySubjectExampleActivity:  First onNext value : 2
11-30 14:21:52.076 19690-19690/com.rxjava2.android.samples D/ReplaySubjectExampleActivity:  First onNext value : 3
11-30 14:21:52.079 19690-19690/com.rxjava2.android.samples D/ReplaySubjectExampleActivity:  First onNext value : 4
11-30 14:21:52.081 19690-19690/com.rxjava2.android.samples D/ReplaySubjectExampleActivity:  First onComplete
11-30 14:21:52.083 19690-19690/com.rxjava2.android.samples D/ReplaySubjectExampleActivity:  Second onSubscribe : false
11-30 14:21:52.087 19690-19690/com.rxjava2.android.samples D/ReplaySubjectExampleActivity:  Second onNext value : 1
11-30 14:21:52.090 19690-19690/com.rxjava2.android.samples D/ReplaySubjectExampleActivity:  Second onNext value : 2
11-30 14:21:52.092 19690-19690/com.rxjava2.android.samples D/ReplaySubjectExampleActivity:  Second onNext value : 3
11-30 14:21:52.094 19690-19690/com.rxjava2.android.samples D/ReplaySubjectExampleActivity:  Second onNext value : 4
11-30 14:21:52.096 19690-19690/com.rxjava2.android.samples D/ReplaySubjectExampleActivity:  Second onComplete
```
* 23、感觉就是两个订阅者，分开订阅 ，其中一个订阅者只关心一个结果
```
 /* PublishSubject emits to an observer only those items that are emitted
     * by the source Observable, subsequent to the time of the subscription.
     * 发布主题仅向观察者发射那些被发射的项目。由来源可观察到，在订阅的时间之后。
     */
    private void doSomeWork() {

        PublishSubject<Integer> source = PublishSubject.create();

        source.subscribe(getFirstObserver()); // it will get 1, 2, 3, 4 and onComplete

        source.onNext(1);
        source.onNext(2);
        source.onNext(3);

        /*
         * it will emit 4 and onComplete for second observer also.
         * 它将发射4和完成为第二观察员也。
         */
        source.subscribe(getSecondObserver());

        source.onNext(4);
        source.onComplete();

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
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " Second onError : " + e.getMessage());
            }

            @Override
            public void onComplete() {
                textView.append(" Second onComplete");
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " Second onComplete");
            }
        };
    }
```
* 输出结果
```
11-30 14:23:25.179 19690-19690/com.rxjava2.android.samples D/PublishSubjectExampleActivity:  First onSubscribe : false
11-30 14:23:25.186 19690-19690/com.rxjava2.android.samples D/PublishSubjectExampleActivity:  First onNext value : 1
11-30 14:23:25.188 19690-19690/com.rxjava2.android.samples D/PublishSubjectExampleActivity:  First onNext value : 2
11-30 14:23:25.191 19690-19690/com.rxjava2.android.samples D/PublishSubjectExampleActivity:  First onNext value : 3
11-30 14:23:25.193 19690-19690/com.rxjava2.android.samples D/PublishSubjectExampleActivity:  Second onSubscribe : false
11-30 14:23:25.198 19690-19690/com.rxjava2.android.samples D/PublishSubjectExampleActivity:  First onNext value : 4
11-30 14:23:25.201 19690-19690/com.rxjava2.android.samples D/PublishSubjectExampleActivity:  Second onNext value : 4
11-30 14:23:25.202 19690-19690/com.rxjava2.android.samples D/PublishSubjectExampleActivity:  First onComplete
11-30 14:23:25.205 19690-19690/com.rxjava2.android.samples D/PublishSubjectExampleActivity:  Second onComplete
```
* 24、BehaviorSubject 发出一个消息，然后订阅了，另外一个订阅者就会马上走这个结果输出，而且是交替输出的，不是单独输出的

```
private void doSomeWork() {

        BehaviorSubject<Integer> source = BehaviorSubject.create();

        source.subscribe(getFirstObserver()); // it will get 1, 2, 3, 4 and onComplete

        source.onNext(1);
        source.onNext(2);
        source.onNext(3);

        /*
         * it will emit 3(last emitted), 4 and onComplete for second observer also.
         */
        source.subscribe(getSecondObserver());

        source.onNext(4);
        source.onComplete();

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
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " Second onError : " + e.getMessage());
            }

            @Override
            public void onComplete() {
                textView.append(" Second onComplete");
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " Second onComplete");
            }
        };
    }

```
* 输出结果：当第一个观察者结束的时候，倒数第二个位置就会接收到值，结果如下 a1 a2 a3 b3 a4 b4
```
11-30 14:27:29.950 19690-19690/com.rxjava2.android.samples D/BehaviorSubjectExampleActivity:  First onSubscribe : false
11-30 14:27:29.955 19690-19690/com.rxjava2.android.samples D/BehaviorSubjectExampleActivity:  First onNext value : 1
11-30 14:27:29.958 19690-19690/com.rxjava2.android.samples D/BehaviorSubjectExampleActivity:  First onNext value : 2
11-30 14:27:29.965 19690-19690/com.rxjava2.android.samples D/BehaviorSubjectExampleActivity:  First onNext value : 3
11-30 14:27:29.967 19690-19690/com.rxjava2.android.samples D/BehaviorSubjectExampleActivity:  Second onSubscribe : false
11-30 14:27:29.970 19690-19690/com.rxjava2.android.samples D/BehaviorSubjectExampleActivity:  Second onNext value : 3
11-30 14:27:29.974 19690-19690/com.rxjava2.android.samples D/BehaviorSubjectExampleActivity:  First onNext value : 4
11-30 14:27:29.978 19690-19690/com.rxjava2.android.samples D/BehaviorSubjectExampleActivity:  Second onNext value : 4
11-30 14:27:29.981 19690-19690/com.rxjava2.android.samples D/BehaviorSubjectExampleActivity:  First onComplete
11-30 14:27:29.985 19690-19690/com.rxjava2.android.samples D/BehaviorSubjectExampleActivity:  Second onComplete
```
* 25、AsyncSubject 就只能观察到最后一个值，另个订阅者 就只能接收到一个值
```
rivate void doSomeWork() {
        AsyncSubject<Integer> source = AsyncSubject.create();
        source.subscribe(getFirstObserver()); // it will emit only 4 and onComplete
        source.onNext(1);
        source.onNext(2);
        source.onNext(3);

        /*
         * it will emit 4 and onComplete for second observer also.
         */
        source.subscribe(getSecondObserver());

        source.onNext(4);
        source.onComplete();

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
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " Second onError : " + e.getMessage());
            }

            @Override
            public void onComplete() {
                textView.append(" Second onComplete");
                textView.append(AppConstant.LINE_SEPARATOR);
                Log.d(TAG, " Second onComplete");
            }
        };
    }

```
* 输出结果
```
11-30 14:31:14.631 19690-19690/com.rxjava2.android.samples D/AsyncSubjectExampleActivity:  First onSubscribe : false
11-30 14:31:14.637 19690-19690/com.rxjava2.android.samples D/AsyncSubjectExampleActivity:  Second onSubscribe : false
11-30 14:31:14.641 19690-19690/com.rxjava2.android.samples D/AsyncSubjectExampleActivity:  First onNext value : 4
11-30 14:31:14.642 19690-19690/com.rxjava2.android.samples D/AsyncSubjectExampleActivity:  First onComplete
11-30 14:31:14.643 19690-19690/com.rxjava2.android.samples D/AsyncSubjectExampleActivity:  Second onNext value : 4
11-30 14:31:14.645 19690-19690/com.rxjava2.android.samples D/AsyncSubjectExampleActivity:  Second onComplete
```
* 26、throttleFirst  一定时间内取第一次发送的事件。例子：防止按钮的连续点击
```
 private void doSomeWork() {
        getObservable()
                .throttleFirst(500, TimeUnit.MILLISECONDS)
                // Run on a background thread
                .subscribeOn(Schedulers.io())
                // Be notified on the main thread
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(getObserver());
    }

    private Observable<Integer> getObservable() {
        return Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
                // send events with simulated time wait
                // TODO: 2018/11/26 发送模拟时间等待事件
                Thread.sleep(0);
                emitter.onNext(1); // deliver
                emitter.onNext(2); // skip 跳过
                Thread.sleep(505);
                emitter.onNext(3); // deliver
                Thread.sleep(99);
                emitter.onNext(4); // skip
                Thread.sleep(100);
                emitter.onNext(5); // skip
                emitter.onNext(6); // skip
                Thread.sleep(305);
                emitter.onNext(7); // deliver
                Thread.sleep(510);
                emitter.onComplete();
            }
        });
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
* 输出结果，说通俗一点就是跨过一段时间输出一个事件，并且时间到了才输出
```
11-30 14:32:50.433 19690-19690/com.rxjava2.android.samples D/ThrottleFirstExampleActivity:  onSubscribe : false
11-30 14:32:50.447 19690-19690/com.rxjava2.android.samples D/ThrottleFirstExampleActivity:  onNext 
     value : 1
11-30 14:32:50.957 19690-19690/com.rxjava2.android.samples D/ThrottleFirstExampleActivity:  onNext 
11-30 14:32:50.958 19690-19690/com.rxjava2.android.samples D/ThrottleFirstExampleActivity:  value : 3
11-30 14:32:51.467 19690-19690/com.rxjava2.android.samples D/ThrottleFirstExampleActivity:  onNext 
     value : 7
11-30 14:32:51.976 19690-19690/com.rxjava2.android.samples D/ThrottleFirstExampleActivity:  onComplete

```
* 27、throttleLast 这个是结尾啊 ，取这段时间的结尾 ，而不是开头的时间
```
  private void doSomeWork() {
        getObservable()
                .throttleLast(500, TimeUnit.MILLISECONDS)
                // Run on a background thread
                .subscribeOn(Schedulers.io())
                // Be notified on the main thread
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(getObserver());
    }

    private Observable<Integer> getObservable() {
        return Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
                // send events with simulated time wait
                Thread.sleep(0);
                emitter.onNext(1); // skip
                emitter.onNext(2); // deliver
                Thread.sleep(505);
                emitter.onNext(3); // skip
                Thread.sleep(99);
                emitter.onNext(4); // skip
                Thread.sleep(100);
                emitter.onNext(5); // skip
                emitter.onNext(6); // deliver
                Thread.sleep(305);
                emitter.onNext(7); // deliver
                Thread.sleep(510);
                emitter.onComplete();
            }
        });
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
* 输出结果，延迟一段时间
```
11-30 14:34:48.726 19690-19690/com.rxjava2.android.samples D/ThrottleLastExampleActivity:  onSubscribe : false
11-30 14:34:49.246 19690-19690/com.rxjava2.android.samples D/ThrottleLastExampleActivity:  onNext 
     value : 2
11-30 14:34:49.745 19690-19690/com.rxjava2.android.samples D/ThrottleLastExampleActivity:  onNext 
     value : 6
11-30 14:34:50.247 19690-19690/com.rxjava2.android.samples D/ThrottleLastExampleActivity:  onNext 
11-30 14:34:50.248 19690-19690/com.rxjava2.android.samples D/ThrottleLastExampleActivity:  value : 7
11-30 14:34:50.274 19690-19690/com.rxjava2.android.samples D/ThrottleLastExampleActivity:  onComplete

```
* 28、类似一个弹簧，如果一个事件相当于挤压它一下的话，它回到初始状态需要一段时间，那如果一直有事件不断的挤压它，那它一直回不到初始状态，就一个事件也弹不出来。一旦有一段时间里面没有人挤压它，他就把最后一个弹出来了。周而复始

```
 private void doSomeWork() {
        getObservable()
                .debounce(500, TimeUnit.MILLISECONDS)
                // Run on a background thread
                .subscribeOn(Schedulers.io())
                // Be notified on the main thread
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(getObserver());
    }

    private Observable<Integer> getObservable() {
        return Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
                // send events with simulated time wait
                emitter.onNext(1); // skip
                Thread.sleep(400);
                emitter.onNext(2); // deliver
                Thread.sleep(505);
                emitter.onNext(3); // skip
                Thread.sleep(100);
                emitter.onNext(4); // deliver
                Thread.sleep(605);
                emitter.onNext(5); // deliver
                Thread.sleep(510);
                emitter.onComplete();
            }
        });
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
* 输出结果：这种的应用场景：在Edittext上添加监听，当里面输入的内容变化后进行搜索。换句话说就是当用户的输入操作停止几秒钟之后再去搜索。
```
11-30 14:42:45.659 19690-19690/com.rxjava2.android.samples D/DebounceExampleActivity:  onSubscribe : false
11-30 14:42:46.584 19690-19690/com.rxjava2.android.samples D/DebounceExampleActivity:  onNext 
     value : 2
11-30 14:42:47.190 19690-19690/com.rxjava2.android.samples D/DebounceExampleActivity:  onNext 
11-30 14:42:47.191 19690-19690/com.rxjava2.android.samples D/DebounceExampleActivity:  value : 4
11-30 14:42:47.799 19690-19690/com.rxjava2.android.samples D/DebounceExampleActivity:  onNext 
11-30 14:42:47.800 19690-19690/com.rxjava2.android.samples D/DebounceExampleActivity:  value : 5
11-30 14:42:47.807 19690-19690/com.rxjava2.android.samples D/DebounceExampleActivity:  onComplete
```
* 29、 RxJava的window()函数和buffer()很像，但是它发射的是Observable而不是列表。
```
  protected void doSomeWork() {
        //interval轮询
        // take()函数用整数N来作为一个参数，从原始的序列中发射前N个元素，然后完成：
        Observable.interval(1, TimeUnit.SECONDS).take(12)
                .window(3, TimeUnit.SECONDS)
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(getConsumer());
    }

    public Consumer<Observable<Long>> getConsumer() {
        return new Consumer<Observable<Long>>() {
            @Override
            public void accept(Observable<Long> observable) {
                Log.d(TAG, "Sub Divide begin....");
                textView.append("Sub Divide begin ....");
                textView.append(AppConstant.LINE_SEPARATOR);
                observable
                        .subscribeOn(Schedulers.io())
                        .observeOn(AndroidSchedulers.mainThread())
                        .subscribe(new Consumer<Long>() {
                            @Override
                            public void accept(Long value) {
                                Log.d(TAG, "Next:" + value);
                                textView.append("Next:" + value);
                                textView.append(AppConstant.LINE_SEPARATOR);
                            }
                        });
            }
        };
    }
```
* 输出结果:一组一组的输出
```
11-30 14:44:56.147 19690-19690/com.rxjava2.android.samples D/WindowExampleActivity: Sub Divide begin....
11-30 14:44:57.154 19690-19690/com.rxjava2.android.samples D/WindowExampleActivity: Next:0
11-30 14:44:58.151 19690-19690/com.rxjava2.android.samples D/WindowExampleActivity: Next:1
11-30 14:44:59.149 19690-19690/com.rxjava2.android.samples D/WindowExampleActivity: Next:2
11-30 14:44:59.164 19690-19690/com.rxjava2.android.samples D/WindowExampleActivity: Sub Divide begin....
11-30 14:45:00.148 19690-19690/com.rxjava2.android.samples D/WindowExampleActivity: Next:3
11-30 14:45:01.150 19690-19690/com.rxjava2.android.samples D/WindowExampleActivity: Next:4
11-30 14:45:02.150 19690-19690/com.rxjava2.android.samples D/WindowExampleActivity: Next:5
11-30 14:45:02.164 19690-19690/com.rxjava2.android.samples D/WindowExampleActivity: Sub Divide begin....
11-30 14:45:03.151 19690-19690/com.rxjava2.android.samples D/WindowExampleActivity: Next:6
11-30 14:45:04.150 19690-19690/com.rxjava2.android.samples D/WindowExampleActivity: Next:7
11-30 14:45:05.149 19690-19690/com.rxjava2.android.samples D/WindowExampleActivity: Sub Divide begin....
11-30 14:45:05.174 19690-19690/com.rxjava2.android.samples D/WindowExampleActivity: Next:8
11-30 14:45:06.150 19690-19690/com.rxjava2.android.samples D/WindowExampleActivity: Next:9
11-30 14:45:07.152 19690-19690/com.rxjava2.android.samples D/WindowExampleActivity: Next:10
```
* 30、 使用延迟2秒后发射的简单示例
```
 private void doSomeWork() {
        getObservable().delay(2, TimeUnit.SECONDS)
                // Run on a background thread
                .subscribeOn(Schedulers.io())
                // Be notified on the main thread
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(getObserver());
    }

    private Observable<String> getObservable() {
        return Observable.just("Amit");
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
* 延迟2s输出结果

* 31、 每当源Observable发出一个新项时，它将取消订阅并停止镜像从先前发出的项生成的Observable，并开始只镜像当前项。
```
  private void doSomeWork() {
        getObservable()
                .switchMap(new Function<Integer, ObservableSource<String>>() {
                    @Override
                    public ObservableSource<String> apply(Integer integer) {
                        int delay = new Random().nextInt(2);

                        return Observable.just(integer.toString() + "x")
                                .delay(delay, TimeUnit.SECONDS, Schedulers.io());
                    }
                })
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(getObserver());
    }

    private Observable<Integer> getObservable() {
        return Observable.just(1, 2, 3, 4, 5);
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
* 输出结果:输出最后的值
```
11-30 14:47:59.787 19690-19690/com.rxjava2.android.samples D/SwitchMapExampleActivity:  onSubscribe : false
11-30 14:48:00.814 19690-19690/com.rxjava2.android.samples D/SwitchMapExampleActivity:  onNext value : 5x
11-30 14:48:00.816 19690-19690/com.rxjava2.android.samples D/SwitchMapExampleActivity:  onComplete

```
#### RxBus

### NetWork

### Pagination 分页Demo

### SearchDemo，主要是延迟请求网络，和 `SearchView` 相互的结合 



* 最后说明几点
* 1、Rxjava2定时器周期执行任务，可以很好地避免内存泄漏，因为使用handler如果处理的不好，容易内存泄露
* 2、RxJava2实现定时器的优势也是简洁，但它的简洁的与众不同之处在于，随着程序逻辑变得越来越复杂，它依然能够保持简洁。
* 3、[三角形数](https://zh.wikipedia.org/wiki/%E4%B8%89%E8%A7%92%E5%BD%A2%E6%95%B8) 1 3 6 10 15 ... 一定数目的点或圆在等距离的排列下可以形成一个等边三角形，这样的数被称为三角形数。比如10个点可以组成一个等边三角形，因此10是一个三角形数
* 4、合并多个Observables的发射物， Merge 可能会让合并的Observables发射的数据交错（有一个类似的操作符 Concat 不会让数 据交错，它会按顺序一个接着一个发射多个Observables的发射物