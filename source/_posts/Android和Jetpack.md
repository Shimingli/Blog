---
title: Android和Jetpack
date: 2019-07-31 09:37:11
tags:  [Android，Jetpack]   
---
* **欢迎关注我的公众号**
![公众号](https://upload-images.jianshu.io/upload_images/5363507-de80ef2503c3072c.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![狗熊镇楼](https://upload-images.jianshu.io/upload_images/5363507-f083ce7660bdceb8.gif?imageMogr2/auto-orient/strip)
<!--  more  -->

#### 什么是 Android Jetpack？Android Jetpack 是一套组件、工具和指导，可以帮助您构建出色的 Android 应用
-----
* *google爸爸*
![官方图片](https://upload-images.jianshu.io/upload_images/5363507-c653f0b091872d67.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* Architecture:体系结构
* Foundation:基础
* UI：viewhi
* Behavior：行为
* **每个组件都可以单独采用和构建，以保持向后兼容性**

>>* **Android体系结构组件非常的模块化，我们就能够选择我们想要的使用我们程序的功能集**
>>对生命周期有意识的观察者和可观察组件，允许关注点的分离，从而方便测试，提供可靠的高质量应用程序
>>* **充分利用了Koltin语言功能，使工作效率更加的高**
>>* **数据库持久性一直是Android的噩梦，而Room是个巨大的改进**
>>Room：可测试性：能够在持久层上添加测试覆盖，而不需要构建任何基础架构
>>* **借助Android Jetpack 消除了所有类型的问题**
>> 使代码更加容易的维护
>> * **所有组件都可以在旧版本使用**

-----

* **Android Jetpack 基于现代化设计做法构建，其中包括问题与可测试性分离，以及 Kotlin 集成等可以提高效率的功能。这让您可以更轻松地构建稳定、优质的应用，同时减少代码数量。虽然 Android Jetpack 的组件为了协同操作而构建（例如，生命周期感知和实时数据），但是您不必使用所有组件。您可以集成能够解决您的问题的 Android Jetpack 部分，同时保留您的应用中运行出色的部分。**


****


>* **Android Jetpack 附带五个新组件**
>>* *WorkManager alpha 版*
>> 主要来约束后台作业（需要有保障的执行）提供一站式解决的方案，消除了使用作业或者异步Adapter的架构需求，主要是结合在 `OneTimeWorkRequestBuilder` 使用,而`SeedDatabaseWorker`其实`CoroutineWorker` 就是协程
>>```
>>  val request = OneTimeWorkRequestBuilder<SeedDatabaseWorker>().build()
 >>   //将一个或多个项目排队进行后台处理。
 >>    WorkManager.getInstance(context).enqueue(request)
>>```
>>* *导航 navigation alpha 版,其实就是在res文件下创建一个navgation目录*
![image.png](https://upload-images.jianshu.io/upload_images/5363507-f543c3749705aacc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
>> 在以前Activity作为系统的应用程序入口，但是在分享数据和转场方面表现的不够灵活（我觉得还好）。这个导航组件重点就是让单Activity应用成为架构的首选，利用导航组件对Fragment的原生支持，可以获得架构组件所有的好处：生命周期和ViewModel。导航组件可以声明转场，同时在最新版的 `Android Studio 3.2`可以很直白的看到和管理导航属性
>>![image.png](https://upload-images.jianshu.io/upload_images/5363507-7d98bb39e61476da.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
>> * 个人的理解就是包裹着Fragment，同时指定了跳转的方法
>>![](https://upload-images.jianshu.io/upload_images/5363507-44fb1407efd4a79d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 >>  ![官方的Demo](https://upload-images.jianshu.io/upload_images/5363507-dfc5aceea16db6b1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 >>    * *分页 Paging*
>> 应用中呈现的数据可能非常大，这就导致加载的开销比较大，因此，避免一次下载、创建或呈现过多数据就显得非常重要。分页组件1.0.0 版让您可以轻松加载和呈现大型数据集，同时在您的 RecyclerView 中进行快速、无限滚动。它可以从本地存储和/或网络加载分页数据，并让您能够定义内容的加载方式。此组件原生支持 Room、LiveData 和 RxJava。
>> 分页库的关键组件是 [`PagedList`](https://developer.android.google.cn/reference/androidx/paging/PagedList) 类，它分块加载应用程序的数据或页面。当需要更多数据时，会被加载到已有的 `PagedList` 对象当中。 如果任何加载的数据发生更改，则会从 [`LiveData`](https://developer.android.google.cn/reference/androidx/lifecycle/LiveData) 或基于 RxJava2 的对象向可观察数据持有者发射新的 `PagedList` 实例。[`PagedList`](https://developer.android.google.cn/reference/androidx/paging/PagedList) 对象一旦生成，应用的 UI 就会展示内容，同时遵循 UI 控制器的 [生命周期](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_4_Handling_Lifecycles.md)。
>> * *切片*
>> 切片”是一种以搜索结果形式在 Google 智能助理内部显示应用界面的方式 ! 我自己写的一篇文章[Android 9 Pie 正式版总结]([https://www.jianshu.com/p/a19c40aecdd0](https://www.jianshu.com/p/a19c40aecdd0)
),有去分析这个切片的过程
>> * *Android KTX*
>> Android KTX 可以让您将类似下面所示的 Kotlin 代码：
>>![](https://upload-images.jianshu.io/upload_images/5363507-9acf5bb6c0270030.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
>>转换成如下所示的更精简的 Kotlin 代码：
>>`view.doOnPreDraw { actionToBeTriggered() }`


####  对官方的[android-sunflower](https://github.com/Shimingli/android-sunflower)的解读
* **1、入口`GardenActivity`类,明显的单Actvitity+多Fragment的应用**

```
        val binding: ActivityGardenBinding = DataBindingUtil.setContentView(this,
                R.layout.activity_garden)
        drawerLayout = binding.drawerLayout
        mLayout= binding.mLayout
         //查找给定视图ID及其包含的[NavController]
        navController = findNavController(R.id.garden_nav_fragment)
        // navController.graph  获取与此导航控制器关联的最顶层导航图。
        appBarConfiguration = AppBarConfiguration(navController.graph, drawerLayout)

        // Set up ActionBar
        setSupportActionBar(binding.toolbar)
        setupActionBarWithNavController(navController, appBarConfiguration)

        // Set up navigation menu
        binding.navigationView.setupWithNavController(navController)

```

* 在布局中最关键的布局 ,指向了 app:navGraph="@navigation/nav_garden"
```
  <fragment
                android:id="@+id/garden_nav_fragment"
                android:name="androidx.navigation.fragment.NavHostFragment"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                app:defaultNavHost="true"
                app:navGraph="@navigation/nav_garden" />
```


* **2、navigation/nav_garden,新的一个导航**
把所有的Fragment放在一起，指明了转场的动画和方式，目的的Fragment的全包名的路径，同时指定了Fragment 的layout的文件，
```
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    app:startDestination="@+id/garden_fragment">
     <!--把所有的 Fragment 定义一起啊  然后制定了第一个Fragment的Id  -->
    <fragment
        android:id="@+id/garden_fragment"
        android:name="com.google.samples.apps.sunflower.GardenFragment"
        android:label="@string/my_garden_title"
        tools:layout="@layout/fragment_garden">
         <!--destination 目的地-->
        <action
            android:id="@+id/action_garden_fragment_to_plant_detail_fragment"
            app:destination="@id/plant_detail_fragment"
            app:enterAnim="@anim/slide_in_right"
            app:exitAnim="@anim/slide_out_left"
            app:popEnterAnim="@anim/slide_in_left"
            app:popExitAnim="@anim/slide_out_right" />
    </fragment>

    <fragment
        android:id="@+id/plant_list_fragment"
        android:name="com.google.samples.apps.sunflower.PlantListFragment"
        android:label="@string/plant_list_title"
        tools:layout="@layout/fragment_plant_list">

        <action
            android:id="@+id/action_plant_list_fragment_to_plant_detail_fragment"
            app:destination="@id/plant_detail_fragment"
            app:enterAnim="@anim/slide_in_right"
            app:exitAnim="@anim/slide_out_left"
            app:popEnterAnim="@anim/slide_in_left"
            app:popExitAnim="@anim/slide_out_right" />
    </fragment>

    <fragment
        android:id="@+id/plant_detail_fragment"
        android:name="com.google.samples.apps.sunflower.PlantDetailFragment"
        android:label="@string/plant_details_title"
        tools:layout="@layout/fragment_plant_detail">
        <argument
            android:name="plantId"
            app:argType="string" />
    </fragment>

</navigation>
```

* **3、Fragment 的layout的文件**
在布局文件中指明了这个页面我需要什么的数据和数据类型
```
   <data>

        <variable
                name="hasPlantings"
                type="boolean" />

    </data>
```
同时在相对于的View上去引用这个方法，当然这个方法是自定义的，比如以下布局
`RecyclerView可以指定LinearLayoutManager，而不用去代码中使用，也可以指定item的布局`
```
 <androidx.recyclerview.widget.RecyclerView
                android:id="@+id/garden_list"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:clipToPadding="false"
                android:paddingLeft="@dimen/margin_normal"
                android:paddingRight="@dimen/margin_normal"
                app:isGone="@{!hasPlantings}"
                app:layoutManager="androidx.recyclerview.widget.LinearLayoutManager"
                tools:listitem="@layout/list_item_garden_planting"/>

        <TextView
                android:id="@+id/empty_garden"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:gravity="center"
                android:text="@string/garden_empty"
                android:textSize="24sp"
                app:isGone="@{hasPlantings}"/>
```
注意观察这个方法`app:isGone`，其实需要自己去定义，具体使用如下
```
@BindingAdapter("isGone")
fun bindIsGone(view: View, isGone: Boolean) {
    view.visibility = if (isGone) {
        View.GONE
    } else {
        View.VISIBLE
    }
}
```
方便简洁
* **4、如何自定义去赋值布局文件**
首先定义一个tip类型为String的，我先说一个坑，这是我自己创建，不是上个页面传递给的，***所以这个tip为null***
![](https://upload-images.jianshu.io/upload_images/5363507-5cc2205749412ea5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 在布局中去引用
![](https://upload-images.jianshu.io/upload_images/5363507-2168480b7b3484c3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 代码中去Binding
![](https://upload-images.jianshu.io/upload_images/5363507-80a9076a3bb9d73e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 最后的结果
![](https://upload-images.jianshu.io/upload_images/5363507-f60c261927af4520.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* ***如上面我们所说的问题 setTip为null***
![image.png](https://upload-images.jianshu.io/upload_images/5363507-355a430bbbb51cc5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

所以我们的定义方法的时候，就要告诉编译期我们的setTip有可能为null `,setTip: String?`,要不然就会报错！



* **5、在GardenFragment中代码简洁的让人难以置信**

```
class GardenFragment : Fragment() {

    private val viewModel: GardenPlantingListViewModel by viewModels {
        InjectorUtils.provideGardenPlantingListViewModelFactory(requireContext())
    }

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        val binding = FragmentGardenBinding.inflate(inflater, container, false)
        val adapter = GardenPlantingAdapter()
        binding.gardenList.adapter = adapter
        subscribeUi(adapter, binding)
        return binding.root
    }

    private fun subscribeUi(adapter: GardenPlantingAdapter, binding: FragmentGardenBinding) {
        viewModel.gardenPlantings.observe(viewLifecycleOwner) { plantings ->
            binding.hasPlantings = !plantings.isNullOrEmpty()
        }

        viewModel.plantAndGardenPlantings.observe(viewLifecycleOwner) { result ->
            if (!result.isNullOrEmpty())
                adapter.submitList(result)
        }
    }
}
```
* **6、kotlin中定义工具类非常的简单，使用object修饰就可以了**
![](https://upload-images.jianshu.io/upload_images/5363507-c4af27c3e9abfc10.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
内部调用的就用private去修饰，不是的话，就去掉这个关键字就行了

* **7、LiveData**

LiveData是一个可以在给定生命周期内观察到的数据持有者类。
* LiveData是一个可观测数据的容器类。与普通的可观测类不同，LiveData 能感知生命周期，也就意味着它合乎 activity、fragment 或 service 等其他应用组件的生命周期。因此，LiveData 确保只在这些可观测的应用组件处于活动状态的时候才会更新它们。
* **使用 LiveData 的好处**
* *确保您的 UI 和数据的状态保持一致，UI 的代码放到这些 Observer 的观测者对象中，从而让代码更稳健*
* *没有内存泄漏，观测者受 [`LifeCycle`](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.html) 的作用域限制，而且还会在与其关联的生命周期被销毁后自动清理自己。*
* *不会因为 activity 停止而导致应用崩溃，如果观测者的生命周期处于非活动状态，例如在回退栈（back stack）里的 activity，那么它就不会收到任何 LiveData 的事件。*
* *无须手动管理生命周期，UI 组件只观测相关的数据，无须手动开始或停止。LiveData 会自动处理好这一切，因为它能在观测的同时，感知到相关的生命周期状态变动。*
* *总是最新的数据，即使一个生命周期进入非活动状态，在它恢复活动状态的时候仍能接收最新的数据。例如，一个后台的 activity 在回到前台的时刻就会立即接收到最新的数据。*
* *安稳度过配置变更（configuration changes，如果一个 activity 或 fragment 因屏幕旋转等配置变更而被重建，它仍能立即接收到最新的可用的数据。*
* *分享资源，您可以使用单例模式来继承一个 [`LiveData`](https://developer.android.google.cn/reference/android/arch/lifecycle/LiveData.html) 对象来把系统服务包装起来，因而使其能够分享给整个应用*

* **8、关于项目中的单利模式实现**
在读它项目的代码的时候，我发现它的实现的方式和我实际在项目中使用的不一样，使用了Volatile关键字和同步锁。
![单利](https://upload-images.jianshu.io/upload_images/5363507-79ef851c98ea50ea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> * **volatile关键字  Java多线程的三大核心**
>>* 1、 原子性 :java原子性和数据库事务的原子性差不多，一个操作要么是全部执行成功或者是失败.
>>* 2、可见性 :现在的计算机，由于 cpu 直接从 主内存中读取数据的效率不高。所以都会对应的 cpu高速缓存，先将主内存中的数据读取到缓存中，线程修改数据之后首先更新到缓存中，之后才会更新到主内存。如果此时还没有将数据更新到主内存其他的线程此时读取就是修改之前的数据
>>volatile关键字就是用于保存内存的可见性，当线程A更新了volatite的修饰的变量的话，他会立即刷新到主线程，并且将其余缓存中该变量的值清空，导致其余线程只能去主内存读取最新的值
>>synchronized 和加锁也能保证可见性，实现原理就是在释放锁之前其余线程是访问不到这个共享变量的。但是和volatile 相比较起来开销比较大 ！
>> 但是volatile不能够替换synchronized因为volatile 不能够保证原子性 (要么执行成功或者失败，没有中间的状态)
>>* 3、顺序性
>>```
>>int a = 100 ; //1
>>int b = 200 ; //2
>>int c = a + b ; //3
>>```
>> 正常的代码的执行顺序应该是1》》2》》3。但是有时候 JVM为了提高整体的效率会进行指令重排导致执行顺序可能是 2》》1》》3。但是JVM 也不能是 什么都进行重排，是在保证最终结果和代码顺序执行结果是一致的情况下才可能会进行重排
>>java中可以使用 volatile 关键字来保证顺序性，synchronized和lock 也可以来保证有序性，和保证 原子性的方式一样，通过同一段时间只能一个线程访问来实现的
>> [常用集合的原理分析]([https://www.jianshu.com/p/a5f638bafd3b](https://www.jianshu.com/p/a5f638bafd3b))



* **9、我自己实现的单利模式:RetrofitManager**
*  在我实际的项目中，具体如下
* 私有化构造方法
`class RetrofitManager private constructor() `
* 初始化
```
  private val instance: RetrofitManager by lazy(mode = LazyThreadSafetyMode.SYNCHRONIZED) {
          RetrofitManager()
      }
```
* 然后通过谷歌爸爸提供的Demo，我来修改一下我的单利实现的方式，由于项目中有些遗留原因，后续回去更新一下代码
![image.png](https://upload-images.jianshu.io/upload_images/5363507-35b3585dd630e261.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* **10、官方还在不断的更新中，好多的依赖都是alpha版本，所以不建议现在在实际项目中去使用**

#### 安利一个小工具，如何去使用vector绘制drawable的xml
* 举个例子
![](https://upload-images.jianshu.io/upload_images/5363507-ccc4d676d883390d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
<vector android:height="24dp" android:tint="#FFFFFF"
    android:viewportHeight="24.0" android:viewportWidth="24.0"
    android:width="24dp" xmlns:android="http://schemas.android.com/apk/res/android">
    <path android:fillColor="#FF000000" android:pathData="M18,16.08c-0.76,0 -1.44,0.3 -1.96,0.77L8.91,12.7c0.05,-0.23 0.09,-0.46 0.09,-0.7s-0.04,-0.47 -0.09,-0.7l7.05,-4.11c0.54,0.5 1.25,0.81 2.04,0.81 1.66,0 3,-1.34 3,-3s-1.34,-3 -3,-3 -3,1.34 -3,3c0,0.24 0.04,0.47 0.09,0.7L8.04,9.81C7.5,9.31 6.79,9 6,9c-1.66,0 -3,1.34 -3,3s1.34,3 3,3c0.79,0 1.5,-0.31 2.04,-0.81l7.12,4.16c-0.05,0.21 -0.08,0.43 -0.08,0.65 0,1.61 1.31,2.92 2.92,2.92 1.61,0 2.92,-1.31 2.92,-2.92s-1.31,-2.92 -2.92,-2.92z"/>
</vector>

```

*  **我们可以发现这个path毫无规律，根本都不知道怎么绘制，对吧？不要慌，看下图**
![image.png](https://upload-images.jianshu.io/upload_images/5363507-8ed2847941b33603.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/5363507-b0580a18635852fd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/5363507-09107b337ff2dd2d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* **同时也可以导入SVG的图片格式** 
推荐一个 [SVG在线作图](http://bigjpg.com/)的网站
![](https://upload-images.jianshu.io/upload_images/5363507-60b5b7944c1e2fbf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* **同时也可以使用 Iamge Asset 去生成自己想要的图片**

![image.png](https://upload-images.jianshu.io/upload_images/5363507-9ab3b2950a1273c9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/5363507-0aef9954c621c733.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


* **最后说声谢谢，这两天没有人打扰我，要不然累死了**
* 官方Demo的注释版本，我自己加了一些注释 **[android-sunflower](https://github.com/Shimingli/android-sunflower)**


>> * **感谢以下文档、博客和视屏对我的帮助**
>>> [Android Jetpack 官方文档 中文翻译](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation)
>>>[使用 Android Jetpack 加快应用开发速度](https://googledeveloperschina.blogspot.com/2018/05/android-jetpack.html)
>>>[YouTube](https://www.youtube.com/playlist?list=PLWz5rJ2EKKc9mxIBd0DRw9gwXuQshgmn2)
>>>  [常用集合的原理分析]([https://www.jianshu.com/p/a5f638bafd3b](https://www.jianshu.com/p/a5f638bafd3b))
>>>[SVG在线作图](http://bigjpg.com/)






