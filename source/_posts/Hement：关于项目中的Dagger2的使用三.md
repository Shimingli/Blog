---
title: Hement：关于项目中的Dagger2的使用三
date: 2018-12-07 15:23:32
tags: Dagger2
---


* 写在前面的话，要讲好这个`Dagger2`真的不是一件简单的事情
* `Dagger 1` :匕首 一个用于Android和Java的快速依赖注入。由`SQUAR`公司开发
* `Dagger 2`:由谷歌公司接手开发 [Dagger 2](https://github.com/google/dagger)

#### 依赖注入的原理 
* 首先记住：new 创建一个对象是有毒的；
* 首先什么是依赖注入的原理：在软件工程领域中，依赖注入是用于实现控制反转的最常见的方式之一
* 引用`Martin Flower`在解释介绍注入时使用的一部分代码来说明这个问题

<!--  more  -->
```
public class MovieLister {
    private MovieFinder finder;
    public MovieLister() {
        finder = new MovieFinderImpl();
    }
    public Movie[] moviesDirectedBy(String arg) {
        List allMovies = finder.findAll();
        for (Iterator it = allMovies.iterator(); it.hasNext();) {
            Movie movie = (Movie) it.next();
            if (!movie.getDirector().equals(arg)) it.remove();
        }
        return (Movie[]) allMovies.toArray(new Movie[allMovies.size()]);
    }
    ...
}
```
* MovieFinder 
```
public interface MovieFinder {
    List findAll();
}
```
* 创建类MovieFinder 来提供需要的电影列表，moviesDirectedBy方法提供导演的名称来搜索电影，真正负责搜索电影的是MovieFinderImpl类，目前看起来情况不错，但是当我们需要修改MovieFinderImpl的实现的方法，增加一个参数表明Movie的数据来源是那个数据库，我们不仅仅需要修改MovieFinderImpl类中的实现的代码，同时也要修改MovieLister类中创建的MovieFinderImpl对象。
* 这就需要依赖注入要处理耦合，这种在MovieLister中创建MoviefinderImpl的方式，是的MovieLister不仅仅依赖于MovieFinder这个接口，它还依赖于MovieListImpl这个实现，这种在一个类创建两位一个类的对象的代码，和硬编码以及硬编码数字一样，是一种导致耦合的坏味道，可以把这种坏味道称为硬初始化，我们也该要像记住硬编码一样，new 创建一个对象是有毒的。
* **以上带来的坏处有两点**
* 1、修改其实现的时候，也会修改创建地方的代码
* 2、以上创建代码的方法不便于测试，也会导致代码的可读性变差（如果一段代码不便于测试，那么它一定不便于阅读），当然MVP这种模式也是解决这种问题的！
* **这个时候就需要依赖注入了** 
* 依赖注入的三种方式
 *  **1、构造函数的注入**
```
public class MovieLister {
    private MovieFinder finder;
    public MovieLister(MovieFinder finder) {
        this.finder = finder;
    }
    ...
}
```
 *  **2、set注入**
```
public class MovieLister {
    s...
    public void setFinder(MovieFinder finder) {
        this.finder = finder;
    }
}
```
 *  **3、接口注入**
* 创建一个注入的接口
```
public interface InjectFinder {
    void injectFinder(MovieFinder finder);
}
```
* 之后实现这个接口
```
class MovieLister implements InjectFinder {
    ...
    public void injectFinder(MovieFinder finder) {
      this.finder = finder;
    }
    ...
}
```

* **依赖注入降低了依赖和被依赖类型的耦合，在修改被依赖的类型实现时候，不需要修改依赖类型的实现，同时对于依赖类型的测试更加方便**

#### Hement中Dagger2的使用
* 引入依赖
```
   //dagger2
    api "com.google.dagger:dagger:2.15"
    compileOnly 'org.glassfish:javax.annotation:10.0-b28'
```

* **1、构建依赖**：Dagger2中，这个负责提供依赖的组件被称为Module，@Module标识类型为module，并用@Provides标识提供依赖的方法。
* ActivityModule
```
@Module
public class ActivityModule {
    private Activity mActivity;

    public ActivityModule(Activity activity) {
        mActivity = activity;
    }

    @Provides
    Activity provideActivity() {
        return mActivity;
    }

    @Provides
    @ActivityContext
    Context providesContext() {
        return mActivity;
    }
}

```
* ApplicationModule:提供远程Server，说通俗就是接口`IRemoteServer`,通过Retrofit得到，这个有个坏处，就是所有的接口请求信息都放在一个类了，后续应该根据项目的壮大，需要增加多个`Server`
```

@Module
public class ApplicationModule {

    protected final HementApplication mApplication;

    public ApplicationModule(HementApplication application) {
        mApplication = application;
    }

    @Provides
    Application provideApplication() {
        return mApplication;
    }

    @Provides
    @ApplicationContext
    Context provideContext() {
        return mApplication;
    }


    @Provides
    @Singleton
    IRemoteServer provideRibotsService() {
        return IRemoteServer.Creator.newHementService();
    }
}
```

* **2、构建Injector**
* 有了提供依赖的组件，还需要把依赖注入到需要的对象中，连接提供依赖和消费依赖对象的组件被称为Injector. Dagger2中，我们将其称为`component`。`ActivityComponent`代码如下：
```
@PerActivity
@Subcomponent(modules = ActivityModule.class)
public interface ActivityComponent {

    /**
     * 注入activity
     * @param mainActivity
     */
    void inject(MainActivity mainActivity);

    /**
     * 每一个类都得单独的注入
     * @param baseActivity
     */
    void inject(NetWorkActivity baseActivity);

    void inject(SPreferencesActivity sPreferencesActivity);

    void inject(DBNetWorkDemoActivity dbDemoActivity);

    void inject(RxEventBusActivity rxEventBusActivity);

    void inject(RxPermissionsActivity rxPermissionsActivity);

    void inject(ImageLoaderActivity imageLoaderActivity);
}
```
* 注入的类或者对象全部是`null`的问题？
* 原因是必须是真正消耗依赖的类型`MainActivity`，而不可以写成其父类，比如`Activity`。因为`Dagger2`在编译时生成依赖注入的代码，会到`inject`方法的参数类型中寻找可以注入的对象，但是实际上这些对象存在于`MainActivity`，而不是`Activity`中。如果函数声明参数为`Activity`，`Dagger2`会认为没有需要注入的对象。当真正在`MainActivity`中创建`Component`实例进行注入时，会直接执行按照Activity作为参数生成的inject方法，导致所有注入都失败。
* 同时提供了ApplicationComponent 来初始化和应用生命周期一样的类
```
@Singleton
@Component(modules = ApplicationModule.class)
public interface ApplicationComponent {

    @ApplicationContext
    Context context();
    Application application();

    IRemoteServer remoteServer();

    PreferencesHelper preferencesHelper();

    DatabaseHelper databaseHelper();

    DataManager dataManager();

    RxEventBus eventBus();
}

```

* **3、关于Scope和Qualifier的使用**
*  Scope 的用法，@Scope是元注解，是用来标注自定义注解的
```
@Scope //  Scope 的用法，@Scope是元注解，是用来标注自定义注解的
@Retention(RetentionPolicy.RUNTIME)
public  @interface ConfigPersistent {
}
```
* 如果使用了`ConfigPersistent`注解的话,可以复用之前的依赖实例,在Hement中我使用在了`NetWorkPresenter`类中
```
@ConfigPersistent
public class NetWorkPresenter  extends BasePresenter<NetWorkView> {
    .....
}
```
* 同时为了得到`Activity`的复用,定义ConfigPersistentComponent：意思就是持久的Component，和App的生命周期一样
```
@ConfigPersistent
@Component(dependencies = ApplicationComponent.class)
public interface ConfigPersistentComponent {

    ActivityComponent activityComponent(ActivityModule activityModule);
}
```
* dependencies 依赖关系，一个 Component 依赖其他 Compoent 公开的依赖实例，用 Component 中的dependencies声明。
* Component 和Subcomponent继承关系，一个 Component 继承（也可以叫扩展）某 Component 提供更多的依赖，SubComponent 就是继承关系的体现。
* **@Qualifier** 限定符,ApplicationContext 标识是Application的Context对象
```
@Qualifier
@Retention(RetentionPolicy.RUNTIME)
public @interface ApplicationContext {
}
```
* 在需要使用的地方，加上这个注解即可,比如是在`DbOpenHelper `中。
```
@Singleton
public class DbOpenHelper extends SQLiteOpenHelper {

    public static final String DATABASE_NAME = "hement.db";
    public static final int DATABASE_VERSION = 1;

    @Inject
    public DbOpenHelper(@ApplicationContext Context context) {
        super(context, DATABASE_NAME, null, DATABASE_VERSION);
    }
}
```

#### Hement中`BaseActivity、BaseFragment`关于`Dagger2`的封装

* 在这里讲一种在`BaseActivity`中的封装，Hement中定义了一个持久的`ConfigPersistentComponent`,在`BaseActivity`定义一个`LongSparseArray`  具体原理请看 [常用集合的原理分析](https://www.jianshu.com/p/a5f638bafd3b )
  *  LongSparseArray是android里为<Long,Object> 这样的Hashmap而专门写的类,目的是提高效率，其核心是折半查找函数（binarySearch）。 SparseArray仅仅提高内存效率，而不是提高执行效率
 所以也决定它只适用于android系统（内存对android项目有多重要）SparseArray不需要开辟内存空间来额外存储外部映射，从而节省内存。
```
    private static final LongSparseArray<ConfigPersistentComponent> sComponentsMap = new LongSparseArray<>();
   @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        //创建ActivityComponent，如果配置更改后调用缓存的ConfigPersistentComponent，则重用它。
        mActivityId = savedInstanceState != null ? savedInstanceState.getLong(KEY_ACTIVITY_ID) : NEXT_ID.getAndIncrement();

        ConfigPersistentComponent configPersistentComponent = sComponentsMap.get(mActivityId, null);
        if (null == configPersistentComponent) {
            Timber.tag(getClassName()).i("创建新的configPersistentComponent id=%d",mActivityId);
            configPersistentComponent = DaggerConfigPersistentComponent.builder()
                    .applicationComponent(HementApplication.get(this).getComponent())
                    .build();
            sComponentsMap.put(mActivityId, configPersistentComponent);
        }
        mActivityComponent = configPersistentComponent.activityComponent(new ActivityModule(this));

        //状态栏的颜色
        QMUIStatusBarHelper.setStatusBarLightMode(this);
    }

```

* 同时也定义了一个 `AtomicLong`对象：AtomicLong是作用是对长整形进行原子操作,线程安全.主要作用就是
`NEXT_ID.getAndIncrement()`获取一个自增长的Id。
```
  private static final AtomicLong NEXT_ID = new AtomicLong(0);
```
* **在java1.8中新加入了一个新的原子类LongAdder，该类也可以保证Long类型操作的原子性，相对于AtomicLong，LongAdder有着更高的性能和更好的表现，可以完全替代AtomicLong的来进行原子操作但是对 java的版本有要求，这里就不使用 LongAdder了**

* 原子递增一个当前值。
```
 NEXT_ID.getAndIncrement()
```

* 完整的`BaseActivity`中的代码
```
package com.shiming.hement.ui.base;

import android.os.Bundle;
import android.support.annotation.Nullable;
import android.support.v4.util.LongSparseArray;
import android.support.v7.app.AppCompatActivity;

import com.shiming.base.ui.QMUIActivity;
import com.shiming.base.utils.QMUIDisplayHelper;
import com.shiming.base.utils.QMUIStatusBarHelper;
import com.shiming.hement.HementApplication;
import com.shiming.hement.injection.component.ActivityComponent;
import com.shiming.hement.injection.component.ConfigPersistentComponent;
import com.shiming.hement.injection.component.DaggerConfigPersistentComponent;
import com.shiming.hement.injection.module.ActivityModule;

import java.util.concurrent.atomic.AtomicLong;

import timber.log.Timber;

import static com.shiming.base.BaseApplication.getContext;

/**
 * <p>
 *  抽象应用程序中的其他活动必须实现的活动。它处理Dagger组件的创建，并确保ConfigPersistentComponent的实例跨配置更改存活。
 * </p>
 *
 * @author shiming
 * @version v1.0
 * @since 2018/11/28 10:04
 */

public class BaseActivity extends QMUIActivity {

    private static final String KEY_ACTIVITY_ID = "KEY_ACTIVITY_ID";
    /**
     * AtomicLong是作用是对长整形进行原子操作。 线程安全
     */
    private static final AtomicLong NEXT_ID = new AtomicLong(0);
    /**
     * java1.8中新加入了一个新的原子类LongAdder，该类也可以保证Long类型操作的原子性，
     * 相对于AtomicLong，LongAdder有着更高的性能和更好的表现，可以完全替代AtomicLong的来进行原子操作
     * 但是对 java的版本有要求，这里就不使用 LongAdder了
     */
   // private static final LongAdder NEXT_ID = new LongAdder();

    /**
     * LongSparseArray是android里为<Long,Object> 这样的Hashmap而专门写的类,目的是提高效率，其核心是折半查找函数（binarySearch）。
     * SparseArray仅仅提高内存效率，而不是提高执行效率
     * ，所以也决定它只适用于android系统（内存对android项目有多重要）SparseArray不需要开辟内存空间来额外存储外部映射，从而节省内存。
      */
    // https://www.jianshu.com/p/a5f638bafd3b   常用集合的原理分析 Dagger does not support injection into private fields
    private static final LongSparseArray<ConfigPersistentComponent> sComponentsMap = new LongSparseArray<>();

    private long mActivityId;

    private ActivityComponent mActivityComponent;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        //创建ActivityComponent，如果配置更改后调用缓存的ConfigPersistentComponent，则重用它。
        mActivityId = savedInstanceState != null ? savedInstanceState.getLong(KEY_ACTIVITY_ID) : NEXT_ID.getAndIncrement();

        ConfigPersistentComponent configPersistentComponent = sComponentsMap.get(mActivityId, null);
        if (null == configPersistentComponent) {
            Timber.tag(getClassName()).i("创建新的configPersistentComponent id=%d",mActivityId);
            configPersistentComponent = DaggerConfigPersistentComponent.builder()
                    .applicationComponent(HementApplication.get(this).getComponent())
                    .build();
            sComponentsMap.put(mActivityId, configPersistentComponent);
        }
        mActivityComponent = configPersistentComponent.activityComponent(new ActivityModule(this));

        //状态栏的颜色
        QMUIStatusBarHelper.setStatusBarLightMode(this);
    }
    protected String getClassName(){
        return this.getClass().getSimpleName();
    }
    @Override
    protected void onSaveInstanceState(Bundle outState) {
        super.onSaveInstanceState(outState);
        outState.putLong(KEY_ACTIVITY_ID, mActivityId);
    }

    /**
     * isChangingConfigurations()函数在是Api level 11（Android 3.0.x） 中引入的
     * 也就是用来检测当前的Activity是否 因为Configuration的改变被销毁了，然后又使用新的Configuration来创建该Activity。
     * 常见的案例就是 Android设备的屏幕方向发生变化，比如从横屏变为竖屏。
     */
    @Override
    protected void onDestroy() {
        //检查此活动是否处于销毁过程中，以便用新配置重新创建。
        if (!isChangingConfigurations()) {
            Timber.tag(getClassName()).i("销毁的configPersistentComponent id=%d",mActivityId);
            sComponentsMap.remove(mActivityId);
        }
        super.onDestroy();
    }
    public ActivityComponent activityComponent() {
        return mActivityComponent;
    }

    @Override
    protected int backViewInitOffset() {
        return QMUIDisplayHelper.dp2px(getContext(), 100);
    }

}


```

* 未完待续 [下一篇文章]()
 *  **GitHub地址**：[Hement](https://github.com/Shimingli/Hement):持续更新中








* **最后说明几点** 
   * 如果一段代码不便于测试，那么它一定不便于阅读），当然MVP这种模式也是解决这种问题的！
   * 依赖注入只是控制反转的一种实现方式。控制反转还有一种常见的实现方式称为依赖查找。
   * dependencies 依赖关系，一个 Component 依赖其他 Compoent 公开的依赖实例，用 Component 中的dependencies声明。
  * Component 和Subcomponent继承关系，一个 Component 继承（也可以叫扩展）某 Component 提供更多的依赖，SubComponent 就是继承关系的体现。
  *  **在java1.8中新加入了一个新的原子类LongAdder，该类也可以保证Long类型操作的原子性，相对于AtomicLong，LongAdder有着更高的性能和更好的表现，可以完全替代AtomicLong的来进行原子操作但是对 java的版本有要求，这里就不使用 LongAdder了**


* **谢谢一下博客对我的帮助**
   * [Dagger 2 完全解析（一），Dagger 2 的基本使用与原理](https://www.jianshu.com/p/26d9f99ea3bb)
   * [依赖注入原理](http://codethink.me/2015/08/01/dependency-injection-theory/)
   * [Dagger 2 完全解析（二），进阶使用 Lazy、Qualifier、Scope 等](https://www.jianshu.com/p/9703a931c7e7)

