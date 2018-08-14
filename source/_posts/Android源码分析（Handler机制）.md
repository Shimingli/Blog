---
title: Android源码分析（Handler机制）
date: 2018-05-26 16:20:50
tags: [Handler机制]
---
## 源码基于安卓8.0分析结果
#### 关键类ActivityThread、Handler、Looper、Message、MessageQueue
* ActivityThread中的流程：应用程序入口是在ActivityThread的main方法中，程序启动，底层去调用C/C++去调用main方法
<!--  more  -->
* ActivityThread中的main的方法
```
 /*
将当前线程初始化为一个活套，将其标记为
*应用程序的主要活套。应用程序的主要套接字
*是由Android环境创建的，所以您永远不需要
*自己调用这个函数
 */
 public static void main(String[] args) {
        Looper.prepareMainLooper();
        ActivityThread thread = new ActivityThread();
        thread.attach(false);
        if (sMainThreadHandler == null) { sMainThreadHandler = thread.getHandler(); }
        //if(false){}之类的语句，这种写法是方便调试的，通过一个标志就可以控制某些代码是否执行，比如说是否输出一些系统的Log
        if (false) { myLooper().setMessageLogging(new LogPrinter(Log.DEBUG, "ActivityThread")); }
        Looper.loop();
}
```
*    Looper.prepareMainLooper();方法
```
    ////Looper的prepare方法，并且关联到主线程
    public static void prepareMainLooper() {
        //Only one Looper may be created per thread"
        // false意思不允许我们程序员退出（面向我们开发者），因为这是在主线程里面
        // TODO: 2018/5/17   
        prepare(false);
        synchronized (Looper.class) {
            if (sMainLooper != null) {
                throw new IllegalStateException("The main Looper has already been prepared.");
            }
            //把Looper设置为主线程的Looper
            sMainLooper = myLooper();
        }
    }
```
* 关于prepare（false）方法： Only one Looper may be created per thread 也就是说，一个线程只有一个Looper对象，要不然会抛出异常

```
   private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));
    }
```
* 关于ThreadLocal的set方法，可以找到ThreadLocal的构造函数，底层的实现是一个Entry的数组.
```
 ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();
 //1、ThreadLocal的set方法
  public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }
 //2、ThreadLocal的createMap方法
  void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }
```
ThreadLocalMap的构造函数里面有一个长度为16的Entry的数组，当然这个机制和HashMap差不多，也有扩容机制,就是当容器装不下了，在此的基础上增加一倍的长度，同时把原来的数据copy到新的Entry数组中
```
 //3、ThreadLocal的构造函数
     ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
            table = new Entry[INITIAL_CAPACITY];
            int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
            table[i] = new Entry(firstKey, firstValue);
            size = 1;
            setThreshold(INITIAL_CAPACITY);
        }
```
ThreadLocalMap的（扩容机制）Double the capacity of the table.
```
      /**
         * Double the capacity of the table.
         */
        private void resize() {
            Entry[] oldTab = table;
            int oldLen = oldTab.length;
            int newLen = oldLen * 2;
            Entry[] newTab = new Entry[newLen];
            int count = 0;

            for (int j = 0; j < oldLen; ++j) {
                Entry e = oldTab[j];
                if (e != null) {
                    ThreadLocal<?> k = e.get();
                    if (k == null) {
                        e.value = null; // Help the GC
                    } else {
                        int h = k.threadLocalHashCode & (newLen - 1);
                        while (newTab[h] != null)
                            h = nextIndex(h, newLen);
                        newTab[h] = e;
                        count++;
                    }
                }
            }

            setThreshold(newLen);
            size = count;
            table = newTab;
        }
```

* 同时关注Entry的类，可以发现这是WeakReference的子类，关系到了弱引用：弱引用是比软引用更弱的一种的引用的类型，只有弱引用指向的对象的生命周期更短，当垃圾回收器扫描到只有具有弱引用的对象的时候，不敢当前空间是否不足，都会对弱引用对象进行回收，不太明白的可以看我另外一篇文章 [安卓代码、图片、布局、网络和电量优化](https://www.jianshu.com/p/82b76e0cb41e)
```
  static class Entry extends WeakReference<ThreadLocal<?>> {
            /** The value associated with this ThreadLocal. */
            Object value;
            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }
```

## 为什么我要提起它？？？可能我理解的不太准确，肯定不太准确，在我现在工作中，维护和开发一个硬件的应用早餐机（用户通过App预定早餐，第二天早上去机器上取早餐），就像蜂巢的快递柜一样，取早餐的机器，在深圳工作的大佬，可能也许看见过我们的机器，[PLC、安卓、物联网](https://www.jianshu.com/p/82b88ac8e329)这篇文章有详细的介绍。在测试过程中，由于App常驻在前台，有几率导致App直接挂掉，通过日志发现是内存不足，直接kill了这个App，我想这里可能就是这个原因，这个一个弱应用，只要虚拟机扫描导致这里了，我不管你了，我直接把你回收掉。仅仅是个人的理解，同时我们安卓的开发板也不太稳定，如果在这一点有见解的大佬，欢迎讨论，谢谢了


*   Looper.loop();根据我们的常识知道，如果程序没有死循环的话，执行完main函数（比如构建视图等等代码）以后就会立马退出了。之所以我们的APP能够一直运行着，就是因为Looper.loop()里面是一个死循环
      *  1、 首先拿到Looper对象（me），如果当前的线程没有Looper，那么就会抛出异常， // TODO: 2018/5/17   在子线程中创建handler的话，需要looper也要准备好 ，要不然会报错。这就是为什么在子线程里面创建Handler如果不手动创建和启动Looper会报错的原因
          *  这个Looper对象就是通过sThreadLocal.get();细心的话可以发现前面已经sThreadLocal.set()
```
   public static @Nullable Looper myLooper() {
        return sThreadLocal.get();
    }
```


  *  2、然后拿到Looper的成员变量MessageQueue，在MessageQueue里面不断地去取消息，关于MessageQueue的next方法如下：如果这个msg为null的，这个结束掉这个
 *    3、msg.target.dispatchMessage(msg)就是处理消息，紧接着在loop方法的最后调用了msg.recycleUnchecked()这就是回收了Message。
    *  4、我们平时写Handler的时候不需要我们手动回收，因为谷歌的工程师已经有考虑到这方面的问题了。消息是在Handler分发处理之后就会被自动回收的：


```
    public static void loop() {
        final Looper me = myLooper();
        if (me == null) {
            // TODO: 2018/5/17   在子线程中创建handler的话，需要looper也要准备好 ，要不然会报错
            // 1、 首先拿到Looper对象（me），如果当前的线程没有Looper，那么就会抛出异常，
            // 这就是为什么在子线程里面创建Handler如果不手动创建和启动Looper会报错的原因
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        final MessageQueue queue = me.mQueue;

        // Make sure the identity of this thread is that of the local process,
        // and keep track of what that identity token actually is.
        Binder.clearCallingIdentity();
        final long ident = Binder.clearCallingIdentity();

        for (; ; ) {
            // TODO: 2018/5/17   Message
           // 2、然后拿到Looper的成员变量MessageQueue，在MessageQueue里面不断地去取消息，关于MessageQueue的next方法如下：
            Message msg = queue.next(); // might block
            if (msg == null) {
                // No message indicates that the message queue is quitting.
                return;
            }

            // This must be in a local variable, in case a UI event sets the logger
            final Printer logging = me.mLogging;
            if (logging != null) {
                logging.println(">>>>> Dispatching to " + msg.target + " " +
                        msg.callback + ": " + msg.what);
            }

            final long slowDispatchThresholdMs = me.mSlowDispatchThresholdMs;

            final long traceTag = me.mTraceTag;
            if (traceTag != 0 && Trace.isTagEnabled(traceTag)) {
                Trace.traceBegin(traceTag, msg.target.getTraceName(msg));
            }
            final long start = (slowDispatchThresholdMs == 0) ? 0 : SystemClock.uptimeMillis();
            final long end;
          //  msg.target.dispatchMessage(msg)就是处理消息，紧接着在loop方法的最后调用了msg.recycleUnchecked()这就是回收了Message。
            // TODO: 2018/5/17
            处理消息
            try {
                处理消息
                // TODO: 2018/5/17  msg中的target 就是handler的本体的对象  ，直接去handler中发送这个对象
                msg.target.dispatchMessage(msg);
                end = (slowDispatchThresholdMs == 0) ? 0 : SystemClock.uptimeMillis();
            } finally {
                if (traceTag != 0) {
                    Trace.traceEnd(traceTag);
                }
            }
            if (slowDispatchThresholdMs > 0) {
                final long time = end - start;
                if (time > slowDispatchThresholdMs) {
                    Slog.w(TAG, "Dispatch took " + time + "ms on "
                            + Thread.currentThread().getName() + ", h=" +
                            msg.target + " cb=" + msg.callback + " msg=" + msg.what);
                }
            }

            if (logging != null) {
                logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
            }

            // Make sure that during the course of dispatching the
            // identity of the thread wasn't corrupted.
            final long newIdent = Binder.clearCallingIdentity();
            if (ident != newIdent) {
                Log.wtf(TAG, "Thread identity changed from 0x"
                        + Long.toHexString(ident) + " to 0x"
                        + Long.toHexString(newIdent) + " while dispatching to "
                        + msg.target.getClass().getName() + " "
                        + msg.callback + " what=" + msg.what);
            }
            // TODO: 2018/5/17
            我们平时写Handler的时候不需要我们手动回收，因为谷歌的工程师已经有考虑到这方面的问题了。消息是在Handler分发处理之后就会被自动回收的：
            msg.recycleUnchecked();
        }
    }

```
* 关于Message类
 1、对象是实现了Parcelable接口的，因为Message消息可能需要跨进程通信，这时候就需要进程序列化以及反序列化操作了。 
```
public final class Message implements Parcelable Message   
```
2、obtain()得到Message对象，其中的设计模式享元模式：我见过最好的Demo,理解：采用一个共享类避免大量拥有相同的内容的“小类的开销”
享元模式德优缺点：优点在于大幅度的降低内存中对象的数量，但是，它做到这一点代价优点高，享元模式使得系统更加复杂为了使对象可以共享，需要将一些状态外部化，这使得一些程序逻辑更加的复杂享元模式将享元对象的状态外部化，而读取外部状态使得运行的时间稍微变长，更多的Demo可以看这篇文章[二十三种设计模式](https://www.jianshu.com/p/4e01479b6a2c)
```
 public static Message obtain() {
        synchronized (sPoolSync) {
            if (sPool != null) {
                Message m = sPool;
                sPool = m.next;
                m.next = null;
                m.flags = 0; // clear in-use flag
                sPoolSize--;
                return m;
            }
        }
        return new Message();
    }
```

3、消息的回收机制方法一:这个方法调用的时机是在MessageQueueen中。MessageQueueen.queueMessage(Message msg, long when)
```
    public void recycle() {
        if (isInUse()) {
            if (gCheckRecycle) {
                throw new IllegalStateException("This message cannot be recycled because it "
                        + "is still in use.");
            }
            return;
        }
        recycleUnchecked();
    }
```
4、消息的回收机制方法二:调用的地方是在  Looper.loop();谷歌的工程师帮我们调用，所以我们在开发过程中，没有去调用这个消息回收，哈哈，向谷歌致敬，同时这个方法也会在recycle中调用。
```
 void recycleUnchecked() {
        // Mark the message as in use while it remains in the recycled object pool.
        // Clear out all other details.
        flags = FLAG_IN_USE;
        what = 0;
        arg1 = 0;
        arg2 = 0;
        obj = null;
        replyTo = null;
        sendingUid = -1;
        when = 0;
        target = null;
        callback = null;
        data = null;

        synchronized (sPoolSync) {
            if (sPoolSize < MAX_POOL_SIZE) {
                next = sPool;
                sPool = this;
                sPoolSize++;
            }
        }
    }
```
* 关于MessageQueue类
1、 Message next() 方法:看到消息的取出用到了一些native方法，这样做是为了获得更高的效率，消息的去取出并不是直接就从队列的头部取出的，而是根据了消息的when时间参数有关的，因为我们可以发送延时消息、也可以发送一个指定时间点的消息
   * 1、for循环的使用native 方法
   *  2、根据时间戳获取消息
```
  Message next() {
        final long ptr = mPtr;
        if (ptr == 0) {
            return null;
        }
        int pendingIdleHandlerCount = -1; // -1 only during first iteration
        int nextPollTimeoutMillis = 0;
        //防止被反射修改了这个标记，直接写出for循环
        for (;;) {
            if (nextPollTimeoutMillis != 0) {
                Binder.flushPendingCommands();
            }
            //native 方法
            nativePollOnce(ptr, nextPollTimeoutMillis);

            synchronized (this) {
                // Try to retrieve the next message.  Return if found.
                //拿到当前的时间戳
                final long now = SystemClock.uptimeMillis();
                Message prevMsg = null;
                Message msg = mMessages;
                //判断头指针的Target（Handler是否为空（因为头指针只是一个指针的作用））
                if (msg != null && msg.target == null) {
                    // Stalled by a barrier.  Find the next asynchronous message in the queue.
                    do {
                        //遍历下一条Message
                        prevMsg = msg;
                        msg = msg.next;
                    } while (msg != null && !msg.isAsynchronous());
                }
                if (msg != null) {
                    if (now < msg.when) {
                        //还没有到执行的时间
                        nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                    } else {
                        //到了执行时间，直接返回
                        mBlocked = false;
                        if (prevMsg != null) {
                            //拿出消息，断开链表
                            prevMsg.next = msg.next;
                        } else {
                            mMessages = msg.next;
                        }
                        msg.next = null;
                        if (DEBUG) Log.v(TAG, "Returning message: " + msg);
                        msg.markInUse();
                        return msg;
                    }
                } else {
                    // No more messages.
                    nextPollTimeoutMillis = -1;
                }

                // Process the quit message now that all pending messages have been handled.
                if (mQuitting) {
                    dispose();
                    return null;
                }

                // If first time idle, then get the number of idlers to run.
                // Idle handles only run if the queue is empty or if the first message
                // in the queue (possibly a barrier) is due to be handled in the future.
                if (pendingIdleHandlerCount < 0
                        && (mMessages == null || now < mMessages.when)) {
                    pendingIdleHandlerCount = mIdleHandlers.size();
                }
                if (pendingIdleHandlerCount <= 0) {
                    // No idle handlers to run.  Loop and wait some more.
                    mBlocked = true;
                    continue;
                }

                if (mPendingIdleHandlers == null) {
                    mPendingIdleHandlers = new IdleHandler[Math.max(pendingIdleHandlerCount, 4)];
                }
                mPendingIdleHandlers = mIdleHandlers.toArray(mPendingIdleHandlers);
            }

            // Run the idle handlers.
            // We only ever reach this code block during the first iteration.
            for (int i = 0; i < pendingIdleHandlerCount; i++) {
                final IdleHandler idler = mPendingIdleHandlers[i];
                mPendingIdleHandlers[i] = null; // release the reference to the handler

                boolean keep = false;
                try {
                    keep = idler.queueIdle();
                } catch (Throwable t) {
                    Log.wtf(TAG, "IdleHandler threw exception", t);
                }

                if (!keep) {
                    synchronized (this) {
                        mIdleHandlers.remove(idler);
                    }
                }
            }

            // Reset the idle handler count to 0 so we do not run them again.
            pendingIdleHandlerCount = 0;

            // While calling an idle handler, a new message could have been delivered
            // so go back and look again for a pending message without waiting.
            nextPollTimeoutMillis = 0;
        }
    }
```

2、 boolean enqueueMessage(Message msg, long when)方法中调用了  msg.recycle();
```
  boolean enqueueMessage(Message msg, long when) {
        if (msg.target == null) {
            throw new IllegalArgumentException("Message must have a target.");
        }
        if (msg.isInUse()) {
            throw new IllegalStateException(msg + " This message is already in use.");
        }

        synchronized (this) {
            if (mQuitting) {
                IllegalStateException e = new IllegalStateException(
                        msg.target + " sending message to a Handler on a dead thread");
                Log.w(TAG, e.getMessage(), e);
                // TODO: 2018/5/21  在这里调用的 释放消息
                msg.recycle();
                return false;
            }
}
```

2、quit的方法: Looper.myLooper().quit();调用的就是下面的方法 只不过safe==false
```
 void quit(boolean safe) {
        if (!mQuitAllowed) {
            throw new IllegalStateException("Main thread not allowed to quit.");
        }

        synchronized (this) {
            if (mQuitting) {
                return;
            }
            //置位正在退出的标志
            mQuitting = true;
           //清空所有消息
            if (safe) {
                //安全的（系统的），未来未处理的消息都移除
                removeAllFutureMessagesLocked();
            } else {
                //如果是不安全的，例如我们自己定义的消息，就一次性全部移除掉
                removeAllMessagesLocked();
            }

            // We can assume mPtr != 0 because mQuitting was previously false.
            nativeWake(mPtr);
        }
    }
```
*   安全的（系统的），未来未处理的消息都移除
```
 private void removeAllFutureMessagesLocked() {
        final long now = SystemClock.uptimeMillis();
        Message p = mMessages;
        if (p != null) {
            if (p.when > now) {
                //如果所有消息都处理完了，就一次性把全部消息移除掉
                removeAllMessagesLocked();
            } else {
                //否则就通过for循环拿到还没有把还没有执行的Message，利用do循环
                //把这些未处理的消息通过recycleUnchecked方法回收，放回到消息池里面
                Message n;
                for (;;) {
                    n = p.next;
                    if (n == null) {
                        return;
                    }
                    if (n.when > now) {
                        break;
                    }
                    p = n;
                }
                p.next = null;
                do {
                    p = n;
                    n = p.next;
                    p.recycleUnchecked();
                } while (n != null);
            }
        }
    }
```
* 如果是不安全的，例如我们自己定义的消息，就一次性全部移除掉
```
    private void removeAllMessagesLocked() {
        Message p = mMessages;
        while (p != null) {
            Message n = p.next;
            p.recycleUnchecked();
            p = n;
        }
        mMessages = null;
    }
```
* 关于Handler发送消息流程
* 通过一个Handler发送一个延迟5s的消息，
```
  innerHandler.postDelayed(new Runnable() {
            @Override
            public void run() {
                //todo
            }
        },5000);
```
* 调用到Handler中的postDelayed方法
```
    public final boolean postDelayed(Runnable r, long delayMillis) {
        return sendMessageDelayed(getPostMessage(r), delayMillis);
    }
```
* getPostMessage(r)方法:通过obtain得到一个Message的对象
```
   private static Message getPostMessage(Runnable r) {
        Message m = Message.obtain();
        m.callback = r;
        return m;
    }
```
* sendMessageDelayed(getPostMessage(r), delayMillis):这里的delayMillis的时间小于0的话，也会为0
```
  public final boolean sendMessageDelayed(Message msg, long delayMillis)
    {
        if (delayMillis < 0) {
            delayMillis = 0;
        }
        return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
    }
```
*  关于sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis)；SystemClock.uptimeMillis() 系统的时间返回为milliseconds==毫秒，在这个方法就可以看出，MessageQueue不可获取
```
  public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
        MessageQueue queue = mQueue;
        if (queue == null) {
            RuntimeException e = new RuntimeException(
                    this + " sendMessageAtTime() called with no mQueue");
            Log.w("Looper", e.getMessage(), e);
            return false;
        }
        return enqueueMessage(queue, msg, uptimeMillis);
    }
```
* 那么关于mQueue何时初始化的呢，请看代码分析
1、我们平时都是new Handler（），开始使用的
```
    public Handler() {
        this(null, false);
    }
```
 直接调用了这里的方法，同时这个构造方法是Hide了的，在外界调用不掉，为啥把它Handler，我还不太懂，反正可以注意到    mQueue = mLooper.mQueue; 原MessageQueue是在Looper中初始化的，ok，往下走
```
   public Handler(Callback callback, boolean async) {
        if (FIND_POTENTIAL_LEAKS) {
            final Class<? extends Handler> klass = getClass();
            if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &&
                    (klass.getModifiers() & Modifier.STATIC) == 0) {
                Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
                    klass.getCanonicalName());
            }
        }

        mLooper = Looper.myLooper();
        if (mLooper == null) {
            throw new RuntimeException(
                "Can't create handler inside thread that has not called Looper.prepare()");
        }
        mQueue = mLooper.mQueue;
        mCallback = callback;
        mAsynchronous = async;
    }
```
Looper的构造方法，可以看到MessageQueue初始化
```
   private Looper(boolean quitAllowed) {
        mQueue = new MessageQueue(quitAllowed);
        mThread = Thread.currentThread();
    }
```
那么Looper又在哪里初始化的呢:通过代码可以发现prepare()方法中初始化，通过前面的代码的分析又在ActivityThread中的main方法，通过调用Looper.prepareMainLooper（）方法
```
   private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));
    
```

* 关于enqueueMessage(queue, msg, uptimeMillis);其实也就是调用到MessageQueue.enqueueMessage方法
```
   private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
        msg.target = this;
        if (mAsynchronous) {
            msg.setAsynchronous(true);
        }
        return queue.enqueueMessage(msg, uptimeMillis);
    }
```

* 关于MessageQueue.enqueueMessage():对msg一些的赋值，同时呢，也调用了，本地方法，这样性能很高，如果真的需要看懂源码的流程，一定打个断点，一步步的走下去，就可以看到很良好的结果。
```
 boolean enqueueMessage(Message msg, long when) {
        //1、目标为空，那么抛出异常
        if (msg.target == null) {
            throw new IllegalArgumentException("Message must have a target.");
        }
        //2、如果这个消息已经被使用了的话，也抛出异常
        // /*package*/ boolean isInUse() {
        //        return ((flags & FLAG_IN_USE) == FLAG_IN_USE);
        //    }
        if (msg.isInUse()) {
            throw new IllegalStateException(msg + " This message is already in use.");
        }

        synchronized (this) {
            //3、如果是退出了，就是App退出了，退出了的标记
            if (mQuitting) {
                IllegalStateException e = new IllegalStateException(
                        msg.target + " sending message to a Handler on a dead thread");
                Log.w(TAG, e.getMessage(), e);
                // TODO: 2018/5/21  在这里调用的 释放消息
                msg.recycle();
                return false;
            }
            //4、标记它正在使用中，
            msg.markInUse();
            //5、当前的时间
            msg.when = when;
            Message p = mMessages;
            boolean needWake;
            if (p == null || when == 0 || when < p.when) {
                // 6、新的头，如果阻塞，唤醒事件队列。
                msg.next = p;
                mMessages = msg;
                needWake = mBlocked;
            } else {
                // Inserted within the middle of the queue.  Usually we don't have to wake
                // up the event queue unless there is a barrier at the head of the queue
                // and the message is the earliest asynchronous message in the queue.
                //插入队列中间。通常我们不必醒来
                //增加事件队列，除非队列头上有障碍物。
                //消息是队列中最早的异步消息。
                needWake = mBlocked && p.target == null && msg.isAsynchronous();
                Message prev;
                //7、for保持消息的不断的移动
                for (;;) {
                    //前一个消息，如果走到这里，那么这个p不会为null
                    prev = p;
                    //把这个消息下一个赋值给P，如果下个值为null的话，就直接break
                    p = p.next;
                    if (p == null || when < p.when) {
                        break;
                    }
                    //8、需要醒来，同时消息是异步的
                    if (needWake && p.isAsynchronous()) {
                        needWake = false;
                    }
                }
                msg.next = p; // invariant: p == prev.next
                prev.next = msg;
            }

            // We can assume mPtr != 0 because mQuitting is false.
            if (needWake) {
                nativeWake(mPtr);
            }
        }
        return true;
    }
```
* 这样MessageQueue.next() 方法,就不断的取出Message，做相应的动作,
* 如何分发消息呢？还得看Looper.loop()方法
```
 public static void loop() {
   //前面省略了方法
    for (;;) {
            //这样MessageQueue.next() 方法,就不断的取出Message
            Message msg = queue.next(); // might block
            msg.target.dispatchMessage(msg);
   //省略了方法
   }
```

Handle system messages here. 这样就把消息分发下去了！
```
    /**
     * Handle system messages here.
     */
    public void dispatchMessage(Message msg) {
        // TODO: 2018/5/17
        //这个callback呢，即使他妈的一个线程
        if (msg.callback != null) {
            handleCallback(msg);
        } else {
            //两个都没有的话，就去把这个消息发送到handleMessage中去
            if (mCallback != null) {
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            handleMessage(msg);
        }
    }
```
![Handler机制.png](https://upload-images.jianshu.io/upload_images/5363507-366abc390f851a73.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 说明几点 
  * 1、 安卓的程序的入口的函数是ActivityThread.main()，反正每个App启动都会经过它，具体为啥，我也不清楚
  * 2、首先初始化的是Looper，Looper的构造方法初始化MessageQueue，然后ThreadLocal.set()方法保存，原来是ThreadLocal里面有个ThreadLocalMap容器，底层的原理和HashMap差不多，有个初始长度为16的Entry数组，也有扩容机制
```
private Looper(boolean quitAllowed) {
        mQueue = new MessageQueue(quitAllowed);
        mThread = Thread.currentThread();
    }
```

   * 3、由于Entry继承的是一个WeakReference类，那么是弱应用的子类，当内存不足，扫描到这里，就不被回收，导致App被kill，系统回到Launch，（当然这仅仅是我的假设，不正确），安卓系统不会把内置的软件给kill，不如说时间，主题，launch，如果要杀，就只能杀开发者的应用了
  * 4、在ActivityThread.main()后，有个Looper.loop(),可以得出在：子线程中创建handler的话，需要looper也要准备好 ，要不然会报错。这就是为什么在子线程里面创建Handler如果不手动创建和启动Looper会报错的原因
```
   final Looper me = myLooper();
        if (me == null) {
            // TODO: 2018/5/17   在子线程中创建handler的话，需要looper也要准备好 ，要不然会报错
            // 1、 首先拿到Looper对象（me），如果当前的线程没有Looper，那么就会抛出异常，
            // 这就是为什么在子线程里面创建Handler如果不手动创建和启动Looper会报错的原因
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
```
* 5、程序能够不断的运行着的原因，是Looper.loop中是一个死循环，当消息队列没有消息了，程序就会退出
* 6、消息的分发msg.target.dispatchMessage(msg); msg.target其实就是Handler对象，可以看到分发消息的最终结果，也可以从这里表明Message的使用有好多种可能。
```
    public void dispatchMessage(Message msg) {
        // TODO: 2018/5/17
        //这个callback呢，即使他妈的一个线程
        if (msg.callback != null) {
            handleCallback(msg);
        } else {
            //两个都没有的话，就去把这个消息发送到handleMessage中去
            if (mCallback != null) {
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            handleMessage(msg);
        }
    }
```
* 7、Message中，目前是我见过享元模式最好的实现的方式。更多的Demo可以看这篇文章[二十三种设计模式](https://www.jianshu.com/p/4e01479b6a2c)
* 8、为啥没有消息回收，因为谷歌工程师，已经帮我们做了。致敬谷歌
* 9、MessageQueue.next()，使用的是本地方法，因为效率的问题，需要更高的效率，所以需要本地，原理说实话，我想扯一下，看了好多文档，发现我自己也不明白，反正不是我们常规的队列中取出，而是根据when时间参数有关。
* 10、Handler，发送消息的姿势很多（注意是姿势），需要不断的尝试，在集合源码，就可以发现新大陆
* 11、后续会讲到View的绘制啊 `RESUME_ACTIVITY`activity获取焦点，底层通过的也是Handler，在ActivityThread 内部类H 继承的是Handler，这里也是View绘制的开始，后续会写一篇文章分析。
```
 private class H extends Handler{
        public void handleMessage(Message msg) {
            case RESUME_ACTIVITY:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "activityResume");
                    SomeArgs args = (SomeArgs) msg.obj;
                    handleResumeActivity((IBinder) args.arg1, true, args.argi1 != 0, true,
          }
}
```
* 12、退出程序其实就是 `mLooper.myLooper().quit();`
```
                case EXIT_APPLICATION:
                    if (mInitialApplication != null) {
                        mInitialApplication.onTerminate();
                    }
                    //退出Looper的循环 这里实际上是调用了MessageQueue的quit，清空所有Message。
                    mLooper.myLooper().quit();
                    break;
```
* 13、`handler`正确的使用的姿势，可以看这篇文章[安卓代码、图片、布局、网络和电量优化](https://www.jianshu.com/p/82b76e0cb41e)



