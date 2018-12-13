---
title: Hement：项目中的三方依赖（四）
date: 2018-12-13 17:05:20
tags: 依赖
---
* **欢迎关注我的公众号**
![公众号](https://upload-images.jianshu.io/upload_images/5363507-0a0cf2e5fd8f843d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<!--  more  -->
 *  **GitHub地址**：[Hement](https://github.com/Shimingli/Hement):持续更新中
 * **由于使用Dagger2的未知性，所以创建一个分支没有使用Dagger2**  [Hement](https://github.com/Shimingli/Hement/tree/No_Dagger2) 

####   1、**[RxPermissions](https://github.com/tbruyelle/RxPermissions)** 版本0.10.2
* 使用方式
* 初始化
```
  RxPermissions      mRxPermissions = new RxPermissions(this);
```
* 请求两个权限的代码
```
   //连续获取两个权限以上
        mRxPermissions
                .requestEach(Manifest.permission.CALL_PHONE,
                        Manifest.permission.BLUETOOTH)
                .subscribe(new Consumer<Permission>() {
                               @Override
                               public void accept(Permission permission) throws Exception {
                                   if (permission.granted) {
                                       // 获取了权限
                                       Toast.makeText(RxPermissionsActivity.this,
                                               "获取两个权限",
                                               Toast.LENGTH_SHORT).show();
                                   } else if (permission.shouldShowRequestPermissionRationale) {
                                       //没有获取权限，但是用户没有点不在询问
                                       Toast.makeText(RxPermissionsActivity.this,
                                               "权限拒绝，但是没有点不在询问",
                                               Toast.LENGTH_SHORT).show();
                                   } else {
                                       //用户已经点了不在询问，需要去启动设置开启权限
                                       Toast.makeText(RxPermissionsActivity.this,
                                               "权限拒绝，并且不能询问",
                                               Toast.LENGTH_SHORT).show();
                                   }
                               }
                           },
                        new Consumer<Throwable>() {
                            @Override
                            public void accept(Throwable t) {
                                Timber.tag(getClassName()).i("发生异常" + t);
                            }
                        },
                        new Action() {
                            @Override
                            public void run() {
                                Timber.tag(getClassName()).i("完成");
                            }
                        });

```



####  2、**[RxBinding](https://github.com/JakeWharton/RxBinding)** 版本2.1.1
* RXJava绑定API用于Android UI小部件从平台和支持库。
* 引入此依赖主要是演示`RxPermissions`的使用,我在实际工程中，没有使用过
```
    //使用RxView的简单的例子
        disposable = RxView.clicks(findViewById(R.id.enableCamera))
                // 单击按钮时请求权限
                .compose(mRxPermissions.ensureEach(Manifest.permission.CAMERA))
                .subscribe(new Consumer<Permission>() {
                               @Override
                               public void accept(Permission permission) {
                                   Timber.tag(getClassName()).i("Permission result " + permission);
                                   if (permission.granted) {
                                       releaseCamera();
                                       camera = Camera.open(0);
                                       try {
                                           camera.setPreviewDisplay(surfaceView.getHolder());
                                           camera.startPreview();
                                       } catch (IOException e) {
                                           Timber.tag(getClassName()).i("IOException result " + e);
                                       }
                                   } else if (permission.shouldShowRequestPermissionRationale) {
                                       Toast.makeText(RxPermissionsActivity.this,
                                               "权限拒绝，但是没有点不在询问",
                                               Toast.LENGTH_SHORT).show();
                                   } else {
                                       Toast.makeText(RxPermissionsActivity.this,
                                               "用户已经点了不在询问，需要去启动设置开启权限",
                                               Toast.LENGTH_SHORT).show();
                                   }
                               }
                           },
                        new Consumer<Throwable>() {
                            @Override
                            public void accept(Throwable t) {
                                Timber.tag(getClassName()).i("发生异常" + t);
                            }
                        },
                        new Action() {
                            @Override
                            public void run() {
                                Timber.tag(getClassName()).i("完成");
                            }
                        });

```
####  3、**[dagger](https://github.com/google/dagger)**版本2.15
* 具体使用请看此文章 ： [Hement：关于项目中的Dagger2的使用（三）](https://www.jianshu.com/p/6c189567cb43)

####  4、**[timber](https://github.com/JakeWharton/timber)** 版本4.6.1

*   使用方法 
* gradle的配置
```
 buildTypes {
        // timber 的配置  debuggable
        release {
            minifyEnabled false
            debuggable  false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
        debug {
            debuggable true
        }
    }
```
* HementApplication的配置
```
 @Override
    public void onCreate() {
        super.onCreate();
        if (BuildConfig.DEBUG) {
            //也可以设置log一直开
            Timber.plant(new DebugTree());
        } else {
            //上线的话  就关闭有些不必要的日记输出
            Timber.plant(new CrashReportingTree());

        }
        QMUISwipeBackActivityManager.init(this);
        //打印tag为类名
        Timber.v("---HementApplication ---");

    }
```

* CrashReportingTree 类的详情
```
    private static final class CrashReportingTree extends Timber.Tree {
        
        @Override
        protected boolean isLoggable(@Nullable String tag, int priority) {
            return priority >=INFO;
    }

        @Override
        protected void log(int priority, @org.jetbrains.annotations.Nullable String tag, @NotNull String message, @org.jetbrains.annotations.Nullable Throwable t) {
            if (priority == Log.VERBOSE || priority == Log.DEBUG) {
                return;
            }
            FakeCrashLibrary.log(priority, tag, message);
            if (t != null) {
                if (priority == Log.ERROR) {
                    FakeCrashLibrary.logError(t);
                } else if (priority == Log.WARN) {
                    FakeCrashLibrary.logWarning(t);
                }
            }

        }
    }

```
* 我个人对Timber的理解就是 非常的简单，也非常的方便控制那些日志上线和Debug中不可以输出，虽然我们自己也可以做到，但是Timer还有个其他的功能，可以和 `%d` `%s` 一起使用

* `%d`后面接上Long Int等类型   `%s`后面接上字符串类型,在Hement的项目中，我是这样封装使用的
* 在BaseActivity 中获取这个类名
 ```
   protected String getClassName(){
        return this.getClass().getSimpleName();
    }
```
```
       Timber.tag(getClassName()).i("创建新的configPersistentComponent id=%d",mActivityId);

        Timber.tag(getClassName()).i("mMainPresenter   =%s",mMainPresenter);
```
* 这样就可以很方便的看出是那个类的输出日志，当然git上有个日记类可以通过日记输出点击直接跳转到类里面，非常牛逼，但是我个人觉得自己使用的最舒服最重要，况且那个日记类有30k左右代码！有点不值得！
![image.png](https://upload-images.jianshu.io/upload_images/5363507-0f780a12ce06a7f1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 5、数据库**[sqlbrite](https://github.com/square/sqlbrite)** 版本2.0.0
* 数据库 `sqlbrite`对 `Android` 系统的`SQLiteOpenHelper`和 `ContentResolver` 的轻量级封装,配合`Rxjava2`使用。
* 一个响应式数据库框架`sqlbrite`,完美解决数据库和`UI`的同步更新！
```
  // 数据库
  api 'com.squareup.sqlbrite3:sqlbrite:3.2.0'
```
* 使用
```
public class DB {

    public DB() { }

    public abstract static class HementTable {
        public static final String TABLE_NAME = "hement";
        public static final String COLUMN_TITLE = "title";
        public static final String COLUMN_DATE = "date";
        public static final String COLUMN_DAY = "day";

        // 注意创建表的结构是否正确
        public static final String CREATE =
                "CREATE TABLE " + TABLE_NAME + " (" +
                        COLUMN_TITLE + " TEXT NOT NULL, " +
                        COLUMN_DATE + " TEXT NOT NULL, " +
                        COLUMN_DAY + " TEXT NOT NULL" +
                " ); ";
        public static ContentValues toContentValues(TodayBean ribot) {
            ContentValues values = new ContentValues();
            values.put(COLUMN_TITLE, ribot.title);
            values.put(COLUMN_DATE, ribot.date);
            values.put(COLUMN_DAY, ribot.day);
            return values;
        }
        public static TodayBean parseCursor(Cursor cursor) {
            String title = cursor.getString(cursor.getColumnIndexOrThrow(COLUMN_TITLE));
            String date = cursor.getString(cursor.getColumnIndexOrThrow(COLUMN_DATE));
            String day = cursor.getString(cursor.getColumnIndexOrThrow(COLUMN_DAY));
            TodayBean bean = new TodayBean();
            bean.setDate(date);
            bean.setTitle(title);
            bean.setDay(day);
            return bean;
        }
    }

```
* 关于`ContentValues ` :该类用于存储处理的一组值，底层原理就是HashMap。`ContentResolver`能够去处理，等于他们结合使用
* DbOpenHelper类
```
@Singleton
public class DbOpenHelper extends SQLiteOpenHelper {

    public static final String DATABASE_NAME = "hement.db";
    public static final int DATABASE_VERSION = 1;

    @Inject
    public DbOpenHelper(@ApplicationContext Context context) {
        super(context, DATABASE_NAME, null, DATABASE_VERSION);
    }

    @Override
    public void onConfigure(SQLiteDatabase db) {
        super.onConfigure(db);
    }

    @Override
    public void onCreate(SQLiteDatabase db) {
        db.beginTransaction();
        try {
            db.execSQL(DB.HementTable.CREATE);
            db.setTransactionSuccessful();
        } finally {
            db.endTransaction();
        }
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) { }

```

* 关于DatabaseHelper类
```
@Singleton
public class DatabaseHelper {

    private final BriteDatabase mDb;

    @Inject
    public DatabaseHelper(DbOpenHelper dbOpenHelper) {
        this(dbOpenHelper, Schedulers.io());
    }

    public DatabaseHelper(DbOpenHelper dbOpenHelper, Scheduler scheduler) {
        SqlBrite.Builder briteBuilder = new SqlBrite.Builder();
        mDb = briteBuilder.build().wrapDatabaseHelper(dbOpenHelper, scheduler);
    }
    public BriteDatabase getDb() {
        return mDb;
    }

    /**
     * 保存数据库中的数据，这个数据的来源是网络请求
     * @param newRibots
     * @return
     */
    public Observable<TodayBean> setDBData(final List<TodayBean> newRibots) {
        return Observable.create(new ObservableOnSubscribe<TodayBean>() {
            @Override
            public void subscribe(ObservableEmitter<TodayBean> emitter) throws Exception {
                if (emitter.isDisposed()) return;
                BriteDatabase.Transaction transaction = mDb.newTransaction();
                try {
                    mDb.delete(DB.HementTable.TABLE_NAME, null);
                    for (TodayBean ribot : newRibots) {
                        long result = mDb.insert(DB.HementTable.TABLE_NAME,
                                DB.HementTable.toContentValues(ribot),
                                SQLiteDatabase.CONFLICT_REPLACE);
                        if (result >= 0) emitter.onNext(ribot);
                    }
                    transaction.markSuccessful();
                    emitter.onComplete();
                } finally {
                    transaction.end();
                }
            }
        });
    }

    /**
     * 获取数据库中的数据
     * @return
     */
    public Observable<List<TodayBean>> getDBData() {
        return mDb.createQuery(DB.HementTable.TABLE_NAME,
                "SELECT * FROM " + DB.HementTable.TABLE_NAME)
                .mapToList(new Function<Cursor, TodayBean>() {
                    @Override
                    public TodayBean apply(@NonNull Cursor cursor) throws Exception {
                        return DB.HementTable.parseCursor(cursor);
                    }
                });
    }
}

```

#### 6、 **[glide](https://github.com/bumptech/glide)**版本4.7.1
* 请看这里：[基于Glide4.7.1二次封装](https://www.jianshu.com/p/aecd92515cea)


#### 7、网络框架的依赖
 **retrofit converter-gson  adapter-rxjava2 rxandroid  rxjava logging-interceptor**
* 请看这里：[Hement：MVP架构中的网络框架(RxJava2+Retrofit2+RxAndroid)（二）](https://www.jianshu.com/p/e46cace343a4)

#### 9、 **[QMUI_Android](https://github.com/Tencent/QMUI_Android)** 项目中的基本UI库
* `QMUI Android` 的设计目的是用于辅助快速搭建一个具备基本设计还原效果的` Android `项目，同时利用自身提供的丰富控件及兼容处理，让开发者能专注于业务需求而无需耗费精力在基础代码的设计上。不管是新项目的创建，或是已有项目的维护，均可使开发效率和项目质量得到大幅度提升。


* **功能特性**
  *  全局 UI 配置
  只需要修改一份配置表就可以调整 App 的全局样式，包括组件颜色、导航栏、对话框、列表等。一处修改，全局生效。
  * 丰富的 UI 控件
   提供丰富常用的 UI 控件，例如 BottomSheet、Tab、圆角 ImageView、下拉刷新等，使用方便灵活，并且支持自定义控件的样式。
   *  高效的工具方法
    提供高效的工具方法，包括设备信息、屏幕信息、键盘管理、状态栏管理等，可以解决各种常见场景并大幅度提升开发效率。

* **在此感谢QMUI的小伙伴无私贡献** 

* 未完待续 [下一篇文章]()
 *  **GitHub地址**：[Hement](https://github.com/Shimingli/Hement):持续更新中
 * **由于使用Dagger2的未知性，所以创建一个分支没有使用Dagger2**  [Hement](https://github.com/Shimingli/Hement/tree/No_Dagger2) 




* **感谢一下博客对我的帮助**
   * [Android SqlBrite使用介绍和官方demo详解](https://blog.csdn.net/niubitianping/article/details/62226099)
