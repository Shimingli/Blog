---
title: Fragment.setNextAnim(int) on a null object reference
date: 2017-08-14 16:33:20
tags: Fragment.setNextAnim(int) on a null object reference
---
#### 产生的原因：java.lang.NullPointerException: Attempt to invoke virtual method 'void android.support.v4.app.Fragment.setNextAnim(int)' on a null object reference
虽然知道是这个原因，但是呢？我们代码中并没有对MainActivity中的Fragment进入和出入动画做设置，可以这样说，这段代码是谷歌官方的，但是它抛异常
<!--  more  -->
![image.png](http://upload-images.jianshu.io/upload_images/5363507-7c4adab74fa7b3df.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](http://upload-images.jianshu.io/upload_images/5363507-1dc01a163b603d8f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
####影响的结果，app奔溃率的90%的原因都是这个原因。
![image.png](http://upload-images.jianshu.io/upload_images/5363507-2a1bfe270abd8678.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](http://upload-images.jianshu.io/upload_images/5363507-c4d5706681621beb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###定位问题
[Android N 指纹识别 NullPointerException: Attempt toFragment.mNextAnim 的错误](http://blog.csdn.net/easyhood/article/details/53446830) 在这个博客下作者一句话：已录制指纹，关闭指纹后重新开启指纹识别，设置停止运行，抛出异常为
```
java.lang.NullPointerException: Attempt to write to field 'int android.app.Fragment.mNextAnim' on a null object reference
```
虽然和我们的程序中抛出的异常不一样，但是也是出入动画的为null，解释的原因为：fragment为空的时候hide remove 或是show了，但是这毕竟是谷歌大神写的源码，毫无为空的地方，因为它也不显示到底第几行报错
```
Like commit() but allows the commit to be executed after an activity’s state is saved. This is dangerous because the commit can be lost if the activity needs to later be restored from its state, so this should only be used for cases where it is okay for the UI state to change unexpectedly on the user.
```

大致意思是说我使用的 commit方法是在Activity的onSaveInstanceState()之后调用的，这样会出错，因为
onSaveInstanceState方法是在该Activity即将被销毁前调用，来保存Activity数据的如果在保存完状态后再给它添加Fragment就会出错。
到这里我大概明白什么原因了，就是当onSaveInstanceState（）调用以后回来再次commit fragment的话，会导致这个bug的产生，所以在这里入手解决问题！
###模拟问题的出现
android3.0之前：onResume() -- [optional]onSaveInstanceState() -- onPause(),即调用onPause()之前，可能调用onSaveInstanceState()
android3.0之后：onResume() -- onPause() -- [optional]onSaveInstanceState() -- onStop(),即调用onStop()之前，可能调用onSaveInstanceState()
就是由于这个可能，花费了很多时间，由于onSaveInstanceState()不是必然出现，所以要模拟出来，开始我的模拟之路！
####1、模拟横竖屏的情况（这种情况简单，但是由于项目工程中，在横竖屏上有很多null异常，导致自己修复的时候，很多代码需要注释掉，反而更加麻烦，最后没有完美呈现横屏情况下app的状态分析）
####2、模拟MainActivity异常关闭的情况，同时App正常的运行下来，在失去焦点了，2s后开启一个新的Activity同时关闭当前的Activity，但是呢？app不会异常，只不过在栈内最下的位置不是MainActivity了，当返回的时候，就直接退出程序了。
![5FXVLX)A{_09PM8`26B7ZTI.png](http://upload-images.jianshu.io/upload_images/5363507-753791c6434863cd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

####3、仔细研究到底合适会触发Activity的 onSaveInstanceState() 和 onRestoreInstanceState()；
Activity的 onSaveInstanceState() 和 onRestoreInstanceState()并不是生命周期方法，它们不同于 onCreate()、onPause()等生命周期方法，它们并不一定会被触发。当应用遇到意外情况（如：内存不足、用户直接按Home键）由系统销毁一个Activity时，onSaveInstanceState() 会被调用。但是当用户主动去销毁一个Activity时，例如在应用中按返回键，onSaveInstanceState()就不会被调用。因为在这种情况下，用户的行为决定了不需要保存Activity的状态。通常onSaveInstanceState()只适合用于保存一些临时性的状态，而onPause()适合用于数据的持久化保存。
onSaveInstanceState()方法会在什么时候被执行：
   　　(1)、当用户按下HOME键时。
　　这是显而易见的，系统不知道你按下HOME后要运行多少其他的程序，自然也不知道activity A是否会被销毁，因此系统会调用onSaveInstanceState()，让用户有机会保存某些非永久性的数据。以下几种情况的分析都遵循该原则
　　(2)、长按HOME键，选择运行其他的程序时。
　　(3)、按下电源按键（关闭屏幕显示）时。
　　(4)、从activity A中启动一个新的activity时。
　　(5)、屏幕方向切换时，例如从竖屏切换到横屏时。
##更具上面的结论，其实我在不断测试中已经在不断的走onSaveInstanceState()，只不过我自己以为没有走这个方法。但是测试过程中， Fragment.setNextAnim(int)这个bug产生的几率很小，可以忽略不计。所以这里可以得出结论：当onSaveInstanceState（）调用以后回来再次commit fragment的话，不会导致这个bug的产生，那么以上的全部都过程，都是不成立的


#那到底是什么样的问题导致Fragment.setNextAnim(int) null异常的呢？看下Acitivty的生命周期，当内存不足的时候，app能在后台被后台回收掉，但是Activity并没有被销毁掉，还是存在栈内中，只不过会走onCreate（）方法。ok！到这里就可以定位了出问题出现的点
![image.png](http://upload-images.jianshu.io/upload_images/5363507-192b3dffa53b523d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##再次试图重现问题，这次聪明了，直接哪一个内存比较小的手机，不断的启动手机里其他的APP，同时也要启动目标App，然他的进程一直在前台进程和非前台进程中切换，需要注意的是，当启动一个比较耗内存的app比如地图，游戏之类的，会把app直接回收掉，这不是bug，这是安卓系统的原因，同时还需的注意点，在这里我们关心的MainActivity被回收了，所以建议把MainActivity的位置置于在栈内第三个或者是第四个位置，一切准备就绪，开始测试（主要看人品，我个人在测试3次左右就能出现）

![image.png](http://upload-images.jianshu.io/upload_images/5363507-37abf23257d293af.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
哦原来是，这样子fragment为null，导致了java.lang.NullPointerException: Attempt to invoke virtual method 'void android.support.v4.app.Fragment.setNextAnim(int)' on a null object reference
###来个手动的测试的结果，如图
![image.png](http://upload-images.jianshu.io/upload_images/5363507-ca95680aa5370125.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#总结：当Activty别系统紧急回收了，没有被销毁，重新走到onCreate（）的方法中了，初始化Fragment的时候，导致fragment的变量为null，就会引起如下bug
```
java.lang.RuntimeException:Unable to resume activity {cn.changniannian.broodon/cn.changniannian.broodon.view.navigation.main.MainActivity}: java.lang.NullPointerException: Attempt to invoke virtual method 'void android.support.v4.app.Fragment.setNextAnim(int)' on a null object reference
2 android.app.ActivityThread.performResumeActivity(ActivityThread.java:3030)
3 ......
4 Caused by:
5 java.lang.NullPointerException:Attempt to invoke virtual method 'void android.support.v4.app.Fragment.setNextAnim(int)' on a null object reference
6 android.support.v4.app.BackStackRecord.executeOps(BackStackRecord.java:768)
7 android.support.v4.app.FragmentManagerImpl.executeOps(FragmentManager.java:2415)
8 android.support.v4.app.FragmentManagerImpl.executeOpsTogether(FragmentManager.java:2200)
9 android.support.v4.app.FragmentManagerImpl.optimizeAndExecuteOps(FragmentManager.java:2153)
10 android.support.v4.app.FragmentManagerImpl.execPendingActions(FragmentManager.java:2063)
11 android.support.v4.app.FragmentController.execPendingActions(FragmentController.java:388)
12 android.support.v4.app.FragmentActivity.onResume(FragmentActivity.java:440)
13 com.broodon.core.base.BaseActivity.onResume(BaseActivity.java:137)
14 cn.changniannian.broodon.view.navigation.main.MainActivity.onResume(MainActivity.java:525)
15 android.app.Instrumentation.callActivityOnResume(Instrumentation.java:1243)
16 android.app.Activity.performResume(Activity.java:6111)
17 android.app.Activity.performResume(Activity.java:6313)
18 android.app.ActivityThread.performResumeActivity(ActivityThread.java:3012)
19 android.app.ActivityThread.handleResumeActivity(ActivityThread.java:3061)
20 android.app.ActivityThread$H.handleMessage(ActivityThread.java:1375)
21 android.os.Handler.dispatchMessage(Handler.java:102)
22 android.os.Looper.loop(Looper.java:135)
23 android.app.ActivityThread.main(ActivityThread.java:5318)
24 java.lang.reflect.Method.invoke(Native Method)
25 java.lang.reflect.Method.invoke(Method.java:372)
26 com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:922)
27 com.android.internal.os.ZygoteInit.main(ZygoteInit.java:717)
```
![image.png](http://upload-images.jianshu.io/upload_images/5363507-f2a0addeddb761f8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)